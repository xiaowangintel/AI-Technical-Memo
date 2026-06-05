# sampler.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/sampler.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Implements CUDA sampling helpers for repetition penalties and efficient per-row top-k selection in both decode and prefill scenarios. / [CN] 实现 CUDA 采样辅助逻辑，包括重复惩罚和面向 decode/prefill 场景的高效逐行 top-k 选择。

## Line-by-Line Analysis / 逐行分析

### [Repetition penalty kernel / 重复惩罚内核]
```cpp
template <typename scalar_t>
__global__ void apply_repetition_penalties_kernel(
    scalar_t* __restrict__ logits,
    const bool* __restrict__ prompt_mask,
    const bool* __restrict__ output_mask,
    const scalar_t* __restrict__ repetition_penalties,
    const int num_seqs, const int vocab_size, const int tile_size) {
  const int seq_idx = blockIdx.x;
  const int tile_start = blockIdx.y * tile_size;
  ...
  const bool is_repeated = prompt_mask[idx] || output_mask[idx];
  if (is_repeated) {
    scalar_t logit = logits[idx];
    logits[idx] = (logit > 0) ? logit / penalty : logit * penalty;
  }
}
```
**EN:** Each block covers one sequence and one vocabulary tile. The kernel checks whether a token has appeared either in the prompt or generated output, then applies the standard repetition penalty rule in-place: positive logits are divided by the penalty, negative logits are multiplied by it. The 2D grid shape lets the implementation spread a single large vocabulary row across multiple blocks.  
**CN:** 每个 block 处理一个序列和一个词表 tile。内核先判断某个 token 是否出现在 prompt 或已生成输出中，然后原地应用标准重复惩罚规则：正 logits 除以惩罚系数，负 logits 乘以惩罚系数。二维网格设计使得超大词表的一行能够被多个 block 并行处理。

### [Radix-friendly helpers and vectorized reads / 适配 radix 的辅助函数与向量化读取]
```cpp
template <int step>
static inline __device__ uint32_t extractBinIdx(float x) {
  if constexpr (step == 0) {
    __half hx = __float2half(x);
    uint16_t bits = __half_as_ushort(hx);
    ...
    return bits >> 5;
  } else if constexpr (step == 1) {
    return bits >> 21;
  } else if constexpr (step == 2) {
    return (bits >> 10) & 0x7ff;
  } else {
    return bits & 0x3ff;
  }
}

template <typename T, typename idxT, typename Func>
__device__ void vectorized_process(size_t thread_rank, size_t num_threads,
                                   const T* in, idxT len, Func f) {
  using WideT = float4;
  ...
}
```
**EN:** The top-k path avoids sorting the full row. Instead, it repeatedly extracts coarse-to-fine radix bins from a transformed float representation: a half-precision coarse stage followed by higher-resolution FP32 stages. `vectorized_process` tries to read input in `float4`-sized chunks when alignment permits, then falls back to scalar handling for misaligned prefixes and tails. This reduces memory overhead during histogram construction.  
**CN:** top-k 路径并不对整行做完整排序，而是反复从变换后的浮点表示中提取由粗到细的 radix 桶：先用半精度做粗筛，再进入更高分辨率的 FP32 阶段。`vectorized_process` 会在对齐允许时按 `float4` 宽度读取输入，对前缀错位和尾部元素则自动回退到标量处理，从而降低直方图构建阶段的访存开销。

