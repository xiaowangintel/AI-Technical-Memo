# binary.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/masked/maskedtensor/binary.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements masked-tensor helpers and namespace wrappers for masked operations.
- **Purpose (CN)**: 实现 masked tensor 辅助逻辑以及 masked 运算的命名空间包装层。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates

import torch

from .core import (
    _map_mt_args_kwargs,
    _masks_match,
    _tensors_match,
    _wrap_result,
    is_masked_tensor,
)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, .core.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.core。

### Lines 15-28 / 第 15-28 行
````python
__all__ = []  # type: ignore[var-annotated]

BINARY_NAMES = [
    "add",
    "atan2",
    "arctan2",
    "bitwise_and",
    "bitwise_or",
    "bitwise_xor",
    "bitwise_left_shift",
    "bitwise_right_shift",
    "div",
    "divide",
    "floor_divide",
````
- **EN**: `__all__` defines the public symbols that this module chooses to export.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。

### Lines 29-42 / 第 29-42 行
````python
    "fmod",
    "logaddexp",
    "logaddexp2",
    "mul",
    "multiply",
    "nextafter",
    "remainder",
    "sub",
    "subtract",
    "true_divide",
    "eq",
    "ne",
    "le",
    "ge",
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 43-54 / 第 43-54 行
````python
    "greater",
    "greater_equal",
    "gt",
    "less_equal",
    "lt",
    "less",
    "maximum",
    "minimum",
    "fmax",
    "fmin",
    "not_equal",
]
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 56-69 / 第 56-69 行
````python
INPLACE_BINARY_NAMES = [
    n + "_"
    for n in (
        list(
            set(BINARY_NAMES)
            - {
                "logaddexp",
                "logaddexp2",
                "equal",
                "fmin",
                "minimum",
                "maximum",
                "fmax",
            }
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 70-82 / 第 70-82 行
````python
        )
    )
]


def _get_at_least_one_mask(a, b):
    if not is_masked_tensor(a) and not is_masked_tensor(b):
        raise TypeError("At least one of `a` and `b` must be a MaskedTensor")
    if not _masks_match(a, b):
        raise ValueError("a and b must have matching masks")
    if is_masked_tensor(a):
        return a.get_mask()
    return b.get_mask()
````
- **EN**: This chunk defines `_get_at_least_one_mask`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_at_least_one_mask`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 85-97 / 第 85-97 行
````python
def _binary_helper(fn, args, kwargs, inplace):
    if len(kwargs) != 0:
        raise ValueError("len(kwargs) must equal 0")
    for a in args[2:]:
        if torch.is_tensor(a):
            raise TypeError(
                "MaskedTensor binary ops do not support Tensor arguments aside from the lhs and rhs"
            )

    if not _masks_match(*args[:2]):
        raise ValueError(
            "Input masks must match. If you need support for this, please open an issue on Github."
        )
````
- **EN**: This chunk defines `_binary_helper`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_binary_helper`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 99-112 / 第 99-112 行
````python
    data_args, _data_kwargs = _map_mt_args_kwargs(args, kwargs, lambda x: x.get_data())
    mask_args, _mask_kwargs = _map_mt_args_kwargs(args, kwargs, lambda x: x.get_mask())

    args0_layout = data_args[0].layout
    same_layout = (
        torch.is_tensor(data_args[1]) or is_masked_tensor(data_args[1])
    ) and (args0_layout == data_args[1].layout)

    if args0_layout == torch.sparse_coo:
        if same_layout:
            if not _tensors_match(data_args[0].indices(), data_args[1].indices()):
                raise ValueError(
                    "sparse_coo indices must match. If you need support for this, please open an issue on Github."
                )
