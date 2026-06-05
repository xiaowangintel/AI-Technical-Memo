# activation_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/activation_kernels.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements fused activation/gating quantization kernels, including a simple elementwise path and a persistent FP8 kernel specialized for Mixture-of-Experts token routing. / [CN] 实现融合激活/门控量化内核，包括简单的逐元素路径，以及针对 MoE token 路由优化的持久化 FP8 内核。

## Line-by-Line Analysis / 逐行分析

### Elementwise act-and-mul quant kernel
```cpp
// Activation and gating kernel template.
template <typename scalar_t, scalar_t (*ACT_FN)(const scalar_t&),
          typename fp8_type>
__global__ void act_and_mul_quant_kernel(
    fp8_type* __restrict__ out,          // [..., d]
    const scalar_t* __restrict__ input,  // [..., 2, d]
    const float* scale, const int d) {
  const int32_t blocks_per_token = gridDim.y;

  const int32_t elems_per_128bit_load = (128 / 8) / sizeof(scalar_t);

  // We don't expect the hidden dimension to exceed 32 bits so int32 should
  // be safe here.
  const int32_t tgt_elems_per_block = div_ceil(d, blocks_per_token);
  const int32_t elems_per_block =
      round_to_next_multiple_of(tgt_elems_per_block, elems_per_128bit_load);
  const int32_t block_start = blockIdx.y * elems_per_block;
  int32_t block_end = block_start + elems_per_block;
  block_end = block_end > d ? d : block_end;

  // token_idx is 64 bit to prevent 32 bit overflow when the number of tokens
  // is very large
  const int64_t token_idx = blockIdx.x;
  const scalar_t* __restrict__ x_ptr = input + token_idx * 2 * d;
  const scalar_t* __restrict__ y_ptr = input + token_idx * 2 * d + d;
  fp8_type* __restrict__ out_ptr = out + token_idx * d;

  // 128-bit vectorized code
  const int32_t vec_loop_end =
      round_to_previous_multiple_of(elems_per_128bit_load, block_end);
  const int32_t vec_end_idx = vec_loop_end / elems_per_128bit_load;
  const int32_t vec_start_idx = block_start / elems_per_128bit_load;

  const int4* __restrict__ x_128bit_ptr = reinterpret_cast<const int4*>(x_ptr);
  const int4* __restrict__ y_128bit_ptr = reinterpret_cast<const int4*>(y_ptr);
  int2* __restrict__ out_128bit_ptr = reinterpret_cast<int2*>(out_ptr);

  float inverted_scale = 1 / *scale;
#pragma unroll
  for (int32_t vec_idx = vec_start_idx + threadIdx.x; vec_idx < vec_end_idx;
       vec_idx += blockDim.x) {
    const int4 x_128bit = VLLM_LDG(&x_128bit_ptr[vec_idx]);
    const int4 y_128bit = VLLM_LDG(&y_128bit_ptr[vec_idx]);
    using scalar_128bit_vec_t = std::array<scalar_t, elems_per_128bit_load>;
    using scalar_64bit_vec_t = std::array<fp8_type, elems_per_128bit_load>;

    scalar_64bit_vec_t out_vec;
    const auto x_vec = reinterpret_cast<scalar_128bit_vec_t const&>(x_128bit);
    const auto y_vec = reinterpret_cast<scalar_128bit_vec_t const&>(y_128bit);

#pragma unroll
    for (int i = 0; i < elems_per_128bit_load; i++) {
      out_vec[i] = scaled_fp8_conversion<true, fp8_type>(
          ACT_FN(x_vec[i]) * y_vec[i], inverted_scale);
    }

    out_128bit_ptr[vec_idx] = reinterpret_cast<const int2&>(out_vec);
  }

  // Scalar cleanup code
  if (block_end > vec_loop_end) {
    for (int64_t idx = vec_loop_end + threadIdx.x; idx < block_end;
         idx += blockDim.x) {
      const scalar_t x = VLLM_LDG(&x_ptr[idx]);
      const scalar_t y = VLLM_LDG(&y_ptr[idx]);
      out_ptr[idx] =
          scaled_fp8_conversion<true, fp8_type>(ACT_FN(x) * y, inverted_scale);
    }
  }
}
```
**EN:** `act_and_mul_quant_kernel` reads the two halves of the input tensor as gate/up activations, applies `ACT_FN` to the first half, multiplies with the second half, and quantizes the result to FP8. The kernel vectorizes 128-bit loads and stores whenever possible, then falls back to a scalar tail loop for the remainder.
**CN:** `act_and_mul_quant_kernel` 把输入张量的前后两半分别视为 gate/up 激活，对前半部分应用 `ACT_FN`，再与后半部分相乘并量化为 FP8。内核优先走 128-bit 向量化加载/存储，对尾部剩余元素再使用标量清理循环。

