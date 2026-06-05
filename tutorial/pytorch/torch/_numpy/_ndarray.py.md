# _ndarray.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_numpy/_ndarray.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements NumPy-compatibility helpers, wrappers, and interoperability utilities.
- **Purpose (CN)**: 实现 NumPy 兼容辅助逻辑、包装层以及互操作工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行
````python
# mypy: ignore-errors

from __future__ import annotations

import builtins
import math
import operator
from collections.abc import Sequence

import torch

from . import _dtypes, _dtypes_impl, _funcs, _ufuncs, _util
from ._normalizations import (
    ArrayLike,
    normalize_array_like,
    normalizer,
    NotImplementedType,
)


newaxis = None
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, ., ._normalizations; standard-library helpers such as __future__, builtins, math, .... The future import postpones annotation evaluation, keeping type hints lightweight at import time.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.、._normalizations；标准库辅助模块，如 __future__、builtins、math、...。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。

### Lines 23-47 / 第 23-47 行
````python
FLAGS = [
    "C_CONTIGUOUS",
    "F_CONTIGUOUS",
    "OWNDATA",
    "WRITEABLE",
    "ALIGNED",
    "WRITEBACKIFCOPY",
    "FNC",
    "FORC",
    "BEHAVED",
    "CARRAY",
    "FARRAY",
]

SHORTHAND_TO_FLAGS = {
    "C": "C_CONTIGUOUS",
    "F": "F_CONTIGUOUS",
    "O": "OWNDATA",
    "W": "WRITEABLE",
    "A": "ALIGNED",
    "X": "WRITEBACKIFCOPY",
    "B": "BEHAVED",
    "CA": "CARRAY",
    "FA": "FARRAY",
}
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 50-72 / 第 50-72 行
````python
class Flags:
    def __init__(self, flag_to_value: dict):
        invalid_keys = [k for k in flag_to_value if k not in FLAGS]
        if invalid_keys:
            raise AssertionError(f"Invalid flag keys: {invalid_keys}")
        self._flag_to_value = flag_to_value

    def __getattr__(self, attr: str):
        if attr.islower() and attr.upper() in FLAGS:
            return self[attr.upper()]
        else:
            raise AttributeError(f"No flag attribute '{attr}'")

    def __getitem__(self, key):
        if key in SHORTHAND_TO_FLAGS:
            key = SHORTHAND_TO_FLAGS[key]
        if key in FLAGS:
            try:
                return self._flag_to_value[key]
            except KeyError as e:
                raise NotImplementedError(f"{key=}") from e
        else:
            raise KeyError(f"No flag key '{key}'")
