# fused_deepseek_v4_qnorm_rope_kv_insert_kernel.cu — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/fused_deepseek_v4_qnorm_rope_kv_insert_kernel.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Implements a DeepSeek-V4-specific fused kernel that performs Q-side RMSNorm+RoPE and KV-side RoPE+FP8 cache insertion in one pass. **CN:** 实现一个面向 DeepSeek-V4 的融合内核，在一次遍历中同时完成 Q 侧 RMSNorm+RoPE 以及 KV 侧 RoPE+FP8 cache insert。

## Line-by-Line Analysis / 逐行分析

### Fixed DeepSeek-V4 constants and warp helpers / 固定的 DeepSeek-V4 常量与 warp 辅助函数
```cpp
namespace {
inline int getSMVersion() {
  auto* props = at::cuda::getCurrentDeviceProperties();
  return props->major * 10 + props->minor;
}
}  // namespace

// ────────────────────────────────────────────────────────────────────────────
// Constants
// ────────────────────────────────────────────────────────────────────────────
constexpr int kHeadDim = 512;
constexpr int kRopeDim = 64;
constexpr int kNopeDim = kHeadDim - kRopeDim;  // 448
constexpr int kQuantBlock = 64;
constexpr int kNumQuantBlocks = kNopeDim / kQuantBlock;   // 7
constexpr int kScaleBytesPerToken = kNumQuantBlocks + 1;  // 8 (7 real + 1 pad)
constexpr int kTokenDataBytes = kNopeDim + kRopeDim * 2;  // 448 + 128 = 576
constexpr float kFp8Max = 448.0f;

// Per-warp layout:  32 lanes × 16 elems/lane = 512 elems = HEAD_DIM.
constexpr int kNumLanes = 32;
constexpr int kElemsPerLane = kHeadDim / kNumLanes;  // 16

// ────────────────────────────────────────────────────────────────────────────
// Small inline helpers
// ────────────────────────────────────────────────────────────────────────────
__device__ __forceinline__ float warp4MaxAbs(float val) {
  // Reduce absolute max across 4 consecutive lanes (lane id & 3 group).
  float peer = __shfl_xor_sync(FINAL_MASK, val, 1);
  val = fmaxf(val, peer);
  peer = __shfl_xor_sync(FINAL_MASK, val, 2);
  val = fmaxf(val, peer);
  return val;
}

template <typename T>
__device__ __forceinline__ float warpSum(float val) {
#pragma unroll
  for (int mask = 16; mask > 0; mask >>= 1) {
    val += __shfl_xor_sync(FINAL_MASK, val, mask, 32);
  }
  return val;
}
```
**EN:** The kernel hard-codes head_dim=512, rope_dim=64, the NoPE/rope split, quant block size, and the FP8 clipping range. warp4MaxAbs and warpSum provide the small warp-collective reductions needed for quantization and RMSNorm.
**CN:** 该内核把 head_dim=512、rope_dim=64、NoPE/rope 切分、量化块大小以及 FP8 截断范围全部写死。warp4MaxAbs 与 warpSum 提供了量化和 RMSNorm 所需的小规模 warp 级归约操作。

