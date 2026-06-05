# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_refs/special/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import math
from typing import Optional, Union

import torch
import torch._prims as prims
import torch._prims_common as utils
import torch._refs as refs
from torch import Tensor
from torch._decomp import register_decomposition
from torch._prims_common import (
    ELEMENTWISE_TYPE_PROMOTION_KIND,
    Number,
    NumberType,
    TensorLike,
    TensorLikeType,
)
from torch._prims_common.wrappers import elementwise_type_promotion_wrapper, out_wrapper
from torch._refs import (
    _make_alias,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._prims, torch._prims_common, ...; standard-library helpers such as math, typing.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._prims、torch._prims_common、...；标准库辅助模块，如 math、typing。

### Lines 21-40 / 第 21-40 行
````python
    _make_elementwise_binary_reference,
    _make_elementwise_unary_reference,
)


__all__ = [
    "bessel_j0",
    "bessel_j1",
    "entr",
    "erfcx",
    "expit",
    "i0e",
    "i1",
    "i1e",
    "log_ndtr",
    "logit",
    "log_softmax",
    "multigammaln",
    "ndtr",
    "ndtri",
````
- **EN**: `__all__` defines the public symbols that this module chooses to export.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。

### Lines 41-53 / 第 41-53 行
````python
    "softmax",
    "spherical_bessel_j0",
    "xlog1py",
    "zeta",
]
aten = torch._ops.ops.aten


@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def bessel_j0(a: TensorLikeType) -> TensorLikeType:
    return prims.bessel_j0(a)
````
- **EN**: This chunk defines `bessel_j0`, which implements a focused helper used by the surrounding module. Decorators such as `_make_elementwise_unary_reference` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `bessel_j0`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_make_elementwise_unary_reference` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 56-74 / 第 56-74 行
````python
@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def bessel_j1(a: TensorLikeType) -> TensorLikeType:
    return prims.bessel_j1(a)


@register_decomposition(aten.special_entr)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def entr(a: TensorLikeType) -> TensorLikeType:
    return torch.where(
        torch.isnan(a),
        a,
        torch.where(a > 0, -a * torch.log(a), torch.where(a == 0, 0, -torch.inf)),
    )
````
- **EN**: This chunk defines `entr`, which implements a focused helper used by the surrounding module. Decorators such as `_make_elementwise_unary_reference`, `register_decomposition`, `out_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `entr`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_make_elementwise_unary_reference`、`register_decomposition`、`out_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 77-95 / 第 77-95 行
````python
@register_decomposition(aten.special_erfcx)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def erfcx(a: TensorLikeType) -> TensorLikeType:
    return prims.erfcx(a)


# alias for sigmoid
expit = _make_alias(torch.sigmoid, "expit")


@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def i0e(a: TensorLikeType) -> TensorLikeType:
    return prims.bessel_i0e(a)
````
- **EN**: This chunk defines `i0e`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper`, ... adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `i0e`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper`、... 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 98-109 / 第 98-109 行
````python
@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def i1(a: TensorLikeType) -> TensorLikeType:
    return prims.bessel_i1(a)


@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def i1e(a: TensorLikeType) -> TensorLikeType:
    return prims.bessel_i1e(a)
