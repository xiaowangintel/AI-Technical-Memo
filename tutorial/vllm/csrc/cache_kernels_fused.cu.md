# cache_kernels_fused.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cache_kernels_fused.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Fuses RoPE application for MLA query/key tensors with cache writing, reducing intermediate memory traffic for DeepSeek-style attention. / [CN] 把 MLA 查询/键张量的 RoPE 计算与 cache 写入融合在一起，减少 DeepSeek 风格注意力中的中间访存。

## Line-by-Line Analysis / 逐行分析
### Fused kernel interface / 融合内核接口
```cpp
template <typename qk_t, typename cos_sin_t, bool IS_NEOX,
          typename raw_kv_scalar_t, typename cache_t, Fp8KVCacheDataType kv_dt>
__global__ void concat_and_cache_mla_rope_fused_kernel(
    const int64_t* __restrict__ positions,  // [num_tokens]
    qk_t* __restrict__ q_pe,        // [num_tokens, num_q_heads, rot_dim]
    qk_t* __restrict__ k_pe,        // [num_tokens, rot_dim]
    const qk_t* __restrict__ kv_c,  // [num_tokens, kv_lora_rank]
    const cos_sin_t* __restrict__ rope_cos_sin_cache,  // [max_position, 2,
                                                       // rot_dim // 2]
    const int rot_dim, const int64_t q_pe_stride_token,
    const int64_t q_pe_stride_head, const int64_t k_pe_stride,
    const int64_t kv_c_stride, const int num_q_heads,
    cache_t* __restrict__ kv_cache,  // [num_blocks, block_size, (kv_lora_rank +
                                     // rot_dim)]
    const int64_t* __restrict__ slot_mapping,  // [num_tokens]
    const int block_stride, const int entry_stride, const int kv_lora_rank,
    const int block_size, const float* kv_cache_quant_scale) {
```
**EN:** The template signature captures query/key scalar types, RoPE cache scalar type, NeoX-vs-GPT-J rotary layout, raw KV storage type, cache storage type, and KV-cache numeric format. That lets one kernel cover plain, FP8, and different rotary conventions.
**CN:** 这个模板签名同时编码了 query/key 标量类型、RoPE cache 标量类型、NeoX 与 GPT-J 两种旋转布局、原始 KV 存储类型、cache 存储类型以及 KV-cache 数值格式，因此一套内核就能覆盖普通、FP8 和不同 rotary 约定。

### In-place Q RoPE rotation / 原地 Q RoPE 旋转
```cpp
  const int embed_dim = rot_dim / 2;

  // Q ROPE
  const int nq = num_q_heads * embed_dim;
  for (int i = threadIdx.x; i < nq; i += blockDim.x) {
    int head_idx = i / embed_dim;
    int pair_idx = i % embed_dim;

    // NOTE: Would be nice to have interleaved sin/cos so we could just load
    // both at the same time.
    qk_t cos = static_cast<qk_t>(VLLM_LDG(cos_sin_ptr + pair_idx));
    qk_t sin = static_cast<qk_t>(VLLM_LDG(cos_sin_ptr + pair_idx + embed_dim));

    qk_t* q_pe_head_ptr =
        q_pe + token_idx * q_pe_stride_token + head_idx * q_pe_stride_head;

    int pair_idx_x, pair_idx_y;
    if constexpr (IS_NEOX) {
      // GPT-NeoX style rotary embedding.
      pair_idx_x = pair_idx;
      pair_idx_y = embed_dim + pair_idx;
    } else {
      // GPT-J style rotary embedding.
      pair_idx_x = pair_idx * 2;
      pair_idx_y = pair_idx * 2 + 1;
    }

    qk_t x_src = q_pe_head_ptr[pair_idx_x];
    qk_t y_src = q_pe_head_ptr[pair_idx_y];

    qk_t x_dst = x_src * cos - y_src * sin;
    qk_t y_dst = y_src * cos + x_src * sin;

    q_pe_head_ptr[pair_idx_x] = x_dst;
    q_pe_head_ptr[pair_idx_y] = y_dst;
  }
```
**EN:** Each block owns one token. The first loop walks all query heads and rotates pairs of coordinates in place using the cosine/sine cache, with index mapping switched by `IS_NEOX`.
**CN:** 每个 block 负责一个 token。第一段循环遍历所有 query head，利用 cosine/sine cache 对坐标对进行原地旋转，具体索引映射由 `IS_NEOX` 决定。