### Warp mapping and vectorized loads / warp 映射与向量化加载
```cpp
template <typename scalar_t_in>
__global__ void fusedDeepseekV4QNormRopeKVRopeQuantInsertKernel(
    scalar_t_in* __restrict__ q_inout,         // [N, H, 512] bf16, in place
    scalar_t_in const* __restrict__ kv_in,     // [N, 512] bf16
    uint8_t* __restrict__ k_cache,             // [num_blocks, block_stride]
    int64_t const* __restrict__ slot_mapping,  // [num_tokens_insert] i64
    int64_t const* __restrict__ position_ids,  // [N] i64
    float const* __restrict__ cos_sin_cache,   // [max_pos, 64] fp32
    float const eps,
    int const num_tokens_full,    // = q.size(0) = kv.size(0)
    int const num_tokens_insert,  // = slot_mapping.size(0), ≤ num_tokens_full
    int const num_heads_q,        // H
    int const cache_block_size,   // tokens per paged-cache block
    int const kv_block_stride) {  // bytes per paged-cache block
#if (!defined(__CUDA_ARCH__) || __CUDA_ARCH__ < 800) && !defined(USE_ROCM)
  // BF16 _typeConvert specialization is unavailable on pre-Ampere.  The
  // DeepseekV4 kernel only runs with bf16 inputs in practice, so compile a
  // no-op stub for sm_70/sm_75 to keep multi-arch builds happy.
  if constexpr (std::is_same_v<scalar_t_in, c10::BFloat16>) {
    return;
  } else {
#endif
    using Converter = vllm::_typeConvert<scalar_t_in>;

    int const warpsPerBlock = blockDim.x / 32;
    int const warpId = threadIdx.x / 32;
    int const laneId = threadIdx.x % 32;
    int const globalWarpIdx = blockIdx.x * warpsPerBlock + warpId;

    int const total_slots_per_token = num_heads_q + 1;
    int const tokenIdx = globalWarpIdx / total_slots_per_token;
    int const slotIdx = globalWarpIdx % total_slots_per_token;
    if (tokenIdx >= num_tokens_full) return;

    bool const isKV = (slotIdx == num_heads_q);
    // KV branch: skip DP-padded tokens (no slot reserved for them).
    if (isKV && tokenIdx >= num_tokens_insert) return;

    // PDL: wait for predecessor kernel (upstream q/kv producer) to signal
    // before touching any global memory.  No-op when PDL is not enabled on
    // the launch.  The CUDA runtime wrapper emits the griddepcontrol.wait
    // PTX with the required memory clobber internally.
#if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900)
    cudaGridDependencySynchronize();
#endif

    // Dim range this lane owns within the 512-wide head.
    int const dim_base = laneId * kElemsPerLane;  // in [0, 512) step 16

    // ── Load 16 bf16 → 16 fp32 registers (one 16-byte + one 16-byte LDG) ────
    float elements[kElemsPerLane];
    float sumOfSquares = 0.0f;

    scalar_t_in const* src_ptr;
    if (isKV) {
      src_ptr = kv_in + static_cast<int64_t>(tokenIdx) * kHeadDim + dim_base;
    } else {
      int64_t const q_row_offset =
          (static_cast<int64_t>(tokenIdx) * num_heads_q + slotIdx) * kHeadDim +
          dim_base;
      src_ptr = q_inout + q_row_offset;
    }

    // Two 16-byte loads per thread (8 bf16 each).  Use uint4 as the vector
    // type and bitcast to scalar_t_in packed pairs for conversion.
    uint4 v0 = *reinterpret_cast<uint4 const*>(src_ptr);
    uint4 v1 = *reinterpret_cast<uint4 const*>(src_ptr + 8);

    {
      typename Converter::packed_hip_type const* p0 =
          reinterpret_cast<typename Converter::packed_hip_type const*>(&v0);
      typename Converter::packed_hip_type const* p1 =
          reinterpret_cast<typename Converter::packed_hip_type const*>(&v1);
// Each packed_hip_type holds 2 bf16 → 4 packed = 8 elems per uint4.
#pragma unroll
      for (int i = 0; i < 4; i++) {
        float2 f2 = Converter::convert(p0[i]);
        elements[2 * i] = f2.x;
        elements[2 * i + 1] = f2.y;
      }
#pragma unroll
      for (int i = 0; i < 4; i++) {
        float2 f2 = Converter::convert(p1[i]);
        elements[8 + 2 * i] = f2.x;
        elements[8 + 2 * i + 1] = f2.y;
      }
    }
```
**EN:** Each warp is assigned one (token, head_slot) pair, where head_slot<num_heads_q means Q work and head_slot==num_heads_q means KV work. Every lane loads 16 elements using two uint4 transactions, immediately converting packed bf16 values into fp32 registers for math.
**CN:** 每个 warp 负责一个 (token, head_slot) 对，其中 head_slot<num_heads_q 表示处理 Q，head_slot==num_heads_q 表示处理 KV。每个 lane 通过两个 uint4 事务加载 16 个元素，并立刻把打包的 bf16 值转换为 fp32 寄存器用于计算。

