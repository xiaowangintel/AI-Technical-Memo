# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_refs/nn/functional/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-26 / 第 1-26 行
````python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import math
from collections.abc import Callable
from functools import wraps
from typing import Concatenate, Optional, TypeVar, Union
from typing_extensions import ParamSpec

import torch
import torch._prims as prims
import torch._prims_common as utils
import torch._refs as refs
from torch._decomp import register_decomposition
from torch._prims_common import (
    ELEMENTWISE_TYPE_PROMOTION_KIND,
    NumberType,
    ShapeType,
    TensorLike,
    TensorLikeType,
)
from torch._prims_common.wrappers import (
    elementwise_type_promotion_wrapper,
    elementwise_unary_scalar_wrapper,
    out_wrapper,
)
from torch._refs import _make_inplace
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._prims, torch._prims_common, ...; standard-library helpers such as math, collections.abc, functools, ...; other helper packages such as typing_extensions.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._prims、torch._prims_common、...；标准库辅助模块，如 math、collections.abc、functools、...；其他辅助包，如 typing_extensions。

### Lines 29-56 / 第 29-56 行
````python
__all__ = [
    "alpha_dropout",
    "celu",
    "celu_",
    "channel_shuffle",
    "dropout",
    "elu",
    "elu_",
    "gelu",
    "glu",
    "group_norm",
    "hardshrink",
    "hardtanh",
    "hinge_embedding_loss",
    "huber_loss",
    "l1_loss",
    "layer_norm",
    "leaky_relu",
    "log_softmax",
    "margin_ranking_loss",
    "mish",
    "mish_",
    "mse_loss",
    "nll_loss",
    "pairwise_distance",
    "pdist",
    "poisson_nll_loss",
    "prelu",
````
- **EN**: `__all__` defines the public symbols that this module chooses to export.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。

### Lines 57-77 / 第 57-77 行
````python
    "relu",
    "relu6",
    "selu",
    "selu_",
    "smooth_l1_loss",
    "softmax",
    "softmin",
    "softplus",
    "softshrink",
    "tanhshrink",
    "threshold",
    "threshold_",
    "triplet_margin_loss",
]

_T = TypeVar("_T")
_P = ParamSpec("_P")

Tensor = torch.Tensor
aten = torch._ops.ops.aten
DispatchKey = torch._C.DispatchKey  # type: ignore[attr-defined]
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。

### Lines 80-102 / 第 80-102 行
````python
def _dropout_helper(
    self: TensorLikeType,
    val: float,
) -> TensorLikeType:
    """
    Helper function for all dropout-type operators. During training,
    some of the elements of the input tensor are randomly masked.

    Returns the masked tensor of the boolean values.

    """

    return (
        refs._uniform_helper(
            self.shape,
            low=0.0,
            high=1.0,
            dtype=torch.float32,
            device=self.device,
            stride=self.stride(),
        )
        < val
    )
````
- **EN**: This chunk defines `_dropout_helper`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_dropout_helper`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 105-126 / 第 105-126 行
````python
@register_decomposition(aten.alpha_dropout)
def alpha_dropout(
    self: TensorLikeType, p: float = 0.5, training: bool = False, inplace: bool = False
) -> TensorLikeType:
    if inplace:
        raise NotImplementedError

    if not training:
        return self

    torch._check(
        p <= 1 and p >= 0,
        lambda: f"dropout probability has to be between 0 and 1, but got, {p}",
    )

    if p == 1:
        return torch.zeros_like(self)

    if p == 0:
        return self

    dropout_mask = _dropout_helper(self, 1 - p)
````
- **EN**: This chunk defines `alpha_dropout`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `alpha_dropout`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 128-152 / 第 128-152 行
````python
    # From paper: Self-Normalizing Neural Networks (https://arxiv.org/pdf/1706.02515.pdf)
    # alpha = - SELU.alpha * SELU.scale, here
    # SELU.alpha = 1.6732632423543772848170429916717 and
    # SELU.scale = 1.0507009873554804934193349852946
    alpha = -1.7580993408473766

    a = 1.0 / math.sqrt((alpha * alpha * p + 1) * (1 - p))
    b = torch.logical_not(dropout_mask)
    b = b * (alpha * a) + alpha * a * p
    dropout_mask = a * dropout_mask

    return self * dropout_mask + b


def _inplace_wrapper(fn: Callable[_P, _T]) -> Callable[_P, _T]:
    """
    Given a nn.functional non-linearity, implements its `inplace: bool` argument
    """

    # nb. We use the name of the first argument used in the unary references
    @wraps(fn)
    def _fn(*args: _P.args, **kwargs: _P.kwargs) -> _T:
        a = args[0]
        if "inplace" not in kwargs:
            kwargs["inplace"] = False
````
- **EN**: This chunk defines `_fn`, which implements a focused helper used by the surrounding module. Decorators such as `wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_fn`，其作用是实现周边模块使用的关键辅助逻辑。 像 `wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 154-181 / 第 154-181 行
````python
        if kwargs["inplace"]:
            torch._check(
                "out" not in kwargs,
                lambda: "Cannot set inplace=True and pass out= at the same time",
            )
            kwargs["inplace"] = False
            kwargs["out"] = a
            return fn(*args, **kwargs)
        else:
            return fn(*args, **kwargs)

    return _fn


