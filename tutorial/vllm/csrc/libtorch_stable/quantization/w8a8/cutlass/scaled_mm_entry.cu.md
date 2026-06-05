# scaled_mm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Main entry point and dispatch logic for CUTLASS scaled matrix multiplication across all GPU architectures / [CN] CUTLASS 缩放矩阵乘法在所有 GPU 架构上的主入口点和分发逻辑

## Line-by-Line Analysis / 逐行分析

### System and Library Headers
```cpp
#include <cudaTypedefs.h>

#include <torch/csrc/stable/tensor.h>

#include "libtorch_stable/torch_utils.h"

#include "cutlass_extensions/common.hpp"
```
**EN:** Includes CUDA type definitions, PyTorch stable tensor API, torch utilities, and CUTLASS common extensions. These provide the foundation for CUDA operations and PyTorch integration.

**CN:** 包含 CUDA 类型定义、PyTorch 稳定张量 API、torch 工具和 CUTLASS 通用扩展。这些为 CUDA 操作和 PyTorch 集成提供基础。

### Forward Declarations - Legacy Architectures
```cpp
void cutlass_scaled_mm_sm75(torch::stable::Tensor& c,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias);

void cutlass_scaled_mm_sm80(/* ... */);

void cutlass_scaled_mm_sm89(/* ... */);
```
**EN:** Forward declarations for Turing (SM75), Ampere (SM80), and Ada Lovelace (SM89) GPU architectures. These use CUTLASS 2.x API and support older quantization formats.

**CN:** Turing (SM75)、Ampere (SM80) 和 Ada Lovelace (SM89) GPU 架构的前向声明。这些使用 CUTLASS 2.x API 并支持较旧的量化格式。

### Conditional Forward Declarations - Hopper
```cpp
#if defined ENABLE_SCALED_MM_SM90 && ENABLE_SCALED_MM_SM90
void cutlass_scaled_mm_sm90(torch::stable::Tensor& c,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias);
#endif
```
**EN:** Conditionally declares SM90 (Hopper) function only when enabled. Hopper introduces FP8 support and uses CUTLASS 3.x API.

**CN:** 仅在启用时有条件地声明 SM90 (Hopper) 函数。Hopper 引入了 FP8 支持并使用 CUTLASS 3.x API。

### MOE (Mixture of Experts) Support
```cpp
#if defined ENABLE_CUTLASS_MOE_SM90 && ENABLE_CUTLASS_MOE_SM90
void cutlass_moe_mm_sm90(torch::stable::Tensor& out_tensors,
                         torch::stable::Tensor const& a_tensors,
                         torch::stable::Tensor const& b_tensors,
                         /* ... */
                         bool per_act_token, bool per_out_ch);
#endif
```
**EN:** Declares grouped GEMM function for Mixture of Experts models, which require batched matrix multiplications with different expert weights. Supports SM90 and SM100 with dynamic routing.

**CN:** 为专家混合模型声明分组 GEMM 函数，这些模型需要使用不同专家权重进行批处理矩阵乘法。支持具有动态路由的 SM90 和 SM100。

### Blackwell Architecture Support
```cpp
#if defined ENABLE_SCALED_MM_SM120 && ENABLE_SCALED_MM_SM120
void cutlass_scaled_mm_sm120(/* ... */);
#endif

#if defined ENABLE_SCALED_MM_SM100 && ENABLE_SCALED_MM_SM100
void cutlass_scaled_mm_sm100(/* ... */);
#endif
```
**EN:** Declares functions for SM100 and SM120 (Blackwell variants). These are the latest architectures with enhanced FP8 capabilities and improved tensor cores.

**CN:** 声明 SM100 和 SM120（Blackwell 变体）的函数。这些是具有增强 FP8 功能和改进张量核心的最新架构。