### Q branch: RMSNorm plus RoPE in place / Q 分支：原地执行 RMSNorm 与 RoPE
```cpp
    // ── Q branch: RMSNorm with no weight (has_weight=False) ─────────────────
    // Variance + rsqrt + multiply all in fp32, no intermediate bf16 round.
    // The downstream bf16 round only happens at the final store.
    if (!isKV) {
#pragma unroll
      for (int i = 0; i < kElemsPerLane; i++) {
        sumOfSquares += elements[i] * elements[i];
      }
      sumOfSquares = warpSum<float>(sumOfSquares);
      float const rms_rcp =
          rsqrtf(sumOfSquares / static_cast<float>(kHeadDim) + eps);
#pragma unroll
      for (int i = 0; i < kElemsPerLane; i++) {
        elements[i] = elements[i] * rms_rcp;
      }
    }

    // ── GPT-J RoPE on dims [NOPE_DIM, HEAD_DIM) ─────────────────────────────
    // All math in fp32.  cos_sin_cache is loaded as fp32 (its native storage).
    bool const is_rope_lane = dim_base >= kNopeDim;
    if (is_rope_lane) {
      int64_t const pos = position_ids[tokenIdx];
      constexpr int kHalfRope = kRopeDim / 2;  // 32
      float const* cos_ptr = cos_sin_cache + pos * kRopeDim;
      float const* sin_ptr = cos_ptr + kHalfRope;

      int const rope_local_base = dim_base - kNopeDim;  // in [0, 64) step 16
#pragma unroll
      for (int p = 0; p < kElemsPerLane / 2; p++) {
        int const pair_dim = rope_local_base + 2 * p;
        int const half_idx = pair_dim / 2;
        float const cos_v = VLLM_LDG(cos_ptr + half_idx);
        float const sin_v = VLLM_LDG(sin_ptr + half_idx);
        float const x_even = elements[2 * p];
        float const x_odd = elements[2 * p + 1];
        elements[2 * p] = x_even * cos_v - x_odd * sin_v;
        elements[2 * p + 1] = x_even * sin_v + x_odd * cos_v;
      }
    }

    // ═══════════════════════════════════════════════════════════════════════
    // Q branch: cast to bf16 and store back in place.
    // ═══════════════════════════════════════════════════════════════════════
    if (!isKV) {
      uint4 out0, out1;
      typename Converter::packed_hip_type* po0 =
          reinterpret_cast<typename Converter::packed_hip_type*>(&out0);
      typename Converter::packed_hip_type* po1 =
          reinterpret_cast<typename Converter::packed_hip_type*>(&out1);
#pragma unroll
      for (int i = 0; i < 4; i++) {
        po0[i] = Converter::convert(
            make_float2(elements[2 * i], elements[2 * i + 1]));
      }
#pragma unroll
      for (int i = 0; i < 4; i++) {
        po1[i] = Converter::convert(
            make_float2(elements[8 + 2 * i], elements[8 + 2 * i + 1]));
      }
      scalar_t_in* dst =
          q_inout +
          (static_cast<int64_t>(tokenIdx) * num_heads_q + slotIdx) * kHeadDim +
          dim_base;
      *reinterpret_cast<uint4*>(dst) = out0;
      *reinterpret_cast<uint4*>(dst + 8) = out1;
#if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900)
      cudaTriggerProgrammaticLaunchCompletion();
#endif
      return;
    }
```
**EN:** For Q heads, the kernel reduces the sum of squares across the warp, computes the RMS reciprocal, scales all 512 dimensions in fp32, applies GPT-J style interleaved RoPE on the last 64 dimensions, and finally writes bf16 values back into q_inout.
**CN:** 对于 Q 头，内核会在 warp 内归约平方和，计算 RMS 倒数，在 fp32 中缩放全部 512 个维度，再对最后 64 个维度应用 GPT-J 风格的交错式 RoPE，最后把 bf16 结果写回 q_inout。