# celu is implemented specially because it has an alpha argument
# celu is very similar to elu
@register_decomposition(aten.celu)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def celu(
    a: TensorLikeType, alpha: NumberType | None = None, inplace: bool = False
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.celu
````
- **EN**: This chunk defines `celu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `celu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 182-206 / 第 182-206 行
````python
    """

    if inplace:
        raise NotImplementedError

    rhs: TensorLikeType
    if alpha is not None:
        python_type = utils.dtype_to_type(a.dtype)
        if not utils.is_weakly_lesser_type(type(alpha), python_type):
            msg = f"alpha argument of type {type(alpha)} cannot be safely cast to type {python_type}!"
            raise ValueError(msg)
        rhs = alpha * torch.expm1(torch.true_divide(a, alpha))  # type: ignore[arg-type]
    else:
        rhs = torch.expm1(a)

    return torch.where(a > 0, a, rhs)


@_inplace_wrapper
@out_wrapper()
def dropout(
    a: TensorLikeType, p: float = 0.5, training: bool = True, inplace: bool = False
) -> TensorLikeType:
    if inplace:
        raise NotImplementedError
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `dropout`, which implements a focused helper used by the surrounding module. Decorators such as `_inplace_wrapper`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `dropout`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_inplace_wrapper`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 208-225 / 第 208-225 行
````python
    if not training:
        return a

    torch._check(
        p <= 1 and p >= 0,
        lambda: f"dropout probability has to be between 0 and 1, but got, {p}",
    )

    if p == 1:
        return torch.zeros_like(a)

    if p == 0:
        return a

    scale = 1 / (1 - p)
    dropout_mask = _dropout_helper(a, 1 - p)

    return a * dropout_mask * scale
````
- **EN**: This chunk continues `dropout` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `dropout`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 228-246 / 第 228-246 行
````python
@register_decomposition(aten.elu)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def elu(
    a: TensorLikeType,
    alpha: NumberType = 1.0,
    scale: NumberType = 1.0,
    input_scale: NumberType = 1.0,
    inplace: bool = False,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.elu
    """
    if inplace:
        raise NotImplementedError
````
- **EN**: This chunk defines `elu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `elu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 248-275 / 第 248-275 行
````python
    # nb. This should be factored out into a can_cast aux function
    python_type = utils.dtype_to_type(a.dtype)
    torch._check(
        utils.is_weakly_lesser_type(type(input_scale), python_type),
        lambda: f"input_scale argument of type {type(input_scale)} cannot be safely cast to type {python_type}!",
    )
    torch._check(
        utils.is_weakly_lesser_type(type(scale), python_type),
        lambda: f"scale argument of type {type(scale)} cannot be safely cast to type {python_type}!",
    )
    torch._check(
        utils.is_weakly_lesser_type(type(alpha), python_type),
        lambda: f"alpha argument of type {type(alpha)} cannot be safely cast to type {python_type}!",
    )

    return torch.where(a > 0, scale * a, (alpha * scale) * torch.expm1(a * input_scale))


@register_decomposition(aten.relu)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def relu(a: TensorLikeType, inplace: bool = False) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.relu
````
- **EN**: This chunk defines `relu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `relu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 276-303 / 第 276-303 行
````python
    """

    if inplace:
        raise NotImplementedError

    return torch.where(torch.le(a, 0), 0, a)


@register_decomposition(aten.channel_shuffle)
@out_wrapper()
def channel_shuffle(input: TensorLikeType, groups: int) -> TensorLikeType:
    """
    Reference implementation of :func:`torch.nn.functional.channel_shuffle`.
    """
    from torch._meta_registrations import device_hint

    torch._check(
        input.dim() > 2,
        lambda: f"channel_shuffle expects input with > 2 dims, but got input with sizes {list(input.size())}",
    )
    c = input.shape[1]
    torch._check(
        groups > 0,
        lambda: f"Number of groups to divide channels in must be positive. Value of groups:{groups}",
    )
    torch._check(
        (c % groups) == 0,
        lambda: f"Number of channels must be divisible by groups. Got {c} channels and {groups} groups.",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._meta_registrations. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `channel_shuffle`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._meta_registrations。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `channel_shuffle`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 304-331 / 第 304-331 行
````python
    )
    n = input.shape[0]
    cg = c // groups
    dhw = input.shape[2:]

    if input.numel() == 0 or (
        device_hint(input) == "cuda" and (groups == 1 or groups == c)
    ):
        return input.view(input.shape)

    return (
        input.reshape(n, groups, cg, *dhw)
        .transpose(1, 2)
        .reshape(input.shape)
        .contiguous()
    )


def group_norm(
    input: Tensor,
    num_groups: int,
    weight: Tensor | None = None,
    bias: Tensor | None = None,
    eps: float = 1e-5,
) -> Tensor:
    """
    Reference implementation of :func:`torch.nn.functional.group_norm`.
    """
````
- **EN**: This chunk defines `group_norm`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `group_norm`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 332-359 / 第 332-359 行
````python
    torch._check(
        input.ndim >= 2,
        lambda: f"Expected at least 2 dimensions for input tensor but received {input.ndim}",
    )

    batch_size = input.shape[0]
    num_channels = input.shape[1]
    torch._check(
        num_channels % num_groups == 0,
        lambda: "Expected number of channels in input to be divisible by num_groups, "
        + f"but got input of shape {input.shape} and num_groups = {num_groups}",
    )

    # input shape is (N, C, *), so we flatten all inner dimensions except (N, C)
    flattened_inner_size = 1
    for dim_length in input.shape[2:]:
        flattened_inner_size *= dim_length

    return torch.native_group_norm(
        input,
        weight,
        bias,
        batch_size,
        num_channels,
        flattened_inner_size,
        num_groups,
        eps,
    )[0]
````
- **EN**: This chunk continues `group_norm` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `group_norm`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 362-387 / 第 362-387 行
````python
def layer_norm(
    input: Tensor,
    normalized_shape: ShapeType,
    weight: Tensor | None = None,
    bias: Tensor | None = None,
    eps: float = 1e-5,
) -> Tensor:
    """
    Reference implementation of :func:`torch.nn.functional.layer_norm`.
    """
    return torch.native_layer_norm(input, normalized_shape, weight, bias, eps)[0]


@register_decomposition(aten.leaky_relu)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def leaky_relu(
    a: TensorLikeType, negative_slope: float = 0.01, inplace: bool = False
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.leaky_relu
    """
````
- **EN**: This chunk defines `leaky_relu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `leaky_relu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 389-413 / 第 389-413 行
````python
    if inplace:
        raise NotImplementedError

    python_type = utils.dtype_to_type(a.dtype)
    if not utils.is_weakly_lesser_type(type(negative_slope), python_type):
        msg = f"negative_slope argument of type {type(negative_slope)} cannot be safely cast to type {python_type}!"
        raise ValueError(msg)
    return torch.where(torch.gt(a, 0), a, torch.mul(a, negative_slope))


@register_decomposition(aten.mish)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def mish(a: TensorLikeType, inplace: bool = False) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.mish
    """

    if inplace:
        raise NotImplementedError
    return a * torch.tanh(torch.nn.functional.softplus(a))
````
- **EN**: This chunk defines `mish`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mish`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 416-435 / 第 416-435 行
````python
@register_decomposition(aten.selu)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def selu(a: TensorLikeType, inplace: bool = False) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.selu
    """
    if inplace:
        raise NotImplementedError

    alpha = 1.6732632423543772848170429916717
    scale = 1.0507009873554804934193349852946

    rhs = alpha * torch.expm1(a)

    return scale * torch.where(a > 0, a, rhs)
````
- **EN**: This chunk defines `selu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `selu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 438-465 / 第 438-465 行
````python
# Forwarding alias: the functional variant doesn't support the out kwarg
# CompositeImplicitAutograd - don't register decomp
def softmax(
    a: TensorLikeType,
    dim: int | None = None,
    _stacklevel: int = 3,  # for compat when using TorchRefsMode(strict=True)
    dtype: torch.dtype | None = None,
) -> TensorLikeType:
    # The error is for compat with regular PyTorch, which has this behavior
    # deprecated.  For PrimTorch, it's fine to drop support for deprecated
    # behavior because it requires explicit opt in.  This error is to inform
    # users how to update their calls.
    torch._check(dim is not None, lambda: "implicit dim not supported, use dim=X")
    return torch.softmax(a=a, dim=dim, dtype=dtype)  # type: ignore[call-overload]


# CompositeImplicitAutograd - don't register decomp
def softmin(
    a: TensorLikeType,
    dim: int | None = None,
    _stacklevel: int = 3,  # for compat when using TorchRefsMode(strict=True)
    dtype: torch.dtype | None = None,
) -> TensorLikeType:
    # The error is for compat with regular PyTorch, which has this behavior
    # deprecated.  For PrimTorch, it's fine to drop support for deprecated
    # behavior because it requires explicit opt in.  This error is to inform
    # users how to update their calls.
    torch._check(dim is not None, lambda: "implicit dim not supported, use dim=X")
````
- **EN**: This chunk defines `softmin`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `softmin`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 466-488 / 第 466-488 行
````python
    return torch.softmax(a=-a, dim=dim, dtype=dtype)  # type: ignore[call-overload]


# softplus is implemented specially because it has beta and threshold arguments
@register_decomposition(aten.softplus)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def softplus(
    a: TensorLikeType,
    beta: NumberType | None = None,
    threshold: NumberType = 20,
    inplace: bool = False,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.softplus
    """

    if inplace:
        raise NotImplementedError
````
- **EN**: This chunk defines `softplus`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `softplus`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 490-514 / 第 490-514 行
````python
    rhs: TensorLikeType
    if beta is not None:
        python_type = utils.dtype_to_type(a.dtype)
        if not utils.is_weakly_lesser_type(type(beta), python_type):
            msg = f"beta argument of type {type(beta)} cannot be safely cast to type {python_type}!"
            raise ValueError(msg)
        scaled_input = a * beta
        rhs = torch.true_divide(torch.log1p(torch.exp(scaled_input)), beta)  # type: ignore[arg-type]

    else:
        scaled_input = a
        rhs = torch.log1p(torch.exp(scaled_input))

    return torch.where(scaled_input > threshold, a, rhs)


@aten.hardshrink.default.py_impl(DispatchKey.Autograd)
@register_decomposition(aten.hardshrink)
@out_wrapper()
def hardshrink(a: TensorLikeType, lambd: float = 0.5):
    # Formula for reference,
    # hardshrink(x) = x if x > lambd
    #               = x if x < -lambd
    #               = 0 otherwise
    return torch.where(torch.abs(a) <= lambd, 0, a)
````
- **EN**: This chunk defines `hardshrink`, which implements a focused helper used by the surrounding module. Decorators such as `aten.hardshrink.default.py_impl`, `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `hardshrink`，其作用是实现周边模块使用的关键辅助逻辑。 像 `aten.hardshrink.default.py_impl`、`register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 517-537 / 第 517-537 行
````python
@aten.softshrink.default.py_impl(DispatchKey.Autograd)
@register_decomposition(aten.softshrink)
@out_wrapper()
def softshrink(a: TensorLikeType, lambd: float = 0.5):
    # Formula for reference,
    # softshrink(x) = x - lambd if x > lambd
    #               = x + lambd if x < -lambd
    #               = 0 otherwise
    torch._check(
        0 <= lambd <= torch.finfo(a.dtype).max,
        lambda: f"lambda must be in range [0, {torch.finfo(a.dtype).max}] for input dtype {a.dtype}, but found {lambd}",
    )
    # We implement this in one torch.where to generate better code in the backward
    # see https://github.com/pytorch/pytorch/pull/107052#discussion_r1293748211
    # We multiply by 0 for dealing with nans
    return torch.where(torch.abs(a) > lambd, a - torch.sign(a) * lambd, a * 0)


# Losses
def _reduction_int_to_str(reduction: int) -> str:
    from torch._decomp.decompositions import Reduction
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._decomp.decompositions. This chunk defines `_reduction_int_to_str`, which implements a focused helper used by the surrounding module. Decorators such as `aten.softshrink.default.py_impl`, `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._decomp.decompositions。 这一段定义了 `_reduction_int_to_str`，其作用是实现周边模块使用的关键辅助逻辑。 像 `aten.softshrink.default.py_impl`、`register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 539-560 / 第 539-560 行
````python
    if reduction == Reduction.NONE.value:
        return "none"
    elif reduction == Reduction.MEAN.value:
        return "mean"
    elif reduction == Reduction.SUM.value:
        return "sum"
    else:
        raise ValueError(f"{reduction} is not a valid value for reduction")


def _apply_loss_reduction(loss: TensorLikeType, reduction: str) -> TensorLikeType:
    if reduction == "sum":
        return torch.sum(loss)
    elif reduction == "mean":
        return torch.mean(loss)
    else:  # reduction == "none"
        return loss


def _check_reduction_value(reduction: str):
    if reduction not in ("mean", "sum", "none"):
        raise ValueError(f"{reduction} is not a valid value for reduction")
````
- **EN**: This chunk defines `_check_reduction_value`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_reduction_value`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 563-590 / 第 563-590 行
````python
# This helper function maps depreciated arguments, "size_average" and "reduce"
# to their corresponding "reduction" string argument
def _get_string_reduction_arg(*, size_average: bool | None, reduce: bool | None) -> str:
    if size_average is None:
        size_average = True
    if reduce is None:
        reduce = True
    if size_average and reduce:
        ret = "mean"
    elif reduce:
        ret = "sum"
    else:
        ret = "none"
    return ret


# CompositeImplicitAutograd - don't register decomp
@elementwise_type_promotion_wrapper(
    type_promoting_args=("input", "target"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.COMPLEX_TO_FLOAT,
)
def l1_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    size_average: bool | None = None,
    reduce: bool | None = None,
    reduction: str = "mean",
) -> TensorLikeType:
````
- **EN**: This chunk defines `l1_loss`, which implements a focused helper used by the surrounding module. Decorators such as `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `l1_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 591-618 / 第 591-618 行
````python
    """
    Reference implementation of torch.nn.functional.l1_loss
    """
    if size_average is not None or reduce is not None:
        # TODO: Raise exception instead of converting value.  This is only for
        # primTorch since it can drop support for deprecated arguments.
        # msg = "size_average and reduce args are deprecated, please use reduction argument."
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)
    _check_reduction_value(reduction)
    loss = torch.abs(input - target)
    return _apply_loss_reduction(loss, reduction)


@elementwise_type_promotion_wrapper(
    type_promoting_args=("input", "target"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.COMPLEX_TO_FLOAT,
)
def smooth_l1_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    size_average: bool | None = None,
    reduce: bool | None = None,
    reduction: str = "mean",
    beta: float = 1.0,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.smooth_l1_loss
    """
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `smooth_l1_loss`, which implements a focused helper used by the surrounding module. Decorators such as `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `smooth_l1_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 619-646 / 第 619-646 行
````python
    if size_average is not None or reduce is not None:
        # TODO: Raise exception instead of converting value.  This is only for
        # primTorch since it can drop support for deprecated arguments.
        # msg = "size_average and reduce args are deprecated, please use reduction argument."
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)
    _check_reduction_value(reduction)

    if beta == 0.0:
        return torch.nn.functional.l1_loss(
            input, target, size_average=size_average, reduce=reduce, reduction=reduction
        )
    else:
        loss = torch.abs(input - target)

        loss = torch.where(loss < beta, 0.5 * loss**2 / beta, loss - 0.5 * beta)
        return _apply_loss_reduction(loss, reduction)


# Forwarding alias: the functional variant doesn't support the out kwarg
# CompositeImplicitAutograd - don't register decomp
def log_softmax(
    a: TensorLikeType,
    dim: int | None = None,
    _stacklevel: int = 3,  # for compat when using TorchRefsMode(strict=True)
    dtype: torch.dtype | None = None,
) -> TensorLikeType:
    # The error is for compat with regular PyTorch, which has this behavior
    # deprecated.  For PrimTorch, it's fine to drop support for deprecated
````
- **EN**: This chunk defines `log_softmax`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `log_softmax`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 647-669 / 第 647-669 行
````python
    # behavior because it requires explicit opt in.  This error is to inform
    # users how to update their calls.
    torch._check(dim is not None, lambda: "implicit dim not supported, use dim=X")
    return torch.log_softmax(a=a, dim=dim, dtype=dtype)  # type: ignore[call-overload]


@register_decomposition(aten.margin_ranking_loss)
def margin_ranking_loss(
    input1: TensorLikeType,
    input2: TensorLikeType,
    target: TensorLikeType,
    margin: float = 0.0,
    reduction: str = "mean",
) -> TensorLikeType:
    # loss_without_reduction = max(0, -target * (input1 - input2) + margin)
    if input1.ndim != input2.ndim or input1.ndim != target.ndim:
        raise RuntimeError(
            "margin_ranking_loss : All input tensors should have same dimension but got sizes: "
            f"input1: {input1.shape}, input2: {input2.shape}, target: {target.shape} "
        )
    _check_reduction_value(reduction)
    loss = torch.clamp_min(-target * (input1 - input2) + margin, 0)
    return _apply_loss_reduction(loss, reduction)
````
- **EN**: This chunk defines `margin_ranking_loss`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `margin_ranking_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 672-690 / 第 672-690 行
````python
@elementwise_type_promotion_wrapper(
    type_promoting_args=("input", "target"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.COMPLEX_TO_FLOAT,
)
def mse_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    size_average: bool | None = None,
    reduce: bool | None = None,
    reduction: str = "mean",
) -> TensorLikeType:
    if size_average is not None or reduce is not None:
        # TODO: Raise exception instead of converting value.  This is only for
        # primTorch since it can drop support for deprecated arguments.
        # msg = "size_average and reduce args are deprecated, please use reduction argument."
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)
    _check_reduction_value(reduction)
    loss = torch.pow(input - target, 2)
    return _apply_loss_reduction(loss, reduction)
````
- **EN**: This chunk defines `mse_loss`, which implements a focused helper used by the surrounding module. Decorators such as `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mse_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 693-720 / 第 693-720 行
````python
@register_decomposition(aten.hinge_embedding_loss)
def hinge_embedding_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    margin: float = 1.0,
    reduction: str = "mean",
) -> TensorLikeType:
    # loss_without_reduction = input if y == 1
    #                        = max(0, margin - input) if y == -1
    _check_reduction_value(reduction)
    margin_clamp = torch.clamp_min(margin - input, 0)
    output_margin = torch.where(target != 1, margin_clamp, 0)
    output_self = torch.where(target != -1, input, 0)
    loss = output_margin + output_self
    return _apply_loss_reduction(loss, reduction)