### Async copy and clipping helpers
```cpp
template <typename T, typename U>
__device__ __forceinline__ void cp_async4(T* _smem_ptr, const U* _glob_ptr) {
#if __CUDACC_VER_MAJOR__ >= 11 && __CUDA_ARCH__ >= 800
  auto smem_ptr = reinterpret_cast<void*>(_smem_ptr);
  auto glob_ptr = reinterpret_cast<const void*>(_glob_ptr);
  const int BYTES = 16;
  uint32_t smem = static_cast<uint32_t>(__cvta_generic_to_shared(smem_ptr));
  asm volatile(
      "{\n"
      "   cp.async.cg.shared.global [%0], [%1], %2;\n"
      "}\n" ::"r"(smem),
      "l"(glob_ptr), "n"(BYTES));
#else
  _smem_ptr[0] = _glob_ptr[0];
#endif
}

__device__ __forceinline__ void cp_async_fence() {
#if __CUDACC_VER_MAJOR__ >= 11 && __CUDA_ARCH__ >= 800
  asm volatile("cp.async.commit_group;\n" ::);
#else
#endif
}

template <int N>
__device__ __forceinline__ void cp_async_wait() {
#if __CUDACC_VER_MAJOR__ >= 11 && __CUDA_ARCH__ >= 800
  asm volatile("cp.async.wait_group %0;\n" ::"n"(N));
#else
#endif
}

template <>
__device__ __forceinline__ void cp_async_wait<0>() {
#if __CUDACC_VER_MAJOR__ >= 11 && __CUDA_ARCH__ >= 800
  asm volatile("cp.async.wait_all;\n" ::);
#else
#endif
}

__device__ __forceinline__ float clip(float v, float mmin, float mmax) {
  return fminf(mmax, fmaxf(v, mmin));
}

__device__ __forceinline__ __nv_bfloat16 clip(__nv_bfloat16 v,
                                              __nv_bfloat16 mmin,
                                              __nv_bfloat16 mmax) {
  return __hmin(mmax, __hmax(v, mmin));
}

__device__ __forceinline__ __nv_bfloat162 clip(__nv_bfloat162 v,
                                               __nv_bfloat162 mmin,
                                               __nv_bfloat162 mmax) {
  return __hmin2(mmax, __hmax2(v, mmin));
}

// We use the following values for fp8 min/max:
//  __nv_fp8_e4m3 = (-448, +448)
//  __nv_fp8_e4m3uz = (-240.0, +240.0)
// It is currently assumed that only
template <class T>
constexpr __nv_bfloat16 get_fp8_max() {
  static_assert(std::is_same_v<T, c10::Float8_e4m3fn> ||
                std::is_same_v<T, c10::Float8_e4m3fnuz>);
  if constexpr (std::is_same_v<T, c10::Float8_e4m3fn>) {
    return __nv_bfloat16(__nv_bfloat16_raw{.x = 17376});
  } else {
    return __nv_bfloat16(__nv_bfloat16_raw{.x = 17264});
  }
}

template <class T>
constexpr __nv_bfloat16 get_fp8_min() {
  static_assert(std::is_same_v<T, c10::Float8_e4m3fn> ||
                std::is_same_v<T, c10::Float8_e4m3fnuz>);
  if constexpr (std::is_same_v<T, c10::Float8_e4m3fn>) {
    return __nv_bfloat16(__nv_bfloat16_raw{.x = 50144});
  } else {
    return __nv_bfloat16(__nv_bfloat16_raw{.x = 50032});
  }
}
```
**EN:** This block defines `cp_async` wrappers, clip helpers, and FP8 min/max constants for BF16 math. They isolate backend-specific details so the persistent kernel can pipeline global-memory loads into shared memory and then clamp the normalized values into the valid FP8 range.
**CN:** 这一段定义了 `cp_async` 包装器、裁剪辅助函数，以及 BF16 计算所需的 FP8 最小/最大值常量。它把后端相关细节隔离出来，使持久化内核能够把全局内存加载流水化到共享内存，并将归一化后的数值裁剪到合法的 FP8 范围。

