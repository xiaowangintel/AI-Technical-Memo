# kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/marlin_moe_wna16/kernel.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Marlin-based weight-only MoE GEMM kernels and supporting templates. / 实现基于 Marlin 的仅权重 MoE GEMM 内核及其模板支持代码。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 2-10)
```cpp
#ifndef MARLIN_NAMESPACE_NAME
  #define MARLIN_NAMESPACE_NAME marlin_moe_wna16
#endif

#include "quantization/marlin/marlin.cuh"
#include "quantization/marlin/marlin_dtypes.cuh"
#include "core/scalar_type.hpp"

#define MARLIN_KERNEL_PARAMS                                          \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Low-bit quantization / 低比特量化
- Weight-only quantization / 仅权重量化
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `quantization/marlin/marlin.cuh`, `quantization/marlin/marlin_dtypes.cuh`, `core/scalar_type.hpp`
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
