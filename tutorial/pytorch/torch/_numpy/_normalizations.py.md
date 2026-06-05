# _normalizations.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_numpy/_normalizations.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements NumPy-compatibility helpers, wrappers, and interoperability utilities.
- **Purpose (CN)**: 实现 NumPy 兼容辅助逻辑、包装层以及互操作工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
````python
# mypy: ignore-errors

""" "Normalize" arguments: convert array_likes to tensors, dtypes to torch dtypes and so on."""

from __future__ import annotations

import functools
import inspect
import operator
import types
import typing

import torch

from . import _dtypes, _dtypes_impl, _util
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, .; standard-library helpers such as __future__, functools, inspect, .... The future import postpones annotation evaluation, keeping type hints lightweight at import time. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.；标准库辅助模块，如 __future__、functools、inspect、...。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 18-37 / 第 18-37 行
````python
ArrayLike = typing.TypeVar("ArrayLike")
Scalar = int | float | complex | bool
ArrayLikeOrScalar = ArrayLike | Scalar

DTypeLike = typing.TypeVar("DTypeLike")
AxisLike = typing.TypeVar("AxisLike")
NDArray = typing.TypeVar("NDArray")
CastingModes = typing.TypeVar("CastingModes")
KeepDims = typing.TypeVar("KeepDims")

# OutArray is to annotate the out= array argument.
#
# This one is special is several respects:
# First, It needs to be an NDArray, and we need to preserve the `result is out`
# semantics. Therefore, we cannot just extract the Tensor from the out array.
# So we never pass the out array to implementer functions and handle it in the
# `normalizer` below.
# Second, the out= argument can be either keyword or positional argument, and
# as a positional arg, it can be anywhere in the signature.
# To handle all this, we define a special `OutArray` annotation and dispatch on it.
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。

### Lines 38-55 / 第 38-55 行
````python
#
OutArray = typing.TypeVar("OutArray")

NotImplementedType = typing.TypeVar(
    "NotImplementedType", bound=types.NotImplementedType
)


def normalize_array_like(x, parm=None):  # codespell:ignore
    from ._ndarray import asarray

    return asarray(x).tensor


def normalize_array_like_or_scalar(x, parm=None):  # codespell:ignore
    if _dtypes_impl.is_scalar_or_symbolic(x):
        return x
    return normalize_array_like(x, parm)  # codespell:ignore
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `normalize_array_like_or_scalar`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `normalize_array_like_or_scalar`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 58-71 / 第 58-71 行
````python
def normalize_optional_array_like_or_scalar(x, parm=None):  # codespell:ignore
    if x is None:
        return None
    return normalize_array_like_or_scalar(x, parm)  # codespell:ignore


def normalize_optional_array_like(x, parm=None):  # codespell:ignore
    # This explicit normalizer is needed because otherwise normalize_array_like
    # does not run for a parameter annotated as Optional[ArrayLike]
    return None if x is None else normalize_array_like(x, parm)  # codespell:ignore


def normalize_seq_array_like(x, parm=None):  # codespell:ignore
    return tuple(normalize_array_like(value) for value in x)
````
- **EN**: This chunk defines `normalize_seq_array_like`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `normalize_seq_array_like`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 74-91 / 第 74-91 行
````python
def normalize_dtype(dtype, parm=None):  # codespell:ignore
    # cf _decorators.dtype_to_torch
    torch_dtype = None
    if dtype is not None:
        dtype = _dtypes.dtype(dtype)
        torch_dtype = dtype.torch_dtype
    return torch_dtype


def normalize_not_implemented(arg, parm):  # codespell:ignore
    if arg != parm.default:  # codespell:ignore
        raise NotImplementedError(
            f"'{parm.name}' parameter is not supported."  # codespell:ignore
        )


def normalize_axis_like(arg, parm=None):  # codespell:ignore
    from ._ndarray import ndarray
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `normalize_axis_like`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `normalize_axis_like`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 93-107 / 第 93-107 行
````python
    if isinstance(arg, ndarray):
        arg = operator.index(arg)
    return arg


