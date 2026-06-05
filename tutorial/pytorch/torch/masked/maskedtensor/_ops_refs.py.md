# _ops_refs.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/masked/maskedtensor/_ops_refs.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements masked-tensor helpers and namespace wrappers for masked operations.
- **Purpose (CN)**: 实现 masked tensor 辅助逻辑以及 masked 运算的命名空间包装层。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates

from collections.abc import Callable
from functools import partial
from typing import Any, TYPE_CHECKING

import torch

from .binary import _apply_native_binary, NATIVE_BINARY_FNS, NATIVE_INPLACE_BINARY_FNS
from .core import (
    _get_data,
    _masks_match,
    _maybe_get_mask,
    is_masked_tensor,
    MaskedTensor,
)
from .passthrough import _apply_pass_through_fn, PASSTHROUGH_FNS
from .reductions import (
    _apply_reduction,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, .binary, .core, ...; standard-library helpers such as collections.abc, functools, typing.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.binary、.core、...；标准库辅助模块，如 collections.abc、functools、typing。

### Lines 21-32 / 第 21-32 行
````python
    NATIVE_REDUCE_FNS,
    TENSOR_REDUCE_FNS,
    TORCH_REDUCE_FNS,
)
from .unary import _apply_native_unary, NATIVE_INPLACE_UNARY_FNS, NATIVE_UNARY_FNS


if TYPE_CHECKING:
    from torch._ops import OpOverload


__all__ = []  # type: ignore[var-annotated]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as .unary, torch._ops. `__all__` defines the public symbols that this module chooses to export. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 .unary、torch._ops。 `__all__` 定义了本模块选择导出的公共符号。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 35-53 / 第 35-53 行
````python
def _check_args_kwargs_length(
    args, kwargs, error_prefix, len_args=None, len_kwargs=None
):
    if len_args is not None and len_args != len(args):
        raise ValueError(
            f"{error_prefix}: len(args) must be {len_args} but got {len(args)}"
        )
    if len_kwargs is not None and len_kwargs != len(kwargs):
        raise ValueError(
            f"{error_prefix}: len(kwargs) must be {len_kwargs} but got {len(kwargs)}"
        )


class _MaskedContiguous(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, input):
        if not is_masked_tensor(input):
            raise ValueError("MaskedContiguous forward: input must be a MaskedTensor.")
````
- **EN**: It introduces or extends `_MaskedContiguous`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 它引入或扩展了 `_MaskedContiguous`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 55-66 / 第 55-66 行
````python
        if input.is_contiguous():
            return input

        data = input.get_data()
        mask = input.get_mask()

        return MaskedTensor(data.contiguous(), mask.contiguous())

    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        return grad_output
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 69-83 / 第 69-83 行
````python
class _MaskedToDense(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, input):
        if not is_masked_tensor(input):
            raise ValueError("MaskedToDense forward: input must be a MaskedTensor.")

        if input.layout == torch.strided:
            return input

        ctx.layout = input.layout
        data = input.get_data()
        mask = input.get_mask()

        return MaskedTensor(data.to_dense(), mask.to_dense())
````
- **EN**: It introduces or extends `_MaskedToDense`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_MaskedToDense`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 85-96 / 第 85-96 行
````python
    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        layout = ctx.layout

        if layout == torch.sparse_coo:
            return grad_output.to_sparse_coo()
        elif layout == torch.sparse_csr:
            return grad_output.to_sparse_csr()
        elif layout == torch.strided:
            return grad_output.to_dense()
        raise ValueError(f"to_dense: Unsupported input layout: {layout}")
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 99-115 / 第 99-115 行
````python
class _MaskedToSparse(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, input):
        if not is_masked_tensor(input):
            raise ValueError("MaskedToSparse forward: input must be a MaskedTensor.")

        # Following the convention from sparse tensors that to_sparse always means that we convert to sparse_coo
        if input.layout == torch.sparse_coo:
            return input

        data = input.get_data()
        mask = input.get_mask()
        sparse_mask = mask.to_sparse_coo().coalesce()
        sparse_data = data.sparse_mask(sparse_mask)

        return MaskedTensor(sparse_data, sparse_mask)
````
- **EN**: It introduces or extends `_MaskedToSparse`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_MaskedToSparse`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 117-133 / 第 117-133 行
````python
    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        return grad_output.to_dense()