### KV branch: quantize NoPE and insert paged cache / KV 分支：量化 NoPE 并写入分页缓存
```cpp
    int64_t const slot_id = slot_mapping[tokenIdx];
    if (slot_id < 0) {
#if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900)
      cudaTriggerProgrammaticLaunchCompletion();
#endif
      return;
    }

    int64_t const block_idx = slot_id / cache_block_size;
    int64_t const pos_in_block = slot_id % cache_block_size;
    uint8_t* block_base =
        k_cache + block_idx * static_cast<int64_t>(kv_block_stride);
    uint8_t* token_fp8_ptr = block_base + pos_in_block * kTokenDataBytes;
    uint8_t* token_bf16_ptr = token_fp8_ptr + kNopeDim;
    uint8_t* token_scale_ptr =
        block_base + static_cast<int64_t>(cache_block_size) * kTokenDataBytes +
        pos_in_block * kScaleBytesPerToken;

    // Round K to bf16 first, matching the unfused reference path where K is
    // materialized as bf16 before K quantization.  absmax, clamp, and FP8
    // quant below all run on these bf16-rounded values.
#pragma unroll
    for (int i = 0; i < kElemsPerLane; i++) {
      elements[i] = Converter::convert(Converter::convert(elements[i]));
    }

    // Per-quant-block absmax must be computed by ALL 32 lanes (warp-collective
    // shuffle requires full participation).  RoPE lanes contribute garbage,
    // but their values are gated out below via `!is_rope_lane`.
    float local_absmax = 0.0f;
#pragma unroll
    for (int i = 0; i < kElemsPerLane; i++) {
      local_absmax = fmaxf(local_absmax, fabsf(elements[i]));
    }
    float const absmax = fmaxf(warp4MaxAbs(local_absmax), 1e-4f);
    float const exponent = ceilf(log2f(absmax / kFp8Max));
    float const inv_scale = exp2f(-exponent);

    if (!is_rope_lane) {
      // ── NoPE lane: UE8M0 FP8 quant ───────────────────────────────────────
      uint8_t out_bytes[kElemsPerLane];
#pragma unroll
      for (int i = 0; i < kElemsPerLane; i++) {
        float scaled = elements[i] * inv_scale;
        scaled = fminf(fmaxf(scaled, -kFp8Max), kFp8Max);
#ifndef USE_ROCM
        __nv_fp8_storage_t s =
            __nv_cvt_float_to_fp8(scaled, __NV_SATFINITE, __NV_E4M3);
        out_bytes[i] = static_cast<uint8_t>(s);
#else
      out_bytes[i] = rocm_cvt_float_to_fp8_e4m3(scaled);
#endif
      }
      // One 16-byte STG per lane.
      *reinterpret_cast<uint4*>(token_fp8_ptr + dim_base) =
          *reinterpret_cast<uint4 const*>(out_bytes);

      // Lane (4k) of each 4-lane group writes the scale byte for block k<7.
      if ((laneId & 3) == 0) {
        int const q_block_idx = laneId >> 2;  // 0..6 for NoPE lanes
        float encoded = fmaxf(fminf(exponent + 127.0f, 255.0f), 0.0f);
        token_scale_ptr[q_block_idx] = static_cast<uint8_t>(encoded);
      }
      // Lane 0 also writes the padding byte at index 7.
      if (laneId == 0) {
        token_scale_ptr[kNumQuantBlocks] = 0;  // pad
      }
    } else {
      // ── RoPE lane: cast back to bf16 and store to cache bf16 tail ────────
      uint4 out0, out1;
      typename Converter::packed_hip_type* po0 =
          reinterpret_cast<typename Converter::packed_hip_type*>(&out0);
      typename Converter::packed_hip_type* po1 =
          reinterpret_cast<typename Converter::packed_hip_type*>(&out1);
#pragma unroll
      for (int i = 0; i < 4; i++) {
        po0[i] = Converter::convert(
            make_float2(elements[2 * i], elements[2 * i + 1]));
      }
#pragma unroll
      for (int i = 0; i < 4; i++) {
        po1[i] = Converter::convert(
            make_float2(elements[8 + 2 * i], elements[8 + 2 * i + 1]));
      }
      int const rope_local_base = dim_base - kNopeDim;  // in [0, 64)
      scalar_t_in* bf16_dst =
          reinterpret_cast<scalar_t_in*>(token_bf16_ptr) + rope_local_base;
      *reinterpret_cast<uint4*>(bf16_dst) = out0;
      *reinterpret_cast<uint4*>(bf16_dst + 8) = out1;
    }
```
**EN:** For KV work, the kernel first resolves the paged-cache slot, rounds values back through bf16 to match the unfused reference path, computes one UE8M0 scale per 64-value NoPE block, stores quantized FP8 bytes for the first 448 dims, and stores the last 64 RoPE dims as bf16 tail data.
**CN:** 对于 KV 路径，内核先解析分页缓存中的 slot，再先经由 bf16 重新舍入以匹配未融合参考路径，然后为每个 64 值的 NoPE 块计算一个 UE8M0 scale，把前 448 维写成量化后的 FP8 字节，并把最后 64 个 RoPE 维度以 bf16 尾部数据形式保存。

