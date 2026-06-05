# persistent_topk.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/persistent_topk.cuh`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Implements several CUDA top-k selection paths for ragged float logits, including short-sequence histogram selection, medium-sequence buffered radix refinement, multi-CTA persistent radix selection, and a FlashInfer-derived fallback kernel. / [CN] 为不规则长度的 float logits 实现多条 CUDA top-k 选择路径，包括短序列直方图选择、中等序列缓冲式 radix 细化、多 CTA persistent radix 选择，以及源自 FlashInfer 的回退内核。

## Line-by-Line Analysis / 逐行分析

### [Constants, key transforms, and state structs / 常量、键变换与状态结构]
```cpp
constexpr int kThreadsPerBlock = 1024;
constexpr int RADIX = 256;
constexpr uint32_t RADIX_THRESHOLD = 32768;
constexpr int kDecodeBins = 2048;
constexpr uint32_t HIST2048_THRESHOLD = 8192;

__device__ __forceinline__ auto convert_to_uint32_v2(float x) -> uint32_t {
  uint32_t bits = __float_as_uint(x);
  return (bits & 0x80000000u) ? ~bits : (bits | 0x80000000u);
}

struct PersistentTopKParams {
  const float* __restrict__ input;
  int32_t* __restrict__ output;
  int32_t* __restrict__ lengths;
  RadixRowState* row_states;
  uint32_t num_rows;
  uint32_t stride;
  uint32_t top_k;
  uint32_t chunk_size;
  uint32_t ctas_per_group;
  uint32_t max_seq_len;
};
```
**EN:** The file first defines the execution regime: 1024-thread blocks, 256-way radix buckets, and thresholds that decide whether a row is handled by the short decode path, the medium histogram path, or the large cooperative path. `convert_to_uint32_v2` maps IEEE float bits into an order-preserving unsigned key, so radix logic can compare floats with integer bucket operations. `PersistentTopKParams` packages the row-major logits tensor, ragged lengths, output buffer, and cooperative scheduling metadata into one kernel argument block.  
**CN:** 文件首先定义执行框架：1024 线程块、256 路 radix 桶，以及决定使用短序列 decode 路径、中等直方图路径或大序列协作路径的阈值。`convert_to_uint32_v2` 将 IEEE 浮点位模式映射为保持顺序的无符号键，使 radix 逻辑可以用整数桶操作比较浮点数。`PersistentTopKParams` 把行主序 logits、变长长度、输出缓冲区以及协作调度元数据打包成统一的内核参数。

### [Short-sequence decode path / 短序列 decode 路径]
```cpp
template <int TopK>
__device__ __noinline__ void histogram_2048_topk(
    const float* __restrict__ logits, int32_t* __restrict__ output_indices,
    int32_t seq_len) {
  extern __shared__ int decode_smem[];
  ...
  for (int i = tx; i < n_vec; i += kThreadsPerBlock) {
    ...
    const uint16_t b0 = static_cast<uint16_t>(decode_bin(v0));
    ...
    atomicAdd(&histo[b0], 1);
  }
  ...
  BlockScanT(scan_storage).ExclusiveSum(pair_sum, pair_prefix, total);
  if (pair_suffix >= TopK && (pair_suffix - h0) < TopK) {
    decode_smem[SBASE + sTHR] = 2 * tx;
  }
}
```
**EN:** `histogram_2048_topk` targets rows with `seq_len <= 8192`. It converts each logit to an 11-bit half-based bucket (`decode_bin`), uses vectorized `float4` loads when alignment allows, and accumulates a 2048-bin histogram in shared memory. A CUB block scan turns bucket counts into suffix counts, letting the kernel locate the threshold bucket without sorting the row. The function then emits indices strictly above the threshold immediately and buffers threshold-equal candidates for a rare refinement step.  
**CN:** `histogram_2048_topk` 面向 `seq_len <= 8192` 的行。它把每个 logit 转成基于 half 表示的 11 位桶（`decode_bin`），在对齐条件满足时使用向量化 `float4` 读取，并在共享内存中累计 2048 桶直方图。随后借助 CUB 的块级扫描把桶计数变成后缀计数，从而无需完整排序就能定位阈值桶。函数会立即写出严格大于阈值的索引，并把等于阈值的候选缓存起来，只有少见情况下才进入进一步细化。

