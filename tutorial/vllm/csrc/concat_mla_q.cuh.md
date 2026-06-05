# concat_mla_q.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/concat_mla_q.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines a warp-oriented kernel that concatenates MLA NoPE and RoPE query fragments into one contiguous query tensor. / [CN] 定义一个以 warp 为单位的内核，把 MLA 的 NoPE 与 RoPE query 片段拼接成连续的 query 张量。

## Line-by-Line Analysis / 逐行分析
### Kernel mapping and tensor interface / 内核映射与张量接口
```cpp
template <typename DType, int NOPE_DIM>
__global__ void ConcatMLAQKernel(
    DType* __restrict__ q_out, const DType* __restrict__ ql_nope,
    const DType* __restrict__ q_pe, const int num_tokens, const int num_heads,
    const int64_t out_stride_0, const int64_t out_stride_1,
    const int64_t nope_stride_0, const int64_t nope_stride_1,
    const int64_t pe_stride_0, const int64_t pe_stride_1) {
  const int flat_warp_id = (blockIdx.x * blockDim.x + threadIdx.x) >> 5;
  if (flat_warp_id >= num_tokens * num_heads) return;

  const int token_id = flat_warp_id / num_heads;
  const int head_id = flat_warp_id % num_heads;
  const int lane_id = threadIdx.x & 31;
```
**EN:** The kernel template is specialized by datatype and `NOPE_DIM`, and each warp is mapped to one `(token, head)` pair via `flat_warp_id`. This is a natural fit because the fixed 64-dim RoPE tail also maps cleanly onto 32 lanes.
**CN:** 该内核通过数据类型和 `NOPE_DIM` 模板参数特化，并用 `flat_warp_id` 把每个 warp 映射到一个 `(token, head)` 对。这种设计很自然，因为固定 64 维的 RoPE 尾部也能很好地映射到 32 个 lane。

### Vectorized NoPE copy / 向量化 NoPE 拷贝
```cpp
  constexpr bool use_256b = VLLM_256B_PTX_ENABLED;
  constexpr int nope_vec_loads =
      NOPE_DIM * sizeof(DType) / (VecTraits<use_256b>::ARCH_MAX_VEC_SIZE * 32);

  const DType* nope_src =
      ql_nope + token_id * nope_stride_0 + head_id * nope_stride_1;
  DType* nope_dst = q_out + token_id * out_stride_0 + head_id * out_stride_1;

#pragma unroll
  for (int i = 0; i < nope_vec_loads; i++) {
    const int offset = i * 32 + lane_id;
    if constexpr (use_256b) {
      st256_cs(reinterpret_cast<u32x8_t*>(nope_dst) + offset,
               ld256_cs(reinterpret_cast<const u32x8_t*>(nope_src) + offset));
    } else {
      st128_cs(reinterpret_cast<int4*>(nope_dst) + offset,
               ld128_cs(reinterpret_cast<const int4*>(nope_src) + offset));
    }
  }
```
**EN:** The NoPE portion is copied in cache-streaming vector transactions. The code picks 256-bit PTX when available, otherwise 128-bit `int4`, and each lane handles one vector slot within the current tile.
**CN:** NoPE 部分通过 cache-streaming 向量事务完成拷贝。代码在可用时选择 256 位 PTX，否则退回 128 位 `int4`；每个 lane 负责当前 tile 中的一个向量槽位。

### Fixed-width RoPE tail copy / 固定宽度 RoPE 尾部拷贝
```cpp
  const int* rope_src = reinterpret_cast<const int*>(
      q_pe + token_id * pe_stride_0 + head_id * pe_stride_1);
  int* rope_dst = reinterpret_cast<int*>(q_out + token_id * out_stride_0 +
                                         head_id * out_stride_1 + NOPE_DIM);

  st32_cs(rope_dst + lane_id, ld32_cs(rope_src + lane_id));
}
```
**EN:** The final 64 RoPE values are copied as 32-bit words, one word per lane, into the output region immediately following the NoPE payload. This avoids an extra kernel launch for a simple concatenation.
**CN:** 最后的 64 个 RoPE 值按 32 位字进行拷贝，每个 lane 负责一个字，并直接写入紧跟在 NoPE 负载之后的输出区域。这样可以避免为了简单拼接再启动一个额外内核。

## Key Concepts / 关键概念
- **EN:** The kernel is intentionally specialized for DeepSeek MLA assumptions: fixed 64-dim RoPE and currently instantiated `NOPE_DIM=512`.
  **CN:** 该内核明确针对 DeepSeek MLA 假设进行特化：RoPE 维度固定为 64，当前实例化的 `NOPE_DIM=512`。
- **EN:** Warp-level ownership keeps indexing simple and ensures coalesced copies for both source tensors.
  **CN:** 以 warp 为单位分配工作可以简化索引，并保证两个源张量都具备较好的合并访问。

## Dependencies / 依赖关系
- **EN:** `cuda_vec_utils.cuh` supplies `VecTraits`, `ld/st128_cs`, and `ld/st256_cs` for the fast copy path.
  **CN:** `cuda_vec_utils.cuh` 为快速拷贝路径提供了 `VecTraits`、`ld/st128_cs` 和 `ld/st256_cs`。
- **EN:** The public caller lives in `cache_kernels.cu` and validates shapes before launching this kernel.
  **CN:** 公开调用入口位于 `cache_kernels.cu` 中，并会在启动该内核前完成形状校验。
