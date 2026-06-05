# _tensor_str.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_tensor_str.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行
````python
# mypy: allow-untyped-defs
import contextlib
import dataclasses
import math
import textwrap
from typing import Any

import torch
from torch import inf


@dataclasses.dataclass
class __PrinterOptions:
    precision: int = 4
    threshold: float = 1000
    edgeitems: int = 3
    linewidth: int = 80
    sci_mode: bool | None = None


PRINT_OPTS = __PrinterOptions()
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as contextlib, dataclasses, math, .... It introduces or extends `__PrinterOptions`, which hold the main object-oriented state for this portion of the file. Decorators such as `dataclasses.dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 contextlib、dataclasses、math、...。 它引入或扩展了 `__PrinterOptions`，这些类承载了本段涉及的主要面向对象状态。 像 `dataclasses.dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 24-50 / 第 24-50 行
````python
# We could use **kwargs, but this will give better docs
def set_printoptions(
    precision=None,
    threshold=None,
    edgeitems=None,
    linewidth=None,
    profile=None,
    sci_mode=None,
):
    r"""Set options for printing. Items shamelessly taken from NumPy

    Args:
        precision: Number of digits of precision for floating point output
            (default = 4).
        threshold: Total number of array elements which trigger summarization
            rather than full `repr` (default = 1000).
        edgeitems: Number of array items in summary at beginning and end of
            each dimension (default = 3).
        linewidth: The number of characters per line for the purpose of
            inserting line breaks (default = 80). Thresholded matrices will
            ignore this parameter.
        profile: Sane defaults for pretty printing. Can override with any of
            the above options. (any one of `default`, `short`, `full`)
        sci_mode: Enable (True) or disable (False) scientific notation. If
            None (default) is specified, the value is defined by
            `torch._tensor_str._Formatter`. This value is automatically chosen
            by the framework.
````
- **EN**: This chunk defines `set_printoptions`, which mutates configuration or backend state that affects later execution. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `set_printoptions`，其作用是修改会影响后续执行的配置或后端状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 52-79 / 第 52-79 行
````python
    Example::

        >>> # Limit the precision of elements
        >>> torch.set_printoptions(precision=2)
        >>> torch.tensor([1.12345])
        tensor([1.12])
        >>> # Limit the number of elements shown
        >>> torch.set_printoptions(threshold=5)
        >>> torch.arange(10)
        tensor([0, 1, 2, ..., 7, 8, 9])
        >>> # Restore defaults
        >>> torch.set_printoptions(profile='default')
        >>> torch.tensor([1.12345])
        tensor([1.1235])
        >>> torch.arange(10)
        tensor([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

    """
    if profile is not None:
        if profile == "default":
            PRINT_OPTS.precision = 4
            PRINT_OPTS.threshold = 1000
            PRINT_OPTS.edgeitems = 3
            PRINT_OPTS.linewidth = 80
        elif profile == "short":
            PRINT_OPTS.precision = 2
            PRINT_OPTS.threshold = 1000
            PRINT_OPTS.edgeitems = 2
````
- **EN**: This chunk continues `set_printoptions` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `set_printoptions`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 80-102 / 第 80-102 行
````python
            PRINT_OPTS.linewidth = 80
        elif profile == "full":
            PRINT_OPTS.precision = 4
            PRINT_OPTS.threshold = inf
            PRINT_OPTS.edgeitems = 3
            PRINT_OPTS.linewidth = 80

    if precision is not None:
        PRINT_OPTS.precision = precision
    if threshold is not None:
        PRINT_OPTS.threshold = threshold
    if edgeitems is not None:
        PRINT_OPTS.edgeitems = edgeitems
    if linewidth is not None:
        PRINT_OPTS.linewidth = linewidth
    PRINT_OPTS.sci_mode = sci_mode


def get_printoptions() -> dict[str, Any]:
    r"""Gets the current options for printing, as a dictionary that
    can be passed as ``**kwargs`` to set_printoptions().
    """
    return dataclasses.asdict(PRINT_OPTS)