### Expert/token scheduling helpers
```cpp
template <typename Idx_t>
__device__ __forceinline__ int warp_expert_search(
    int idx, int n, const Idx_t* __restrict__ input, Idx_t val) {
  const Idx_t* input_ptr = input + idx;
  int base_offset = 0;

  for (;;) {
    bool move_on = (idx < n && *input_ptr <= val);

    unsigned mask = __ballot_sync(0xffffffff, move_on);

    if (mask != 0xffffffffu) {
      int last_lane = 31 - __clz(mask);
      return base_offset + last_lane;
    }

    input_ptr += 32;
    base_offset += 32;
    idx += 32;
  }
}

template <int num_parallel_tokens>
__device__ __forceinline__ void token_bounds(int32_t n_tokens,
                                             int32_t worker_id,
                                             int32_t& n_tokens_lower,
                                             int32_t& n_tokens_upper) {
  if (n_tokens < num_parallel_tokens && worker_id < n_tokens) {
    if (worker_id >= num_parallel_tokens) return;
    n_tokens_lower = worker_id;
    n_tokens_upper = worker_id + 1;
  } else {
    int32_t chunk_size = n_tokens / num_parallel_tokens;
    int32_t residual = n_tokens - chunk_size * num_parallel_tokens;
    auto calc_id = [&](int32_t id) {
      if (id < residual)
        return min(n_tokens, id * (chunk_size + 1));
      else
        return min(n_tokens, id * chunk_size + residual);
    };
    n_tokens_lower = calc_id(worker_id);
    n_tokens_upper = calc_id(worker_id + 1);
  }
}
```
**EN:** `warp_expert_search` maps a flattened token index back to its owning expert using a warp-synchronous search over prefix sums, and `token_bounds` divides total work among persistent blocks with better balance than a naive ceiling division.
**CN:** `warp_expert_search` 通过 warp 同步搜索前缀和，把扁平化 token 索引映射回所属 expert；`token_bounds` 则以比简单向上取整更均衡的方式，将总工作分配给持久化 block。

