# sm100_fmha_mla_reduction.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/attention/mla/cutlass_sm100_mla/kernel/sm100_fmha_mla_reduction.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Final reduction kernel for split-KV MLA that merges per-split outputs and log-sum-exp statistics into the final attention result. / split-KV MLA 的最终归约内核，将每个 split 的输出与 log-sum-exp 统计量合并成最终注意力结果。

## Line-by-Line Analysis / 逐行分析
### Kernel contract
```cpp
template<
    class ElementOut,
    class ElementAcc,
    class ElementScale,
    size_t kNumHeads,
    size_t kHeadDimLatent,
    int kMaxSplits
>
struct Sm100FmhaMlaReductionKernel {
  static const int SharedStorageSize = 0;
  static const int MaxThreadsPerBlock = 128;
  ...
  struct Arguments {
    ElementAcc* ptr_oaccum = nullptr;
    ElementOut* ptr_o = nullptr;
    ElementAcc* ptr_lseaccum = nullptr;
    ElementAcc* ptr_lse = nullptr;
    ElementScale scale = 1.f;
    int num_batches = 0;
    int split_kv = -1;
```
**EN:** The reduction kernel is intentionally small: one CTA handles one `(head, batch)` pair, reads split partials from accumulation buffers, and writes the final output and optional LSE.
**CN:** 该归约内核设计得非常紧凑：一个 CTA 负责一个 `(head, batch)` 对，从累加缓冲区读取各个 split 的部分结果，并写回最终输出以及可选的 LSE。

### Split-aware LSE merge
```cpp
Tensor gLSEaccum = make_tensor(make_gmem_ptr(params.ptr_lseaccum + offset_lseaccum),
                               make_shape(params.split_kv), Stride<Int<kNumHeads>>{});
...
for (int i = 0; i < kNLsePerThread; ++i) {
  const int split = i * 32 + threadIdx.x;
  local_lse[i] = split < local_split_kv ? gLSEaccum(split) : -std::numeric_limits<ElementAcc>::infinity();
}
...
for (int offset = 16; offset >= 1; offset /= 2) {
  lse_max = max(lse_max, __shfl_xor_sync(0xffffffff, lse_max, offset));
}
...
ElementAcc global_lse = (sum_lse == 0.f || sum_lse != sum_lse)
    ? std::numeric_limits<ElementAcc>::infinity()
    : logf(sum_lse) + lse_max;
```
**EN:** The kernel performs a numerically stable log-sum-exp merge across all active splits. It first finds the maximum split LSE, then sums `exp(local_lse - lse_max)`, and finally reconstructs the merged LSE.
**CN:** 内核采用数值稳定的 log-sum-exp 方式合并所有有效 split 的 LSE。它先求出 split LSE 的最大值，再累加 `exp(local_lse - lse_max)`，最后恢复合并后的全局 LSE。

### Reweighting and output accumulation
```cpp
for (int i = 0; i < kNLsePerThread; ++i) {
  const int split = i * 32 + threadIdx.x;
  if (split < local_split_kv) {
    sLseScale[split] = expf(local_lse[i] - global_lse);
  }
}
...
for (int split = 0; split < local_split_kv; ++split) {
  ElementAcc lse_scale = sLseScale[split];
  CUTLASS_PRAGMA_UNROLL
  for(int i = 0; i < Elements; ++i) {
    local_val[i] += lse_scale * gOaccum(threadIdx.x + MaxThreadsPerBlock * i);
  }
  gOaccum.data() = gOaccum.data() + kHeadDimLatent;
}
```
**EN:** Once the merged LSE is known, each split's partial output is rescaled by its normalized softmax weight and accumulated into the final latent vector. The shared-memory `sLseScale` array avoids recomputing exponentials during the output merge.
**CN:** 在得到合并后的 LSE 之后，每个 split 的部分输出都会按归一化 softmax 权重重新缩放，并累加成最终的 latent 向量。共享内存中的 `sLseScale` 数组避免了在输出合并阶段重复计算指数。

## Key Concepts / 关键概念
- **Log-sum-exp merge / Log-sum-exp 合并**: Stable reduction of softmax statistics across KV splits.
- **Per-head CTA mapping / 按头映射 CTA**: Grid shape is `(num_heads, 1, num_batches)`.
- **Partial-output fusion / 部分输出融合**: Weighted sum over `ptr_oaccum` produces final attention output.

## Dependencies / 依赖关系
- Used by `device/sm100_mla.hpp` when `split_kv > 1`.
- Relies on `cute::Tensor` helpers for compact global-memory views.
- Uses CUDA warp shuffle intrinsics for fast intra-warp reductions.
