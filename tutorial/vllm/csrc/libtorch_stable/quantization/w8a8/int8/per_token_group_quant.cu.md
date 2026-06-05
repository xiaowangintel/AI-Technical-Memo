# per_token_group_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/int8/per_token_group_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Wrapper implementation for INT8 per-token group quantization, delegating to shared 8-bit quantization kernel. / [CN] INT8 每令牌分组量化的包装实现，委托给共享的 8 位量化内核。

## Line-by-Line Analysis / 逐行分析

### Include Statements / 包含语句
```cpp
#include <torch/csrc/stable/tensor.h>
#include "libtorch_stable/quantization/w8a8/per_token_group_quant_8bit.h"
```
**EN:** Imports PyTorch stable ABI tensor interface and the shared header declaring the common 8-bit quantization function used by both FP8 and INT8 variants.  
**CN:** 导入 PyTorch 稳定 ABI 张量接口和声明 FP8 和 INT8 变体共用的 8 位量化函数的共享头文件。

### INT8 Quantization Function / INT8 量化函数
```cpp
void per_token_group_quant_int8(const torch::stable::Tensor& input,
                                torch::stable::Tensor& output_q,
                                torch::stable::Tensor& output_s,
                                int64_t group_size, double eps, double int8_min,
                                double int8_max) {
  per_token_group_quant_8bit(input, output_q, output_s, group_size, eps,
                             int8_min, int8_max);
}
```
**EN:** Thin wrapper that calls the shared `per_token_group_quant_8bit` implementation with INT8-specific min/max bounds. This design allows code reuse between FP8 and INT8 quantization paths since both use the same underlying 8-bit quantization algorithm.  
**CN:** 调用共享的 `per_token_group_quant_8bit` 实现的轻量级包装器，传递 INT8 特定的最小/最大边界。这种设计允许在 FP8 和 INT8 量化路径之间重用代码，因为两者使用相同的底层 8 位量化算法。

## Key Concepts / 关键概念

### EN:
- **Code Reuse Architecture**: INT8 and FP8 quantization share the same kernel implementation (`per_token_group_quant_8bit`), differing only in output data type and value range
- **INT8 Range**: Typically [-128, 127] for signed INT8, defining the clamp bounds for quantized values
- **Minimal Overhead**: Wrapper design adds zero runtime cost, only providing a type-safe API boundary
- **Stable ABI Compatibility**: Uses `torch::stable::Tensor` for cross-version PyTorch compatibility

### CN:
- **代码重用架构**：INT8 和 FP8 量化共享相同的内核实现（`per_token_group_quant_8bit`），仅在输出数据类型和值范围上有所不同
- **INT8 范围**：有符号 INT8 通常为 [-128, 127]，定义量化值的钳制边界
- **最小开销**：包装器设计不增加运行时成本，仅提供类型安全的 API 边界
- **稳定 ABI 兼容性**：使用 `torch::stable::Tensor` 以实现跨版本 PyTorch 兼容性

## Dependencies / 依赖关系

### EN:
- **PyTorch Stable ABI**: `torch/csrc/stable/tensor.h` for tensor type definitions
- **Shared Quantization Kernel**: `per_token_group_quant_8bit.h` declares the common implementation used by both INT8 and FP8 variants
- **Upstream Implementation**: Actual kernel code resides in `csrc/libtorch_stable/quantization/w8a8/fp8/per_token_group_quant.cu`
- **Registration**: Registered as `torch.ops._C.per_token_group_quant_int8` in `torch_bindings.cpp`

### CN:
- **PyTorch 稳定 ABI**：`torch/csrc/stable/tensor.h` 提供张量类型定义
- **共享量化内核**：`per_token_group_quant_8bit.h` 声明 INT8 和 FP8 变体共用的实现
- **上游实现**：实际内核代码位于 `csrc/libtorch_stable/quantization/w8a8/fp8/per_token_group_quant.cu`
- **注册**：在 `torch_bindings.cpp` 中注册为 `torch.ops._C.per_token_group_quant_int8`