### Launch wrapper with architecture gating / 带架构门控的启动包装器
```cpp
template <typename scalar_t_in>
void launchFusedDeepseekV4QNormRopeKVRopeQuantInsert(
    scalar_t_in* q_inout, scalar_t_in const* kv_in, uint8_t* k_cache,
    int64_t const* slot_mapping, int64_t const* position_ids,
    float const* cos_sin_cache, float const eps, int const num_tokens_full,
    int const num_tokens_insert, int const num_heads_q,
    int const cache_block_size, int const kv_block_stride,
    cudaStream_t stream) {
  constexpr int kBlockSize = 256;
  constexpr int kWarpsPerBlock = kBlockSize / 32;
  int64_t const total_warps =
      static_cast<int64_t>(num_tokens_full) * (num_heads_q + 1);
  int const grid =
      static_cast<int>((total_warps + kWarpsPerBlock - 1) / kWarpsPerBlock);

  // PDL: enable programmatic stream serialization whenever the hardware
  // supports it (SM90+).  On pre-Hopper GPUs the attribute is unavailable,
  // so leave numAttrs = 0 and launch as a regular kernel.
#ifndef USE_ROCM
  static int const sm_version = getSMVersion();
  // Host-side guard: the device kernel body is compiled as a no-op for
  // bf16 on pre-Ampere (sm_70/sm_75) because _typeConvert<BFloat16> is
  // unavailable there.  Refuse the launch loudly instead of silently
  // skipping the work.
  TORCH_CHECK(
      sm_version >= 80,
      "fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert requires sm_80+ "
      "(Ampere or newer); got sm_",
      sm_version);
  cudaLaunchConfig_t config;
  config.gridDim = dim3(grid);
  config.blockDim = dim3(kBlockSize);
  config.dynamicSmemBytes = 0;
  config.stream = stream;
  cudaLaunchAttribute attrs[1];
  attrs[0].id = cudaLaunchAttributeProgrammaticStreamSerialization;
  attrs[0].val.programmaticStreamSerializationAllowed = 1;
  config.attrs = attrs;
  config.numAttrs = (sm_version >= 90) ? 1 : 0;

  cudaLaunchKernelEx(
      &config, fusedDeepseekV4QNormRopeKVRopeQuantInsertKernel<scalar_t_in>,
      q_inout, kv_in, k_cache, slot_mapping, position_ids, cos_sin_cache, eps,
      num_tokens_full, num_tokens_insert, num_heads_q, cache_block_size,
      kv_block_stride);
#else
  // ROCm: use standard kernel launch syntax (no PDL/stream serialization)
  // clang-format off
  fusedDeepseekV4QNormRopeKVRopeQuantInsertKernel<scalar_t_in>
      <<<grid, kBlockSize, 0, stream>>>(
          q_inout, kv_in, k_cache, slot_mapping, position_ids, cos_sin_cache,
          eps, num_tokens_full, num_tokens_insert, num_heads_q,
          cache_block_size, kv_block_stride);
#endif
}
```
**EN:** The launcher computes the warp-based grid size, rejects pre-sm80 NVIDIA devices, and enables Hopper-era programmatic stream serialization when available via cudaLaunchKernelEx. ROCm falls back to a normal kernel launch because the CUDA-specific PDL attribute does not exist there.
**CN:** 启动器会根据 warp 级工作划分计算 grid 大小，拒绝 pre-sm80 的 NVIDIA 设备，并在可用时通过 cudaLaunchKernelEx 启用 Hopper 时代的 programmatic stream serialization。ROCm 则退回普通 kernel launch，因为 CUDA 专有的 PDL 属性在该平台不存在。

