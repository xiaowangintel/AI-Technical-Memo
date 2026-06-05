# torch_bindings.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/torch_bindings.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Registers CPU custom operators and bindings with PyTorch. / 向 PyTorch 注册 CPU 自定义算子与绑定。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-9)
```cpp
#include "cache.h"
#include "ops.h"
#include "core/registration.h"

#include <torch/library.h>

// Note: overwrite the external definition for sharing same name between
// libraries use different ISAs.
#define TORCH_EXTENSION_NAME _C
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Registration: TORCH_LIBRARY (lines 262-619)
```cpp
TORCH_LIBRARY_EXPAND(TORCH_EXTENSION_NAME, ops) {
  // vLLM custom ops

  ops.def(
      "dynamic_4bit_int_moe("
      "Tensor x, Tensor topk_ids, Tensor topk_weights,"
      "Tensor w13_packed, Tensor w2_packed, int H, int I, int I2,"
      "int group_size, bool apply_router_weight_on_input, int activation_kind"
      ") -> Tensor");

  ops.impl("dynamic_4bit_int_moe", torch::kCPU, &dynamic_4bit_int_moe_cpu);

  // Activation ops

// ...
      "bool no_draft_probs) -> ()",
      &cpu_utils::sample_recovered_tokens_kernel_impl);
}
```
**EN:** This block exposes the low-level implementation as a PyTorch extension entrypoint, so Python code can invoke the kernel through a stable schema.
**CN:** 该代码块把底层实现注册为 PyTorch 扩展入口，使 Python 侧能够通过稳定的 schema 调用这些内核。

### Registration: REGISTER_EXTENSION (lines 621-621)
```cpp
REGISTER_EXTENSION(TORCH_EXTENSION_NAME)
```
**EN:** This block exposes the low-level implementation as a PyTorch extension entrypoint, so Python code can invoke the kernel through a stable schema.
**CN:** 该代码块把底层实现注册为 PyTorch 扩展入口，使 Python 侧能够通过稳定的 schema 调用这些内核。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Softplus-based gating / 基于 Softplus 的门控
- Attention computation / 注意力计算
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `core/registration.h`
- **External libraries / 外部库**: C++ standard library or platform support, PyTorch / ATen
- **Runtime coupling / 运行时耦合**: Registered through the PyTorch extension mechanism / 通过 PyTorch 扩展机制注册