````
- **EN**: It introduces or extends `Flags`, which hold the main object-oriented state for this portion of the file. This chunk defines `__getitem__`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `Flags`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__getitem__`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 74-95 / 第 74-95 行
````python
    def __setattr__(self, attr, value):
        if attr.islower() and attr.upper() in FLAGS:
            self[attr.upper()] = value
        else:
            super().__setattr__(attr, value)

    def __setitem__(self, key, value):
        if key in FLAGS or key in SHORTHAND_TO_FLAGS:
            raise NotImplementedError("Modifying flags is not implemented")
        else:
            raise KeyError(f"No flag key '{key}'")


def create_method(fn, name=None):
    name = name or fn.__name__

    def f(*args, **kwargs):
        return fn(*args, **kwargs)

    f.__name__ = name
    f.__qualname__ = f"ndarray.{name}"
    return f
````
- **EN**: This chunk defines `f`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `f`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 98-125 / 第 98-125 行
````python
# Map ndarray.name_method -> np.name_func
# If name_func == None, it means that name_method == name_func
methods = {
    "clip": None,
    "nonzero": None,
    "repeat": None,
    "round": None,
    "squeeze": None,
    "swapaxes": None,
    "ravel": None,
    # linalg
    "diagonal": None,
    "dot": None,
    "trace": None,
    # sorting
    "argsort": None,
    "searchsorted": None,
    # reductions
    "argmax": None,
    "argmin": None,
    "any": None,
    "all": None,
    "max": None,
    "min": None,
    "ptp": None,
    "sum": None,
    "prod": None,
    "mean": None,
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 126-145 / 第 126-145 行
````python
    "var": None,
    "std": None,
    # scans
    "cumsum": None,
    "cumprod": None,
    # advanced indexing
    "take": None,
    "choose": None,
}

dunder = {
    "abs": "absolute",
    "invert": None,
    "pos": "positive",
    "neg": "negative",
    "gt": "greater",
    "lt": "less",
    "ge": "greater_equal",
    "le": "less_equal",
}
````
- **EN**: This chunk continues `f` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `f`，进一步展开其内部控制流或状态更新。

### Lines 147-171 / 第 147-171 行
````python
# dunder methods with right-looking and in-place variants
ri_dunder = {
    "add": None,
    "sub": "subtract",
    "mul": "multiply",
    "truediv": "divide",
    "floordiv": "floor_divide",
    "pow": "power",
    "mod": "remainder",
    "and": "bitwise_and",
    "or": "bitwise_or",
    "xor": "bitwise_xor",
    "lshift": "left_shift",
    "rshift": "right_shift",
    "matmul": None,
}


def _upcast_int_indices(index):
    if isinstance(index, torch.Tensor):
        if index.dtype in (torch.int8, torch.int16, torch.int32, torch.uint8):
            return index.to(torch.int64)
    elif isinstance(index, tuple):
        return tuple(_upcast_int_indices(i) for i in index)
    return index
````
- **EN**: This chunk defines `_upcast_int_indices`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_upcast_int_indices`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 174-192 / 第 174-192 行
````python
def _has_advanced_indexing(index):
    """Check if there's any advanced indexing"""
    return any(
        isinstance(idx, (Sequence, bool))
        or (isinstance(idx, torch.Tensor) and (idx.dtype == torch.bool or idx.ndim > 0))
        for idx in index
    )


def _numpy_compatible_indexing(index):
    """Convert scalar indices to lists when advanced indexing is present for NumPy compatibility."""
    if not isinstance(index, tuple):
        index = (index,)

    # Check if there's any advanced indexing (sequences, booleans, or tensors)
    has_advanced = _has_advanced_indexing(index)

    if not has_advanced:
        return index
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_numpy_compatible_indexing`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_numpy_compatible_indexing`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 194-213 / 第 194-213 行
````python
    # Convert integer scalar indices to single-element lists when advanced indexing is present
    # Note: Do NOT convert boolean scalars (True/False) as they have special meaning in NumPy
    converted = []
    for idx in index:
        if isinstance(idx, int) and not isinstance(idx, bool):
            # Integer scalars should be converted to lists
            converted.append([idx])
        elif (
            isinstance(idx, torch.Tensor)
            and idx.ndim == 0
            and not torch.is_floating_point(idx)
            and idx.dtype != torch.bool
        ):
            # Zero-dimensional tensors holding integers should be treated the same as integer scalars
            converted.append([idx])
        else:
            # Everything else (booleans, lists, slices, etc.) stays as is
            converted.append(idx)

    return tuple(converted)
