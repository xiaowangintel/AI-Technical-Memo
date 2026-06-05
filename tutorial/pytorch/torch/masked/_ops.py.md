# _ops.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/masked/_ops.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements masked-tensor helpers and namespace wrappers for masked operations.
- **Purpose (CN)**: 实现 masked tensor 辅助逻辑以及 masked 运算的命名空间包装层。
## Line-by-Line Analysis / 逐行分析

### Lines 1-34 / 第 1-34 行
````python
# mypy: allow-untyped-defs
import warnings
from collections.abc import Callable
from typing import Any, TYPE_CHECKING, TypeAlias, TypeVar
from typing_extensions import ParamSpec

import torch
from torch import sym_float, Tensor
from torch._prims_common import corresponding_real_dtype
from torch.masked import _docs
from torch.masked.maskedtensor.core import is_masked_tensor, MaskedTensor
from torch.masked.maskedtensor.creation import as_masked_tensor


if TYPE_CHECKING:
    from torch._prims_common import DimsType
    from torch.types import _dtype as DType

    DimOrDims: TypeAlias = DimsType | None
else:
    # The JIT doesn't understand Union, nor torch.dtype here
    DType = int
    DimOrDims = tuple[int, ...] | None


__all__: list[str] = []

_T = TypeVar("_T")
_P = ParamSpec("_P")

# All masked reduction/normalization operations have the same
# signatures. Here we introduce docstring templates that are applied
# to docstrings of reduction/normalization functions via
# _apply_docstring_templates decorator.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._prims_common, torch.masked, ...; standard-library helpers such as warnings, collections.abc, typing; other helper packages such as typing_extensions. `__all__` defines the public symbols that this module chooses to export. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._prims_common、torch.masked、...；标准库辅助模块，如 warnings、collections.abc、typing；其他辅助包，如 typing_extensions。 `__all__` 定义了本模块选择导出的公共符号。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 37-72 / 第 37-72 行
````python
def _apply_docstring_templates(func: Callable[_P, _T]) -> Callable[_P, _T]:
    """Decorator that applies docstring templates to function docstring
    and returns the function instance.
    """

    doc_string = getattr(_docs, f"{func.__name__}_docstring", None)
    if doc_string is None:
        warnings.warn(
            f"No documentation string available for {func.__name__}."
            " PyTorch team should run `python tools/update_masked_docs.py`"
            " to generate the missing docstrings.",
            stacklevel=2,
        )
    else:
        func.__doc__ = doc_string

    # Expose function as public symbol
    __all__.append(func.__name__)

    return func


def _generate_docstring(func):
    """A utility function called from tools/update_masked_docs.py
    script to update the module torch.masked._docs.py
    """
    docstring_templates = dict(
        reduction_signature="""\
{function_name}(input, {operation_args}, *, {operation_kwargs}) -> Tensor""",
        reduction_descr="""\
Returns {operation name} of all the elements in the :attr:`input`
tensor along the given dimension(s) :attr:`dim` while the :attr:`input`
elements are masked out according to the boolean tensor
:attr:`mask`.""",
        reduction_args="""\
If :attr:`keepdim` is ``True``, the output tensor is of the same size
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. `__all__` defines the public symbols that this module chooses to export. This chunk defines `_generate_docstring`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `_generate_docstring`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 73-107 / 第 73-107 行
````python
as :attr:`input` except in the dimension(s) :attr:`dim` where it is of
size 1. Otherwise, :attr:`dim` is squeezed (see
:func:`torch.squeeze`), resulting in the output tensor having 1 (or
``len(dim)``) fewer dimension(s).

The boolean tensor :attr:`mask` defines the "validity" of
:attr:`input` tensor elements: if :attr:`mask` element is True
then the corresponding element in :attr:`input` tensor will be
included in {operation name} computation, otherwise the element is
ignored.

When all elements of :attr:`input` along the given dimension
:attr:`dim` are ignored (fully masked-out), the corresponding element
of the output tensor will have undefined value: it may or may not
correspond to the identity value of {operation name} operation; the
choice may correspond to the value that leads to the most efficient
storage of :attr:`output` tensor.

The mask of the output tensor can be computed as
``torch.any(torch.broadcast_to(mask, input.shape), dim, keepdim=keepdim,
dtype=torch.bool)``.

The shapes of the :attr:`mask` tensor and the :attr:`input` tensor
don't need to match, but they must be :ref:`broadcastable
<broadcasting-semantics>` and the dimensionality of the :attr:`mask`
tensor must not be greater than of the :attr:`input` tensor.

Args:
    input (Tensor): the input tensor
    {args_declarations}

Keyword args:
    {kwargs_declarations}""",
        reduction_example="""\
Example::
````
- **EN**: Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 109-141 / 第 109-141 行
````python
    >>> input = {example_input}
    >>> input
    {indent_example_input}
    >>> mask = {example_mask}
    >>> mask
    {indent_example_mask}
    >>> {full_function_name}(input, {example_args}, mask=mask)
    {indent_example_output}
""",
        reduction_identity="""\
The identity value of {operation name} operation, which is used to start the reduction, is ``{identity_int32}``.""",
        reduction_identity_dtype="""\
The identity value of {operation name} operation, which is used to start the
reduction, depends on input dtype. For instance, for float32, uint8,
and int32 dtypes, the identity values are ``{identity_float32}``, ``{identity_uint8}``, and ``{identity_int32}``, respectively.""",
        normalization_signature="""\
{function_name}(input, {operation_args}, *, {operation_kwargs}) -> Tensor""",
        normalization_descr="""\
Returns {operation name} of all the slices in the :attr:`input` tensor
along :attr:`dim` while the :attr:`input` elements are masked out
according to the boolean tensor :attr:`mask`.

{definition}""",
        normalization_args="""\
The boolean tensor :attr:`mask` defines the "validity" of
:attr:`input` tensor elements: if :attr:`mask` element is True then
the corresponding element in :attr:`input` tensor will be included in
{operation name} computation, otherwise the element is ignored.

The values of masked-out elements of the output tensor have undefined
value: it may or may not be set to zero or nan; the choice may correspond to
the value that leads to the most efficient storage of :attr:`output`
tensor.
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 143-169 / 第 143-169 行
````python
The mask of the {operation name} output tensor can be computed as
``torch.broadcast_to(mask, input.shape)``.

The shapes of the :attr:`mask` tensor and the :attr:`input` tensor
don't need to match, but they must be :ref:`broadcastable
<broadcasting-semantics>` and the dimensionality of the :attr:`mask`
tensor must not be greater than of the :attr:`input` tensor.

Args:
    input (Tensor): the input tensor
    {args_declarations}

Keyword args:
    {kwargs_declarations}""",
        normalization_example="""\
Example::

    >>> input = {example_input}
    >>> input
    {indent_example_input}
    >>> mask = {example_mask}
    >>> mask
    {indent_example_mask}
    >>> {full_function_name}(input, {example_args}, mask=mask)
    {indent_example_output}
""",
    )
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 171-204 / 第 171-204 行
````python
    args_and_kwargs = {
        # argument name sufficies separated by double underscore will
        # be removed in the final documentation string.
        "sum": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "prod": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "cumsum": (("dim__as_int",), ("dtype=None", "mask=None")),
        "cumprod": (("dim__as_int",), ("dtype=None", "mask=None")),
        "amin": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "amax": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "argmin": (("dim__as_int",), ("keepdim=False", "dtype=None", "mask=None")),
        "argmax": (("dim__as_int",), ("keepdim=False", "dtype=None", "mask=None")),
        "mean": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "median": (("dim__as_int",), ("keepdim=False", "dtype=None", "mask=None")),
        "norm": (
            (
                "ord",
                "dim",
            ),
            ("keepdim=False", "dtype=None", "mask=None"),
        ),
        "var": (("dim", "unbiased"), ("keepdim=False", "dtype=None", "mask=None")),
        "std": (("dim", "unbiased"), ("keepdim=False", "dtype=None", "mask=None")),
        "logsumexp": (("dim",), ("keepdim=False", "dtype=None", "mask=None")),
        "softmax": (("dim__as_int",), ("dtype=None", "mask=None")),
        "log_softmax": (("dim__as_int",), ("dtype=None", "mask=None")),
        "softmin": (("dim__as_int",), ("dtype=None", "mask=None")),
        "normalize": (
            (
                "ord__required",
                "dim__as_int",
            ),
            ("eps=1e-12", "dtype=None", "mask=None"),
        ),
    }
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。

