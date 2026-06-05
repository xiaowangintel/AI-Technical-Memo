# fft.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_refs/fft.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides reference Python implementations of operators used for correctness and decomposition paths.
- **Purpose (CN)**: 提供算子的参考 Python 实现，用于正确性验证和 decomposition 路径。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
import math
from collections.abc import Iterable, Sequence
from typing import Literal, NamedTuple

import torch
import torch._prims as prims
import torch._prims_common as utils
from torch._decomp import register_decomposition
from torch._prims_common import DimsType, ShapeType, TensorLikeType
from torch._prims_common.wrappers import _maybe_convert_to_dtype, out_wrapper


__all__ = [
    # Transforms
    "fft",
    "fft2",
    "fftn",
    "hfft",
    "hfft2",
    "hfftn",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._prims, torch._prims_common, ...; standard-library helpers such as math, collections.abc, typing. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._prims、torch._prims_common、...；标准库辅助模块，如 math、collections.abc、typing。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 21-36 / 第 21-36 行
````python
    "rfft",
    "rfft2",
    "rfftn",
    "ifft",
    "ifft2",
    "ifftn",
    "ihfft",
    "ihfft2",
    "ihfftn",
    "irfft",
    "irfft2",
    "irfftn",
    # Helpers
    "fftshift",
    "ifftshift",
]
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 38-55 / 第 38-55 行
````python
NormType = None | Literal["forward", "backward", "ortho"]
_NORM_VALUES = {None, "forward", "backward", "ortho"}
aten = torch._ops.ops.aten


def _apply_norm(
    x: TensorLikeType, norm: NormType, signal_numel: int, forward: bool
) -> TensorLikeType:
    """Apply normalization to the un-normalized FFT result"""
    torch._check(norm in _NORM_VALUES, lambda: f"Invalid normalization mode: {norm}")

    if norm == "ortho":
        return x * (1 / math.sqrt(signal_numel))

    normalize = (not forward and (norm is None or norm == "backward")) or (
        forward and norm == "forward"
    )
    return x * (1 / signal_numel) if normalize else x
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `_apply_norm`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `_apply_norm`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 58-74 / 第 58-74 行
````python
def _promote_type_fft(
    dtype: torch.dtype, require_complex: bool, device: torch.device
) -> torch.dtype:
    """Helper to promote a dtype to one supported by the FFT primitives"""
    if dtype.is_complex:
        return dtype

    # Promote integral to default float type
    if not dtype.is_floating_point:
        dtype = torch.get_default_dtype()

    allowed_types = [torch.float32, torch.float64]
    maybe_support_half = device.type in ["cuda", "meta", "xpu"]

    if maybe_support_half:
        allowed_types.append(torch.float16)
    torch._check(dtype in allowed_types, lambda: f"Unsupported dtype {dtype}")
````
- **EN**: This chunk defines `_promote_type_fft`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_promote_type_fft`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 76-88 / 第 76-88 行
````python
    if require_complex:
        dtype = utils.corresponding_complex_dtype(dtype)

    return dtype


def _maybe_promote_tensor_fft(
    t: TensorLikeType, require_complex: bool = False
) -> TensorLikeType:
    """Helper to promote a tensor to a dtype supported by the FFT primitives"""
    cur_type = t.dtype
    new_type = _promote_type_fft(cur_type, require_complex, t.device)
    return _maybe_convert_to_dtype(t, new_type)  # type: ignore[return-value]
````
- **EN**: This chunk defines `_maybe_promote_tensor_fft`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_maybe_promote_tensor_fft`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 91-107 / 第 91-107 行
````python
def _resize_fft_input(
    x: TensorLikeType, dims: tuple[int, ...], sizes: tuple[int, ...]
) -> TensorLikeType:
    """
    Fixes the shape of x such that x.size(dims[i]) == sizes[i],
    either by zero-padding, or by slicing x starting from 0.
    """
    if len(dims) != len(sizes):
        raise AssertionError(
            f"dims and sizes must have the same length, got {len(dims)} and {len(sizes)}"
        )
    must_copy = False
    x_sizes = x.shape
    pad_amount = [0] * len(x_sizes) * 2
    for i in range(len(dims)):
        if sizes[i] == -1:
            continue
