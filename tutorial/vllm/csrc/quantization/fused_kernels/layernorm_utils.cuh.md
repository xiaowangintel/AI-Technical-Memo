# layernorm_utils.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/fused_kernels/layernorm_utils.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides reusable device helpers for RMS-norm-style quantization, including scalar and vectorized implementations of RMS computation, dynamic scale calculation, and quantized writeback. / [CN] 提供 RMSNorm 风格量化所需的可复用设备端辅助函数，包括标量版与向量化版的 RMS 计算、动态 scale 计算以及量化写回。

## Line-by-Line Analysis / 逐行分析

### Scalar RMS computation
```cpp
// has_residual must be true, if residual is not a nullptr
template <typename scalar_t, bool has_residual = false>
__device__ void compute_rms(float* rms, scalar_t const* __restrict__ input,
                            int32_t const hidden_size,
                            int32_t const input_stride, float const epsilon,
                            scalar_t const* __restrict__ residual = nullptr) {
  int64_t const input_token_offset =
      blockIdx.x * static_cast<int64_t>(input_stride);
  int64_t const token_offset = blockIdx.x * static_cast<int64_t>(hidden_size);
  // sum of squares
  float ss = 0.0f;

  for (auto i = threadIdx.x; i < hidden_size; i += blockDim.x) {
    float x = static_cast<float>(input[input_token_offset + i]);
    if constexpr (has_residual) {
      x += static_cast<float>(residual[token_offset + i]);
    }

    ss += x * x;
  }

  using BlockReduce = cub::BlockReduce<float, 1024>;
  __shared__ typename BlockReduce::TempStorage reduceStore;
  ss = BlockReduce(reduceStore).Reduce(ss, CubAddOp{}, blockDim.x);

  __shared__ float s_rms;
  if (threadIdx.x == 0) {
    s_rms = rsqrtf(ss / hidden_size + epsilon);
  }
  __syncthreads();

  *rms = s_rms;
}
```
**EN:** The scalar `compute_rms` walks over one token, optionally adds a residual tensor elementwise, accumulates the sum of squares, reduces it with CUB block reduction, and writes `rsqrt(mean_square + epsilon)` into shared memory for all threads to reuse.
**CN:** 标量版 `compute_rms` 会遍历一个 token，按需逐元素叠加 residual，累加平方和，再用 CUB block 规约得到总和，并把 `rsqrt(mean_square + epsilon)` 写入共享内存供所有线程复用。

### Warp-local max reducer
```cpp
__device__ float warpReduceMaxSpecialized(volatile float* val, int64_t tid,
                                          int64_t thread_in_warp,
                                          int64_t reduced_elems) {
  static_assert(WARP_SIZE == 32 || WARP_SIZE == 64);
  if constexpr (WARP_SIZE == 64) {
    if (thread_in_warp + 64 < reduced_elems)
      val[tid] = fmaxf(val[tid], val[tid + 64]);
  }
  if (thread_in_warp + 32 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 32]);
  if (thread_in_warp + 16 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 16]);
  if (thread_in_warp + 8 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 8]);
  if (thread_in_warp + 4 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 4]);
  if (thread_in_warp + 2 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 2]);
  if (thread_in_warp + 1 < reduced_elems)
    val[tid] = fmaxf(val[tid], val[tid + 1]);
  return val[tid];
}
```
**EN:** `warpReduceMaxSpecialized` is a manual shared-memory warp reduction used by grouped scale computation. It supports both 32-lane and 64-lane warp sizes and avoids a heavier generic reduction when only a small per-group maximum is needed.
**CN:** `warpReduceMaxSpecialized` 是一个基于共享内存的手写 warp 级最大值规约，用于分组 scale 计算。它同时兼容 32/64 lane warp，并避免在只需要较小分组最大值时使用更重的通用规约。

