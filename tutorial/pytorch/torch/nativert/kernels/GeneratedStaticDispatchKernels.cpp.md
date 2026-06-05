# GeneratedStaticDispatchKernels.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/GeneratedStaticDispatchKernels.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for GeneratedStaticDispatchKernels, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 GeneratedStaticDispatchKernels 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```cpp
// @generated
// @lint-ignore-every CLANGTIDY HOWTOEVEN
#include <ATen/CPUFunctions.h>
#include <ATen/InferSize.h>
#include <ATen/NativeFunctions.h>
#include <ATen/Parallel.h>
#include <ATen/ScalarOps.h>
#include <ATen/TensorUtils.h>
#include <ATen/cpu/vec/functional.h>
#include <ATen/cpu/vec/vec.h>
#include <ATen/native/EmbeddingBag.h>
#include <ATen/native/Fill.h>
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 13-24
```cpp
#include <ATen/native/IndexingUtils.h>
#include <ATen/native/NonSymbolicBC.h>
#include <ATen/native/Resize.h>
#include <ATen/native/SharedReduceOps.h>
#include <ATen/native/TensorAdvancedIndexing.h>
#include <ATen/native/cpu/SerialStackImpl.h>
#include <ATen/native/layer_norm.h>
#include <ATen/native/quantized/cpu/fbgemm_utils.h>
#include <ATen/native/quantized/cpu/qembeddingbag.h>
#include <ATen/native/quantized/cpu/qembeddingbag_prepack.h>
#include <ATen/quantized/QTensorImpl.h>
#include <ATen/quantized/Quantizer.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `ATen/native/IndexingUtils.h`, `ATen/native/NonSymbolicBC.h`, `ATen/native/Resize.h`, `ATen/native/SharedReduceOps.h`, `ATen/native/TensorAdvancedIndexing.h`, `ATen/native/cpu/SerialStackImpl.h`, `...`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`ATen/native/IndexingUtils.h`, `ATen/native/NonSymbolicBC.h`, `ATen/native/Resize.h`, `ATen/native/SharedReduceOps.h`, `ATen/native/TensorAdvancedIndexing.h`, `ATen/native/cpu/SerialStackImpl.h`, `...`；外部依赖：无。

### Lines 25-37
```cpp
#include <c10/core/ScalarType.h>
#include <c10/core/WrapDimMinimal.h>
#include <c10/util/irange.h>

#include <torch/nativert/kernels/KernelRegistry.h>

#include <iterator>