### K rotation and cache store / K 旋转与 cache 写入
```cpp
  // K with 1 HEAD
  for (int i = threadIdx.x; i < embed_dim; i += blockDim.x) {
    int pair_idx = i;

    qk_t cos = static_cast<qk_t>(VLLM_LDG(cos_sin_ptr + pair_idx));
    qk_t sin = static_cast<qk_t>(VLLM_LDG(cos_sin_ptr + pair_idx + embed_dim));

    qk_t* k_pe_head_ptr = k_pe + token_idx * k_pe_stride;

    int pair_idx_x, pair_idx_y;
    if constexpr (IS_NEOX) {
      // GPT-NeoX style rotary embedding.
      pair_idx_x = pair_idx;
      pair_idx_y = embed_dim + pair_idx;
    } else {
      // GPT-J style rotary embedding.
      pair_idx_x = pair_idx * 2;
      pair_idx_y = pair_idx * 2 + 1;
    }

    qk_t x_src = k_pe_head_ptr[pair_idx_x];
    qk_t y_src = k_pe_head_ptr[pair_idx_y];

    qk_t x_dst = x_src * cos - y_src * sin;
    qk_t y_dst = y_src * cos + x_src * sin;

    k_pe_head_ptr[pair_idx_x] = x_dst;
    k_pe_head_ptr[pair_idx_y] = y_dst;

    // NOTE Why is this monster necessary?
    // When K is of type float16, the actual template replacement for
    // raw_kv_scalar_t with be u16. That's why it's used at the last moment
    // otherwise CUDA ALU would break.
    const raw_kv_scalar_t raw_x_value =
        *reinterpret_cast<const raw_kv_scalar_t*>(&x_dst);
    const raw_kv_scalar_t raw_y_value =
        *reinterpret_cast<const raw_kv_scalar_t*>(&y_dst);

    cache_t* kv_cache_ptr = kv_cache + block_idx * block_stride +
                            entry_idx * entry_stride + kv_lora_rank;

    // MLA Cache Store
    if constexpr (kv_dt == Fp8KVCacheDataType::kAuto) {
      kv_cache_ptr[pair_idx_x] = raw_x_value;
      kv_cache_ptr[pair_idx_y] = raw_y_value;
    } else {
      kv_cache_ptr[pair_idx_x] =
          fp8::scaled_convert<cache_t, raw_kv_scalar_t, kv_dt>(
              raw_x_value, *kv_cache_quant_scale);
      kv_cache_ptr[pair_idx_y] =
          fp8::scaled_convert<cache_t, raw_kv_scalar_t, kv_dt>(
              raw_y_value, *kv_cache_quant_scale);
    }
  }

  // NOPE
  for (int i = threadIdx.x; i < kv_lora_rank; i += blockDim.x) {
    const qk_t* src_ptr = kv_c + token_idx * kv_c_stride + i;
    const raw_kv_scalar_t src_value =
        *reinterpret_cast<const raw_kv_scalar_t*>(src_ptr);

    cache_t* kv_cache_ptr =
        kv_cache + block_idx * block_stride + entry_idx * entry_stride;

    if constexpr (kv_dt == Fp8KVCacheDataType::kAuto) {
      kv_cache_ptr[i] = src_value;
    } else {
      kv_cache_ptr[i] = fp8::scaled_convert<cache_t, raw_kv_scalar_t, kv_dt>(
          src_value, *kv_cache_quant_scale);
    }
  }
```
**EN:** The second half rotates `k_pe`, then immediately writes rotated K and raw `kv_c` into the destination cache slot. `raw_kv_scalar_t` is important because FP16/BF16 values are sometimes moved as their underlying 16-bit storage type before optional FP8 conversion.
**CN:** 后半段先旋转 `k_pe`，随后立刻把旋转后的 K 和原始 `kv_c` 写入目标 cache 槽位。`raw_kv_scalar_t` 很关键，因为 FP16/BF16 有时需要先按其底层 16 位存储类型搬运，再选择性做 FP8 转换。

