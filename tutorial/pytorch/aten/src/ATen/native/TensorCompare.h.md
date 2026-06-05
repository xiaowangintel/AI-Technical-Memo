# TensorCompare.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TensorCompare.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on tensor compare; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 tensor compare；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <ATen/native/DispatchStub.h>

namespace c10 {
class Scalar;
}

namespace at {
class Tensor;
struct TensorIterator;
struct TensorIteratorBase;
} // namespace at

namespace at::native {

using reduce_minmax_fn =
    void (*)(Tensor&, Tensor&, const Tensor&, int64_t, bool);
using structured_reduce_minmax_fn =
    void (*)(const Tensor&, const Tensor&, const Tensor&, int64_t, bool);

DECLARE_DISPATCH(structured_reduce_minmax_fn, max_stub)
DECLARE_DISPATCH(structured_reduce_minmax_fn, min_stub)

using where_fn = void (*)(TensorIterator&);
DECLARE_DISPATCH(where_fn, where_kernel)

using is_infinity_op_fn = void (*)(TensorIteratorBase&);
DECLARE_DISPATCH(is_infinity_op_fn, isposinf_stub)
DECLARE_DISPATCH(is_infinity_op_fn, isneginf_stub)
```
- EN: Lines 1-30 pull in 1 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 1 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-56
```cpp

using mode_fn = void (*)(Tensor&, Tensor&, const Tensor&, int64_t, bool);
DECLARE_DISPATCH(mode_fn, mode_stub)

using clamp_tensor_fn = void (*)(TensorIteratorBase&);
DECLARE_DISPATCH(clamp_tensor_fn, clamp_stub)

namespace detail {
enum class ClampLimits { Min, Max, MinMax };
}

DECLARE_DISPATCH(
    void (*)(TensorIteratorBase&, const c10::Scalar&, const c10::Scalar&),
    clamp_scalar_stub)
DECLARE_DISPATCH(
    void (*)(TensorIteratorBase&, c10::Scalar),
    clamp_min_scalar_stub)
DECLARE_DISPATCH(
    void (*)(TensorIteratorBase&, c10::Scalar),
    clamp_max_scalar_stub)

using isin_default_fn =
    void (*)(const Tensor&, const Tensor&, bool, const Tensor&);
DECLARE_DISPATCH(isin_default_fn, isin_default_stub)

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Notable symbols: DECLARE_DISPATCH.
- CN: 重要符号：DECLARE_DISPATCH。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/DispatchStub.h`.
- CN: 主要内部头文件：`ATen/native/DispatchStub.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `DECLARE_DISPATCH`.
- CN: 实现围绕 `DECLARE_DISPATCH` 等符号展开。
