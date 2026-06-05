# core.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/masked/maskedtensor/core.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements masked-tensor helpers and namespace wrappers for masked operations.
- **Purpose (CN)**: 实现 masked tensor 辅助逻辑以及 masked 运算的命名空间包装层。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates

import warnings
from typing import Any
from typing_extensions import TypeIs

import torch
from torch.overrides import get_default_nowrap_functions


__all__ = [
    "MaskedTensor",
    "is_masked_tensor",
]


def is_masked_tensor(obj: Any, /) -> TypeIs["MaskedTensor"]:
    r"""Returns True if the input is a MaskedTensor, else False
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.overrides; standard-library helpers such as warnings, typing; other helper packages such as typing_extensions. `__all__` defines the public symbols that this module chooses to export. This chunk defines `is_masked_tensor`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.overrides；标准库辅助模块，如 warnings、typing；其他辅助包，如 typing_extensions。 `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `is_masked_tensor`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 21-34 / 第 21-34 行
````python
    Args:
        a: any input

    Examples:

        >>> # xdoctest: +SKIP
        >>> from torch.masked import MaskedTensor
        >>> data = torch.arange(6).reshape(2, 3)
        >>> mask = torch.tensor([[True, False, False], [True, True, False]])
        >>> mt = MaskedTensor(data, mask)
        >>> is_masked_tensor(mt)
        True
    """
    return isinstance(obj, MaskedTensor)
````
- **EN**: This chunk continues `is_masked_tensor` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `is_masked_tensor`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 37-56 / 第 37-56 行
````python
def _tensors_match(a, b, exact=True, rtol=1e-05, atol=1e-08):
    if is_masked_tensor(a) or is_masked_tensor(b):
        raise ValueError("Neither `a` nor `b` can be a MaskedTensor.")
    if a.layout != b.layout:
        raise ValueError(
            f"`a` and `b` must have the same layout. Got {a.layout} and {b.layout}"
        )

    if a.dtype != b.dtype:
        b = b.type(a.dtype)
    if a.layout == b.layout == torch.sparse_coo:
        return _tensors_match(a.values(), b.values(), exact) and _tensors_match(
            a.indices(), b.indices(), exact
        )
    elif a.layout == b.layout == torch.sparse_csr:
        return (
            _tensors_match(a.crow_indices(), b.crow_indices(), exact)
            and _tensors_match(a.col_indices(), b.col_indices(), exact)
            and _tensors_match(a.values(), b.values(), exact)
        )
````
- **EN**: This chunk defines `_tensors_match`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_tensors_match`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-76 / 第 57-76 行
````python
    if exact:
        return (a.dim() == b.dim()) and torch.eq(a, b).all().item()
    return (a.dim() == b.dim()) and torch.allclose(a, b, rtol=rtol, atol=atol)


def _masks_match(a, b):
    if is_masked_tensor(a) and is_masked_tensor(b):
        mask_a = a.get_mask()
        mask_b = b.get_mask()
        return _tensors_match(mask_a, mask_b, exact=True)
    return True


def _map_mt_args_kwargs(args, kwargs, map_fn):
    def _helper(a, map_fn):
        if is_masked_tensor(a):
            return map_fn(a)
        elif torch.is_tensor(a):
            return a
        elif isinstance(a, list):
````
- **EN**: This chunk defines `_helper`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_helper`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 77-93 / 第 77-93 行
````python
            a_impl, _ = _map_mt_args_kwargs(a, {}, map_fn)
            return a_impl
        elif isinstance(a, tuple):
            a_impl, _ = _map_mt_args_kwargs(a, {}, map_fn)
            return tuple(a_impl)
        else:
            return a

    if kwargs is None:
        kwargs = {}
    impl_args = []
    for a in args:
        impl_args.append(_helper(a, map_fn))
    impl_kwargs = {}
    for k in kwargs:
        impl_kwargs[k] = _helper(a, map_fn)
    return impl_args, impl_kwargs