class _MaskedToSparseCsr(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, input):
        if not is_masked_tensor(input):
            raise ValueError("MaskedToSparseCsr forward: input must be a MaskedTensor.")

        if input._masked_data.ndim != 2:
            raise ValueError(
                f"Only 2D tensors can be converted to the SparseCsr layout but got shape: {input._masked_data.size()}"
            )
````
- **EN**: It introduces or extends `_MaskedToSparseCsr`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_MaskedToSparseCsr`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 135-148 / 第 135-148 行
````python
        if input.layout == torch.sparse_csr:
            return input

        data = input.get_data()
        mask = input.get_mask()
        sparse_mask = mask.to_sparse_csr()
        sparse_data = data.sparse_mask(sparse_mask)

        return MaskedTensor(sparse_data, sparse_mask)

    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        return grad_output.to_dense()
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 151-165 / 第 151-165 行
````python
class _MaskedWhere(torch.autograd.Function):
    @staticmethod
    # pyrefly: ignore [bad-override]
    def forward(ctx, cond, self, other):
        ctx.mark_non_differentiable(cond)
        ctx.save_for_backward(cond)
        return torch.ops.aten.where(cond, self, other)

    @staticmethod
    # pyrefly: ignore [bad-override]
    def backward(ctx, grad_output):
        (cond,) = ctx.saved_tensors

        def masked_out_like(mt):
            return MaskedTensor(mt.get_data(), torch.zeros_like(mt.get_mask()).bool())
````
- **EN**: It introduces or extends `_MaskedWhere`, which hold the main object-oriented state for this portion of the file. This chunk defines `masked_out_like`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_MaskedWhere`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `masked_out_like`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 167-182 / 第 167-182 行
````python
        return (
            None,
            torch.ops.aten.where(cond, grad_output, masked_out_like(grad_output)),
            torch.ops.aten.where(cond, masked_out_like(grad_output), grad_output),
        )


_MASKEDTENSOR_FUNCTION_TABLE = {}

_function_fn_apply_map = {
    (
        tuple(NATIVE_REDUCE_FNS),
        tuple(TORCH_REDUCE_FNS),
        tuple(TENSOR_REDUCE_FNS),
    ): _apply_reduction,
}
````
- **EN**: This chunk continues `_MaskedWhere` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_MaskedWhere`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 184-200 / 第 184-200 行
````python
for fn_map_list, apply_fn in _function_fn_apply_map.items():
    for fn_map in fn_map_list:
        for fn in fn_map:
            _MASKEDTENSOR_FUNCTION_TABLE[fn] = partial(apply_fn, fn)


def register_function_func(ops):
    """
    Used for registering a new __torch_function__ function to MaskedTensor
    Called via _MASKEDTENSOR_FUNCTION_TABLE[func](*args, **kwargs)

    The code to register a new function looks like:

    @register_function_func(list_of_ops)
    def foo(func, *args, **kwargs):
        <implementation>
    """
````
- **EN**: This chunk defines `foo`, which implements a focused helper used by the surrounding module. Decorators such as `register_function_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `foo`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_function_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 202-219 / 第 202-219 行
````python
    def wrapper(func):
        for op in ops:
            _MASKEDTENSOR_FUNCTION_TABLE[op] = partial(func, op)

    return wrapper


@register_function_func(NATIVE_REDUCE_FNS + TORCH_REDUCE_FNS + TENSOR_REDUCE_FNS)
def _general_function_reductions(func, *args, **kwargs):
    return _apply_reduction(func, *args, **kwargs)


@register_function_func([torch.Tensor.where, torch.where])
def _function_where(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, "__torch_function__, torch.where", len_args=3, len_kwargs=0
    )
    return _MaskedWhere.apply(*args)
