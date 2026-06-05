# paged_attention_v2.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/attention/paged_attention_v2.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Launch the partitioned paged-attention v2 path, which computes per-partition attention statistics first and then reduces them into the final output tensor. / [CN] 启动分区式的 paged-attention v2 路径：先计算每个分区的注意力统计量，再将其归约成最终输出张量。

## Line-by-Line Analysis / 逐行分析
### Two-stage launch macro / 双阶段启动宏
```cpp
#define LAUNCH_PAGED_ATTENTION_V2(HEAD_SIZE)                                     vllm::paged_attention_v2_kernel<T, CACHE_T, HEAD_SIZE, BLOCK_SIZE,                                             NUM_THREADS, KV_DTYPE, IS_BLOCK_SPARSE,                                        PARTITION_SIZE>                                    <<<grid, block, shared_mem_size, stream>>>(                                        exp_sums_ptr, max_logits_ptr, tmp_out_ptr, query_ptr, key_cache_ptr,           value_cache_ptr, num_kv_heads, scale, block_tables_ptr,                        seq_lens_ptr, max_num_blocks_per_seq, alibi_slopes_ptr, q_stride,              kv_block_stride, kv_head_stride, k_scale_ptr, v_scale_ptr, tp_rank,            blocksparse_local_blocks, blocksparse_vert_stride,                             blocksparse_block_size, blocksparse_head_sliding_step);                vllm::paged_attention_v2_reduce_kernel<T, HEAD_SIZE, NUM_THREADS,                                                     PARTITION_SIZE>                             <<<reduce_grid, block, reduce_shared_mem_size, stream>>>(                          out_ptr, exp_sums_ptr, max_logits_ptr, tmp_out_ptr, seq_lens_ptr,              max_num_partitions);
```
**EN:** Unlike v1, v2 is explicitly split into two kernels. The first kernel computes per-partition outputs plus normalization metadata (`exp_sums`, `max_logits`, `tmp_out`), and the second kernel reduces those partition-local results back into the final attention output.
**CN:** 与 v1 不同，v2 明确拆成两个内核。第一个内核计算每个分区的输出及其归一化元数据（`exp_sums`、`max_logits`、`tmp_out`），第二个内核再把这些分区级结果归约成最终 attention 输出。

### Launcher setup and partition sizing / 启动器初始化与分区大小计算
```cpp
template <typename T, typename CACHE_T, int BLOCK_SIZE,
          vllm::Fp8KVCacheDataType KV_DTYPE, bool IS_BLOCK_SPARSE,
          int NUM_THREADS = 128, int PARTITION_SIZE = 512>
void paged_attention_v2_launcher(
    torch::Tensor& out, torch::Tensor& exp_sums, torch::Tensor& max_logits,
    torch::Tensor& tmp_out, torch::Tensor& query, torch::Tensor& key_cache,
    torch::Tensor& value_cache, int num_kv_heads, float scale,
    torch::Tensor& block_tables, torch::Tensor& seq_lens, int max_seq_len,
    const std::optional<torch::Tensor>& alibi_slopes, torch::Tensor& k_scale,
    torch::Tensor& v_scale, const int tp_rank,
    const int blocksparse_local_blocks, const int blocksparse_vert_stride,
    const int blocksparse_block_size, const int blocksparse_head_sliding_step) {
  ...
  const int NUM_WARPS = NUM_THREADS / WARP_SIZE;
  int max_num_partitions = DIVIDE_ROUND_UP(max_seq_len, PARTITION_SIZE);
  int logits_size = PARTITION_SIZE * sizeof(float);
  int outputs_size = (NUM_WARPS / 2) * head_size * sizeof(float);

  dim3 grid(num_heads, num_seqs, max_num_partitions);
  int shared_mem_size = std::max(logits_size, outputs_size);
  dim3 reduce_grid(num_heads, num_seqs);
  int reduce_shared_mem_size = 2 * max_num_partitions * sizeof(float);
```
**EN:** v2 introduces `PARTITION_SIZE` (default 512), so a long sequence is split into several partitions. The first grid adds a `z` dimension for partitions, while the reduction grid collapses back to `(num_heads, num_seqs)`. It also allocates distinct shared-memory budgets for the compute stage and the reduction stage.
**CN:** v2 引入了 `PARTITION_SIZE`（默认 512），因此长序列会被拆成多个分区。第一个 grid 在 `z` 维上表示分区，而归约 grid 又回到 `(num_heads, num_seqs)`。同时，它为计算阶段和归约阶段分别准备了不同的共享内存预算。

### Head-size specialization / head 大小特化分发
```cpp
dim3 block(NUM_THREADS);
const at::cuda::OptionalCUDAGuard device_guard(device_of(query));
const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
switch (head_size) {
  case 32:
    LAUNCH_PAGED_ATTENTION_V2(32);
    break;
  case 64:
    LAUNCH_PAGED_ATTENTION_V2(64);
    break;
  ...
  case 256:
    LAUNCH_PAGED_ATTENTION_V2(256);
    break;
  default:
    TORCH_CHECK(false, "Unsupported head size: ", head_size);
    break;
}
```
**EN:** The head-size switch mirrors v1, but here every case expands to two launches instead of one. The specialization still matters because both the partition kernel and the reduce kernel use `HEAD_SIZE` as a compile-time constant for vector layout and reduction structure.
**CN:** 这个 head-size 分支与 v1 类似，但这里每个 case 都会展开成两次 kernel 启动。特化依然很重要，因为分区内核和归约内核都把 `HEAD_SIZE` 当作编译期常量，用于决定向量布局和归约结构。