````
- **EN**: This chunk defines `_resize_fft_input`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_resize_fft_input`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 109-128 / 第 109-128 行
````python
        if x_sizes[dims[i]] < sizes[i]:
            must_copy = True
            pad_idx = len(pad_amount) - 2 * dims[i] - 1

            pad_amount[pad_idx] = sizes[i] - x_sizes[dims[i]]

        if x_sizes[dims[i]] > sizes[i]:
            x = x.narrow(dims[i], 0, sizes[i])

    return torch.constant_pad_nd(x, pad_amount) if must_copy else x


def _fft_c2r(
    func_name: str,
    input: TensorLikeType,
    n: int | None,
    dim: int,
    norm: NormType,
    forward: bool,
) -> TensorLikeType:
````
- **EN**: This chunk defines `_fft_c2r`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_fft_c2r`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-145 / 第 129-145 行
````python
    """Common code for performing any complex to real FFT (irfft or hfft)"""
    input = _maybe_promote_tensor_fft(input, require_complex=True)
    dims = (utils.canonicalize_dim(input.ndim, dim, wrap_scalar=False),)
    last_dim_size = n if n is not None else 2 * (input.shape[dim] - 1)
    torch._check(
        last_dim_size >= 1,
        lambda: f"Invalid number of data points ({last_dim_size}) specified",
    )

    if n is not None:
        input = _resize_fft_input(input, dims=dims, sizes=(last_dim_size // 2 + 1,))

    if forward:
        input = torch.conj(input)

    output = prims.fft_c2r(input, dim=dims, last_dim_size=last_dim_size)
    return _apply_norm(output, norm=norm, signal_numel=last_dim_size, forward=forward)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `_fft_c2r` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `_fft_c2r`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 148-167 / 第 148-167 行
````python
def _fft_r2c(
    func_name: str,
    input: TensorLikeType,
    n: int | None,
    dim: int,
    norm: NormType,
    forward: bool,
    onesided: bool,
) -> TensorLikeType:
    """Common code for performing any real to complex FFT (rfft or ihfft)"""
    torch._check(
        not input.dtype.is_complex,
        lambda: f"{func_name} expects a floating point input tensor, but got {input.dtype}",
    )
    input = _maybe_promote_tensor_fft(input)
    dims = (utils.canonicalize_dim(input.ndim, dim, wrap_scalar=False),)
    dim_size = n if n is not None else input.shape[dim]
    torch._check(
        dim_size >= 1, lambda: f"Invalid number of data points ({dim_size}) specified"
    )
````
- **EN**: This chunk defines `_fft_r2c`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_fft_r2c`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 169-188 / 第 169-188 行
````python
    if n is not None:
        input = _resize_fft_input(input, dims, (n,))

    ret = prims.fft_r2c(input, dim=dims, onesided=onesided)
    ret = _apply_norm(ret, norm, dim_size, forward)
    return ret if forward else torch.conj(ret)


def _fft_c2c(
    func_name: str,
    input: TensorLikeType,
    n: int | None,
    dim: int,
    norm: NormType,
    forward: bool,
) -> TensorLikeType:
    """Common code for performing any complex to complex FFT (fft or ifft)"""
    torch._check(
        input.dtype.is_complex,
        lambda: f"{func_name} expects a complex input tensor, but got {input.dtype}",
````
- **EN**: This chunk defines `_fft_c2c`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_fft_c2c`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 189-200 / 第 189-200 行
````python
    )
    dims = (utils.canonicalize_dim(input.ndim, dim, wrap_scalar=False),)
    dim_size = n if n is not None else input.shape[dim]
    torch._check(
        dim_size >= 1, lambda: f"Invalid number of data points ({dim_size}) specified"
    )

    if n is not None:
        input = _resize_fft_input(input, dims, (n,))

    ret = prims.fft_c2c(input, dim=dims, forward=forward)
    return _apply_norm(ret, norm, dim_size, forward)
````
- **EN**: This chunk continues `_fft_c2c` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_fft_c2c`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 203-214 / 第 203-214 行
````python
@register_decomposition(aten.fft_fft)
@out_wrapper()
def fft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    if input.dtype.is_complex:
        return _fft_c2c("fft", input, n, dim, norm, forward=True)
    else:
        return _fft_r2c("fft", input, n, dim, norm, forward=True, onesided=False)
````
- **EN**: This chunk defines `fft`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `fft`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 217-228 / 第 217-228 行
````python
@register_decomposition(aten.fft_ifft)
@out_wrapper()
def ifft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    if input.dtype.is_complex:
        return _fft_c2c("ifft", input, n, dim, norm, forward=False)
    else:
        return _fft_r2c("ifft", input, n, dim, norm, forward=False, onesided=False)
````
- **EN**: This chunk defines `ifft`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ifft`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 231-250 / 第 231-250 行
````python
@register_decomposition(aten.fft_rfft)
@out_wrapper()
def rfft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    return _fft_r2c("rfft", input, n, dim, norm, forward=True, onesided=True)


@register_decomposition(aten.fft_irfft)
@out_wrapper()
def irfft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    return _fft_c2r("irfft", input, n, dim, norm, forward=False)
````
- **EN**: This chunk defines `irfft`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `irfft`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 253-272 / 第 253-272 行
````python
@register_decomposition(aten.fft_hfft)
@out_wrapper()
def hfft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    return _fft_c2r("hfft", input, n, dim, norm, forward=True)


@register_decomposition(aten.fft_ihfft)
@out_wrapper()
def ihfft(
    input: TensorLikeType,
    n: int | None = None,
    dim: int = -1,
    norm: NormType = None,
) -> TensorLikeType:
    return _fft_r2c("ihfft", input, n, dim, norm, forward=False, onesided=True)
````
- **EN**: This chunk defines `ihfft`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ihfft`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 275-290 / 第 275-290 行
````python
class _ShapeAndDims(NamedTuple):
    shape: tuple[int, ...]
    dims: tuple[int, ...]


def _canonicalize_fft_shape_and_dim_args(
    input: TensorLikeType, shape: ShapeType | None, dim: DimsType | None
) -> _ShapeAndDims:
    """Convert the shape and dim arguments into a canonical form where neither are optional"""
    input_dim = input.ndim
    input_sizes = input.shape

    if dim is not None:
        if not isinstance(dim, Sequence):
            dim = (dim,)
        ret_dims = utils.canonicalize_dims(input_dim, dim, wrap_scalar=False)
````
- **EN**: It introduces or extends `_ShapeAndDims`, which hold the main object-oriented state for this portion of the file. This chunk defines `_canonicalize_fft_shape_and_dim_args`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `_ShapeAndDims`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_canonicalize_fft_shape_and_dim_args`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 292-306 / 第 292-306 行
````python
        # Check dims are unique
        torch._check(
            len(set(ret_dims)) == len(ret_dims), lambda: "FFT dims must be unique"
        )

    if shape is not None:
        if not isinstance(shape, Sequence):
            shape = (shape,)

        # Has shape, might have dim
        torch._check(
            dim is None or len(dim) == len(shape),
            lambda: "When given, dim and shape arguments must have the same length",
        )
        transform_ndim = len(shape)
````
- **EN**: This chunk continues `_ShapeAndDims` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_ShapeAndDims`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 308-327 / 第 308-327 行
````python
        torch._check(
            transform_ndim <= input_dim,
            lambda: f"Got shape with {transform_ndim} values but input tensor "
            f"only has {input_dim} dimensions.",
        )

        # If shape is given, dims defaults to the last len(shape) dimensions
        if dim is None:
            ret_dims = tuple(range(input_dim - transform_ndim, input_dim))

        # Translate any -1 values in shape to the default length
        ret_shape = tuple(
            s if s != -1 else input_sizes[d]
            for (s, d) in zip(shape, ret_dims)  # type: ignore[possibly-undefined]
        )
    elif dim is None:
        # No shape, no dim
        ret_dims = tuple(range(input_dim))
        ret_shape = tuple(input_sizes)
    else:
````
- **EN**: This chunk continues `_ShapeAndDims` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_ShapeAndDims`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 328-342 / 第 328-342 行
````python
        # No shape, has dim
        ret_shape = tuple(input_sizes[d] for d in ret_dims)  # type: ignore[possibly-undefined]

    for n in ret_shape:
        torch._check(n > 0, lambda: f"Invalid number of data points ({n}) specified")

    return _ShapeAndDims(shape=ret_shape, dims=ret_dims)  # type: ignore[possibly-undefined]


def _prod(xs: Iterable[int]) -> int:
    """Compute product of a list"""
    prod = 1
    for x in xs:
        prod *= x
    return prod
````
- **EN**: This chunk defines `_prod`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_prod`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 345-361 / 第 345-361 行
````python
def _fftn_c2c(
    function_name: str,
    input: TensorLikeType,
    shape: tuple[int, ...],
    dim: tuple[int, ...],
    norm: NormType,
    forward: bool,
) -> TensorLikeType:
    """Common code for n-dimensional complex to complex FFTs (fftn or ifftn)"""
    torch._check(
        input.dtype.is_complex,
        lambda: f"{function_name} expects a complex input tensor, "
        f"but got {input.dtype}",
    )
    x = _resize_fft_input(input, dim, shape)
    output = prims.fft_c2c(x, dim=dim, forward=forward)
    return _apply_norm(output, norm=norm, signal_numel=_prod(shape), forward=forward)
````
- **EN**: This chunk defines `_fftn_c2c`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_fftn_c2c`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 364-383 / 第 364-383 行
````python
@register_decomposition(aten.fft_fftn)
@out_wrapper()
def fftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
) -> TensorLikeType:
    (shape, dim) = _canonicalize_fft_shape_and_dim_args(input, s, dim)
    x = _maybe_promote_tensor_fft(input, require_complex=True)
    return _fftn_c2c("fftn", x, shape, dim, norm, forward=True)


