# paged_attention_v1.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/attention/paged_attention_v1.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provide the host-side launcher for paged attention v1 by selecting shared memory size, compile-time head/block variants, KV-cache dtype, and optional block-sparse mode. / [CN] 作为 paged attention v1 的 Host 侧启动器，负责选择共享内存大小、编译期的 head/block 变体、KV cache 数据类型，以及可选的 block-sparse 模式。

## Line-by-Line Analysis / 逐行分析
### Launch macro for the concrete CUDA kernel / 具体 CUDA 内核的启动宏
```cpp
#define LAUNCH_PAGED_ATTENTION_V1(HEAD_SIZE)                                  VLLM_DevFuncAttribute_SET_MaxDynamicSharedMemorySize(                           ((void*)vllm::paged_attention_v1_kernel<T, CACHE_T, HEAD_SIZE,                                                      BLOCK_SIZE, NUM_THREADS,                                                    KV_DTYPE, IS_BLOCK_SPARSE>),        shared_mem_size);                                                       vllm::paged_attention_v1_kernel<T, CACHE_T, HEAD_SIZE, BLOCK_SIZE,                                          NUM_THREADS, KV_DTYPE, IS_BLOCK_SPARSE>         <<<grid, block, shared_mem_size, stream>>>(                                     out_ptr, query_ptr, key_cache_ptr, value_cache_ptr, num_kv_heads,           scale, block_tables_ptr, seq_lens_ptr, max_num_blocks_per_seq,              alibi_slopes_ptr, q_stride, kv_block_stride, kv_head_stride,                k_scale_ptr, v_scale_ptr, tp_rank, blocksparse_local_blocks,                blocksparse_vert_stride, blocksparse_block_size,                            blocksparse_head_sliding_step);
```
**EN:** This macro materializes a fully specialized kernel instantiation. Before launching, it requests enough dynamic shared memory for that variant. The parameter list shows that v1 writes the final output directly and consumes the query tensor, paged KV caches, block-table indirection, optional ALiBi slopes, FP8 cache scales, and block-sparse metadata.
**CN:** 这个宏会实例化一个完全特化的内核版本。启动前，它先为该变体申请足够的动态共享内存。参数列表也表明：v1 会直接写最终输出，并消费 query、分页 KV cache、block table 间接索引、可选的 ALiBi slope、FP8 cache scale，以及 block-sparse 元数据。

### Launcher setup and shared-memory sizing / 启动器初始化与共享内存大小计算
```cpp
template <typename T, typename CACHE_T, int BLOCK_SIZE,
          vllm::Fp8KVCacheDataType KV_DTYPE, bool IS_BLOCK_SPARSE,
          int NUM_THREADS = 128>
void paged_attention_v1_launcher(
    torch::Tensor& out, torch::Tensor& query, torch::Tensor& key_cache,
    torch::Tensor& value_cache, int num_kv_heads, float scale,
    torch::Tensor& block_tables, torch::Tensor& seq_lens, int max_seq_len,
    const std::optional<torch::Tensor>& alibi_slopes, torch::Tensor& k_scale,
    torch::Tensor& v_scale, const int tp_rank,
    const int blocksparse_local_blocks, const int blocksparse_vert_stride,
    const int blocksparse_block_size, const int blocksparse_head_sliding_step) {
  int num_seqs = query.size(0);
  int num_heads = query.size(1);
  int head_size = query.size(2);
  int max_num_blocks_per_seq = block_tables.size(1);
  int q_stride = query.stride(0);
  int kv_block_stride = key_cache.stride(0);
  int kv_head_stride = key_cache.stride(1);
  ...
  const int NUM_WARPS = NUM_THREADS / WARP_SIZE;
  int padded_max_seq_len =
      DIVIDE_ROUND_UP(max_seq_len, BLOCK_SIZE) * BLOCK_SIZE;
  int logits_size = padded_max_seq_len * sizeof(float);
  int outputs_size = (NUM_WARPS / 2) * head_size * sizeof(float);
  int shared_mem_size = std::max(logits_size, outputs_size);
```
**EN:** The launcher extracts dimensions and strides from PyTorch tensors, converts optional `alibi_slopes` into a nullable pointer, and computes the dynamic shared memory footprint. The workspace is the max of a logits buffer and a partial-output buffer, matching the implementation detail in `attention_kernels.cuh`.
**CN:** 启动器先从 PyTorch 张量中取出维度和 stride，把可选的 `alibi_slopes` 转成可空指针，然后计算动态共享内存需求。这个工作区大小取 logits 缓冲区和局部输出缓冲区中的较大者，对应 `attention_kernels.cuh` 里的实现细节。

