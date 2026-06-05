# ops.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/ops.h`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Declares the native C++/CUDA operator surface used by the vLLM torch extension, spanning attention, normalization, quantization, activation, custom collectives, and MiniMax-specific kernels. / [CN] 声明 vLLM torch 扩展所使用的原生 C++/CUDA 算子接口，覆盖 attention、归一化、量化、激活、自定义集合通信以及 MiniMax 专用内核。

## Line-by-Line Analysis / 逐行分析

### Header dependencies and utility helper / 头文件依赖与工具函数
```cpp
#include <optional>
#include <string>
#include <torch/library.h>
#include <tuple>

#include "core/scalar_type.hpp"

#include <vector>

torch::Tensor weak_ref_tensor(torch::Tensor& tensor) {
  if (!tensor.is_cuda()) {
    throw std::runtime_error("Tensor must be on CUDA device");
  }
  ...
  auto new_tensor = torch::from_blob(data_ptr, sizes, strides, options);
  return new_tensor;
}
```
**EN:** Most of the file is forward declarations, but `weak_ref_tensor` is an inline utility. It creates a tensor view from the raw CUDA data pointer, reusing shape, stride, and options without owning the underlying storage.
**CN:** 该文件大部分内容都是前向声明，但 `weak_ref_tensor` 是一个真正实现的内联工具函数。它使用原始 CUDA 数据指针重新构造一个张量视图，复用原张量的 shape、stride 和 options，但并不拥有底层存储。

### Attention-related native entry points / Attention 相关原生入口
```cpp
void paged_attention_v1(...);
void paged_attention_v2(...);
void merge_attn_states(...);
#ifndef USE_ROCM
void convert_vertical_slash_indexes(...);
void convert_vertical_slash_indexes_mergehead(...);
#endif
```
**EN:** This group exposes the paged-attention runtime API plus auxiliary sparse/block-sparse index conversion helpers. `v1` and `v2` suggest multiple kernel generations or execution strategies coexisting behind the same extension boundary.
**CN:** 这一组接口暴露了 paged-attention 运行时 API，以及稀疏/块稀疏索引转换辅助函数。`v1` 和 `v2` 的并存说明扩展层同时维护着多代内核或不同执行策略。

### Normalization and fused norm/rope declarations / 归一化与 fused norm/rope 声明
```cpp
void rms_norm(torch::Tensor& out, torch::Tensor& input, torch::Tensor& weight,
              double epsilon);
void fused_add_rms_norm(torch::Tensor& input, torch::Tensor& residual,
                        torch::Tensor& weight, double epsilon);
void fused_qk_norm_rope(...);
void fused_deepseek_v4_qnorm_rope_kv_rope_quant_insert(...);
```
**EN:** These declarations correspond to latency-sensitive transformer preprocessing paths: RMSNorm itself, fused residual + RMSNorm, and more specialized fusions that combine normalization with rotary position embedding or cache insertion.
**CN:** 这些声明对应 Transformer 中延迟敏感的预处理路径：包括 RMSNorm、本身 fused residual + RMSNorm，以及把归一化与 rotary position embedding 或 cache 插入进一步融合的专用算子。

### Quantization-oriented operators / 面向量化的算子声明
```cpp
void rms_norm_static_fp8_quant(...);
void fused_add_rms_norm_static_fp8_quant(...);
void rms_norm_dynamic_per_token_quant(...);
void rms_norm_per_block_quant(...);
void silu_and_mul_per_block_quant(...);
...
void static_scaled_int8_quant(...);
void dynamic_scaled_int8_quant(...);
void static_scaled_fp8_quant(...);
void dynamic_scaled_fp8_quant(...);
void dynamic_per_token_scaled_fp8_quant(...);
```
**EN:** This header centralizes many quantization interfaces, covering norm-driven quantization, per-block scaling, per-token scaling, INT8 paths, and FP8 paths. Several of these APIs are fused so that activation or normalization output can be quantized immediately.
**CN:** 该头文件集中声明了大量量化接口，涵盖由归一化驱动的量化、per-block/per-token 缩放、INT8 路径和 FP8 路径。其中不少 API 都是 fused 形式，允许激活或归一化结果在产生后立即量化。