````
- **EN**: This chunk defines `get_printoptions`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_printoptions`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 105-127 / 第 105-127 行
````python
@contextlib.contextmanager
def printoptions(**kwargs):
    r"""Context manager that temporarily changes the print options.  Accepted
    arguments are same as :func:`set_printoptions`."""
    old_kwargs = get_printoptions()
    set_printoptions(**kwargs)
    try:
        yield
    finally:
        set_printoptions(**old_kwargs)


def tensor_totype(t):
    dtype = (
        torch.float
        if (
            t.is_mps
            or (t.is_xpu and not torch.xpu.get_device_properties(t.device).has_fp64)
            or t.is_maia
        )
        else torch.double
    )
    return t.to(dtype=dtype)
````
- **EN**: This chunk defines `tensor_totype`, which implements a focused helper used by the surrounding module. Decorators such as `contextlib.contextmanager` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `tensor_totype`，其作用是实现周边模块使用的关键辅助逻辑。 像 `contextlib.contextmanager` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 130-156 / 第 130-156 行
````python
class _Formatter:
    def __init__(self, tensor):
        self.floating_dtype = tensor.dtype.is_floating_point
        self.int_mode = True
        self.sci_mode = False
        self.max_width = 1

        with torch.no_grad():
            tensor_view = tensor.reshape(-1)

        if not self.floating_dtype:
            for value in tensor_view:
                value_str = f"{value}"
                self.max_width = max(self.max_width, len(value_str))

        else:
            if tensor.dtype == torch.float4_e2m1fn_x2:  # type: ignore[attr-defined]
                # torch.float4_e2m1fn_x2 is special and does not support the casts necessary
                # to print it, we choose to display the uint8 representation here for
                # convenience of being able to print a tensor.
                # TODO(#146647): extend this to other dtypes without casts defined, such
                # as the bits, uint1..7 and int1..7 dtypes.
                tensor_view = tensor_view.view(torch.uint8)

            nonzero_finite_vals = torch.masked_select(
                tensor_view, torch.isfinite(tensor_view) & tensor_view.ne(0)
            )
````
- **EN**: It introduces or extends `_Formatter`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `_Formatter`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 158-179 / 第 158-179 行
````python
            if nonzero_finite_vals.numel() == 0:
                # no valid number, do nothing
                return

            if tensor.dtype == torch.float8_e8m0fnu:  # type: ignore[attr-defined]
                # float8_e8m0fnu is special and does not define arithmetic ops,
                # and printing code further in this file assumes the existence
                # of various arithmetic ops to figure out what to print. We hack
                # and convert to float here to make printing work correctly.
                # TODO(#113663): also add the other float8 dtypes here after arithmetic
                # support for them is removed
                nonzero_finite_vals = nonzero_finite_vals.float()

            # Convert to double (or float) for easy calculation. HalfTensor overflows with 1e8, and there's no div() on CPU.
            nonzero_finite_abs = tensor_totype(nonzero_finite_vals.abs())
            nonzero_finite_min = tensor_totype(nonzero_finite_abs.min())
            nonzero_finite_max = tensor_totype(nonzero_finite_abs.max())

            for value in nonzero_finite_vals:
                if value != torch.ceil(value):
                    self.int_mode = False
                    break
````
- **EN**: This chunk continues `_Formatter` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_Formatter`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 181-208 / 第 181-208 行
````python
            self.sci_mode = (
                nonzero_finite_max / nonzero_finite_min > 1000.0
                or nonzero_finite_max > 1.0e8
                or nonzero_finite_min < 1.0e-4
                if PRINT_OPTS.sci_mode is None
                else PRINT_OPTS.sci_mode
            )

            if self.int_mode:
                # in int_mode for floats, all numbers are integers, and we append a decimal to nonfinites
                # to indicate that the tensor is of floating type. add 1 to the len to account for this.
                if self.sci_mode:
                    for value in nonzero_finite_vals:
                        value_str = f"{{:.{PRINT_OPTS.precision}e}}".format(value)
                        self.max_width = max(self.max_width, len(value_str))
                else:
                    for value in nonzero_finite_vals:
                        value_str = f"{value:.0f}"
                        self.max_width = max(self.max_width, len(value_str) + 1)
            else:
                # Check if scientific representation should be used.
                if self.sci_mode:
                    for value in nonzero_finite_vals:
                        value_str = f"{{:.{PRINT_OPTS.precision}e}}".format(value)
                        self.max_width = max(self.max_width, len(value_str))
                else:
                    for value in nonzero_finite_vals:
                        value_str = f"{{:.{PRINT_OPTS.precision}f}}".format(value)