### Head-size specialization / head 大小特化分发
```cpp
dim3 grid(num_heads, num_seqs, 1);
dim3 block(NUM_THREADS);
const at::cuda::OptionalCUDAGuard device_guard(device_of(query));
const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
switch (head_size) {
  case 32:
    LAUNCH_PAGED_ATTENTION_V1(32);
    break;
  case 64:
    LAUNCH_PAGED_ATTENTION_V1(64);
    break;
  ...
  case 256:
    LAUNCH_PAGED_ATTENTION_V1(256);
    break;
  default:
    TORCH_CHECK(false, "Unsupported head size: ", head_size);
    break;
}
```
**EN:** v1 only compiles a curated list of head sizes used in real models, trading genericity for lower compile time and better code generation. The launch grid is `(num_heads, num_seqs, 1)`, so each block handles one `(sequence, head)` pair without partitioning the sequence dimension.
**CN:** v1 只为真实模型中常见的一组 head size 生成代码，用更低的编译成本和更好的代码生成来换取通用性。其 launch grid 为 `(num_heads, num_seqs, 1)`，意味着每个 block 处理一个 `(sequence, head)` 对，不会再对序列维度做分区。

### Block-size and sparsity dispatch macros / block 大小与稀疏模式分发宏
```cpp
#define CALL_V1_LAUNCHER_SPARSITY(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE)   if (is_block_sparse) {                                                       CALL_V1_LAUNCHER(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE, true);         } else {                                                                     CALL_V1_LAUNCHER(T, CACHE_T, BLOCK_SIZE, IS_FP8_KV_CACHE, false);        }
```
```cpp
#define CALL_V1_LAUNCHER_BLOCK_SIZE(T, CACHE_T, KV_DTYPE)           switch (block_size) {                                               case 8:                                                             CALL_V1_LAUNCHER_SPARSITY(T, CACHE_T, 8, KV_DTYPE);               break;                                                          case 16:                                                            CALL_V1_LAUNCHER_SPARSITY(T, CACHE_T, 16, KV_DTYPE);              break;                                                          case 32:                                                            CALL_V1_LAUNCHER_SPARSITY(T, CACHE_T, 32, KV_DTYPE);              break;                                                          default:                                                            TORCH_CHECK(false, "Unsupported block size: ", block_size);       break;                                                        }
```
**EN:** The public API still accepts runtime `block_size` and sparse-mode flags, but the launcher converts them into compile-time template parameters through nested macros. This is important because the underlying kernel relies heavily on static constants for layout, loop unrolling, and shared-memory organization.
**CN:** 对外 API 虽然接收运行时的 `block_size` 和稀疏模式标志，但启动器会通过嵌套宏把它们转换成编译期模板参数。这很重要，因为底层内核强依赖静态常量来完成布局选择、循环展开和共享内存组织。

### Public entry point / 对外入口函数
```cpp
void paged_attention_v1(
    torch::Tensor& out,
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
                             CALL_V1_LAUNCHER_BLOCK_SIZE)
}
```
**EN:** The top-level function is thin: it infers whether block-sparse attention is active and then dispatches on both the query dtype and the KV-cache storage format. `DISPATCH_BY_KV_CACHE_DTYPE` is what lets the same Python-visible API support normal and FP8 KV caches.
**CN:** 顶层函数本身很薄：它先根据 `blocksparse_vert_stride` 判断是否启用 block-sparse attention，然后同时按 query 数据类型与 KV cache 存储格式进行分发。`DISPATCH_BY_KV_CACHE_DTYPE` 让同一个 Python 可见 API 同时支持普通 KV cache 和 FP8 KV cache。

## Key Concepts / 关键概念
- **EN:** This file does not implement attention math directly; it specializes and launches kernels defined in `attention_kernels.cuh`.  
  **CN:** 这个文件本身不直接实现 attention 数学，而是对 `attention_kernels.cuh` 中定义的内核做特化与启动。
- **EN:** Dynamic shared memory size depends on sequence length and head size, so the launcher must compute it per call.  
  **CN:** 动态共享内存大小依赖序列长度与 head size，因此必须由启动器在每次调用时计算。
- **EN:** v1 is a single-pass path: one kernel produces the final `out` tensor directly.  
  **CN:** v1 是单阶段路径：一个内核直接产出最终的 `out` 张量。
- **EN:** Runtime options are funneled into compile-time template choices for performance.  
  **CN:** 运行时选项会被收束成编译期模板选择，以获得更好的性能。

## Dependencies / 依赖关系
- **EN:** `attention_kernels.cuh` defines `paged_attention_v1_kernel`, which this file instantiates and launches.  
  **CN:** `attention_kernels.cuh` 定义了 `paged_attention_v1_kernel`，本文件负责实例化并启动它。
- **EN:** `../cuda_compat.h` provides CUDA-compatibility helpers such as `VLLM_DevFuncAttribute_SET_MaxDynamicSharedMemorySize`.  
  **CN:** `../cuda_compat.h` 提供 CUDA 兼容辅助，例如 `VLLM_DevFuncAttribute_SET_MaxDynamicSharedMemorySize`。
- **EN:** `DISPATCH_BY_KV_CACHE_DTYPE` selects the concrete cache element type and FP8-cache mode.  
  **CN:** `DISPATCH_BY_KV_CACHE_DTYPE` 用于选择具体的 cache 元素类型以及 FP8 cache 模式。
- **EN:** The tensors `block_tables`, `seq_lens`, `k_scale`, and `v_scale` provide the indirection and quantization metadata required by the kernel.  
  **CN:** `block_tables`、`seq_lens`、`k_scale` 与 `v_scale` 这些张量向内核提供了分页寻址与量化所需的元数据。