### Persistent kernel setup
```cpp
__global__ void silu_mul_fp8_quant_deep_gemm_kernel(
    const __nv_bfloat16* __restrict__ _input, fp8_type* __restrict__ _y_q,
    scale_t* __restrict__ _y_s, const int32_t* __restrict__ tokens_per_expert,
    // sizes
    Idx_t E, Idx_t T, Idx_t H,
    // strides (in elements)
    Idx_t stride_i_e, Idx_t stride_i_t, Idx_t stride_i_h, Idx_t stride_yq_e,
    Idx_t stride_yq_t, Idx_t stride_yq_h, Idx_t stride_ys_e, Idx_t stride_ys_t,
    Idx_t stride_ys_g, Idx_t stride_ys_p, Idx_t stride_counts_e) {
#ifndef USE_ROCM
  static constexpr int NUM_WARPS = THREADS / WARP_SIZE;

  static constexpr int LOAD_STAGE_SIZE = 2 * GROUP_SIZE / 8;
  static constexpr int LOAD_STAGE_MOD = NUM_STAGES * LOAD_STAGE_SIZE;

  static constexpr int COMPUTE_STAGE_SIZE = 2 * GROUP_SIZE / 4;
  static constexpr int COMPUTE_STAGE_MOD = COMPUTE_STAGE_SIZE * NUM_STAGES;

  extern __shared__ __align__(16) __int128_t smem_128[];

  int* s_expert_offsets =
      reinterpret_cast<int*>(smem_128 + (SMEM_SIZE_BYTES_Y / 16));

  static constexpr __nv_bfloat16 fp8_min = get_fp8_min<fp8_type>();
  static constexpr __nv_bfloat16 fp8_max = get_fp8_max<fp8_type>();
  // We assign EPS with it's 16-bit unsigned counterpart to allow constexpr.
  static constexpr __nv_bfloat16 EPS = (__nv_bfloat16_raw{.x = 11996});
  int tid = threadIdx.x;
  int warp_id = tid >> 5;
  int lane_id = tid & 0x1f;

  int running_sum{};
  if (!warp_id) {
    for (int i = 0; i < E; i += WARP_SIZE) {
      bool valid = (i + threadIdx.x) < E;
      int value =
          (valid ? tokens_per_expert[i + threadIdx.x * stride_counts_e] : 0) +
          (!lane_id ? running_sum : 0);

      for (int offset = 1; offset < 32; offset *= 2) {
        int n = __shfl_up_sync(0xFFFFFFFFu, value, offset);
        if (lane_id >= offset) value += n;
      }

      if (valid) {
        s_expert_offsets[i + threadIdx.x + 1] = value;
      }

      running_sum = __shfl_sync(0xFFFFFFFFu, value, WARP_SIZE - 1);
    }

    if (!lane_id) {
      s_expert_offsets[0] = 0;
    }
  }

  __syncthreads();

  int32_t total_tokens = s_expert_offsets[E];

  const int warp_position_yq = warp_id * (H / NUM_WARPS);
  const int warp_position_scales = warp_id * (H / (GROUP_SIZE * NUM_WARPS));

  // A single block will handle tokens_per_block tokens.
  // Each block i iterates over tokens of a slice of n_tokens =
  // expert_counts[i], with the size of chunk being
  // (n_tokens / NUM_PARALLEL_TOKENS) + residual, instead of
  // updiv(n_tokens, NUM_PARALLEL_TOKENS) for better scheduling.

  // Each warp will get space to store its hidden dim for gate and up.
  __int128_t* s_hidden_load = smem_128 + warp_id * ((2 * 128 / 8) * NUM_STAGES);
  __int128_t* smem_load_ptr = s_hidden_load + lane_id;

  const __nv_bfloat16 fp8_inv = __hdiv(__float2bfloat16(1.f), fp8_max);

  int32_t compute_pipeline_offset_64 = 0;
  int32_t load_stage_offset{};
  const __nv_bfloat16 one_bf16 = __float2bfloat16_rn(1.f);
```
**EN:** The main persistent kernel precomputes expert prefix sums in shared memory, determines each warp's slice of hidden-state output and scale groups, and lays out shared memory as a multi-stage load/compute pipeline. The design assumes BF16 input, FP8 output, and a fixed group size of 128.
**CN:** 主持久化内核先在共享内存中计算 expert 的前缀和，确定每个 warp 负责的隐藏维输出片段与 scale group，并把共享内存组织成多阶段的加载/计算流水线。该设计假定输入为 BF16、输出为 FP8，且 group size 固定为 128。

### Pipeline load stage
```cpp
  auto load_and_advance_y_pred = [&] {
    if (t_load < t_load_bound) {
      // Here we are simply continuing to load data
      // from the current token.
      auto smem_load_ptr_staged = smem_load_ptr + load_stage_offset;

      // It is very important that LOAD_STAGE_SIZE is constexpr to avoid
      // unnecessary ALU ops.
      load_stage_offset += LOAD_STAGE_SIZE;
      load_stage_offset %= LOAD_STAGE_MOD;

      cp_async4(smem_load_ptr_staged, load_ptr);
      load_ptr += GROUP_SIZE / 8;
      ++t_load;
    } else if (token_id + 1 < tokens_upper) {
      // We loaded everything from the current token, let's move on
      // to the next one, and we checked that we have more tokens to load.
      ++token_id;
      t_load = 0;
      if (token_id >= next_expert_offset) {
        // We need to find the next expert.
        do {
          // This is a loop because it's possible
          // that some experts are assigned 0 tokens.
          // NOTE: We are guaranteed that there's at least
          // one more token left so we don't have to check for
          // expert_id bounds.
          ++expert_id;
          // This skips 1 memory read.
          expert_offset = next_expert_offset;
          next_expert_offset = s_expert_offsets[expert_id + 1];
        } while (next_expert_offset == expert_offset);

        base_i = expert_id * (stride_i_e / 8);
        token_offset = 0;
        load_ptr = const_cast<__int128_t*>(input_128_ptr + base_i);
      } else {
        // We remain within the same expert, so just
        // move by H/4 __int128_t (2 * H/8).
        base_i += stride_yq_t / 4;
        token_offset++;
      }

      load_ptr = const_cast<__int128_t*>(input_128_ptr + base_i);

      auto smem_load_ptr_staged = smem_load_ptr + load_stage_offset;

      // It is very important that LOAD_STAGE_SIZE is constexpr to avoid
      // unnecessary ALU ops.
      load_stage_offset += LOAD_STAGE_SIZE;
      load_stage_offset %= LOAD_STAGE_MOD;

      cp_async4(smem_load_ptr_staged, load_ptr);
      load_ptr += GROUP_SIZE / 8;
      ++t_load;
    }
    // We fence even if there is nothing to load to simplify pipelining.
    cp_async_fence();
  };
```
**EN:** The `load_and_advance_y_pred` lambda is the heart of the producer side. It issues `cp.async` loads for gate/up data, advances stage offsets modulo the pipeline depth, and transparently rolls over from one token or expert to the next when the current slice is exhausted.
**CN:** `load_and_advance_y_pred` lambda 是生产者侧的核心。它负责为 gate/up 数据发起 `cp.async` 加载、按流水线深度循环推进 stage 偏移，并在当前切片耗尽时自动切换到下一个 token 或 expert。