### Dispatch macro / 分发宏
```cpp
#define CALL_CONCAT_AND_CACHE_MLA_ROPE_FUSED(RAW_KV_T, CACHE_T, KV_DTYPE)     \
  do {                                                                        \
    VLLM_DISPATCH_FLOATING_TYPES(q_pe.scalar_type(), "qk_scalar_type", [&] {  \
      using qk_t = scalar_t;                                                  \
      VLLM_DISPATCH_FLOATING_TYPES(                                           \
          rope_cos_sin_cache.scalar_type(), "rope_cos_sin_cache_scalar_type", \
          [&] {                                                               \
            using cos_sin_t = scalar_t;                                       \
            if (rope_is_neox) {                                               \
              vllm::concat_and_cache_mla_rope_fused_kernel<                   \
                  qk_t, cos_sin_t, true, RAW_KV_T, CACHE_T, KV_DTYPE>         \
                  <<<grid, block, 0, stream>>>(                               \
                      positions.data_ptr<int64_t>(), q_pe.data_ptr<qk_t>(),   \
                      k_pe.data_ptr<qk_t>(), kv_c.data_ptr<qk_t>(),           \
                      rope_cos_sin_cache.data_ptr<cos_sin_t>(), rot_dim,      \
                      q_pe_stride_token, q_pe_stride_head, k_pe_stride,       \
                      kv_c_stride, num_q_heads,                               \
                      reinterpret_cast<CACHE_T*>(kv_cache.data_ptr()),        \
                      slot_mapping.data_ptr<int64_t>(), block_stride,         \
                      entry_stride, kv_lora_rank, block_size,                 \
                      kv_cache_quant_scale.data_ptr<float>());                \
            } else {                                                          \
              vllm::concat_and_cache_mla_rope_fused_kernel<                   \
                  qk_t, cos_sin_t, false, RAW_KV_T, CACHE_T, KV_DTYPE>        \
                  <<<grid, block, 0, stream>>>(                               \
                      positions.data_ptr<int64_t>(), q_pe.data_ptr<qk_t>(),   \
                      k_pe.data_ptr<qk_t>(), kv_c.data_ptr<qk_t>(),           \
                      rope_cos_sin_cache.data_ptr<cos_sin_t>(), rot_dim,      \
                      q_pe_stride_token, q_pe_stride_head, k_pe_stride,       \
                      kv_c_stride, num_q_heads,                               \
                      reinterpret_cast<CACHE_T*>(kv_cache.data_ptr()),        \
                      slot_mapping.data_ptr<int64_t>(), block_stride,         \
                      entry_stride, kv_lora_rank, block_size,                 \
                      kv_cache_quant_scale.data_ptr<float>());                \
            }                                                                 \
          });                                                                 \
    });                                                                       \
  } while (false)
```
**EN:** This macro nests two floating-type dispatches and one rotary-layout branch around the fused kernel launch. It avoids duplicating the complicated launch expression in the public wrapper.
**CN:** 这个宏在融合内核 launch 外层嵌套了两层浮点类型分发和一层 rotary 布局分支，避免在公开包装函数中重复书写复杂的启动表达式。