````
- **EN**: This chunk continues `_numpy_compatible_indexing` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_numpy_compatible_indexing`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 216-240 / 第 216-240 行
````python
def _get_bool_depth(s):
    """Returns the depth of a boolean sequence/tensor"""
    if isinstance(s, bool):
        return True, 0
    if isinstance(s, torch.Tensor) and s.dtype == torch.bool:
        return True, s.ndim
    if not (isinstance(s, Sequence) and s and s[0] != s):
        return False, 0
    is_bool, depth = _get_bool_depth(s[0])
    return is_bool, depth + 1


def _numpy_empty_ellipsis_patch(index, tensor_ndim):
    """
    Patch for NumPy-compatible ellipsis behavior when ellipsis doesn't match any dimensions.

    In NumPy, when an ellipsis (...) doesn't actually match any dimensions of the input array,
    it still acts as a separator between advanced indices. PyTorch doesn't have this behavior.

    This function detects when we have:
    1. Advanced indexing on both sides of an ellipsis
    2. The ellipsis doesn't actually match any dimensions
    """
    if not isinstance(index, tuple):
        index = (index,)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_numpy_empty_ellipsis_patch`, which drops cached state so a later execution phase can rebuild it cleanly. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_numpy_empty_ellipsis_patch`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 242-265 / 第 242-265 行
````python
    # Find ellipsis position
    ellipsis_pos = None
    for i, idx in enumerate(index):
        if idx is Ellipsis:
            ellipsis_pos = i
            break

    # If no ellipsis, no patch needed
    if ellipsis_pos is None:
        return index, lambda x: x, lambda x: x

    # Count non-ellipsis dimensions consumed by the index
    consumed_dims = 0
    for idx in index:
        is_bool, depth = _get_bool_depth(idx)
        if is_bool:
            consumed_dims += depth
        elif idx is Ellipsis or idx is None:
            continue
        else:
            consumed_dims += 1

    # Calculate how many dimensions the ellipsis should match
    ellipsis_dims = tensor_ndim - consumed_dims
````
- **EN**: This chunk continues `_numpy_empty_ellipsis_patch` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_numpy_empty_ellipsis_patch`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 267-291 / 第 267-291 行
````python
    # Check if ellipsis doesn't match any dimensions
    if ellipsis_dims == 0:
        # Check if we have advanced indexing on both sides of ellipsis
        left_advanced = _has_advanced_indexing(index[:ellipsis_pos])
        right_advanced = _has_advanced_indexing(index[ellipsis_pos + 1 :])

        if left_advanced and right_advanced:
            # This is the case where NumPy and PyTorch differ
            # We need to ensure the advanced indices are treated as separated
            new_index = index[:ellipsis_pos] + (None,) + index[ellipsis_pos + 1 :]
            end_ndims = 1 + sum(
                1 for idx in index[ellipsis_pos + 1 :] if isinstance(idx, slice)
            )

            def squeeze_fn(x):
                return x.squeeze(-end_ndims)

            def unsqueeze_fn(x):
                if isinstance(x, torch.Tensor) and x.ndim >= end_ndims:
                    return x.unsqueeze(-end_ndims)
                return x

            return new_index, squeeze_fn, unsqueeze_fn

    return index, lambda x: x, lambda x: x