### [One histogram refinement step / 单轮直方图细化步骤]
```cpp
template <int step, int kNumThreadsPerBlock, int kNumBins, int kNumFinalItems,
          bool multipleBlocksPerRow, bool mergeBlocks, typename SmemFinalType,
          typename SmemOutputType>
__device__ bool processHistogramStep(...) {
  ...
  auto distributeToBins = [&](float logit, int /* idx */ = 0) {
    if (isPartialMatch<patternShift>(logit, logitPattern)) {
      uint32_t binIdx = extractBinIdx<step>(logit);
      atomicAdd(&smemFinal.histo.data[binIdx], 1);
    }
  };
  ...
  Scan(smemFinal.histo.scan).ExclusiveSum(binCount, prefixSum, totalSum);
  ...
  if (binIdx < thresholdBinIdx && shouldWriteDirectly) {
    int dstIdx = atomicAdd(&smemFoundTopKValues[0], 1);
    smemOutput[dstIdx] = ...;
  }
}
```
**EN:** `processHistogramStep` is the core primitive of the sampler’s top-k algorithm. It builds a histogram for the current radix slice, runs a block-wide prefix scan with CUB to find the threshold bin, and then separates three categories: elements already confirmed in top-k, elements tied at the threshold that must be carried to the next step, and everything else. The template flags let the same logic serve normal per-row selection, split-row partial selection, and block-merge mode.  
**CN:** `processHistogramStep` 是采样器 top-k 算法的核心原语。它先为当前 radix 切片构建直方图，再借助 CUB 做块级前缀扫描以定位阈值桶，最后把元素分成三类：已经确定属于 top-k 的元素、位于阈值桶中需要传给下一轮的元素，以及其余元素。模板参数使同一套逻辑能够同时服务普通逐行选择、分块部分选择和多块合并模式。

### [Per-row top-k pipeline / 逐行 top-k 流水线]
```cpp
template <int kNumThreadsPerBlock, int kNumBins, bool useRadixSort,
          bool multipleBlocksPerRow = false, bool mergeBlocks = false>
static __device__ void topKPerRowJob(...) {
  ...
  bool continueToNextStep = processHistogramStep<0, ...>(...);
  if (continueToNextStep) {
    continueToNextStep = processHistogramStep<1, ...>(...);
  }
  if (continueToNextStep) {
    continueToNextStep = processHistogramStep<2, ...>(...);
  }
  if (continueToNextStep) {
    processHistogramStep<3, ...>(...);
  }
  if (!continueToNextStep) {
    FinalSort(smemFinal.finalSort)
        .SortDescendingBlockedToStriped(finalLogits, finalIndices);
  }
}
```
**EN:** `topKPerRowJob` chains up to four histogram stages, following a coarse-to-fine bit schedule (`half -> 11 bits -> 11 bits -> 10 bits`). If the threshold bucket becomes small enough before the last step, the remaining candidates are sorted exactly inside shared memory—either with CUB radix sort or a fallback insertion sort. This hybrid strategy keeps the common path fast while preserving exact top-k semantics.  
**CN:** `topKPerRowJob` 最多串联四轮直方图阶段，沿着“粗到细”的位宽计划推进（`half -> 11 位 -> 11 位 -> 10 位`）。如果在最后一步之前阈值桶已经缩小到足够小，剩余候选会在共享内存中做精确排序——优先使用 CUB radix sort，必要时回退到插入排序。这种混合策略既让常见路径足够快，又保持精确的 top-k 语义。

### [Decode/prefill kernels and long-row split mode / Decode/Prefill 内核与长行拆分模式]
```cpp
template <int kNumThreadsPerBlock, bool useRadixSort,
          bool multipleBlocksPerRow = false, bool mergeBlocks = false>
static __global__ void topKPerRowDecode(..., int seqLensIs2D = 0,
                                        float* outLogits = nullptr,
                                        const int numBlocksToMerge = 0,
                                        const int* indices = nullptr) {
  int batch_idx = rowIdx / next_n;
  int next_n_idx = rowIdx % next_n;
  int seq_len = seqLensIs2D ? seqLens[rowIdx] : seqLens[batch_idx];
  int rowEnd = seqLensIs2D ? max(0, seq_len)
                           : max(0, seq_len - next_n + next_n_idx + 1);
  ...
}
```
**EN:** The decode kernel supports two sequence-length layouts: 1D batch lengths and pre-expanded 2D per-row lengths. For very long rows, the implementation can split one row across `gridDim.y` blocks, write intermediate `(index, logit)` pairs, and then launch a second merge kernel that runs the same histogram job over those partial results. Prefill uses a simpler row-start/row-end interface but shares the same selection primitive.  
**CN:** decode 内核支持两种长度布局：按 batch 给出的 1D 长度，以及预展开后的逐行 2D 长度。对超长行，实现在 `gridDim.y` 维度上把单行拆成多个 block，先写出中间 `(index, logit)` 对，再启动第二个合并内核，对这些部分结果再次运行同样的直方图选择逻辑。prefill 则使用更简单的 `rowStart/rowEnd` 接口，但底层复用了同一套选择原语。

