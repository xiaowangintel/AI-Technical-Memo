# marlin.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/marlin.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Host-side Marlin entry point: validate tensors, choose a kernel specialization, preprocess act-order inputs, and launch GEMM. / [CN] Marlin 的主机端入口：校验张量、选择内核特化、预处理 act-order 输入并启动 GEMM。

## Line-by-Line Analysis / 逐行分析
### Default kernel placeholder and act-order permutation kernel
```cpp
__global__ void MarlinDefault(MARLIN_KERNEL_PARAMS){};
...
__global__ void permute_cols_kernel(int4 const* __restrict__ a_int4_ptr,
                                    int const* __restrict__ perm_int_ptr,
                                    int4* __restrict__ out_int4_ptr, int size_m,
                                    int size_k, int lda, int block_rows) {
```
**EN:** `MarlinDefault` is the sentinel used when no specialization matches. `permute_cols_kernel` is the one preprocessing kernel in this file: it reorders activation columns according to `perm` so the main GEMM can run on reordered act-order inputs.
**CN:** `MarlinDefault` 是“未匹配到任何特化”时使用的哨兵函数。`permute_cols_kernel` 是本文件唯一的预处理内核：它按照 `perm` 重排激活列，使主 GEMM 可以直接处理 act-order 后的输入。

### Candidate thread-block configurations and cache sizing
```cpp
thread_config_t small_batch_thread_configs[] = {
    {128, 128, 256},
    {64, 128, 128},
    {128, 64, 128}};
...
int get_kernel_cache_size(thread_config_t const& th_config, int thread_m_blocks,
                          int prob_m, int prob_n, int prob_k, int num_bits,
                          int group_size, bool has_act_order, bool is_k_full,
                          int has_zp, bool is_zp_float, bool is_a_8bit,
                          int stages) {
```
**EN:** Marlin does not hardcode one tile shape. It evaluates a small set of prioritized `thread_k/thread_n/num_threads` candidates and estimates each candidate’s shared-memory footprint, including scales, zero-points, act-order metadata, and reduction buffers.
**CN:** Marlin 并不会写死单一 tile 形状。它会评估一组有优先级的 `thread_k/thread_n/num_threads` 候选配置，并估算每个配置的共享内存占用，包括 scale、zero point、act-order 元数据和归约缓冲区。

### Configuration validation and kernel selection
```cpp
bool is_valid_config(thread_config_t const& th_config, ...,
                     int max_shared_mem) {
  if (prob_k % th_config.thread_k != 0 || prob_n % th_config.thread_n != 0) {
    return false;
  }
  int cache_size = get_kernel_cache_size(...);
  return cache_size <= max_shared_mem;
}
...
MarlinFuncPtr get_marlin_kernel(...){
  auto kernel = MarlinDefault;
  #include "kernel_selector.h"
  return kernel;
}
```
**EN:** `is_valid_config` filters out impossible launch shapes, while `get_marlin_kernel` maps the runtime type/config tuple onto a concrete template instantiation generated elsewhere. The `kernel_selector.h` include expands into a large dispatch table at compile time.
**CN:** `is_valid_config` 用于筛掉不可能的启动形状，而 `get_marlin_kernel` 则把运行时类型/配置组合映射到某个具体模板实例。`kernel_selector.h` 会在编译期展开成一个庞大的分发表。

### `marlin_mm`: preprocessing and device capability checks
```cpp
if (has_act_order) {
  permute_cols_kernel<<<sms, default_threads, 0, stream>>>(
      A_ptr, perm_ptr, a_tmp_ptr, prob_m, prob_k, lda, block_rows);
  A_ptr = a_tmp_ptr;
  lda = prob_k;
  if (is_k_full) has_act_order = false;
}
...
TORCH_CHECK(major_capability * 10 + minor_capability >= 75,
            "marlin kernel only support Turing or newer GPUs.");
```
**EN:** `marlin_mm` is the true launcher backend. It optionally permutes activations, normalizes `group_blocks`, rejects unsupported architectures, and applies extra restrictions such as FP8 activation support only on newer GPUs.
**CN:** `marlin_mm` 是真正的启动后端。它会按需重排激活、规范化 `group_blocks`、拒绝不支持的架构，并施加额外限制，例如 FP8 激活只允许在较新的 GPU 上运行。

### Autotuning-like launch loop over M-splits
```cpp
while (rest_m) {
  int thread_m_blocks = min(div_ceil(prob_m_split, 16), max_thread_m_blocks);
  exec_config_t exec_cfg = determine_exec_config(...);
  auto kernel = get_marlin_kernel(...);
  kernel<<<blocks, num_threads, max_shared_mem_new, stream>>>(...);
  A_ptr += prob_m_split * (lda / (is_a_8bit ? 16 : 8));
  C_ptr += prob_m_split * (prob_n / 8);
  rest_m -= prob_m_split;
}
```
**EN:** Large `M` is processed in chunks. For each chunk, Marlin chooses a viable execution config, possibly lowers the `M` block count if shared memory is too tight, and then advances the A/C pointers to process the next slice.
**CN:** 较大的 `M` 会被拆成多个分块处理。对每个分块，Marlin 会选择一个可行的执行配置；若共享内存过紧，还可能降低 `M` 的块数，然后移动 A/C 指针继续处理下一个切片。

### `marlin_gemm`: dtype inference, validation, and Torch registration
```cpp
torch::Tensor marlin_gemm(..., vllm::ScalarTypeId const& b_type_id, ... ) {
  ...
  TORCH_CHECK(a.size(0) == size_m, ...);
  TORCH_CHECK(b_q_weight.is_contiguous(), ...);
  ...
  marlin::marlin_mm(...);
  return c;
}
TORCH_LIBRARY_IMPL_EXPAND(TORCH_EXTENSION_NAME, CUDA, m) {
  m.impl("marlin_gemm", &marlin_gemm);
}
```
**EN:** The public binding translates Torch dtypes into vLLM scalar IDs, validates all optional inputs (`a_scales`, `global_scale`, `b_zeros`, `g_idx`, `perm`, `workspace`), allocates output buffers, and finally hands raw pointers to `marlin_mm`.
**CN:** 对外暴露的绑定函数会把 Torch dtype 映射为 vLLM scalar ID，校验所有可选输入（`a_scales`、`global_scale`、`b_zeros`、`g_idx`、`perm`、`workspace`），分配输出缓冲区，最后把原始指针交给 `marlin_mm`。

## Key Concepts / 关键概念
- Host code selects among many precompiled kernel specializations. / 主机端代码会在许多预编译的内核特化中进行选择。
- Act-order may require an explicit activation-column permutation pass. / act-order 可能需要额外的激活列重排预处理步骤。
- Shared-memory limits are a first-class constraint in launch selection. / 共享内存上限是启动配置选择中的一等约束。

## Dependencies / 依赖关系
- Depends on `kernel.h`, generated `kernel_selector.h`, and Torch registration helpers. / 依赖 `kernel.h`、生成的 `kernel_selector.h` 以及 Torch 注册辅助代码。
- Launches the templated kernel defined in `marlin_template.h`. / 启动的是 `marlin_template.h` 中定义的模板内核。
