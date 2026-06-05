# scaled_mm_c3x_sm90.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c3x_sm90.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements quantized GEMM operations using CUTLASS 3.x API for NVIDIA Hopper (SM90a) GPUs / [CN] 使用 CUTLASS 3.x API 为 NVIDIA Hopper (SM90a) GPU 实现量化 GEMM 操作

## Line-by-Line Analysis / 逐行分析

### Header Includes
```cpp
#include "c3x/scaled_mm_helper.hpp"
#include "c3x/scaled_mm_kernels.hpp"
```
**EN:** Includes helper utilities and kernel declarations specific to CUTLASS 3.x scaled matrix multiplication, providing dispatch and kernel execution infrastructure.

**CN:** 包含 CUTLASS 3.x 缩放矩阵乘法特定的辅助工具和内核声明，提供分发和内核执行基础设施。

### File Documentation Comment
```cpp
/*
   This file defines quantized GEMM operations using the CUTLASS 3.x API, for
   NVIDIA GPUs with sm90a (Hopper).
*/
```
**EN:** Documents that this file targets SM90a (Hopper architecture), which is NVIDIA's architecture generation before Blackwell, featuring the first generation of FP8 tensor cores.

**CN:** 说明此文件针对 SM90a（Hopper 架构），这是 Blackwell 之前的 NVIDIA 架构，具有第一代 FP8 张量核心。

### Conditional Compilation Guard
```cpp
#if defined ENABLE_SCALED_MM_SM90 && ENABLE_SCALED_MM_SM90
```
**EN:** Ensures the code is only compiled when SM90 support is enabled, allowing selective compilation for different GPU targets.

**CN:** 确保仅在启用 SM90 支持时才编译代码，允许针对不同 GPU 目标进行选择性编译。

### Main Scaled MM Function
```cpp
void cutlass_scaled_mm_sm90(torch::stable::Tensor& c,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias) {
```
**EN:** Primary function for SM90 scaled matrix multiplication. Unlike SM100/SM120, this function supports both FP8 and INT8 quantization formats.

**CN:** SM90 缩放矩阵乘法的主函数。与 SM100/SM120 不同，此函数支持 FP8 和 INT8 量化格式。

### Dispatch with Multiple Formats
```cpp
  dispatch_scaled_mm(c, a, b, a_scales, b_scales, bias,
                     vllm::cutlass_scaled_mm_sm90_fp8,
                     vllm::cutlass_scaled_mm_sm90_int8,
                     vllm::cutlass_scaled_mm_blockwise_sm90_fp8);
```
**EN:** Dispatches to appropriate kernel based on data type. SM90 supports three kernel variants: FP8 tensor-wise, INT8 tensor-wise, and FP8 block-wise. The dispatch logic examines input tensor dtypes and scale shapes to select the correct kernel.

**CN:** 根据数据类型分发到适当的内核。SM90 支持三种内核变体：FP8 张量级、INT8 张量级和 FP8 块级。分发逻辑检查输入张量数据类型和缩放形状以选择正确的内核。

### AZP (Asymmetric Zero Point) Function Declaration
```cpp
void cutlass_scaled_mm_azp_sm90(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, torch::stable::Tensor const& azp_adj,
    std::optional<torch::stable::Tensor> const& azp,
    std::optional<torch::stable::Tensor> const& bias) {
```
**EN:** Extended function supporting asymmetric quantization with zero-point adjustment. This is important for INT8 quantization where the quantization range may not be symmetric around zero. Parameters include adjustment factor `azp_adj` and optional per-row zero point `azp`.

**CN:** 支持带零点调整的非对称量化的扩展函数。这对于 INT8 量化很重要，其中量化范围可能不以零为中心对称。参数包括调整因子 `azp_adj` 和可选的按行零点 `azp`。

### Scale Type Validation
```cpp
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
```
**EN:** Validates that both scaling factors are float32 tensors. This ensures numerical stability in the dequantization process, as scales typically require higher precision than the quantized data.