def _nll_loss_nd(
    input: TensorLikeType,
    target: TensorLikeType,
    weight: TensorLikeType | None,
    reduction: str,
    ignore_index: int,
) -> TensorLikeType:
    torch._check(
        input.ndim > 0 and input.ndim <= 3,
        lambda: f"Expected input dimension to be either [1, 2, 3] but received {input.ndim}.",
    )
````
- **EN**: This chunk defines `_nll_loss_nd`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_nll_loss_nd`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 722-746 / 第 722-746 行
````python
    torch._check(
        (input.ndim == 1) or (input.shape[0] == target.shape[0]),
        lambda: f"Expected input batch size {input.shape[0]} to match target batch size {target.shape[0]}.",
    )

    _check_reduction_value(reduction)

    flat_target = torch.flatten(target)
    ignore_classes_mask = torch.eq(flat_target, ignore_index)

    # TODO: Enable data-dependent checks with debug mode
    # TODO: This check does not work with FakeTensor inputs; See Issue #85834
    # Explicit cast for class_check to bool; See Issue #78071
    """
    from torch._subclasses.fake_tensor import FakeTensor
    num_classes = input.shape[1] if input.ndim > 1 else input.shape[0]
    valid_classes_mask = torch.logical_and(
        (flat_target >= 0), (flat_target < num_classes)
    )
    class_check = torch.all(torch.logical_or(ignore_classes_mask, valid_classes_mask))
    torch._check(
        isinstance(target, FakeTensor) or bool(class_check.item()),
        lambda: "A target class is out-of-bounds and not the ignore index.",
    )
    """
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._subclasses.fake_tensor. This chunk continues `_nll_loss_nd` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._subclasses.fake_tensor。 这一段延续了 `_nll_loss_nd`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 748-775 / 第 748-775 行
````python
    ignore_class_weight = torch.scalar_tensor(0, dtype=input.dtype, device=input.device)
    class_weight = (
        torch.scalar_tensor(1, dtype=input.dtype, device=input.device)
        if weight is None
        else weight[flat_target]
    )
    current_weight = torch.where(
        ignore_classes_mask,
        ignore_class_weight,
        class_weight,
    )

    if input.ndim == 1:
        # implicit batch size = 1
        # input (1 batch size, C classes)
        loss = -input[target] * current_weight
    elif input.ndim == 2:
        # input (N batch size, C classes)
        batch_size = input.shape[0]
        loss = -input[torch.arange(batch_size), target] * current_weight
    else:
        # 3D case (N batch size, C classes, K dimensions)
        # input (N batch size, C classes, K)
        batch_size = input.shape[0]
        extent = input.shape[2]
        numel = batch_size * extent
        indices = torch.arange(numel)
        bdx = indices // extent
````
- **EN**: This chunk continues `_nll_loss_nd` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_nll_loss_nd`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 776-803 / 第 776-803 行
````python
        kdx = indices % extent
        loss = -input[bdx, flat_target, kdx] * current_weight
    loss = torch.reshape(loss, target.shape)

    if reduction == "none":
        return loss
    elif reduction == "sum":
        return torch.sum(loss)
    else:
        # calculate weighted mean of the loss function
        return torch.sum(loss) / torch.sum(current_weight)