````
- **EN**: This chunk defines `_function_where`, which implements a focused helper used by the surrounding module. Decorators such as `register_function_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_function_where`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_function_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 222-239 / 第 222-239 行
````python
@register_function_func([torch.Tensor.contiguous])
def _function_contiguous(func, *args, **kwargs):
    return _MaskedContiguous.apply(args[0])


@register_function_func([torch.Tensor.to_dense])
def _function_to_dense(func, *args, **kwargs):
    return _MaskedToDense.apply(args[0])


@register_function_func([torch.Tensor.to_sparse])
def _function_to_sparse(func, *args, **kwargs):
    return _MaskedToSparse.apply(args[0])


@register_function_func([torch.Tensor.to_sparse_csr])
def _function_to_sparse_csr(func, *args, **kwargs):
    return _MaskedToSparseCsr.apply(args[0])
````
- **EN**: This chunk defines `_function_to_sparse_csr`, which parses structured input into internal objects or validated metadata. Decorators such as `register_function_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_function_to_sparse_csr`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 像 `register_function_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 242-259 / 第 242-259 行
````python
_MASKEDTENSOR_DISPATCH_TABLE: dict["OpOverload", Callable[..., Any]] = {}


def register_dispatch_func(aten_ops):
    """
    Used for registering a new __torch_dispatch__ function to MaskedTensor
    Called via _MASKEDTENSOR_DISPATCH_TABLE[func](*args, **kwargs)

    The code to register a new function looks like:

    @register_dispatch_func(list_of_ops)
    def foo(func, *args, **kwargs):
        <implementation>
    """

    def wrapper(func):
        for aten_op in aten_ops:
            _MASKEDTENSOR_DISPATCH_TABLE[aten_op] = partial(func, aten_op)
````
- **EN**: This chunk defines `wrapper`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `wrapper`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 261-276 / 第 261-276 行
````python
    return wrapper


@register_dispatch_func(NATIVE_REDUCE_FNS + TORCH_REDUCE_FNS + TENSOR_REDUCE_FNS)
def _general_reduction(func, *args, **kwargs):
    return _apply_reduction(func, *args, **kwargs)


@register_dispatch_func(PASSTHROUGH_FNS)
def _general_passthrough(func, *args, **kwargs):
    return _apply_pass_through_fn(func, *args, **kwargs)


@register_dispatch_func(NATIVE_UNARY_FNS + NATIVE_INPLACE_UNARY_FNS)
def _general_unary(func, *args, **kwargs):
    return _apply_native_unary(func, *args, **kwargs)
````
- **EN**: This chunk defines `_general_unary`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_general_unary`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 279-296 / 第 279-296 行
````python
@register_dispatch_func(NATIVE_BINARY_FNS + NATIVE_INPLACE_BINARY_FNS)
def _general_binary(func, *args, **kwargs):
    return _apply_native_binary(func, *args, **kwargs)


@register_dispatch_func([torch.ops.aten.stride])
def stride(func, *args, **kwargs):
    return None


@register_dispatch_func([torch.ops.aten.sym_stride])
def sym_stride(func, *args, **kwargs):
    return None


@register_dispatch_func([torch.ops.prim.layout])
def layout(func, *args, **kwargs):
    return _get_data(args[0]).layout
````
- **EN**: This chunk defines `layout`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `layout`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 299-316 / 第 299-316 行
````python
@register_dispatch_func(
    [torch.ops.aten.is_contiguous, torch.ops.aten.sym_is_contiguous]
)
def is_contiguous(func, *args, **kwargs):
    data = _get_data(args[0])
    if data.is_sparse:
        raise ValueError("MaskedTensors with sparse data do not have is_contiguous")
    return func(data, *args[1:], **kwargs)


@register_dispatch_func([torch.ops.aten.is_strides_like_format])
def is_strides_like_format(func, *args, **kwargs):
    data = _get_data(args[0])
    if data.is_sparse:
        raise ValueError(
            "MaskedTensors with sparse data do not have is_strides_like_format"
        )
    return func(data, *args[1:], **kwargs)