**CN:** 验证两个缩放因子都是 float32 张量。这确保了反量化过程中的数值稳定性，因为缩放因子通常需要比量化数据更高的精度。

### AZP Kernel Call
```cpp
  vllm::cutlass_scaled_mm_azp_sm90_int8(out, a, b, a_scales, b_scales, azp_adj,
                                        azp, bias);
```
**EN:** Calls the INT8 asymmetric quantization kernel implementation. This kernel handles the additional complexity of zero-point correction during the GEMM operation.

**CN:** 调用 INT8 非对称量化内核实现。此内核处理 GEMM 操作期间零点校正的额外复杂性。

### Closing Guards
```cpp
}

#endif
```
**EN:** Closes both function definitions and the conditional compilation block.

**CN:** 关闭两个函数定义和条件编译块。

## Key Concepts / 关键概念

- **Hopper Architecture (SM90a)** - [EN] NVIDIA's H100 GPU generation with first-gen FP8 support and transformer engine / [CN] NVIDIA 的 H100 GPU 代，具有第一代 FP8 支持和 Transformer 引擎
- **Dual Format Support** - [EN] SM90 uniquely supports both FP8 and INT8, providing flexibility for different model types / [CN] SM90 独特地支持 FP8 和 INT8，为不同模型类型提供灵活性
- **Asymmetric Zero Point (AZP)** - [EN] Quantization technique allowing non-zero center for better accuracy with asymmetric distributions / [CN] 量化技术允许非零中心，以便在非对称分布中获得更好的精度
- **Zero-Point Adjustment** - [EN] Per-column adjustment `azp_adj` and optional per-row zero-point `azp` for fine-grained control / [CN] 每列调整 `azp_adj` 和可选的每行零点 `azp` 以实现细粒度控制
- **Float32 Scales** - [EN] Scaling factors maintained at higher precision for numerical stability / [CN] 缩放因子保持更高精度以实现数值稳定性
- **Block-wise FP8** - [EN] Finer-grained FP8 quantization with per-block scales for improved accuracy / [CN] 更细粒度的 FP8 量化，具有每块缩放因子以提高精度
- **INT8 vs FP8** - [EN] INT8 better for activations with asymmetric distributions, FP8 for symmetric weight distributions / [CN] INT8 更适合非对称分布的激活，FP8 适合对称权重分布

## Dependencies / 依赖关系

- **Helper Infrastructure** - [EN] `c3x/scaled_mm_helper.hpp` provides multi-format dispatch logic / [CN] `c3x/scaled_mm_helper.hpp` 提供多格式分发逻辑
- **Kernel Declarations** - [EN] `c3x/scaled_mm_kernels.hpp` declares all kernel variants / [CN] `c3x/scaled_mm_kernels.hpp` 声明所有内核变体
- **PyTorch Stable API** - [EN] `torch::stable::Tensor` for ABI stability across versions / [CN] `torch::stable::Tensor` 实现跨版本 ABI 稳定性
- **Kernel Implementations** - [EN] `cutlass_scaled_mm_sm90_fp8`, `cutlass_scaled_mm_sm90_int8`, `cutlass_scaled_mm_blockwise_sm90_fp8`, `cutlass_scaled_mm_azp_sm90_int8` / [CN] 四个内核实现函数
- **CUDA 12.0+** - [EN] Minimum CUDA version required for SM90 FP8 tensor core support / [CN] SM90 FP8 张量核心支持所需的最低 CUDA 版本
- **Validation Macros** - [EN] `STD_TORCH_CHECK` for runtime parameter validation / [CN] `STD_TORCH_CHECK` 用于运行时参数验证
- **Build Configuration** - [EN] `ENABLE_SCALED_MM_SM90` must be defined at compile time / [CN] 编译时必须定义 `ENABLE_SCALED_MM_SM90`