````
- **EN**: This chunk continues `_helper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_helper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 96-115 / 第 96-115 行
````python
def _wrap_result(result_data, result_mask):
    if isinstance(result_data, list):
        return [_wrap_result(r, m) for (r, m) in zip(result_data, result_mask)]
    if isinstance(result_data, tuple):
        return tuple(_wrap_result(r, m) for (r, m) in zip(result_data, result_mask))
    if torch.is_tensor(result_data):
        return MaskedTensor(result_data, result_mask)
    # Expect result_data and result_mask to be Tensors only
    return NotImplemented


def _masked_tensor_str(data, mask, formatter):
    if data.layout in {torch.sparse_coo, torch.sparse_csr}:
        data = data.to_dense()
        mask = mask.to_dense()
    if data.dim() == 1:
        formatted_elements = [
            formatter.format(d.item()) if isinstance(d.item(), float) else str(d.item())
            for d in data
        ]
````
- **EN**: This chunk defines `_masked_tensor_str`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_masked_tensor_str`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 116-129 / 第 116-129 行
````python
        max_len = max(8 if x[1] else len(x[0]) for x in zip(formatted_elements, ~mask))
        return (
            "["
            + ", ".join(
                [
                    "--".rjust(max_len) if m else e
                    for (e, m) in zip(formatted_elements, ~mask)
                ]
            )
            + "]"
        )
    sub_strings = [_masked_tensor_str(d, m, formatter) for (d, m) in zip(data, mask)]
    sub_strings = ["\n".join(["  " + si for si in s.split("\n")]) for s in sub_strings]
    return "[\n" + ",\n".join(sub_strings) + "\n]"
````
- **EN**: This chunk continues `_masked_tensor_str` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_masked_tensor_str`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 132-151 / 第 132-151 行
````python
def _get_data(a):
    if is_masked_tensor(a):
        return a._masked_data
    return a


def _maybe_get_mask(a):
    if is_masked_tensor(a):
        return a.get_mask()
    return None


class MaskedTensor(torch.Tensor):
    @staticmethod
    def __new__(cls, data, mask, requires_grad=False):
        if is_masked_tensor(data) or not torch.is_tensor(data):
            raise TypeError("data must be a Tensor")
        if is_masked_tensor(mask) or not torch.is_tensor(mask):
            raise TypeError("mask must be a Tensor")
        # Use a Tensor that of the give size for the wrapper.
````
- **EN**: It introduces or extends `MaskedTensor`, which hold the main object-oriented state for this portion of the file. This chunk defines `__new__`, which initializes object state and binds construction-time configuration. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `MaskedTensor`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__new__`，其作用是初始化对象状态并绑定构造期配置。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 152-171 / 第 152-171 行
````python
        kwargs = {
            "device": data.device,
            "dtype": data.dtype,
            "layout": data.layout,
            "requires_grad": requires_grad,
            "dispatch_sizes_strides_policy": "strides",
            "dispatch_layout": True,
        }
        warnings.warn(
            (
                "The PyTorch API of MaskedTensors is in prototype stage "
                "and will change in the near future. Please open a Github issue "
                "for features requests and see our documentation on the torch.masked "
                "module for further information about the project."
            ),
            UserWarning,
            stacklevel=2,
        )
        if data.requires_grad:
            warnings.warn(
````
- **EN**: This chunk continues `MaskedTensor` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `MaskedTensor`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 172-191 / 第 172-191 行
````python
                "It is not recommended to create a MaskedTensor with a tensor that requires_grad. "
                "To avoid this, you can use data.detach().clone()",
                UserWarning,
                stacklevel=2,
            )
        # pyrefly: ignore [bad-argument-type]
        return torch.Tensor._make_wrapper_subclass(cls, data.size(), **kwargs)

    def _preprocess_data(self, data, mask):
        from .._ops import _sparse_coo_where, _sparse_csr_where

        if data.layout != mask.layout:
            raise TypeError("data and mask must have the same layout.")
        if data.layout == torch.sparse_coo:
            data = data.coalesce()
            mask = mask.coalesce()
            if data._nnz() != mask._nnz():
                data = _sparse_coo_where(mask, data, torch.tensor(0))
        elif data.layout == torch.sparse_csr:
            if data._nnz() != mask._nnz():
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as .._ops. This chunk defines `_preprocess_data`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 .._ops。 这一段定义了 `_preprocess_data`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 192-211 / 第 192-211 行
````python
                data = _sparse_csr_where(mask, data, torch.tensor(0))

        # Have to pick awkward names to not conflict with existing fields such as data
        self._masked_data = data.clone()
        self._masked_mask = mask.clone()

    def _validate_members(self):
        data = self._masked_data
        mask = self.get_mask()
        if type(data) is not type(mask):
            raise TypeError(
                f"data and mask must have the same type. Got {type(data)} and {type(mask)}"
            )
        if data.layout not in {torch.strided, torch.sparse_coo, torch.sparse_csr}:
            raise TypeError(f"data layout of {data.layout} is not supported.")
        if data.layout == torch.sparse_coo:
            if not _tensors_match(data.indices(), mask.indices(), exact=True):
                raise ValueError(
                    "data and mask are both sparse COO tensors but do not have the same indices."
                )