````
- **EN**: This chunk defines `is_strides_like_format`, which checks a capability or invariant before later code relies on it. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_strides_like_format`，其作用是检查某项能力或不变量，供后续逻辑依赖。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 319-333 / 第 319-333 行
````python
@register_dispatch_func([torch.ops.aten.is_non_overlapping_and_dense])
def is_non_overlapping_and_dense(func, *args, **kwargs):
    data = _get_data(args[0])
    if data.is_sparse:
        raise ValueError(
            "MaskedTensors with sparse data do not have is_non_overlapping_and_dense"
        )
    return func(data, *args[1:], **kwargs)


@register_dispatch_func([torch.ops.aten.contiguous])
def contiguous(func, *args, **kwargs):
    if _get_data(args[0]).is_sparse:
        raise ValueError("MaskedTensors with sparse data do not have contiguous")
    return _MaskedContiguous.apply(args[0])
````
- **EN**: This chunk defines `contiguous`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `contiguous`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 336-349 / 第 336-349 行
````python
@register_dispatch_func([torch.ops.aten.new_empty_strided])
def new_empty_strided(func, *args, **kwargs):
    _check_args_kwargs_length(args, kwargs, f"__torch_dispatch__, {func}", len_args=3)
    data = _get_data(args[0])
    mask = _maybe_get_mask(args[0])
    if tuple(args[1]) != tuple(data.size()):
        raise ValueError(
            f"__torch_dispatch__, {func}: args[1] expected to be the same as data.size()"
        )
    if tuple(args[2]) != tuple(data.stride()):
        raise ValueError(
            f"__torch_dispatch__, {func}: args[2] expected to be the same as data.stride()"
        )
    return MaskedTensor(func(data, args[1], args[2], **kwargs), mask)
````
- **EN**: This chunk defines `new_empty_strided`, which drops cached state so a later execution phase can rebuild it cleanly. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `new_empty_strided`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 352-370 / 第 352-370 行
````python
@register_dispatch_func([torch.ops.aten._local_scalar_dense])
def _local_scalar_dense(func, *args, **kwargs):
    if not _maybe_get_mask(args[0]):
        raise ValueError(f"__torch_dispatch__, {func}: expected a mask tensor")
    return torch.ops.aten._local_scalar_dense(_get_data(args[0]))


@register_dispatch_func([torch.ops.aten.detach, torch.ops.aten.clone])
def _apply_fn_on_data(func, *args, **kwargs):
    return MaskedTensor(func(_get_data(args[0])), _maybe_get_mask(args[0]))


@register_dispatch_func([torch.ops.aten._to_copy])
def _to_copy(func, *args, **kwargs):
    new_data = func(_get_data(args[0]), *args[1:], **kwargs)
    cloned_kwargs = kwargs.copy()
    cloned_kwargs["dtype"] = torch.bool
    new_mask = func(_maybe_get_mask(args[0]), *args[1:], **cloned_kwargs)
    return MaskedTensor(new_data, new_mask)
````
- **EN**: This chunk defines `_to_copy`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_to_copy`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 373-388 / 第 373-388 行
````python
@register_dispatch_func([torch.ops.aten._softmax])
def _softmax(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=3, len_kwargs=0
    )
    data = _get_data(args[0])
    mask = _maybe_get_mask(args[0])
    result_data = torch.ops.aten._masked_softmax(data, ~mask, args[1], 2)
    return MaskedTensor(result_data, mask)


@register_dispatch_func([torch.ops.aten.ones_like])
def ones_like(func, *args, **kwargs):
    _check_args_kwargs_length(args, kwargs, f"__torch_dispatch__, {func}", len_args=1)
    result_data = func(_get_data(args[0]), **kwargs)
    return MaskedTensor(result_data, _maybe_get_mask(args[0]))