@register_decomposition(aten.nll_loss)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("input",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def nll_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    weight: TensorLikeType | None = None,
    size_average: bool | None = None,
    ignore_index: int = -100,
    reduce: bool | None = None,
    reduction: str = "mean",
) -> TensorLikeType:
````
- **EN**: This chunk defines `nll_loss`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `nll_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 804-830 / 第 804-830 行
````python
    """
    Reference implementation of torch.nn.functional.nll_loss
    """
    torch._check(
        input.ndim > 0,
        lambda: f"Expected input tensor to have 1 or more dimensions (got {input.ndim})",
    )

    # TODO: raise exception instead of converting value
    # msg = "size_average and reduce args are deprecated, please use reduction argument."
    # Convert these options for consistency with the eager mode
    if size_average is not None or reduce is not None:
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)

    # The expected behavior when the target and input have zero elements:
    #   reduction = 'none' --- tensor([])
    #   reduction = 'sum'  --- tensor(0.)
    #   reduction = 'mean' --- tensor(nan)
    # Mean reduction on empty tensors produces NaN. See the discussion in
    # https://github.com/pytorch/pytorch/pull/64572#issuecomment-926504162
    if input.numel() == 0 and target.numel() == 0:
        if reduction == "none":
            return torch.zeros_like(target)
        elif reduction == "sum":
            return torch.empty_like(target)
        else:
            return torch.full_like(target, float("nan"))
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `nll_loss` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `nll_loss`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 832-856 / 第 832-856 行
````python
    # The _nll_loss_nd helper function handles the most common cases.
    # ndim == 1 (Single Example)
    #   => Batch Size: 1, Input: (C), Target: ()
    # ndim == 2 (k = 1)
    #   => Batch Size: N, Input: (N, C), Target: (N)
    # ndim == 3 (k > 1)
    #   => Batch Size: N, Input: (N, C, K), Target: (N, K)
    if input.ndim <= 3:
        return _nll_loss_nd(input, target, weight, reduction, ignore_index)

    # For ndim > 3, we reshape the input and target to 3-D case.
    # Input (N batch-size, C classes, k-dimensions)
    # Target (N batch-size, k-dimensions)
    torch._check(
        input.ndim > 0 and target.ndim > 0 and target.shape[1:] == input.shape[2:],
        lambda: (
            "Expected input and target to both have ndim > 0 and "
            "target.shape[1:] == input.shape[2:], but got "
            f"target.shape {target.shape} and input.shape {input.shape}"
        ),
    )

    batch_size = input.shape[0]
    num_classes = input.shape[1]
    out_size = [batch_size] + list(target.shape[1:])
````
- **EN**: This chunk continues `nll_loss` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `nll_loss`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 858-885 / 第 858-885 行
````python
    input = torch.reshape(input, [batch_size, num_classes, -1])
    target = torch.reshape(target, [batch_size, -1])
    if reduction != "none":
        return _nll_loss_nd(input, target, weight, reduction, ignore_index)
    else:
        result = _nll_loss_nd(input, target, weight, reduction, ignore_index)
        # reshape flattened inner-dim to original k-dimensions
        return torch.reshape(result, out_size)


# TODO: This ref supports int reduction and out kwarg to be compatible with ATen:
# https://github.com/pytorch/pytorch/issues/83931
# TODO: Could be rewritten to support complex:
# https://github.com/pytorch/pytorch/pull/85041
@register_decomposition(aten.huber_loss)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("input", "target"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def huber_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    reduction: str | int = "mean",
    delta: float = 1.0,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.huber_loss
````
- **EN**: This chunk defines `huber_loss`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `huber_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 886-913 / 第 886-913 行
````python
    """
    if type(reduction) is int:
        reduction = _reduction_int_to_str(reduction)
    _check_reduction_value(reduction)  # type: ignore[arg-type]
    torch._check(
        delta > 0,
        lambda: "huber_loss does not support non-positive values for delta.",
    )
    z = (input - target).abs()
    loss = torch.where(z < delta, 0.5 * z * z, delta * (z - 0.5 * delta))
    return _apply_loss_reduction(loss, reduction)  # type: ignore[arg-type]


# tanhshrink does not use _make_elementwise_unary_reference because it does not support out
@elementwise_unary_scalar_wrapper
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def tanhshrink(a: TensorLikeType) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.tanhshrink
    """
    if not isinstance(a, TensorLike):
        raise RuntimeError(
            "Expected a tensor input for an elementwise unary operation!"
        )
    return a - torch.tanh(a)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `tanhshrink`, which implements a focused helper used by the surrounding module. Decorators such as `elementwise_unary_scalar_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `tanhshrink`，其作用是实现周边模块使用的关键辅助逻辑。 像 `elementwise_unary_scalar_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 916-936 / 第 916-936 行