### Scalar grouped scale computation
```cpp
  if (group_size > 0) {
    int64_t num_groups = hidden_size / group_size;
    __shared__ float s_max_vals[1024];
    int64_t const threads_per_group = blockDim.x / num_groups;
    int64_t const thread_in_group = threadIdx.x % threads_per_group;
    int64_t const group_offset = threadIdx.x / threads_per_group * group_size;
    int64_t const thread_offset = group_offset + thread_in_group;
    int64_t const thread_end =
        min(group_offset + group_size, static_cast<int64_t>(hidden_size));
    for (auto i = thread_offset; i < thread_end; i += threads_per_group) {
      float x = static_cast<float>(input[input_token_offset + i]);
      if constexpr (has_residual) {
        x += static_cast<float>(residual[token_offset + i]);
      }
      x = static_cast<float>(static_cast<scalar_t>(x * rms) * weight[i]);
      block_absmax_val_maybe = fmaxf(block_absmax_val_maybe, fabsf(x));
    }
    s_max_vals[threadIdx.x] = block_absmax_val_maybe;
    __syncthreads();

    int64_t const warp_size = WARP_SIZE;
    int64_t const num_warps = blockDim.x / warp_size;
    int64_t const warp_id = threadIdx.x / warp_size;
    int64_t const thread_in_warp = threadIdx.x % warp_size;
    int64_t const groups_per_warp = (num_groups + num_warps - 1) / num_warps;
    for (auto i = 0; i < groups_per_warp; ++i) {
      int64_t const group_id = i * num_warps + warp_id;
      if (group_id < num_groups) {
        int64_t warp_start = group_id * threads_per_group;
        int64_t const start = warp_start + thread_in_warp;
        int64_t const warp_end = min(warp_start + threads_per_group,
                                     static_cast<int64_t>(hidden_size));
        for (auto j = start; j + warp_size < warp_end; j += warp_size) {
          s_max_vals[start] =
              fmaxf(s_max_vals[start], s_max_vals[j + warp_size]);
        }
        warpReduceMaxSpecialized(s_max_vals, start, thread_in_warp,
                                 min(warp_end - warp_start, warp_size));
      }
    }
    __syncthreads();

    if (thread_in_group == 0 && thread_offset < thread_end) {
      block_absmax_val_maybe = s_max_vals[threadIdx.x];
      float scale = 0.0f;
      if (scale_ub) {
        scale = min(block_absmax_val_maybe, *scale_ub);
      } else {
        scale = block_absmax_val_maybe;
      }
      // token scale computation
      scale = max(scale / qmax, min_scaling_factor<scalar_out_t>::val());
      // Global output store
      if constexpr (is_scale_transposed) {
        int64_t const scale_rows = (gridDim.x + outer_scale_stride - 1) /
                                   outer_scale_stride * outer_scale_stride;
        all_token_scales[(threadIdx.x / threads_per_group) * scale_rows +
                         blockIdx.x] = scale;
      } else {
        all_token_scales[blockIdx.x * num_groups +
                         threadIdx.x / threads_per_group] = scale;
      }
    }
    __syncthreads();
```
**EN:** When `group_size > 0`, threads are partitioned into logical groups, each group computes the post-norm absolute maximum for its hidden slice, and one thread per group writes that scale either in token-major or transposed layout. This is the per-block/per-group path used by grouped quantization.
**CN:** 当 `group_size > 0` 时，线程会被划分为多个逻辑 group；每个 group 计算自己隐藏维片段在归一化后的绝对值最大值，再由每组一个线程按 token-major 或转置布局写出 scale。这就是分块/分组量化使用的路径。

### Scalar per-token scale computation
```cpp
  } else {
    for (auto i = threadIdx.x; i < hidden_size; i += blockDim.x) {
      float x = static_cast<float>(input[input_token_offset + i]);
      if constexpr (has_residual) {
        x += static_cast<float>(residual[token_offset + i]);
      }

      x = static_cast<float>(static_cast<scalar_t>(x * rms) * weight[i]);
      block_absmax_val_maybe = fmaxf(block_absmax_val_maybe, fabsf(x));
    }
    using BlockReduce = cub::BlockReduce<float, 1024>;
    __shared__ typename BlockReduce::TempStorage reduceStore;
    block_absmax_val_maybe =
        BlockReduce(reduceStore)
            .Reduce(block_absmax_val_maybe, CubMaxOp{}, blockDim.x);

    __shared__ float s_token_scale;
    if (threadIdx.x == 0) {
      float scale = 0.0f;
      if (scale_ub) {
        scale = min(block_absmax_val_maybe, *scale_ub);
      } else {
        scale = block_absmax_val_maybe;
      }
      // token scale computation
      scale = max(scale / qmax, min_scaling_factor<scalar_out_t>::val());
      s_token_scale = scale;                 // Shared memory store
      all_token_scales[blockIdx.x] = scale;  // Global output store
    }
    __syncthreads();

    *token_scale = s_token_scale;
  }
```
**EN:** If `group_size == 0`, the helper reduces a single absolute maximum across the whole token and stores one scale per token. This is the dynamic per-token quantization mode.
**CN:** 如果 `group_size == 0`，该辅助函数会在整个 token 范围上规约出单个绝对值最大值，并为每个 token 存一份 scale。这对应动态 per-token 量化模式。

