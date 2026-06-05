# common.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_subclasses/complex_tensor/_ops/common.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements tensor-subclass helpers, fake tensor support, and subclass-aware dispatch utilities.
- **Purpose (CN)**: 实现张量子类辅助逻辑、fake tensor 支持以及面向子类的 dispatch 工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
````python
from collections.abc import Callable
from typing import Any, overload, TypeAlias
from typing_extensions import Never, ParamSpec, TypeIs, TypeVar

import torch
from torch import Tensor
from torch._decomp import get_decompositions
from torch._ops import OpOverload, OpOverloadPacket
from torch._refs import is_complex as _is_complex
from torch.types import Number
from torch.utils._python_dispatch import TorchDispatchMode
from torch.utils._pytree import tree_flatten, tree_map, tree_unflatten

from .._core import ComplexTensor


_P = ParamSpec("_P")
_R = TypeVar("_R")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._decomp, torch._ops, ...; standard-library helpers such as collections.abc, typing; other helper packages such as typing_extensions.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._decomp、torch._ops、...；标准库辅助模块，如 collections.abc、typing；其他辅助包，如 typing_extensions。

### Lines 20-34 / 第 20-34 行
````python
OpType: TypeAlias = OpOverloadPacket | OpOverload

# pyrefly: ignore [implicit-any]
TableType: TypeAlias = dict[OpType, Callable]

# Mapping from ops to implementations
COMPLEX_OPS_TABLE: TableType = {}

COMPLEX_TO_REAL = {
    torch.complex128: torch.float64,
    torch.complex64: torch.float32,
    torch.complex32: torch.float16,
}

REAL_TO_COMPLEX = {v: k for k, v in COMPLEX_TO_REAL.items()}
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 36-50 / 第 36-50 行
````python
# Used to promote dtypes in `promote_real_cpu_tensors`
PROMOTE_TYPES = {
    torch.float16: torch.float32,
    torch.bfloat16: torch.float32,
    torch.complex32: torch.complex64,
}


def is_complex_tensor(obj: Any, /) -> TypeIs[ComplexTensor]:
    r"""Returns True if the input is a ComplexTensor, else False

    Args:
        a: any input

    Examples:
````
- **EN**: This chunk defines `is_complex_tensor`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `is_complex_tensor`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 52-65 / 第 52-65 行
````python
        >>> # xdoctest: +SKIP
        >>> from torch.complex import ComplexTensor
        >>> data = torch.zeros((3, 2), dtype=torch.complex64)
        >>> ct = ComplexTensor.from_interleaved(data)
        >>> is_complex_tensor(ct)
        True
    """
    return isinstance(obj, ComplexTensor)


@overload
def promote_tensors(
    *tensors: ComplexTensor,
) -> tuple[torch.dtype, tuple[ComplexTensor, ...]]: ...
````
- **EN**: This chunk defines `promote_tensors`, which implements a focused helper used by the surrounding module. Decorators such as `overload` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `promote_tensors`，其作用是实现周边模块使用的关键辅助逻辑。 像 `overload` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 68-85 / 第 68-85 行
````python
@overload
def promote_tensors(
    *tensors: Tensor,
) -> tuple[torch.dtype, tuple[Tensor, ...]]: ...


def promote_tensors(
    *tensors: Tensor | ComplexTensor,
) -> tuple[torch.dtype, tuple[Tensor | ComplexTensor, ...]]:
    """
    Promotes all tensors to a common dtype.
    Additionally promotes CPU tensors to at least `float32`.
    """
    tensor = next(t for t in tensors if isinstance(t, Tensor))
    out_dt = tensor.dtype
    for t in tensors:
        if isinstance(t, Tensor):
            out_dt = torch.promote_types(out_dt, t.dtype)
