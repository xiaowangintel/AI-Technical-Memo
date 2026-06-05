# forward_ad.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/autograd/forward_ad.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements automatic-differentiation helpers, gradient APIs, profiling hooks, and graph-facing utilities.
- **Purpose (CN)**: 实现自动求导辅助逻辑、梯度 API、profiling 钩子以及面向计算图的工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
# mypy: allow-untyped-defs
import os
from typing import Any, NamedTuple

import torch

from .grad_mode import _DecoratorContextManager


__all__ = [
    "UnpackedDualTensor",
    "enter_dual_level",
    "exit_dual_level",
    "make_dual",
    "unpack_dual",
    "dual_level",
]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, .grad_mode; standard-library helpers such as os, typing. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、.grad_mode；标准库辅助模块，如 os、typing。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 19-38 / 第 19-38 行
````python
# Global variable used to make the python API simpler to use
_current_level = -1


def enter_dual_level():
    r"""Enter a new forward grad level.

    This level can be used to make and unpack dual Tensors to compute
    forward gradients.

    This function also updates the current level that is used by default
    by the other functions in this API.
    """
    global _current_level
    new_level = torch._C._enter_dual_level()
    if new_level != _current_level + 1:
        raise RuntimeError(
            "Entering a new forward AD level but the current level "
            "is not valid. Make sure you did not modified it directly."
        )
````
- **EN**: This chunk defines `enter_dual_level`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `enter_dual_level`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 39-58 / 第 39-58 行
````python
    _current_level = new_level
    return new_level


def exit_dual_level(*, level=None):
    r"""Exit a forward grad level.

    This function deletes all the gradients associated with this
    level. Only deleting the latest entered level is allowed.

    This function also updates the current level that is used by default
    by the other functions in this API.
    """
    global _current_level
    if level is None:
        level = _current_level
    if level != _current_level:
        raise RuntimeError(
            "Trying to exit a forward AD level that was not the last one "
            "that was created. This is not supported."
````
- **EN**: This chunk defines `exit_dual_level`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `exit_dual_level`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 59-76 / 第 59-76 行
````python
        )
    torch._C._exit_dual_level(level=level)
    _current_level = level - 1


def _maybe_load_decompositions():
    if os.environ.get("PYTORCH_JIT", "1") == "1" and __debug__:
        from torch._decomp import decompositions_for_jvp  # noqa: F401


def make_dual(tensor, tangent, *, level=None):
    r"""Associate a tensor value with its tangent to create a "dual tensor" for forward AD gradient computation.

    The result is a new tensor aliased to :attr:`tensor` with :attr:`tangent` embedded
    as an attribute as-is if it has the same storage layout or copied otherwise.
    The tangent attribute can be recovered with :func:`unpack_dual`.

    This function is backward differentiable.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._decomp. This chunk defines `make_dual`, which implements a focused step in autograd bookkeeping or gradient propagation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._decomp。 这一段定义了 `make_dual`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 78-90 / 第 78-90 行
````python
    Given a function `f` whose jacobian is `J`, it allows one to compute the Jacobian-vector product (`jvp`)
    between `J` and a given vector `v` as follows.

    Example::

        >>> # xdoctest: +SKIP("Undefined variables")
        >>> with dual_level():
        ...     inp = make_dual(x, v)
        ...     out = f(inp)
        ...     y, jvp = unpack_dual(out)

    Please see the `forward-mode AD tutorial <https://pytorch.org/tutorials/intermediate/forward_ad_usage.html>`__
    for detailed steps on how to use this API.
````
- **EN**: This chunk continues `make_dual` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `make_dual`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 92-109 / 第 92-109 行
````python
    """
    # See NOTE: [forward-mode AD decompositions mechanism]
    #
    # Import from torch._decomp import decompositions_for_jvp to register
    # decompositions for jvp to the jit registry
    #
    # FIXME: We specify that __debug__ must be True because
    # if python is run with -OO or -O flags (i.e., __debug__ is False), we encounter the
    # following error:
    #
    # Return value was annotated as having type Tuple[NoneType, NoneType] but is actually of
    # type Tuple[Tensor, Tensor]:
    #   File ".../torch/_decomp/__init__.py", line 1585
    #     else:
    #         buffer = z
    #     return min - torch.log1p(z), buffer
    #     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ <--- HERE
    _maybe_load_decompositions()
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `make_dual` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `make_dual`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 111-128 / 第 111-128 行
````python
    if level is None:
        level = _current_level

    if level < 0:
        raise RuntimeError(
            "Trying to create a dual Tensor for forward AD but no level "
            "exists, make sure to enter_dual_level() first."
        )
    if not (tensor.is_floating_point() or tensor.is_complex()):
        raise ValueError(
            f"Expected primal to be floating point or complex, but got: {tensor.dtype}"
        )
    if not (tangent.is_floating_point() or tangent.is_complex()):
        raise ValueError(
            f"Expected tangent to be floating point or complex, but got: {tangent.dtype}"
        )

    return torch._VF._make_dual(tensor, tangent, level=level)
````
- **EN**: This chunk continues `make_dual` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `make_dual`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 131-148 / 第 131-148 行
````python
class UnpackedDualTensor(NamedTuple):
    r"""Namedtuple returned by :func:`unpack_dual` containing the primal and tangent components of the dual tensor.

    See :func:`unpack_dual` for more details.
    """

    primal: torch.Tensor
    tangent: torch.Tensor | None