### [Host-side policy / 主机侧策略]
```cpp
void top_k_per_row_decode(...) {
  constexpr int kSortingAlgorithmThreshold = 12288;
  constexpr int kSplitWorkThreshold = 200 * 1000;
  ...
  if (numColumns < kSortingAlgorithmThreshold) {
    topKPerRowDecode<kNumThreadsPerBlock, false><<<...>>>();
  } else if (numColumns < kSplitWorkThreshold) {
    topKPerRowDecode<kNumThreadsPerBlock, true><<<...>>>();
  } else {
    topKPerRowDecode<kNumThreadsPerBlock, true, true><<<...>>>();
    topKPerRowDecode<1024, true, false, true><<<...>>>();
  }
}
```
**EN:** The exported wrappers choose among multiple CUDA strategies at runtime. Decode switches between insertion-sort finishing, radix-sort finishing, and a two-pass split/merge path based on row width. Prefill chooses insertion sort for the first group of rows and radix-sort finishing for the remainder. `apply_repetition_penalties_` also computes a tile count from the device SM count so vocabulary work is better balanced.  
**CN:** 导出的封装函数会在运行时选择不同 CUDA 策略。decode 会根据行宽在“插入排序收尾”“radix-sort 收尾”和“两阶段拆分/合并”之间切换；prefill 则对前一部分行使用插入排序，对剩余行使用 radix-sort 收尾。`apply_repetition_penalties_` 还会根据设备的 SM 数量计算 tile 数，以便更均衡地分配词表处理工作。

## Key Concepts / 关键概念
- **EN:** The sampler’s top-k code is selection-oriented, not sort-oriented: it repeatedly narrows the threshold bucket before resorting to exact sorting.  
  **CN:** 采样器的 top-k 代码本质上是“选择”而不是“排序”：它先不断收缩阈值桶，只有在候选足够小时才做精确排序。
- **EN:** The same core device function supports normal rows, split rows, and merge rows through template flags instead of duplicated kernels.  
  **CN:** 同一个核心 device 函数通过模板标志同时支持普通行、拆分行和合并行，而不是复制多套内核。
- **EN:** CUB is used both for prefix scans and for the final shared-memory radix sort path.  
  **CN:** CUB 同时用于前缀扫描和最终共享内存中的 radix sort 收尾路径。

## Dependencies / 依赖关系
- **EN:** Depends on `cuda_compat.h`, `dispatch_utils.h`, and CUB/HIPCUB primitives.  
  **CN:** 依赖 `cuda_compat.h`、`dispatch_utils.h` 以及 CUB/HIPCUB 原语。
- **EN:** `torch_bindings.cpp` exposes `apply_repetition_penalties_`, `top_k_per_row_prefill`, and `top_k_per_row_decode` to PyTorch.  
  **CN:** `torch_bindings.cpp` 会把 `apply_repetition_penalties_`、`top_k_per_row_prefill` 和 `top_k_per_row_decode` 注册给 PyTorch。
- **EN:** Decode mode relies on external `seqLens` semantics: either one length per batch item or one effective length per `(batch, next_n)` row.  
  **CN:** decode 模式依赖外部 `seqLens` 语义：要么每个 batch item 一个长度，要么为每个 `(batch, next_n)` 行预先给出有效长度。