@register_decomposition(aten.fft_ifftn)
@out_wrapper()
def ifftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
````
- **EN**: This chunk defines `ifftn`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ifftn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 384-403 / 第 384-403 行
````python
) -> TensorLikeType:
    (shape, dim) = _canonicalize_fft_shape_and_dim_args(input, s, dim)
    x = _maybe_promote_tensor_fft(input, require_complex=True)
    return _fftn_c2c("ifftn", x, shape, dim, norm, forward=False)


@register_decomposition(aten.fft_rfftn)
@out_wrapper()
def rfftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
) -> TensorLikeType:
    torch._check(
        not input.dtype.is_complex,
        lambda: f"rfftn expects a real-valued input tensor, but got {input.dtype}",
    )
    shape, dim = _canonicalize_fft_shape_and_dim_args(input, s, dim)
    input = _maybe_promote_tensor_fft(input, require_complex=False)
````
- **EN**: This chunk defines `rfftn`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `rfftn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 404-423 / 第 404-423 行
````python
    input = _resize_fft_input(input, dim, shape)
    out = prims.fft_r2c(input, dim=dim, onesided=True)
    return _apply_norm(out, norm=norm, signal_numel=_prod(shape), forward=True)


@register_decomposition(aten.fft_ihfftn)
@out_wrapper()
def ihfftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
) -> TensorLikeType:
    torch._check(
        not input.dtype.is_complex,
        lambda: f"ihfftn expects a real-valued input tensor, but got {input.dtype}",
    )
    shape, dim = _canonicalize_fft_shape_and_dim_args(input, s, dim)
    torch._check(len(shape) > 0, lambda: "ihfftn must transform at least one axis")
    input = _maybe_promote_tensor_fft(input, require_complex=False)
````
- **EN**: This chunk defines `ihfftn`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ihfftn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 424-440 / 第 424-440 行
````python
    input = _resize_fft_input(input, dim, shape)

    tmp = prims.fft_r2c(input, dim=dim[-1:], onesided=True)

    if len(dim) == 1:
        tmp = _apply_norm(tmp, norm=norm, signal_numel=shape[0], forward=False)
        return prims.conj(tmp)

    tmp = prims.conj_physical(tmp)
    tmp = prims.fft_c2c(tmp, dim=dim[:-1], forward=False)
    return _apply_norm(tmp, norm=norm, signal_numel=_prod(shape), forward=False)


class _CanonicalizeC2rReturn(NamedTuple):
    shape: tuple[int, ...]
    dim: tuple[int, ...]
    last_dim_size: int
````
- **EN**: It introduces or extends `_CanonicalizeC2rReturn`, which hold the main object-oriented state for this portion of the file. This chunk continues `_CanonicalizeC2rReturn` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_CanonicalizeC2rReturn`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `_CanonicalizeC2rReturn`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 443-457 / 第 443-457 行
````python
def _canonicalize_fft_c2r_shape_and_dim_args(
    fname: str,
    input: TensorLikeType,
    s: ShapeType | None,
    dim: DimsType | None,
) -> _CanonicalizeC2rReturn:
    """Canonicalize shape and dim arguments for n-dimensional c2r transforms,
    as well as calculating the last_dim_size which is shape[dim[-1]] for the output"""
    (shape, dim) = _canonicalize_fft_shape_and_dim_args(input, s, dim)
    torch._check(len(shape) > 0, lambda: f"{fname} must transform at least one axis")

    if s is None or s[-1] == -1:
        last_dim_size = 2 * (input.shape[dim[-1]] - 1)
    else:
        last_dim_size = shape[-1]