````
- **EN**: This chunk continues `_Formatter` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_Formatter`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 209-226 / 第 209-226 行
````python
                        self.max_width = max(self.max_width, len(value_str))

    def width(self):
        return self.max_width

    def format(self, value):
        if self.floating_dtype:
            if self.sci_mode:
                ret = f"{{:{self.max_width}.{PRINT_OPTS.precision}e}}".format(value)
            elif self.int_mode:
                ret = f"{value:.0f}"
                if not (math.isinf(value) or math.isnan(value)):
                    ret += "."
            else:
                ret = f"{{:.{PRINT_OPTS.precision}f}}".format(value)
        else:
            ret = f"{value}"
        return (self.max_width - len(ret)) * " " + ret
````
- **EN**: This chunk defines `format`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `format`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 229-251 / 第 229-251 行
````python
def _scalar_str(self, formatter1, formatter2=None):
    if formatter2 is not None:
        real_str = _scalar_str(self.real, formatter1)
        imag_str = (_scalar_str(self.imag, formatter2) + "j").lstrip()
        # handles negative numbers, +0.0, -0.0
        if imag_str[0] == "+" or imag_str[0] == "-":
            return real_str + imag_str
        else:
            return real_str + "+" + imag_str
    else:
        return formatter1.format(self.item())


def _vector_str(self, indent, summarize, formatter1, formatter2=None):
    # length includes spaces and comma between elements
    element_length = formatter1.width() + 2
    if formatter2 is not None:
        # width for imag_formatter + an extra j for complex
        element_length += formatter2.width() + 1

    elements_per_line = max(
        1, math.floor((PRINT_OPTS.linewidth - indent) / (element_length))
    )
````
- **EN**: This chunk defines `_vector_str`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_vector_str`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 253-271 / 第 253-271 行
````python
    def _val_formatter(val, formatter1=formatter1, formatter2=formatter2):
        if formatter2 is not None:
            real_str = formatter1.format(val.real)
            imag_str = (formatter2.format(val.imag) + "j").lstrip()
            # handles negative numbers, +0.0, -0.0
            if imag_str[0] == "+" or imag_str[0] == "-":
                return real_str + imag_str
            else:
                return real_str + "+" + imag_str
        else:
            return formatter1.format(val)

    if self.dtype == torch.float4_e2m1fn_x2:  # type: ignore[attr-defined]
        # torch.float4_e2m1fn_x2 is special and does not support the casts necessary
        # to print it, we choose to display the uint8 representation here for
        # convenience of being able to print a tensor.
        # TODO(#146647): extend this to other dtypes without casts defined, such
        # as the bits, uint1..7 and int1..7 dtypes.
        self = self.view(torch.uint8)
