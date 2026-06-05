# scaled_mm_c3x_sm120.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c3x_sm120.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements quantized GEMM operations using CUTLASS 3.x API for NVIDIA SM120 (next-generation Blackwell) GPUs / [CN] 使用 CUTLASS 3.x API 为 NVIDIA SM120（下一代 Blackwell）GPU 实现量化 GEMM 操作

## Line-by-Line Analysis / 逐行分析

### Header Includes
```cpp
#include "c3x/scaled_mm_helper.hpp"
#include "c3x/scaled_mm_kernels.hpp"
```
**EN:** Includes helper functions and kernel declarations for CUTLASS 3.x scaled matrix multiplication. These provide the infrastructure for dispatching and executing optimized GEMM kernels.

**CN:** 包含 CUTLASS 3.x 缩放矩阵乘法的辅助函数和内核声明。这些提供了分发和执行优化 GEMM 内核的基础设施。

### File Documentation Comment
```cpp
/*
   This file defines quantized GEMM operations using the CUTLASS 3.x API, for
   NVIDIA GPUs with sm120 (Blackwell).
*/
```
**EN:** Indicates this file targets SM120 compute capability, which is a variant or future iteration of the Blackwell architecture with potentially enhanced features over SM100.

**CN:** 表明此文件针对 SM120 计算能力，这是 Blackwell 架构的变体或未来版本，可能具有比 SM100 更增强的功能。

### Conditional Compilation Guard
```cpp
#if defined ENABLE_SCALED_MM_SM120 && ENABLE_SCALED_MM_SM120
```
**EN:** Conditional compilation directive ensuring this code is only included when SM120 support is explicitly enabled in the build system. This provides fine-grained control over which GPU architectures are supported in the final binary.

**CN:** 条件编译指令确保仅在构建系统中显式启用 SM120 支持时才包含此代码。这提供了对最终二进制文件中支持哪些 GPU 架构的细粒度控制。

### Main Function Interface
```cpp
void cutlass_scaled_mm_sm120(torch::stable::Tensor& c,
                             torch::stable::Tensor const& a,
                             torch::stable::Tensor const& b,
                             torch::stable::Tensor const& a_scales,
                             torch::stable::Tensor const& b_scales,
                             std::optional<torch::stable::Tensor> const& bias) {
```
**EN:** Entry point for SM120 scaled matrix multiplication. Parameters include output tensor `c` (modified in-place), input matrices `a` and `b`, scaling factors for both inputs, and an optional bias vector. The stable ABI ensures compatibility across PyTorch versions.

**CN:** SM120 缩放矩阵乘法的入口点。参数包括输出张量 `c`（就地修改）、输入矩阵 `a` 和 `b`、两个输入的缩放因子以及可选的偏置向量。稳定 ABI 确保跨 PyTorch 版本的兼容性。

### Dispatch Call
```cpp
  dispatch_scaled_mm(c, a, b, a_scales, b_scales, bias,
                     vllm::cutlass_scaled_mm_sm120_fp8,
                     nullptr,  // int8 not supported on SM120
                     vllm::cutlass_scaled_mm_blockwise_sm120_fp8);
```
**EN:** Dispatches to SM120-specific kernel implementations. Like SM100, SM120 only supports FP8 quantization (both tensor-wise and block-wise), with INT8 explicitly unsupported. The dispatch layer automatically selects the appropriate kernel variant based on the scale tensor shapes.

**CN:** 分发到 SM120 特定的内核实现。与 SM100 类似，SM120 仅支持 FP8 量化（张量级和块级），明确不支持 INT8。分发层根据缩放张量形状自动选择适当的内核变体。

### Closing Guards
```cpp
}

#endif
```
**EN:** Closes the function definition and conditional compilation block, matching the opening `#if defined` directive.

**CN:** 关闭函数定义和条件编译块，与开始的 `#if defined` 指令匹配。

## Key Concepts / 关键概念

- **SM120 Architecture** - [EN] Next-generation or variant Blackwell GPU with compute capability 120 / [CN] 下一代或变体 Blackwell GPU，计算能力为 120
- **FP8-Only Quantization** - [EN] Exclusive support for FP8 data type, leveraging specialized tensor cores for maximum performance / [CN] 专门支持 FP8 数据类型，利用专用张量核心以实现最佳性能
- **Tensor-wise vs Block-wise Scaling** - [EN] Two granularities of quantization - single scale per tensor or multiple scales per tensor blocks / [CN] 两种量化粒度 - 每个张量一个缩放因子或每个张量块多个缩放因子
- **CUTLASS 3.x Dispatch** - [EN] Polymorphic dispatch mechanism that selects optimal kernel based on runtime tensor properties / [CN] 多态分发机制，根据运行时张量属性选择最优内核
- **In-place Output** - [EN] Output tensor `c` is modified directly without allocation, improving memory efficiency / [CN] 输出张量 `c` 直接修改而不分配，提高内存效率
- **Optional Bias Fusion** - [EN] Bias addition can be fused into the GEMM kernel for better performance / [CN] 偏置加法可以融合到 GEMM 内核中以获得更好的性能
- **Build-time Conditional Compilation** - [EN] SM120 support can be toggled at compile time for smaller binaries / [CN] SM120 支持可在编译时切换以获得更小的二进制文件

## Dependencies / 依赖关系

- **Helper Headers** - [EN] `c3x/scaled_mm_helper.hpp` provides `dispatch_scaled_mm` function / [CN] `c3x/scaled_mm_helper.hpp` 提供 `dispatch_scaled_mm` 函数
- **Kernel Headers** - [EN] `c3x/scaled_mm_kernels.hpp` declares kernel function prototypes / [CN] `c3x/scaled_mm_kernels.hpp` 声明内核函数原型
- **PyTorch Stable API** - [EN] `torch::stable::Tensor` and `std::optional` for cross-version compatibility / [CN] `torch::stable::Tensor` 和 `std::optional` 实现跨版本兼容性
- **CUTLASS Library** - [EN] NVIDIA's CUDA template library for high-performance linear algebra / [CN] NVIDIA 的 CUDA 模板库，用于高性能线性代数
- **Kernel Implementations** - [EN] `vllm::cutlass_scaled_mm_sm120_fp8` and `vllm::cutlass_scaled_mm_blockwise_sm120_fp8` / [CN] `vllm::cutlass_scaled_mm_sm120_fp8` 和 `vllm::cutlass_scaled_mm_blockwise_sm120_fp8`
- **Build Macro** - [EN] `ENABLE_SCALED_MM_SM120` must be defined to enable this code path / [CN] 必须定义 `ENABLE_SCALED_MM_SM120` 以启用此代码路径
- **CUDA Runtime** - [EN] Requires CUDA toolkit compatible with SM120 architecture / [CN] 需要与 SM120 架构兼容的 CUDA 工具包