### [Medium path: FP16 coarse histogram + FP32 refinement / 中路径：FP16 粗直方图 + FP32 细化]
```cpp
template <int TopK>
__device__ __noinline__ void histogram_256_topk(
    const float* __restrict__ logits, int* __restrict__ output_indices,
    int logits_offset, int seq_len) {
  extern __shared__ char medium_smem[];
  ...
  for (int idx = thread_id; idx < seq_len; idx += kThreadsPerBlock) {
    const auto bin = convert_to_uint8(logits[idx + logits_offset]);
    atomicAdd(&shared_histogram[0][bin], 1);
  }
  ...
  if (bin > threshold_bin) {
    const int output_pos = atomicAdd(&shared_output_count, 1);
    output_indices[output_pos] = idx;
  } else if (bin == threshold_bin) {
    ...
    const uint32_t fp32_bits = convert_to_uint32_v2(logit_value);
    const int next_bin = (fp32_bits >> 24) & 0xFF;
    atomicAdd(&shared_histogram[0][next_bin], 1);
  }
}
```
**EN:** For sequences between the decode and cooperative thresholds, the kernel first bins values by the high 8 bits of their FP16-ordered representation. That cheap coarse pass identifies the threshold region and directly outputs elements already known to be in the final top-k. Elements tied at the coarse threshold are buffered, then refined across up to four 8-bit passes over the ordered FP32 key. This design avoids global-memory coordination while keeping shared-memory usage bounded by `MAX_BUFFERED_ITEMS`.  
**CN:** 对于位于 decode 阈值和协作阈值之间的序列，内核先按 FP16 有序表示的高 8 位对数值分桶。这个廉价的粗筛过程先定位阈值区间，并直接输出已经确定属于最终 top-k 的元素。落在粗阈值桶内的元素会被缓存，再基于有序 FP32 键进行最多四轮 8 位细化。这样既避免了全局内存级协作，又把共享内存占用限制在 `MAX_BUFFERED_ITEMS` 范围内。

### [Inter-CTA synchronization and large-row radix selection / CTA 间同步与大序列 radix 选择]
```cpp
__device__ __forceinline__ int ld_acquire(int* ptr) { ... }
__device__ __forceinline__ void red_release(int* ptr, int val) { ... }
__device__ __forceinline__ void st_release(int* ptr, int val) { ... }

template <int TopK, uint32_t VEC_SIZE>
__device__ void radix_topk(..., RadixRowState* state, uint32_t cta_in_group,
                           uint32_t ctas_per_group, int& barrier_phase,
                           uint32_t iter, uint32_t tx) {
  ...
  for (uint32_t round = 0; round < 4; round++) {
    ...
    atomicAdd(&current_hist[i], local_histogram[i]);
    ...
    if (count_ge >= remaining_k && count_gt < remaining_k) {
      shared_scalars[2] = tx;
      shared_scalars[3] = remaining_k - count_gt;
    }
  }
}
```
**EN:** The large-row path splits one row across multiple CTAs. Each CTA loads its chunk into shared memory as ordered `uint32_t` keys, computes a local histogram for the current radix digit, then contributes into a group-wide `RadixRowState`. The custom `ld_acquire` / `red_release` / `st_release` primitives provide GPU memory ordering for a spin-wait barrier built on global counters, so every CTA observes the same histogram state before the next radix round. After four rounds, the kernel first writes all items strictly greater than the pivot, then fills the remaining slots with pivot-equal items until `TopK` is satisfied.  
**CN:** 大序列路径把同一行拆分给多个 CTA 协同处理。每个 CTA 先把自己的分块加载到共享内存中，并转换为有序的 `uint32_t` 键；随后针对当前 radix 位计算局部直方图，并把结果累加到组级 `RadixRowState`。自定义的 `ld_acquire` / `red_release` / `st_release` 原语为基于全局计数器的自旋屏障提供 GPU 内存顺序保障，从而确保所有 CTA 在进入下一轮 radix 前看到一致的直方图状态。四轮之后，内核先写出严格大于枢轴值的元素，再用等于枢轴的元素补满剩余的 `TopK` 槽位。