### Scalar norm-and-quant path
```cpp
template <typename scalar_t, typename scalar_out_t, bool is_scale_inverted,
          bool has_residual = false, bool is_scale_transposed = false>
__device__ void norm_and_quant(
    scalar_out_t* __restrict__ output, scalar_t const* __restrict__ input,
    scalar_t const* __restrict__ weight, float const rms, float* const scale,
    int32_t const hidden_size, int32_t const input_stride,
    scalar_t* __restrict__ residual = nullptr, int32_t const group_size = 0,
    int64_t outer_scale_stride = 1) {
  int64_t const input_token_offset =
      blockIdx.x * static_cast<int64_t>(input_stride);
  int64_t const token_offset = blockIdx.x * static_cast<int64_t>(hidden_size);

  for (auto i = threadIdx.x; i < hidden_size; i += blockDim.x) {
    float x = static_cast<float>(input[input_token_offset + i]);
    if constexpr (has_residual) {
      x += static_cast<float>(residual[token_offset + i]);
      residual[token_offset + i] = static_cast<scalar_t>(x);
    }
    // Norm
    x = static_cast<float>(static_cast<scalar_t>(x * rms) * weight[i]);
    // Quant
    // If groupwise is_scale_inverted is true, so we invert the scale here.
    int64_t scale_idx = 0;
    if (group_size > 0) {
      if constexpr (is_scale_transposed) {
        int64_t const scale_rows = (gridDim.x + outer_scale_stride - 1) /
                                   outer_scale_stride * outer_scale_stride;
        scale_idx = (i / group_size) * scale_rows + blockIdx.x;
      } else {
        scale_idx = blockIdx.x * (hidden_size / group_size) + i / group_size;
      }
    }
    auto scale_val =
        (group_size > 0
             ? (is_scale_inverted ? 1.0f / scale[scale_idx] : scale[scale_idx])
             : *scale);
    output[token_offset + i] =
        ScaledQuant<scalar_out_t, is_scale_inverted>::quant_fn(x, scale_val);
  }
```
**EN:** `norm_and_quant` replays the input, optionally updates residual in place, applies RMS normalization with elementwise weights, selects either a shared token scale or a per-group scale, and finally quantizes with `ScaledQuant`. For int8 groupwise quantization it can invert scales on the fly.
**CN:** `norm_and_quant` 会再次遍历输入，按需原地更新 residual，执行 RMS 归一化与逐元素权重缩放，随后选择共享的 token scale 或按组 scale，最后用 `ScaledQuant` 完成量化。对于 int8 分组量化，它还可以在使用时按需取倒数。

### Vectorized RMS computation
```cpp
// Compute 1.0/rms(input)
// hidden_size must be a multiple of 4
template <typename scalar_t, bool has_residual = false>
__device__ void compute_rms(float* rms, scalar_t const* __restrict__ input,
                            int32_t const hidden_size,
                            int32_t const input_stride, float const epsilon,
                            scalar_t const* __restrict__ residual = nullptr) {
  int64_t const input_token_offset =
      blockIdx.x * static_cast<int64_t>(input_stride);
  int64_t const token_offset = blockIdx.x * static_cast<int64_t>(hidden_size);

  // Vectorized input/output to better utilize memory bandwidth.
  vec4_t<scalar_t> const* vec_input =
      reinterpret_cast<vec4_t<scalar_t> const*>(&input[input_token_offset]);
  vec4_t<scalar_t> const* vec_residual = nullptr;
  if constexpr (has_residual) {
    vec_residual =
        reinterpret_cast<vec4_t<scalar_t> const*>(&residual[token_offset]);
  }

  // sum of squares
  float ss = 0.0f;

  const int VEC_SIZE = 4;
  int32_t const num_vec_elems = hidden_size >> 2;

#pragma unroll 4
  for (auto i = threadIdx.x; i < num_vec_elems; i += blockDim.x) {
    vec4_t<scalar_t> in = vec_input[i];

    vec4_t<float> x;
#pragma unroll
    for (int j = 0; j < VEC_SIZE; ++j) {
      x.val[j] = static_cast<float>(in.val[j]);
    }

    if constexpr (has_residual) {
      vec4_t<scalar_t> r = vec_residual[i];
#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        x.val[j] += static_cast<float>(r.val[j]);
      }
    }

#pragma unroll
    for (int j = 0; j < VEC_SIZE; ++j) {
      ss += x.val[j] * x.val[j];
    }
  }

  using BlockReduce = cub::BlockReduce<float, 1024>;
  __shared__ typename BlockReduce::TempStorage reduceStore;
  ss = BlockReduce(reduceStore).Reduce(ss, CubAddOp{}, blockDim.x);

  __shared__ float s_rms;
  if (threadIdx.x == 0) {
    s_rms = rsqrtf(ss / hidden_size + epsilon);
  }
  __syncthreads();

  *rms = s_rms;
}
```
**EN:** The vectorized namespace reimplements the same logic over `vec4_t` chunks, assuming the hidden size is divisible by 4. It improves memory throughput by loading four elements at a time before the same CUB reduction.
**CN:** 向量化命名空间使用 `vec4_t` 分块重写了相同逻辑，前提是隐藏维可被 4 整除。它通过一次加载四个元素来提升内存吞吐，然后仍使用同样的 CUB 规约。

