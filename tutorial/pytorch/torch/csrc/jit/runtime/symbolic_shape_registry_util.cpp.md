# symbolic_shape_registry_util.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/symbolic_shape_registry_util.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#include <torch/csrc/jit/runtime/symbolic_shape_registry_util.h>

namespace torch::jit {

const OperatorMap<std::string>& get_tensorexpr_elementwise_set() {
  // clang-format off
 static const OperatorMap<std::string> tensorexpr_elementwise_set{
      {"aten::add.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor", "unary"},
      {"aten::_cast_Float(Tensor self, bool non_blocking) -> Tensor", "unary"},
      {"aten::sub.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> Tensor", "unary"},
      {"aten::mul.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::div.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/symbolic_shape_registry_util.h. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem. This chunk defines `get_tensorexpr_elementwise_set`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/symbolic_shape_registry_util.h。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。 这一段定义了 `get_tensorexpr_elementwise_set`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 13-24
```cpp
      {"aten::eq.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::ne.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::ge.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::gt.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::le.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::lt.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::pow.Tensor_Scalar(Tensor self, Scalar exponent) -> Tensor", "unary"},
      {"aten::clamp(Tensor self, Scalar? min=None, Scalar? max=None) -> Tensor", "unary"},
      {"aten::to.dtype(Tensor self, ScalarType dtype, bool non_blocking=False, bool copy=False, MemoryFormat? memory_format=None) -> Tensor", "unary"},
      {"aten::to.device(Tensor self, Device device, ScalarType dtype, bool non_blocking=False, bool copy=False, MemoryFormat? memory_format=None) -> Tensor", "unary"},
      {"aten::to.dtype_layout(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, bool non_blocking=False, bool copy=False, MemoryFormat? memory_format=None) -> Tensor", "unary"},
      {"aten::to.prim_Device(Tensor(a) self, Device? device, int? dtype=None, bool non_blocking=False, bool copy=False) -> Tensor(a|b)", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 25-36
```cpp
      {"aten::to.prim_dtype(Tensor(a) self, int? dtype=None, bool non_blocking=False, bool copy=False) -> Tensor(a|b)", "unary"},
      {"aten::_autocast_to_reduced_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled, ScalarType cuda_dtype, ScalarType cpu_dtype) -> Tensor(a)", "unary"},
      {"aten::_autocast_to_full_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled) -> Tensor(a)", "unary"},
      {"aten::isnan(Tensor self) -> Tensor", "unary"},
      {"aten::lgamma(Tensor self) -> Tensor", "unary"},
      {"aten::log10(Tensor self) -> Tensor", "unary"},
      {"aten::log(Tensor self) -> Tensor", "unary"},
      {"aten::log2(Tensor self) -> Tensor", "unary"},
      {"aten::log1p(Tensor self) -> Tensor", "unary"},
      {"aten::exp(Tensor self) -> Tensor", "unary"},
      {"aten::erf(Tensor self) -> Tensor", "unary"},
      {"aten::erfc(Tensor self) -> Tensor", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 37-48
```cpp
      // TODO: uncomment when we properly support pow
      // "aten::pow.Tensor_Tensor(Tensor self, Tensor exponent) -> Tensor",
      // "aten::pow.Scalar(Scalar self, Tensor exponent) -> Tensor",
      // TODO: support clamp_min, clamp_max
      // "aten::masked_fill.Scalar(Tensor self, Tensor mask, Scalar value) -> Tensor",
      // "aten::masked_fill.Tensor(Tensor self, Tensor mask, Tensor value) -> Tensor", TODO: requires 0-dim Tensor
      // "aten::remainder.Scalar(Tensor self, Scalar other) -> Tensor",
      // TODO: uncomment once we can handle rand+broadcasts
      // "aten::rand_like(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None, bool? pin_memory=None, MemoryFormat? memory_format=None) -> Tensor",
      {"aten::fmod.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::cos(Tensor self) -> Tensor", "unary"},
      {"aten::sin(Tensor self) -> Tensor", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 49-60
```cpp
      {"aten::tan(Tensor self) -> Tensor", "unary"},
      {"aten::acos(Tensor self) -> Tensor", "unary"},
      {"aten::asin(Tensor self) -> Tensor", "unary"},
      {"aten::atan(Tensor self) -> Tensor", "unary"},
      {"aten::cosh(Tensor self) -> Tensor", "unary"},
      {"aten::sinh(Tensor self) -> Tensor", "unary"},
      {"aten::tanh(Tensor self) -> Tensor", "unary"},
      {"aten::hardtanh(Tensor self, Scalar min_val=-1, Scalar max_val=1) -> Tensor", "unary"},
      {"aten::hardsigmoid(Tensor self) -> Tensor", "unary"},
      {"aten::hardswish(Tensor self) -> Tensor", "unary"},
      {"aten::hardshrink(Tensor self, Scalar lambd=0.5) -> Tensor", "unary"},
      {"aten::sqrt(Tensor self) -> Tensor", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 61-72
```cpp
      {"aten::rsqrt(Tensor self) -> Tensor", "unary"},
      {"aten::abs(Tensor self) -> Tensor", "unary"},
      {"aten::floor(Tensor self) -> Tensor", "unary"},
      {"aten::ceil(Tensor self) -> Tensor", "unary"},
      {"aten::round(Tensor self) -> Tensor", "unary"},
      {"aten::trunc(Tensor self) -> Tensor", "unary"},
      {"aten::threshold(Tensor self, Scalar threshold, Scalar value) -> Tensor", "unary"},
      {"aten::sigmoid(Tensor self) -> Tensor", "unary"},
      {"aten::relu(Tensor self) -> Tensor", "unary"},
      {"aten::leaky_relu(Tensor self, Scalar negative_slope=0.01) -> Tensor", "unary"},
      {"aten::softplus(Tensor self, Scalar beta=1, Scalar threshold=20) -> Tensor", "unary"},
      {"aten::mish(Tensor self) -> Tensor", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 73-84
```cpp
      {"aten::elu(Tensor self, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1) -> Tensor", "unary"},
      {"aten::relu6(Tensor self) -> Tensor", "unary"},
      {"aten::gelu(Tensor self, *, str approximate='none') -> Tensor", "unary"},
      {"aten::silu(Tensor self) -> Tensor", "unary"},
      {"aten::neg(Tensor self) -> Tensor", "unary"},
      {"aten::reciprocal(Tensor self) -> Tensor", "unary"},
      {"aten::expm1(Tensor self) -> Tensor", "unary"},
      {"aten::frac(Tensor self) -> Tensor", "unary"},
      {"aten::__and__.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::__or__.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::__xor__.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::__lshift__.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 85-96
```cpp
      {"aten::__rshift__.Scalar(Tensor self, Scalar other) -> Tensor", "unary"},
      {"aten::where.Scalar(Tensor condition, Scalar self, Scalar other) -> Tensor", "unary"},
      {"aten::add.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor", "broadcast"},
      {"aten::where.ScalarOther(Tensor condition, Tensor self, Scalar other) -> Tensor", "broadcast"},
      {"aten::type_as(Tensor self, Tensor other) -> Tensor", "unary"},
      {"aten::sub.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> Tensor", "broadcast"},
      {"aten::mul.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::div.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::eq.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::ne.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::ge.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::gt.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 97-108
```cpp
      {"aten::le.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::lt.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::lerp.Scalar(Tensor self, Tensor end, Scalar weight) -> Tensor", "broadcast"},
      {"aten::fmod.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::atan2(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::remainder.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::__and__.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::__or__.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::__xor__.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::__lshift__.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::__rshift__.Tensor(Tensor self, Tensor other) -> Tensor", "broadcast"},
      // TODO: enable other min/max variants, operators that can be both
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

### Lines 109-120
```cpp
      // elementwise or reductions:
      {"aten::min.other(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::max.other(Tensor self, Tensor other) -> Tensor", "broadcast"},
      {"aten::lerp.Tensor(Tensor self, Tensor end, Tensor weight) -> Tensor", "broadcast_three"},
      {"aten::addcmul(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> Tensor", "broadcast_three"},
      {"aten::where.self(Tensor condition, Tensor self, Tensor other) -> Tensor", "broadcast_three"},
      {"aten::where.ScalarSelf(Tensor condition, Scalar self, Tensor other) -> Tensor", "broadcast_one_three"},
      // TODO: enable slice, shape inference is not implemented for this op yet
  };
  // clang-format on
  return tensorexpr_elementwise_set;
}
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 122-122
```cpp
} // namespace torch::jit
```
- **EN**: This chunk continues `get_tensorexpr_elementwise_set` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_tensorexpr_elementwise_set`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **get_tensorexpr_elementwise_set**
  - EN: `get_tensorexpr_elementwise_set` is a central symbol declared or implemented in this file.
  - CN: `get_tensorexpr_elementwise_set` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Tensor Expression IR**
  - EN: Represents lowered tensor computations in an optimization-friendly intermediate form.
  - CN: 以便于优化的中间表示来表达降级后的张量计算。
- **CUDA support**
  - EN: Handles GPU-oriented lowering, runtime calls, or emitted kernel code.
  - CN: 处理面向 GPU 的降级、运行时调用或生成的内核代码。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/symbolic_shape_registry_util.h`
- **Primary symbols in this file / 本文件核心符号**: `get_tensorexpr_elementwise_set`