### Feature Detection - FP8 Support
```cpp
bool cutlass_scaled_mm_supports_fp8(int64_t cuda_device_capability) {
  // CUTLASS FP8 kernels need at least
  //   CUDA 12.0 on SM90 systems (Hopper)
  //   CUDA 12.4 on SM89 systems (Lovelace)

#if defined CUDA_VERSION
  if (cuda_device_capability >= 90) {
    return CUDA_VERSION >= 12000;
  } else if (cuda_device_capability >= 89) {
    return CUDA_VERSION >= 12040;
  }
#endif

  return false;
}
```
**EN:** Runtime check for FP8 support based on GPU compute capability and CUDA version. SM90+ requires CUDA 12.0+, while SM89 requires CUDA 12.4+ for FP8 operations. This prevents running incompatible kernels.

**CN:** 基于 GPU 计算能力和 CUDA 版本的 FP8 支持运行时检查。SM90+ 需要 CUDA 12.0+，而 SM89 需要 CUDA 12.4+ 才能进行 FP8 操作。这可以防止运行不兼容的内核。

### Feature Detection - Block-wise FP8
```cpp
bool cutlass_scaled_mm_supports_block_fp8(int64_t cuda_device_capability) {
  // CUTLASS block-quantized FP8 kernels need at least CUDA 12.0
  // and at least SM90 (Hopper)

#if defined CUDA_VERSION
  if (cuda_device_capability >= 100) {
    return CUDA_VERSION >= 12080;
  } else if (cuda_device_capability >= 90) {
    return CUDA_VERSION >= 12000;
  }
#endif

  return false;
}
```
**EN:** Checks for block-wise FP8 quantization support. Requires CUDA 12.0+ for SM90 and CUDA 12.8+ for SM100+. Block-wise quantization provides better accuracy by using different scales for different tensor blocks.

**CN:** 检查块级 FP8 量化支持。SM90 需要 CUDA 12.0+，SM100+ 需要 CUDA 12.8+。块级量化通过对不同张量块使用不同的缩放因子来提供更好的精度。

### Feature Detection - Grouped GEMM
```cpp
bool cutlass_group_gemm_supported(int64_t cuda_device_capability) {
  // CUTLASS grouped FP8 kernels need at least CUDA 12.3 and SM90 (Hopper)
  // or CUDA 12.8 and SM100 (Blackwell)

#if defined CUDA_VERSION
  if (cuda_device_capability >= 100) {
    return CUDA_VERSION >= 12080;
  }
  if (cuda_device_capability >= 90) {
    return CUDA_VERSION >= 12030;
  }
#endif

  return false;
}
```
**EN:** Determines if grouped GEMM (for MOE models) is supported. Requires CUDA 12.3+ on SM90 or CUDA 12.8+ on SM100+. Grouped GEMM allows batching multiple independent matrix multiplications efficiently.

**CN:** 确定是否支持分组 GEMM（用于 MOE 模型）。SM90 需要 CUDA 12.3+，SM100+ 需要 CUDA 12.8+。分组 GEMM 允许高效地批处理多个独立的矩阵乘法。

### Main Entry Point - cutlass_scaled_mm
```cpp
void cutlass_scaled_mm(torch::stable::Tensor& c, torch::stable::Tensor const& a,
                       torch::stable::Tensor const& b,
                       torch::stable::Tensor const& a_scales,
                       torch::stable::Tensor const& b_scales,
                       std::optional<torch::stable::Tensor> const& bias) {
```
**EN:** Main entry point that dispatches to architecture-specific implementations. This function is called from Python and routes to the appropriate kernel based on GPU architecture.

**CN:** 分发到特定架构实现的主入口点。此函数从 Python 调用并根据 GPU 架构路由到适当的内核。

### Input Validation - Tensor Dimensions
```cpp
  // Checks for conformality
  STD_TORCH_CHECK(a.dim() == 2 && b.dim() == 2 && c.dim() == 2);
  STD_TORCH_CHECK(c.size(0) == a.size(0) && a.size(1) == b.size(0) &&
                  b.size(1) == c.size(1));
```
**EN:** Validates that all tensors are 2D matrices and dimensions are compatible for matrix multiplication: C(m,n) = A(m,k) × B(k,n).

