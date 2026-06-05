# scaled_mm_c2x.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/cutlass/scaled_mm_c2x.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Main implementation file for quantized GEMM operations using CUTLASS 2.x API for NVIDIA GPUs with SM versions prior to SM90 (Hopper). Provides scaled matrix multiplication with int8 and fp8 quantization support. / [CN] 使用 CUTLASS 2.x API 为 SM90 之前的 NVIDIA GPU 实现量化 GEMM 操作的主文件。提供支持 int8 和 fp8 量化的缩放矩阵乘法。

## Line-by-Line Analysis / 逐行分析

### Includes and Dependencies
```cpp
#include <stddef.h>
#include <torch/csrc/stable/tensor.h>
#include <torch/headeronly/core/ScalarType.h>
#include "cutlass/cutlass.h"

#include "scaled_mm_c2x.cuh"
#include "scaled_mm_c2x_sm75_dispatch.cuh"
#include "scaled_mm_c2x_sm80_dispatch.cuh"
#include "scaled_mm_c2x_sm89_fp8_dispatch.cuh"
#include "scaled_mm_c2x_sm89_int8_dispatch.cuh"
```
**EN:** Includes standard libraries, PyTorch stable tensor APIs, CUTLASS library, and architecture-specific dispatch headers for SM75, SM80, and SM89.
**CN:** 包含标准库、PyTorch 稳定张量 API、CUTLASS 库以及针对 SM75、SM80 和 SM89 的架构特定分发头文件。

### SM75 Entry Point with Epilogue Template
```cpp
template <template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
void cutlass_scaled_mm_sm75_epilogue(torch::stable::Tensor& out,
                                     torch::stable::Tensor const& a,
                                     torch::stable::Tensor const& b,
                                     EpilogueArgs&&... epilogue_args) {
  STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
  STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);
```
**EN:** Template function that validates input types and dispatches to appropriate SM75 GEMM kernel based on output data type (BFloat16 or Half).
**CN:** 模板函数，验证输入类型并根据输出数据类型（BFloat16 或 Half）分发到适当的 SM75 GEMM 内核。

### SM75 Basic Scaled MM
```cpp
void cutlass_scaled_mm_sm75(torch::stable::Tensor& out,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias) {
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  if (bias) {
    STD_TORCH_CHECK(bias->scalar_type() == out.scalar_type(),
                    "currently bias dtype must match output dtype ",
                    out.scalar_type());
    return cutlass_scaled_mm_sm75_epilogue<c2x::ScaledEpilogueBias>(
        out, a, b, a_scales, b_scales, *bias);
  } else {
    return cutlass_scaled_mm_sm75_epilogue<c2x::ScaledEpilogue>(
        out, a, b, a_scales, b_scales);
  }
}
```
**EN:** Public API for SM75 scaled matrix multiplication. Validates scale tensors are float32, checks bias dtype matches output, and selects epilogue with or without bias.
**CN:** SM75 缩放矩阵乘法的公共 API。验证缩放张量为 float32，检查偏置数据类型与输出匹配，并选择带或不带偏置的 epilogue。

### SM75 AZP (Asymmetric Zero Point) Variant
```cpp
void cutlass_scaled_mm_azp_sm75(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, torch::stable::Tensor const& azp_adj,
    std::optional<torch::stable::Tensor> const& azp,
    std::optional<torch::stable::Tensor> const& bias) {
  if (azp) {
    return cutlass_scaled_mm_sm75_epilogue<c2x::ScaledEpilogueBiasAzpToken>(
        out, a, b, a_scales, b_scales, azp_adj, *azp, bias);
  } else {
    return cutlass_scaled_mm_sm75_epilogue<c2x::ScaledEpilogueBiasAzp>(
        out, a, b, a_scales, b_scales, azp_adj, bias);
  }
}
```
**EN:** Handles asymmetric zero-point quantization for SM75. Supports per-token AZP (with azp parameter) or per-channel AZP adjustment only.
**CN:** 处理 SM75 的非对称零点量化。支持逐令牌 AZP（带 azp 参数）或仅逐通道 AZP 调整。

### SM80 Implementation Pattern
```cpp
void cutlass_scaled_mm_sm80(torch::stable::Tensor& out,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias) {
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  if (bias) {
    return cutlass_scaled_mm_sm80_epilogue<c2x::ScaledEpilogueBias>(
        out, a, b, a_scales, b_scales, *bias);
  } else {
    return cutlass_scaled_mm_sm80_epilogue<c2x::ScaledEpilogue>(
        out, a, b, a_scales, b_scales);
  }
}
```
**EN:** SM80 (Ampere) implementation following same pattern as SM75, with improved tensor core capabilities.
**CN:** SM80（Ampere）实现遵循与 SM75 相同的模式，具有改进的张量核心功能。