````
- **EN**: This chunk defines `unsqueeze_fn`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unsqueeze_fn`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 294-317 / 第 294-317 行
````python
# Used to indicate that a parameter is unspecified (as opposed to explicitly
# `None`)
class _Unspecified:
    pass


_Unspecified.unspecified = _Unspecified()

###############################################################
#                      ndarray class                          #
###############################################################


class ndarray:
    def __init__(self, t=None):
        if t is None:
            self.tensor = torch.Tensor()
        elif isinstance(t, torch.Tensor):
            self.tensor = t
        else:
            raise ValueError(
                "ndarray constructor is not recommended; prefer"
                "either array(...) or zeros/empty(...)"
            )
````
- **EN**: It introduces or extends `_Unspecified`, `ndarray`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 它引入或扩展了 `_Unspecified`、`ndarray`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 319-342 / 第 319-342 行
````python
    # Register NumPy functions as methods
    for method, name in methods.items():
        fn = getattr(_funcs, name or method)
        vars()[method] = create_method(fn, method)

    # Regular methods but coming from ufuncs
    conj = create_method(_ufuncs.conjugate, "conj")
    conjugate = create_method(_ufuncs.conjugate)

    for method, name in dunder.items():
        fn = getattr(_ufuncs, name or method)
        method = f"__{method}__"
        vars()[method] = create_method(fn, method)

    for method, name in ri_dunder.items():
        fn = getattr(_ufuncs, name or method)
        plain = f"__{method}__"
        vars()[plain] = create_method(fn, plain)
        rvar = f"__r{method}__"
        vars()[rvar] = create_method(lambda self, other, fn=fn: fn(other, self), rvar)
        ivar = f"__i{method}__"
        vars()[ivar] = create_method(
            lambda self, other, fn=fn: fn(self, other, out=self), ivar
        )
````
- **EN**: This chunk continues `ndarray` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `ndarray`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 344-367 / 第 344-367 行
````python
    # There's no __idivmod__
    __divmod__ = create_method(_ufuncs.divmod, "__divmod__")
    __rdivmod__ = create_method(
        lambda self, other: _ufuncs.divmod(other, self), "__rdivmod__"
    )

    # prevent loop variables leaking into the ndarray class namespace
    del ivar, rvar, name, plain, fn, method

    @property
    def shape(self):
        return tuple(self.tensor.shape)

    @property
    def size(self):
        return self.tensor.numel()

    @property
    def ndim(self):
        return self.tensor.ndim

    @property
    def dtype(self):
        return _dtypes.dtype(self.tensor.dtype)
````
- **EN**: This chunk defines `dtype`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `dtype`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 369-392 / 第 369-392 行
````python
    @property
    def strides(self):
        elsize = self.tensor.element_size()
        return tuple(stride * elsize for stride in self.tensor.stride())

    @property
    def itemsize(self):
        return self.tensor.element_size()

    @property
    def flags(self):
        # Note contiguous in torch is assumed C-style
        return Flags(
            {
                "C_CONTIGUOUS": self.tensor.is_contiguous(),
                "F_CONTIGUOUS": self.T.tensor.is_contiguous(),
                "OWNDATA": self.tensor._base is None,
                "WRITEABLE": True,  # pytorch does not have readonly tensors
            }
        )

    @property
    def data(self):
        return self.tensor.data_ptr()
````
- **EN**: This chunk defines `data`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `data`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 394-420 / 第 394-420 行
````python
    @property
    def nbytes(self):
        return self.tensor.storage().nbytes()

    @property
    def T(self):
        return self.transpose()

    @property
    def real(self):
        return _funcs.real(self)

    @real.setter
    def real(self, value):
        self.tensor.real = asarray(value).tensor

    @property
    def imag(self):
        return _funcs.imag(self)

    @imag.setter
    def imag(self, value):
        self.tensor.imag = asarray(value).tensor

    @property
    def flat(self):
        return self.ravel()
````
- **EN**: This chunk defines `flat`, which implements a focused helper used by the surrounding module. Decorators such as `property`, `real.setter`, `imag.setter` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `flat`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property`、`real.setter`、`imag.setter` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 422-444 / 第 422-444 行
````python
    # ctors
    def astype(self, dtype, order="K", casting="unsafe", subok=True, copy=True):
        if order != "K":
            raise NotImplementedError(f"astype(..., order={order} is not implemented.")
        if casting != "unsafe":
            raise NotImplementedError(
                f"astype(..., casting={casting} is not implemented."
            )
        if not subok:
            raise NotImplementedError(f"astype(..., subok={subok} is not implemented.")
        if not copy:
            raise NotImplementedError(f"astype(..., copy={copy} is not implemented.")
        torch_dtype = _dtypes.dtype(dtype).torch_dtype
        t = self.tensor.to(torch_dtype)
        return ndarray(t)

    @normalizer
    def copy(self: ArrayLike, order: NotImplementedType = "C"):
        return self.clone()

    @normalizer
    def flatten(self: ArrayLike, order: NotImplementedType = "C"):
        return torch.flatten(self)