### Pipeline compute and quantize stage
```cpp
    for (int i = 0; i < COMPUTE_LIMIT; i++) {
      cp_async_wait<NUM_STAGES - 2>();
      __syncthreads();
      load_and_advance_y_pred();

      __int64_t* gate64_ptr = s_gate64_ptr + compute_pipeline_offset_64;
      __int64_t* up64_ptr = s_up64_ptr + compute_pipeline_offset_64;

      // COMPUTE_STAGE_SIZE/MOD must also be constexpr!
      compute_pipeline_offset_64 += COMPUTE_STAGE_SIZE;
      compute_pipeline_offset_64 %= COMPUTE_STAGE_MOD;

      __int64_t gate64 = *gate64_ptr;
      __int64_t up64 = *up64_ptr;

      // Compute
      __nv_bfloat162 res[2];
      __nv_bfloat162* s_up_comp = reinterpret_cast<__nv_bfloat162*>(&up64);
      __nv_bfloat162* s_gate_comp = reinterpret_cast<__nv_bfloat162*>(&gate64);

  #pragma unroll
      for (int32_t k = 0; k < 2; ++k) {
        __nv_bfloat162 gate = silu2_v2(__bfloat1622float2(s_gate_comp[k]));
        res[k] = __hmul2(gate, s_up_comp[k]);
      }

      auto _y_max2 = __hmax2(__habs2(res[0]), __habs2(res[1]));

      _y_max2.x = __hmax(__hmax(_y_max2.x, _y_max2.y), EPS);

      __nv_bfloat16 y_s = __hmul(warp_max(_y_max2.x), fp8_inv);

      if constexpr (CEIL_UE8M0) {
        y_s = hexp2(hceil(hlog2(y_s)));
      }

      __nv_bfloat16 inv_y = __hdiv(one_bf16, y_s);

      auto y_s2 = make_bfloat162(inv_y, inv_y);

  #pragma unroll
      for (int32_t k = 0; k < 2; ++k) {
        res[k] = clip(__hmul2(res[k], y_s2), __bfloat162bfloat162(fp8_min),
                      __bfloat162bfloat162(fp8_max));
      }

      *y_q_ptr = __nv_fp8x4_e4m3(res[0], res[1]);
      y_q_ptr += WARP_SIZE * stride_yq_h;

      if (!lane_id) {
        // Store scales.
        if constexpr (std::is_same<scale_t, uint8_t>::value) {
          // Packed UE8M0 format. Remove Mantissa.
          *y_s_ptr = reinterpret_cast<int16_t&>(y_s) >> 7;

          bool const jump_pack = (current_group_id + 1) % 4 == 0;
          // Minus 3 because we need to get to the first group in the
          // next pack.
          y_s_ptr += jump_pack ? (stride_ys_p - 3) : stride_ys_g;

        } else {
          // float32 format
          static_assert(std::is_same<scale_t, float>::value);
          *y_s_ptr = y_s;
          y_s_ptr += stride_ys_g;
        }

        current_group_id += 1;
      }
```
**EN:** After waiting for the load stages, each warp computes `SiLU(gate) * up` on BF16 pairs, finds the per-group absolute maximum, derives the output scale, optionally rounds that scale up to UE8M0 powers of two, clips the normalized values into FP8 range, and finally stores both packed FP8 values and their scales.
**CN:** 在等待加载阶段完成后，每个 warp 会对 BF16 成对数据计算 `SiLU(gate) * up`，求出每个 group 的绝对值最大值，推导输出 scale，并在需要时把 scale 向上取整到 UE8M0 的 2 次幂形式，然后把归一化结果裁剪到 FP8 范围，最终写回打包后的 FP8 值及其 scale。