### Public wrapper and validation / 公开包装函数与校验
```cpp
void concat_and_cache_mla_rope_fused(
    torch::Tensor& positions,           // [num_tokens]
    torch::Tensor& q_pe,                // [num_tokens, num_q_heads, rot_dim]
    torch::Tensor& k_pe,                // [num_tokens, rot_dim]
    torch::Tensor& kv_c,                // [num_tokens, kv_lora_rank]
    torch::Tensor& rope_cos_sin_cache,  // [max_position, rot_dim]
    bool rope_is_neox,
    torch::Tensor& slot_mapping,  // [num_tokens] or [num_actual_tokens]
    torch::Tensor&
        kv_cache,  // [num_blocks, block_size, (kv_lora_rank + rot_dim)]
    const std::string& kv_cache_dtype, torch::Tensor& kv_cache_quant_scale) {
  // NOTE(woosuk): In vLLM V1, query/key/position.size(0) can be different from
  // slot_mapping.size(0) because of padding for CUDA graphs.
  // In vLLM V0, key.size(0) is always equal to slot_mapping.size(0) because
  // both include padding.
  // In vLLM V1, however, key.size(0) can be larger than slot_mapping.size(0)
  // since key includes padding for CUDA graphs, while slot_mapping does not.
  // In this case, slot_mapping.size(0) represents the actual number of tokens
  // before padding.
  // For compatibility with both cases, we use slot_mapping.size(0) as the
  // number of tokens.
  int num_tokens = slot_mapping.size(0);
  int num_padded_tokens = q_pe.size(0);
  TORCH_CHECK_GE(num_padded_tokens, num_tokens);

  const int num_q_heads = q_pe.size(1);
  const int rot_dim = q_pe.size(2);
  const int kv_lora_rank = kv_c.size(1);

  TORCH_CHECK_EQ(positions.size(0), num_padded_tokens);
  TORCH_CHECK_EQ(positions.dim(), 1);
  TORCH_CHECK_EQ(positions.scalar_type(), c10::ScalarType::Long);

  TORCH_CHECK_EQ(q_pe.dim(), 3);
  TORCH_CHECK_EQ(q_pe.size(0), num_padded_tokens);
  TORCH_CHECK_EQ(q_pe.size(1), num_q_heads);
  TORCH_CHECK_EQ(q_pe.size(2), rot_dim);

  TORCH_CHECK_EQ(k_pe.dim(), 2);
  TORCH_CHECK_EQ(k_pe.size(0), num_padded_tokens);
  TORCH_CHECK_EQ(k_pe.size(1), rot_dim);
  TORCH_CHECK_EQ(k_pe.scalar_type(), q_pe.scalar_type());

  TORCH_CHECK_EQ(kv_c.dim(), 2);
  TORCH_CHECK_EQ(kv_c.size(0), num_padded_tokens);
  TORCH_CHECK_EQ(kv_c.size(1), kv_lora_rank);
  TORCH_CHECK_EQ(kv_c.scalar_type(), q_pe.scalar_type());
  TORCH_CHECK_EQ(kv_c.dtype(), q_pe.dtype());

  TORCH_CHECK_EQ(rope_cos_sin_cache.size(1), rot_dim);

  TORCH_CHECK_EQ(slot_mapping.size(0), num_tokens);
  TORCH_CHECK_EQ(slot_mapping.scalar_type(), c10::ScalarType::Long);

  TORCH_CHECK_EQ(kv_cache.size(2), kv_lora_rank + rot_dim);
  TORCH_CHECK_EQ(kv_cache.dim(), 3);

  TORCH_CHECK_EQ(kv_cache_quant_scale.numel(), 1);
  TORCH_CHECK_EQ(kv_cache_quant_scale.scalar_type(), c10::ScalarType::Float);

  int64_t q_pe_stride_token = q_pe.stride(0);
  int64_t q_pe_stride_head = q_pe.stride(1);

  int64_t k_pe_stride = k_pe.stride(0);
  int64_t kv_c_stride = kv_c.stride(0);

  int block_size = kv_cache.size(1);

  int block_stride = kv_cache.stride(0);
  int entry_stride = kv_cache.stride(1);

  int rope_block_size = std::min(num_q_heads * rot_dim / 2, 512);
  int mla_block_size = kv_lora_rank;
  int thread_block_size =
      std::min(std::max(rope_block_size, mla_block_size), 512);

  dim3 grid(num_tokens, 1, 1);
  dim3 block(thread_block_size, 1, 1);

  const at::cuda::OptionalCUDAGuard device_guard(device_of(positions));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  DISPATCH_BY_KV_CACHE_DTYPE(kv_c.dtype(), kv_cache_dtype,
                             CALL_CONCAT_AND_CACHE_MLA_ROPE_FUSED);
}
```
**EN:** `concat_and_cache_mla_rope_fused` validates the padded-token convention used by CUDA graphs, checks tensor ranks and dtypes, derives strides, chooses a block size from the larger of RoPE work and MLA copy work, and then dispatches to the fused kernel.
**CN:** `concat_and_cache_mla_rope_fused` 会验证 CUDA graph 使用的 padded-token 约定，检查 tensor 维度和 dtype，推导 stride，根据 RoPE 工作量和 MLA 复制工作量中的较大者选择 block 大小，最后分发到融合内核。

## Key Concepts / 关键概念
- **EN:** RoPE and cache writing are fused to reduce extra reads/writes of `q_pe` and `k_pe`.
  **CN:** 通过把 RoPE 与 cache 写入融合，减少了 `q_pe`、`k_pe` 的额外读写。
- **EN:** The kernel mutates `q_pe` and `k_pe` in place, so callers must treat them as writable work buffers, not immutable inputs.
  **CN:** 该内核会原地修改 `q_pe` 和 `k_pe`，因此调用方必须把它们视为可写工作缓冲区，而不是只读输入。
- **EN:** Rotary layout is compile-time selected to avoid inner-loop branching overhead.
  **CN:** Rotary 布局在编译期选择，以避免内层循环产生额外分支开销。

## Dependencies / 依赖关系
- **EN:** Uses `dispatch_utils.h` for floating-type and KV-cache-format dispatch.
  **CN:** 使用 `dispatch_utils.h` 完成浮点类型与 KV-cache 格式分发。
- **EN:** Uses FP8 helpers from `quantization/w8a8/fp8/.../quant_utils.cuh` when the destination cache is quantized.
  **CN:** 当目标 cache 为量化格式时，会调用 `quantization/w8a8/fp8/.../quant_utils.cuh` 中的 FP8 辅助函数。
- **EN:** Depends on `cuda_compat.h` for `VLLM_LDG` so RoPE lookup code works across CUDA and ROCm.
  **CN:** 依赖 `cuda_compat.h` 中的 `VLLM_LDG`，从而让 RoPE 读取代码在 CUDA 与 ROCm 上都能工作。
