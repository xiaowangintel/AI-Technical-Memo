# torch_bindings.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/torch_bindings.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Registers all vLLM custom CUDA operations with PyTorch using the stable ABI for cross-version compatibility. / [CN] 使用稳定 ABI 向 PyTorch 注册所有 vLLM 自定义 CUDA 操作，以实现跨版本兼容性。

## Line-by-Line Analysis / 逐行分析

### Header Includes / 头文件包含
```cpp
#include "ops.h"
#include "core/registration.h"
#include <torch/csrc/stable/library.h>
```
**EN:** Includes operation declarations (`ops.h`), registration utilities, and PyTorch stable library API for ABI-stable operation registration.  
**CN:** 包含操作声明（`ops.h`）、注册实用工具和 PyTorch 稳定库 API，用于 ABI 稳定的操作注册。

### Library Fragment Definition / 库片段定义
```cpp
// Register ops with STABLE_TORCH_LIBRARY for libtorch stable ABI compatibility.
// Note: We register under namespace "_C" so ops are accessible as
// torch.ops._C.<op_name> for compatibility with existing code.
STABLE_TORCH_LIBRARY_FRAGMENT(_C, ops) {
```
**EN:** Uses `STABLE_TORCH_LIBRARY_FRAGMENT` macro to register operations in the `_C` namespace, ensuring stable ABI across PyTorch versions. Operations are accessible from Python as `torch.ops._C.<op_name>`.  
**CN:** 使用 `STABLE_TORCH_LIBRARY_FRAGMENT` 宏在 `_C` 命名空间中注册操作，确保跨 PyTorch 版本的稳定 ABI。可从 Python 访问操作为 `torch.ops._C.<op_name>`。

### Quantization Operations / 量化操作
```cpp
#ifndef USE_ROCM
  // Compute per-token-group FP8 quantized tensor and scaling factor.
  // The dummy arguments are here so we can correctly fuse with RMSNorm.
  ops.def(
      "per_token_group_fp8_quant(Tensor input, Tensor! output_q, Tensor! "
      "output_s, int group_size, float eps, float fp8_min, float fp8_max, bool "
      "scale_ue8m0, bool dummy_is_scale_transposed, bool dummy_is_tma_aligned "
      ") -> ()");
  // Compute per-token-group 8-bit quantized tensor and UE8M0-packed,
  // TMA-aligned scales for DeepGEMM.
  ops.def(
      "per_token_group_fp8_quant_packed(Tensor input, Tensor! output_q, "
      "Tensor! output_s_packed, int group_size, float eps, float fp8_min, "
      "float fp8_max) -> ()");
  // Compute per-token-group INT8 quantized tensor and scaling factor.
  ops.def(
      "per_token_group_quant_int8(Tensor input, Tensor! output_q, Tensor! "
      "output_s, int group_size, float eps, float int8_min, float int8_max) -> "
      "()");
```
**EN:** Declares three per-token group quantization operations:
- `per_token_group_fp8_quant`: FP8 quantization with dummy args for RMSNorm fusion compatibility
- `per_token_group_fp8_quant_packed`: Optimized FP8 with UE8M0-packed scales for DeepGEMM/Blackwell
- `per_token_group_quant_int8`: INT8 quantization variant

Tensor arguments marked with `!` are in-place modified (outputs).

**CN:** 声明三个每令牌分组量化操作：
- `per_token_group_fp8_quant`：FP8 量化，带有用于 RMSNorm 融合兼容性的虚拟参数
- `per_token_group_fp8_quant_packed`：针对 DeepGEMM/Blackwell 的优化 FP8，带有 UE8M0 打包缩放
- `per_token_group_quant_int8`：INT8 量化变体

标记为 `!` 的张量参数是就地修改的（输出）。

### CUTLASS GEMM Operations / CUTLASS GEMM 操作
```cpp
  // CUTLASS w8a8 GEMM, supporting symmetric per-tensor or per-row/column
  // quantization, as well as bias
  ops.def(
      "cutlass_scaled_mm(Tensor! out, Tensor a,"
      "                  Tensor b, Tensor a_scales,"
      "                  Tensor b_scales, Tensor? bias) -> ()");

  // CUTLASS w8a8 GEMM, supporting asymmetric per-tensor or per-row/column
  // quantization.
  ops.def(
      "cutlass_scaled_mm_azp(Tensor! out, Tensor a,"
      "                  Tensor b, Tensor a_scales,"
      "                  Tensor b_scales, Tensor azp_adj,"
      "                  Tensor? azp, Tensor? bias) -> ()");
```
**EN:** Declares CUTLASS-accelerated 8-bit weight, 8-bit activation (W8A8) matrix multiplication:
- `cutlass_scaled_mm`: Symmetric quantization with optional bias
- `cutlass_scaled_mm_azp`: Asymmetric quantization with asymmetric zero-point (AZP) support