**CN:** 验证所有张量都是 2D 矩阵并且维度与矩阵乘法兼容：C(m,n) = A(m,k) × B(k,n)。

### Input Validation - Memory Layout
```cpp
  // Check for strides and alignment
  STD_TORCH_CHECK(a.stride(1) == 1 && c.stride(1) == 1);  // Row-major
  STD_TORCH_CHECK(b.stride(0) == 1);                      // Column-major
  STD_TORCH_CHECK(c.stride(0) % 16 == 0 &&
                  b.stride(1) % 16 == 0);  // 16 Byte Alignment
```
**EN:** Enforces memory layout requirements: A and C must be row-major (contiguous rows), B must be column-major (contiguous columns), and leading dimensions must be 16-byte aligned for optimal GPU memory access.

**CN:** 强制内存布局要求：A 和 C 必须是行优先（连续行），B 必须是列优先（连续列），并且前导维度必须 16 字节对齐以实现最佳 GPU 内存访问。

### Bias Validation
```cpp
  if (bias) {
    STD_TORCH_CHECK(bias->numel() == b.size(1) && bias->is_contiguous() &&
                    bias->dim() == 1);
  }
```
**EN:** If bias is provided, validates it's a 1D contiguous tensor with size matching output columns (n). Bias is broadcast across all rows.

**CN:** 如果提供偏置，验证它是一个与输出列（n）大小匹配的 1D 连续张量。偏置在所有行上广播。

### Device Guard and SM Detection
```cpp
  const torch::stable::accelerator::DeviceGuard device_guard(
      a.get_device_index());
  int32_t version_num = get_sm_version_num();
```
**EN:** Sets the active CUDA device to match the input tensor's device and retrieves the GPU's compute capability (SM version). The device guard ensures operations execute on the correct GPU in multi-GPU systems.

**CN:** 将活动 CUDA 设备设置为与输入张量的设备匹配，并检索 GPU 的计算能力（SM 版本）。设备保护确保在多 GPU 系统中操作在正确的 GPU 上执行。

### Architecture Dispatch - Blackwell SM120
```cpp
#if defined ENABLE_SCALED_MM_SM120 && ENABLE_SCALED_MM_SM120
  if (version_num >= 120) {
    cutlass_scaled_mm_sm120(c, a, b, a_scales, b_scales, bias);
    return;
  }
#endif
```
**EN:** Dispatches to SM120 implementation for compute capability >= 120. Uses early return pattern to avoid unnecessary checks once a match is found.

**CN:** 为计算能力 >= 120 分发到 SM120 实现。使用提前返回模式以避免在找到匹配后进行不必要的检查。

### Architecture Dispatch - Blackwell SM100
```cpp
#if defined ENABLE_SCALED_MM_SM100 && ENABLE_SCALED_MM_SM100
  if (version_num >= 100 && version_num < 120) {
    cutlass_scaled_mm_sm100(c, a, b, a_scales, b_scales, bias);
    return;
  }
#endif
```
**EN:** Dispatches to SM100 for compute capability 100-119. The range check ensures correct routing for different Blackwell variants.

**CN:** 为计算能力 100-119 分发到 SM100。范围检查确保不同 Blackwell 变体的正确路由。

### Architecture Dispatch - Hopper SM90
```cpp
#if defined ENABLE_SCALED_MM_SM90 && ENABLE_SCALED_MM_SM90
  if (version_num >= 90 && version_num < 100) {
    // Hopper
    cutlass_scaled_mm_sm90(c, a, b, a_scales, b_scales, bias);
    return;
  }
#endif
```
**EN:** Dispatches to SM90 (Hopper) for compute capability 90-99. Hopper introduces first-generation FP8 support and CUTLASS 3.x API.

**CN:** 为计算能力 90-99 分发到 SM90 (Hopper)。Hopper 引入了第一代 FP8 支持和 CUTLASS 3.x API。