### Activation, scan, and GEMM-family ops / 激活、扫描和 GEMM 家族算子
```cpp
void silu_and_mul(torch::Tensor& out, torch::Tensor& input);
void gelu_and_mul(torch::Tensor& out, torch::Tensor& input);
void gelu_tanh_and_mul(torch::Tensor& out, torch::Tensor& input);
void fatrelu_and_mul(torch::Tensor& out, torch::Tensor& input,
                     double threshold);
...
void selective_scan_fwd(...);
...
torch::Tensor ggml_dequantize(...);
torch::Tensor ggml_mul_mat_vec_a8(...);
torch::Tensor ggml_mul_mat_a8(...);
torch::Tensor ggml_moe_a8(...);
```
**EN:** Besides transformer attention and norms, the extension also exposes activation fusion kernels, state-space-model scan kernels, and GGML-compatible matrix/dequantization routines. `ops.h` is therefore a broad registry header rather than a file focused on one subsystem.
**CN:** 除了 Transformer attention 和归一化之外，这个扩展还暴露了激活融合内核、状态空间模型扫描内核以及兼容 GGML 的矩阵/反量化例程。因此 `ops.h` 更像是一个总注册表头文件，而不是单一子系统的专用头。

### Custom collective and shared-buffer API / 自定义集合通信与共享缓冲区 API
```cpp
using fptr_t = int64_t;
fptr_t init_custom_ar(const std::vector<int64_t>& fake_ipc_ptrs,
                      torch::Tensor& rank_data, int64_t rank,
                      bool fully_connected);
void all_reduce(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out,
                fptr_t reg_buffer, int64_t reg_buffer_sz_bytes);
void dispose(fptr_t _fa);
...
std::tuple<int64_t, torch::Tensor> allocate_shared_buffer_and_handle(
    int64_t size);
```
**EN:** These declarations form a small runtime for custom all-reduce / graph-buffer sharing. The opaque `fptr_t` handle lets Python-visible code manage native communication objects without exposing their true C++ type.
**CN:** 这一组声明构成了一个小型运行时，用于自定义 all-reduce 和图缓冲区共享。通过不透明句柄 `fptr_t`，Python 可见层可以管理原生通信对象，而无需暴露真实的 C++ 类型。

### MiniMax-specific exported ops / MiniMax 专用导出算子
```cpp
#ifndef USE_ROCM
torch::Tensor minimax_allreduce_rms(torch::Tensor const& input,
                                    torch::Tensor const& norm_weight,
                                    torch::Tensor workspace, int64_t const rank,
                                    int64_t const nranks, double const eps);
std::tuple<torch::Tensor, torch::Tensor> minimax_allreduce_rms_qk(
    torch::Tensor qkv, torch::Tensor const& norm_weight_q,
    torch::Tensor const& norm_weight_k, torch::Tensor workspace,
    int64_t const q_size, int64_t const kv_size, int64_t const rank,
    int64_t const nranks, double const eps);
#endif
```
**EN:** The tail of the file exposes the MiniMax all-reduce RMS kernels to the extension layer. They are guarded out on ROCm, matching the CUDA-specific implementation in `minimax_reduce_rms_kernel.cu`.
**CN:** 文件末尾把 MiniMax all-reduce RMS 内核导出给扩展层使用。它们在 ROCm 下被条件编译排除，这与 `minimax_reduce_rms_kernel.cu` 中的 CUDA 专用实现保持一致。

## Key Concepts / 关键概念
- **EN:** `ops.h` is the native operator index for the torch extension: it declares what the Python side can bind to.  
  **CN:** `ops.h` 是 torch 扩展的原生算子索引，决定了 Python 侧能够绑定哪些底层实现。
- **EN:** The file spans multiple subsystems, so grouping by functionality is more informative than reading it as one linear API list.  
  **CN:** 该文件跨越多个子系统，因此按功能分组理解要比把它当成线性 API 列表逐条阅读更有效。
- **EN:** ROCm guards reveal which features remain CUDA-only, especially custom collectives and MiniMax kernels.  
  **CN:** ROCm 条件编译能直接看出哪些功能仍是 CUDA 专用，尤其是自定义集合通信和 MiniMax 内核。

## Dependencies / 依赖关系
- **EN:** Declares functions implemented across many `.cu` and `.cc` files, including the RMSNorm, attention, quantization, and MiniMax sources analyzed here.  
  **CN:** 声明的函数由许多 `.cu` 和 `.cc` 文件实现，其中就包括这里分析的 RMSNorm、attention、量化和 MiniMax 相关源码。
- **EN:** Depends on PyTorch C++ extension types (`torch::Tensor`, `torch::library`) and standard C++ containers/utilities.  
  **CN:** 依赖 PyTorch C++ 扩展类型（`torch::Tensor`、`torch::library`）以及标准 C++ 容器/工具。
- **EN:** Serves as the interface boundary between Python operator registration and compiled CUDA/C++ kernels.  
  **CN:** 它是 Python 算子注册层与已编译 CUDA/C++ 内核之间的接口边界。