### Block-size and sparsity dispatch / block 大小与稀疏模式分发
```cpp
#define CALL_V2_LAUNCHER_SPARSITY(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE)   if (is_block_sparse) {                                                       CALL_V2_LAUNCHER(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE, true);         } else {                                                                     CALL_V2_LAUNCHER(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE, false);        }
```
```cpp
#define CALL_V2_LAUNCHER_BLOCK_SIZE(T, CACHE_T, KV_DTYPE)           switch (block_size) {                                               case 8:                                                             CALL_V2_LAUNCHER_SPARSITY(T, CACHE_T, 8, KV_DTYPE);               break;                                                          case 16:                                                            CALL_V2_LAUNCHER_SPARSITY(T, CACHE_T, 16, KV_DTYPE);              break;                                                          case 32:                                                            CALL_V2_LAUNCHER_SPARSITY(T, CACHE_T, 32, KV_DTYPE);              break;                                                          default:                                                            TORCH_CHECK(false, "Unsupported block size: ", block_size);       break;                                                        }
```
**EN:** The block-size and sparse-mode dispatch pattern is the same as v1, because the same underlying paged-attention template supports both algorithms. The difference is that v2 also threads through temporary tensors that hold per-partition statistics for later reduction.
**CN:** block 大小和稀疏模式的分发方式与 v1 相同，因为两种算法底层共用同一个 paged-attention 模板。不同点在于 v2 还要额外传递用于后续归约的临时张量，以保存每个分区的统计信息。

### Public API / 对外 API
```cpp
void paged_attention_v2(
    torch::Tensor& out,
    torch::Tensor& exp_sums,
    torch::Tensor& max_logits,
    torch::Tensor& tmp_out,
    torch::Tensor& query,
    torch::Tensor& key_cache,
    torch::Tensor& value_cache,
    int64_t num_kv_heads,
    double scale,
    torch::Tensor& block_tables,
    torch::Tensor& seq_lens,
    int64_t block_size, int64_t max_seq_len,
    const std::optional<torch::Tensor>& alibi_slopes,
    const std::string& kv_cache_dtype, torch::Tensor& k_scale,
    torch::Tensor& v_scale, const int64_t tp_rank,
    const int64_t blocksparse_local_blocks,
    const int64_t blocksparse_vert_stride, const int64_t blocksparse_block_size,
    const int64_t blocksparse_head_sliding_step) {
  const bool is_block_sparse = (blocksparse_vert_stride > 1);
  DISPATCH_BY_KV_CACHE_DTYPE(query.dtype(), kv_cache_dtype,
                             CALL_V2_LAUNCHER_BLOCK_SIZE)
}
```
**EN:** The visible API differs from v1 mainly by requiring `exp_sums`, `max_logits`, and `tmp_out`. Those tensors are the signature of the partitioned algorithm: they persist intermediate values that make long-sequence reduction numerically stable and efficient.
**CN:** 相比 v1，这个对外 API 最大的差别是需要 `exp_sums`、`max_logits` 和 `tmp_out`。这些张量正是分区算法的特征：它们保留中间值，使长序列归约既数值稳定又高效。

## Key Concepts / 关键概念
- **EN:** v2 is designed for partitioned attention, which is useful when a single sequence is too large for the v1 one-shot path.  
  **CN:** v2 面向分区式 attention，当单个序列过长、不适合 v1 的一次性路径时尤其有用。
- **EN:** The first kernel computes partition-local softmax statistics; the second kernel combines them using a stable reduction.  
  **CN:** 第一个内核计算分区级 softmax 统计量；第二个内核再用稳定的方式把它们合并。
- **EN:** `PARTITION_SIZE` is a compile-time constant, defaulting to 512 in this launcher.  
  **CN:** `PARTITION_SIZE` 是编译期常量，在这个启动器里默认值为 512。
- **EN:** Like v1, runtime choices are lowered into template instantiations for performance-critical code.  
  **CN:** 和 v1 一样，这里的运行时选择也会下沉为模板实例化，以服务性能关键路径。

## Dependencies / 依赖关系
- **EN:** `attention_kernels.cuh` defines both `paged_attention_v2_kernel` and `paged_attention_v2_reduce_kernel`.  
  **CN:** `attention_kernels.cuh` 同时定义了 `paged_attention_v2_kernel` 与 `paged_attention_v2_reduce_kernel`。
- **EN:** `../cuda_compat.h` contributes CUDA helper macros and compatibility wrappers.  
  **CN:** `../cuda_compat.h` 提供 CUDA 辅助宏和兼容层封装。
- **EN:** `DISPATCH_BY_KV_CACHE_DTYPE` selects the KV-cache storage type, including FP8 cache variants.  
  **CN:** `DISPATCH_BY_KV_CACHE_DTYPE` 用于选择 KV cache 的存储类型，包括 FP8 cache 变体。
- **EN:** `exp_sums`, `max_logits`, and `tmp_out` are mandatory temporary buffers owned by the caller for the partitioned algorithm.  
  **CN:** `exp_sums`、`max_logits` 与 `tmp_out` 是分区算法必须由调用方提供的临时缓冲区。
- **EN:** The reduce kernel behavior is defined in `attention_kernels.cuh`, where single-partition cases can skip reduction and copy directly.  
  **CN:** 归约内核的具体行为定义在 `attention_kernels.cuh` 中，其中单分区场景会跳过归约、直接复制结果。