### Lines 206-236 / 第 206-236 行
````python
    argument_declarations = {
        "dim": """\
    dim (int or tuple of ints, optional): the dimension or dimensions to reduce.
    Default: None that is equivalent to ``tuple(range(input.ndim))``.""",
        "dim__as_int": """\
    dim (int): the dimension along which {operation name} is computed.""",
        "ord": """\
    ord (int, float, optional): the order of vector norm. Default: 2.
    See :func:`torch.linalg.vector_norm` for a list of supported norms.""",
        "ord__required": """\
    ord (int, float): the order of vector norm. Default: 2.
    See :func:`torch.linalg.vector_norm` for a list of supported norms.""",
        "unbiased": """\
    unbiased (bool): when True, use Bessel's correction, otherwise, compute
    the uncorrected sample variance.""",
        "eps": """\
    eps (float, optional): small value to avoid division by zero. Default: {default}.""",
        "keepdim": """\
    keepdim (bool, optional): whether the output tensor has
    :attr:`dim` retained or not. Default: {default}.""",
        "dtype": """\
    dtype (:class:`torch.dtype`, optional): the desired data type
    of returned tensor.  If specified, the input tensor is
    casted to :attr:`dtype` before the operation is
    performed. Default: {default}.""",
        "mask": """\
    mask (:class:`torch.Tensor`, optional): the boolean tensor
    containing the binary mask of validity of input tensor
    elements.
    Default: None that is equivalent to ``torch.ones(input.shape, dtype=torch.bool)``.""",
    }
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。