**CN:** 声明 CUTLASS 加速的 8 位权重、8 位激活（W8A8）矩阵乘法：
- `cutlass_scaled_mm`：对称量化，可选偏置
- `cutlass_scaled_mm_azp`：非对称量化，支持非对称零点（AZP）

### MoE Operations / MoE 操作
```cpp
  // CUTLASS w8a8 grouped GEMM
  ops.def(
      "cutlass_moe_mm(Tensor! out_tensors, Tensor a_tensors, Tensor b_tensors, "
      "               Tensor a_scales, Tensor b_scales, Tensor expert_offsets, "
      "               Tensor problem_sizes, Tensor a_strides, "
      "               Tensor b_strides, Tensor c_strides, bool per_act_token, "
      "               bool per_out_ch) -> ()");

  // A function that computes data required to run fused MoE with w8a8 grouped
  // GEMM. It takes topk_ids as an input, and computes expert_offsets
  // (token start indices of each expert). In addition to this, it computes
  // problem sizes for each expert's multiplication used by the two mms called
  // from fused MoE operation, and arrays with permutations required to shuffle
  // and de-shuffle the input/output of the fused operation.
  ops.def(
      "get_cutlass_moe_mm_data(Tensor topk_ids, Tensor! expert_offsets, "
      "                        Tensor! problem_sizes1, Tensor! problem_sizes2, "
      "                        Tensor! input_permutation, "
      "                        Tensor! output_permutation, int num_experts, "
      "                        int n, int k, Tensor? blockscale_offsets, "
      "                        bool is_gated) -> ()");
```
**EN:** Mixture-of-Experts (MoE) operations:
- `cutlass_moe_mm`: Grouped GEMM for multiple experts with W8A8 quantization
- `get_cutlass_moe_mm_data`: Preprocessing function computing expert offsets, problem sizes, and permutations for token routing

**CN:** 混合专家（MoE）操作：
- `cutlass_moe_mm`：多专家分组 GEMM，带 W8A8 量化
- `get_cutlass_moe_mm_data`：预处理函数，计算专家偏移、问题大小和令牌路由所需的排列

### FP4/NVFP4 Operations / FP4/NVFP4 操作
```cpp
  // CUTLASS nvfp4 block scaled GEMM
  ops.def(
      "cutlass_scaled_fp4_mm(Tensor! out, Tensor a, Tensor b,"
      "                      Tensor block_scale_a, Tensor block_scale_b,"
      "                      Tensor alpha) -> ()");

  // Compute NVFP4 block quantized tensor.
  ops.def(
      "scaled_fp4_quant(Tensor input,"
      "                 Tensor input_scale, bool "
      "is_sf_swizzled_layout) -> (Tensor, Tensor)");
```
**EN:** 4-bit floating-point (FP4/NVFP4) operations for ultra-low-precision inference:
- `cutlass_scaled_fp4_mm`: Block-scaled FP4 matrix multiplication
- `scaled_fp4_quant`: FP4 quantization with swizzled scale layout support

**CN:** 4 位浮点（FP4/NVFP4）操作，用于超低精度推理：
- `cutlass_scaled_fp4_mm`：块缩放 FP4 矩阵乘法
- `scaled_fp4_quant`：FP4 量化，支持交错缩放布局

### Implementation Registration / 实现注册
```cpp
STABLE_TORCH_LIBRARY_IMPL(_C, CUDA, ops) {
#ifndef USE_ROCM
  // Per-token group quantization
  ops.impl("per_token_group_fp8_quant", TORCH_BOX(&per_token_group_quant_fp8));
  ops.impl("per_token_group_fp8_quant_packed",
           TORCH_BOX(&per_token_group_quant_8bit_packed));
  ops.impl("per_token_group_quant_int8",
           TORCH_BOX(&per_token_group_quant_int8));

  // CUTLASS scaled_mm ops
  ops.impl("cutlass_scaled_mm", TORCH_BOX(&cutlass_scaled_mm));
  ops.impl("cutlass_scaled_mm_azp", TORCH_BOX(&cutlass_scaled_mm_azp));
  ops.impl("cutlass_moe_mm", TORCH_BOX(&cutlass_moe_mm));
  // ... more implementations
#endif
}
```
**EN:** Registers CUDA implementations for the declared operations. `TORCH_BOX` wraps C++ function pointers for PyTorch's dispatcher. Implementations are device-specific (CUDA backend).  
**CN:** 为声明的操作注册 CUDA 实现。`TORCH_BOX` 为 PyTorch 调度器包装 C++ 函数指针。实现是设备特定的（CUDA 后端）。