### Architecture Dispatch - Legacy Architectures
```cpp
#if defined ENABLE_SCALED_MM_C2X && ENABLE_SCALED_MM_C2X
  if (version_num == 89) {
    // Ada Lovelace
    cutlass_scaled_mm_sm89(c, a, b, a_scales, b_scales, bias);
    return;
  }

  if (version_num >= 80) {
    // Ampere
    cutlass_scaled_mm_sm80(c, a, b, a_scales, b_scales, bias);
    return;
  }

  if (version_num >= 75) {
    // Turing
    cutlass_scaled_mm_sm75(c, a, b, a_scales, b_scales, bias);
    return;
  }
#endif
```
**EN:** Handles legacy architectures using CUTLASS 2.x API: Ada Lovelace (SM89), Ampere (SM80), and Turing (SM75). These use INT8 quantization primarily.

**CN:** 使用 CUTLASS 2.x API 处理旧架构：Ada Lovelace (SM89)、Ampere (SM80) 和 Turing (SM75)。这些主要使用 INT8 量化。

### Error Handling
```cpp
  STD_TORCH_CHECK_NOT_IMPLEMENTED(
      false,
      "No compiled cutlass_scaled_mm for a compute capability less than "
      "CUDA device capability: ",
      version_num);
```
**EN:** Throws error if no kernel is available for the detected GPU architecture. This can occur if the binary was compiled without support for the current GPU or if the GPU is too old.

**CN:** 如果检测到的 GPU 架构没有可用的内核，则抛出错误。如果二进制文件编译时不支持当前 GPU 或 GPU 太旧，则可能发生这种情况。

### MOE Matrix Multiplication Entry Point
```cpp
void cutlass_moe_mm(torch::stable::Tensor& out_tensors,
                    torch::stable::Tensor const& a_tensors,
                    torch::stable::Tensor const& b_tensors,
                    torch::stable::Tensor const& a_scales,
                    torch::stable::Tensor const& b_scales,
                    torch::stable::Tensor const& expert_offsets,
                    torch::stable::Tensor const& problem_sizes,
                    torch::stable::Tensor const& a_strides,
                    torch::stable::Tensor const& b_strides,
                    torch::stable::Tensor const& c_strides, bool per_act_token,
                    bool per_out_ch) {
```
**EN:** Entry point for Mixture of Experts grouped GEMM. Takes additional parameters for expert routing: expert_offsets (which tokens go to which experts), problem_sizes (dimensions for each expert), and strides for batched operations. Supports both per-token and per-channel quantization.

**CN:** 专家混合分组 GEMM 的入口点。为专家路由采用额外参数：expert_offsets（哪些 token 去哪些专家）、problem_sizes（每个专家的维度）和批处理操作的步长。支持按 token 和按通道量化。

### AZP Entry Point
```cpp
void cutlass_scaled_mm_azp(torch::stable::Tensor& c,
                           torch::stable::Tensor const& a,
                           torch::stable::Tensor const& b,
                           torch::stable::Tensor const& a_scales,
                           torch::stable::Tensor const& b_scales,
                           torch::stable::Tensor const& azp_adj,
                           std::optional<torch::stable::Tensor> const& azp,
                           std::optional<torch::stable::Tensor> const& bias) {
```
**EN:** Entry point for asymmetric zero-point quantization. Includes azp_adj (per-column adjustment) and optional azp (per-row zero-point) for handling asymmetric quantization ranges in INT8.

**CN:** 非对称零点量化的入口点。包括 azp_adj（每列调整）和可选的 azp（每行零点），用于处理 INT8 中的非对称量化范围。

### AZP Scale Validation
```cpp
  STD_TORCH_CHECK(a_scales.numel() == 1 || a_scales.numel() == a.size(0));
  STD_TORCH_CHECK(b_scales.numel() == 1 || b_scales.numel() == b.size(1));
```
**EN:** Validates that scales are either single scalars (tensor-wise) or vectors matching matrix dimensions (per-row/per-column). This allows both coarse and fine-grained quantization.

**CN:** 验证缩放因子是单个标量（张量级）或与矩阵维度匹配的向量（按行/按列）。这允许粗粒度和细粒度量化。