````
- **EN**: This chunk defines `_val_formatter`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_val_formatter`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 273-299 / 第 273-299 行
````python
    if summarize and not PRINT_OPTS.edgeitems:
        # Deal with edge case that negative zero is zero
        data = ["..."]
    elif summarize and self.size(0) > 2 * PRINT_OPTS.edgeitems:
        data = (
            [_val_formatter(val) for val in self[: PRINT_OPTS.edgeitems].tolist()]
            + [" ..."]
            + [_val_formatter(val) for val in self[-PRINT_OPTS.edgeitems :].tolist()]
        )
    else:
        data = [_val_formatter(val) for val in self.tolist()]

    data_lines = [
        data[i : i + elements_per_line] for i in range(0, len(data), elements_per_line)
    ]
    lines = [", ".join(line) for line in data_lines]
    return "[" + ("," + "\n" + " " * (indent + 1)).join(lines) + "]"


# formatter2 is only used for printing complex tensors.
# For complex tensors, formatter1 and formatter2 are the formatters for tensor.real
# and tensor.imag respesectively
def _tensor_str_with_formatter(self, indent, summarize, formatter1, formatter2=None):
    dim = self.dim()

    if dim == 0:
        return _scalar_str(self, formatter1, formatter2)
````
- **EN**: This chunk defines `_tensor_str_with_formatter`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_tensor_str_with_formatter`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 301-326 / 第 301-326 行
````python
    if dim == 1:
        return _vector_str(self, indent, summarize, formatter1, formatter2)

    if summarize and self.size(0) > 2 * PRINT_OPTS.edgeitems:
        slices = (
            [
                _tensor_str_with_formatter(
                    self[i], indent + 1, summarize, formatter1, formatter2
                )
                for i in range(PRINT_OPTS.edgeitems)
            ]
            + ["..."]
            + [
                _tensor_str_with_formatter(
                    self[i], indent + 1, summarize, formatter1, formatter2
                )
                for i in range(len(self) - PRINT_OPTS.edgeitems, len(self))
            ]
        )
    else:
        slices = [
            _tensor_str_with_formatter(
                self[i], indent + 1, summarize, formatter1, formatter2
            )
            for i in range(self.size(0))
        ]
````
- **EN**: This chunk continues `_tensor_str_with_formatter` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_tensor_str_with_formatter`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 328-351 / 第 328-351 行
````python
    tensor_str = ("," + "\n" * (dim - 1) + " " * (indent + 1)).join(slices)
    return "[" + tensor_str + "]"


def _tensor_str(self, indent):
    if self.numel() == 0:
        return "[]"

    if self.has_names():
        # There are two main codepaths (possibly more) that tensor printing goes through:
        # - tensor data can fit comfortably on screen
        # - tensor data needs to be summarized
        # Some of the codepaths don't fully support named tensors, so we send in
        # an unnamed tensor to the formatting code as a workaround.
        self = self.rename(None)

    summarize = self.numel() > PRINT_OPTS.threshold

    if self._is_zerotensor():
        self = self.clone()

    # handle the negative bit
    if self.is_neg():
        self = self.resolve_neg()
````
- **EN**: This chunk defines `_tensor_str`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_tensor_str`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 353-376 / 第 353-376 行
````python
    # TODO: Remove me when `masked_select` is implemented for FP8
    if self.dtype in [
        torch.float8_e5m2,
        torch.float8_e5m2fnuz,
        torch.float8_e4m3fn,
        torch.float8_e4m3fnuz,
    ]:
        self = self.half()

    if self.dtype.is_complex:
        # handle the conjugate bit
        self = self.resolve_conj()
        real_formatter = _Formatter(
            get_summarized_data(self.real) if summarize else self.real
        )
        imag_formatter = _Formatter(
            get_summarized_data(self.imag) if summarize else self.imag
        )
        return _tensor_str_with_formatter(
            self, indent, summarize, real_formatter, imag_formatter
        )
    else:
        formatter = _Formatter(get_summarized_data(self) if summarize else self)
        return _tensor_str_with_formatter(self, indent, summarize, formatter)