### Capability Check Registration / 能力检查注册
```cpp
STABLE_TORCH_LIBRARY_IMPL(_C, CompositeExplicitAutograd, ops) {
#ifndef USE_ROCM
  ops.impl("cutlass_scaled_mm_supports_fp8",
           TORCH_BOX(&cutlass_scaled_mm_supports_fp8));
  ops.impl("cutlass_group_gemm_supported",
           TORCH_BOX(&cutlass_group_gemm_supported));
  ops.impl("cutlass_scaled_mm_supports_block_fp8",
           TORCH_BOX(&cutlass_scaled_mm_supports_block_fp8));
  ops.impl("cutlass_scaled_mm_supports_fp4",
           TORCH_BOX(&cutlass_scaled_mm_supports_fp4));
#endif
}
```
**EN:** Registers device capability check functions in `CompositeExplicitAutograd` dispatch key. These functions take primitive arguments (no tensors), making them backend-agnostic and callable before device initialization.  
**CN:** 在 `CompositeExplicitAutograd` 调度键中注册设备能力检查函数。这些函数接受原始参数（无张量），使其与后端无关，可在设备初始化前调用。

### Extension Registration / 扩展注册
```cpp
REGISTER_EXTENSION(_C_stable_libtorch)
```
**EN:** Registers the entire extension module `_C_stable_libtorch` with PyTorch, making all declared operations available in Python.  
**CN:** 向 PyTorch 注册整个扩展模块 `_C_stable_libtorch`，使所有声明的操作在 Python 中可用。

## Key Concepts / 关键概念

### EN:
- **Stable ABI Registration**: Uses `STABLE_TORCH_LIBRARY_FRAGMENT` instead of traditional `TORCH_LIBRARY` to ensure binary compatibility across PyTorch versions
- **Namespace Organization**: All operations registered under `_C` namespace for backward compatibility with existing Python code
- **Dual Registration Pattern**: Operations are first declared (`ops.def`) with signatures, then implemented (`ops.impl`) with function pointers
- **Dispatch Key Specialization**: 
  - `CUDA` dispatch key for device-specific implementations
  - `CompositeExplicitAutograd` for device-agnostic capability checks
- **In-Place Mutation Markers**: `Tensor!` syntax indicates in-place modification, important for PyTorch's aliasing analysis
- **Optional Arguments**: `Tensor?` syntax for optional tensor arguments, `str?` for optional strings
- **Conditional Compilation**: `#ifndef USE_ROCM` excludes NVIDIA-specific operations when building for AMD ROCm

### CN:
- **稳定 ABI 注册**：使用 `STABLE_TORCH_LIBRARY_FRAGMENT` 而非传统 `TORCH_LIBRARY`，以确保跨 PyTorch 版本的二进制兼容性
- **命名空间组织**：所有操作在 `_C` 命名空间下注册，以实现与现有 Python 代码的向后兼容
- **双重注册模式**：操作首先用签名声明（`ops.def`），然后用函数指针实现（`ops.impl`）
- **调度键专业化**：
  - `CUDA` 调度键用于设备特定实现
  - `CompositeExplicitAutograd` 用于设备无关的能力检查
- **就地修改标记**：`Tensor!` 语法表示就地修改，对 PyTorch 的别名分析很重要
- **可选参数**：`Tensor?` 语法表示可选张量参数，`str?` 表示可选字符串
- **条件编译**：`#ifndef USE_ROCM` 在为 AMD ROCm 构建时排除 NVIDIA 特定操作

## Dependencies / 依赖关系

### EN:
- **PyTorch Stable Library**: `torch/csrc/stable/library.h` for stable ABI registration macros
- **Operation Declarations**: `ops.h` declares function signatures for all registered operations
- **Core Registration**: `core/registration.h` provides `REGISTER_EXTENSION` macro
- **Implementation Files**: Links to numerous `.cu` and `.cpp` files implementing the operations:
  - Quantization: `per_token_group_quant.cu` (FP8, INT8)
  - CUTLASS kernels: `cutlass_scaled_mm.cu`, `cutlass_moe_mm.cu`, etc.
  - FP4 kernels: `scaled_fp4_quant.cu`, `cutlass_scaled_fp4_mm.cu`
  - AWQ: `awq_gemm.cu`, `awq_dequantize.cu`
  - Specialty kernels: `dsv3_fused_a_gemm.cu`, `sm100_cutlass_mla_decode.cu`
- **Python Access**: Registered operations callable as `torch.ops._C.<op_name>` from Python

### CN:
- **PyTorch 稳定库**：`torch/csrc/stable/library.h` 提供稳定 ABI 注册宏
- **操作声明**：`ops.h` 声明所有注册操作的函数签名
- **核心注册**：`core/registration.h` 提供 `REGISTER_EXTENSION` 宏
- **实现文件**：链接到实现操作的众多 `.cu` 和 `.cpp` 文件：
  - 量化：`per_token_group_quant.cu`（FP8、INT8）
  - CUTLASS 内核：`cutlass_scaled_mm.cu`、`cutlass_moe_mm.cu` 等
  - FP4 内核：`scaled_fp4_quant.cu`、`cutlass_scaled_fp4_mm.cu`
  - AWQ：`awq_gemm.cu`、`awq_dequantize.cu`
  - 特殊内核：`dsv3_fused_a_gemm.cu`、`sm100_cutlass_mla_decode.cu`
- **Python 访问**：注册的操作可从 Python 作为 `torch.ops._C.<op_name>` 调用