namespace torch::nativert {

REGISTER_CPU_KERNEL("torch.ops.aten.absolute.default", aten_absolute_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`。

### Lines 38-56
```cpp
    KernelOutput(0) = at::native::absolute(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::absolute_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.angle.default", aten_angle_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::angle(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::angle_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `absolute`, `toTensor`, `fastResizeToZero`, `absolute_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `absolute`, `toTensor`, `fastResizeToZero`, `absolute_out`, `KernelInput`, `...`。

### Lines 57-70
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.sgn.default", aten_sgn_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::sgn(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sgn_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.acos.default", aten_acos_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sgn`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sgn`, `fastResizeToZero`, `...`。

### Lines 71-89
```cpp
    KernelOutput(0) = at::cpu::acos(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::acos_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.arccos.default", aten_arccos_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::arccos(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arccos_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `acos`, `toTensor`, `fastResizeToZero`, `acos_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `acos`, `toTensor`, `fastResizeToZero`, `acos_out`, `KernelInput`, `...`。

### Lines 90-102
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.add.Tensor", aten_add_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  const auto alpha = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::add(self, other, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::add_out(out, self, other, alpha);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `add`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `add`, `...`。

### Lines 103-114
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.add.Scalar", aten_add_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  const auto alpha = KernelInput(2).toScalar();
  if (auto& out = KernelOutput(0); out.isNone()) {
    out = create_empty_from(self);
  }
  auto& out_t = KernelOutput(0).toTensor();
  fastResizeToZero(out_t);
  at::add_out(out_t, self, other, alpha);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `create_empty_from`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `create_empty_from`, `...`。

### Lines 115-127
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten._add_relu.Tensor", aten__add_relu_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  const auto alpha = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::add_relu(self, other, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::add_relu_out(self, other, alpha, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `add_relu`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `add_relu`, `...`。

### Lines 128-142
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.addmv.default", aten_addmv_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& mat = KernelInput(1).toTensor();
  const auto& vec = KernelInput(2).toTensor();
  const auto beta = KernelInput(3).toScalar();
  const auto alpha = KernelInput(4).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::addmv(self, mat, vec, beta, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::addmv_out(out, self, mat, vec, beta, alpha);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `addmv`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `addmv`, `...`。

### Lines 143-157
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.addr.default", aten_addr_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& vec1 = KernelInput(1).toTensor();
  const auto& vec2 = KernelInput(2).toTensor();
  const auto beta = KernelInput(3).toScalar();
  const auto alpha = KernelInput(4).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::addr(self, vec1, vec2, beta, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::addr_out(self, vec1, vec2, beta, alpha, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `addr`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `addr`, `...`。

### Lines 158-170
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.all.dim", aten_all_dim, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto keepdim = KernelInput(2).toBool();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::all(self, dim, keepdim);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::all_out(out, self, dim, keepdim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 171-183
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.any.dim", aten_any_dim, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto keepdim = KernelInput(2).toBool();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::any(self, dim, keepdim);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::any_out(out, self, dim, keepdim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 184-196
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.argmax.default", aten_argmax_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toOptional<int64_t>();
  const auto keepdim = KernelInput(2).toBool();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::argmax(self, dim, keepdim);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::argmax_out(out, self, dim, keepdim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `argmax`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `argmax`, `...`。

### Lines 197-210
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.acosh.default", aten_acosh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::acosh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::acosh_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.asinh.default", aten_asinh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `acosh`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `acosh`, `fastResizeToZero`, `...`。

### Lines 211-229
```cpp
    KernelOutput(0) = at::cpu::asinh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::asinh_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.arcsinh.default", aten_arcsinh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::arcsinh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arcsinh_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `asinh`, `toTensor`, `fastResizeToZero`, `asinh_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `asinh`, `toTensor`, `fastResizeToZero`, `asinh_out`, `KernelInput`, `...`。

### Lines 230-243
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.atanh.default", aten_atanh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::atanh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::atanh_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.arctanh.default", aten_arctanh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `atanh`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `atanh`, `fastResizeToZero`, `...`。

### Lines 244-262
```cpp
    KernelOutput(0) = at::native::arctanh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arctanh_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.asin.default", aten_asin_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::asin(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::asin_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `arctanh`, `toTensor`, `fastResizeToZero`, `arctanh_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `arctanh`, `toTensor`, `fastResizeToZero`, `arctanh_out`, `KernelInput`, `...`。

### Lines 263-276
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.arcsin.default", aten_arcsin_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::arcsin(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arcsin_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.atan.default", aten_atan_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `arcsin`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `arcsin`, `fastResizeToZero`, `...`。

### Lines 277-295
```cpp
    KernelOutput(0) = at::cpu::atan(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::atan_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.arctan.default", aten_arctan_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::arctan(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arctan_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `atan`, `toTensor`, `fastResizeToZero`, `atan_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `atan`, `toTensor`, `fastResizeToZero`, `atan_out`, `KernelInput`, `...`。

### Lines 296-310
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.baddbmm.default", aten_baddbmm_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& batch1 = KernelInput(1).toTensor();
  const auto& batch2 = KernelInput(2).toTensor();
  const auto beta = KernelInput(3).toScalar();
  const auto alpha = KernelInput(4).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::baddbmm(self, batch1, batch2, beta, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::baddbmm_out(out, self, batch1, batch2, beta, alpha);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `baddbmm`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `baddbmm`, `...`。

### Lines 311-324
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_not.default",
    aten_bitwise_not_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_not(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_not_out(out, self);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_not`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_not`, `fastResizeToZero`, `...`。

### Lines 325-336
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.copysign.Tensor", aten_copysign_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::copysign(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::copysign_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `copysign`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `copysign`, `fastResizeToZero`, `...`。

### Lines 337-350
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logical_not.default",
    aten_logical_not_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::logical_not(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::logical_not_out(self, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_not`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_not`, `fastResizeToZero`, `...`。

### Lines 351-365
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logical_xor.default",
    aten_logical_xor_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::logical_xor(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::logical_xor_out(self, other, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_xor`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_xor`, `fastResizeToZero`, `...`。

### Lines 366-380
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logical_and.default",
    aten_logical_and_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::logical_and(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::logical_and_out(self, other, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_and`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_and`, `fastResizeToZero`, `...`。

### Lines 381-395
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logical_or.default",
    aten_logical_or_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::logical_or(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::logical_or_out(self, other, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_or`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logical_or`, `fastResizeToZero`, `...`。

### Lines 396-411
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.ceil.default", aten_ceil_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::ceil(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::ceil_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.clamp.default", aten_clamp_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto min = KernelInput(1).toOptional<at::Scalar>();
  const auto max = KernelInput(2).toOptional<at::Scalar>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `ceil`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `ceil`, `fastResizeToZero`, `...`。

### Lines 412-424
```cpp
    KernelOutput(0) = at::cpu::clamp(self, min, max);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::clamp_out(out, self, min, max);
})

REGISTER_CPU_KERNEL("torch.ops.aten.clamp.Tensor", aten_clamp_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto min = KernelInput(1).toOptional<at::Tensor>();
  const auto max = KernelInput(2).toOptional<at::Tensor>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `clamp`, `toTensor`, `fastResizeToZero`, `clamp_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `clamp`, `toTensor`, `fastResizeToZero`, `clamp_out`, `KernelInput`, `...`。

### Lines 425-439
```cpp
    KernelOutput(0) = at::cpu::clamp(self, min, max);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::clamp_out(out, self, min, max);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.clamp_max.default",
    aten_clamp_max_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto max = KernelInput(1).toScalar();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `clamp`, `toTensor`, `fastResizeToZero`, `clamp_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `clamp`, `toTensor`, `fastResizeToZero`, `clamp_out`, `KernelInput`, `...`。

### Lines 440-451
```cpp
        KernelOutput(0) = at::cpu::clamp_max(self, max);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::clamp_max_out(out, self, max);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.clamp_max.Tensor", aten_clamp_max_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& max = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `clamp_max`, `toTensor`, `fastResizeToZero`, `clamp_max_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `clamp_max`, `toTensor`, `fastResizeToZero`, `clamp_max_out`, `KernelInput`, `...`。

### Lines 452-464
```cpp
    KernelOutput(0) = at::cpu::clamp_max(self, max);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::clamp_max_out(out, self, max);
})

REGISTER_CPU_KERNEL("torch.ops.aten.clip.default", aten_clip_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto min = KernelInput(1).toOptional<at::Scalar>();
  const auto max = KernelInput(2).toOptional<at::Scalar>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `clamp_max`, `toTensor`, `fastResizeToZero`, `clamp_max_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `clamp_max`, `toTensor`, `fastResizeToZero`, `clamp_max_out`, `KernelInput`, `...`。

### Lines 465-476
```cpp
    KernelOutput(0) = at::native::clip(self, min, max);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::clip_out(self, min, max, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.complex.default", aten_complex_default, {
  const auto& real = KernelInput(0).toTensor();
  const auto& imag = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `clip`, `toTensor`, `fastResizeToZero`, `clip_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `clip`, `toTensor`, `fastResizeToZero`, `clip_out`, `KernelInput`, `...`。

### Lines 477-488
```cpp
    KernelOutput(0) = at::native::complex(real, imag);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::complex_out(real, imag, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.polar.default", aten_polar_default, {
  const auto& abs = KernelInput(0).toTensor();
  const auto& angle = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `complex`, `toTensor`, `fastResizeToZero`, `complex_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `complex`, `toTensor`, `fastResizeToZero`, `complex_out`, `KernelInput`, `...`。

### Lines 489-507
```cpp
    KernelOutput(0) = at::native::polar(abs, angle);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::polar_out(abs, angle, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.cos.default", aten_cos_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::cos(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::cos_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `polar`, `toTensor`, `fastResizeToZero`, `polar_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `polar`, `toTensor`, `fastResizeToZero`, `polar_out`, `KernelInput`, `...`。

### Lines 508-523
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.cosh.default", aten_cosh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::cosh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::cosh_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.cumprod.default", aten_cumprod_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto dtype = KernelInput(2).toOptional<at::ScalarType>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `cosh`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `cosh`, `fastResizeToZero`, `...`。

### Lines 524-538
```cpp
    KernelOutput(0) = at::cpu::cumprod(self, dim, dtype);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::cumprod_out(out, self, dim, dtype);
})

REGISTER_CPU_KERNEL("torch.ops.aten.diff.default", aten_diff_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto n = KernelInput(1).toInt();
  const auto dim = KernelInput(2).toInt();
  const auto prepend = KernelInput(3).toOptional<at::Tensor>();
  const auto append = KernelInput(4).toOptional<at::Tensor>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `cumprod`, `toTensor`, `fastResizeToZero`, `cumprod_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `cumprod`, `toTensor`, `fastResizeToZero`, `cumprod_out`, `KernelInput`, `...`。

### Lines 539-550
```cpp
    KernelOutput(0) = at::native::diff(self, n, dim, prepend, append);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::diff_out(self, n, dim, prepend, append, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.div.Tensor", aten_div_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `diff`, `toTensor`, `fastResizeToZero`, `diff_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `diff`, `toTensor`, `fastResizeToZero`, `diff_out`, `KernelInput`, `...`。

### Lines 551-563
```cpp
    KernelOutput(0) = at::cpu::div(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::div_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.div.Tensor_mode", aten_div_Tensor_mode, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  const auto rounding_mode = KernelInput(2).toOptional<std::string_view>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `div`, `toTensor`, `fastResizeToZero`, `div_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `div`, `toTensor`, `fastResizeToZero`, `div_out`, `KernelInput`, `...`。

### Lines 564-575
```cpp
    KernelOutput(0) = at::cpu::div(self, other, rounding_mode);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::div_out(out, self, other, rounding_mode);
})

REGISTER_CPU_KERNEL("torch.ops.aten.divide.Tensor", aten_divide_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `div`, `toTensor`, `fastResizeToZero`, `div_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `div`, `toTensor`, `fastResizeToZero`, `div_out`, `KernelInput`, `...`。

### Lines 576-590
```cpp
    KernelOutput(0) = at::native::divide(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::divide_out(self, other, out);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.true_divide.Tensor",
    aten_true_divide_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `divide`, `toTensor`, `fastResizeToZero`, `divide_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `divide`, `toTensor`, `fastResizeToZero`, `divide_out`, `KernelInput`, `...`。

### Lines 591-602
```cpp
        KernelOutput(0) = at::native::true_divide(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::true_divide_out(self, other, out);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.dot.default", aten_dot_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& tensor = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `true_divide`, `toTensor`, `fastResizeToZero`, `true_divide_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `true_divide`, `toTensor`, `fastResizeToZero`, `true_divide_out`, `KernelInput`, `...`。

### Lines 603-614
```cpp
    KernelOutput(0) = at::native::dot(self, tensor);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::dot_out(self, tensor, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.vdot.default", aten_vdot_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `dot`, `toTensor`, `fastResizeToZero`, `dot_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `dot`, `toTensor`, `fastResizeToZero`, `dot_out`, `KernelInput`, `...`。

### Lines 615-633
```cpp
    KernelOutput(0) = at::native::vdot(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::vdot_out(self, other, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.erf.default", aten_erf_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::erf(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::erf_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `vdot`, `toTensor`, `fastResizeToZero`, `vdot_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `vdot`, `toTensor`, `fastResizeToZero`, `vdot_out`, `KernelInput`, `...`。

### Lines 634-647
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.erfc.default", aten_erfc_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::erfc(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::erfc_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.exp.default", aten_exp_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `erfc`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `erfc`, `fastResizeToZero`, `...`。

### Lines 648-666
```cpp
    KernelOutput(0) = at::cpu::exp(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::exp_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.exp2.default", aten_exp2_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::exp2(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::exp2_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `exp`, `toTensor`, `fastResizeToZero`, `exp_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `exp`, `toTensor`, `fastResizeToZero`, `exp_out`, `KernelInput`, `...`。

### Lines 667-680
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.expm1.default", aten_expm1_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::expm1(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::expm1_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.floor.default", aten_floor_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `expm1`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `expm1`, `fastResizeToZero`, `...`。

### Lines 681-699
```cpp
    KernelOutput(0) = at::cpu::floor(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::floor_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.frac.default", aten_frac_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::frac(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::frac_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `floor`, `toTensor`, `fastResizeToZero`, `floor_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `floor`, `toTensor`, `fastResizeToZero`, `floor_out`, `KernelInput`, `...`。

### Lines 700-711
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.gcd.default", aten_gcd_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::gcd(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::gcd_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `gcd`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `gcd`, `fastResizeToZero`, `...`。

### Lines 712-723
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.lcm.default", aten_lcm_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::lcm(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lcm_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `lcm`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `lcm`, `fastResizeToZero`, `...`。

### Lines 724-740
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.index_copy.default",
    aten_index_copy_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto& index = KernelInput(2).toTensor();
      const auto& source = KernelInput(3).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::index_copy(self, dim, index, source);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::index_copy_out(out, self, dim, index, source);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `index_copy`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `index_copy`, `...`。

### Lines 741-758
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.isin.Tensor_Tensor",
    aten_isin_Tensor_Tensor,
    {
      const auto& elements = KernelInput(0).toTensor();
      const auto& test_elements = KernelInput(1).toTensor();
      const auto assume_unique = KernelInput(2).toBool();
      const auto invert = KernelInput(3).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::isin(elements, test_elements, assume_unique, invert);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::isin_out(out, elements, test_elements, assume_unique, invert);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `isin`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `isin`, `...`。

### Lines 759-776
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.isin.Tensor_Scalar",
    aten_isin_Tensor_Scalar,
    {
      const auto& elements = KernelInput(0).toTensor();
      const auto test_element = KernelInput(1).toScalar();
      const auto assume_unique = KernelInput(2).toBool();
      const auto invert = KernelInput(3).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::isin(elements, test_element, assume_unique, invert);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::isin_out(out, elements, test_element, assume_unique, invert);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 777-794
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.isin.Scalar_Tensor",
    aten_isin_Scalar_Tensor,
    {
      const auto element = KernelInput(0).toScalar();
      const auto& test_elements = KernelInput(1).toTensor();
      const auto assume_unique = KernelInput(2).toBool();
      const auto invert = KernelInput(3).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::isin(element, test_elements, assume_unique, invert);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::isin_out(out, element, test_elements, assume_unique, invert);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toScalar`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toScalar`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 795-806
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.kron.default", aten_kron_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::kron(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::kron_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `kron`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `kron`, `fastResizeToZero`, `...`。

### Lines 807-818
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.ldexp.Tensor", aten_ldexp_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::ldexp(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::ldexp_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `ldexp`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `ldexp`, `fastResizeToZero`, `...`。

### Lines 819-832
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.log10.default", aten_log10_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::log10(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::log10_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.log1p.default", aten_log1p_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `log10`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `log10`, `fastResizeToZero`, `...`。

### Lines 833-851
```cpp
    KernelOutput(0) = at::cpu::log1p(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::log1p_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.log2.default", aten_log2_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::log2(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::log2_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `log1p`, `toTensor`, `fastResizeToZero`, `log1p_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `log1p`, `toTensor`, `fastResizeToZero`, `log1p_out`, `KernelInput`, `...`。

### Lines 852-866
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logaddexp.default",
    aten_logaddexp_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::logaddexp(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::logaddexp_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logaddexp`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logaddexp`, `fastResizeToZero`, `...`。

### Lines 867-881
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logaddexp2.default",
    aten_logaddexp2_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::logaddexp2(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::logaddexp2_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logaddexp2`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `logaddexp2`, `fastResizeToZero`, `...`。

### Lines 882-893
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.xlogy.Tensor", aten_xlogy_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::xlogy(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::xlogy_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `xlogy`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `xlogy`, `fastResizeToZero`, `...`。

### Lines 894-909
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten._log_softmax.default",
    aten__log_softmax_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto half_to_float = KernelInput(2).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::_log_softmax(self, dim, half_to_float);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::_log_softmax_out(out, self, dim, half_to_float);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 910-924
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten._logcumsumexp.default",
    aten__logcumsumexp_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::_logcumsumexp_cpu(self, dim);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::_logcumsumexp_out_cpu(self, dim, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `_logcumsumexp_cpu`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `_logcumsumexp_cpu`, `...`。

### Lines 925-939
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.logcumsumexp.default",
    aten_logcumsumexp_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::logcumsumexp(self, dim);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::logcumsumexp_out(self, dim, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `logcumsumexp`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `logcumsumexp`, `...`。

### Lines 940-954
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.matrix_power.default",
    aten_matrix_power_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto n = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::matrix_power(self, n);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::matrix_power_out(self, n, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `matrix_power`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `matrix_power`, `...`。

### Lines 955-966
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.mm.default", aten_mm_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& mat2 = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::mm(self, mat2);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::mm_out(out, self, mat2);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `mm`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `mm`, `fastResizeToZero`, `...`。

### Lines 967-978
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.multiply.Tensor", aten_multiply_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::multiply(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::multiply_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `multiply`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `multiply`, `fastResizeToZero`, `...`。

### Lines 979-990
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.mv.default", aten_mv_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& vec = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::mv(self, vec);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::mv_out(self, vec, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `mv`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `mv`, `fastResizeToZero`, `...`。

### Lines 991-1002
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.mvlgamma.default", aten_mvlgamma_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto p = KernelInput(1).toInt();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::mvlgamma(self, p);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::mvlgamma_out(self, p, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `mvlgamma`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `mvlgamma`, `...`。

### Lines 1003-1016
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.rad2deg.default", aten_rad2deg_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::rad2deg(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::rad2deg_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.deg2rad.default", aten_deg2rad_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `rad2deg`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `rad2deg`, `fastResizeToZero`, `...`。

### Lines 1017-1030
```cpp
    KernelOutput(0) = at::native::deg2rad(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::deg2rad_out(self, out);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.reciprocal.default",
    aten_reciprocal_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `deg2rad`, `toTensor`, `fastResizeToZero`, `deg2rad_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `deg2rad`, `toTensor`, `fastResizeToZero`, `deg2rad_out`, `KernelInput`, `...`。

### Lines 1031-1049
```cpp
        KernelOutput(0) = at::cpu::reciprocal(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::reciprocal_out(out, self);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.neg.default", aten_neg_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::neg(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::neg_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `reciprocal`, `toTensor`, `fastResizeToZero`, `reciprocal_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `reciprocal`, `toTensor`, `fastResizeToZero`, `reciprocal_out`, `KernelInput`, `...`。

### Lines 1050-1063
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.negative.default", aten_negative_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::negative(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::negative_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.round.default", aten_round_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `negative`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `negative`, `fastResizeToZero`, `...`。

### Lines 1064-1075
```cpp
    KernelOutput(0) = at::cpu::round(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::round_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.round.decimals", aten_round_decimals, {
  const auto& self = KernelInput(0).toTensor();
  const auto decimals = KernelInput(1).toInt();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `round`, `toTensor`, `fastResizeToZero`, `round_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `round`, `toTensor`, `fastResizeToZero`, `round_out`, `KernelInput`, `...`。

### Lines 1076-1087
```cpp
    KernelOutput(0) = at::cpu::round(self, decimals);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::round_out(out, self, decimals);
})

REGISTER_CPU_KERNEL("torch.ops.aten.gelu.default", aten_gelu_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto approximate = KernelInput(1).toStringView();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `round`, `toTensor`, `fastResizeToZero`, `round_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `round`, `toTensor`, `fastResizeToZero`, `round_out`, `KernelInput`, `...`。

### Lines 1088-1102
```cpp
    KernelOutput(0) = at::cpu::gelu(self, approximate);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::gelu_out(out, self, approximate);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.hardshrink.default",
    aten_hardshrink_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto lambd = KernelInput(1).toScalar();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `gelu`, `toTensor`, `fastResizeToZero`, `gelu_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `gelu`, `toTensor`, `fastResizeToZero`, `gelu_out`, `KernelInput`, `...`。

### Lines 1103-1118
```cpp
        KernelOutput(0) = at::cpu::hardshrink(self, lambd);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::hardshrink_out(out, self, lambd);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.hardshrink_backward.default",
    aten_hardshrink_backward_default,
    {
      const auto& grad_out = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      const auto lambd = KernelInput(2).toScalar();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `hardshrink`, `toTensor`, `fastResizeToZero`, `hardshrink_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `hardshrink`, `toTensor`, `fastResizeToZero`, `hardshrink_out`, `KernelInput`, `...`。

### Lines 1119-1137
```cpp
        KernelOutput(0) = at::cpu::hardshrink_backward(grad_out, self, lambd);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::hardshrink_backward_out(grad_input, grad_out, self, lambd);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.rsqrt.default", aten_rsqrt_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::rsqrt(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::rsqrt_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `hardshrink_backward`, `toTensor`, `fastResizeToZero`, `hardshrink_backward_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `hardshrink_backward`, `toTensor`, `fastResizeToZero`, `hardshrink_backward_out`, `KernelInput`, `...`。

### Lines 1138-1155
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.silu.default", aten_silu_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::silu(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::silu_out(out, self);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.silu_backward.default",
    aten_silu_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `silu`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `silu`, `fastResizeToZero`, `...`。

### Lines 1156-1174
```cpp
        KernelOutput(0) = at::cpu::silu_backward(grad_output, self);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::silu_backward_out(grad_input, grad_output, self);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.mish.default", aten_mish_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::mish(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::mish_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `silu_backward`, `toTensor`, `fastResizeToZero`, `silu_backward_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `silu_backward`, `toTensor`, `fastResizeToZero`, `silu_backward_out`, `KernelInput`, `...`。

### Lines 1175-1188
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.sigmoid.default", aten_sigmoid_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::sigmoid(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sigmoid_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.sin.default", aten_sin_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sigmoid`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sigmoid`, `fastResizeToZero`, `...`。

### Lines 1189-1207
```cpp
    KernelOutput(0) = at::cpu::sin(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sin_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.sinc.default", aten_sinc_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::sinc(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sinc_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `sin`, `toTensor`, `fastResizeToZero`, `sin_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `sin`, `toTensor`, `fastResizeToZero`, `sin_out`, `KernelInput`, `...`。

### Lines 1208-1223
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.sinh.default", aten_sinh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::sinh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sinh_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten._softmax.default", aten__softmax_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto half_to_float = KernelInput(2).toBool();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sinh`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `sinh`, `fastResizeToZero`, `...`。

### Lines 1224-1242
```cpp
    KernelOutput(0) = at::cpu::_softmax(self, dim, half_to_float);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::_softmax_out(out, self, dim, half_to_float);
})

REGISTER_CPU_KERNEL("torch.ops.aten.sqrt.default", aten_sqrt_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::sqrt(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::sqrt_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `_softmax`, `toTensor`, `fastResizeToZero`, `_softmax_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `_softmax`, `toTensor`, `fastResizeToZero`, `_softmax_out`, `KernelInput`, `...`。

### Lines 1243-1257
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.square.default", aten_square_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::square(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::square_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.prod.default", aten_prod_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dtype = KernelInput(1).toOptional<at::ScalarType>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `square`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `square`, `fastResizeToZero`, `...`。

### Lines 1258-1271
```cpp
    KernelOutput(0) = at::native::prod(self, dtype);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::prod_out(self, dtype, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.prod.dim_int", aten_prod_dim_int, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto keepdim = KernelInput(2).toBool();
  const auto dtype = KernelInput(3).toOptional<at::ScalarType>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `prod`, `toTensor`, `fastResizeToZero`, `prod_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `prod`, `toTensor`, `fastResizeToZero`, `prod_out`, `KernelInput`, `...`。

### Lines 1272-1290
```cpp
    KernelOutput(0) = at::cpu::prod(self, dim, keepdim, dtype);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::prod_out(out, self, dim, keepdim, dtype);
})

REGISTER_CPU_KERNEL("torch.ops.aten.tan.default", aten_tan_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::tan(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::tan_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `prod`, `toTensor`, `fastResizeToZero`, `prod_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `prod`, `toTensor`, `fastResizeToZero`, `prod_out`, `KernelInput`, `...`。

### Lines 1291-1309
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.tanh.default", aten_tanh_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::tanh(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::tanh_out(out, self);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.threshold.default",
    aten_threshold_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto threshold = KernelInput(1).toScalar();
      const auto value = KernelInput(2).toScalar();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `tanh`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `tanh`, `fastResizeToZero`, `...`。

### Lines 1310-1325
```cpp
        KernelOutput(0) = at::cpu::threshold(self, threshold, value);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::threshold_out(out, self, threshold, value);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.threshold_backward.default",
    aten_threshold_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      const auto threshold = KernelInput(2).toScalar();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `threshold`, `toTensor`, `fastResizeToZero`, `threshold_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `threshold`, `toTensor`, `fastResizeToZero`, `threshold_out`, `KernelInput`, `...`。

### Lines 1326-1337
```cpp
        KernelOutput(0) =
            at::cpu::threshold_backward(grad_output, self, threshold);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::threshold_backward_out(grad_input, grad_output, self, threshold);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.trunc.default", aten_trunc_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `threshold_backward`, `toTensor`, `fastResizeToZero`, `threshold_backward_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `threshold_backward`, `toTensor`, `fastResizeToZero`, `threshold_backward_out`, `KernelInput`, `...`。

### Lines 1338-1356
```cpp
    KernelOutput(0) = at::cpu::trunc(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::trunc_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.fix.default", aten_fix_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::fix(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::fix_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `trunc`, `toTensor`, `fastResizeToZero`, `trunc_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `trunc`, `toTensor`, `fastResizeToZero`, `trunc_out`, `KernelInput`, `...`。

### Lines 1357-1371
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.nuclear_norm.default",
    aten_nuclear_norm_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto keepdim = KernelInput(1).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::nuclear_norm(self, keepdim);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::nuclear_norm_out(self, keepdim, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `nuclear_norm`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `nuclear_norm`, `...`。

### Lines 1372-1384
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.subtract.Tensor", aten_subtract_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  const auto alpha = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::subtract(self, other, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::subtract_out(self, other, alpha, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `subtract`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `subtract`, `...`。

### Lines 1385-1399
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.heaviside.default",
    aten_heaviside_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& values = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::heaviside(self, values);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::heaviside_out(out, self, values);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `heaviside`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `heaviside`, `fastResizeToZero`, `...`。

### Lines 1400-1420
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten._addmm_activation.default",
    aten__addmm_activation_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& mat1 = KernelInput(1).toTensor();
      const auto& mat2 = KernelInput(2).toTensor();
      const auto beta = KernelInput(3).toScalar();
      const auto alpha = KernelInput(4).toScalar();
      const auto use_gelu = KernelInput(5).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::_addmm_activation(self, mat1, mat2, beta, alpha, use_gelu);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::_addmm_activation_out(
          out, self, mat1, mat2, beta, alpha, use_gelu);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 1421-1438
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.index_add.default",
    aten_index_add_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto& index = KernelInput(2).toTensor();
      const auto& source = KernelInput(3).toTensor();
      const auto alpha = KernelInput(4).toScalar();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::index_add(self, dim, index, source, alpha);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::index_add_out(out, self, dim, index, source, alpha);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`。

### Lines 1439-1452
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.scatter.src", aten_scatter_src, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto& index = KernelInput(2).toTensor();
  const auto& src = KernelInput(3).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::scatter(self, dim, index, src);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::scatter_out(out, self, dim, index, src);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `scatter`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `scatter`, `...`。

### Lines 1453-1466
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.scatter.value", aten_scatter_value, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto& index = KernelInput(2).toTensor();
  const auto value = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::scatter(self, dim, index, value);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::scatter_out(out, self, dim, index, value);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`。

### Lines 1467-1481
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.scatter.reduce", aten_scatter_reduce, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto& index = KernelInput(2).toTensor();
  const auto& src = KernelInput(3).toTensor();
  const auto reduce = KernelInput(4).toStringView();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::scatter(self, dim, index, src, reduce);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::scatter_out(out, self, dim, index, src, reduce);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toStringView`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toStringView`, `KernelOutput`, `isNone`, `...`。

### Lines 1482-1499
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.scatter.value_reduce",
    aten_scatter_value_reduce,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto& index = KernelInput(2).toTensor();
      const auto value = KernelInput(3).toScalar();
      const auto reduce = KernelInput(4).toStringView();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::scatter(self, dim, index, value, reduce);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::scatter_out(out, self, dim, index, value, reduce);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toScalar`, `toStringView`, `KernelOutput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toScalar`, `toStringView`, `KernelOutput`, `...`。

### Lines 1500-1516
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.scatter_add.default",
    aten_scatter_add_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto& index = KernelInput(2).toTensor();
      const auto& src = KernelInput(3).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::scatter_add(self, dim, index, src);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::scatter_add_out(out, self, dim, index, src);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `scatter_add`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `scatter_add`, `...`。

### Lines 1517-1537
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.scatter_reduce.two",
    aten_scatter_reduce_two,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto dim = KernelInput(1).toInt();
      const auto& index = KernelInput(2).toTensor();
      const auto& src = KernelInput(3).toTensor();
      const auto reduce = KernelInput(4).toStringView();
      const auto include_self = KernelInput(5).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::scatter_reduce(
            self, dim, index, src, reduce, include_self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::scatter_reduce_out(
          out, self, dim, index, src, reduce, include_self);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toStringView`, `toBool`, `KernelOutput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toStringView`, `toBool`, `KernelOutput`, `...`。

### Lines 1538-1549
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.eq.Scalar", aten_eq_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::eq(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::eq_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `eq`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `eq`, `...`。

### Lines 1550-1561
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.eq.Tensor", aten_eq_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::eq(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::eq_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `eq`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `eq`, `fastResizeToZero`, `...`。

### Lines 1562-1576
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_and.Tensor",
    aten_bitwise_and_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_and(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_and_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_and`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_and`, `fastResizeToZero`, `...`。

### Lines 1577-1589
```cpp
// __and__ is the Python dunder alias for bitwise_and
REGISTER_CPU_KERNEL("torch.ops.aten.__and__.Tensor", aten___and___Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::bitwise_and(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::bitwise_and_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_and`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_and`, `fastResizeToZero`, `...`。

### Lines 1590-1604
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_or.Tensor",
    aten_bitwise_or_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_or(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_or_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_or`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_or`, `fastResizeToZero`, `...`。

### Lines 1605-1617
```cpp
// __or__ is the Python dunder alias for bitwise_or
REGISTER_CPU_KERNEL("torch.ops.aten.__or__.Tensor", aten___or___Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::bitwise_or(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::bitwise_or_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_or`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_or`, `fastResizeToZero`, `...`。

### Lines 1618-1630
```cpp
REGISTER_NATIVE_CPU_KERNEL(
    "torch.ops.aten.scalar_tensor.default",
    aten_scalar_tensor_default,
    {
      const auto s = KernelInput(0).toScalar();
      const auto dtype = KernelInput(1).toOptional<at::ScalarType>();
      const auto layout = KernelInput(2).toOptional<at::Layout>();
      const auto device = KernelInput(3).toOptional<at::Device>();
      const auto pin_memory = KernelInput(4).toOptional<bool>();
      KernelOutput(0) =
          at::native::scalar_tensor(s, dtype, layout, device, pin_memory);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toScalar`, `KernelOutput`, `scalar_tensor`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toScalar`, `KernelOutput`, `scalar_tensor`。

### Lines 1631-1645
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_xor.Tensor",
    aten_bitwise_xor_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_xor(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_xor_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_xor`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_xor`, `fastResizeToZero`, `...`。

### Lines 1646-1660
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_left_shift.Tensor",
    aten_bitwise_left_shift_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_left_shift(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_left_shift_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_left_shift`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_left_shift`, `fastResizeToZero`, `...`。

### Lines 1661-1675
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.bitwise_right_shift.Tensor",
    aten_bitwise_right_shift_Tensor,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::bitwise_right_shift(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::bitwise_right_shift_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_right_shift`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `bitwise_right_shift`, `fastResizeToZero`, `...`。

### Lines 1676-1687
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.tril.default", aten_tril_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto diagonal = KernelInput(1).toInt();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::tril(self, diagonal);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::tril_out(out, self, diagonal);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `tril`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `tril`, `...`。

### Lines 1688-1699
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.triu.default", aten_triu_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto diagonal = KernelInput(1).toInt();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::triu(self, diagonal);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::triu_out(out, self, diagonal);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `triu`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `triu`, `...`。

### Lines 1700-1715
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.digamma.default", aten_digamma_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::digamma(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::digamma_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.lerp.Scalar", aten_lerp_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto& end = KernelInput(1).toTensor();
  const auto weight = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `digamma`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `digamma`, `fastResizeToZero`, `...`。

### Lines 1716-1728
```cpp
    KernelOutput(0) = at::cpu::lerp(self, end, weight);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lerp_out(out, self, end, weight);
})

REGISTER_CPU_KERNEL("torch.ops.aten.lerp.Tensor", aten_lerp_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& end = KernelInput(1).toTensor();
  const auto& weight = KernelInput(2).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `lerp`, `toTensor`, `fastResizeToZero`, `lerp_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `lerp`, `toTensor`, `fastResizeToZero`, `lerp_out`, `KernelInput`, `...`。

### Lines 1729-1743
```cpp
    KernelOutput(0) = at::cpu::lerp(self, end, weight);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lerp_out(out, self, end, weight);
})

REGISTER_CPU_KERNEL("torch.ops.aten.addbmm.default", aten_addbmm_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& batch1 = KernelInput(1).toTensor();
  const auto& batch2 = KernelInput(2).toTensor();
  const auto beta = KernelInput(3).toScalar();
  const auto alpha = KernelInput(4).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `lerp`, `toTensor`, `fastResizeToZero`, `lerp_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `lerp`, `toTensor`, `fastResizeToZero`, `lerp_out`, `KernelInput`, `...`。

### Lines 1744-1756
```cpp
    KernelOutput(0) = at::native::addbmm(self, batch1, batch2, beta, alpha);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::addbmm_out(self, batch1, batch2, beta, alpha, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.cross.default", aten_cross_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  const auto dim = KernelInput(2).toOptional<int64_t>();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `addbmm`, `toTensor`, `fastResizeToZero`, `addbmm_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `addbmm`, `toTensor`, `fastResizeToZero`, `addbmm_out`, `KernelInput`, `...`。

### Lines 1757-1768
```cpp
    KernelOutput(0) = at::native::cross(self, other, dim);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::cross_out(self, other, dim, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.ne.Scalar", aten_ne_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `cross`, `toTensor`, `fastResizeToZero`, `cross_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `cross`, `toTensor`, `fastResizeToZero`, `cross_out`, `KernelInput`, `...`。

### Lines 1769-1780
```cpp
    KernelOutput(0) = at::cpu::ne(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::ne_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.ne.Tensor", aten_ne_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `ne`, `toTensor`, `fastResizeToZero`, `ne_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `ne`, `toTensor`, `fastResizeToZero`, `ne_out`, `KernelInput`, `...`。

### Lines 1781-1792
```cpp
    KernelOutput(0) = at::cpu::ne(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::ne_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.ge.Scalar", aten_ge_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `ne`, `toTensor`, `fastResizeToZero`, `ne_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `ne`, `toTensor`, `fastResizeToZero`, `ne_out`, `KernelInput`, `...`。

### Lines 1793-1804
```cpp
    KernelOutput(0) = at::cpu::ge(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::ge_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.ge.Tensor", aten_ge_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `ge`, `toTensor`, `fastResizeToZero`, `ge_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `ge`, `toTensor`, `fastResizeToZero`, `ge_out`, `KernelInput`, `...`。

### Lines 1805-1816
```cpp
    KernelOutput(0) = at::cpu::ge(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::ge_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.le.Scalar", aten_le_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `ge`, `toTensor`, `fastResizeToZero`, `ge_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `ge`, `toTensor`, `fastResizeToZero`, `ge_out`, `KernelInput`, `...`。

### Lines 1817-1828
```cpp
    KernelOutput(0) = at::cpu::le(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::le_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.le.Tensor", aten_le_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `le`, `toTensor`, `fastResizeToZero`, `le_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `le`, `toTensor`, `fastResizeToZero`, `le_out`, `KernelInput`, `...`。

### Lines 1829-1840
```cpp
    KernelOutput(0) = at::cpu::le(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::le_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.gt.Scalar", aten_gt_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `le`, `toTensor`, `fastResizeToZero`, `le_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `le`, `toTensor`, `fastResizeToZero`, `le_out`, `KernelInput`, `...`。

### Lines 1841-1852
```cpp
    KernelOutput(0) = at::cpu::gt(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::gt_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.gt.Tensor", aten_gt_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `gt`, `toTensor`, `fastResizeToZero`, `gt_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `gt`, `toTensor`, `fastResizeToZero`, `gt_out`, `KernelInput`, `...`。

### Lines 1853-1864
```cpp
    KernelOutput(0) = at::cpu::gt(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::gt_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.lt.Scalar", aten_lt_Scalar, {
  const auto& self = KernelInput(0).toTensor();
  const auto other = KernelInput(1).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `gt`, `toTensor`, `fastResizeToZero`, `gt_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `gt`, `toTensor`, `fastResizeToZero`, `gt_out`, `KernelInput`, `...`。

### Lines 1865-1876
```cpp
    KernelOutput(0) = at::cpu::lt(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lt_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.lt.Tensor", aten_lt_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `lt`, `toTensor`, `fastResizeToZero`, `lt_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `lt`, `toTensor`, `fastResizeToZero`, `lt_out`, `KernelInput`, `...`。

### Lines 1877-1888
```cpp
    KernelOutput(0) = at::cpu::lt(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lt_out(out, self, other);
})

REGISTER_CPU_KERNEL("torch.ops.aten.take.default", aten_take_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& index = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `lt`, `toTensor`, `fastResizeToZero`, `lt_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `lt`, `toTensor`, `fastResizeToZero`, `lt_out`, `KernelInput`, `...`。

### Lines 1889-1904
```cpp
    KernelOutput(0) = at::native::take(self, index);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::take_out(self, index, out);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.take_along_dim.default",
    aten_take_along_dim_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& indices = KernelInput(1).toTensor();
      const auto dim = KernelInput(2).toOptional<int64_t>();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `take`, `toTensor`, `fastResizeToZero`, `take_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `take`, `toTensor`, `fastResizeToZero`, `take_out`, `KernelInput`, `...`。

### Lines 1905-1919
```cpp
        KernelOutput(0) = at::native::take_along_dim(self, indices, dim);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::take_along_dim_out(self, indices, dim, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.masked_select.default",
    aten_masked_select_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& mask = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `take_along_dim`, `toTensor`, `fastResizeToZero`, `take_along_dim_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `take_along_dim`, `toTensor`, `fastResizeToZero`, `take_along_dim_out`, `KernelInput`, `...`。

### Lines 1920-1933
```cpp
        KernelOutput(0) = at::native::masked_select_cpu(self, mask);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::masked_select_out_cpu(self, mask, out);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.gather.default", aten_gather_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  const auto& index = KernelInput(2).toTensor();
  const auto sparse_grad = KernelInput(3).toBool();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `masked_select_cpu`, `toTensor`, `fastResizeToZero`, `masked_select_out_cpu`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `masked_select_cpu`, `toTensor`, `fastResizeToZero`, `masked_select_out_cpu`, `KernelInput`, `...`。

### Lines 1934-1947
```cpp
    KernelOutput(0) = at::cpu::gather(self, dim, index, sparse_grad);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::gather_out(out, self, dim, index, sparse_grad);
})

REGISTER_CPU_KERNEL("torch.ops.aten.addcmul.default", aten_addcmul_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& tensor1 = KernelInput(1).toTensor();
  const auto& tensor2 = KernelInput(2).toTensor();
  const auto value = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `gather`, `toTensor`, `fastResizeToZero`, `gather_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `gather`, `toTensor`, `fastResizeToZero`, `gather_out`, `KernelInput`, `...`。

### Lines 1948-1961
```cpp
    KernelOutput(0) = at::cpu::addcmul(self, tensor1, tensor2, value);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::addcmul_out(out, self, tensor1, tensor2, value);
})

REGISTER_CPU_KERNEL("torch.ops.aten.addcdiv.default", aten_addcdiv_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& tensor1 = KernelInput(1).toTensor();
  const auto& tensor2 = KernelInput(2).toTensor();
  const auto value = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `addcmul`, `toTensor`, `fastResizeToZero`, `addcmul_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `addcmul`, `toTensor`, `fastResizeToZero`, `addcmul_out`, `KernelInput`, `...`。

### Lines 1962-1979
```cpp
    KernelOutput(0) = at::cpu::addcdiv(self, tensor1, tensor2, value);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::addcdiv_out(out, self, tensor1, tensor2, value);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_solve_triangular.default",
    aten_linalg_solve_triangular_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& B = KernelInput(1).toTensor();
      const auto upper = KernelInput(2).toBool();
      const auto left = KernelInput(3).toBool();
      const auto unitriangular = KernelInput(4).toBool();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `addcdiv`, `toTensor`, `fastResizeToZero`, `addcdiv_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `addcdiv`, `toTensor`, `fastResizeToZero`, `addcdiv_out`, `KernelInput`, `...`。

### Lines 1980-1997
```cpp
        KernelOutput(0) = at::native::linalg_solve_triangular(
            self, B, upper, left, unitriangular);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_solve_triangular_out(
          self, B, upper, left, unitriangular, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.cholesky_solve.default",
    aten_cholesky_solve_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& input2 = KernelInput(1).toTensor();
      const auto upper = KernelInput(2).toBool();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_solve_triangular`, `toTensor`, `fastResizeToZero`, `linalg_solve_triangular_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_solve_triangular`, `toTensor`, `fastResizeToZero`, `linalg_solve_triangular_out`, `KernelInput`, `...`。

### Lines 1998-2012
```cpp
        KernelOutput(0) = at::native::cholesky_solve(self, input2, upper);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::cholesky_solve_out(self, input2, upper, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.cholesky_inverse.default",
    aten_cholesky_inverse_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto upper = KernelInput(1).toBool();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `cholesky_solve`, `toTensor`, `fastResizeToZero`, `cholesky_solve_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `cholesky_solve`, `toTensor`, `fastResizeToZero`, `cholesky_solve_out`, `KernelInput`, `...`。

### Lines 2013-2024
```cpp
        KernelOutput(0) = at::native::cholesky_inverse(self, upper);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::cholesky_inverse_out(self, upper, out);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.orgqr.default", aten_orgqr_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& input2 = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `cholesky_inverse`, `toTensor`, `fastResizeToZero`, `cholesky_inverse_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `cholesky_inverse`, `toTensor`, `fastResizeToZero`, `cholesky_inverse_out`, `KernelInput`, `...`。

### Lines 2025-2039
```cpp
    KernelOutput(0) = at::native::orgqr(self, input2);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::orgqr_out(self, input2, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.ormqr.default", aten_ormqr_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& input2 = KernelInput(1).toTensor();
  const auto& input3 = KernelInput(2).toTensor();
  const auto left = KernelInput(3).toBool();
  const auto transpose = KernelInput(4).toBool();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `orgqr`, `toTensor`, `fastResizeToZero`, `orgqr_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `orgqr`, `toTensor`, `fastResizeToZero`, `orgqr_out`, `KernelInput`, `...`。

### Lines 2040-2058
```cpp
    KernelOutput(0) = at::native::ormqr(self, input2, input3, left, transpose);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::ormqr_out(self, input2, input3, left, transpose, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.lgamma.default", aten_lgamma_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::lgamma(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::lgamma_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `ormqr`, `toTensor`, `fastResizeToZero`, `ormqr_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `ormqr`, `toTensor`, `fastResizeToZero`, `ormqr_out`, `KernelInput`, `...`。

### Lines 2059-2073
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.polygamma.default",
    aten_polygamma_default,
    {
      const auto n = KernelInput(0).toInt();
      const auto& self = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::polygamma(n, self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::polygamma_out(out, n, self);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toInt`, `toTensor`, `KernelOutput`, `isNone`, `polygamma`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toInt`, `toTensor`, `KernelOutput`, `isNone`, `polygamma`, `...`。

### Lines 2074-2087
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.erfinv.default", aten_erfinv_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::erfinv(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::erfinv_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.i0.default", aten_i0_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `erfinv`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `erfinv`, `fastResizeToZero`, `...`。

### Lines 2088-2106
```cpp
    KernelOutput(0) = at::cpu::i0(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::i0_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.signbit.default", aten_signbit_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::signbit(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::signbit_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `i0`, `toTensor`, `fastResizeToZero`, `i0_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `i0`, `toTensor`, `fastResizeToZero`, `i0_out`, `KernelInput`, `...`。

### Lines 2107-2118
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.atan2.default", aten_atan2_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::atan2(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::atan2_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `atan2`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `atan2`, `fastResizeToZero`, `...`。

### Lines 2119-2130
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.arctan2.default", aten_arctan2_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::arctan2(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::arctan2_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `arctan2`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `arctan2`, `fastResizeToZero`, `...`。

### Lines 2131-2144
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.histc.default", aten_histc_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto bins = KernelInput(1).toInt();
  const auto min = KernelInput(2).toScalar();
  const auto max = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::histogram_histc(self, bins, min, max);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::histogram_histc_out(self, bins, min, max, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toScalar`, `KernelOutput`, `isNone`, `...`。

### Lines 2145-2156
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.fmod.Tensor", aten_fmod_Tensor, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::fmod(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::fmod_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmod`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmod`, `fastResizeToZero`, `...`。

### Lines 2157-2168
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.hypot.default", aten_hypot_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::hypot(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::hypot_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hypot`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hypot`, `fastResizeToZero`, `...`。

### Lines 2169-2180
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.igamma.default", aten_igamma_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::igamma(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::igamma_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `igamma`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `igamma`, `fastResizeToZero`, `...`。

### Lines 2181-2192
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.igammac.default", aten_igammac_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::igammac(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::igammac_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `igammac`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `igammac`, `fastResizeToZero`, `...`。

### Lines 2193-2207
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.nextafter.default",
    aten_nextafter_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::nextafter(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::nextafter_out(out, self, other);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `nextafter`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `nextafter`, `fastResizeToZero`, `...`。

### Lines 2208-2219
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.fmin.default", aten_fmin_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::fmin(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::fmin_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmin`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmin`, `fastResizeToZero`, `...`。

### Lines 2220-2231
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.fmax.default", aten_fmax_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::fmax(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::fmax_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmax`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `fmax`, `fastResizeToZero`, `...`。

### Lines 2232-2243
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.maximum.default", aten_maximum_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::maximum(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::maximum_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `maximum`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `maximum`, `fastResizeToZero`, `...`。

### Lines 2244-2255
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.minimum.default", aten_minimum_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::minimum(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::minimum_out(out, self, other);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `minimum`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `minimum`, `fastResizeToZero`, `...`。

### Lines 2256-2267
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.min.other", aten_min_other, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::min(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::min_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `min`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `min`, `fastResizeToZero`, `...`。

### Lines 2268-2283
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.quantile.default", aten_quantile_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& q = KernelInput(1).toTensor();
  const auto dim = KernelInput(2).toOptional<int64_t>();
  const auto keepdim = KernelInput(3).toBool();
  const auto interpolation = KernelInput(4).toStringView();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) =
        at::native::quantile(self, q, dim, keepdim, interpolation);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::quantile_out(self, q, dim, keepdim, interpolation, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `toStringView`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `toStringView`, `KernelOutput`, `isNone`, `...`。

### Lines 2284-2302
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.nanquantile.default",
    aten_nanquantile_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& q = KernelInput(1).toTensor();
      const auto dim = KernelInput(2).toOptional<int64_t>();
      const auto keepdim = KernelInput(3).toBool();
      const auto interpolation = KernelInput(4).toStringView();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::native::nanquantile(self, q, dim, keepdim, interpolation);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::nanquantile_out(self, q, dim, keepdim, interpolation, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `toStringView`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `toStringView`, `KernelOutput`, `isNone`, `...`。

### Lines 2303-2316
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.msort.default", aten_msort_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::msort(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::msort_out(self, out);
})

REGISTER_CPU_KERNEL("torch.ops.aten.all.default", aten_all_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `msort`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `msort`, `fastResizeToZero`, `...`。

### Lines 2317-2335
```cpp
    KernelOutput(0) = at::cpu::all(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::all_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.any.default", aten_any_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::any(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::any_out(out, self);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `all`, `toTensor`, `fastResizeToZero`, `all_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `all`, `toTensor`, `fastResizeToZero`, `all_out`, `KernelInput`, `...`。

### Lines 2336-2349
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.renorm.default", aten_renorm_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto p = KernelInput(1).toScalar();
  const auto dim = KernelInput(2).toInt();
  const auto maxnorm = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::renorm(self, p, dim, maxnorm);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::renorm_out(out, self, p, dim, maxnorm);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toInt`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toInt`, `KernelOutput`, `isNone`, `...`。

### Lines 2350-2366
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten._convert_indices_from_coo_to_csr.default",
    aten__convert_indices_from_coo_to_csr_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto size = KernelInput(1).toInt();
      const auto out_int32 = KernelInput(2).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::_convert_indices_from_coo_to_csr(self, size, out_int32);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::_convert_indices_from_coo_to_csr_out(out, self, size, out_int32);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 2367-2385
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten._convert_indices_from_csr_to_coo.default",
    aten__convert_indices_from_csr_to_coo_default,
    {
      const auto& crow_indices = KernelInput(0).toTensor();
      const auto& col_indices = KernelInput(1).toTensor();
      const auto out_int32 = KernelInput(2).toBool();
      const auto transpose = KernelInput(3).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::_convert_indices_from_csr_to_coo(
            crow_indices, col_indices, out_int32, transpose);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::_convert_indices_from_csr_to_coo_out(
          out, crow_indices, col_indices, out_int32, transpose);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `_convert_indices_from_csr_to_coo`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `_convert_indices_from_csr_to_coo`, `...`。

### Lines 2386-2398
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.mse_loss.default", aten_mse_loss_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& target = KernelInput(1).toTensor();
  const auto reduction = KernelInput(2).toInt();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::mse_loss(self, target, reduction);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::mse_loss_out(out, self, target, reduction);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `mse_loss`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `mse_loss`, `...`。

### Lines 2399-2419
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.multi_margin_loss.default",
    aten_multi_margin_loss_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& target = KernelInput(1).toTensor();
      const auto p = KernelInput(2).toScalar();
      const auto margin = KernelInput(3).toScalar();
      const auto weight = KernelInput(4).toOptional<at::Tensor>();
      const auto reduction = KernelInput(5).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::multi_margin_loss_cpu(
            self, target, p, margin, weight, reduction);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::multi_margin_loss_cpu_out(
          self, target, p, margin, weight, reduction, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toInt`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toInt`, `KernelOutput`, `isNone`, `...`。

### Lines 2420-2436
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.multilabel_margin_loss.default",
    aten_multilabel_margin_loss_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& target = KernelInput(1).toTensor();
      const auto reduction = KernelInput(2).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::native::multilabel_margin_loss(self, target, reduction);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::multilabel_margin_loss_out(self, target, reduction, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `multilabel_margin_loss`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `multilabel_margin_loss`, `...`。

### Lines 2437-2452
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.soft_margin_loss.default",
    aten_soft_margin_loss_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& target = KernelInput(1).toTensor();
      const auto reduction = KernelInput(2).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::soft_margin_loss(self, target, reduction);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::soft_margin_loss_out(self, target, reduction, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `soft_margin_loss`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `soft_margin_loss`, `...`。

### Lines 2453-2466
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.elu.default", aten_elu_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto alpha = KernelInput(1).toScalar();
  const auto scale = KernelInput(2).toScalar();
  const auto input_scale = KernelInput(3).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::elu(self, alpha, scale, input_scale);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::elu_out(out, self, alpha, scale, input_scale);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `elu`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `elu`, `...`。

### Lines 2467-2490
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.elu_backward.default",
    aten_elu_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto alpha = KernelInput(1).toScalar();
      const auto scale = KernelInput(2).toScalar();
      const auto input_scale = KernelInput(3).toScalar();
      const auto is_result = KernelInput(4).toBool();
      const auto& self_or_result = KernelInput(5).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::elu_backward(
            grad_output, alpha, scale, input_scale, is_result, self_or_result);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::elu_backward_out(
          grad_input,
          grad_output,
          alpha,
          scale,
          input_scale,
          is_result,
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 2491-2505
```cpp
          self_or_result);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.glu.default", aten_glu_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto dim = KernelInput(1).toInt();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::glu(self, dim);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::glu_out(out, self, dim);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `glu`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `glu`, `...`。

### Lines 2506-2519
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.hardsigmoid.default",
    aten_hardsigmoid_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::hardsigmoid(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::hardsigmoid_out(out, self);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardsigmoid`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardsigmoid`, `fastResizeToZero`, `...`。

### Lines 2520-2534
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.hardsigmoid_backward.default",
    aten_hardsigmoid_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::hardsigmoid_backward(grad_output, self);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::hardsigmoid_backward_out(grad_input, grad_output, self);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardsigmoid_backward`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardsigmoid_backward`, `fastResizeToZero`, `...`。

### Lines 2535-2547
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.hardtanh.default", aten_hardtanh_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto min_val = KernelInput(1).toScalar();
  const auto max_val = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::hardtanh(self, min_val, max_val);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::hardtanh_out(self, min_val, max_val, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `hardtanh`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `hardtanh`, `...`。

### Lines 2548-2561
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.hardswish.default",
    aten_hardswish_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::hardswish(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::hardswish_out(self, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardswish`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `hardswish`, `fastResizeToZero`, `...`。

### Lines 2562-2580
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.leaky_relu_backward.default",
    aten_leaky_relu_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      const auto negative_slope = KernelInput(2).toScalar();
      const auto self_is_result = KernelInput(3).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::leaky_relu_backward(
            grad_output, self, negative_slope, self_is_result);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::leaky_relu_backward_out(
          grad_input, grad_output, self, negative_slope, self_is_result);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `toBool`, `KernelOutput`, `isNone`, `...`。

### Lines 2581-2594
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.log_sigmoid.default",
    aten_log_sigmoid_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::log_sigmoid(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::log_sigmoid_out(self, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `log_sigmoid`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `log_sigmoid`, `fastResizeToZero`, `...`。

### Lines 2595-2607
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.softplus.default", aten_softplus_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto beta = KernelInput(1).toScalar();
  const auto threshold = KernelInput(2).toScalar();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::softplus(self, beta, threshold);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::softplus_out(out, self, beta, threshold);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softplus`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softplus`, `...`。

### Lines 2608-2626
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.softplus_backward.default",
    aten_softplus_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      const auto beta = KernelInput(2).toScalar();
      const auto threshold = KernelInput(3).toScalar();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::softplus_backward(grad_output, self, beta, threshold);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::softplus_backward_out(
          grad_input, grad_output, self, beta, threshold);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softplus_backward`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softplus_backward`, `...`。

### Lines 2627-2641
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.softshrink.default",
    aten_softshrink_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto lambd = KernelInput(1).toScalar();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::cpu::softshrink(self, lambd);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::softshrink_out(out, self, lambd);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softshrink`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softshrink`, `...`。

### Lines 2642-2658
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.softshrink_backward.default",
    aten_softshrink_backward_default,
    {
      const auto& grad_output = KernelInput(0).toTensor();
      const auto& self = KernelInput(1).toTensor();
      const auto lambd = KernelInput(2).toScalar();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) =
            at::cpu::softshrink_backward(grad_output, self, lambd);
        return;
      }
      auto& grad_input = KernelOutput(0).toTensor();
      fastResizeToZero(grad_input);
      at::cpu::softshrink_backward_out(grad_input, grad_output, self, lambd);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softshrink_backward`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toScalar`, `KernelOutput`, `isNone`, `softshrink_backward`, `...`。

### Lines 2659-2672
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.isposinf.default", aten_isposinf_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::cpu::isposinf(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::isposinf_out(out, self);
})

REGISTER_CPU_KERNEL("torch.ops.aten.isneginf.default", aten_isneginf_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `isposinf`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `isposinf`, `fastResizeToZero`, `...`。

### Lines 2673-2686
```cpp
    KernelOutput(0) = at::cpu::isneginf(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::cpu::isneginf_out(out, self);
})

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_entr.default",
    aten_special_entr_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `isneginf`, `toTensor`, `fastResizeToZero`, `isneginf_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `isneginf`, `toTensor`, `fastResizeToZero`, `isneginf_out`, `KernelInput`, `...`。

### Lines 2687-2700
```cpp
        KernelOutput(0) = at::cpu::special_entr(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_entr_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_ndtri.default",
    aten_special_ndtri_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_entr`, `toTensor`, `fastResizeToZero`, `special_entr_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_entr`, `toTensor`, `fastResizeToZero`, `special_entr_out`, `KernelInput`, `...`。

### Lines 2701-2714
```cpp
        KernelOutput(0) = at::cpu::special_ndtri(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_ndtri_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_log_ndtr.default",
    aten_special_log_ndtr_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_ndtri`, `toTensor`, `fastResizeToZero`, `special_ndtri_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_ndtri`, `toTensor`, `fastResizeToZero`, `special_ndtri_out`, `KernelInput`, `...`。

### Lines 2715-2728
```cpp
        KernelOutput(0) = at::cpu::special_log_ndtr(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_log_ndtr_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_expm1.default",
    aten_special_expm1_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_log_ndtr`, `toTensor`, `fastResizeToZero`, `special_log_ndtr_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_log_ndtr`, `toTensor`, `fastResizeToZero`, `special_log_ndtr_out`, `KernelInput`, `...`。

### Lines 2729-2742
```cpp
        KernelOutput(0) = at::native::special_expm1(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_expm1_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_exp2.default",
    aten_special_exp2_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_expm1`, `toTensor`, `fastResizeToZero`, `special_expm1_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_expm1`, `toTensor`, `fastResizeToZero`, `special_expm1_out`, `KernelInput`, `...`。

### Lines 2743-2756
```cpp
        KernelOutput(0) = at::native::special_exp2(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_exp2_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_psi.default",
    aten_special_psi_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_exp2`, `toTensor`, `fastResizeToZero`, `special_exp2_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_exp2`, `toTensor`, `fastResizeToZero`, `special_exp2_out`, `KernelInput`, `...`。

### Lines 2757-2770
```cpp
        KernelOutput(0) = at::native::special_psi(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_psi_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_digamma.default",
    aten_special_digamma_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_psi`, `toTensor`, `fastResizeToZero`, `special_psi_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_psi`, `toTensor`, `fastResizeToZero`, `special_psi_out`, `KernelInput`, `...`。

### Lines 2771-2784
```cpp
        KernelOutput(0) = at::native::special_digamma(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_digamma_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_gammaln.default",
    aten_special_gammaln_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_digamma`, `toTensor`, `fastResizeToZero`, `special_digamma_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_digamma`, `toTensor`, `fastResizeToZero`, `special_digamma_out`, `KernelInput`, `...`。

### Lines 2785-2798
```cpp
        KernelOutput(0) = at::native::special_gammaln(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_gammaln_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_erf.default",
    aten_special_erf_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_gammaln`, `toTensor`, `fastResizeToZero`, `special_gammaln_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_gammaln`, `toTensor`, `fastResizeToZero`, `special_gammaln_out`, `KernelInput`, `...`。

### Lines 2799-2812
```cpp
        KernelOutput(0) = at::native::special_erf(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_erf_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_erfc.default",
    aten_special_erfc_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_erf`, `toTensor`, `fastResizeToZero`, `special_erf_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_erf`, `toTensor`, `fastResizeToZero`, `special_erf_out`, `KernelInput`, `...`。

### Lines 2813-2826
```cpp
        KernelOutput(0) = at::native::special_erfc(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_erfc_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_erfcx.default",
    aten_special_erfcx_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_erfc`, `toTensor`, `fastResizeToZero`, `special_erfc_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_erfc`, `toTensor`, `fastResizeToZero`, `special_erfc_out`, `KernelInput`, `...`。

### Lines 2827-2840
```cpp
        KernelOutput(0) = at::cpu::special_erfcx(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_erfcx_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_erfinv.default",
    aten_special_erfinv_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_erfcx`, `toTensor`, `fastResizeToZero`, `special_erfcx_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_erfcx`, `toTensor`, `fastResizeToZero`, `special_erfcx_out`, `KernelInput`, `...`。

### Lines 2841-2854
```cpp
        KernelOutput(0) = at::native::special_erfinv(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_erfinv_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_ndtr.default",
    aten_special_ndtr_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_erfinv`, `toTensor`, `fastResizeToZero`, `special_erfinv_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_erfinv`, `toTensor`, `fastResizeToZero`, `special_erfinv_out`, `KernelInput`, `...`。

### Lines 2855-2869
```cpp
        KernelOutput(0) = at::native::special_ndtr(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_ndtr_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_xlog1py.default",
    aten_special_xlog1py_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_ndtr`, `toTensor`, `fastResizeToZero`, `special_ndtr_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_ndtr`, `toTensor`, `fastResizeToZero`, `special_ndtr_out`, `KernelInput`, `...`。

### Lines 2870-2884
```cpp
        KernelOutput(0) = at::cpu::special_xlog1py(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_xlog1py_out(out, self, other);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_xlogy.default",
    aten_special_xlogy_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_xlog1py`, `toTensor`, `fastResizeToZero`, `special_xlog1py_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_xlog1py`, `toTensor`, `fastResizeToZero`, `special_xlog1py_out`, `KernelInput`, `...`。

### Lines 2885-2899
```cpp
        KernelOutput(0) = at::native::special_xlogy(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_xlogy_out(self, other, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_zeta.default",
    aten_special_zeta_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_xlogy`, `toTensor`, `fastResizeToZero`, `special_xlogy_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_xlogy`, `toTensor`, `fastResizeToZero`, `special_xlogy_out`, `KernelInput`, `...`。

### Lines 2900-2913
```cpp
        KernelOutput(0) = at::cpu::special_zeta(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_zeta_out(out, self, other);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_i0.default",
    aten_special_i0_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_zeta`, `toTensor`, `fastResizeToZero`, `special_zeta_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_zeta`, `toTensor`, `fastResizeToZero`, `special_zeta_out`, `KernelInput`, `...`。

### Lines 2914-2927
```cpp
        KernelOutput(0) = at::native::special_i0(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_i0_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_i0e.default",
    aten_special_i0e_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_i0`, `toTensor`, `fastResizeToZero`, `special_i0_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_i0`, `toTensor`, `fastResizeToZero`, `special_i0_out`, `KernelInput`, `...`。

### Lines 2928-2941
```cpp
        KernelOutput(0) = at::cpu::special_i0e(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_i0e_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_i1.default",
    aten_special_i1_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_i0e`, `toTensor`, `fastResizeToZero`, `special_i0e_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_i0e`, `toTensor`, `fastResizeToZero`, `special_i0e_out`, `KernelInput`, `...`。

### Lines 2942-2955
```cpp
        KernelOutput(0) = at::cpu::special_i1(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_i1_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_i1e.default",
    aten_special_i1e_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_i1`, `toTensor`, `fastResizeToZero`, `special_i1_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_i1`, `toTensor`, `fastResizeToZero`, `special_i1_out`, `KernelInput`, `...`。

### Lines 2956-2970
```cpp
        KernelOutput(0) = at::cpu::special_i1e(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::special_i1e_out(out, self);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_polygamma.default",
    aten_special_polygamma_default,
    {
      const auto n = KernelInput(0).toInt();
      const auto& self = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_i1e`, `toTensor`, `fastResizeToZero`, `special_i1e_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_i1e`, `toTensor`, `fastResizeToZero`, `special_i1e_out`, `KernelInput`, `...`。

### Lines 2971-2984
```cpp
        KernelOutput(0) = at::native::special_polygamma(n, self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_polygamma_out(n, self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_expit.default",
    aten_special_expit_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_polygamma`, `toTensor`, `fastResizeToZero`, `special_polygamma_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_polygamma`, `toTensor`, `fastResizeToZero`, `special_polygamma_out`, `KernelInput`, `...`。

### Lines 2985-2998
```cpp
        KernelOutput(0) = at::native::special_expit(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_expit_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_sinc.default",
    aten_special_sinc_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_expit`, `toTensor`, `fastResizeToZero`, `special_expit_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_expit`, `toTensor`, `fastResizeToZero`, `special_expit_out`, `KernelInput`, `...`。

### Lines 2999-3013
```cpp
        KernelOutput(0) = at::native::special_sinc(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_sinc_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_round.default",
    aten_special_round_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto decimals = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_sinc`, `toTensor`, `fastResizeToZero`, `special_sinc_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_sinc`, `toTensor`, `fastResizeToZero`, `special_sinc_out`, `KernelInput`, `...`。

### Lines 3014-3027
```cpp
        KernelOutput(0) = at::native::special_round(self, decimals);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_round_out(self, decimals, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_log1p.default",
    aten_special_log1p_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_round`, `toTensor`, `fastResizeToZero`, `special_round_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_round`, `toTensor`, `fastResizeToZero`, `special_round_out`, `KernelInput`, `...`。

### Lines 3028-3042
```cpp
        KernelOutput(0) = at::native::special_log1p(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_log1p_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_gammainc.default",
    aten_special_gammainc_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_log1p`, `toTensor`, `fastResizeToZero`, `special_log1p_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_log1p`, `toTensor`, `fastResizeToZero`, `special_log1p_out`, `KernelInput`, `...`。

### Lines 3043-3057
```cpp
        KernelOutput(0) = at::native::special_gammainc(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_gammainc_out(self, other, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_gammaincc.default",
    aten_special_gammaincc_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_gammainc`, `toTensor`, `fastResizeToZero`, `special_gammainc_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_gammainc`, `toTensor`, `fastResizeToZero`, `special_gammainc_out`, `KernelInput`, `...`。

### Lines 3058-3072
```cpp
        KernelOutput(0) = at::native::special_gammaincc(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_gammaincc_out(self, other, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.special_multigammaln.default",
    aten_special_multigammaln_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto p = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_gammaincc`, `toTensor`, `fastResizeToZero`, `special_gammaincc_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_gammaincc`, `toTensor`, `fastResizeToZero`, `special_gammaincc_out`, `KernelInput`, `...`。

### Lines 3073-3088
```cpp
        KernelOutput(0) = at::native::special_multigammaln(self, p);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::special_multigammaln_out(self, p, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_cross.default",
    aten_linalg_cross_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      const auto dim = KernelInput(2).toInt();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `special_multigammaln`, `toTensor`, `fastResizeToZero`, `special_multigammaln_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `special_multigammaln`, `toTensor`, `fastResizeToZero`, `special_multigammaln_out`, `KernelInput`, `...`。

### Lines 3089-3102
```cpp
        KernelOutput(0) = at::cpu::linalg_cross(self, other, dim);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::cpu::linalg_cross_out(out, self, other, dim);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_det.default",
    aten_linalg_det_default,
    {
      const auto& A = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_cross`, `toTensor`, `fastResizeToZero`, `linalg_cross_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_cross`, `toTensor`, `fastResizeToZero`, `linalg_cross_out`, `KernelInput`, `...`。

### Lines 3103-3117
```cpp
        KernelOutput(0) = at::native::linalg_det(A);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_det_out(A, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_matmul.default",
    aten_linalg_matmul_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto& other = KernelInput(1).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_det`, `toTensor`, `fastResizeToZero`, `linalg_det_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_det`, `toTensor`, `fastResizeToZero`, `linalg_det_out`, `KernelInput`, `...`。

### Lines 3118-3131
```cpp
        KernelOutput(0) = at::native::linalg_matmul(self, other);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_matmul_out(self, other, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_eigvals.default",
    aten_linalg_eigvals_default,
    {
      const auto& self = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_matmul`, `toTensor`, `fastResizeToZero`, `linalg_matmul_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_matmul`, `toTensor`, `fastResizeToZero`, `linalg_matmul_out`, `KernelInput`, `...`。

### Lines 3132-3145
```cpp
        KernelOutput(0) = at::native::linalg_eigvals(self);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_eigvals_out(self, out);
    })

REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_inv.default",
    aten_linalg_inv_default,
    {
      const auto& A = KernelInput(0).toTensor();
      if (KernelOutput(0).isNone()) {
```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_eigvals`, `toTensor`, `fastResizeToZero`, `linalg_eigvals_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_eigvals`, `toTensor`, `fastResizeToZero`, `linalg_eigvals_out`, `KernelInput`, `...`。

### Lines 3146-3164
```cpp
        KernelOutput(0) = at::native::linalg_inv(A);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_inv_out(A, out);
    })

REGISTER_CPU_KERNEL("torch.ops.aten.inverse.default", aten_inverse_default, {
  const auto& self = KernelInput(0).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::inverse(self);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::inverse_out(self, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelOutput`, `linalg_inv`, `toTensor`, `fastResizeToZero`, `linalg_inv_out`, `KernelInput`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelOutput`, `linalg_inv`, `toTensor`, `fastResizeToZero`, `linalg_inv_out`, `KernelInput`, `...`。

### Lines 3165-3176
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.inner.default", aten_inner_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& other = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::inner(self, other);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::inner_out(self, other, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `inner`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `inner`, `fastResizeToZero`, `...`。

### Lines 3177-3188
```cpp
REGISTER_CPU_KERNEL("torch.ops.aten.outer.default", aten_outer_default, {
  const auto& self = KernelInput(0).toTensor();
  const auto& vec2 = KernelInput(1).toTensor();
  if (KernelOutput(0).isNone()) {
    KernelOutput(0) = at::native::outer(self, vec2);
    return;
  }
  auto& out = KernelOutput(0).toTensor();
  fastResizeToZero(out);
  at::native::outer_out(self, vec2, out);
})

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `outer`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `outer`, `fastResizeToZero`, `...`。

### Lines 3189-3203
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_cond.default",
    aten_linalg_cond_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto p = KernelInput(1).toOptional<at::Scalar>();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::linalg_cond(self, p);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_cond_out(self, p, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `linalg_cond`, `fastResizeToZero`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `linalg_cond`, `fastResizeToZero`, `...`。

### Lines 3204-3219
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_solve.default",
    aten_linalg_solve_default,
    {
      const auto& A = KernelInput(0).toTensor();
      const auto& B = KernelInput(1).toTensor();
      const auto left = KernelInput(2).toBool();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::linalg_solve(A, B, left);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_solve_out(A, B, left, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `linalg_solve`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toBool`, `KernelOutput`, `isNone`, `linalg_solve`, `...`。

### Lines 3220-3234
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_tensorinv.default",
    aten_linalg_tensorinv_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto ind = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::linalg_tensorinv(self, ind);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_tensorinv_out(self, ind, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `linalg_tensorinv`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `linalg_tensorinv`, `...`。

### Lines 3235-3249
```cpp
REGISTER_CPU_KERNEL(
    "torch.ops.aten.linalg_matrix_power.default",
    aten_linalg_matrix_power_default,
    {
      const auto& self = KernelInput(0).toTensor();
      const auto n = KernelInput(1).toInt();
      if (KernelOutput(0).isNone()) {
        KernelOutput(0) = at::native::linalg_matrix_power(self, n);
        return;
      }
      auto& out = KernelOutput(0).toTensor();
      fastResizeToZero(out);
      at::native::linalg_matrix_power_out(self, n, out);
    })

```
- EN: Registers externally visible hooks, bindings, or exported entry points. Representative symbols: `KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `linalg_matrix_power`, `...`.
- CN: 注册对外可见的钩子、绑定或导出入口。代表性符号：`KernelInput`, `toTensor`, `toInt`, `KernelOutput`, `isNone`, `linalg_matrix_power`, `...`。

### Lines 3250-3250
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for GeneratedStaticDispatchKernels. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GeneratedStaticDispatchKernels 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `ATen/CPUFunctions.h`, `ATen/InferSize.h`, `ATen/NativeFunctions.h`, `ATen/Parallel.h`, `ATen/ScalarOps.h`, `ATen/TensorUtils.h`, `ATen/cpu/vec/functional.h`, `ATen/cpu/vec/vec.h`, `ATen/native/EmbeddingBag.h`, `ATen/native/Fill.h`, `...`
- External includes / 外部头文件: `iterator`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `KernelInput`, `toTensor`, `KernelOutput`, `isNone`, `absolute`, `fastResizeToZero`, `absolute_out`, `angle`, `angle_out`, `sgn`, `...`