````
- **EN**: This chunk defines `promote_tensors`, which implements a focused helper used by the surrounding module. Decorators such as `overload` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `promote_tensors`，其作用是实现周边模块使用的关键辅助逻辑。 像 `overload` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 87-104 / 第 87-104 行
````python
    prom_dt = PROMOTE_TYPES.get(out_dt, out_dt)
    return out_dt, tuple(
        t.to(prom_dt) if isinstance(t, Tensor) else torch.asarray(t, dtype=prom_dt)
        for t in tensors
    )


def register_complex(
    op: OpType,
    func_impl: Callable[..., Any] | None = None,
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]] | Callable[..., Any]:
    """Decorator to register an implementation for some ops in some dispatch tables"""

    def inner(func: Callable[_P, _R]) -> Callable[_P, _R]:
        if COMPLEX_OPS_TABLE.get(op, func) is not func:
            raise RuntimeError(f"Attempted to register multiple functions for {op}")
        COMPLEX_OPS_TABLE[op] = func
        return func
````
- **EN**: This chunk defines `inner`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `inner`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 106-123 / 第 106-123 行
````python
    if func_impl is None:
        return inner

    return inner(func_impl)


FORCE_TEST_LIST: list[OpType] = []


def register_force_test(
    op: OpType, func_impl: Callable[..., Any] | None = None
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]] | Callable[..., Any]:
    """Will attempt to test these ops even if they err on "normal" inputs"""
    FORCE_TEST_LIST.append(op)
    return register_complex(op, func_impl)


DECOMPOSITIONS = get_decompositions(list(torch.ops.aten))  # type: ignore[no-matching-overload]
````
- **EN**: This chunk defines `register_force_test`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `register_force_test`，其作用是向周边基础设施注册钩子、schema、算子或回调。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 126-142 / 第 126-142 行
````python
def lookup_complex(
    func: OpOverload, *args: Any, **kwargs: Any
) -> Callable[..., Any] | None:
    """
    Lookup an impl from the table.

    Try the particular overload first, then the overload packet.

    If nothing is found, try the decompositions with both.
    """
    return COMPLEX_OPS_TABLE.get(
        func,
        COMPLEX_OPS_TABLE.get(
            func.overloadpacket,
            DECOMPOSITIONS.get(func, DECOMPOSITIONS.get(func.overloadpacket)),
        ),
    )
````
- **EN**: This chunk defines `lookup_complex`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `lookup_complex`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 145-159 / 第 145-159 行
````python
def is_complex(x: Any, /) -> bool:
    """Utility to detect if a given object is (known) to be complex."""
    return (isinstance(x, Tensor) and _is_complex(x)) or isinstance(x, complex)


@overload
def split_complex_arg(
    arg: Tensor | ComplexTensor,
) -> tuple[Tensor, Tensor]: ...


@overload
def split_complex_arg(
    arg: complex | Number,
) -> tuple[Number, Number]: ...
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `split_complex_arg`, which implements a focused helper used by the surrounding module. Decorators such as `overload` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `split_complex_arg`，其作用是实现周边模块使用的关键辅助逻辑。 像 `overload` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 162-181 / 第 162-181 行
````python
def split_complex_arg(
    arg: Tensor | ComplexTensor | complex | Number,
) -> tuple[Tensor, Tensor] | tuple[Number, Number]:
    """
    Split a complex argument into a real/imaginary component.

    If real, use zero for the imaginary part.
    """
    if isinstance(arg, ComplexTensor):
        return split_complex_tensor(arg)
    if isinstance(arg, Tensor):
        if is_complex(arg):
            return arg.real, arg.imag
        return arg, torch.zeros_like(arg)
    # TODO (hameerabbasi): Should there be a `torch.SymComplex`?
    if isinstance(arg, complex):
        return arg.real, arg.imag
    if isinstance(arg, float | torch.SymFloat):
        return arg, 0.0
    if isinstance(arg, int | torch.SymInt):
````
- **EN**: This chunk defines `split_complex_arg`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `split_complex_arg`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 182-195 / 第 182-195 行
````python
        return arg, 0
    if isinstance(arg, bool | torch.SymBool):
        return arg, False
    raise TypeError(f"Expected tensor or number got, {type(arg)}")