````
- **EN**: This chunk defines `i1e`, which implements a focused helper used by the surrounding module. Decorators such as `_make_elementwise_unary_reference` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `i1e`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_make_elementwise_unary_reference` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 112-126 / 第 112-126 行
````python
@register_decomposition(aten.special_log_ndtr)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def log_ndtr(a: TensorLikeType) -> TensorLikeType:
    # Note: M_SQRT1_2 is the value of 1 / sqrt(2)
    M_SQRT1_2 = 0.707106781186547524400844362104849039
    t = a * M_SQRT1_2
    return torch.where(
        a < 1.0,
        torch.log(torch.special.erfcx(-t) / 2) - t * t,
        torch.log1p(-torch.erfc(t) / 2),
    )
````
- **EN**: This chunk defines `log_ndtr`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `log_ndtr`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-141 / 第 129-141 行
````python
@register_decomposition(aten.logit)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("self",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def logit(self: TensorLikeType, eps: float | None = None) -> TensorLikeType:
    if eps is None:
        eps = -1.0
    lo = eps
    hi = 1 - eps
    self = torch.where(self < lo, lo, torch.where(self > hi, hi, self))
    return torch.log(torch.true_divide(self, torch.sub(1, self)))
````
- **EN**: This chunk defines `logit`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `logit`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 144-162 / 第 144-162 行
````python
@register_decomposition(aten.special_xlog1py)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a", "b"),
    type_promotion_kind=ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def xlog1py(a: TensorLikeType | NumberType, b: TensorLikeType | NumberType):
    torch._check(
        isinstance(a, TensorLike) or isinstance(b, TensorLike),
        lambda: 'Expected either argument a or b to be a Tensor"',
    )

    # Operations like eq and log do not handle scalar values, so we convert them to scalar_tensors.
    if isinstance(a, TensorLike) and isinstance(b, Number):
        # pyrefly: ignore [bad-argument-type]
        b = refs.scalar_tensor(b, dtype=a.dtype, device=a.device)
    elif isinstance(b, TensorLike) and isinstance(a, Number):
        # pyrefly: ignore [bad-argument-type]
        a = refs.scalar_tensor(a, dtype=b.dtype, device=b.device)
````
- **EN**: This chunk defines `xlog1py`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `xlog1py`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 164-182 / 第 164-182 行
````python
    # mypy: expected "Tensor"
    if not isinstance(a, TensorLike):
        raise AssertionError(f"a must be TensorLike, got {type(a)}")
    if not isinstance(b, TensorLike):
        raise AssertionError(f"b must be TensorLike, got {type(b)}")
    rhs = torch.where(torch.eq(a, 0), 0, torch.mul(a, torch.log1p(b)))
    return torch.where(torch.isnan(b), float("nan"), rhs)


@register_decomposition(aten.mvlgamma)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def multigammaln(a: TensorLikeType, p: int) -> TensorLikeType:
    c = 0.25 * p * (p - 1) * math.log(math.pi)
    b = 0.5 * torch.arange(start=(1 - p), end=1, step=1, dtype=a.dtype, device=a.device)
    return torch.sum(torch.lgamma(a.unsqueeze(-1) + b), dim=-1) + c
````
- **EN**: This chunk defines `multigammaln`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `multigammaln`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 185-204 / 第 185-204 行
````python
@register_decomposition(aten.special_ndtr)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def ndtr(a: TensorLikeType) -> TensorLikeType:
    # Note: M_SQRT1_2 is the value of 1 / sqrt(2)
    M_SQRT1_2 = 0.707106781186547524400844362104849039
    a_sqrt_2 = a * M_SQRT1_2
    return (1 + torch.erf(a_sqrt_2)) * 0.5


@register_decomposition(aten.special_ndtri)
@out_wrapper()
@elementwise_type_promotion_wrapper(
    type_promoting_args=("a",),
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def ndtri(a: TensorLikeType) -> TensorLikeType:
````
- **EN**: This chunk defines `ndtri`, which implements a focused helper used by the surrounding module. Decorators such as `register_decomposition`, `out_wrapper`, `elementwise_type_promotion_wrapper` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `ndtri`，其作用是实现周边模块使用的关键辅助逻辑。 像 `register_decomposition`、`out_wrapper`、`elementwise_type_promotion_wrapper` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 205-224 / 第 205-224 行
````python
    return prims.ndtri(a)


# Forwarding alias: the special variant doesn't support the out kwarg
# CompositeImplicitAutograd - don't register decomp
def log_softmax(
    a: TensorLikeType,
    dim: int,
    dtype: torch.dtype | None = None,
) -> TensorLikeType:
    return torch.log_softmax(a=a, dim=dim, dtype=dtype)  # type: ignore[call-overload]


# Forwarding alias: the special variant doesn't support the out kwarg
# CompositeImplicitAutograd - don't register decomp
def softmax(
    a: TensorLikeType,
    dim: int,
    dtype: torch.dtype | None = None,
) -> TensorLikeType:
````
- **EN**: This chunk defines `softmax`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `softmax`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 225-240 / 第 225-240 行
````python
    return torch.softmax(a=a, dim=dim, dtype=dtype)  # type: ignore[call-overload]


@_make_elementwise_unary_reference(
    ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def spherical_bessel_j0(a: TensorLikeType) -> TensorLikeType:
    return prims.spherical_bessel_j0(a)


# TODO: add docstring
@_make_elementwise_binary_reference(
    type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.INT_TO_FLOAT,
)
def zeta(a: TensorLikeType, b: TensorLikeType) -> TensorLikeType:
    return prims.zeta(a, b)
````
- **EN**: This chunk defines `zeta`, which implements a focused helper used by the surrounding module. Decorators such as `_make_elementwise_unary_reference`, `_make_elementwise_binary_reference` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `zeta`，其作用是实现周边模块使用的关键辅助逻辑。 像 `_make_elementwise_unary_reference`、`_make_elementwise_binary_reference` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Special functions**
  - EN: Exposes numerical special functions through Python namespace wrappers.
  - CN: 通过 Python 命名空间包装层暴露数值特殊函数。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **bessel_j0**
  - EN: `bessel_j0` is one of the main symbols declared or implemented in this file.
  - CN: `bessel_j0` 是本文件声明或实现的主要符号之一。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Decomposition**
  - EN: The implementation rewrites larger operators into smaller primitives for reuse and lowering.
  - CN: 实现把较大的算子重写为更小的 primitive，以便复用和降级。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._prims`, `torch._prims_common`, `torch._refs`, `torch._decomp`, `torch._prims_common.wrappers`
- **Standard library / 标准库**: `math`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `bessel_j0`, `bessel_j1`, `entr`, `erfcx`, `i0e`, `i1`, `i1e`, `log_ndtr`, `logit`