````
- **EN**: This chunk defines `_canonicalize_fft_c2r_shape_and_dim_args`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_canonicalize_fft_c2r_shape_and_dim_args`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 459-478 / 第 459-478 行
````python
    torch._check(
        last_dim_size >= 1,
        lambda: f"Invalid number of data points ({last_dim_size}) specified",
    )

    shape_list = list(shape)
    shape_list[-1] = last_dim_size // 2 + 1
    return _CanonicalizeC2rReturn(
        shape=tuple(shape_list), dim=dim, last_dim_size=last_dim_size
    )


@register_decomposition(aten.fft_irfftn)
@out_wrapper()
def irfftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
) -> TensorLikeType:
````
- **EN**: This chunk defines `irfftn`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `irfftn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 479-498 / 第 479-498 行
````python
    shape, dim, last_dim_size = _canonicalize_fft_c2r_shape_and_dim_args(
        "irfftn", input, s, dim
    )
    input = _maybe_promote_tensor_fft(input, require_complex=True)
    input = _resize_fft_input(input, dim, shape)
    out = prims.fft_c2r(input, dim=dim, last_dim_size=last_dim_size)
    return _apply_norm(out, norm, _prod(out.shape[d] for d in dim), forward=False)


@register_decomposition(aten.fft_hfftn)
@out_wrapper()
def hfftn(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = None,
    norm: NormType = None,
) -> TensorLikeType:
    shape, dim, last_dim_size = _canonicalize_fft_c2r_shape_and_dim_args(
        "hfftn", input, s, dim
    )