### Simple PyTorch launcher
```cpp
// Launch activation, gating, and quantize kernel.
#define LAUNCH_ACTIVATION_GATE_KERNEL(KERNEL)                               \
  int d = input.size(-1) / 2;                                               \
  int64_t num_tokens = input.numel() / input.size(-1);                      \
  dim3 grid(num_tokens, num_tokens > 16 ? num_tokens > 32 ? 1 : 2 : 4);     \
  dim3 block(std::min(d, 512));                                             \
  const at::cuda::OptionalCUDAGuard device_guard(device_of(input));         \
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();             \
  VLLM_DISPATCH_FLOATING_TYPES(                                             \
      input.scalar_type(), "act_and_mul_kernel", [&] {                      \
        VLLM_DISPATCH_FP8_TYPES(                                            \
            out.scalar_type(), "fused_add_rms_norm_kernel_fp8_type", [&] {  \
              vllm::act_and_mul_quant_kernel<scalar_t, KERNEL<scalar_t>,    \
                                             fp8_t>                         \
                  <<<grid, block, 0, stream>>>(out.data_ptr<fp8_t>(),       \
                                               input.data_ptr<scalar_t>(),  \
                                               scale.data_ptr<float>(), d); \
            });                                                             \
      });

void silu_and_mul_quant(torch::Tensor& out,    // [..., d]
                        torch::Tensor& input,  // [..., 2 * d]
                        torch::Tensor& scale) {
  TORCH_CHECK(out.dtype() == torch::kFloat8_e4m3fn ||
              out.dtype() == torch::kFloat8_e4m3fnuz);
  TORCH_CHECK(input.dtype() == torch::kFloat16 ||
              input.dtype() == torch::kBFloat16);
  TORCH_CHECK(input.size(-1) % 2 == 0);
  LAUNCH_ACTIVATION_GATE_KERNEL(vllm::silu_kernel);
}
```
**EN:** The `LAUNCH_ACTIVATION_GATE_KERNEL` macro and `silu_and_mul_quant()` wrapper expose the simple fused path to PyTorch. They validate dtypes, derive token count and hidden size, pick a grid that splits long rows over multiple blocks, and instantiate the template over input and FP8 output types.
**CN:** `LAUNCH_ACTIVATION_GATE_KERNEL` 宏与 `silu_and_mul_quant()` 包装器把简单融合路径暴露给 PyTorch。它们会校验 dtype，推导 token 数与隐藏维大小，为长行选择可拆分的 grid，并按输入类型与 FP8 输出类型实例化模板。