````
- **EN**: This chunk continues `_tensor_str` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_tensor_str`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 379-406 / 第 379-406 行
````python
def _add_suffixes(tensor_str, suffixes, indent, force_newline):
    tensor_strs = [tensor_str]
    last_line_len = len(tensor_str) - tensor_str.rfind("\n") + 1
    for suffix in suffixes:
        suffix_len = len(suffix)
        if force_newline or last_line_len + suffix_len + 2 > PRINT_OPTS.linewidth:
            tensor_strs.append(",\n" + " " * indent + suffix)
            last_line_len = indent + suffix_len
            force_newline = False
        else:
            tensor_strs.append(", " + suffix)
            last_line_len += suffix_len + 2
    tensor_strs.append(")")
    return "".join(tensor_strs)


def get_summarized_data(self):
    dim = self.dim()
    if dim == 0:
        return self
    if dim == 1:
        if self.size(0) > 2 * PRINT_OPTS.edgeitems:
            return torch.cat(
                (self[: PRINT_OPTS.edgeitems], self[-PRINT_OPTS.edgeitems :])
            )
        else:
            return self
    if not PRINT_OPTS.edgeitems:
````
- **EN**: This chunk defines `get_summarized_data`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_summarized_data`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 407-430 / 第 407-430 行
````python
        return self.new_empty([0] * self.dim())
    elif self.size(0) > 2 * PRINT_OPTS.edgeitems:
        start = [self[i] for i in range(PRINT_OPTS.edgeitems)]
        end = [self[i] for i in range(len(self) - PRINT_OPTS.edgeitems, len(self))]
        return torch.stack([get_summarized_data(x) for x in (start + end)])
    else:
        return torch.stack([get_summarized_data(x) for x in self])


def _str_intern(inp, *, tensor_contents=None):
    if torch._C._functorch.is_functorch_wrapped_tensor(inp):
        return _functorch_wrapper_str_intern(inp, tensor_contents=tensor_contents)
    is_plain_tensor = type(inp) is torch.Tensor or type(inp) is torch.nn.Parameter
    if inp.is_nested:
        prefix = "nested_tensor("
    elif is_plain_tensor:
        prefix = "tensor("
    else:
        prefix = f"{type(inp).__name__}("
    indent = len(prefix)
    suffixes = []
    custom_contents_provided = tensor_contents is not None
    if custom_contents_provided:
        tensor_str = tensor_contents