````
- **EN**: This chunk defines `_validate_members`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_validate_members`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 212-231 / 第 212-231 行
````python
        elif data.layout == torch.sparse_csr:
            if not _tensors_match(
                data.crow_indices(), mask.crow_indices(), exact=True
            ) or not _tensors_match(data.col_indices(), mask.col_indices(), exact=True):
                raise ValueError(
                    "data and mask are both sparse CSR tensors but do not share either crow or col indices."
                )
        if mask.dtype != torch.bool:
            raise TypeError("mask must have dtype bool.")
        if not (
            data.dtype == torch.float16
            or data.dtype == torch.float32
            or data.dtype == torch.float64
            or data.dtype == torch.bool
            or data.dtype == torch.int8
            or data.dtype == torch.int16
            or data.dtype == torch.int32
            or data.dtype == torch.int64
        ):
            raise TypeError(f"{data.dtype} is not supported in MaskedTensor.")
````
- **EN**: This chunk continues `_validate_members` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_validate_members`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 232-249 / 第 232-249 行
````python
        if data.dim() != mask.dim():
            raise ValueError("data.dim() must equal mask.dim()")
        if data.size() != mask.size():
            raise ValueError("data.size() must equal mask.size()")

    def __init__(self, data, mask, requires_grad=False):
        self._preprocess_data(data, mask)
        self._validate_members()

    @staticmethod
    def _from_values(data, mask):
        """Differentiable constructor for MaskedTensor"""

        class Constructor(torch.autograd.Function):
            @staticmethod
            # pyrefly: ignore [bad-override]
            def forward(ctx, data, mask):
                return MaskedTensor(data, mask)
````
- **EN**: It introduces or extends `Constructor`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `Constructor`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 251-270 / 第 251-270 行
````python
            @staticmethod
            # pyrefly: ignore [bad-override]
            def backward(ctx, grad_output):
                return grad_output, None

        result = Constructor.apply(data, mask)
        return result

    def _set_data_mask(self, data, mask):
        self._masked_data = data
        self._masked_mask = mask
        self._validate_members()

    def __repr__(self):  # type: ignore[override]
        formatter = "{0:8.4f}"
        if self.dim() == 0:
            scalar_data = self.get_data().item()
            data_formatted = (
                formatter.format(scalar_data)
                if isinstance(scalar_data, float)
````
- **EN**: This chunk defines `__repr__`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__repr__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 271-289 / 第 271-289 行
````python
                else str(scalar_data)
            )
            if not self.get_mask().item():
                data_formatted = "--"
            return (
                "MaskedTensor("
                + data_formatted
                + ", "
                + str(self.get_mask().item())
                + ")"
            )
        s = _masked_tensor_str(self.get_data(), self.get_mask(), formatter)
        s = "\n".join("  " + si for si in s.split("\n"))
        return "MaskedTensor(\n" + s + "\n)"

    # Seems like this needs to be defined before torch_dispatch to work
    @classmethod
    def __torch_function__(cls, func, types, args=(), kwargs=None):
        kwargs = kwargs or {}