### Persistent kernel launch selection
```cpp
#ifndef USE_ROCM

  // This kernel currently only supports H % 128 == 0 and assumes a
  // fixed GROUP_SIZE of 128.
  static constexpr int GROUP_SIZE = 128;

  TORCH_CHECK(input.dtype() == torch::kBFloat16);
  TORCH_CHECK(y_q.dtype() == torch::kFloat8_e4m3fn ||
              y_q.dtype() == torch::kFloat8_e4m3fnuz);
  TORCH_CHECK(input.size(-1) % (GROUP_SIZE * 2) == 0);

  bool const is_packed_ue8m0 =
      (y_s.dtype() == torch::kInt32 && cast_scale_ue8m0);
  TORCH_CHECK(y_s.dtype() == torch::kFloat32 || is_packed_ue8m0);

  using Idx_t = int64_t;

  Idx_t E = input.size(0);
  Idx_t T = input.size(1);
  Idx_t H = input.size(2) / 2;
  Idx_t stride_i_e = input.stride(0);
  Idx_t stride_i_t = input.stride(1);
  Idx_t stride_i_h = input.stride(2);
  Idx_t stride_yq_e = y_q.stride(0);
  Idx_t stride_yq_t = y_q.stride(1);
  Idx_t stride_yq_h = y_q.stride(2);

  Idx_t stride_counts_e = tokens_per_expert.stride(0);

  int const NUM_GROUPS = H / GROUP_SIZE;

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  // TODO: Get this from cuda_arch ?
  static constexpr int SILU_V2_BLOCK_COUNT = 132 * 32;

  #define KERNEL(BLOCK_COUNT, scale_t, STRIDE_YS_E, STRIDE_YS_T, STRIDE_YS_G,  \
                 STRIDE_YS_P, CEIL_UE8M0, THREAD_COUNT, STAGES)                \
    static constexpr int NUM_WARPS = THREAD_COUNT / WARP_SIZE;                 \
    int sms = SILU_V2_BLOCK_COUNT;                                             \
    static constexpr int max_shared_mem_bytes =                                \
        GROUP_SIZE * 2 * STAGES * NUM_WARPS * 2;                               \
    dim3 grid(sms), block(THREAD_COUNT);                                       \
    const at::cuda::OptionalCUDAGuard device_guard(device_of(input));          \
    VLLM_DISPATCH_FP8_TYPES(                                                   \
        y_q.scalar_type(), "silu_mul_fp8_quant_deep_gemm_kernel", [&] {        \
          vllm::silu_mul_fp8_quant_deep_gemm_kernel<                           \
              BLOCK_COUNT, max_shared_mem_bytes, fp8_t, scale_t, THREAD_COUNT, \
              Idx_t, CEIL_UE8M0, GROUP_SIZE, STAGES>                           \
              <<<grid, block, max_shared_mem_bytes + (E + 1) * 16, stream>>>(  \
                  reinterpret_cast<__nv_bfloat16*>(input.data_ptr()),          \
                  (fp8_t*)y_q.data_ptr(),                                      \
                  reinterpret_cast<scale_t*>(y_s.data_ptr()),                  \
                  reinterpret_cast<int32_t*>(tokens_per_expert.data_ptr()), E, \
                  T, H, stride_i_e, stride_i_t, stride_i_h, stride_yq_e,       \
                  stride_yq_t, stride_yq_h, STRIDE_YS_E, STRIDE_YS_T,          \
                  STRIDE_YS_G, STRIDE_YS_P, stride_counts_e);                  \
        });

  #define LAUNCH_ON_H(scale_t, STRIDE_YS_E, STRIDE_YS_T, STRIDE_YS_G,         \
                      STRIDE_YS_P, CEIL_UE8M0)                                \
    if (H >= 4096 && (NUM_GROUPS % 8) == 0) {                                 \
      /* 8 warp config */                                                     \
      static constexpr int NUM_STAGES = 4;                                    \
      static constexpr int THREAD_COUNT = 256;                                \
      KERNEL(SILU_V2_BLOCK_COUNT, scale_t, STRIDE_YS_E, STRIDE_YS_T,          \
             STRIDE_YS_G, STRIDE_YS_P, CEIL_UE8M0, THREAD_COUNT, NUM_STAGES); \
    } else {                                                                  \
      /* 1 warp config */                                                     \
      static constexpr int THREAD_COUNT = 32;                                 \
      KERNEL(SILU_V2_BLOCK_COUNT, scale_t, STRIDE_YS_E, STRIDE_YS_T,          \
             STRIDE_YS_G, STRIDE_YS_P, CEIL_UE8M0, THREAD_COUNT, 2);          \
    }
```
**EN:** The persistent launcher checks layout constraints, decides whether scales are stored as FP32 or packed UE8M0 bytes, and picks either a 1-warp or 8-warp configuration based on hidden size. Those launch macros specialize both thread count and pipeline stage count at compile time.
**CN:** 持久化启动逻辑会检查布局约束，判断 scale 是以 FP32 还是打包 UE8M0 字节形式存储，并根据隐藏维大小在 1-warp 或 8-warp 配置之间做选择。这些启动宏会在编译期固化线程数与流水线 stage 数。