### Torch operator validation and dispatch / Torch 算子校验与分发
```cpp
void fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert(
    torch::Tensor& q,                    // [N, H, 512] bf16, in place
    torch::Tensor const& kv,             // [N, 512] bf16 (read-only)
    torch::Tensor& k_cache,              // [num_blocks, block_bytes] uint8
    torch::Tensor const& slot_mapping,   // [N] int64
    torch::Tensor const& position_ids,   // [N] int64
    torch::Tensor const& cos_sin_cache,  // [max_pos, rope_dim] bf16
    double eps, int64_t cache_block_size) {
  TORCH_CHECK(q.is_cuda() && q.is_contiguous(), "q must be contiguous CUDA");
  TORCH_CHECK(kv.is_cuda() && kv.is_contiguous(), "kv must be contiguous CUDA");
  TORCH_CHECK(k_cache.is_cuda(), "k_cache must be CUDA");
  TORCH_CHECK(slot_mapping.is_cuda() && slot_mapping.dtype() == torch::kInt64,
              "slot_mapping must be int64 CUDA");
  TORCH_CHECK(position_ids.is_cuda() && position_ids.dtype() == torch::kInt64,
              "position_ids must be int64 CUDA");
  TORCH_CHECK(cos_sin_cache.is_cuda(), "cos_sin_cache must be CUDA");
  TORCH_CHECK(q.dim() == 3 && q.size(2) == 512, "q shape [N, H, 512]");
  TORCH_CHECK(kv.dim() == 2 && kv.size(1) == 512, "kv shape [N, 512]");
  TORCH_CHECK(q.dtype() == kv.dtype(), "q and kv dtype must match");
  TORCH_CHECK(k_cache.dtype() == torch::kUInt8, "k_cache must be uint8");
  TORCH_CHECK(cos_sin_cache.dim() == 2 && cos_sin_cache.size(1) == 64,
              "cos_sin_cache shape [max_pos, 64]");
  TORCH_CHECK(cos_sin_cache.dtype() == torch::kFloat32,
              "cos_sin_cache must be float32");

  // With DP padding, slot_mapping can be shorter than q/kv/positions.
  // Q-norm+RoPE runs on all q.size(0) rows (downstream attention uses them);
  // KV quant+insert runs only on the first slot_mapping.size(0) rows.
  int const num_tokens_full = static_cast<int>(q.size(0));
  int const num_tokens_insert = static_cast<int>(slot_mapping.size(0));
  TORCH_CHECK(static_cast<int>(kv.size(0)) == num_tokens_full &&
                  static_cast<int>(position_ids.size(0)) == num_tokens_full,
              "q/kv/position_ids row counts must match");
  TORCH_CHECK(num_tokens_insert <= num_tokens_full,
              "slot_mapping must not exceed q row count");
  int const num_heads_q = static_cast<int>(q.size(1));
  int const cache_block_size_i = static_cast<int>(cache_block_size);
  int const kv_block_stride = static_cast<int>(k_cache.stride(0));

  at::cuda::OptionalCUDAGuard device_guard(device_of(q));
  auto stream = at::cuda::getCurrentCUDAStream();

  VLLM_DISPATCH_HALF_TYPES(
      q.scalar_type(), "fused_deepseek_v4_qnorm_rope_kv_insert", [&] {
        using qkv_scalar_t = scalar_t;
        vllm::deepseek_v4_fused_ops::
            launchFusedDeepseekV4QNormRopeKVRopeQuantInsert<qkv_scalar_t>(
                reinterpret_cast<qkv_scalar_t*>(q.data_ptr()),
                reinterpret_cast<qkv_scalar_t const*>(kv.data_ptr()),
                reinterpret_cast<uint8_t*>(k_cache.data_ptr()),
                reinterpret_cast<int64_t const*>(slot_mapping.data_ptr()),
                reinterpret_cast<int64_t const*>(position_ids.data_ptr()),
                cos_sin_cache.data_ptr<float>(), static_cast<float>(eps),
                num_tokens_full, num_tokens_insert, num_heads_q,
                cache_block_size_i, kv_block_stride, stream);
      });
}
```
**EN:** The exported wrapper validates tensor shapes, dtypes, and CUDA placement, handles the case where slot_mapping is shorter than q/kv due to DP padding, and dispatches only half/bfloat16 inputs through VLLM_DISPATCH_HALF_TYPES.
**CN:** 导出的包装函数会校验张量形状、dtype 与 CUDA 放置位置，处理由于 DP padding 导致 slot_mapping 短于 q/kv 的情况，并且只通过 VLLM_DISPATCH_HALF_TYPES 分发 half/bfloat16 输入。