````
- **EN**: This chunk defines `_str_intern`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_str_intern`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 432-458 / 第 432-458 行
````python
    # This is used to extract the primal value and thus disable the forward AD
    # within this function.
    # TODO(albanD) This needs to be updated when more than one level is supported
    self, tangent = torch.autograd.forward_ad.unpack_dual(inp)

    # Note [Print tensor device]:
    # A general logic here is we only print device when it doesn't match
    # the device specified in default tensor type.
    # Currently torch.set_default_tensor_type() only supports CPU/CUDA, thus
    # torch._C._get_default_device() only returns either cpu or cuda.
    # In other cases, we don't have a way to set them as default yet,
    # and we should always print out device for them.
    if (
        self.device.type != torch._C._get_default_device()
        or (
            self.device.type == "cuda"
            and torch.cuda.current_device() != self.device.index
        )
        or (self.device.type == "mps")
    ):
        suffixes.append("device='" + str(self.device) + "'")

    # Tensor printing performs tensor operations like slice, indexing, etc to make it in a
    # representable format. These operations on ipu/xla/lazy/mtia tensor results in compilations. Hence,
    # to avoid compilations, copying the tensor to cpu before printing.
    if self.device.type in ["xla", "lazy", "ipu", "mtia"]:
        self = self.to("cpu")
````
- **EN**: This chunk continues `_str_intern` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 460-487 / 第 460-487 行
````python
    # TODO: add an API to map real -> complex dtypes
    _default_complex_dtype = (
        torch.cdouble if torch.get_default_dtype() == torch.double else torch.cfloat
    )
    has_default_dtype = self.dtype in (
        torch.get_default_dtype(),
        _default_complex_dtype,
        torch.int64,
        torch.bool,
    )
    if self.is_sparse:
        suffixes.append("size=" + str(tuple(self.shape)))
        from torch._subclasses.fake_tensor import FakeTensor

        is_meta = self.is_meta or isinstance(self, FakeTensor)
        if not is_meta:
            suffixes.append("nnz=" + str(self._nnz()))
        if not has_default_dtype:
            suffixes.append("dtype=" + str(self.dtype))
        if not custom_contents_provided:
            indices_prefix = "indices=tensor("
            indices = self._indices().detach()
            if is_meta:
                indices_str = "..."
            else:
                indices_str = _tensor_str(indices, indent + len(indices_prefix))
            if is_meta or indices.numel() == 0:
                indices_str += ", size=" + str(tuple(indices.shape))
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._subclasses.fake_tensor. This chunk continues `_str_intern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._subclasses.fake_tensor。 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 488-511 / 第 488-511 行
````python
            values_prefix = "values=tensor("
            values = self._values().detach()
            if is_meta:
                values_str = "..."
            else:
                values_str = _tensor_str(values, indent + len(values_prefix))
            if is_meta or values.numel() == 0:
                values_str += ", size=" + str(tuple(values.shape))
            tensor_str = (
                indices_prefix
                + indices_str
                + "),\n"
                + " " * indent
                + values_prefix
                + values_str
                + ")"
            )
    elif self.layout in {
        torch.sparse_csr,
        torch.sparse_csc,
        torch.sparse_bsr,
        torch.sparse_bsc,
    }:
        from torch._subclasses.fake_tensor import FakeTensor
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._subclasses.fake_tensor. This chunk continues `_str_intern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._subclasses.fake_tensor。 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 513-540 / 第 513-540 行
````python
        suffixes.append("size=" + str(tuple(self.shape)))
        is_meta = self.is_meta or isinstance(self, FakeTensor)
        if not is_meta:
            suffixes.append("nnz=" + str(self._nnz()))
        if not has_default_dtype:
            suffixes.append("dtype=" + str(self.dtype))
        if not custom_contents_provided:
            compressed_indices_method, plain_indices_method = {
                torch.sparse_csr: (torch.Tensor.crow_indices, torch.Tensor.col_indices),
                torch.sparse_csc: (torch.Tensor.ccol_indices, torch.Tensor.row_indices),
                torch.sparse_bsr: (torch.Tensor.crow_indices, torch.Tensor.col_indices),
                torch.sparse_bsc: (torch.Tensor.ccol_indices, torch.Tensor.row_indices),
            }[self.layout]
            if self.layout in {torch.sparse_csr, torch.sparse_bsr}:
                cdimname, pdimname = "row", "column"
            else:
                cdimname, pdimname = "column", "row"
            compressed_indices_prefix = f"c{cdimname[:3]}_indices=tensor("
            compressed_indices = compressed_indices_method(self).detach()
            if is_meta:
                compressed_indices_str = "..."
            else:
                compressed_indices_str = _tensor_str(
                    compressed_indices, indent + len(compressed_indices_prefix)
                )
            if compressed_indices.numel() == 0 or is_meta:
                compressed_indices_str += ", size=" + str(
                    tuple(compressed_indices.shape)
````
- **EN**: This chunk continues `_str_intern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 541-568 / 第 541-568 行
````python
                )
            plain_indices_prefix = f"{pdimname[:3]}_indices=tensor("
            plain_indices = plain_indices_method(self).detach()
            if is_meta:
                plain_indices_str = "..."
            else:
                plain_indices_str = _tensor_str(
                    plain_indices, indent + len(plain_indices_prefix)
                )
            if plain_indices.numel() == 0 or is_meta:
                plain_indices_str += ", size=" + str(tuple(plain_indices.shape))
            values_prefix = "values=tensor("
            values = self.values().detach()
            if is_meta:
                values_str = "..."
            else:
                values_str = _tensor_str(values, indent + len(values_prefix))
            if values.numel() == 0 or is_meta:
                values_str += ", size=" + str(tuple(values.shape))
            tensor_str = (
                compressed_indices_prefix
                + compressed_indices_str
                + "),\n"
                + " " * indent
                + plain_indices_prefix
                + plain_indices_str
                + "),\n"
                + " " * indent
````
- **EN**: This chunk continues `_str_intern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 569-595 / 第 569-595 行
````python
                + values_prefix
                + values_str
                + ")"
            )
    elif self.is_quantized:
        suffixes.append("size=" + str(tuple(self.shape)))
        if not has_default_dtype:
            suffixes.append("dtype=" + str(self.dtype))
        suffixes.append("quantization_scheme=" + str(self.qscheme()))
        if (
            self.qscheme() == torch.per_tensor_affine
            or self.qscheme() == torch.per_tensor_symmetric
        ):
            suffixes.append("scale=" + str(self.q_scale()))
            suffixes.append("zero_point=" + str(self.q_zero_point()))
        elif (
            self.qscheme() == torch.per_channel_affine
            or self.qscheme() == torch.per_channel_symmetric
            or self.qscheme() == torch.per_channel_affine_float_qparams
        ):
            suffixes.append("scale=" + str(self.q_per_channel_scales()))
            suffixes.append("zero_point=" + str(self.q_per_channel_zero_points()))
            suffixes.append("axis=" + str(self.q_per_channel_axis()))
        if not custom_contents_provided:
            tensor_str = _tensor_str(self.dequantize(), indent)
    elif self.is_nested:
        if not custom_contents_provided:
````
- **EN**: This chunk continues `_str_intern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_str_intern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 597-624 / 第 597-624 行
````python
            def indented_str(s, indent):
                return "\n".join(f"  {line}" for line in s.split("\n"))

            strs = ",\n".join(
                indented_str(str(t), indent + 1)
                for t in torch.ops.aten.unbind.int(self, 0)
            )
            tensor_str = f"[\n{strs}\n]"
    elif torch._is_functional_tensor(self):
        prefix = "_to_functional_tensor("
        tensor_str = repr(torch._from_functional_tensor(self))
    else:
        # Circular import problem, so we import it here
        from torch._subclasses.fake_tensor import FakeTensor

        if self.is_meta or isinstance(self, FakeTensor):
            suffixes.append("size=" + str(tuple(self.shape)))
            if self.dtype != torch.get_default_dtype():
                suffixes.append("dtype=" + str(self.dtype))
            # TODO: This implies that ellipses is valid syntax for allocating
            # a meta tensor or FakeTensor, which it could be, but it isn't right now
            if not custom_contents_provided:
                tensor_str = "..."
        else:
            if self.numel() == 0 and not self.is_sparse:
                # Explicitly print the shape if it is not (0,), to match NumPy behavior
                if self.dim() != 1:
                    suffixes.append("size=" + str(tuple(self.shape)))
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._subclasses.fake_tensor. This chunk defines `indented_str`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._subclasses.fake_tensor。 这一段定义了 `indented_str`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 626-646 / 第 626-646 行
````python
                # In an empty tensor, there are no elements to infer if the dtype
                # should be int64, so it must be shown explicitly.
                if self.dtype != torch.get_default_dtype():
                    suffixes.append("dtype=" + str(self.dtype))
                if not custom_contents_provided:
                    tensor_str = "[]"
            else:
                if not PRINT_OPTS.edgeitems:
                    suffixes.append("size=" + str(tuple(self.shape)))

                if not has_default_dtype:
                    suffixes.append("dtype=" + str(self.dtype))

                if not custom_contents_provided:
                    if self.layout != torch.strided:
                        tensor_str = _tensor_str(self.to_dense(), indent)
                    else:
                        tensor_str = _tensor_str(self, indent)

    if self.layout != torch.strided:
        suffixes.append("layout=" + str(self.layout))
````
- **EN**: This chunk continues `indented_str` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `indented_str`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 648-672 / 第 648-672 行
````python
    # Use inp here to get the original grad_fn and not the one generated by the forward grad
    # unpacking.
    grad_fn_name = None
    try:
        grad_fn = inp.grad_fn
    except RuntimeError:
        # Accessing the grad_fn calls rebasing logic which would cause an error
        # if that tensor is a view created in no-grad mode modified in-place in
        # no-grad mode. See: https://github.com/pytorch/pytorch/issues/99968
        grad_fn_name = "Invalid"

    if grad_fn_name is None and grad_fn is not None:  # type: ignore[possibly-undefined]
        # pyrefly: ignore [unbound-name]
        grad_fn_name = type(grad_fn).__name__
        if grad_fn_name == "CppFunction":
            # pyrefly: ignore [unbound-name]
            grad_fn_name = grad_fn.name().rsplit("::", 1)[-1]

    if grad_fn_name is not None:
        suffixes.append(f"grad_fn=<{grad_fn_name}>")
    elif inp.requires_grad:
        suffixes.append("requires_grad=True")

    if self.has_names():
        suffixes.append(f"names={self.names}")
````
- **EN**: This chunk continues `indented_str` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `indented_str`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 674-697 / 第 674-697 行
````python
    if tangent is not None:
        suffixes.append(f"tangent={tangent}")

    string_repr = _add_suffixes(
        prefix + tensor_str,  # type: ignore[possibly-undefined]
        suffixes,
        indent,
        force_newline=self.is_sparse,
    )

    # Check if this instance is flagged as a parameter and change the repr accordingly.
    # Unfortunately, this function has to be aware of this detail.
    # NB: This is currently skipped for plain tensor parameters to maintain BC. In the future,
    # this should be done for those as well to produce a valid repr.
    if isinstance(self, torch.nn.Parameter) and not is_plain_tensor:
        string_repr = f"Parameter({string_repr})"

    return string_repr


def _functorch_wrapper_str_intern(tensor, *, tensor_contents=None):
    level = torch._C._functorch.maybe_get_level(tensor)
    if level == -1:
        raise AssertionError("expected functorch level to be >= 0, got -1")
````
- **EN**: This chunk defines `_functorch_wrapper_str_intern`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_functorch_wrapper_str_intern`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 699-726 / 第 699-726 行
````python
    if torch._C._functorch.is_functionaltensor(tensor):
        # Since we're unwrapping the FunctionalTensorWrapper, we need to make sure
        # that it's up to date first
        torch._sync(tensor)

    value = torch._C._functorch.get_unwrapped(tensor)
    value_repr = repr(value)

    indented_value_repr = textwrap.indent(value_repr, " " * 4)
    if torch._C._functorch.is_batchedtensor(tensor):
        bdim = torch._C._functorch.maybe_get_bdim(tensor)
        if bdim == -1:
            raise AssertionError("expected batch dimension to be >= 0, got -1")
        return (
            f"BatchedTensor(lvl={level}, bdim={bdim}, value=\n{indented_value_repr}\n)"
        )
    if torch._C._functorch.is_gradtrackingtensor(tensor):
        return f"GradTrackingTensor(lvl={level}, value=\n{indented_value_repr}\n)"
    if torch._C._functorch.is_functionaltensor(tensor):
        return f"FunctionalTensor(lvl={level}, value=\\\n{value_repr})"

    raise ValueError("We don't know how to print this, please file us an issue")


def _str(self, *, tensor_contents=None):
    with torch.no_grad(), torch.utils._python_dispatch._disable_current_modes():
        guard = torch._C._DisableFuncTorch()  # noqa: F841
        return _str_intern(self, tensor_contents=tensor_contents)
````
- **EN**: This chunk defines `_str`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_str`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **__PrinterOptions**
  - EN: `__PrinterOptions` is one of the main symbols declared or implemented in this file.
  - CN: `__PrinterOptions` 是本文件声明或实现的主要符号之一。
- **PRINT_OPTS**
  - EN: `PRINT_OPTS` is one of the main symbols declared or implemented in this file.
  - CN: `PRINT_OPTS` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **NumPy compatibility**
  - EN: The implementation mirrors NumPy-style semantics to keep array/tensor behavior familiar.
  - CN: 实现对齐 NumPy 风格语义，使数组/张量行为更易理解。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._subclasses.fake_tensor`
- **Standard library / 标准库**: `contextlib`, `dataclasses`, `math`, `textwrap`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__PrinterOptions`, `PRINT_OPTS`, `set_printoptions`, `get_printoptions`, `printoptions`, `tensor_totype`, `_Formatter`, `_scalar_str`, `_vector_str`, `_tensor_str_with_formatter`