````
- **EN**: This chunk defines `ones_like`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ones_like`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 391-410 / 第 391-410 行
````python
@register_dispatch_func([torch.ops.aten._softmax_backward_data])
def _softmax_backward_data(func, *args, **kwargs):
    _check_args_kwargs_length(args, kwargs, f"__torch_dispatch__, {func}", len_args=4)
    grad, output, dim, _input_dtype = args
    if is_masked_tensor(grad) and is_masked_tensor(output):
        if not _masks_match(grad, output):
            raise ValueError(
                f"__torch_dispatch__, {func}: expected the masks of grad and output to match"
            )
        grad_data = _get_data(grad)
        new_grad_data = torch.ops.aten._masked_softmax_backward(
            grad_data,
            _get_data(output),
            ~_maybe_get_mask(grad),
            dim % grad_data.ndim,
        )
        res = MaskedTensor(new_grad_data, _maybe_get_mask(grad))
        return res
    else:
        raise ValueError(
````
- **EN**: This chunk defines `_softmax_backward_data`, which implements differentiation-time behavior that complements the forward path. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_softmax_backward_data`，其作用是实现与前向路径配套的求导期行为。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 411-430 / 第 411-430 行
````python
            f"__torch_dispatch__, {func}: grad and output must both be MaskedTensors"
        )


@register_dispatch_func([torch.ops.aten.copy_])
def copy_(func, *args, **kwargs):
    _check_args_kwargs_length(args, kwargs, f"__torch_dispatch__, {func}", len_args=2)
    if not _masks_match(_maybe_get_mask(args[0]), _maybe_get_mask(args[1])):
        raise ValueError("args[0] mask and args[1] mask must match but do not")
    func(_get_data(args[0]), _get_data(args[1]))
    return args[0]


@register_dispatch_func([torch.ops.aten.where])
def where(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=3, len_kwargs=0
    )
    if not torch.is_tensor(args[0]):
        raise ValueError(f"__torch_dispatch__, {func}: expected args[0] to be a tensor")
````
- **EN**: This chunk defines `where`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `where`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 431-450 / 第 431-450 行
````python
    mx = args[1]
    my = args[2]
    if not is_masked_tensor(mx):
        mx = MaskedTensor(mx, torch.ones_like(mx, dtype=torch.bool))
    if not is_masked_tensor(my):
        my = MaskedTensor(my, torch.ones_like(my, dtype=torch.bool))
    new_data = func(args[0], mx.get_data(), my.get_data())
    new_mask = func(args[0], mx.get_mask(), my.get_mask())
    return MaskedTensor(new_data, new_mask)