## Key Concepts / 关键概念

- **EN:** The kernel horizontally fuses Q work and KV cache work by letting different head slots in the same grid execute different branches.
  **CN:** 该内核通过让同一 grid 中不同 head slot 执行不同分支，把 Q 计算与 KV cache 写入做成横向融合。
- **EN:** NoPE dimensions are quantized to FP8 while RoPE dimensions stay in bf16, matching the target cache layout.
  **CN:** NoPE 维度被量化成 FP8，而 RoPE 维度保持 bf16，以匹配目标 cache 布局。
- **EN:** The implementation is tightly specialized for DeepSeek-V4 attention geometry rather than being a generic RoPE kernel.
  **CN:** 该实现紧密绑定于 DeepSeek-V4 的注意力几何参数，而不是通用 RoPE 内核。

## Dependencies / 依赖关系

- **EN:** Uses dispatch_utils.h for dtype dispatch and type_convert.cuh for packed bf16/fp32 conversion.
  **CN:** 依赖 dispatch_utils.h 做 dtype 分发，并使用 type_convert.cuh 完成打包 bf16/fp32 转换。
- **EN:** Uses ATen CUDA context helpers and current-device property queries.
  **CN:** 依赖 ATen CUDA 上下文辅助函数与当前设备属性查询。
- **EN:** Depends on CUDA FP8 intrinsics on NVIDIA and HIP FP8 helpers on ROCm.
  **CN:** 在 NVIDIA 上依赖 CUDA FP8 intrinsic，在 ROCm 上依赖 HIP FP8 辅助函数。