````
- **EN**: This chunk continues `_binary_helper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_binary_helper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 113-124 / 第 113-124 行
````python
            if data_args[0].size() != data_args[1].size():
                raise ValueError(
                    "input1 and input2 must have the same size for binary functions."
                )

            data_args[1] = data_args[1].values()

        i = data_args[0].indices()
        size = data_args[0].size()
        data_args[0] = data_args[0].values()
        v = fn(*data_args)
        result_data = torch.sparse_coo_tensor(i, v, size)
````
- **EN**: This chunk continues `_binary_helper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_binary_helper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 126-138 / 第 126-138 行
````python
    elif args0_layout == torch.sparse_csr:
        if same_layout:
            if not (
                _tensors_match(data_args[0].crow_indices(), data_args[1].crow_indices())
                and _tensors_match(
                    data_args[0].col_indices(), data_args[1].col_indices()
                )
            ):
                raise ValueError(
                    "sparse_csr indices must match. If you need support for this, please open an issue on Github."
                )

            data_args[1] = data_args[1].values()
````
- **EN**: This chunk continues `_binary_helper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_binary_helper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 140-148 / 第 140-148 行
````python
        crow = data_args[0].crow_indices()
        col = data_args[0].col_indices()
        size = data_args[0].size()
        data_args[0] = data_args[0].values()
        v = fn(*data_args)
        result_data = torch.sparse_csr_tensor(crow, col, v, size)

    else:
        result_data = fn(*data_args)
````
- **EN**: This chunk continues `_binary_helper` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_binary_helper`，进一步展开其内部控制流或状态更新。

### Lines 150-162 / 第 150-162 行
````python
    if inplace:
        args[0]._set_data_mask(result_data, mask_args[0])
        return args[0]
    else:
        result_mask = _get_at_least_one_mask(*args[:2])
        # sparse tensors don't have strides so we can only expand if the layout is strided
        if args0_layout == torch.strided:
            result_mask = result_mask.expand_as(result_data)
        return _wrap_result(result_data, result_mask)


def _torch_binary(fn_name):
    fn = getattr(torch.ops.aten, fn_name)
````
- **EN**: This chunk defines `_torch_binary`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_torch_binary`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 164-176 / 第 164-176 行
````python
    def binary_fn(*args, **kwargs):
        return _binary_helper(fn, args, kwargs, inplace=False)

    return binary_fn


def _torch_inplace_binary(fn_name):
    fn = getattr(torch.ops.aten, fn_name)

    def binary_fn(*args, **kwargs):
        return _binary_helper(fn, args, kwargs, inplace=True)

    return binary_fn
````
- **EN**: This chunk defines `_torch_inplace_binary`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_torch_inplace_binary`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 179-188 / 第 179-188 行
````python
NATIVE_BINARY_MAP = {
    getattr(torch.ops.aten, name): _torch_binary(name) for name in BINARY_NAMES
}
NATIVE_INPLACE_BINARY_MAP = {
    getattr(torch.ops.aten, name): _torch_inplace_binary(name)
    for name in INPLACE_BINARY_NAMES
}

NATIVE_BINARY_FNS = list(NATIVE_BINARY_MAP.keys())
NATIVE_INPLACE_BINARY_FNS = list(NATIVE_INPLACE_BINARY_MAP.keys())
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 191-200 / 第 191-200 行
````python
def _is_native_binary(fn):
    return fn in NATIVE_BINARY_FNS or fn in NATIVE_INPLACE_BINARY_FNS


def _apply_native_binary(fn, *args, **kwargs):
    if fn in NATIVE_BINARY_FNS:
        return NATIVE_BINARY_MAP[fn](*args, **kwargs)
    if fn in NATIVE_INPLACE_BINARY_FNS:
        return NATIVE_INPLACE_BINARY_MAP[fn](*args, **kwargs)
    return NotImplemented
````
- **EN**: This chunk defines `_apply_native_binary`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_apply_native_binary`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Masked operations**
  - EN: Represents operations that carry masks alongside data and preserve masked semantics.
  - CN: 表示同时携带 mask 与数据的运算，并保持 masked 语义。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **BINARY_NAMES**
  - EN: `BINARY_NAMES` is one of the main symbols declared or implemented in this file.
  - CN: `BINARY_NAMES` 是本文件声明或实现的主要符号之一。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.core`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `BINARY_NAMES`, `INPLACE_BINARY_NAMES`, `_get_at_least_one_mask`, `_binary_helper`, `_torch_binary`, `_torch_inplace_binary`, `NATIVE_BINARY_MAP`, `NATIVE_INPLACE_BINARY_MAP`, `NATIVE_BINARY_FNS`