````python
@register_decomposition(aten.threshold)
@_inplace_wrapper
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def threshold(
    a: TensorLikeType,
    threshold: NumberType,
    value: bool | int | float,
    inplace: bool = False,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.threshold
    """

    if inplace:
        raise NotImplementedError

    return torch.where(a <= threshold, value, a)
````
- **EN**: This chunk defines `threshold`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `threshold`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 939-960 / 第 939-960 行
````python
# CompositeImplicitAutograd - don't register decomp
# No elementwise type promotion - core op doesn't explicitly type promote
def triplet_margin_loss(
    anchor: TensorLikeType,
    positive: TensorLikeType,
    negative: TensorLikeType,
    margin: float = 1.0,
    p: float = 2,
    eps: float = 1e-6,
    swap: bool = False,
    size_average: bool | None = None,
    reduce: bool | None = None,
    reduction: str = "mean",
) -> TensorLikeType:
    if size_average is not None or reduce is not None:
        # TODO: Raise exception instead of converting value.  This is only for
        # primTorch since it can drop support for deprecated arguments.
        # msg = "size_average and reduce args are deprecated, please use reduction argument."
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)

    if margin <= 0:
        raise ValueError(f"margin must be greater than 0, got {margin}")
````
- **EN**: This chunk defines `triplet_margin_loss`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `triplet_margin_loss`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 962-988 / 第 962-988 行
````python
    # torch.nn.functional.triplet_margin_with_distance_loss has no ref defined
    # since it's a pure Python implementation.  Use this helper instead.
    return _triplet_margin_with_distance_loss(
        anchor=anchor,
        positive=positive,
        negative=negative,
        distance_function=lambda x, y: torch.pairwise_distance(x, y, p, eps),
        margin=margin,
        swap=swap,
        reduction=reduction,
    )


# Pure Python impl - don't register decomp and don't add a ref.  Defined as a
# helper here since triplet_margin_loss can be nicely implemented with it.
def _triplet_margin_with_distance_loss(
    anchor: TensorLikeType,
    positive: TensorLikeType,
    negative: TensorLikeType,
    *,
    distance_function: Callable[[TensorLikeType, TensorLikeType], TensorLikeType]
    | None = None,
    margin: float = 1.0,
    swap: bool = False,
    reduction: str = "mean",
) -> TensorLikeType:
    _check_reduction_value(reduction)
````
- **EN**: This chunk defines `_triplet_margin_with_distance_loss`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_triplet_margin_with_distance_loss`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 990-1016 / 第 990-1016 行
````python
    a_dim = anchor.ndim
    p_dim = positive.ndim
    n_dim = negative.ndim
    torch._check(
        a_dim == p_dim and p_dim == n_dim,
        lambda: (
            f"The anchor, positive, and negative tensors are expected to have "
            f"the same number of dimensions, but got: anchor {a_dim}D, "
            f"positive {p_dim}D, and negative {n_dim}D inputs"
        ),
    )

    if distance_function is None:
        distance_function = torch.pairwise_distance

    dist_pos = distance_function(anchor, positive)
    dist_neg = distance_function(anchor, negative)
    # The distance swap is described in the paper "Learning shallow
    # convolutional feature descriptors with triplet losses" by V. Balntas, E.
    # Riba et al.  If True, and if the positive example is closer to the
    # negative example than the anchor is, swaps the positive example and the
    # anchor in the loss computation.
    if swap:
        dist_swap = distance_function(positive, negative)
        dist_neg = torch.minimum(dist_neg, dist_swap)
    loss = torch.clamp_min(margin + dist_pos - dist_neg, 0)
    return _apply_loss_reduction(loss, reduction)
````
- **EN**: This chunk continues `_triplet_margin_with_distance_loss` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_triplet_margin_with_distance_loss`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1019-1039 / 第 1019-1039 行
````python
@register_decomposition(aten.hardtanh)
@_inplace_wrapper
@out_wrapper()
@elementwise_unary_scalar_wrapper
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def hardtanh(
    a: TensorLikeType,
    min_val: NumberType = -1,
    max_val: NumberType = 1,
    inplace: bool = False,
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.hardtanh
    """
    if inplace:
        raise NotImplementedError
    if utils.is_boolean_dtype(a.dtype):
        raise RuntimeError("Bool inputs not supported for hardtanh")
````
- **EN**: This chunk defines `hardtanh`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `hardtanh`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 1041-1068 / 第 1041-1068 行
````python
    # preserve legacy behavior of boundaries not causing type promotion
    if utils.is_integer_dtype(a.dtype):
        min_val = int(min_val)  # type: ignore[arg-type]
        max_val = int(max_val)  # type: ignore[arg-type]
        if not (a.dtype != torch.uint8 or (min_val >= 0 and max_val >= 0)):
            raise RuntimeError(
                "Cannot do hardtanh on an unsigned type with negative limits"
            )

    if min_val > max_val:  # type: ignore[operator]
        raise ValueError("min_val cannot be greater than max_val")

    return torch.clamp(a, min_val, max_val)  # type: ignore[arg-type]


@register_decomposition(aten.gelu)
@out_wrapper()
@elementwise_unary_scalar_wrapper
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def gelu(a: TensorLikeType, approximate: str = "none") -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.gelu
    """
    if not isinstance(a, TensorLike):
        raise RuntimeError(
````
- **EN**: This chunk defines `gelu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_unary_scalar_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `gelu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_unary_scalar_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1069-1096 / 第 1069-1096 行
````python
            "Expected a tensor input for an elementwise unary operation!"
        )
    M_SQRT2 = 1.41421356237309504880
    M_SQRT1_2 = 0.70710678118654752440
    M_2_SQRTPI = 1.12837916709551257390
    if approximate == "tanh":
        kBeta = M_SQRT2 * M_2_SQRTPI * 0.5
        kKappa = 0.044715
        a_cube = a * a * a
        inner = kBeta * (a + kKappa * a_cube)
        return 0.5 * a * (1 + torch.tanh(inner))
    elif approximate == "none":
        kAlpha = M_SQRT1_2
        return a * 0.5 * (1 + torch.erf(a * kAlpha))
    else:
        raise RuntimeError("approximate argument must be either none or tanh.")


# CompositeImplicitAutograd - don't register decomp
@elementwise_type_promotion_wrapper(
    type_promoting_args=("input", "target"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def poisson_nll_loss(
    input: TensorLikeType,
    target: TensorLikeType,
    log_input: bool = True,
    full: bool = False,
````
- **EN**: This chunk defines `poisson_nll_loss`, which implements a focused helper used by the surrounding module. Decorators such as `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `poisson_nll_loss`，其作用是实现周边模块使用的关键辅助逻辑。 像 `elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1097-1122 / 第 1097-1122 行
````python
    size_average: bool | None = None,
    eps: float = 1e-8,
    reduce: bool | None = None,
    reduction: str = "mean",
) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.poisson_nll_loss
    """
    if size_average is not None or reduce is not None:
        # TODO: Raise exception instead of converting value.  This is only for
        # primTorch since it can drop support for deprecated arguments.
        # msg = "size_average and reduce args are deprecated, please use reduction argument."
        reduction = _get_string_reduction_arg(size_average=size_average, reduce=reduce)
    _check_reduction_value(reduction)
    if log_input:
        loss = torch.exp(input) - target * input
    else:
        loss = input - target * torch.log(input + eps)

    if full:
        stirling_term = (
            target * torch.log(target) - target + 0.5 * torch.log(2 * torch.pi * target)
        )
        # avoid inplace add
        loss = loss + stirling_term.masked_fill(target <= 1, 0)
    return _apply_loss_reduction(loss, reduction)
````
- **EN**: This chunk continues `poisson_nll_loss` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `poisson_nll_loss`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1125-1150 / 第 1125-1150 行
````python
@register_decomposition(aten.prelu)
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a", "weight"),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def prelu(a: TensorLikeType, weight: TensorLikeType) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.prelu
    """
    torch._check(
        isinstance(a, TensorLike),
        lambda: f"prelu: Expected `a` to be tensor, but got: {type(a)}",
    )
    torch._check(
        isinstance(weight, TensorLike),
        lambda: f"prelu: Expected `weight` to be tensor, but got: {type(weight)}",
    )

    if weight.numel() != 1:
        torch._check(a.ndim > 0, lambda: "Not allow zero-dim input tensor.")
        channel_size = a.shape[1] if a.ndim >= 2 else 1
        torch._check(
            weight.numel() == channel_size,
            lambda: f"Mismatch of parameter numbers and input channel size. Found parameter numbers ="
            f" {weight.numel()} and channel size = {channel_size}.",
        )
````
- **EN**: This chunk defines `prelu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `prelu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1152-1175 / 第 1152-1175 行
````python
    torch._check(
        weight.ndim == 0 or weight.ndim == 1,
        lambda: f"prelu: Expected `weight` to be a scalar or 1D tensor, but got: "
        f"ndim = {weight.ndim}",
    )
    if a.ndim == 0:
        weight = weight[0] if weight.ndim == 1 else weight
    else:
        weight = prims.broadcast_in_dim(
            weight, a.shape, () if weight.ndim == 0 else (0 if a.ndim == 1 else 1,)
        )

    return torch.where(a > 0, a, a * weight)


@register_decomposition(aten.relu6)
@_inplace_wrapper
@out_wrapper()
def relu6(a: TensorLikeType, inplace: bool = False) -> TensorLikeType:
    """
    Reference implementation of torch.nn.functional.relu6
    """
    if inplace:
        raise NotImplementedError
````
- **EN**: This chunk defines `relu6`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `_inplace_wrapper`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `relu6`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`_inplace_wrapper`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1177-1197 / 第 1177-1197 行
````python
    # See https://github.com/pytorch/pytorch/pull/81142#discussion_r918220126
    # It may be better to use clamp here, but we use hardtanh to replicate
    # the behavior of the existing implementation
    return torch.nn.functional.hardtanh(a, 0, 6)


@register_decomposition(aten.glu)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def glu(a: TensorLikeType, dim: int = -1) -> TensorLikeType:
    dim = utils.canonicalize_dims(a.ndim, dim)
    torch._check(
        a.shape[dim] % 2 == 0,
        lambda: f"Halving dimension must be even, but dimension {dim} is size {a.shape[dim]}",
    )
    b, c = torch.tensor_split(a, 2, dim)

    return b * torch.sigmoid(c)
````
- **EN**: This chunk defines `glu`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `glu`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1200-1227 / 第 1200-1227 行
````python
@register_decomposition(aten.pairwise_distance)
@out_wrapper()
def pairwise_distance(
    x1: TensorLikeType,
    x2: TensorLikeType,
    p: NumberType = 2.0,
    eps: NumberType = 1e-6,
    keepdim=False,
) -> TensorLikeType:
    return torch.linalg.vector_norm(x1 - x2 + eps, ord=p, dim=-1, keepdim=keepdim)


@register_decomposition(aten.pdist)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT,
)
def pdist(a: TensorLikeType, p: float = 2) -> TensorLikeType:
    torch._check(a.ndim == 2, lambda: f"pdist only supports 2D tensors, got: {a.ndim}D")
    torch._check(p >= 0, lambda: "pdist only supports non-negative p values")
    # For p == 2 we can use an efficient implementation, but other values of p
    # require creating a much bigger tensor for an intermediate step
    if p == 2:
        aTa = torch.mm(a, a.T)
        aTa_diag = torch.diag(aTa)
        t = torch.sqrt(torch.clamp(aTa_diag + aTa_diag.unsqueeze(-1) - 2 * aTa, min=0))
    else:
````
- **EN**: This chunk defines `pdist`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `pdist`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1228-1255 / 第 1228-1255 行
````python
        t = torch.linalg.vector_norm(a.unsqueeze(1) - a, ord=p, dim=2)
    i = torch.triu_indices(t.shape[0], t.shape[1], offset=1, device=a.device)
    return t.flatten().index_select(0, i[0] * t.shape[0] + i[1])


@register_decomposition(aten.pixel_shuffle)
@out_wrapper()
def pixel_shuffle(self: Tensor, upscale_factor: int):
    torch._check(
        self.dim() >= 3,
        lambda: f"pixel_shuffle expects input to have at least 3 dimensions, but got input with {self.dim} dimension(s)",
    )
    batch = self.shape[:-3]
    C_out = self.shape[-3] // upscale_factor**2
    HW_out = (self.shape[-2] * upscale_factor, self.shape[-1] * upscale_factor)
    n = len(batch)
    B_dims = range(n)
    C_dim, r1_dim, r2_dim, H_dim, W_dim = range(n, n + 5)
    return (
        self.view(
            *batch,
            C_out,
            upscale_factor,
            upscale_factor,
            self.shape[-2],
            self.shape[-1],
        )
        .permute(*B_dims, C_dim, H_dim, r1_dim, W_dim, r2_dim)
````
- **EN**: This chunk defines `pixel_shuffle`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `pixel_shuffle`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1256-1283 / 第 1256-1283 行
````python
        .reshape(*batch, C_out, *HW_out)
        .clone(memory_format=utils.suggest_memory_format(self))
    )


@register_decomposition(aten.pixel_unshuffle)
@out_wrapper()
def pixel_unshuffle(self: Tensor, downscale_factor: int):
    torch._check(
        self.dim() >= 3,
        lambda: f"pixel_unshuffle expects input to have at least 3 dimensions, but got input with {self.dim} dimension(s)",
    )
    batch = self.shape[:-3]
    C_out = self.shape[-3] * downscale_factor**2
    HW_out = (self.shape[-2] // downscale_factor, self.shape[-1] // downscale_factor)
    n = len(batch)
    B_dims = range(n)
    C_dim, H_dim, r1_dim, W_dim, r2_dim = range(n, n + 5)
    return (
        self.view(
            *batch,
            self.shape[-3],
            HW_out[0],
            downscale_factor,
            HW_out[1],
            downscale_factor,
        )
        .permute(*B_dims, C_dim, r1_dim, r2_dim, H_dim, W_dim)
````
- **EN**: This chunk defines `pixel_unshuffle`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `pixel_unshuffle`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 1284-1294 / 第 1284-1294 行
````python
        .reshape(*batch, C_out, *HW_out)
        .clone(memory_format=utils.suggest_memory_format(self))
    )


# Needed as aten.{celu_,elu_...} exist (even if they don't have the in-place kwarg)
celu_ = _make_inplace(celu)
elu_ = _make_inplace(elu)
mish_ = _make_inplace(mish)
selu_ = _make_inplace(selu)
threshold_ = _make_inplace(threshold)
````
- **EN**: This chunk continues `pixel_unshuffle` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `pixel_unshuffle`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Reference implementations**
  - EN: Provides readable Python operator definitions used as correctness references.
  - CN: 提供可读的 Python 算子定义，作为正确性参考实现。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_T**
  - EN: `_T` is one of the main symbols declared or implemented in this file.
  - CN: `_T` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Decomposition**
  - EN: The implementation rewrites larger operators into smaller primitives for reuse and lowering.
  - CN: 实现把较大的算子重写为更小的 primitive，以便复用和降级。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._prims`, `torch._prims_common`, `torch._refs`, `torch._decomp`, `torch._prims_common.wrappers`, `torch._meta_registrations`, `torch._decomp.decompositions`, `torch._subclasses.fake_tensor`
- **Standard library / 标准库**: `math`, `collections.abc`, `functools`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_T`, `_P`, `_dropout_helper`, `alpha_dropout`, `_inplace_wrapper`, `celu`, `dropout`, `elu`, `relu`