### Vectorized grouped scale computation
```cpp
  if constexpr (group_size > 0) {
    __shared__ float s_max_vals[1024];

    int64_t const num_groups = hidden_size / group_size;
    int64_t const threads_per_group = blockDim.x / num_groups;
    int64_t const thread_in_group = threadIdx.x % threads_per_group;
    int64_t const group_offset =
        threadIdx.x / threads_per_group * (group_size >> 2);
    int64_t const thread_offset = group_offset + thread_in_group;
    int64_t const thread_end = min(group_offset + (group_size >> 2),
                                   static_cast<int64_t>(hidden_size >> 2));
    vec_input =
        reinterpret_cast<vec4_t<scalar_t> const*>(&input[input_token_offset]);
    vec_weight = reinterpret_cast<vec4_t<scalar_t> const*>(weight);
    if constexpr (has_residual) {
      vec_residual =
          reinterpret_cast<vec4_t<scalar_t> const*>(&residual[token_offset]);
    }
    int32_t const num_vec_elems = thread_end;

#pragma unroll 4
    for (auto i = thread_offset; i < num_vec_elems; i += threads_per_group) {
      vec4_t<scalar_t> in = vec_input[i];
      vec4_t<scalar_t> const w = vec_weight[i];

      vec4_t<float> x;
#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        x.val[j] = static_cast<float>(in.val[j]);
      }

      if constexpr (has_residual) {
        vec4_t<scalar_t> r = vec_residual[i];
#pragma unroll
        for (int j = 0; j < VEC_SIZE; ++j) {
          x.val[j] += static_cast<float>(r.val[j]);
        }
      }

#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        block_absmax_val_maybe =
            fmaxf(block_absmax_val_maybe,
                  fabs(static_cast<scalar_t>(x.val[j] * rms) * w.val[j]));
      }
    }

    s_max_vals[threadIdx.x] = block_absmax_val_maybe;
    __syncthreads();

    int64_t const warp_size = WARP_SIZE;
    int64_t const num_warps = blockDim.x / warp_size;
    int64_t const warp_id = threadIdx.x / warp_size;
    int64_t const thread_in_warp = threadIdx.x % warp_size;
    int64_t const groups_per_warp = (num_groups + num_warps - 1) / num_warps;
    for (auto i = 0; i < groups_per_warp; ++i) {
      int64_t const group_id = i * num_warps + warp_id;
      if (group_id < num_groups) {
        int64_t warp_start = group_id * threads_per_group;
        int64_t const start = warp_start + thread_in_warp;
        int64_t const warp_end = min(warp_start + threads_per_group,
                                     static_cast<int64_t>(hidden_size));
        for (auto j = start; j + warp_size < warp_end; j += warp_size) {
          s_max_vals[start] =
              fmaxf(s_max_vals[start], s_max_vals[j + warp_size]);
        }
        warpReduceMaxSpecialized(s_max_vals, start, thread_in_warp,
                                 min(warp_end - warp_start, warp_size));
      }
    }
    __syncthreads();

    if (thread_in_group == 0 && thread_offset < thread_end) {
      block_absmax_val_maybe = s_max_vals[threadIdx.x];
      float scale = 0.0f;
      if (scale_ub) {
        scale = min(block_absmax_val_maybe, *scale_ub);
      } else {
        scale = block_absmax_val_maybe;
      }
      // token scale computation
      scale = max(scale / qmax, min_scaling_factor<scalar_out_t>::val());
      // Global output store
      if constexpr (is_scale_transposed) {
        int64_t const scale_rows = (gridDim.x + outer_scale_stride - 1) /
                                   outer_scale_stride * outer_scale_stride;
        all_token_scales[(threadIdx.x / threads_per_group) * scale_rows +
                         blockIdx.x] = scale;
      } else {
        all_token_scales[blockIdx.x * num_groups +
                         threadIdx.x / threads_per_group] = scale;
      }
    }
    __syncthreads();
```
**EN:** The grouped vectorized path partitions threads and vector lanes together: each thread processes `vec4_t` input/weight pairs, computes post-norm absolute maxima, and then uses the same shared-memory/warp reduction structure to emit one scale per group.
**CN:** 向量化的分组 scale 路径会同时考虑线程划分和向量 lane：每个线程处理 `vec4_t` 形式的输入/权重对，计算归一化后的绝对值最大值，再沿用同样的共享内存与 warp 规约结构，为每个 group 输出一个 scale。