### [Persistent kernel dispatch / Persistent 内核分发]
```cpp
template <int TopK = 2048, uint32_t VEC_SIZE = 1>
__global__ void persistent_topk_kernel(PersistentTopKParams params) {
  ...
  if (seq_len <= RADIX_THRESHOLD) {
    if (cta_in_group == 0) {
      if (seq_len <= static_cast<uint32_t>(TopK)) {
        ...
      } else if (seq_len <= static_cast<uint32_t>(HIST2048_THRESHOLD)) {
        histogram_2048_topk<TopK>(row_input, row_output, seq_len);
      } else {
        histogram_256_topk<TopK>(row_input, row_output, 0, seq_len);
      }
    }
    continue;
  }
  radix_topk<TopK, VEC_SIZE>(...);
}
```
**EN:** `persistent_topk_kernel` is the central router. Rows are assigned to CTA groups in a round-robin fashion. If the row is small, only CTA 0 of the group runs and picks the trivial, decode, or medium histogram subroutine. If the row exceeds `RADIX_THRESHOLD`, every CTA in the group participates in the cooperative radix selector. This keeps the launch persistent and avoids separate kernels for different row classes inside the same batch.  
**CN:** `persistent_topk_kernel` 是整个文件的核心分流器。行按照轮转方式分配给 CTA 组；若行较小，则只有组内的 CTA 0 运行，并在 trivial、decode 或中等直方图子路径之间选择。若行长度超过 `RADIX_THRESHOLD`，则组内所有 CTA 都参与协作式 radix 选择。这样同一批次内无需为不同大小的行分别发射多个独立内核，保持了 persistent launch 的思路。

### [FlashInfer-derived fallback kernel / 基于 FlashInfer 的回退内核]
```cpp
template <typename DType, typename IdType, int VEC_SIZE, uint32_t MAX_K = 2048>
__global__ void FilteredTopKUnifiedKernel(const DType* __restrict__ input,
                                          IdType* __restrict__ output,
                                          const IdType* __restrict__ lengths,
                                          uint32_t num_rows, uint32_t top_k,
                                          uint32_t max_len) {
  ...
  run_cumsum();
  if (tx < RADIX && s_histogram[tx] > topk && s_histogram[tx + 1] <= topk) {
    s_threshold_bin_id = tx;
    s_num_input[0] = 0;
    s_counter = 0;
  }
  ...
}

template <typename DType>
constexpr int ComputeFilteredTopKVecSize(uint32_t max_len) { ... }
```
**EN:** The bottom half of the file embeds a separate FilteredTopK implementation adapted from FlashInfer. Instead of a persistent multi-CTA schedule, it launches one block per row and relies on 128 KB dynamic shared memory plus vectorized loads (`VEC_SIZE` chosen by `ComputeFilteredTopKVecSize`). It follows the same high-level idea—coarse histogram first, then radix refinement of threshold-equal items—but is tuned for larger batch sizes where a simple per-row launch can outperform the persistent scheduler.  
**CN:** 文件后半部分内嵌了一个改编自 FlashInfer 的独立 FilteredTopK 实现。它不是 persistent 的多 CTA 调度，而是按“每行一个 block”发射，并依赖 128 KB 动态共享内存以及由 `ComputeFilteredTopKVecSize` 选择的向量化加载。整体思想仍然是“先粗直方图，再对阈值桶内元素做 radix 细化”，但它更适合 batch 较大、直接按行发射能优于 persistent 调度器的场景。

## Key Concepts / 关键概念
- **EN:** Ordered-float transforms let integer radix logic approximate and then exactly refine top-k without a full sort.  
  **CN:** 有序浮点键变换让整数 radix 逻辑能够先近似筛选，再精确细化 top-k，而不必做完整排序。
- **EN:** The implementation is explicitly multi-path: trivial, 2048-bin decode, 256-bin medium path, multi-CTA radix path, and FilteredTopK fallback.  
  **CN:** 该实现明确采用多路径策略：trivial、2048 桶 decode、256 桶中路径、多 CTA radix 路径以及 FilteredTopK 回退。
- **EN:** Cooperative large-row processing depends on persistent launch sizing and correct global-memory synchronization semantics.  
  **CN:** 大序列协作处理依赖 persistent launch 的容量规划，以及正确的全局内存同步语义。

## Dependencies / 依赖关系
- **EN:** Uses CUDA runtime/device intrinsics, half conversion helpers, and CUB block scans.  
  **CN:** 依赖 CUDA runtime / device intrinsic、half 转换辅助函数以及 CUB 块级扫描。
- **EN:** `topk.cu` provides the host-side policy that chooses this persistent kernel or the embedded FilteredTopK launcher.  
  **CN:** `topk.cu` 提供主机侧策略，用来选择本文件的 persistent 内核或底部嵌入的 FilteredTopK 启动逻辑。
- **EN:** The large cooperative path uses `RadixRowState` in a caller-provided workspace tensor that must be zero-initialized before launch.  
  **CN:** 大序列协作路径使用调用方提供的 workspace tensor 中的 `RadixRowState`，该区域在启动前必须清零。