### Packed UE8M0 scale layout
```cpp
  // Int32 packed ue8m0 scales tensor.
  // Let E, T, G be the number to experts, number of tokens and number of groups
  // respectively. Let, E = 2, T = 4, G = 6, in this case the int32 scales
  // tensor are of shape [1, 4, 2] and stride [8, 1, 4]. The scales are expected
  // to be arranged as follows,
  // [[T0G0-T0G1-T0G2-T0G3, T0G4-T0G5-X-X,],
  //  [T1G0-T1G1-T1G2-T1G3, T1G4-T1G5-X-X,]
  //  [T2G0-T2G1-T2G2-T2G3, T2G4-T2G5-X-X,]
  //  [T3G0-T3G1-T3G2-T3G3, T3G4-T3G5-X-X,]]
  // where, TxGy is the scale ue8m0 scale value of Token x, Group y.
  //
  // In memory (in bytes) the scale values are arranged as,
  //  [T0G0, T0G1, T0G2, T0G3, T1G0, T1G2, T1G3, T1G4, T2G0, T2G1, T2G3, T2G4,
  //   T3G0, T3G1, T3G2, T3G3, T0G4, T0G5, X, X, T1G4, T1G5, X, X, T2G4, T2G5,
  //   X, X, T3G4, T3G5, X, X]
  //
  // An Int32 tensor of size [1, 4, 2] and stride [8, 1, 4] can be represented
  // as an uint8 tensor of shape [1, 2, 4, 4] and stride [32, 16, 4, 1]. In
  // english, ignoring the Experts dimension, the original int32 tensor is
  // simply treated as two packed [4, 4] uint8 tensor (or two [4, 1] int32
  // tensor). The following strides setting reflects this change. Caveat: This
  // means that the G dimension is no longer contiguous. i.e. Note that to move
  // from G3 to G4, we need to jump along the packing dimension. The kernel
  // handles this case.

  stride_ys_e *= sizeof(int32_t);
  stride_ys_p = T * sizeof(int32_t);  // Packing dimension
  stride_ys_t = sizeof(int32_t);
  stride_ys_g = 1;

  LAUNCH_ON_H(uint8_t, stride_ys_e, stride_ys_t, stride_ys_g, stride_ys_p,
              true);
```
**EN:** The final comment block documents how four group scales are packed into each `int32` when `y_s` uses packed UE8M0 storage. The kernel later interprets that tensor as a byte-addressed structure with a separate packing stride, which explains the non-contiguous jump between adjacent scale groups.
**CN:** 最后的注释块说明了在 `y_s` 使用打包 UE8M0 存储时，四个 group scale 如何压缩进一个 `int32`。后续内核会把这个张量解释成带有独立 packing stride 的按字节寻址结构，因此相邻 scale group 之间会出现非连续跳转。

## Key Concepts / 关键概念
- EN: There are two separate fused paths: a generic act-and-mul kernel and a persistent MoE-oriented FP8 kernel.
  CN: 这里包含两条独立的融合路径：通用 act-and-mul 内核，以及面向 MoE 的持久化 FP8 内核。
- EN: The persistent kernel overlaps global-memory loads with compute through a staged shared-memory pipeline.
  CN: 持久化内核通过分阶段共享内存流水线，让全局内存加载与计算重叠执行。
- EN: Scale handling supports FP32 output scales and UE8M0-friendly packed storage.
  CN: scale 处理同时支持 FP32 输出和适配 UE8M0 的打包存储。

## Dependencies / 依赖关系
- EN: Uses ATen CUDA stream/device helpers and vLLM dispatch macros to instantiate kernels from PyTorch tensors.
  CN: 使用 ATen 的 CUDA stream/device 辅助函数以及 vLLM dispatch 宏，从 PyTorch 张量实例化内核。
- EN: Depends on `quantization/w8a8/fp8/common.cuh` for FP8 conversion helpers such as `scaled_fp8_conversion`.
  CN: 依赖 `quantization/w8a8/fp8/common.cuh` 中的 FP8 转换辅助逻辑，例如 `scaled_fp8_conversion`。
- EN: Relies on CUDA warp intrinsics, `cp.async`, and BF16/FP8 vector types for the high-performance kernel.
  CN: 高性能内核依赖 CUDA warp intrinsic、`cp.async` 以及 BF16/FP8 向量类型。