def split_complex_tensor(complex_tensor: ComplexTensor) -> tuple[Tensor, Tensor]:
    """Split a ComplexTensor into its real and imaginary parts."""
    return complex_tensor.re, complex_tensor.im


def complex_to_real_dtype(dtype: torch.dtype) -> torch.dtype:
    """Convert a complex dtype to the dtype of its real part. Return other dtypes as-is."""
    return COMPLEX_TO_REAL.get(dtype, dtype)
````
- **EN**: This chunk defines `complex_to_real_dtype`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `complex_to_real_dtype`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 198-216 / 第 198-216 行
````python
def _get_op_name(op: OpType) -> str:
    """Get the op name from the op."""
    if isinstance(op, OpOverload):
        op = op.overloadpacket
    return str(op).split(".", 1)[1]


def _get_func_name(op: OpType) -> str:
    """Get the name of the implementation function from the op."""
    return f"{_get_op_name(op)}_impl"


def register_error(
    op: OpType, exc_type: type[Exception] = NotImplementedError
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]] | Callable[..., Any]:
    msg = f"`aten.{_get_op_name(op)}` not implemented for `{ComplexTensor.__name__}`."

    def ordered_impl(*args: Any, **kwargs: Any) -> Never:
        raise exc_type(msg)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `ordered_impl`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `ordered_impl`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 218-237 / 第 218-237 行
````python
    func_name = _get_func_name(op)
    ordered_impl.__name__ = func_name
    ordered_impl.__qualname__ = func_name

    return register_force_test(op, ordered_impl)


def register_binary_nonlinear(
    op: OpType,
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]] | Callable[..., Any]:
    """Register a "multiplication-style" op, e.g. aten.mul, aten.mm, ..."""

    def impl(
        lhs: ComplexTensor, rhs: ComplexTensor, *args: Any, **kwargs: Any
    ) -> ComplexTensor:
        a_r, a_i = split_complex_arg(lhs)
        b_r, b_i = split_complex_arg(rhs)
        out_dt, (a_r, a_i, b_r, b_i) = promote_tensors(a_r, a_i, b_r, b_i)
        real = op(a_r, b_r, *args, **kwargs) - op(a_i, b_i, *args, **kwargs)
        imag = op(a_r, b_i, *args, **kwargs) + op(a_i, b_r, *args, **kwargs)
````
- **EN**: This chunk defines `impl`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `impl`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 238-250 / 第 238-250 行
````python
        return ComplexTensor(real.to(out_dt), imag.to(out_dt))

    func_name = _get_func_name(op)
    impl.__name__ = func_name
    impl.__qualname__ = func_name

    return register_complex(op, impl)


def register_simple(
    op: OpType,
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]] | Callable[..., Any]:
    """Register an op which can be applied independently to the real and complex parts to get the result."""
````
- **EN**: This chunk defines `register_simple`, which registers a hook, schema, operator, or callback with surrounding infrastructure. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `register_simple`，其作用是向周边基础设施注册钩子、schema、算子或回调。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 252-267 / 第 252-267 行
````python
    def impl(
        self: ComplexTensor, *args: Any, dtype: torch.dtype | None = None, **kwargs: Any
    ) -> ComplexTensor:
        x, y = split_complex_tensor(self)
        if dtype is not None and dtype not in COMPLEX_TO_REAL:
            raise RuntimeError(
                "Non-complex `dtype` specified, please write custom impl."
            )

        if dtype in COMPLEX_TO_REAL:
            if dtype is None:
                raise AssertionError("dtype must not be None when in COMPLEX_TO_REAL")
            kwargs["dtype"] = COMPLEX_TO_REAL[dtype]

        u = op(x, *args, **kwargs)
        v = op(y, *args, **kwargs)