@register_dispatch_func([torch.ops.aten._to_sparse])
def _to_sparse(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    if not torch.is_tensor(args[0]):
        raise TypeError(f"__torch_dispatch__, {func}: expected args[0] to be a tensor")
    mt = args[0]
    if not is_masked_tensor(mt):
````
- **EN**: This chunk defines `_to_sparse`, which parses structured input into internal objects or validated metadata. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_to_sparse`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 451-470 / 第 451-470 行
````python
        mt = MaskedTensor(mt, torch.ones_like(mt, dtype=torch.bool))
    if mt.is_sparse_coo():
        return mt
    new_mask = func(_maybe_get_mask(args[0])).coalesce()
    new_data = _get_data(args[0]).sparse_mask(new_mask)
    return MaskedTensor(new_data, new_mask)


@register_dispatch_func([torch.ops.aten._to_sparse_csr])
def _to_sparse_csr(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    if not torch.is_tensor(args[0]):
        raise ValueError(f"__torch_dispatch__, {func}: expected args[0] to be a tensor")
    mt = args[0]
    if not is_masked_tensor(mt):
        mt = MaskedTensor(mt, torch.ones_like(mt).bool())
    if mt.is_sparse_csr():
        return mt
````
- **EN**: This chunk defines `_to_sparse_csr`, which parses structured input into internal objects or validated metadata. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_to_sparse_csr`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 471-488 / 第 471-488 行
````python
    new_mask = func(_maybe_get_mask(args[0]))
    new_data = _get_data(args[0]).sparse_mask(new_mask)
    return MaskedTensor(new_data, new_mask)


@register_dispatch_func([torch.ops.aten._to_dense])
def _to_dense(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    if not torch.is_tensor(args[0]):
        raise ValueError(f"__torch_dispatch__, {func}: expected args[0] to be a tensor")
    mt = args[0]
    if not is_masked_tensor(mt):
        mt = MaskedTensor(mt, torch.ones_like(mt).bool())
    new_data = func(_get_data(args[0]))
    new_mask = func(_maybe_get_mask(args[0]))
    return MaskedTensor(new_data, new_mask)
````
- **EN**: This chunk defines `_to_dense`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_to_dense`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 491-507 / 第 491-507 行
````python
@register_dispatch_func([torch.ops.aten._indices])
def _indices(func, *args, **kwargs):
    # Assumes data is sparse
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    data = _get_data(args[0]).indices()
    return MaskedTensor(data, torch.ones_like(data).bool())


@register_dispatch_func([torch.ops.aten._values])
def _values(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    data = _get_data(args[0]).values()
    return MaskedTensor(data, torch.ones_like(data).bool())
````
- **EN**: This chunk defines `_values`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_values`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 510-528 / 第 510-528 行
````python
@register_dispatch_func([torch.ops.aten._sparse_coo_tensor_with_dims_and_tensors])
def _sparse_coo_tensor_with_dims_and_tensors(func, *args, **kwargs):
    new_args = list(args)
    if is_masked_tensor(args[-1]):
        new_args[-1] = args[-1].get_data()
    if is_masked_tensor(args[-2]):
        new_args[-2] = args[-2].get_data()

    new_data = func(*new_args, **kwargs)
    new_args[-1] = torch.ones_like(new_args[-1])
    new_mask = func(*new_args, **kwargs).bool()

    return MaskedTensor(new_data, new_mask)


@register_dispatch_func([torch.ops.aten.is_same_size])
def is_same_size(func, *args, **kwargs):
    _check_args_kwargs_length(args, kwargs, f"__torch_dispatch__, {func}", len_args=2)
    return _get_data(args[0]).is_same_size(_get_data(args[1]))
````
- **EN**: This chunk defines `is_same_size`, which checks a capability or invariant before later code relies on it. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_same_size`，其作用是检查某项能力或不变量，供后续逻辑依赖。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 531-547 / 第 531-547 行
````python
@register_dispatch_func([torch.ops.aten._is_any_true])
def _is_any_true(func, *args, **kwargs):
    _check_args_kwargs_length(
        args, kwargs, f"__torch_dispatch__, {func}", len_args=1, len_kwargs=0
    )
    data = _get_data(args[0])
    mask = _maybe_get_mask(args[0])
    if mask is None:
        raise ValueError(
            f"__torch_dispatch__, {func}: expected args[0] to be a MaskedTensor"
        )
    if data.dtype != torch.bool:
        raise ValueError(f"__torch_dispatch__, {func}: expected a boolean tensor")
    if data.is_sparse:
        raise ValueError(f"MaskedTensors with sparse data do not have {func}")

    return MaskedTensor(func(data & mask), torch.tensor(True))
````
- **EN**: This chunk defines `_is_any_true`, which implements a focused helper used by the surrounding module. Decorators such as `register_dispatch_func` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_is_any_true`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_dispatch_func` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Masked operations**
  - EN: Represents operations that carry masks alongside data and preserve masked semantics.
  - CN: 表示同时携带 mask 与数据的运算，并保持 masked 语义。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_check_args_kwargs_length**
  - EN: `_check_args_kwargs_length` is one of the main symbols declared or implemented in this file.
  - CN: `_check_args_kwargs_length` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.binary`, `.core`, `.passthrough`, `.reductions`, `.unary`, `torch._ops`
- **Standard library / 标准库**: `collections.abc`, `functools`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_check_args_kwargs_length`, `_MaskedContiguous`, `_MaskedToDense`, `_MaskedToSparse`, `_MaskedToSparseCsr`, `_MaskedWhere`, `_MASKEDTENSOR_FUNCTION_TABLE`, `register_function_func`, `_general_function_reductions`