### Vectorized token scale computation
```cpp
  } else {
    vec_input =
        reinterpret_cast<vec4_t<scalar_t> const*>(&input[input_token_offset]);
    vec_weight = reinterpret_cast<vec4_t<scalar_t> const*>(weight);
    if constexpr (has_residual) {
      vec_residual =
          reinterpret_cast<vec4_t<scalar_t> const*>(&residual[token_offset]);
    }

    int32_t const num_vec_elems = (hidden_size >> 2);

#pragma unroll 4
    for (auto i = threadIdx.x; i < num_vec_elems; i += blockDim.x) {
      vec4_t<scalar_t> in = vec_input[i];
      vec4_t<scalar_t> const w = vec_weight[i];

      vec4_t<float> x;
#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        x.val[j] = static_cast<float>(in.val[j]);
      }

      if constexpr (has_residual) {
        vec4_t<scalar_t> r = vec_residual[i];
#pragma unroll
        for (int j = 0; j < VEC_SIZE; ++j) {
          x.val[j] += static_cast<float>(r.val[j]);
        }
      }

#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        block_absmax_val_maybe =
            fmaxf(block_absmax_val_maybe,
                  fabs(static_cast<scalar_t>(x.val[j] * rms) * w.val[j]));
      }
    }

    using BlockReduce = cub::BlockReduce<float, 1024>;
    __shared__ typename BlockReduce::TempStorage reduceStore;
    block_absmax_val_maybe =
        BlockReduce(reduceStore)
            .Reduce(block_absmax_val_maybe, CubMaxOp{}, blockDim.x);

    __shared__ float s_token_scale;
    if (threadIdx.x == 0) {
      float scale = 0.0f;
      if (scale_ub) {
        scale = min(block_absmax_val_maybe, *scale_ub);
      } else {
        scale = block_absmax_val_maybe;
      }
      // token scale computation
      scale = max(scale / qmax, min_scaling_factor<scalar_out_t>::val());
      s_token_scale = scale;                 // shared memory store
      all_token_scales[blockIdx.x] = scale;  // global output store
    }
    __syncthreads();

    *token_scale = s_token_scale;
  }
```
**EN:** The non-grouped vectorized branch reduces a single maximum across the full token and stores it as `s_token_scale`, exactly mirroring the scalar algorithm but with wider loads and arithmetic.
**CN:** 非分组的向量化分支会在整个 token 范围上规约出单个最大值，并将其保存为 `s_token_scale`，算法与标量版本完全对应，只是加载与运算更宽。