### AZP Type Validation
```cpp
  STD_TORCH_CHECK(azp_adj.scalar_type() == torch::headeronly::ScalarType::Int);
  STD_TORCH_CHECK(!azp ||
                  azp->scalar_type() == torch::headeronly::ScalarType::Int);
  STD_TORCH_CHECK(!bias || bias->scalar_type() == c.scalar_type(),
                  "currently bias dtype must match output dtype ",
                  c.scalar_type());
```
**EN:** Enforces that azp_adj and azp are integer tensors (since they represent quantization offsets) and bias must match output dtype. Type safety prevents silent numerical errors.

**CN:** 强制 azp_adj 和 azp 是整数张量（因为它们表示量化偏移量），偏置必须匹配输出数据类型。类型安全防止静默数值错误。

## Key Concepts / 关键概念

- **Multi-Architecture Support** - [EN] Single entry point dispatches to SM75/80/89/90/100/120 implementations / [CN] 单个入口点分发到 SM75/80/89/90/100/120 实现
- **Feature Detection** - [EN] Runtime checks for FP8, block-wise quantization, and grouped GEMM capabilities / [CN] FP8、块级量化和分组 GEMM 功能的运行时检查
- **CUTLASS 2.x vs 3.x** - [EN] Legacy architectures use CUTLASS 2.x, SM90+ use CUTLASS 3.x / [CN] 旧架构使用 CUTLASS 2.x，SM90+ 使用 CUTLASS 3.x
- **Memory Layout Requirements** - [EN] Row-major A/C, column-major B, 16-byte alignment for performance / [CN] 行优先 A/C、列优先 B、16 字节对齐以提高性能
- **Asymmetric Quantization** - [EN] AZP functions support non-zero center quantization for INT8 / [CN] AZP 函数支持 INT8 的非零中心量化
- **Mixture of Experts** - [EN] Specialized grouped GEMM for dynamic expert routing in MOE models / [CN] 用于 MOE 模型中动态专家路由的专用分组 GEMM
- **Conditional Compilation** - [EN] Architecture support controlled via ENABLE_* macros at build time / [CN] 架构支持在构建时通过 ENABLE_* 宏控制
- **Device Guard** - [EN] Ensures CUDA operations execute on correct GPU in multi-GPU systems / [CN] 确保 CUDA 操作在多 GPU 系统中在正确的 GPU 上执行

## Dependencies / 依赖关系

- **CUDA Runtime** - [EN] `cudaTypedefs.h` for CUDA type definitions and runtime / [CN] `cudaTypedefs.h` 用于 CUDA 类型定义和运行时
- **PyTorch Stable API** - [EN] `torch/csrc/stable/tensor.h` for cross-version ABI compatibility / [CN] `torch/csrc/stable/tensor.h` 用于跨版本 ABI 兼容性
- **Torch Utils** - [EN] `libtorch_stable/torch_utils.h` for device guards and utilities / [CN] `libtorch_stable/torch_utils.h` 用于设备保护和工具
- **CUTLASS Extensions** - [EN] `cutlass_extensions/common.hpp` for `get_sm_version_num()` / [CN] `cutlass_extensions/common.hpp` 用于 `get_sm_version_num()`
- **Architecture-Specific Implementations** - [EN] SM75/80/89/90/100/120 kernel implementations / [CN] SM75/80/89/90/100/120 内核实现
- **MOE Helper Functions** - [EN] `get_cutlass_moe_mm_data_caller` and related for expert routing / [CN] `get_cutlass_moe_mm_data_caller` 及相关函数用于专家路由
- **Build System** - [EN] CMake/setup.py must define ENABLE_* macros based on target GPUs / [CN] CMake/setup.py 必须根据目标 GPU 定义 ENABLE_* 宏
- **CUDA Version** - [EN] Compile-time `CUDA_VERSION` macro determines feature availability / [CN] 编译时 `CUDA_VERSION` 宏确定功能可用性