### SM89 Multi-Precision Support
```cpp
template <template <typename, typename> typename Epilogue,
          typename... EpilogueArgs>
void cutlass_scaled_mm_sm89_epilogue(torch::stable::Tensor& out,
                                     torch::stable::Tensor const& a,
                                     torch::stable::Tensor const& b,
                                     EpilogueArgs&&... epilogue_args) {
  if (a.scalar_type() == torch::headeronly::ScalarType::Char) {
    STD_TORCH_CHECK(b.scalar_type() == torch::headeronly::ScalarType::Char);
    // int8 path
  } else {
    STD_TORCH_CHECK(a.scalar_type() ==
                    torch::headeronly::ScalarType::Float8_e4m3fn);
    STD_TORCH_CHECK(b.scalar_type() ==
                    torch::headeronly::ScalarType::Float8_e4m3fn);
    // fp8 path
  }
}
```
**EN:** SM89 (Ada Lovelace) supports both int8 and fp8 (e4m3) quantization. Runtime type checking dispatches to appropriate kernel variant.
**CN:** SM89（Ada Lovelace）同时支持 int8 和 fp8（e4m3）量化。运行时类型检查分发到适当的内核变体。

### SM89 Batch Invariant Optimization
```cpp
void cutlass_scaled_mm_sm89(torch::stable::Tensor& out,
                            torch::stable::Tensor const& a,
                            torch::stable::Tensor const& b,
                            torch::stable::Tensor const& a_scales,
                            torch::stable::Tensor const& b_scales,
                            std::optional<torch::stable::Tensor> const& bias) {
  if (bias) {
    if (vllm_is_batch_invariant()) {
      return cutlass_scaled_mm_sm89_fp8_batch_invariant_epilogue<
          c2x::ScaledEpilogueBias>(out, a, b, a_scales, b_scales, *bias);
    }
    return cutlass_scaled_mm_sm89_epilogue<c2x::ScaledEpilogueBias>(
        out, a, b, a_scales, b_scales, *bias);
  }
}
```
**EN:** Includes batch-invariant optimization path for SM89 fp8 operations. When enabled, uses specialized kernel that maintains consistent performance across batch sizes.
**CN:** 包含 SM89 fp8 操作的批次不变优化路径。启用时，使用在不同批次大小下保持一致性能的专用内核。

## Key Concepts / 关键概念

- **W8A8 Quantization / W8A8 量化**: [EN] 8-bit weight and 8-bit activation quantization scheme / [CN] 8位权重和8位激活量化方案
- **CUTLASS 2.x API / CUTLASS 2.x API**: [EN] NVIDIA's template library for CUDA GEMM operations / [CN] NVIDIA 的 CUDA GEMM 操作模板库
- **Epilogue Fusion / Epilogue 融合**: [EN] Combines scaling, bias addition, and AZP correction in single kernel pass / [CN] 在单个内核传递中组合缩放、偏置加法和 AZP 校正
- **SM-Specific Dispatch / SM 特定分发**: [EN] Selects optimal kernel configuration based on GPU compute capability (SM75/80/89) / [CN] 根据 GPU 计算能力（SM75/80/89）选择最优内核配置
- **Asymmetric Zero Point (AZP) / 非对称零点（AZP）**: [EN] Handles quantization with non-zero offset for activations / [CN] 处理具有非零偏移的激活量化
- **Batch Invariance / 批次不变性**: [EN] Optimization ensuring consistent kernel performance regardless of batch size / [CN] 确保内核性能不受批次大小影响的优化

## Dependencies / 依赖关系

- **PyTorch Stable API / PyTorch 稳定 API**: [EN] `torch::stable::Tensor` for ABI-stable tensor operations / [CN] `torch::stable::Tensor` 用于 ABI 稳定的张量操作
- **CUTLASS Library / CUTLASS 库**: [EN] Core GPU GEMM primitives and types / [CN] 核心 GPU GEMM 原语和类型
- **Architecture Dispatch Headers / 架构分发头文件**:
  - [EN] `scaled_mm_c2x_sm75_dispatch.cuh` - Turing architecture support / [CN] SM75 分发 - Turing 架构支持
  - [EN] `scaled_mm_c2x_sm80_dispatch.cuh` - Ampere architecture support / [CN] SM80 分发 - Ampere 架构支持
  - [EN] `scaled_mm_c2x_sm89_int8_dispatch.cuh` - Ada int8 support / [CN] SM89 int8 分发 - Ada int8 支持
  - [EN] `scaled_mm_c2x_sm89_fp8_dispatch.cuh` - Ada fp8 support / [CN] SM89 fp8 分发 - Ada fp8 支持
- **Epilogue Definitions / Epilogue 定义**: [EN] `cutlass_extensions/epilogue/scaled_mm_epilogues_c2x.hpp` defines fusion patterns / [CN] 定义融合模式
- **Batch Invariant Checker / 批次不变检查器**: [EN] `core/batch_invariant.hpp` for runtime optimization selection / [CN] 用于运行时优化选择