### Lines 238-263 / 第 238-263 行
````python
    definitions = {
        "softmax": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. Softmax of i-th element in ``x`` is
    defined as ``exp(x[i])/sum(exp(x))``.""",
        "log_softmax": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. LogSoftmax of i-th element in ``x`` is
    defined as ``log(exp(x[i])/sum(exp(x)))``.""",
        "softmin": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. Softmin of i-th element in ``x`` is
    defined as ``exp(-x[i])/sum(exp(-x))``.""",
        "normalize": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. Normalize of i-th element in ``x`` is
    defined as ``x[i]/max(norm(x, p), eps)``.""",
        "cumsum": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. Cumsum of i-th element in ``x`` is
    defined as ``sum(x[:i])``.""",
        "cumprod": """\
    Let ``x`` be a sequence of unmasked elements of one-dimensional slice
    of the :attr:`input` tensor. Cumsum of i-th element in ``x`` is
    defined as ``prod(x[:i])``.""",
    }
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。

### Lines 265-291 / 第 265-291 行
````python
    reduction_names = {
        "sum": "sum",
        "prod": "product",
        "amax": "maximum",
        "amin": "minimum",
        "argmax": "argmax",
        "argmin": "argmin",
        "mean": "mean",
        "median": "median",
        "norm": "norm",
        "var": "variance",
        "std": "standard_deviation",
        "logsumexp": "logsumexp",
    }

    normalization_names = {
        "softmax": "softmax",
        "log_softmax": "log_softmax",
        "softmin": "softmin",
        "normalize": "normalize",
        "cumsum": "cumulative_sum",
        "cumprod": "cumulative_prod",
    }

    operation_names = {}
    operation_names.update(reduction_names)
    operation_names.update(normalization_names)
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。

### Lines 293-327 / 第 293-327 行
````python
    # Default example data:
    example_dim = 1
    example_input = torch.tensor([[-3, -2, -1], [0, 1, 2]])
    example_mask = torch.tensor([[True, False, True], [False, False, False]])
    example_args: tuple[Any, ...]
    if func.__name__ in {"norm", "normalize"}:
        example_args = (2.0, example_dim)
        example_input = example_input.to(dtype=torch.float32)
    elif func.__name__ in {"var", "std"}:
        example_args = (example_dim, False)
    elif func.__name__ == "median":
        example_args = (example_dim,)
        example_input = example_input.to(dtype=torch.float32)
    else:
        example_args = (example_dim,)

    operation_args: tuple[str, ...]
    operation_kwargs: tuple[str, ...]
    operation_args, operation_kwargs = args_and_kwargs[func.__name__]
    arg_declarations = [
        "\n    ".join(
            argument_declarations.get(a, f"{a.split('__', 1)[0]}: TBD.").splitlines()
        )
        for a in operation_args
    ]
    kwarg_declarations = [
        "\n    ".join(
            argument_declarations.get(
                a.split("=", 1)[0], f"{a.split('__', 1)[0]}: TBD."
            )
            .format(default=a.split("=", 1)[1])
            .splitlines()
        )
        for a in operation_kwargs
    ]
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 329-355 / 第 329-355 行
````python
    if func.__name__ in reduction_names:
        op_kind = "reduction"
        doc_sections = ["signature", "descr", "identity", "args", "example"]
    elif func.__name__ in normalization_names:
        op_kind = "normalization"
        doc_sections = ["signature", "descr", "args", "example"]
        example_input = example_input.to(dtype=torch.float32)
    else:
        # add function name to operation names dictionaries
        raise AssertionError(f"unknown function {func.__name__}")
    example_output = func(example_input, *example_args, mask=example_mask)

    template_data = {
        "function_name": func.__name__,
        "full_function_name": func.__module__ + "." + func.__name__,
        "operation name": operation_names[func.__name__],
        "operation_args": ", ".join(a.split("__", 1)[0] for a in operation_args),
        "operation_kwargs": ", ".join(a.split("__", 1)[0] for a in operation_kwargs),
        # one-line representation of a tensor:
        "example_input": " ".join(str(example_input).split()),
        "example_args": ", ".join(map(str, example_args)),
        "example_mask": " ".join(str(example_mask).split()),
        # multi-line representation of a tensor with indent
        "indent_example_input": ("\n    ").join(str(example_input).splitlines()),
        "indent_example_mask": ("\n    ").join(str(example_mask).splitlines()),
        "indent_example_output": ("\n    ").join(str(example_output).splitlines()),
    }
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 357-387 / 第 357-387 行
````python
    if func.__name__ in reduction_names:
        template_data.update(
            identity_uint8=_reduction_identity(
                func.__name__, torch.tensor(0, dtype=torch.uint8)
            ),
            identity_int32=_reduction_identity(
                func.__name__, torch.tensor(0, dtype=torch.int32)
            ),
            identity_float32=_reduction_identity(
                func.__name__, torch.tensor(0, dtype=torch.float32)
            ),
        )
        if func.__name__ == "norm":
            template_data.update(
                identity_ord_ninf=_reduction_identity(
                    func.__name__, torch.tensor(0, dtype=torch.float32), float("-inf")
                )
            )
    elif func.__name__ in normalization_names:
        template_data.update(definition=definitions[func.__name__])
    else:
        # add function name to operation names dictionaries
        raise AssertionError(f"unknown function {func.__name__}")
    template_data.update(
        args_declarations=("\n    ".join(arg_declarations)).format_map(template_data)
    )
    template_data.update(
        kwargs_declarations=("\n    ".join(kwarg_declarations)).format_map(
            template_data
        )
    )
````
- **EN**: This chunk continues `_generate_docstring` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_generate_docstring`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 389-419 / 第 389-419 行
````python
    # Apply function name info to docstring templates:
    templates = {
        k: v.format_map(template_data)
        for k, v in docstring_templates.items()
        if k.startswith(op_kind)
    }
    templates.update(
        (k, v.format_map(template_data) if isinstance(v, str) else v)
        for k, v in template_data.items()
    )

    # Apply docstring templates to function doctring:
    if func.__doc__ is None:
        doc_template = "\n\n".join([f"{{{op_kind}_{sec}}}" for sec in doc_sections])
    else:
        doc_template = func.__doc__
    return doc_template.format_map(templates)


def _reduction_identity(op_name: str, input: Tensor, *args):
    """Return identity value as scalar tensor of a reduction operation on
    given input, or None, if the identity value cannot be uniquely
    defined for the given input.

    The identity value of the operation is defined as the initial
    value to reduction operation that has a property ``op(op_identity,
    value) == value`` for any value in the domain of the operation.
    Or put it another way, including or excluding the identity value in
    a list of operands will not change the reduction result.

    See https://github.com/pytorch/rfcs/pull/27 for more information.
````
- **EN**: This chunk defines `_reduction_identity`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_reduction_identity`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 421-456 / 第 421-456 行
````python
    """
    dtype: DType = input.dtype
    device = input.device
    op_name = op_name.rsplit(".", 1)[-1]  # lstrip module name when present
    if op_name in {"sum", "cumsum"}:
        return torch.tensor(0, dtype=dtype, device=device)
    elif op_name in {"prod", "cumprod"}:
        return torch.tensor(1, dtype=dtype, device=device)
    elif op_name in {"amax", "argmax", "logaddexp"}:
        if torch.is_floating_point(input):
            return torch.tensor(-torch.inf, dtype=dtype, device=device)
        elif torch.is_signed(input) or dtype == torch.uint8:
            return torch.tensor(torch.iinfo(dtype).min, dtype=dtype, device=device)
    elif op_name == "logsumexp":
        if torch.is_floating_point(input):
            return torch.tensor(-torch.inf, dtype=dtype, device=device)
        elif torch.is_complex(input):
            return torch.tensor(-torch.inf + 0j, dtype=dtype, device=device)
        elif torch.is_signed(input) or dtype == torch.uint8:
            return torch.tensor(torch.iinfo(dtype).min, dtype=dtype, device=device)
    elif op_name in {"amin", "argmin"}:
        if torch.is_floating_point(input):
            return torch.tensor(torch.inf, dtype=dtype, device=device)
        elif torch.is_signed(input) or dtype == torch.uint8:
            return torch.tensor(torch.iinfo(dtype).max, dtype=dtype, device=device)
    elif op_name == "mean":
        # Strictly speaking, the identity value of the mean operation
        # is the mean of the input. Since the mean value depends on
        # the dim argument and it may be a non-scalar tensor, we
        # consider the identity value of the mean operation ambiguous.
        # Moreover, the mean value of empty input is undefined.
        return None
    elif op_name == "norm":
        ord = args[0] if args else 2
        if ord == float("-inf"):
            if not torch.is_floating_point(input):
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `_reduction_identity` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `_reduction_identity`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 457-492 / 第 457-492 行
````python
                raise AssertionError(f"input must be floating point, got {input.dtype}")
            return torch.tensor(torch.inf, dtype=dtype, device=device)
        return torch.tensor(0, dtype=dtype, device=device)
    elif op_name == "median":
        # We use NaN for now because the implementation is currently using torch.nanmedian
        # and NaN is the identity for that function since it gets ignored
        dtype = input.dtype if torch.is_floating_point(input) else torch.float
        return torch.tensor(torch.nan, dtype=dtype, device=device)
    elif op_name in {"var", "std"}:
        return None
    raise NotImplementedError(f"identity of {op_name} on {dtype} input")


def _canonical_dim(dim: DimOrDims, ndim: int) -> tuple[int, ...]:
    """Return dim argument as a tuple of sorted dim values."""
    dims: list[int] = []
    if dim == ():
        # Currently, `dim=()` in reductions operations means "reduce
        # over all dimensions" while in future, it will read "no
        # reduce". See https://github.com/pytorch/pytorch/issues/29137
        # When gh-29137 is resolved, this if-block must be deleted.
        dim = None
    if dim is None:
        return tuple(range(ndim))
    ndim = max(ndim, 1)
    dim_ = (dim,) if isinstance(dim, (int, torch.SymInt)) else dim
    for d in dim_:
        if d in dims:
            raise RuntimeError(f"dim={d} appears multiple times in the list of dims")
        if d >= ndim or d < -ndim:
            raise IndexError(
                f"Dimension out of range (expected to be in range of [{-ndim}, {ndim - 1}], but got {d})"
            )
        # pyrefly: ignore [bad-argument-type]
        dims.append(d % ndim)
    return tuple(sorted(dims))
````
- **EN**: This chunk defines `_canonical_dim`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_canonical_dim`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 495-526 / 第 495-526 行
````python
def _sparse_coo_flatten_indices(indices: Tensor, shape: tuple):
    # Flatted N-D indices to 1-D indices
    flat_indices = indices.new_zeros(indices.size(1))
    for d, sz in enumerate(shape):
        flat_indices.mul_(sz)
        flat_indices.add_(indices[d])
    return flat_indices


def _any(input: Tensor, dim: tuple, keepdim: bool):
    # Support torch.any with tuple dim argument.
    # Workaround of https://github.com/pytorch/pytorch/issues/56586
    r = input
    for d in reversed(dim):
        r = r.any(dim=d, keepdim=keepdim)
    return r


def _sparse_coo_where(mask: Tensor, input: Tensor, fill_value: Tensor) -> Tensor:
    """Sparse variant of torch.where. Supports sparse COO and hybrid sparse COO tensors.

    _sparse_coo_where implements the following invariant:

      _sparse_coo_where(mask, input, fill_value).to_dense(fill_value) ==
        torch.where(mask.to_dense(), input.to_dense(), torch.full(input.shape, fill_value))

    where `a == b` means `assertEqual(a, b)`, mask is boolean sparse
    tensor, and `to_dense(fill_value)` is like `to_dense()` except
    that the unspecified elements are mapped to `fill_value` rather
    than to `0`.

    Returns a sparse COO tensor with the following features:
````
- **EN**: This chunk defines `_sparse_coo_where`, which parses structured input into internal objects or validated metadata. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_sparse_coo_where`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 528-562 / 第 528-562 行
````python
    - all specified elements correspond to masked-in elements that
      have the values of the input tensor. If there exists a masked-in
      element (as specified by mask) that is not specified in the
      input, in the result tensor, the corresponding element has value
      0. In the dense part of the sparse tensor, the masked-out
      elements are replaced with fill_value.

    - all unspecified elements correspond to masked-out elements.
    """

    if input.layout != torch.sparse_coo:
        raise AssertionError(f"input.layout must be sparse_coo, got {input.layout}")
    if mask.layout != input.layout:
        raise AssertionError(f"mask.layout must match input.layout, got {mask.layout}")
    if mask.shape != input.shape:
        raise AssertionError(
            f"mask.shape must match input.shape: {mask.shape} vs {input.shape}"
        )
    if mask.dense_dim() != input.dense_dim():
        # TODO: eliminate this restriction
        raise AssertionError(
            f"mask.dense_dim() must match input.dense_dim(): "
            f"{mask.dense_dim()} vs {input.dense_dim()}"
        )

    input = input.coalesce()

    # For set operations on sparse tensor indices, we'll convert
    # multi-dimensional indices to 1-D indices for efficiency.
    input_flat_indices = _sparse_coo_flatten_indices(
        input.indices(), input.shape[: input.sparse_dim()]
    )
    mask_flat_indices = _sparse_coo_flatten_indices(
        mask.indices(), mask.shape[: mask.sparse_dim()]
    )
````
- **EN**: This chunk continues `_sparse_coo_where` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_sparse_coo_where`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 564-593 / 第 564-593 行
````python
    # the set of mask flat indices that define masked-in elements:
    if mask.dense_dim() > 0:
        mask_values = _any(
            mask.values(), tuple(range(1, input.sparse_dim() + 1)), False
        )
    else:
        mask_values = mask.values()
    maskin_flat_indices = mask_flat_indices[mask_values.nonzero()[:, 0]]

    def intersection(i1, i2):
        union, counts = torch.cat([i1, i2]).unique(return_counts=True)
        return union, torch.where(counts.gt(1))

    def minus(i1, i2):
        union, counts = torch.cat([i1, i2]).unique(return_counts=True)
        return intersection(union[torch.where(counts.eq(1))], i1)

    def _apply(a):
        obj, w = a
        return obj[w]

    # the set of input flat indices of specified and masked-in elements:
    maskin_input_flat_indices = _apply(
        intersection(maskin_flat_indices, input_flat_indices)
    )
    _, w = intersection(input_flat_indices, maskin_input_flat_indices)

    # the indices and values of masked-in elements
    where_input_indices = input.indices()[(slice(None),) + w]
    where_input_values = input.values()[w]
````
- **EN**: This chunk defines `_apply`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_apply`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 595-629 / 第 595-629 行
````python
    if mask.dense_dim() > 0:
        # apply mask to the dense part of the input values:
        _, w1 = intersection(mask_flat_indices, maskin_input_flat_indices)
        where_mask_values = mask.values()[w1]
        where_input_values = torch.where(
            where_mask_values, where_input_values, fill_value
        )

    # the set of flat indices of unspecified input and masked-in elements:
    maskin_zero_flat_indices = _apply(
        minus(maskin_flat_indices, maskin_input_flat_indices)
    )

    # the indices of masked-in zero elements
    _, w = intersection(mask_flat_indices, maskin_zero_flat_indices)
    where_zero_indices = mask.indices()[(slice(None),) + w]

    # construct result
    n = where_zero_indices.size(1)
    if n == 0:
        # the input is coalesced, hence input_flat_indices are ordered
        # and the result is guaranteed to be coalesced:
        result = torch.sparse_coo_tensor(
            where_input_indices, where_input_values, input.shape
        )
        return result._coalesced_(True)

    where_indices = torch.cat([where_input_indices, where_zero_indices], dim=1)
    where_values = torch.cat(
        [
            where_input_values,
            where_input_values.new_zeros((n,) + where_input_values.shape[1:]),
        ]
    )
    result = torch.sparse_coo_tensor(where_indices, where_values, input.shape)
````
- **EN**: This chunk continues `_apply` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_apply`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 631-659 / 第 631-659 行
````python
    # appending zero elements leads to uncoalesced sparse tensor
    return result.coalesce()


def _sparse_coo_scatter_reduction_helper(
    op,
    mask_input: Tensor,
    dims: tuple[int, ...],
    keepdim: bool,
    dtype: DType | None = None,
) -> Tensor:
    reduce = op.__name__
    valid_reductions = ["sum", "prod", "amax", "amin"]
    if reduce not in valid_reductions:
        raise ValueError(
            f"op must be one of {' '.join(valid_reductions)}, but got {reduce} instead"
        )

    output_dtype = dtype
    values, indices = mask_input._values(), mask_input._indices()
    input_dims = mask_input.dim()
    num_sparse_dims = mask_input.sparse_dim()
    reduced_sparse_dims = []
    retained_sparse_dims = []
    reduced_dense_dims = []

    # promote dtype if specified
    if values.dtype != output_dtype:
        values = values.to(output_dtype)
````
- **EN**: This chunk defines `_sparse_coo_scatter_reduction_helper`, which parses structured input into internal objects or validated metadata. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_sparse_coo_scatter_reduction_helper`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 661-688 / 第 661-688 行
````python
    if keepdim:
        output_shape = tuple(
            1 if i in dims else si for (i, si) in enumerate(mask_input.shape)
        )
    else:
        output_shape = tuple(
            si for (i, si) in enumerate(mask_input.shape) if i not in dims
        )

    for d in dims:
        if d >= input_dims:
            continue

        if d < num_sparse_dims:
            reduced_sparse_dims.append(d)
        else:
            reduced_dense_dims.append(d + 1 - num_sparse_dims)

    # Reduce dense dimensions
    if len(reduced_dense_dims) > 0:
        if reduce == "sum":
            new_values = values
            new_values = op(new_values, dim=reduced_dense_dims, keepdim=bool(keepdim))
        else:
            # FIXME: Implement reductions for dense dimensions for ops with non-zero reduction identities
            return NotImplemented
    else:
        new_values = values.clone()
````
- **EN**: This chunk continues `_sparse_coo_scatter_reduction_helper` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_sparse_coo_scatter_reduction_helper`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 690-719 / 第 690-719 行
````python
    # Reduce sparse dimensions
    if len(reduced_sparse_dims) == num_sparse_dims:
        if reduce in {"amax", "amin"} and new_values.size(0) == 0:
            # IndexError: amax(): Expected reduction dim 0 to have non-zero size.
            # sum()/prod() return the reduction identity when dim has size 0 but amax()/amin() do not
            # See https://github.com/pytorch/pytorch/issues/61901
            new_values = _reduction_identity(reduce, new_values)
        else:
            new_values = op(new_values, dim=0)
        if keepdim:
            for _ in range(num_sparse_dims):
                new_values = new_values.unsqueeze(0)
        return new_values.to(dtype=output_dtype).to_sparse()
    else:
        new_indices = indices.clone()
        if keepdim:
            # zero out reduced sparse dimensions if keepdim = True
            # ensures that the call to torch.unique folds duplicated indices together while preserving the dimension
            new_indices[reduced_sparse_dims, :] = 0
        else:
            # remove reduced sparse dimensions if keepdim = False
            if len(reduced_sparse_dims) > 0:
                retained_sparse_dims = [
                    i
                    for i in range(num_sparse_dims)
                    if i not in set(reduced_sparse_dims)
                ]
                new_indices = new_indices.index_select(
                    0, torch.tensor(retained_sparse_dims).to(mask_input.device)
                )
````
- **EN**: This chunk continues `_sparse_coo_scatter_reduction_helper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_sparse_coo_scatter_reduction_helper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 721-752 / 第 721-752 行
````python
    # Use scatter_reduce to reduce items in the new_values tensor that correspond to the same indices in new_indices
    if new_indices.numel() > 0:
        # lexsort indices and get index tensor for scatter reduction
        new_indices, inverse_indices = torch.unique(
            new_indices, return_inverse=True, dim=1
        )
        out_shape = list(new_values.shape)
        out_shape[0] = new_indices.shape[1]
        for _ in range(new_values.ndim - 1):
            inverse_indices = inverse_indices.unsqueeze(-1)
        scatter_indices = inverse_indices.expand(new_values.shape)
        # FIXME: temporary workaround for issue with bfloat16/float16 remove when acctype is implemented for scatter_reduce
        if output_dtype in {torch.bfloat16, torch.float16}:
            new_values = new_values.to(torch.float)
            out = new_values.new_empty(out_shape)
            new_values = out.scatter_reduce_(
                0, scatter_indices, new_values, reduce=reduce, include_self=False
            )
            new_values = new_values.to(dtype=output_dtype)
        else:
            out = new_values.new_empty(out_shape)
            new_values = out.scatter_reduce_(
                0, scatter_indices, new_values, reduce=reduce, include_self=False
            )

    return torch.sparse_coo_tensor(
        new_indices,
        new_values,
        output_shape,
        dtype=output_dtype,
        device=mask_input.device,
    )
````
- **EN**: This chunk continues `_sparse_coo_scatter_reduction_helper` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_sparse_coo_scatter_reduction_helper`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 755-784 / 第 755-784 行
````python
def _sparse_csr_segment_reduction_helper(
    op,
    mask_input: Tensor,
    dims: tuple[int, ...],
    keepdim: bool,
    dtype: DType | None = None,
) -> Tensor:
    # Currently, while sparse CSR is always 2D with no dense dimensions keepdim must be True
    # FIXME: when dense dimensions are implemented for CSR tensors
    if not keepdim:
        raise AssertionError(
            "reduction operations on CSR tensors with keepdim=False is unsupported"
        )
    reduce = op.__name__
    valid_reductions = ["sum", "prod", "mean", "amax", "amin"]
    if reduce not in valid_reductions:
        raise ValueError(
            f"op must be one of {' '.join(valid_reductions)}, but got {reduce} instead"
        )
    device = mask_input.device
    output_dtype = dtype
    values, crow_indices, col_indices = (
        mask_input.values(),
        mask_input.crow_indices(),
        mask_input.col_indices(),
    )

    # promote dtype if specified
    if values.dtype != output_dtype:
        values = values.to(output_dtype)
````
- **EN**: This chunk defines `_sparse_csr_segment_reduction_helper`, which parses structured input into internal objects or validated metadata. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_sparse_csr_segment_reduction_helper`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 786-821 / 第 786-821 行
````python
    if len(dims) == 0:
        return mask_input
    if len(dims) == 1:
        if dims[0] == 0:
            new_col_indices, scatter_indices = torch.unique(
                col_indices, return_inverse=True
            )
            new_nnz = new_col_indices.shape[0]
            new_crow_indices = torch.tensor([0, new_nnz])
            new_values = values.new_empty(new_col_indices.shape)
            new_values.scatter_reduce_(
                0, scatter_indices, values, reduce, include_self=False
            )
            new_shape = [1, mask_input.size(1)]
        else:
            if dims[0] != 1:
                raise AssertionError(
                    "Sparse CSR tensors are 2D and only support reduction along dim 0 or 1."
                )
            # all intervals new_crow_indices[i] - new_crow_indices[i-1] are 1
            # except for where crow_indices[i] == crow_indices[i-1] where the interval remains as 0
            new_crow_indices = torch.cat(
                (
                    crow_indices.new_zeros(1),
                    torch.cumsum(torch.diff(crow_indices) != 0, 0),
                ),
                0,
            )
            new_nnz = new_crow_indices[-1]
            new_col_indices = col_indices.new_zeros(new_nnz)  # type: ignore[call-overload]
            new_values = torch._segment_reduce(values, reduce, offsets=crow_indices)  # type: ignore[attr-defined]
            new_shape = [mask_input.size(0), 1]
    else:
        if len(dims) != 2:
            raise AssertionError(f"expected len(dims) == 2, got {len(dims)}")
        nnz = min(1, values.numel())
````
- **EN**: This chunk continues `_sparse_csr_segment_reduction_helper` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_sparse_csr_segment_reduction_helper`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 822-855 / 第 822-855 行
````python
        if nnz == 1:
            op_kwargs = {"keepdim": True, "dtype": output_dtype}
            # amax and amin do not support dtype kwarg
            if reduce in ["amax", "amin"]:
                del op_kwargs["dtype"]
            new_values = op(values, 0, **op_kwargs)
        else:
            new_values = torch.empty(0, dtype=output_dtype)
        new_col_indices = col_indices.new_zeros(nnz)
        new_crow_indices = torch.tensor([0, nnz])
        new_shape = [1, nnz]

    return torch.sparse_csr_tensor(
        new_crow_indices,
        new_col_indices,
        new_values,
        new_shape,
        dtype=output_dtype,
        device=device,
    )


def _sparse_csr_where(mask: Tensor, input: Tensor, fill_value: Tensor) -> Tensor:
    """Sparse variant of torch.where. Supports sparse CSR tensors."""
    # TODO: implement sparse CSR specific where operator for efficiency
    return _sparse_coo_where(
        mask.to_sparse_coo(), input.to_sparse_coo(), fill_value
    ).to_sparse_csr()


def _where(mask: Tensor, input: Tensor, fill_value: Tensor) -> Tensor:
    """torch.where with sparse inputs support.

    _where implements the following invariant:
````
- **EN**: This chunk defines `_where`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_where`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 857-889 / 第 857-889 行
````python
      _where(mask, input, fill_value).to_dense(fill_value) ==
        torch.where(mask.to_dense(), input.to_dense(), torch.full(input.shape, fill_value))

    where `a == b` means `assertEqual(a, b)`, mask is boolean sparse
    tensor, and `to_dense(fill_value)` is like `to_dense()` except
    that the unspecified elements are mapped to `fill_value` rather
    than to `0`.

    Returns a sparse tensor with the following features:

    - all specified elements correspond to masked-in elements that
      have the values of the input tensor. If there exists a masked-in
      element (as specified by mask) that is not specified in the
      input, in the result tensor, the corresponding element has value
      0. In the dense part of the sparse tensor, the masked-out
      elements are replaced with fill_value.

    - all unspecified elements correspond to masked-out elements.
    """
    if mask.layout == torch.strided:
        return torch.where(mask, input, fill_value)
    elif mask.layout == torch.sparse_coo:
        return _sparse_coo_where(mask, input, fill_value)
    elif mask.layout == torch.sparse_csr:
        return _sparse_csr_where(mask, input, fill_value)
    else:
        raise ValueError(
            f"_where expects strided or sparse COO or sparse CSR tensor but got {mask.layout}"
        )


def _input_mask(input: Tensor | MaskedTensor, *args, **kwargs) -> Tensor:
    """Return canonical input mask.
````
- **EN**: This chunk defines `_input_mask`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior.
- **CN**: 这一段定义了 `_input_mask`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。

### Lines 891-924 / 第 891-924 行
````python
    A canonical input mask is defined as a boolean mask tensor that
    shape and layout matches with the shape and the layout of the
    input.

    The canonical input mask is computed from the :attr:`mask` tensor
    content to meet the following criteria:

    1. The shape of the canonical input mask is the same as the shape
       of :attr:`input` tensor. If the mask tensor has a smaller shape
       than the shape of the :attr:`input`, broadcasting rules will be
       applied. Downcasting of mask is not supported.

    2. The layout of the canonical input mask is the same as the
       layout of the :attr:`input` tensor. If the mask has different
       layout, it will be converted to the expected layout.  In the
       case of sparse COO layout, the canonical input mask will be
       coalesced.

    3. The dtype of the canonical input mask is torch.bool. If the
       mask dtype is not bool then it will be converted to bool dtype
       using `.to(dtype=bool)` method call.

    4. The elements of the canonical input mask have boolean values
       copied from the content of the :attr:`mask` tensor (after
       possible broadcasting and dtype conversion transforms).  In
       general, the sparsity pattern of the sparse canonical input
       mask need not to be the same as the sparsity pattern of the
       sparse :attr:`input` tensor.

    """
    if input.layout not in {torch.strided, torch.sparse_coo, torch.sparse_csr}:
        raise ValueError(
            f"_input_mask expects strided or sparse COO or sparse CSR tensor but got {input.layout}"
        )
````
- **EN**: This chunk continues `_input_mask` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_input_mask`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 926-949 / 第 926-949 行
````python
    mask = kwargs.get("mask")

    # default mask
    if mask is None:
        raise ValueError("_input_mask requires explicit mask")

    # mask shape must match with input shape
    if mask.shape != input.shape:
        if mask.ndim > input.ndim:
            raise IndexError(
                "_input_mask expected broadcastable mask (got mask dimensionality higher than of the input)"
            )
        if mask.layout == torch.strided:
            mask = torch.broadcast_to(mask.clone(), input.shape).to(dtype=torch.bool)
        elif mask.layout == torch.sparse_coo:
            mask = torch._sparse_broadcast_to(mask, input.shape)
        else:
            if mask.layout != torch.sparse_csr:
                raise AssertionError(f"expected sparse_csr layout, got {mask.layout}")
            # Broadcasting of CSR tensors is not implemented. Working
            # around by using COO layout.
            mask = torch._sparse_broadcast_to(
                mask.to_sparse(), input.shape
            ).to_sparse_csr()
````
- **EN**: This chunk continues `_input_mask` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_input_mask`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 951-972 / 第 951-972 行
````python
    # mask layout must match with input layout
    if mask.layout != input.layout:
        if input.layout == torch.strided:
            mask = mask.to_dense()
        elif input.layout == torch.sparse_coo:
            if mask.layout == torch.strided:
                mask = mask.to_sparse(input.sparse_dim())
            else:
                mask = mask.to_sparse()
        else:
            if input.layout != torch.sparse_csr:
                raise AssertionError(f"expected sparse_csr layout, got {input.layout}")
            mask = mask.to_sparse_csr()

    # sparse mask must be coalesced
    if mask.layout == torch.sparse_coo:
        mask = mask.coalesce()

    # mask is a boolean tensor
    mask = mask.to(dtype=torch.bool)

    return mask
````
- **EN**: This chunk continues `_input_mask` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_input_mask`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 975-1010 / 第 975-1010 行
````python
def _output_mask(op, input: Tensor, *args, **kwargs) -> Tensor:
    """Return output mask of masked operation applied to given arguments."""
    if callable(op):
        is_reduction = op.__name__ in {
            "sum",
            "prod",
            "amax",
            "amin",
            "argmax",
            "argmin",
            "mean",
            "median",
            "norm",
            "var",
            "std",
            "logsumexp",
        }
        is_normalization = op.__name__ in {
            "softmax",
            "log_softmax",
            "softmin",
            "normalize",
            "cumsum",
            "cumprod",
        }
        if is_reduction:
            if op.__name__ == "norm":
                if args:
                    args = args[1:]  # lstrip ord argument
            dim = args[0] if args else kwargs.get("dim")
            outmask = _input_mask(input, *args, **kwargs)
            keepdim = kwargs.get("keepdim", False)
            dim_ = _canonical_dim(dim, input.ndim)
            return _any(outmask, dim_, bool(keepdim))
        elif is_normalization:
            return _input_mask(input, *args, **kwargs)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_output_mask`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_output_mask`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1011-1044 / 第 1011-1044 行
````python
        else:
            raise ValueError(
                f"_output_mask expected masked operation (got callable {op.__module__}.{op.__name__})"
            )
    else:
        raise ValueError(
            f"_output_mask expected masked operation (got {type(op).__name__} object)"
        )


def _combine_input_and_mask(op, input: MaskedTensor | Tensor, mask, *args) -> Tensor:
    def helper(input, mask):
        if mask is None:
            return input
        canonical_mask = _input_mask(input, mask=mask)
        if callable(op):
            fill_value = _reduction_identity(op.__name__, input, *args)
            return _where(canonical_mask, input, fill_value)
        else:
            raise ValueError(
                f"_combine_input_and_mask expected masked operation (got {type(op).__name__} object)"
            )

    class Combine(torch.autograd.Function):
        @staticmethod
        # pyrefly: ignore [bad-override]
        def forward(ctx, input, mask):
            """Return input with masked-out elements eliminated for the given operations."""
            ctx.save_for_backward(mask)

            if mask is not None:
                ctx.mark_non_differentiable(mask)

            return helper(input, mask)
````
- **EN**: It introduces or extends `Combine`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `Combine`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1046-1081 / 第 1046-1081 行
````python
        @staticmethod
        # pyrefly: ignore [bad-override]
        def backward(ctx, grad_output):
            (mask,) = ctx.saved_tensors
            grad_data = (
                grad_output.get_data() if is_masked_tensor(grad_output) else grad_output
            )
            result = as_masked_tensor(grad_data, mask)
            return result, None

    return (
        Combine.apply(input.get_data(), input.get_mask())  # type: ignore[union-attr]
        if is_masked_tensor(input)
        else helper(input, mask)
    )


@_apply_docstring_templates
def sum(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    # __doc__ is generated by _apply_docstring_templates decorator
    if dtype is None:
        # promote integer types to int64 when output dtype is not specified
        if input.layout == torch.sparse_csr:
            if input.dtype in {
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
````
- **EN**: This chunk defines `sum`, which implements a focused helper used by the surrounding module. Decorators such as `staticmethod`, `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `sum`，其作用是实现周边模块使用的关键辅助逻辑。 像 `staticmethod`、`_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1082-1113 / 第 1082-1113 行
````python
            }:
                # csr.to(dtype=torch.int64) is not implemented, so
                # using coo.to on input to ensure the promoted dtype
                input = input.to_sparse_coo().to(dtype=torch.int64).to_sparse_csr()
            else:
                dtype = input.dtype
        else:
            dtype = input.dtype
            if input.dtype in {
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
            }:
                dtype = torch.int64
    dim_ = _canonical_dim(dim, input.ndim)
    mask_input = _combine_input_and_mask(sum, input, mask)
    if mask_input.layout == torch.strided:
        return torch.sum(mask_input, dim_, bool(keepdim), dtype=dtype)
    elif mask_input.layout == torch.sparse_coo:
        return _sparse_coo_scatter_reduction_helper(
            torch.sum, mask_input, dim_, bool(keepdim), dtype
        )
    elif mask_input.layout == torch.sparse_csr:
        return torch._sparse_csr_sum(
            mask_input, dim=list(dim_), keepdim=bool(keepdim), dtype=dtype
        )
    else:
        raise ValueError(
            f"masked sum expects strided, sparse_coo or sparse_csr tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk continues `sum` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `sum`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1116-1151 / 第 1116-1151 行
````python
@_apply_docstring_templates
def prod(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    # __doc__ is generated by _apply_docstring_templates decorator
    if dtype is None:
        # promote integer types to int64 when output dtype is not specified
        if input.layout == torch.sparse_csr:
            if input.dtype in {
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
            }:
                # csr.to(dtype=torch.int64) is not implemented, so
                # using coo.to on input to ensure the promoted dtype
                input = input.to_sparse_coo().to(dtype=torch.int64).to_sparse_csr()
            else:
                dtype = input.dtype
        else:
            dtype = input.dtype
            if input.dtype in {
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
            }:
                dtype = torch.int64
    dim_ = _canonical_dim(dim, input.ndim)
````
- **EN**: This chunk defines `prod`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `prod`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1152-1187 / 第 1152-1187 行
````python
    mask_input = _combine_input_and_mask(prod, input, mask)
    if mask_input.layout == torch.strided:
        # Workaround https://github.com/pytorch/pytorch/issues/56586
        result = mask_input
        result = result.to(dtype=dtype)
        for d in reversed(dim_):
            result = result.prod(dim=d, keepdim=bool(keepdim))
        return result
    elif mask_input.layout == torch.sparse_coo:
        if mask is None:
            # See comment in the sparse_csr branch, the same issue arises for sparse_coo tensors
            raise ValueError(
                "masked prod expects explicit mask for sparse_coo tensor input"
            )
        return _sparse_coo_scatter_reduction_helper(
            torch.prod, mask_input, dim_, bool(keepdim), dtype
        )
    elif mask_input.layout == torch.sparse_csr:
        if mask is None:
            # mask is None corresponds to all-True mask. The
            # unspecified elements in the CSR tensor correspond to
            # zero values. Hence, the prod reduction result is
            # automatically zero unless all elements are specified.
            # A semi-optimal way to take this into account is to use:
            #
            #   masked_prod(csr, ..., mask=None) == torch._sparse_csr_prod(csr, ...) * all(csr.nonzero(), ...)
            #
            # but that requires implementing `all` and `nonzero`
            # support for sparse csr tensors.
            raise ValueError(
                "masked prod expects explicit mask for sparse_csr tensor input"
            )
        return torch._sparse_csr_prod(
            mask_input, dim=list(dim_), keepdim=bool(keepdim), dtype=dtype
        )
    else:
````
- **EN**: This chunk continues `prod` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `prod`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1188-1210 / 第 1188-1210 行
````python
        raise ValueError(
            f"masked prod expects strided, sparse_coo or sparse_csr tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def cumsum(
    input: Tensor,
    dim: int,
    *,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    mask_input = _combine_input_and_mask(sum, input, mask)
    if mask_input.layout == torch.strided:
        return torch.cumsum(mask_input, dim_, dtype=dtype).to(dtype=dtype)
    else:
        raise ValueError(
            f"masked cumsum expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `cumsum`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `cumsum`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1213-1247 / 第 1213-1247 行
````python
@_apply_docstring_templates
def cumprod(
    input: Tensor,
    dim: int,
    *,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    mask_input = _combine_input_and_mask(prod, input, mask)
    if mask_input.layout == torch.strided:
        return torch.cumprod(mask_input, dim_, dtype=dtype).to(dtype=dtype)
    else:
        raise ValueError(
            f"masked cumprod expects strided tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def amax(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}

{reduction_descr}

{reduction_identity_dtype}
````
- **EN**: This chunk defines `amax`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `amax`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1249-1280 / 第 1249-1280 行
````python
{reduction_args}

{reduction_example}"""
    if dtype is None:
        dtype = input.dtype

    mask_input = _combine_input_and_mask(amax, input, mask)
    dim_ = _canonical_dim(dim, mask_input.ndim)
    if mask_input.layout == torch.strided:
        return torch.amax(mask_input, dim_, bool(keepdim)).to(dtype=dtype)
    elif mask_input.layout == torch.sparse_coo:
        if mask is None:
            # See comment in the sparse_csr branch of prod, a similar issue arises here
            # where unspecified elements along a dimension may need to be reduced with the result
            raise ValueError(
                "masked amax expects explicit mask for sparse_coo tensor input"
            )
        return _sparse_coo_scatter_reduction_helper(
            torch.amax, mask_input, dim_, bool(keepdim), dtype
        )
    elif mask_input.layout == torch.sparse_csr:
        if mask is None:
            raise ValueError(
                "masked amax expects explicit mask for sparse_csr tensor input"
            )
        return _sparse_csr_segment_reduction_helper(
            torch.amax, mask_input, dim_, bool(keepdim), dtype
        )
    else:
        raise ValueError(
            f"masked amax expects strided, sparse_coo or sparse_csr tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1283-1318 / 第 1283-1318 行
````python
@_apply_docstring_templates
def amin(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}

{reduction_descr}

{reduction_identity_dtype}

{reduction_args}

{reduction_example}"""
    if dtype is None:
        dtype = input.dtype

    mask_input = _combine_input_and_mask(amin, input, mask)
    dim_ = _canonical_dim(dim, mask_input.ndim)
    if mask_input.layout == torch.strided:
        return torch.amin(mask_input, dim_, bool(keepdim)).to(dtype=dtype)
    elif mask_input.layout == torch.sparse_coo:
        if mask is None:
            # See comment in the sparse_csr branch of prod, a similar issue arises here
            # where unspecified elements along a dimension may need to be reduced with the result
            raise ValueError(
                "masked amax expects explicit mask for sparse_coo tensor input"
            )
        return _sparse_coo_scatter_reduction_helper(
            torch.amin, mask_input, dim_, bool(keepdim), dtype
        )
````
- **EN**: This chunk defines `amin`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `amin`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1319-1354 / 第 1319-1354 行
````python
    elif mask_input.layout == torch.sparse_csr:
        if mask is None:
            raise ValueError(
                "masked amin expects explicit mask for sparse_csr tensor input"
            )
        return _sparse_csr_segment_reduction_helper(
            torch.amin, mask_input, dim_, bool(keepdim), dtype
        )
    else:
        raise ValueError(
            f"masked amin expects strided, sparse_coo or sparse_csr tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def argmax(
    input: Tensor | MaskedTensor,
    dim: int | None = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}
{reduction_descr}
{reduction_identity_dtype}
{reduction_args}
{reduction_example}"""
    if dtype is None:
        dtype = input.dtype
    mask_input = _combine_input_and_mask(argmax, input, mask)
    if mask_input.layout == torch.strided:
        return torch.argmax(mask_input, dim, bool(keepdim)).to(dtype=dtype)
    else:
        raise ValueError(
````
- **EN**: This chunk defines `argmax`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `argmax`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1355-1382 / 第 1355-1382 行
````python
            f"masked argmax expects strided tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def argmin(
    input: Tensor | MaskedTensor,
    dim: int | None = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}
{reduction_descr}
{reduction_identity_dtype}
{reduction_args}
{reduction_example}"""
    if dtype is None:
        dtype = input.dtype
    mask_input = _combine_input_and_mask(argmin, input, mask)
    if mask_input.layout == torch.strided:
        return torch.argmin(mask_input, dim, bool(keepdim)).to(dtype=dtype)
    else:
        raise ValueError(
            f"masked argmin expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `argmin`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `argmin`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1385-1412 / 第 1385-1412 行
````python
@_apply_docstring_templates
def mean(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}

{reduction_descr}

By definition, the identity value of a mean operation is the mean
value of the tensor. If all elements of the input tensor along given
dimension(s) :attr:`dim` are masked-out, the identity value of the
mean is undefined.  Due to this ambiguity, the elements of output
tensor with strided layout, that correspond to fully masked-out
elements, have ``nan`` values.

{reduction_args}

{reduction_example}"""
    dtype_source = "Optional"
    if dtype is None:
        dtype = input.dtype
        dtype_source = "Input"
````
- **EN**: This chunk defines `mean`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `mean`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1414-1449 / 第 1414-1449 行
````python
    if not (dtype.is_floating_point or dtype.is_complex):
        raise ValueError(
            f"mean(): Could not infer output dtype. {dtype_source} dtype must be either "
            f"a floating point or complex dtype. Got: {dtype}"
        )
    if input.layout == torch.strided:
        if mask is None:
            # TODO: compute count analytically
            # pyrefly: ignore [no-matching-overload]
            count = sum(
                torch.ones(input.shape, dtype=torch.int64, device=input.device),
                dim,
                keepdim=keepdim,
            )
            # pyrefly: ignore [no-matching-overload]
            total = sum(input, dim, keepdim=keepdim, dtype=dtype)
        else:
            inmask = _input_mask(input, mask=mask)
            count = inmask.sum(dim=dim, keepdim=bool(keepdim))
            # pyrefly: ignore [no-matching-overload]
            total = sum(input, dim, keepdim=keepdim, dtype=dtype, mask=inmask)
        return total / count
    elif input.layout == torch.sparse_csr:
        mask_input = _combine_input_and_mask(mean, input, mask)
        dim_ = _canonical_dim(dim, mask_input.ndim)
        if mask is None:
            raise ValueError(
                "masked mean expects explicit mask for sparse_csr tensor input"
            )
        return _sparse_csr_segment_reduction_helper(
            torch.mean, mask_input, dim_, bool(keepdim), dtype
        )
    else:
        raise ValueError(
            f"masked mean expects strided or sparse_csr tensor (got {input.layout} tensor)"
        )
````
- **EN**: This chunk continues `mean` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `mean`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1452-1487 / 第 1452-1487 行
````python
@_apply_docstring_templates
def median(
    input: Tensor | MaskedTensor,
    dim: int = -1,
    *,
    keepdim: bool = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}
{reduction_descr}
By definition, the identity value of a median operation is the median
value of the tensor. If all elements of the input tensor along given
dimension(s) :attr:`dim` are masked-out, the identity value of the
median is undefined.  Due to this ambiguity, the elements of output
tensor with strided layout, that correspond to fully masked-out
elements, have ``nan`` values.
{reduction_args}
{reduction_example}"""
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    is_float = torch.is_floating_point(input)
    if not is_float:
        input = input.to(dtype=torch.float)
    mask_input = _combine_input_and_mask(median, input, mask)
    if mask_input.layout == torch.strided:
        output = torch.nanmedian(mask_input, dim_, keepdim).values
        if is_float:
            return output
        elif not is_float and not torch.isnan(output).any():
            return output.to(dtype=dtype)
        else:
            raise ValueError(
                "masked median expects no fully masked out rows if dtype is not floating point"
````
- **EN**: This chunk defines `median`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `median`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1488-1513 / 第 1488-1513 行
````python
            )
    else:
        raise ValueError(
            f"masked median expects strided tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def logsumexp(
    input: Tensor,
    dim: DimOrDims = None,
    *,
    keepdim: bool = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)
    mask_input = _combine_input_and_mask(logsumexp, input, mask)
    if mask_input.layout == torch.strided:
        return torch.logsumexp(mask_input, dim_, keepdim=keepdim).to(dtype=dtype)
    else:
        raise ValueError(
            f"masked logsumexp expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `logsumexp`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `logsumexp`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1516-1539 / 第 1516-1539 行
````python
# Cannot use _apply_docstring_templates as it is only set up for reductions and normalizations
def logaddexp(
    input: Tensor | MaskedTensor,
    other: Tensor | MaskedTensor,
    *,
    dtype: DType | None = None,
    input_mask: Tensor | None = None,
    other_mask: Tensor | None = None,
) -> Tensor:
    """logaddexp(input, other, *, dtype=None, input_mask=None, other_mask=None) -> Tensor

    Returns logaddexp of all the elements in the :attr:`input` and the :attr:`other`
    tensor. The :attr:`input` elements are masked out according to the boolean tensor
    :attr:`input_mask` and the attr:`other` elements are masked out according to the boolean tensor
    :attr:`other_mask`.

    The shapes of a mask tensor and the tensor to be masked
    don't need to match, but they must be :ref:`broadcastable
    <broadcasting-semantics>` and the dimensionality of the mask
    tensor must not be greater than of the tensor to be masked.

    Args:
        input (Tensor): the input tensor
        other (Tensor): the second input tensor
````
- **EN**: This chunk defines `logaddexp`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `logaddexp`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1541-1575 / 第 1541-1575 行
````python
    Keyword args:
        dtype (:class:`torch.dtype`, optional): the desired data type
          of returned tensor.  If specified, the output tensor is
          casted to :attr:`dtype` after the operation is
          performed. Default: None.
        input_mask (:class:`torch.Tensor`, optional): the boolean tensor
          containing the binary mask of validity of :attr:`input` tensor elements.
          Default: None that is equivalent to ``torch.ones(input.shape, dtype=torch.bool)``.
        other_mask (:class:`torch.Tensor`, optional): the boolean tensor
          containing the binary mask of validity of :attr:`other` tensor elements.
          Default: None that is equivalent to ``torch.ones(other.shape, dtype=torch.bool)``.

    Example::

        >>> input = torch.tensor([-100.0, -200, -300])
        >>> input
        tensor([-100., -200., -300.])
        >>> other = torch.tensor([-1.0, -2, -3])
        >>> other
        tensor([-1., -2., -3.])
        >>> mask = torch.tensor([True, False, True])
        >>> mask
        tensor([ True, False,  True])
        >>> torch.masked._ops.logaddexp(input, other, input_mask=mask, other_mask=mask)
        tensor([-1., -inf, -3.])"""
    if dtype is None:
        dtype = input.dtype
    if input.layout == torch.strided and other.layout == torch.strided:
        mask_input = _combine_input_and_mask(logaddexp, input, input_mask)
        mask_other = _combine_input_and_mask(logaddexp, other, other_mask)
        return torch.logaddexp(mask_input, mask_other).to(dtype=dtype)
    else:
        raise ValueError(
            f"masked logaddexp expects strided tensors (got {input.layout} tensor for input, {other.layout} for other)"
        )
````
- **EN**: This chunk continues `logaddexp` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `logaddexp`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1578-1611 / 第 1578-1611 行
````python
@_apply_docstring_templates
def norm(
    input: Tensor | MaskedTensor,
    ord: float | None = 2.0,
    dim: DimOrDims = None,
    *,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}

{reduction_descr}

The identity value of norm operation, which is used to start the
reduction, is ``{identity_float32}``, except for ``ord=-inf`` it is
``{identity_ord_ninf}``.

{reduction_args}

{reduction_example}"""
    if dtype is None:
        dtype = input.dtype
    mask_input = _combine_input_and_mask(norm, input, mask, ord)
    if mask_input.layout == torch.strided:
        dim_ = _canonical_dim(dim, input.ndim)
        return torch.linalg.vector_norm(
            mask_input, ord, dim_, bool(keepdim), dtype=dtype
        )
    else:
        raise ValueError(
            f"masked norm expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `norm`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `norm`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1614-1649 / 第 1614-1649 行
````python
def _std_var(
    input: Tensor | MaskedTensor,
    dim: DimOrDims,
    unbiased: bool | None,
    *,
    correction_opt: int | float | None,
    keepdim: bool | None,
    dtype: DType | None,
    mask: Tensor | None,
    take_sqrt: bool | None,
) -> Tensor:
    if unbiased is not None and correction_opt is not None:
        raise AssertionError("Only one of unbiased and correction may be given")
    correction = 1.0
    if unbiased is not None:
        correction = 1.0 if unbiased else 0.0
    if correction_opt is not None:
        correction = sym_float(correction_opt)

    if dtype is None:
        dtype = input.dtype
        if not (dtype.is_floating_point or dtype.is_complex):
            dtype = torch.float32
    compute_dtype = dtype
    if not (compute_dtype.is_floating_point or compute_dtype.is_complex):
        compute_dtype = torch.float32
    if input.layout == torch.strided:
        if mask is None:
            # TODO: compute count analytically
            # pyrefly: ignore [no-matching-overload]
            count = sum(
                torch.ones(input.shape, dtype=torch.int64, device=input.device),
                dim,
                keepdim=True,
            )
            # pyrefly: ignore [no-matching-overload]
````
- **EN**: This chunk defines `_std_var`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `_std_var`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1650-1685 / 第 1650-1685 行
````python
            sample_total = sum(input, dim, keepdim=True, dtype=dtype)
        else:
            inmask = _input_mask(input, mask=mask)
            count = inmask.sum(dim=dim, keepdim=True)
            # pyrefly: ignore [no-matching-overload]
            sample_total = sum(input, dim, keepdim=True, dtype=dtype, mask=inmask)
        # TODO: replace torch.subtract/divide/square/maximum with
        # masked subtract/divide/square/maximum when these will be
        # available.
        sample_mean = torch.divide(sample_total, count)
        x = torch.subtract(input, sample_mean)
        if mask is None:
            # pyrefly: ignore [no-matching-overload]
            total = sum(x * x.conj(), dim, keepdim=keepdim, dtype=compute_dtype)
        else:
            # pyrefly: ignore [no-matching-overload]
            total = sum(
                x * x.conj(),
                dim,
                keepdim=keepdim,
                dtype=compute_dtype,
                mask=inmask,  # type: ignore[possibly-undefined]
            )
        if not keepdim:
            count = count.reshape(total.shape)
        if correction != 0:
            real_dtype = (
                corresponding_real_dtype(compute_dtype)
                if compute_dtype.is_complex
                else compute_dtype
            )
            count = count.to(real_dtype)
            count = torch.subtract(count, correction)
            count = torch.maximum(count, count.new_zeros([]))
        output = torch.divide(total, count).to(dtype=dtype)
        if take_sqrt:
````
- **EN**: This chunk continues `_std_var` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_std_var`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1686-1721 / 第 1686-1721 行
````python
            output = torch.sqrt(output)
        return output
    else:
        raise ValueError(
            f"masked std/var expects strided tensor (got {input.layout} tensor)"
        )


@_apply_docstring_templates
def var(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    unbiased: bool | None = None,
    *,
    correction: int | float | None = None,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}
{reduction_descr}
The identity value of sample variance operation is undefined. The
elements of output tensor with strided layout, that correspond to
fully masked-out elements, have ``nan`` values.
{reduction_args}
{reduction_example}"""
    return _std_var(
        input=input,
        dim=dim,
        unbiased=unbiased,
        correction_opt=correction,
        keepdim=keepdim,
        dtype=dtype,
        mask=mask,
        take_sqrt=False,
````
- **EN**: This chunk defines `var`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `var`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1722-1753 / 第 1722-1753 行
````python
    )


@_apply_docstring_templates
def std(
    input: Tensor | MaskedTensor,
    dim: DimOrDims = None,
    unbiased: bool | None = None,
    *,
    correction: int | None = None,
    keepdim: bool | None = False,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    """\
{reduction_signature}
{reduction_descr}
The identity value of sample standard deviation operation is undefined. The
elements of output tensor with strided layout, that correspond to
fully masked-out elements, have ``nan`` values.
{reduction_args}
{reduction_example}"""
    return _std_var(
        input=input,
        dim=dim,
        unbiased=unbiased,
        correction_opt=correction,
        keepdim=keepdim,
        dtype=dtype,
        mask=mask,
        take_sqrt=True,
    )
````
- **EN**: This chunk defines `std`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `std`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1756-1791 / 第 1756-1791 行
````python
@_apply_docstring_templates
def softmax(
    input: Tensor | MaskedTensor,
    dim: int,
    *,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    mask_input = _combine_input_and_mask(amax, input, mask)
    if mask_input.layout == torch.strided:
        return torch.nn.functional.softmax(mask_input, dim_, dtype=dtype)
    else:
        raise ValueError(
            f"masked softmax expects strided tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def log_softmax(
    input: Tensor | MaskedTensor,
    dim: int,
    *,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    mask_input = _combine_input_and_mask(amax, input, mask)
    if mask_input.layout == torch.strided:
        return torch.nn.functional.log_softmax(mask_input, dim_, dtype=dtype)
    else:
        raise ValueError(
````
- **EN**: This chunk defines `log_softmax`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `log_softmax`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1792-1813 / 第 1792-1813 行
````python
            f"masked log_softmax expects strided tensor (got {mask_input.layout} tensor)"
        )


@_apply_docstring_templates
def softmin(
    input: Tensor | MaskedTensor,
    dim: int,
    *,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    dim_ = _canonical_dim(dim, input.ndim)[0]
    mask_input = _combine_input_and_mask(amin, input, mask)
    if mask_input.layout == torch.strided:
        return torch.nn.functional.softmin(mask_input, dim_, dtype=dtype)
    else:
        raise ValueError(
            f"masked softmin expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `softmin`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `softmin`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1816-1839 / 第 1816-1839 行
````python
@_apply_docstring_templates
def normalize(
    input: Tensor | MaskedTensor,
    ord: float,
    dim: int,
    *,
    eps: float = 1e-12,
    dtype: DType | None = None,
    mask: Tensor | None = None,
) -> Tensor:
    if dtype is None:
        dtype = input.dtype
    # TODO: eliminate mask_input as unnecessary when using masked divide.
    mask_input = _combine_input_and_mask(sum, input, mask)
    if mask_input.layout == torch.strided:
        nrm_ = norm(input, ord, dim, keepdim=True, dtype=dtype, mask=mask)
        # TODO: replace torch.maximum with masked maximum when available.
        denom = torch.maximum(nrm_, nrm_.new_full([], eps))
        # TODO: replace torch.divide with masked divide when available.
        return torch.divide(mask_input, denom)
    else:
        raise ValueError(
            f"masked normalize expects strided tensor (got {mask_input.layout} tensor)"
        )
````
- **EN**: This chunk defines `normalize`, which implements a focused helper used by the surrounding module. Decorators such as `_apply_docstring_templates` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `normalize`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_apply_docstring_templates` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Masked operations**
  - EN: Represents operations that carry masks alongside data and preserve masked semantics.
  - CN: 表示同时携带 mask 与数据的运算，并保持 masked 语义。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_T**
  - EN: `_T` is one of the main symbols declared or implemented in this file.
  - CN: `_T` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
- **Vectorization**
  - EN: The code exposes SIMD or packed-value helpers for CPU-side performance.
  - CN: 代码暴露 SIMD 或打包数值辅助逻辑，以提升 CPU 侧性能。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._prims_common`, `torch.masked`, `torch.masked.maskedtensor.core`, `torch.masked.maskedtensor.creation`, `torch.types`
- **Standard library / 标准库**: `warnings`, `collections.abc`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_T`, `_P`, `_apply_docstring_templates`, `_generate_docstring`, `_reduction_identity`, `_canonical_dim`, `_sparse_coo_flatten_indices`, `_any`, `_sparse_coo_where`
