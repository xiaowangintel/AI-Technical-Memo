# per_token_group_quant_8bit.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/per_token_group_quant_8bit.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Header file declaring shared 8-bit quantization interfaces used by both FP8 and INT8 quantization implementations. / [CN] 声明 FP8 和 INT8 量化实现共用的 8 位量化接口的头文件。

## Line-by-Line Analysis / 逐行分析

### Header Guard and Includes / 头文件保护和包含
```cpp
#pragma once
#include <torch/csrc/stable/tensor.h>
```
**EN:** Standard header guard using `#pragma once` for compilation efficiency, and includes PyTorch stable ABI tensor interface for cross-version compatibility.  
**CN:** 使用 `#pragma once` 的标准头文件保护以提高编译效率，包含 PyTorch 稳定 ABI 张量接口以实现跨版本兼容性。

### Shared 8-bit Quantization Function / 共享 8 位量化函数
```cpp
// 8-bit per-token-group quantization helper used by both FP8 and INT8
void per_token_group_quant_8bit(const torch::stable::Tensor& input,
                                torch::stable::Tensor& output_q,
                                torch::stable::Tensor& output_s,
                                int64_t group_size, double eps, double min_8bit,
                                double max_8bit, bool scale_ue8m0 = false);
```
**EN:** Core quantization function shared by both FP8 and INT8 variants. Parameters:
- `input`: Input tensor to quantize
- `output_q`: Quantized output (8-bit)
- `output_s`: Scaling factors per group
- `group_size`: Number of elements per quantization group
- `eps`: Epsilon for numerical stability in scale computation
- `min_8bit`, `max_8bit`: Clamp range for quantized values
- `scale_ue8m0`: If true, uses UE8M0 (power-of-2) scale format

**CN:** FP8 和 INT8 变体共享的核心量化函数。参数：
- `input`：要量化的输入张量
- `output_q`：量化输出（8 位）
- `output_s`：每组的缩放因子
- `group_size`：每个量化组的元素数量
- `eps`：缩放计算中的数值稳定性 epsilon
- `min_8bit`、`max_8bit`：量化值的钳制范围
- `scale_ue8m0`：如果为 true，使用 UE8M0（2 的幂次）缩放格式

### Packed Quantization Function / 打包量化函数
```cpp
// Public op: register-resident packed quant for the DeepGEMM Blackwell path.
// Restricted to group_size == 128 and bf16/fp16 input; other configurations
// raise STD_TORCH_CHECK. The legacy shared-memory fallback was removed because
// no production caller (deep_gemm_moe / input_quant_fp8) uses other shapes.
void per_token_group_quant_8bit_packed(const torch::stable::Tensor& input,
                                       torch::stable::Tensor& output_q,
                                       torch::stable::Tensor& output_s_packed,
                                       int64_t group_size, double eps,
                                       double min_8bit, double max_8bit);
```
**EN:** Optimized variant for DeepGEMM on Blackwell architecture. Enforces strict constraints:
- `group_size` must be exactly 128
- Input must be bf16 or fp16 (half-precision)
- Produces UE8M0-packed, TMA-aligned scales in `output_s_packed`
- Uses register-resident kernel (no shared memory) for maximum performance
- Shared-memory fallback removed as no production code requires it

**CN:** 针对 Blackwell 架构上 DeepGEMM 的优化变体。强制严格约束：
- `group_size` 必须恰好为 128
- 输入必须为 bf16 或 fp16（半精度）
- 在 `output_s_packed` 中生成 UE8M0 打包、TMA 对齐的缩放
- 使用寄存器驻留内核（无共享内存）以获得最大性能
- 移除共享内存回退，因为没有生产代码需要它

## Key Concepts / 关键概念

### EN:
- **Unified Interface**: Single header abstracts FP8/INT8 differences, allowing kernel code reuse
- **Dual Implementation Strategy**:
  - General-purpose: `per_token_group_quant_8bit` supports arbitrary group sizes and data types
  - High-performance: `per_token_group_quant_8bit_packed` optimized for group_size=128, bf16/fp16 inputs
- **UE8M0 Scale Format**: Unsigned 8-bit exponent-only format (power-of-2 scales) required by TMA-based matrix engines on Blackwell
- **TMA Alignment**: Tensor Memory Accelerator alignment requirements for efficient Blackwell tensor core operations
- **Production-Oriented Design**: Removes unused fallback paths based on actual caller requirements (deep_gemm_moe, input_quant_fp8)

### CN:
- **统一接口**：单个头文件抽象 FP8/INT8 差异，允许内核代码重用
- **双重实现策略**：
  - 通用：`per_token_group_quant_8bit` 支持任意分组大小和数据类型
  - 高性能：`per_token_group_quant_8bit_packed` 针对 group_size=128、bf16/fp16 输入优化
- **UE8M0 缩放格式**：无符号 8 位仅指数格式（2 的幂次缩放），Blackwell 上基于 TMA 的矩阵引擎所需
- **TMA 对齐**：张量内存加速器对齐要求，用于高效的 Blackwell 张量核心操作
- **面向生产的设计**：根据实际调用者需求（deep_gemm_moe、input_quant_fp8）移除未使用的回退路径

## Dependencies / 依赖关系

### EN:
- **PyTorch Stable ABI**: `torch/csrc/stable/tensor.h` provides stable tensor interface across PyTorch versions
- **Implementations**:
  - `csrc/libtorch_stable/quantization/w8a8/fp8/per_token_group_quant.cu` - contains actual kernel implementations
  - `csrc/libtorch_stable/quantization/w8a8/int8/per_token_group_quant.cu` - thin INT8 wrapper
- **Callers**:
  - Registered in `torch_bindings.cpp` as `torch.ops._C.per_token_group_fp8_quant`, `per_token_group_quant_int8`, etc.
  - Used by DeepGEMM MoE kernels and FP8 input quantization paths

### CN:
- **PyTorch 稳定 ABI**：`torch/csrc/stable/tensor.h` 提供跨 PyTorch 版本的稳定张量接口
- **实现**：
  - `csrc/libtorch_stable/quantization/w8a8/fp8/per_token_group_quant.cu` - 包含实际内核实现
  - `csrc/libtorch_stable/quantization/w8a8/int8/per_token_group_quant.cu` - INT8 轻量级包装器
- **调用者**：
  - 在 `torch_bindings.cpp` 中注册为 `torch.ops._C.per_token_group_fp8_quant`、`per_token_group_quant_int8` 等
  - 由 DeepGEMM MoE 内核和 FP8 输入量化路径使用