````
- **EN**: This chunk defines `flatten`, which implements a focused helper used by the surrounding module. Decorators such as `normalizer` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `flatten`，其作用是实现周边模块使用的关键辅助逻辑。 像 `normalizer` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 446-466 / 第 446-466 行
````python
    def resize(self, *new_shape, refcheck=False):
        # NB: differs from np.resize: fills with zeros instead of making repeated copies of input.
        if refcheck:
            raise NotImplementedError(
                f"resize(..., refcheck={refcheck} is not implemented."
            )
        if new_shape in [(), (None,)]:
            return

        # support both x.resize((2, 2)) and x.resize(2, 2)
        if len(new_shape) == 1:
            new_shape = new_shape[0]
        if isinstance(new_shape, int):
            new_shape = (new_shape,)

        if builtins.any(x < 0 for x in new_shape):
            raise ValueError("all elements of `new_shape` must be non-negative")

        new_numel, old_numel = math.prod(new_shape), self.tensor.numel()

        self.tensor.resize_(new_shape)
````
- **EN**: This chunk defines `resize`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `resize`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 468-494 / 第 468-494 行
````python
        if new_numel >= old_numel:
            # zero-fill new elements
            if not self.tensor.is_contiguous():
                raise AssertionError("tensor must be contiguous for resize with growth")
            b = self.tensor.flatten()  # does not copy
            b[old_numel:].zero_()

    def view(self, dtype=_Unspecified.unspecified, type=_Unspecified.unspecified):
        if dtype is _Unspecified.unspecified:
            dtype = self.dtype
        if type is not _Unspecified.unspecified:
            raise NotImplementedError(f"view(..., type={type} is not implemented.")
        torch_dtype = _dtypes.dtype(dtype).torch_dtype
        tview = self.tensor.view(torch_dtype)
        return ndarray(tview)

    @normalizer
    def fill(self, value: ArrayLike):
        # Both Pytorch and NumPy accept 0D arrays/tensors and scalars, and
        # error out on D > 0 arrays
        self.tensor.fill_(value)

    def tolist(self):
        return self.tensor.tolist()

    def __iter__(self):
        return (ndarray(x) for x in self.tensor.__iter__())
````
- **EN**: This chunk defines `__iter__`, which implements a focused helper used by the surrounding module. Decorators such as `normalizer` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__iter__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `normalizer` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 496-522 / 第 496-522 行
````python
    def __str__(self):
        return (
            str(self.tensor)
            .replace("tensor", "torch.ndarray")
            .replace("dtype=torch.", "dtype=")
        )

    __repr__ = create_method(__str__)

    def __eq__(self, other):
        try:
            return _ufuncs.equal(self, other)
        except (RuntimeError, TypeError):
            # Failed to convert other to array: definitely not equal.
            falsy = torch.full(self.shape, fill_value=False, dtype=bool)
            return asarray(falsy)

    def __ne__(self, other):
        return ~(self == other)

    def __index__(self):
        try:
            return operator.index(self.tensor.item())
        except Exception as exc:
            raise TypeError(
                "only integer scalar arrays can be converted to a scalar index"
            ) from exc
````
- **EN**: This chunk defines `__index__`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__index__`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 524-548 / 第 524-548 行
````python
    def __bool__(self):
        return bool(self.tensor)

    def __int__(self):
        return int(self.tensor)

    def __float__(self):
        return float(self.tensor)

    def __complex__(self):
        return complex(self.tensor)

    def is_integer(self):
        try:
            v = self.tensor.item()
            result = int(v) == v
        except Exception:
            result = False
        return result

    def __len__(self):
        return self.tensor.shape[0]

    def __contains__(self, x):
        return self.tensor.__contains__(x)
````
- **EN**: This chunk defines `__contains__`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__contains__`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 550-575 / 第 550-575 行
````python
    def transpose(self, *axes):
        # np.transpose(arr, axis=None) but arr.transpose(*axes)
        return _funcs.transpose(self, axes)

    def reshape(self, *shape, order="C"):
        # arr.reshape(shape) and arr.reshape(*shape)
        return _funcs.reshape(self, shape, order=order)

    def sort(self, axis=-1, kind=None, order=None):
        # ndarray.sort works in-place
        _funcs.copyto(self, _funcs.sort(self, axis, kind, order))

    def item(self, *args):
        # Mimic NumPy's implementation with three special cases (no arguments,
        # a flat index and a multi-index):
        # https://github.com/numpy/numpy/blob/main/numpy/_core/src/multiarray/methods.c#L702
        if args == ():
            return self.tensor.item()
        elif len(args) == 1:
            # int argument
            return self.ravel()[args[0]]
        else:
            return self.__getitem__(args)

    def __getitem__(self, index):
        tensor = self.tensor
