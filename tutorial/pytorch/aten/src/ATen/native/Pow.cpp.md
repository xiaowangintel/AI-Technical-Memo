# Pow.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/Pow.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on pow; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 pow；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/native/Pow.h>

#include <ATen/core/Tensor.h>
#include <ATen/ScalarOps.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/float_power_native.h>
#include <ATen/ops/pow.h>
#include <ATen/ops/pow_native.h>
#include <ATen/ops/result_type.h>
#endif

namespace at::meta {

TORCH_META_FUNC2(pow, Tensor_Tensor) (const Tensor& base, const Tensor& exp) {
  build_borrowing_binary_op(maybe_get_output(), base, exp);
}

TORCH_META_FUNC2(pow, Tensor_Scalar) (const Tensor& base, const Scalar& exp) {
  // Numpy compatibility check:
  TORCH_CHECK(!(isIntegralType(base.scalar_type(), true) &&
              exp.isIntegral(true) && exp.toLong() < 0),
              "Integers to negative integer powers are not allowed.");

  auto common_dtype = at::result_type(base, exp);
  build_output_borrowing_argument_owning_unary_op(maybe_get_output(), base.to(common_dtype));
```
- EN: Lines 1-30 pull in 9 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_META_FUNC2, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 9 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_META_FUNC2，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
}

TORCH_META_FUNC2(pow, Scalar) (const Scalar& base, const Tensor& exp) {
    // This overload doesn't directly use TensorIterator. It attempts to short-circuit,
    // but otherwise redispatches to the Tensor_Tensor overload.
    auto dtype = maybe_get_output().defined() ? maybe_get_output().scalar_type() : at::result_type(base, exp);
    set_output_raw_strided(0, exp.sizes(), {}, exp.options().dtype(dtype), exp.has_names() ? exp.names() : ArrayRef<Dimname>());
}

} // namespace at::meta

namespace at::native {

DEFINE_DISPATCH(pow_tensor_tensor_stub);
DEFINE_DISPATCH(pow_tensor_scalar_stub);

TORCH_IMPL_FUNC(pow_Tensor_Tensor_out) (const Tensor& base, const Tensor& exp, const Tensor& out) {
  pow_tensor_tensor_stub(device_type(), *this);
}

TORCH_IMPL_FUNC(pow_Tensor_Scalar_out) (const Tensor& base, const Scalar& exp, const Tensor& out) {
  if (exp.equal(0.0) || exp.equal(false)) {
    out.fill_(1);
  } else if (exp.equal(1.0) || exp.equal(true) ) {
    out.copy_(base);
  } else {
    pow_tensor_scalar_stub(device_type(), *this, exp);
  }
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_META_FUNC2, TORCH_IMPL_FUNC, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_META_FUNC2, TORCH_IMPL_FUNC，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

### Lines 61-90
```cpp
TORCH_IMPL_FUNC(pow_Scalar_out) (const Scalar& base, const Tensor& exp, const Tensor& out) {
  if (base.equal(1.0)) {
    out.fill_(1);
  } else {
    // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
    at::pow_out(const_cast<Tensor&>(out), wrapped_scalar_tensor(base, exp.device()), exp); // redispatch!
  }
}

Tensor& float_power_out(const Tensor& base, const Tensor& exp, Tensor& result) {
  auto dtype = (at::isComplexType(base.scalar_type()) || at::isComplexType(exp.scalar_type())) ?
                at::kComplexDouble : at::kDouble;
  TORCH_CHECK(result.scalar_type() == dtype,
              "the output given to float_power has dtype ", result.scalar_type(),
              " but the operation's result requires dtype ", dtype);

  return at::pow_out(result, base.to(dtype), exp.to(dtype));
}

Tensor& float_power_out(const Tensor& base, const Scalar& exp, Tensor& result) {
  auto dtype = (at::isComplexType(base.scalar_type()) || exp.isComplex()) ? at::kComplexDouble : at::kDouble;
  TORCH_CHECK(result.scalar_type() == dtype,
              "the output given to float_power has dtype ", result.scalar_type(),
              " but the operation's result requires dtype ", dtype);

  // Note: need the casts inside the ternary because conversion functions return e.g. c10::complex,
  // which causes a complex scalar to always be returned.
  auto casted_exp = (dtype == at::kComplexDouble) ? Scalar(exp.toComplexDouble()) : Scalar(exp.toDouble());
  return at::pow_out(result, base.to(dtype), casted_exp);
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_IMPL_FUNC, NOLINTNEXTLINE, float_power_out, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_IMPL_FUNC, NOLINTNEXTLINE, float_power_out，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp

Tensor& float_power_out(const Scalar& base, const Tensor& exp, Tensor& result) {
  auto dtype = (at::isComplexType(exp.scalar_type()) || base.isComplex()) ? at::kComplexDouble : at::kDouble;
  TORCH_CHECK(result.scalar_type() == dtype,
              "the output given to float_power has dtype ", result.scalar_type(),
              " but the operation's result requires dtype ", dtype);

  auto casted_base = (dtype == at::kComplexDouble) ? Scalar(base.toComplexDouble()) : Scalar(base.toDouble());
  return at::pow_out(result, casted_base, exp.to(dtype));
}

Tensor float_power(const Tensor& base, const Scalar& exp) {
  auto dtype = (at::isComplexType(base.scalar_type()) || exp.isComplex()) ? at::kComplexDouble : at::kDouble;
  auto casted_exp = (dtype == at::kComplexDouble) ? Scalar(exp.toComplexDouble()) : Scalar(exp.toDouble());
  return at::pow(base.to(dtype), casted_exp);
}

Tensor float_power(const Scalar& base, const Tensor& exp) {
  auto dtype = (at::isComplexType(exp.scalar_type()) || base.isComplex()) ? at::kComplexDouble : at::kDouble;
  auto casted_base = (dtype == at::kComplexDouble) ? Scalar(base.toComplexDouble()) : Scalar(base.toDouble());
  return at::pow(casted_base, exp.to(dtype));
}

Tensor float_power(const Tensor& base, const Tensor& exp) {
  auto dtype = (at::isComplexType(base.scalar_type()) || at::isComplexType(exp.scalar_type())) ? at::kComplexDouble : at::kDouble;
  return at::pow(base.to(dtype), exp.to(dtype));
}

Tensor& float_power_(Tensor& base, const Tensor& exp) {
  auto dtype = (at::isComplexType(base.scalar_type()) || at::isComplexType(exp.scalar_type())) ? at::kComplexDouble : at::kDouble;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are float_power_out, float_power, float_power_, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 float_power_out, float_power, float_power_，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-138
```cpp
  TORCH_CHECK(base.scalar_type() == dtype,
              "the base given to float_power_ has dtype ", base.scalar_type(),
              " but the operation's result requires dtype ", dtype);

  return base.pow_(exp.to(dtype));
}

Tensor& float_power_(Tensor& base, const Scalar& exp) {
  auto dtype = (at::isComplexType(base.scalar_type()) || exp.isComplex()) ? at::kComplexDouble : at::kDouble;
  TORCH_CHECK(base.scalar_type() == dtype,
              "the base given to float_power_ has dtype ", base.scalar_type(),
              " but the operation's result requires dtype ", dtype);

  auto casted_exp = (dtype == at::kComplexDouble) ? Scalar(exp.toComplexDouble()) : Scalar(exp.toDouble());
  return base.pow_(casted_exp);
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are float_power_, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 float_power_，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: TORCH_META_FUNC2, TORCH_IMPL_FUNC, float_power_out, float_power, float_power_.
- CN: 重要符号：TORCH_META_FUNC2, TORCH_IMPL_FUNC, float_power_out, float_power, float_power_。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/Pow.h, ATen/core/Tensor.h, ATen/ScalarOps.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/float_power_native.h, ATen/ops/pow.h, ATen/ops/pow_native.h, ATen/ops/result_type.h`.
- CN: 主要内部头文件：`ATen/native/Pow.h, ATen/core/Tensor.h, ATen/ScalarOps.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/float_power_native.h, ATen/ops/pow.h, ATen/ops/pow_native.h, ATen/ops/result_type.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `TORCH_META_FUNC2, TORCH_IMPL_FUNC, float_power_out, float_power, float_power_`.
- CN: 实现围绕 `TORCH_META_FUNC2, TORCH_IMPL_FUNC, float_power_out, float_power, float_power_` 等符号展开。