def unpack_dual(tensor, *, level=None):
    r"""Unpack a "dual tensor" to get both its Tensor value and its forward AD gradient.

    The result is a namedtuple ``(primal, tangent)`` where ``primal`` is a view of
    :attr:`tensor`'s primal and ``tangent`` is :attr:`tensor`'s tangent as-is.
    Neither of these tensors can be dual tensor of level :attr:`level`.

    This function is backward differentiable.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `UnpackedDualTensor`, which hold the main object-oriented state for this portion of the file. This chunk defines `unpack_dual`, which implements a focused step in autograd bookkeeping or gradient propagation.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `UnpackedDualTensor`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `unpack_dual`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。

### Lines 150-168 / 第 150-168 行
````python
    Example::

        >>> # xdoctest: +SKIP("Undefined variables")
        >>> with dual_level():
        ...     inp = make_dual(x, x_t)
        ...     out = f(inp)
        ...     y, jvp = unpack_dual(out)
        ...     jvp = unpack_dual(out).tangent

    Please see the `forward-mode AD tutorial <https://pytorch.org/tutorials/intermediate/forward_ad_usage.html>`__
    for detailed steps on how to use this API.
    """
    if level is None:
        level = _current_level

    if level < 0:
        return UnpackedDualTensor(tensor, None)

    primal, dual = torch._VF._unpack_dual(tensor, level=level)
````
- **EN**: This chunk continues `UnpackedDualTensor` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `UnpackedDualTensor`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 170-186 / 第 170-186 行
````python
    return UnpackedDualTensor(primal, dual)


class dual_level(_DecoratorContextManager):
    r"""Context-manager for forward AD, where all forward AD computation must occur within the ``dual_level`` context.

    .. Note::

        The ``dual_level`` context appropriately enters and exit the dual level to
        controls the current forward AD level, which is used by default by the other
        functions in this API.

        We currently don't plan to support nested ``dual_level`` contexts, however, so
        only a single forward AD level is supported. To compute higher-order
        forward grads, one can use :func:`torch.func.jvp`.

    Example::
````
- **EN**: It introduces or extends `dual_level`, which hold the main object-oriented state for this portion of the file. This chunk continues `dual_level` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `dual_level`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `dual_level`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 188-205 / 第 188-205 行
````python
        >>> # xdoctest: +SKIP("Undefined variables")
        >>> x = torch.tensor([1])
        >>> x_t = torch.tensor([1])
        >>> with dual_level():
        ...     inp = make_dual(x, x_t)
        ...     # Do computations with inp
        ...     out = your_fn(inp)
        ...     _, grad = unpack_dual(out)
        >>> grad is None
        False
        >>> # After exiting the level, the grad is deleted
        >>> _, grad_after = unpack_dual(out)
        >>> grad is None
        True

    Please see the `forward-mode AD tutorial <https://pytorch.org/tutorials/intermediate/forward_ad_usage.html>`__
    for detailed steps on how to use this API.
    """
````
- **EN**: This chunk continues `dual_level` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `dual_level`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 207-223 / 第 207-223 行
````python
    def __enter__(self):
        return enter_dual_level()

    def __exit__(self, exc_type: Any, exc_value: Any, traceback: Any) -> None:
        exit_dual_level()


# Private helper functions
_is_fwd_grad_enabled = torch._C._is_fwd_grad_enabled


# Private helper function to enable or disable fwd grad.
# If you're a user and want to use this, please file an issue to discuss the use case.
class _set_fwd_grad_enabled(_DecoratorContextManager):
    def __init__(self, mode: bool) -> None:
        self.prev = _is_fwd_grad_enabled()
        torch._C._set_fwd_grad_enabled(mode)
````
- **EN**: It introduces or extends `_set_fwd_grad_enabled`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_set_fwd_grad_enabled`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 225-229 / 第 225-229 行
````python
    def __enter__(self) -> None:
        pass

    def __exit__(self, exc_type: Any, exc_value: Any, traceback: Any) -> None:
        torch._C._set_fwd_grad_enabled(self.prev)
````
- **EN**: This chunk defines `__exit__`, which implements context-manager boundaries around temporary runtime state.
- **CN**: 这一段定义了 `__exit__`，其作用是实现围绕临时运行时状态的上下文管理器边界。

## Key Concepts / 关键概念

- **Autograd engine**
  - EN: Carries saved tensors, gradient metadata, and APIs that connect forward execution to backward logic.
  - CN: 承载保存张量、梯度元数据以及连接前向与反向逻辑的 API。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **enter_dual_level**
  - EN: `enter_dual_level` is one of the main symbols declared or implemented in this file.
  - CN: `enter_dual_level` 是本文件声明或实现的主要符号之一。
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

- **Internal torch modules / torch 内部模块**: `torch`, `.grad_mode`, `torch._decomp`
- **Standard library / 标准库**: `os`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `enter_dual_level`, `exit_dual_level`, `_maybe_load_decompositions`, `make_dual`, `UnpackedDualTensor`, `unpack_dual`, `dual_level`, `_set_fwd_grad_enabled`