````
- **EN**: This chunk defines `__getitem__`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__getitem__`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 577-596 / 第 577-596 行
````python
        def neg_step(i, s):
            if not (isinstance(s, slice) and s.step is not None and s.step < 0):
                return s

            nonlocal tensor
            tensor = torch.flip(tensor, (i,))

            # Account for the fact that a slice includes the start but not the end
            if not (isinstance(s.start, int) or s.start is None):
                raise AssertionError(
                    f"slice start must be int or None, got {type(s.start).__name__}"
                )
            if not (isinstance(s.stop, int) or s.stop is None):
                raise AssertionError(
                    f"slice stop must be int or None, got {type(s.stop).__name__}"
                )
            start = s.stop + 1 if s.stop else None
            stop = s.start + 1 if s.start else None

            return slice(start, stop, -s.step)
````
- **EN**: This chunk defines `neg_step`, which advances mutable state using the current inputs, gradients, or counters. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `neg_step`，其作用是利用当前输入、梯度或计数器推进可变状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 598-622 / 第 598-622 行
````python
        if isinstance(index, Sequence):
            index = type(index)(neg_step(i, s) for i, s in enumerate(index))
        else:
            index = neg_step(0, index)
        index = _util.ndarrays_to_tensors(index)
        index = _upcast_int_indices(index)
        # Apply NumPy-compatible indexing conversion
        index = _numpy_compatible_indexing(index)
        # Apply NumPy-compatible empty ellipsis behavior
        index, maybe_squeeze, _ = _numpy_empty_ellipsis_patch(index, tensor.ndim)
        return maybe_squeeze(ndarray(tensor.__getitem__(index)))

    def __setitem__(self, index, value):
        index = _util.ndarrays_to_tensors(index)
        index = _upcast_int_indices(index)
        # Apply NumPy-compatible indexing conversion
        index = _numpy_compatible_indexing(index)
        # Apply NumPy-compatible empty ellipsis behavior
        index, _, maybe_unsqueeze = _numpy_empty_ellipsis_patch(index, self.tensor.ndim)

        if not _dtypes_impl.is_scalar(value):
            value = normalize_array_like(value)
            value = _util.cast_if_needed(value, self.tensor.dtype)

        return self.tensor.__setitem__(index, maybe_unsqueeze(value))
````
- **EN**: This chunk defines `__setitem__`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__setitem__`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 624-648 / 第 624-648 行
````python
    take = _funcs.take
    put = _funcs.put

    def __dlpack__(self, *, stream=None):
        return self.tensor.__dlpack__(stream=stream)

    def __dlpack_device__(self):
        return self.tensor.__dlpack_device__()


def _tolist(obj):
    """Recursively convert tensors into lists."""
    a1 = []
    for elem in obj:
        if isinstance(elem, (list, tuple)):
            elem = _tolist(elem)
        if isinstance(elem, ndarray):
            a1.append(elem.tensor.tolist())
        else:
            a1.append(elem)
    return a1