````
- **EN**: This chunk defines `__torch_function__`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__torch_function__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 291-307 / 第 291-307 行
````python
        from ._ops_refs import _MASKEDTENSOR_FUNCTION_TABLE

        if func in _MASKEDTENSOR_FUNCTION_TABLE:
            return _MASKEDTENSOR_FUNCTION_TABLE[func](*args, **kwargs)

        if not all(issubclass(cls, t) for t in types):
            return NotImplemented
        with torch._C.DisableTorchFunctionSubclass():
            ret = func(*args, **kwargs)
            if func in get_default_nowrap_functions():
                return ret
            else:
                return torch._tensor._convert(ret, cls)

    @classmethod
    def unary(cls, fn, data, mask):
        return MaskedTensor(fn(data), mask)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ops_refs. This chunk defines `unary`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ops_refs。 这一段定义了 `unary`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 309-326 / 第 309-326 行
````python
    @classmethod
    def __torch_dispatch__(cls, func, types, args, kwargs):  # type: ignore[override]
        func = func.overloadpacket

        from ._ops_refs import _MASKEDTENSOR_DISPATCH_TABLE

        if func in _MASKEDTENSOR_DISPATCH_TABLE:
            return _MASKEDTENSOR_DISPATCH_TABLE[func](*args, **kwargs)

        msg = (
            f"{func.__name__} is not implemented in __torch_dispatch__ for MaskedTensor.\n"
            "If you would like this operator to be supported, please file an issue for a feature request at "
            "https://github.com/pytorch/maskedtensor/issues with a minimal reproducible code snippet.\n"
            "In the case that the semantics for the operator are not trivial, it would be appreciated "
            "to also include a proposal for the semantics."
        )
        warnings.warn(msg, stacklevel=2)
        return NotImplemented
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ops_refs. This chunk defines `__torch_dispatch__`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ops_refs。 这一段定义了 `__torch_dispatch__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 328-341 / 第 328-341 行
````python
    def __lt__(self, other):
        if is_masked_tensor(other):
            return MaskedTensor(self.get_data() < _get_data(other), self.get_mask())
        return MaskedTensor(self.get_data() < other, self.get_mask())

    def to_tensor(self, value):
        return self.get_data().masked_fill(~self.get_mask(), value)

    def get_data(self):
        class GetData(torch.autograd.Function):
            @staticmethod
            # pyrefly: ignore [bad-override]
            def forward(ctx, self):
                return self._masked_data.detach()
````
- **EN**: It introduces or extends `GetData`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `GetData`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 343-359 / 第 343-359 行
````python
            @staticmethod
            # pyrefly: ignore [bad-override]
            def backward(ctx, grad_output):
                if is_masked_tensor(grad_output):
                    return grad_output
                return MaskedTensor(grad_output, self.get_mask())

        return GetData.apply(self)

    def get_mask(self):
        return self._masked_mask

    def is_sparse_coo(self):
        return self.layout == torch.sparse_coo

    def is_sparse_csr(self):  # type: ignore[override]
        return self.layout == torch.sparse_csr
````
- **EN**: This chunk defines `is_sparse_csr`, which checks a capability or invariant before later code relies on it. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_sparse_csr`，其作用是检查某项能力或不变量，供后续逻辑依赖。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 361-364 / 第 361-364 行
````python
    # Update later to support more sparse layouts
    @property
    def is_sparse(self):  # type: ignore[override]
        return self.is_sparse_coo() or self.is_sparse_csr()
````
- **EN**: This chunk defines `is_sparse`, which checks a capability or invariant before later code relies on it. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_sparse`，其作用是检查某项能力或不变量，供后续逻辑依赖。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Masked operations**
  - EN: Represents operations that carry masks alongside data and preserve masked semantics.
  - CN: 表示同时携带 mask 与数据的运算，并保持 masked 语义。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **is_masked_tensor**
  - EN: `is_masked_tensor` is one of the main symbols declared or implemented in this file.
  - CN: `is_masked_tensor` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.overrides`, `.._ops`, `._ops_refs`
- **Standard library / 标准库**: `warnings`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `is_masked_tensor`, `_tensors_match`, `_masks_match`, `_map_mt_args_kwargs`, `_wrap_result`, `_masked_tensor_str`, `_get_data`, `_maybe_get_mask`, `MaskedTensor`