### Vectorized norm-and-quant path
```cpp
// hidden_size must be a multiple of 4
template <typename scalar_t, typename scalar_out_t, bool is_scale_inverted,
          bool has_residual = false, bool is_scale_transposed = false,
          int32_t group_size = 0>
__device__ void norm_and_quant(
    scalar_out_t* __restrict__ output, scalar_t const* __restrict__ input,
    scalar_t const* __restrict__ weight, float const rms, float* const scale,
    int32_t const hidden_size, int32_t const input_stride,
    scalar_t* __restrict__ residual = nullptr, int64_t outer_scale_stride = 1) {
  int64_t const input_token_offset =
      blockIdx.x * static_cast<int64_t>(input_stride);
  int64_t const token_offset = blockIdx.x * static_cast<int64_t>(hidden_size);

  // Vectorized input/output/weight/residual to better utilize memory bandwidth.
  vec4_t<scalar_t> const* vec_input =
      reinterpret_cast<vec4_t<scalar_t> const*>(&input[input_token_offset]);
  vec4_t<scalar_t> const* vec_weight =
      reinterpret_cast<vec4_t<scalar_t> const*>(weight);
  q8x4_t<scalar_out_t>* vec_output =
      reinterpret_cast<q8x4_t<scalar_out_t>*>(&output[token_offset]);
  vec4_t<scalar_t>* vec_residual = nullptr;
  if constexpr (has_residual) {
    vec_residual = reinterpret_cast<vec4_t<scalar_t>*>(&residual[token_offset]);
  }

  const int VEC_SIZE = 4;
  int32_t const num_vec_elems = hidden_size >> 2;

// TODO(luka/varun) extract into type-agnostic vectorized quant function to
//  replace scaled_fp8_conversion_vec
#pragma unroll 4
  for (auto i = threadIdx.x; i < num_vec_elems; i += blockDim.x) {
    vec4_t<scalar_t> const in = vec_input[i];
    vec4_t<scalar_t> const w = vec_weight[i];

    vec4_t<float> x;
#pragma unroll
    for (int j = 0; j < VEC_SIZE; ++j) {
      x.val[j] = static_cast<float>(in.val[j]);
    }

    if constexpr (has_residual) {
      vec4_t<scalar_t> r = vec_residual[i];
#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        x.val[j] += static_cast<float>(r.val[j]);
      }
// Update residual
#pragma unroll
      for (int j = 0; j < VEC_SIZE; ++j) {
        r.val[j] = static_cast<scalar_t>(x.val[j]);
      }
      vec_residual[i] = r;
    }

    q8x4_t<scalar_out_t> out;

    float scale_val;

    if constexpr (group_size > 0) {
      int64_t const num_groups = hidden_size / group_size;
      int64_t scale_idx = 0;
      if constexpr (is_scale_transposed) {
        int64_t const scale_rows = (gridDim.x + outer_scale_stride - 1) /
                                   outer_scale_stride * outer_scale_stride;
        scale_idx = (i * VEC_SIZE / group_size) * scale_rows + blockIdx.x;
      } else {
        scale_idx = blockIdx.x * num_groups + i * VEC_SIZE / group_size;
      }
      scale_val =
          is_scale_inverted ? 1.0f / scale[scale_idx] : scale[scale_idx];
    } else {
      scale_val = *scale;
    }
#pragma unroll
    for (int j = 0; j < VEC_SIZE; ++j) {
      out.val[j] = ScaledQuant<scalar_out_t, is_scale_inverted>::quant_fn(
          static_cast<scalar_t>(x.val[j] * rms) * w.val[j], scale_val);
    }
    vec_output[i] = out;
  }
}
```
**EN:** The final helper quantizes four values at a time. It loads `vec4_t` input, weight, and optional residual values, updates residual if requested, resolves the correct scale index (including transposed grouped layouts), and writes packed 4-element quantized vectors through `q8x4_t`.
**CN:** 最后一个辅助函数一次量化四个值。它加载 `vec4_t` 形式的输入、权重和可选 residual，按需更新 residual，解析正确的 scale 索引（包括转置的分组布局），并通过 `q8x4_t` 写回打包好的 4 元量化向量。

## Key Concepts / 关键概念
- EN: The file separates scalar and vectorized implementations but keeps their numerical contract aligned.
  CN: 该文件把标量实现与向量化实现分开，但两者遵循相同的数值约定。
- EN: `group_size == 0` means one scale per token; `group_size > 0` means one scale per hidden-state block.
  CN: `group_size == 0` 表示每个 token 一个 scale；`group_size > 0` 表示每个隐藏维块一个 scale。
- EN: Residual support is fused into RMS, scale computation, and quantized writeback to avoid extra kernels.
  CN: residual 支持被融合进 RMS、scale 计算和量化写回中，以避免额外内核。

## Dependencies / 依赖关系
- EN: Uses CUB block reductions through `cub_helpers.h`.
  CN: 通过 `cub_helpers.h` 使用 CUB block 规约。
- EN: Depends on `vectorization.cuh` for `vec4_t` and `q8x4_t` packed vector types.
  CN: 依赖 `vectorization.cuh` 提供的 `vec4_t` 与 `q8x4_t` 打包向量类型。
- EN: Uses `quant_conversions.cuh` and `quantization/utils.cuh` for actual type-aware quantization math.
  CN: 使用 `quant_conversions.cuh` 与 `quantization/utils.cuh` 执行与类型相关的量化计算。