# This is the ideally the only place which talks to ndarray directly.
# The rest goes through asarray (preferred) or array.
````
- **EN**: This chunk defines `_tolist`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_tolist`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 651-676 / 第 651-676 行
````python
def array(obj, dtype=None, *, copy=True, order="K", subok=False, ndmin=0, like=None):
    if subok is not False:
        raise NotImplementedError("'subok' parameter is not supported.")
    if like is not None:
        raise NotImplementedError("'like' parameter is not supported.")
    if order != "K":
        raise NotImplementedError

    # a happy path
    if (
        isinstance(obj, ndarray)
        and copy is False
        and dtype is None
        and ndmin <= obj.ndim
    ):
        return obj

    if isinstance(obj, (list, tuple)):
        # FIXME and they have the same dtype, device, etc
        if obj and all(isinstance(x, torch.Tensor) for x in obj):
            # list of arrays: *under torch.Dynamo* these are FakeTensors
            obj = torch.stack(obj)
        else:
            # XXX: remove tolist
            # lists of ndarrays: [1, [2, 3], ndarray(4)] convert to lists of lists
            obj = _tolist(obj)
````
- **EN**: This chunk defines `array`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `array`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 678-704 / 第 678-704 行
````python
    # is obj an ndarray already?
    if isinstance(obj, ndarray):
        obj = obj.tensor

    # is a specific dtype requested?
    torch_dtype = None
    if dtype is not None:
        torch_dtype = _dtypes.dtype(dtype).torch_dtype

    tensor = _util._coerce_to_tensor(obj, torch_dtype, copy, ndmin)
    return ndarray(tensor)


def asarray(a, dtype=None, order="K", *, like=None):
    return array(a, dtype=dtype, order=order, like=like, copy=False, ndmin=0)


def ascontiguousarray(a, dtype=None, *, like=None):
    arr = asarray(a, dtype=dtype, like=like)
    if not arr.tensor.is_contiguous():
        arr.tensor = arr.tensor.contiguous()
    return arr


def from_dlpack(x, /):
    t = torch.from_dlpack(x)
    return ndarray(t)
````
- **EN**: This chunk defines `from_dlpack`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `from_dlpack`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 707-733 / 第 707-733 行
````python
def _extract_dtype(entry):
    try:
        dty = _dtypes.dtype(entry)
    except Exception:
        dty = asarray(entry).dtype
    return dty


def can_cast(from_, to, casting="safe"):
    from_ = _extract_dtype(from_)
    to_ = _extract_dtype(to)

    return _dtypes_impl.can_cast_impl(from_.torch_dtype, to_.torch_dtype, casting)


def result_type(*arrays_and_dtypes):
    tensors = []
    for entry in arrays_and_dtypes:
        try:
            t = asarray(entry).tensor
        except (RuntimeError, ValueError, TypeError):
            dty = _dtypes.dtype(entry)
            t = torch.empty(1, dtype=dty.torch_dtype)
        tensors.append(t)

    torch_dtype = _dtypes_impl.result_type_impl(*tensors)
    return _dtypes.dtype(torch_dtype)
````
- **EN**: This chunk defines `result_type`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `result_type`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **NumPy interop**
  - EN: Keeps PyTorch behavior compatible with NumPy expectations where practical.
  - CN: 在可行范围内让 PyTorch 行为与 NumPy 预期保持兼容。
- **FLAGS**
  - EN: `FLAGS` is one of the main symbols declared or implemented in this file.
  - CN: `FLAGS` 是本文件声明或实现的主要符号之一。
- **SHORTHAND_TO_FLAGS**
  - EN: `SHORTHAND_TO_FLAGS` is one of the main symbols declared or implemented in this file.
  - CN: `SHORTHAND_TO_FLAGS` 是本文件声明或实现的主要符号之一。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.`, `._normalizations`
- **Standard library / 标准库**: `__future__`, `builtins`, `math`, `operator`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `FLAGS`, `SHORTHAND_TO_FLAGS`, `Flags`, `create_method`, `_upcast_int_indices`, `_has_advanced_indexing`, `_numpy_compatible_indexing`, `_get_bool_depth`, `_numpy_empty_ellipsis_patch`, `_Unspecified`