````
- **EN**: This chunk defines `hfftn`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `hfftn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 499-517 / 第 499-517 行
````python
    input = _maybe_promote_tensor_fft(input, require_complex=True)
    input = _resize_fft_input(input, dim, shape)

    tmp = prims.fft_c2c(input, dim=dim[:-1], forward=True) if len(dim) > 1 else input
    tmp = _apply_norm(tmp, norm, _prod(shape[:-1]), forward=True)
    tmp = prims.conj_physical(tmp)
    out = prims.fft_c2r(tmp, dim=dim[-1:], last_dim_size=last_dim_size)
    return _apply_norm(out, norm, last_dim_size, forward=True)


@register_decomposition(aten.fft_fft2)
@out_wrapper()
def fft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.fftn(input, s=s, dim=dim, norm=norm)
````
- **EN**: This chunk defines `fft2`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `fft2`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 520-539 / 第 520-539 行
````python
@register_decomposition(aten.fft_ifft2)
@out_wrapper()
def ifft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.ifftn(input, s=s, dim=dim, norm=norm)


@register_decomposition(aten.fft_rfft2)
@out_wrapper()
def rfft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.rfftn(input, s=s, dim=dim, norm=norm)
````
- **EN**: This chunk defines `rfft2`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `rfft2`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 542-561 / 第 542-561 行
````python
@register_decomposition(aten.fft_irfft2)
@out_wrapper()
def irfft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.irfftn(input, s=s, dim=dim, norm=norm)


@register_decomposition(aten.fft_hfft2)
@out_wrapper()
def hfft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.hfftn(input, s=s, dim=dim, norm=norm)
````
- **EN**: This chunk defines `hfft2`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `hfft2`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 564-582 / 第 564-582 行
````python
@register_decomposition(aten.fft_ihfft2)
@out_wrapper()
def ihfft2(
    input: TensorLikeType,
    s: ShapeType | None = None,
    dim: DimsType | None = (-2, -1),
    norm: NormType = None,
) -> TensorLikeType:
    return torch.fft.ihfftn(input, s=s, dim=dim, norm=norm)


def _default_alldims(dim: DimsType | None, x: TensorLikeType) -> list[int]:
    """Convert Optional[DimsType] to a simple list, defaulting to all dimensions"""
    if dim is None:
        return list(range(x.ndim))
    elif not isinstance(dim, Sequence):
        return [dim]
    else:
        return list(dim)
````
- **EN**: This chunk defines `_default_alldims`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_default_alldims`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 585-596 / 第 585-596 行
````python
@register_decomposition(aten.fft_fftshift)
def fftshift(input: TensorLikeType, dim: DimsType | None = None) -> TensorLikeType:
    dims = _default_alldims(dim, input)
    shift = [input.shape[d] // 2 for d in dims]
    return torch.roll(input, shift, dims)


@register_decomposition(aten.fft_ifftshift)
def ifftshift(input: TensorLikeType, dim: DimsType | None = None) -> TensorLikeType:
    dims = _default_alldims(dim, input)
    shift = [(input.shape[d] + 1) // 2 for d in dims]
    return torch.roll(input, shift, dims)
````
- **EN**: This chunk defines `ifftshift`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ifftshift`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Reference implementations**
  - EN: Provides readable Python operator definitions used as correctness references.
  - CN: 提供可读的 Python 算子定义，作为正确性参考实现。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_NORM_VALUES**
  - EN: `_NORM_VALUES` is one of the main symbols declared or implemented in this file.
  - CN: `_NORM_VALUES` 是本文件声明或实现的主要符号之一。
- **Decomposition**
  - EN: The implementation rewrites larger operators into smaller primitives for reuse and lowering.
  - CN: 实现把较大的算子重写为更小的 primitive，以便复用和降级。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._prims`, `torch._prims_common`, `torch._decomp`, `torch._prims_common.wrappers`
- **Standard library / 标准库**: `math`, `collections.abc`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_NORM_VALUES`, `_apply_norm`, `_promote_type_fft`, `_maybe_promote_tensor_fft`, `_resize_fft_input`, `_fft_c2r`, `_fft_r2c`, `_fft_c2c`, `fft`