````
- **EN**: This chunk defines `impl`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `impl`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 269-282 / 第 269-282 行
````python
        u_flat, u_spec = tree_flatten(u)
        v_flat, v_spec = tree_flatten(v)
        if u_spec != v_spec:
            raise AssertionError(f"Tree specs must match: {u_spec} != {v_spec}")
        out_flat = [
            ComplexTensor(ui, vi) for ui, vi in zip(u_flat, v_flat, strict=False)
        ]
        return tree_unflatten(out_flat, u_spec)

    func_name = _get_func_name(op)
    impl.__name__ = func_name
    impl.__qualname__ = func_name

    return register_complex(op, impl)
````
- **EN**: This chunk continues `impl` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `impl`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 285-300 / 第 285-300 行
````python
def _as_complex_tensor(arg: Tensor | Any) -> Tensor | ComplexTensor | Any:
    """Convert a Tensor with complex dtypes to a ComplexTensor. Pass along other args as-is."""
    if (
        not isinstance(arg, ComplexTensor)
        and isinstance(arg, Tensor)
        and arg.dtype in COMPLEX_TO_REAL
    ):
        return ComplexTensor.from_interleaved(arg)
    return arg


def _as_interleaved(arg: ComplexTensor | Any) -> Tensor | Any:
    """Convert a ComplexTensor to a Tensor with a complex dtype. Pass other arguments as-is."""
    if isinstance(arg, ComplexTensor):
        return arg.as_interleaved()
    return arg
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_as_interleaved`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_as_interleaved`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 303-316 / 第 303-316 行
````python
class ComplexTensorMode(TorchDispatchMode):
    _compile: bool

    """ A TorchDispatchMode to replace any Tensor that has a complex dtype with a ComplexTensor for the computation. """

    def __init__(self, _dispatch_key: Any = None, *, _compile: bool = False) -> None:
        """Initialize a ComplexTensorMode.

        Args:
            _dispatch_key: passed on to TorchDispatchMode
            _compile: Compile the op before the computation
        """
        super().__init__(_dispatch_key)
        self._compile = _compile
````
- **EN**: It introduces or extends `ComplexTensorMode`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 它引入或扩展了 `ComplexTensorMode`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 318-335 / 第 318-335 行
````python
    def __torch_dispatch__(
        self,
        func: OpOverload,
        types: tuple[type, ...],
        args: tuple[Any, ...] = (),
        kwargs: dict[str, Any] | None = None,
    ) -> Any:
        if kwargs is None:
            kwargs = {}

        # TODO (hameerabbasi): Test perf with `_compile` set to `True`
        if self._compile:
            func = torch.compile(func)  # type: ignore[bad-assignment]

        args = tree_map(_as_complex_tensor, args)
        kwargs = tree_map(_as_complex_tensor, kwargs)

        return tree_map(_as_interleaved, func(*args, **kwargs))
````
- **EN**: This chunk defines `__torch_dispatch__`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__torch_dispatch__`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor subclassing**
  - EN: Handles tensor subclasses and fake/symbolic tensor behavior without breaking dispatch.
  - CN: 在不破坏 dispatch 的前提下处理张量子类与 fake/符号张量行为。
- **_P**
  - EN: `_P` is one of the main symbols declared or implemented in this file.
  - CN: `_P` 是本文件声明或实现的主要符号之一。
- **_R**
  - EN: `_R` is one of the main symbols declared or implemented in this file.
  - CN: `_R` 是本文件声明或实现的主要符号之一。
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

- **Internal torch modules / torch 内部模块**: `torch`, `torch._decomp`, `torch._ops`, `torch._refs`, `torch.types`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `.._core`
- **Standard library / 标准库**: `collections.abc`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `_P`, `_R`, `COMPLEX_OPS_TABLE`, `COMPLEX_TO_REAL`, `REAL_TO_COMPLEX`, `PROMOTE_TYPES`, `is_complex_tensor`, `promote_tensors`, `register_complex`, `FORCE_TEST_LIST`