def normalize_ndarray(arg, parm=None):  # codespell:ignore
    # check the arg is an ndarray, extract its tensor attribute
    if arg is None:
        return arg

    from ._ndarray import ndarray

    if not isinstance(arg, ndarray):
        raise TypeError(f"'{parm.name}' must be an array")  # codespell:ignore
    return arg.tensor
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `normalize_ndarray`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `normalize_ndarray`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 110-123 / 第 110-123 行
````python
def normalize_outarray(arg, parm=None):  # codespell:ignore
    # almost normalize_ndarray, only return the array, not its tensor
    if arg is None:
        return arg
    from ._ndarray import ndarray

    # Dynamo can pass torch tensors as out arguments,
    # wrap it in an ndarray before processing
    if isinstance(arg, torch.Tensor):
        arg = ndarray(arg)

    if not isinstance(arg, ndarray):
        raise TypeError(f"'{parm.name}' must be an array")  # codespell:ignore
    return arg
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `normalize_outarray`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `normalize_outarray`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 126-145 / 第 126-145 行
````python
def normalize_casting(arg, parm=None):  # codespell:ignore
    if arg not in ["no", "equiv", "safe", "same_kind", "unsafe"]:
        raise ValueError(
            f"casting must be one of 'no', 'equiv', 'safe', 'same_kind', or 'unsafe' (got '{arg}')"
        )
    return arg


normalizers = {
    "ArrayLike": normalize_array_like,
    "ArrayLikeOrScalar": normalize_array_like_or_scalar,
    "Optional[ArrayLike]": normalize_optional_array_like,
    "ArrayLike | None": normalize_optional_array_like,
    "Sequence[ArrayLike]": normalize_seq_array_like,
    "Optional[ArrayLikeOrScalar]": normalize_optional_array_like_or_scalar,
    "ArrayLikeOrScalar | None": normalize_optional_array_like_or_scalar,
    "Optional[NDArray]": normalize_ndarray,
    "NDArray | None": normalize_ndarray,
    "Optional[OutArray]": normalize_outarray,
    "OutArray | None": normalize_outarray,
````
- **EN**: This chunk defines `normalize_casting`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `normalize_casting`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 146-162 / 第 146-162 行
````python
    "NDArray": normalize_ndarray,
    "Optional[DTypeLike]": normalize_dtype,
    "DTypeLike | None": normalize_dtype,
    "AxisLike": normalize_axis_like,
    "NotImplementedType": normalize_not_implemented,
    "Optional[CastingModes]": normalize_casting,
    "CastingModes | None": normalize_casting,
}


def maybe_normalize(arg, parm):  # codespell:ignore
    """Normalize arg if a normalizer is registered."""
    normalizer = normalizers.get(parm.annotation)  # codespell:ignore
    return normalizer(arg, parm) if normalizer else arg  # codespell:ignore


# ### Return value helpers ###
````
- **EN**: This chunk defines `maybe_normalize`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `maybe_normalize`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 165-184 / 第 165-184 行
````python
def maybe_copy_to(out, result, promote_scalar_result=False):
    # NB: here out is either an ndarray or None
    if out is None:
        return result
    elif isinstance(result, torch.Tensor):
        if result.shape != out.shape:
            can_fit = result.numel() == 1 and out.ndim == 0
            if promote_scalar_result and can_fit:
                result = result.squeeze()
            else:
                raise ValueError(
                    f"Bad size of the out array: out.shape = {out.shape}"
                    f" while result.shape = {result.shape}."
                )
        out.tensor.copy_(result)
        return out
    elif isinstance(result, (tuple, list)):
        return type(result)(
            maybe_copy_to(o, r, promote_scalar_result) for o, r in zip(out, result)
        )
````
- **EN**: This chunk defines `maybe_copy_to`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `maybe_copy_to`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 185-203 / 第 185-203 行
````python
    else:
        raise AssertionError  # We should never hit this path


def wrap_tensors(result):
    from ._ndarray import ndarray

    if isinstance(result, torch.Tensor):
        return ndarray(result)
    elif isinstance(result, (tuple, list)):
        result = type(result)(wrap_tensors(x) for x in result)
    return result


def array_or_scalar(values, py_type=float, return_scalar=False):
    if return_scalar:
        return py_type(values.item())
    else:
        from ._ndarray import ndarray
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ._ndarray. This chunk defines `array_or_scalar`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 ._ndarray。 这一段定义了 `array_or_scalar`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 205-217 / 第 205-217 行
````python
        return ndarray(values)


# ### The main decorator to normalize arguments / postprocess the output ###


def normalizer(_func=None, *, promote_scalar_result=False):
    def normalizer_inner(func):
        @functools.wraps(func)
        def wrapped(*args, **kwds):
            sig = inspect.signature(func)
            params = sig.parameters
            first_param = next(iter(params.values()))
````
- **EN**: This chunk defines `wrapped`, which implements a focused helper used by the surrounding module. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wrapped`，其作用是实现周边模块使用的关键辅助逻辑。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 219-237 / 第 219-237 行
````python
            # NumPy's API does not have positional args before variadic positional args
            if first_param.kind == inspect.Parameter.VAR_POSITIONAL:
                args = [maybe_normalize(arg, first_param) for arg in args]
            else:
                # NB: extra unknown arguments: pass through, will raise in func(*args) below
                args = (
                    tuple(
                        maybe_normalize(arg, parm)  # codespell:ignore
                        for arg, parm in zip(args, params.values())  # codespell:ignore
                    )
                    + args[len(params.values()) :]
                )

            kwds = {
                name: maybe_normalize(arg, params[name]) if name in params else arg
                for name, arg in kwds.items()
            }

            result = func(*args, **kwds)
````
- **EN**: This chunk continues `wrapped` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `wrapped`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 239-258 / 第 239-258 行
````python
            # keepdims
            bound_args = None
            if "keepdims" in params and params["keepdims"].annotation == "KeepDims":
                # keepdims can be in any position so we need sig.bind
                bound_args = sig.bind(*args, **kwds).arguments
                if bound_args.get("keepdims", False):
                    # In this case the first arg is the initial tensor and
                    # the second arg is (optionally) the axis
                    tensor = args[0]
                    axis = bound_args.get("axis")
                    result = _util.apply_keepdims(result, axis, tensor.ndim)

            # out
            if "out" in params:
                # out can be in any position so we need sig.bind
                if bound_args is None:
                    bound_args = sig.bind(*args, **kwds).arguments
                out = bound_args.get("out")
                result = maybe_copy_to(out, result, promote_scalar_result)
            result = wrap_tensors(result)
````
- **EN**: This chunk continues `wrapped` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `wrapped`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 260-267 / 第 260-267 行
````python
            return result

        return wrapped

    if _func is None:
        return normalizer_inner
    else:
        return normalizer_inner(_func)
````
- **EN**: This chunk continues `wrapped` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `wrapped`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **NumPy interop**
  - EN: Keeps PyTorch behavior compatible with NumPy expectations where practical.
  - CN: 在可行范围内让 PyTorch 行为与 NumPy 预期保持兼容。
- **normalize_array_like**
  - EN: `normalize_array_like` is one of the main symbols declared or implemented in this file.
  - CN: `normalize_array_like` 是本文件声明或实现的主要符号之一。
- **normalize_array_like_or_scalar**
  - EN: `normalize_array_like_or_scalar` is one of the main symbols declared or implemented in this file.
  - CN: `normalize_array_like_or_scalar` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `.`, `._ndarray`
- **Standard library / 标准库**: `__future__`, `functools`, `inspect`, `operator`, `types`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `normalize_array_like`, `normalize_array_like_or_scalar`, `normalize_optional_array_like_or_scalar`, `normalize_optional_array_like`, `normalize_seq_array_like`, `normalize_dtype`, `normalize_not_implemented`, `normalize_axis_like`, `normalize_ndarray`, `normalize_outarray`
