# function.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/autograd/function.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements automatic-differentiation helpers, gradient APIs, profiling hooks, and graph-facing utilities.
- **Purpose (CN)**: 实现自动求导辅助逻辑、梯度 API、profiling 钩子以及面向计算图的工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行
````python
# mypy: allow-untyped-defs
import functools
import inspect
import itertools
import warnings
from collections import OrderedDict
from collections.abc import Callable
from typing import Any, Concatenate, TypeVar
from typing_extensions import deprecated, ParamSpec

import torch
import torch._C as _C
import torch._functorch as _functorch
import torch.utils.hooks as hooks
from torch._C import _functions
from torch._functorch.autograd_function import custom_function_call


__all__ = [
    "FunctionCtx",
    "BackwardCFunction",
    "FunctionMeta",
    "Function",
    "once_differentiable",
    "InplaceFunction",
    "NestedIOFunction",
]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._C, torch._functorch, ...; standard-library helpers such as functools, inspect, itertools, ...; other helper packages such as typing_extensions. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._C、torch._functorch、...；标准库辅助模块，如 functools、inspect、itertools、...；其他辅助包，如 typing_extensions。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 29-50 / 第 29-50 行
````python
# Unique id provider for each class inheriting from Function
# This is incremented in FunctionMeta during class definition
AUTOGRAD_FUNCTION_COUNTER = itertools.count()

_T = TypeVar("_T")
_R = TypeVar("_R")
_P = ParamSpec("_P")


# Formerly known as: _ContextMethodMixin
class FunctionCtx:
    def save_for_backward(self, *tensors: torch.Tensor):
        r"""Save given tensors for a future call to :func:`~Function.backward`.

        ``save_for_backward`` should be called at most once, in either the
        :func:`setup_context` or :func:`forward` methods, and only with tensors.

        All tensors intended to be used in the backward pass should be saved
        with ``save_for_backward`` (as opposed to directly on ``ctx``) to prevent
        incorrect gradients and memory leaks, and enable the application of saved
        tensor hooks. See :class:`torch.autograd.graph.saved_tensors_hooks`.
        See :ref:`extending-autograd` for more details.
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. It introduces or extends `FunctionCtx`, which hold the main object-oriented state for this portion of the file. This chunk defines `save_for_backward`, which serializes or reconstructs state across a Python-visible boundary. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 它引入或扩展了 `FunctionCtx`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `save_for_backward`，其作用是在 Python 可见边界上序列化或重建状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 52-71 / 第 52-71 行
````python
        Note that if intermediary tensors, tensors that are neither inputs
        nor outputs of :func:`forward`, are saved for backward, your custom Function
        may not support double backward.
        Custom Functions that do not support double backward should decorate their
        :func:`backward` method with ``@once_differentiable`` so that performing
        double backward raises an error. If you'd like to support double backward,
        you can either recompute intermediaries based on the inputs during backward
        or return the intermediaries as the outputs of the custom Function. See the
        `double backward tutorial <https://pytorch.org/tutorials/intermediate/custom_function_double_backward_tutorial.html>`_
        for more details.

        In :func:`backward`, saved tensors can be accessed through the :attr:`saved_tensors`
        attribute. Before returning them to the user, a check is made to ensure
        they weren't used in any in-place operation that modified their content.

        Arguments can also be ``None``. This is a no-op.

        See :ref:`extending-autograd` for more details on how to use this method.

        Example::
````
- **EN**: This chunk continues `FunctionCtx` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `FunctionCtx`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 73-99 / 第 73-99 行
````python
            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_AUTOGRAD)
            >>> class Func(Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x: torch.Tensor, y: torch.Tensor, z: int):
            >>>         w = x * z
            >>>         out = x * y + y * z + w * y
            >>>         ctx.save_for_backward(x, y, w, out)
            >>>         ctx.z = z  # z is not a tensor
            >>>         return out
            >>>
            >>>     @staticmethod
            >>>     @once_differentiable
            >>>     def backward(ctx, grad_out):
            >>>         x, y, w, out = ctx.saved_tensors
            >>>         z = ctx.z
            >>>         gx = grad_out * (y + y * z)
            >>>         gy = grad_out * (x + z + w)
            >>>         gz = None
            >>>         return gx, gy, gz
            >>>
            >>> a = torch.tensor(1., requires_grad=True, dtype=torch.double)
            >>> b = torch.tensor(2., requires_grad=True, dtype=torch.double)
            >>> c = 4
            >>> d = Func.apply(a, b, c)

        """
        self.to_save = tensors
````
- **EN**: This chunk continues `FunctionCtx` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `FunctionCtx`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 101-128 / 第 101-128 行
````python
    def save_for_forward(self, *tensors: torch.Tensor):
        r"""Save given tensors for a future call to :func:`~Function.jvp`.

        ``save_for_forward`` should be called at most once, in either the
        :func:`setup_context` or :func:`forward` methods, and all arguments
        should be tensors.

        In :func:`jvp`, saved objects can be accessed through the :attr:`saved_tensors`
        attribute.

        Arguments can also be ``None``. This is a no-op.

        See :ref:`extending-autograd` for more details on how to use this method.

        Example::

            >>> # xdoctest: +SKIP
            >>> class Func(torch.autograd.Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x: torch.Tensor, y: torch.Tensor, z: int):
            >>>         ctx.save_for_backward(x, y)
            >>>         ctx.save_for_forward(x, y)
            >>>         ctx.z = z
            >>>         return x * y * z
            >>>
            >>>     @staticmethod
            >>>     def jvp(ctx, x_t, y_t, _):
            >>>         x, y = ctx.saved_tensors
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `save_for_forward`, which serializes or reconstructs state across a Python-visible boundary. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `save_for_forward`，其作用是在 Python 可见边界上序列化或重建状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-155 / 第 129-155 行
````python
            >>>         z = ctx.z
            >>>         return z * (y * x_t + x * y_t)
            >>>
            >>>     @staticmethod
            >>>     def vjp(ctx, grad_out):
            >>>         x, y = ctx.saved_tensors
            >>>         z = ctx.z
            >>>         return z * grad_out * y, z * grad_out * x, None
            >>>
            >>>     a = torch.tensor(1., requires_grad=True, dtype=torch.double)
            >>>     t = torch.tensor(1., dtype=torch.double)
            >>>     b = torch.tensor(2., requires_grad=True, dtype=torch.double)
            >>>     c = 4
            >>>
            >>>     with fwAD.dual_level():
            >>>         a_dual = fwAD.make_dual(a, t)
            >>>         d = Func.apply(a_dual, b, c)

        """
        for tensor in tensors:
            if not (isinstance(tensor, torch.Tensor) or tensor is None):
                raise AssertionError(
                    "save_for_forward expects all arguments to be tensors; you should "
                    "save non-tensors as attributes on ctx."
                )

        self.saved_for_forward = tensors
````
- **EN**: This chunk continues `save_for_forward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `save_for_forward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 157-184 / 第 157-184 行
````python
    def mark_dirty(self, *args: torch.Tensor):
        r"""Mark given tensors as modified in an in-place operation.

        This should be called at most once, in either the :func:`setup_context`
        or :func:`forward` methods, and all arguments should be inputs.

        Every tensor that's been modified in-place in a call to :func:`forward`
        should be given to this function, to ensure correctness of our checks.
        It doesn't matter whether the function is called before or after
        modification.

        Examples::
            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_AUTOGRAD)
            >>> class Inplace(Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x):
            >>>         x_npy = x.numpy() # x_npy shares storage with x
            >>>         x_npy += 1
            >>>         ctx.mark_dirty(x)
            >>>         return x
            >>>
            >>>     @staticmethod
            >>>     @once_differentiable
            >>>     def backward(ctx, grad_output):
            >>>         return grad_output
            >>>
            >>> a = torch.tensor(1., requires_grad=True, dtype=torch.double).clone()
            >>> b = a * a
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `mark_dirty`, which implements a focused step in autograd bookkeeping or gradient propagation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `mark_dirty`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 185-207 / 第 185-207 行
````python
            >>> Inplace.apply(a)  # This would lead to wrong gradients!
            >>>                   # but the engine would not know unless we mark_dirty
            >>> # xdoctest: +SKIP
            >>> b.backward() # RuntimeError: one of the variables needed for gradient
            >>>              # computation has been modified by an inplace operation

        """
        self.dirty_tensors = args

    @deprecated(
        "`mark_shared_storage` is deprecated. "
        "Tensors with shared storages are automatically tracked. "
        "Note that calls to `set_()` are not tracked",
        category=FutureWarning,
    )
    def mark_shared_storage(self, *pairs):
        pass

    def mark_non_differentiable(self, *args: torch.Tensor):
        r"""Mark outputs as non-differentiable.

        This should be called at most once, in either the :func:`setup_context`
        or :func:`forward` methods, and all arguments should be tensor outputs.
````
- **EN**: This chunk defines `mark_non_differentiable`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `deprecated` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `mark_non_differentiable`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `deprecated` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 209-233 / 第 209-233 行
````python
        This will mark outputs as not requiring gradients, increasing the
        efficiency of backward computation. You still need to accept a gradient
        for each output in :meth:`~Function.backward`, but it's always going to
        be a zero tensor with the same shape as the shape of a corresponding
        output.

        This is used e.g. for indices returned from a sort. See example::
            >>> class Func(Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x):
            >>>         sorted, idx = x.sort()
            >>>         ctx.mark_non_differentiable(idx)
            >>>         ctx.save_for_backward(x, idx)
            >>>         return sorted, idx
            >>>
            >>>     @staticmethod
            >>>     @once_differentiable
            >>>     def backward(ctx, g1, g2):  # still need to accept g2
            >>>         x, idx = ctx.saved_tensors
            >>>         grad_input = torch.zeros_like(x)
            >>>         grad_input.index_add_(0, idx, g1)
            >>>         return grad_input

        """
        self.non_differentiable = args
````
- **EN**: This chunk continues `mark_non_differentiable` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `mark_non_differentiable`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 235-262 / 第 235-262 行
````python
    def set_materialize_grads(self, value: bool):
        r"""Set whether to materialize grad tensors. Default is ``True``.

        This should be called only from either the :func:`setup_context` or
        :func:`forward` methods.

        If ``True``, undefined grad tensors will be expanded to tensors full of zeros
        prior to calling the :func:`backward` and :func:`jvp` methods.

        Example::

            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_AUTOGRAD)
            >>> class SimpleFunc(Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x):
            >>>         return x.clone(), x.clone()
            >>>
            >>>     @staticmethod
            >>>     @once_differentiable
            >>>     def backward(ctx, g1, g2):
            >>>         return g1 + g2  # No check for None necessary
            >>>
            >>> # We modify SimpleFunc to handle non-materialized grad outputs
            >>> class Func(Function):
            >>>     @staticmethod
            >>>     def forward(ctx, x):
            >>>         ctx.set_materialize_grads(False)
            >>>         ctx.save_for_backward(x)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `set_materialize_grads`, which mutates configuration or backend state that affects later execution. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `set_materialize_grads`，其作用是修改会影响后续执行的配置或后端状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 263-284 / 第 263-284 行
````python
            >>>         return x.clone(), x.clone()
            >>>
            >>>     @staticmethod
            >>>     @once_differentiable
            >>>     def backward(ctx, g1, g2):
            >>>         x, = ctx.saved_tensors
            >>>         grad_input = torch.zeros_like(x)
            >>>         if g1 is not None:  # We must check for None now
            >>>             grad_input += g1
            >>>         if g2 is not None:
            >>>             grad_input += g2
            >>>         return grad_input
            >>>
            >>> a = torch.tensor(1., requires_grad=True)
            >>> b, _ = Func.apply(a)  # induces g2 to be undefined

        """
        self.materialize_grads = value


# DO NOT USE: This is only defined to be able to load old serialized models
_ContextMethodMixin = FunctionCtx
````
- **EN**: This chunk continues `set_materialize_grads` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `set_materialize_grads`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 287-311 / 第 287-311 行
````python
class _HookMixin:
    @staticmethod
    def _register_hook(backward_hooks, hook):
        if backward_hooks is None:
            backward_hooks = OrderedDict()
        handle = hooks.RemovableHandle(backward_hooks)
        backward_hooks[handle.id] = hook
        return backward_hooks, handle


class BackwardCFunction(_C._FunctionBase, FunctionCtx, _HookMixin):
    r"""
    This class is used for internal autograd work. Do not use.
    """

    def _get_user_fn(self):
        backward_fn = self._forward_cls.backward  # type: ignore[attr-defined]
        vjp_fn = self._forward_cls.vjp  # type: ignore[attr-defined]
        if backward_fn is not Function.backward and vjp_fn is not Function.vjp:
            raise RuntimeError(
                "Implementing both 'backward' and 'vjp' for a custom "
                "Function is not allowed. You should only implement one "
                "of them."
            )
        return vjp_fn if vjp_fn is not Function.vjp else backward_fn
````
- **EN**: It introduces or extends `_HookMixin`, `BackwardCFunction`, which hold the main object-oriented state for this portion of the file. This chunk defines `_get_user_fn`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `_HookMixin`、`BackwardCFunction`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_get_user_fn`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 313-333 / 第 313-333 行
````python
    def apply(self, *args):
        r"""
        Apply method used when executing this Node during the backward.

        Called by the autograd engine (non-boxed path) and by direct
        grad_fn.apply() calls. When boxed_grads_call is True, boxes
        grads into a mutable list before calling user's backward.
        """
        user_fn = self._get_user_fn()
        fwd_cls = self._forward_cls  # type: ignore[attr-defined]  # pyrefly: ignore[missing-attribute]
        if getattr(fwd_cls, "boxed_grads_call", False):
            args = (list(args),)
        return user_fn(self, *args)

    def apply_boxed(self, *args):
        r"""
        Apply method called by the autograd engine when boxed_grads_call
        is True. Grads arrive as a single mutable list argument, allowing
        backward to free individual grads mid-execution.
        """
        return self._get_user_fn()(self, *args)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `apply_boxed`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `apply_boxed`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 335-360 / 第 335-360 行
````python
    def apply_jvp(self, *args):
        r"""
        Apply method used when executing forward mode AD during the forward
        """
        # _forward_cls is defined by derived class
        return self._forward_cls.jvp(self, *args)  # type: ignore[attr-defined]

    def _compiled_autograd_key(self):
        return self._forward_cls._compiled_autograd_key(self)  # type: ignore[attr-defined]


class FunctionMeta(type):
    """Function metaclass.

    This metaclass sets up the following properties:
        _backward_cls: The Function class corresponding to the differentiated
            version of this function (which is generated on the fly by this
            metaclass).
    """

    def __init__(cls, name, bases, attrs):
        backward_fn = type(
            name + "Backward", (BackwardCFunction,), {"_forward_cls": cls}
        )
        backward_fn._autograd_function_id = next(AUTOGRAD_FUNCTION_COUNTER)  # type: ignore[attr-defined]
        cls._backward_cls = backward_fn
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `FunctionMeta`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `FunctionMeta`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 362-385 / 第 362-385 行
````python
        super().__init__(name, bases, attrs)


class _SingleLevelFunction(
    _C._FunctionBase, FunctionCtx, _HookMixin, metaclass=FunctionMeta
):
    @staticmethod
    def forward(*args: Any, **kwargs: Any) -> Any:
        r"""Define the forward of the custom autograd Function.

        This function is to be overridden by all subclasses.
        There are two ways to define forward:

        Usage 1 (Combined forward and ctx)::

            @staticmethod
            def forward(ctx: Any, *args: Any, **kwargs: Any) -> Any:
                pass

        - It must accept a context ctx as the first argument, followed by any
          number of arguments (tensors or other types).
        - See :ref:`combining-forward-context` for more details

        Usage 2 (Separate forward and ctx)::
````
- **EN**: It introduces or extends `_SingleLevelFunction`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `_SingleLevelFunction`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 387-413 / 第 387-413 行
````python
            @staticmethod
            def forward(*args: Any, **kwargs: Any) -> Any:
                pass


            @staticmethod
            def setup_context(ctx: Any, inputs: Tuple[Any, ...], output: Any) -> None:
                pass

        - The forward no longer accepts a ctx argument.
        - Instead, you must also override the :meth:`torch.autograd.Function.setup_context`
          staticmethod to handle setting up the ``ctx`` object.
          ``output`` is the output of the forward, ``inputs`` are a Tuple of inputs
          to the forward.
        - See :ref:`extending-autograd` for more details

        The context can be used to store arbitrary data that can be then
        retrieved during the backward pass. Tensors should not be stored
        directly on `ctx` (though this is not currently enforced for
        backward compatibility). Instead, tensors should be saved either with
        :func:`ctx.save_for_backward` if they are intended to be used in
        ``backward`` (equivalently, ``vjp``) or :func:`ctx.save_for_forward`
        if they are intended to be used for in ``jvp``.
        """
        raise NotImplementedError(
            "You must implement the forward function for custom autograd.Function."
        )
````
- **EN**: This chunk defines `setup_context`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `setup_context`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 415-437 / 第 415-437 行
````python
    @staticmethod
    def setup_context(ctx: Any, inputs: tuple[Any, ...], output: Any) -> Any:
        r"""There are two ways to define the forward pass of an autograd.Function.

        Either:

        1. Override forward with the signature ``forward(ctx, *args, **kwargs)``.
           ``setup_context`` is not overridden. Setting up the ctx for backward
           happens inside the ``forward``.
        2. Override forward with the signature ``forward(*args, **kwargs)`` and
           override ``setup_context``. Setting up the ctx for backward happens
           inside ``setup_context`` (as opposed to inside the ``forward``)

        See :meth:`torch.autograd.Function.forward` and :ref:`extending-autograd` for more details.
        """
        raise NotImplementedError("setup_context is not implemented.")

    @staticmethod
    def backward(ctx: Any, *grad_outputs: Any) -> Any:
        r"""Define a formula for differentiating the operation with backward mode automatic differentiation.

        This function is to be overridden by all subclasses.
        (Defining this function is equivalent to defining the ``vjp`` function.)
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 439-462 / 第 439-462 行
````python
        It must accept a context :attr:`ctx` as the first argument, followed by
        as many outputs as the :func:`forward` returned (None will be passed in
        for non tensor outputs of the forward function),
        and it should return as many tensors, as there were inputs to
        :func:`forward`. Each argument is the gradient w.r.t the given output,
        and each returned value should be the gradient w.r.t. the
        corresponding input. If an input is not a Tensor or is a Tensor not
        requiring grads, you can just pass None as a gradient for that input.

        The context can be used to retrieve tensors saved during the forward
        pass. It also has an attribute :attr:`ctx.needs_input_grad` as a tuple
        of booleans representing whether each input needs gradient. E.g.,
        :func:`backward` will have ``ctx.needs_input_grad[0] = True`` if the
        first input to :func:`forward` needs gradient computed w.r.t. the
        output.
        """
        raise NotImplementedError(
            "You must implement either the backward or vjp method for "
            "your custom autograd.Function to use it with backward "
            "mode AD."
        )

    # vjp and backward are alias of each other
    vjp = backward
````
- **EN**: This chunk continues `backward` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `backward`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 464-488 / 第 464-488 行
````python
    """
    Bool that specifies if PyTorch should clear saved tensors after the first
    access to ``ctx.saved_tensors``. When set to True, accessing saved_tensors
    clears the internal references, allowing the tensors to be cleared as soon
    as the Tensor returned by saved_tensors is deleted.

    This is useful for reducing memory pressure in backward passes when you
    only need to access saved tensors once.

    Default is False.
    """
    clear_saved_tensors_on_access = False

    """
    Bool that specifies if backward should receive grads as a single mutable
    list argument instead of individual args in an immutable tuple. This allows
    backward to free individual grads mid-execution by removing them from the
    list, reducing peak memory.

    When True, ``backward(ctx, grads)`` receives a single list instead of
    ``backward(ctx, *grads)``.

    Default is False.
    """
    boxed_grads_call = False
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `backward` and expands its internal control flow or state updates.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `backward`，进一步展开其内部控制流或状态更新。

### Lines 490-515 / 第 490-515 行
````python
    @staticmethod
    def jvp(ctx: Any, *grad_inputs: Any) -> Any:
        r"""Define a formula for differentiating the operation with forward mode automatic differentiation.

        This function is to be overridden by all subclasses.
        It must accept a context :attr:`ctx` as the first argument, followed by
        as many inputs as the :func:`forward` got (None will be passed in
        for non tensor inputs of the forward function),
        and it should return as many tensors as there were outputs to
        :func:`forward`. Each argument is the gradient w.r.t the given input,
        and each returned value should be the gradient w.r.t. the
        corresponding output. If an output is not a Tensor or the function is not
        differentiable with respect to that output, you can just pass None as a
        gradient for that input.

        You can use the :attr:`ctx` object to pass any value from the forward to this
        functions.
        """
        raise NotImplementedError(
            "You must implement the jvp function for custom "
            "autograd.Function to use it with forward mode AD."
        )


class Function(_SingleLevelFunction):
    r"""Base class to create custom `autograd.Function`.
````
- **EN**: It introduces or extends `Function`, which hold the main object-oriented state for this portion of the file. This chunk defines `jvp`, which implements differentiation-time behavior that complements the forward path. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 它引入或扩展了 `Function`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `jvp`，其作用是实现与前向路径配套的求导期行为。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 517-544 / 第 517-544 行
````python
    To create a custom `autograd.Function`, subclass this class and implement
    the :meth:`forward` and :meth:`backward` static methods. Then, to use your custom
    op in the forward pass, call the class method ``apply``. Do not call
    :meth:`forward` directly.

    To ensure correctness and best performance, make sure you are calling the
    correct methods on ``ctx`` and validating your backward function using
    :func:`torch.autograd.gradcheck`.

    See :ref:`extending-autograd` for more details on how to use this class.

    Examples::

        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_AUTOGRAD)
        >>> class Exp(Function):
        >>>     @staticmethod
        >>>     def forward(ctx, i):
        >>>         result = i.exp()
        >>>         ctx.save_for_backward(result)
        >>>         return result
        >>>
        >>>     @staticmethod
        >>>     def backward(ctx, grad_output):
        >>>         result, = ctx.saved_tensors
        >>>         return grad_output * result
        >>>
        >>> # Use it by calling the apply method:
        >>> # xdoctest: +SKIP
````
- **EN**: This chunk continues `Function` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `Function`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 545-570 / 第 545-570 行
````python
        >>> output = Exp.apply(input)
    """

    def __init__(self, *args, **kwargs):
        warnings.warn(
            f"{self.__class__} should not be instantiated. Methods on autograd functions "
            "are all static, so you should invoke them on the class itself. "
            "Instantiating an autograd function will raise an "
            "error in a future version of PyTorch.",
            DeprecationWarning,
            stacklevel=2,
        )

    def __call__(self, *args, **kwargs):
        raise RuntimeError(
            "Legacy autograd function with non-static forward method is deprecated. "
            "Please use new-style autograd function with static forward method. "
            "(Example: https://pytorch.org/docs/stable/autograd.html#torch.autograd.Function)"
        )

    """
    Bool that specifies if PyTorch should attempt to autogenerate
    :func:`torch.vmap` support for this autograd.Function. You may set this to
    True only if this autograd.Function's forward, backward, and jvp (if they
    exist) are written using PyTorch operations; otherwise, please override
    :meth:`torch.autograd.Function.vmap` to add support for :func:`torch.vmap`.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `__call__`, which implements a focused step in autograd bookkeeping or gradient propagation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `__call__`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 572-595 / 第 572-595 行
````python
    Please see :ref:`func-autograd-function` for more details.
    """
    generate_vmap_rule = False

    @staticmethod
    def vmap(info, in_dims, *args):
        r"""Define the behavior for this autograd.Function underneath :func:`torch.vmap`.

        For a :func:`torch.autograd.Function` to support
        :func:`torch.vmap`, you must either override this static method, or set
        ``generate_vmap_rule`` to ``True`` (you may not do both).

        If you choose to override this staticmethod: it must accept

        - an ``info`` object as the first argument. ``info.batch_size``
          specifies the size of the dimension being vmapped over,
          while ``info.randomness`` is the randomness option passed to
          :func:`torch.vmap`.
        - an ``in_dims`` tuple as the second argument.
          For each arg in ``args``, ``in_dims`` has a corresponding
          ``Optional[int]``. It is ``None`` if the arg is not a Tensor or if
          the arg is not being vmapped over, otherwise, it is an integer
          specifying what dimension of the Tensor is being vmapped over.
        - ``*args``, which is the same as the args to :meth:`~Function.forward`.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `vmap`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `vmap`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 597-620 / 第 597-620 行
````python
        The return of the vmap staticmethod is a tuple of ``(output, out_dims)``.
        Similar to ``in_dims``, ``out_dims`` should be of the same structure as
        ``output`` and contain one ``out_dim`` per output that specifies if the
        output has the vmapped dimension and what index it is in.

        Please see :ref:`func-autograd-function` for more details.
        """
        raise NotImplementedError(
            "To use autograd.Function with vmap, you must either override the "
            "vmap staticmethod or set generate_vmap_rule=True."
        )

    @classmethod
    def apply(cls, *args, **kwargs):
        def bind_default_args(func, *args, **kwargs):
            signature = inspect.signature(func)
            bound_args = signature.bind(*args, **kwargs)
            bound_args.apply_defaults()

            return bound_args.args

        is_setup_ctx_defined = _is_setup_context_defined(cls.setup_context)
        if is_setup_ctx_defined:
            args = bind_default_args(cls.forward, *args, **kwargs)
````
- **EN**: This chunk defines `bind_default_args`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `bind_default_args`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 622-643 / 第 622-643 行
````python
        if not torch._C._are_functorch_transforms_active():
            # See NOTE: [functorch vjp and autograd interaction]
            args = _functorch.utils.unwrap_dead_wrappers(args)
            return super().apply(*args, **kwargs)  # type: ignore[misc]

        if not is_setup_ctx_defined:
            raise RuntimeError(
                "In order to use an autograd.Function with functorch transforms "
                "(vmap, grad, jvp, jacrev, ...), it must override the setup_context "
                "staticmethod. For more details, please see "
                "https://pytorch.org/docs/main/notes/extending.func.html"
            )

        return custom_function_call(cls, *args, **kwargs)

    @staticmethod
    def _compiled_autograd_key(ctx):
        return (ctx._autograd_function_id,)


def _is_setup_context_defined(fn):
    return fn != _SingleLevelFunction.setup_context
````
- **EN**: This chunk defines `_is_setup_context_defined`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `staticmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_is_setup_context_defined`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `staticmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 646-670 / 第 646-670 行
````python
def once_differentiable(
    fn: Callable[Concatenate[_T, _P], _R],
) -> Callable[Concatenate[_T, _P], _R]:
    @functools.wraps(fn)
    def wrapper(ctx: _T, *args: _P.args, **kwargs: _P.kwargs) -> _R:
        with torch.no_grad():
            outputs = fn(ctx, *args, **kwargs)

        if not torch.is_grad_enabled():
            return outputs

        # If any of the inputs have requires_grad=True, we force the outputs
        # to have requires_grad=True but point to a grad_fn which throws an
        # error message during (double) back-propagation.
        # XXX: this is only an approximation of requires_grad - there's no way
        # to figure out if fn didn't use ctx.saved_tensors and as a result
        # some Tensors might require grad, even if no args do.
        # Unfortunately, this leads to unexpected error messages ("no nodes
        # require computing gradients"), but I don't have a better idea.
        # These functions would raise an error in backward anyway.
        requires_grad = any(
            isinstance(arg, torch.Tensor) and arg.requires_grad for arg in args
        )
        if not requires_grad:
            return outputs
````
- **EN**: This chunk defines `wrapper`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `functools.wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wrapper`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `functools.wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 672-694 / 第 672-694 行
````python
        if not isinstance(outputs, tuple):
            outputs_ = (outputs,)
        else:
            outputs_ = outputs

        err_fn = _functions.DelayedError(
            b"trying to differentiate twice a function that was marked "
            b"with @once_differentiable",
            len(outputs_),
        )

        # Create aliases of each output that has requires_grad=True. We need
        # at least one of the inputs to err_fn to require grad so that the
        # output will have a grad_fn.
        def fake_requires_grad(var):
            if var is not None:
                var = var.detach()
                var.requires_grad = True
            return var

        return err_fn(*[fake_requires_grad(v) for v in outputs_])  # type: ignore[return-value]

    return wrapper
````
- **EN**: This chunk defines `fake_requires_grad`, which implements a focused step in autograd bookkeeping or gradient propagation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `fake_requires_grad`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 697-724 / 第 697-724 行
````python
class InplaceFunction(Function):
    r"""
    This class is here only for backward compatibility reasons.
    Use :class:`Function` instead of this for any new use case.
    """

    def __init__(self, inplace=False):
        super().__init__()
        self.inplace = inplace


def _nested_map(condition, fn, condition_msg=None):
    def _map(obj):
        if condition(obj):
            return fn(obj)
        elif obj is None:
            return None
        elif isinstance(obj, (list, tuple)):
            mapped = (_map(x) for x in obj)
            if hasattr(obj, "_fields"):
                # obj is namedtuple
                return type(obj)(*mapped)
            return type(obj)(mapped)
        elif isinstance(obj, dict):
            return {x: _map(obj[x]) for x in obj}
        else:
            raise ValueError(
                "Auto nesting doesn't know how to process "
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `InplaceFunction`, which hold the main object-oriented state for this portion of the file. This chunk defines `_map`, which implements a focused step in autograd bookkeeping or gradient propagation. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `InplaceFunction`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_map`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 725-752 / 第 725-752 行
````python
                "an input object of type "
                + torch.typename(obj)
                + (
                    ". Accepted types: " + condition_msg + ", or lists/tuples of them"
                    if condition_msg
                    else ""
                )
            )

    return _map


def _jit_unwrap_structured(obj):
    if hasattr(obj, "_jit_unwrap"):
        return obj._jit_unwrap()
    return obj


def _iter_filter(condition, allow_unknown=False, condition_msg=None, conversion=None):
    def _iter(obj):
        if conversion is not None:
            obj = conversion(obj)
        if condition(obj):
            yield obj
        elif obj is None:
            return
        elif isinstance(obj, (list, tuple)):
            for o in obj:
````
- **EN**: This chunk defines `_iter`, which implements a focused step in autograd bookkeeping or gradient propagation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_iter`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 753-772 / 第 753-772 行
````python
                yield from _iter(o)
        elif isinstance(obj, dict):
            # We only accept primitive key types, so we needn't inspect them
            for o in obj.values():
                yield from _iter(o)
        elif allow_unknown:
            yield obj
        else:
            raise ValueError(
                "Auto nesting doesn't know how to process "
                "an input object of type "
                + torch.typename(obj)
                + (
                    ". Accepted types: " + condition_msg + ", or lists/tuples of them"
                    if condition_msg
                    else ""
                )
            )

    return _iter
````
- **EN**: This chunk continues `_iter` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_iter`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 775-792 / 第 775-792 行
````python
def _unflatten(input, proto):
    # unflatten a list or tuple input into a nested list/tuple structure
    # specified by proto
    def unflatten_helper(input, proto):
        res: list[torch.Tensor | None] = []
        if hasattr(proto, "_jit_wrap"):
            return proto._jit_wrap(input)
        if not isinstance(proto, (list, tuple)):
            return input[0], input[1:]
        for e in proto:
            if e is None:
                res.append(e)
            else:
                res_e, input = unflatten_helper(input, e)
                res.append(res_e)
        return type(proto)(res), input

    return unflatten_helper(input, proto)[0]
````
- **EN**: This chunk defines `unflatten_helper`, which implements a focused step in autograd bookkeeping or gradient propagation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `unflatten_helper`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 795-821 / 第 795-821 行
````python
_iter_jit_values = _iter_filter(
    lambda o: o is None or isinstance(o, torch._C.Value),
    condition_msg="jit's Values or None",
)
_iter_tensors = _iter_filter(
    lambda x: isinstance(x, torch.Tensor),
    condition_msg="Tensors",
    conversion=_jit_unwrap_structured,
)
_iter_tensors_permissive = _iter_filter(
    lambda x: isinstance(x, torch.Tensor),
    allow_unknown=True,
    condition_msg="Tensors (permissive)",
)
_iter_None_tensors = _iter_filter(
    lambda o: o is None or isinstance(o, torch.Tensor), condition_msg="Tensors or None"
)
_map_tensor_data = _nested_map(
    lambda x: isinstance(x, torch.Tensor), lambda o: o.data, condition_msg="Tensors"
)


class NestedIOFunction(Function):
    r"""
    This class is here only for backward compatibility reasons.
    Use :class:`Function` instead of this for any new use case.
    """
````
- **EN**: It introduces or extends `NestedIOFunction`, which hold the main object-oriented state for this portion of the file.
- **CN**: 它引入或扩展了 `NestedIOFunction`，这些类承载了本段涉及的主要面向对象状态。

### Lines 823-849 / 第 823-849 行
````python
    # The 'type: ignore' statements are needed here because these functions are declared as '@staticmethod' in the
    # superclass (Function) but are instance methods here, which mypy reports as incompatible.

    def _do_forward(self, *input):
        self._nested_input = input
        flat_input = tuple(_iter_tensors(input))
        flat_output = super()._do_forward(*flat_input)  # type: ignore[misc]
        nested_tensors = _unflatten(flat_output, self._nested_output)
        return nested_tensors

    def _do_backward(self, gradients, retain_variables):
        self.retain_variables = retain_variables
        result = super()._do_backward(gradients, retain_variables)  # type: ignore[misc]
        if not retain_variables:
            del self._nested_output
            del self._to_save_nested
        return result

    def backward(self, *gradients: Any) -> Any:  # type: ignore[override]
        r"""
        Shared backward utility.
        """
        nested_gradients = _unflatten(gradients, self._nested_output)
        result = self.backward_extended(*nested_gradients)  # type: ignore[func-returns-value]
        return tuple(_iter_None_tensors(result))

    __call__ = _do_forward
````
- **EN**: This chunk defines `backward`, which implements differentiation-time behavior that complements the forward path. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `backward`，其作用是实现与前向路径配套的求导期行为。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 851-874 / 第 851-874 行
````python
    def forward(self, *args: Any) -> Any:  # type: ignore[override]
        r"""
        Shared forward utility.
        """
        nested_tensors = _map_tensor_data(self._nested_input)
        result = self.forward_extended(*nested_tensors)  # type: ignore[func-returns-value]
        del self._nested_input
        self._nested_output = result
        return tuple(_iter_tensors(result))

    def save_for_backward(self, *args: Any) -> None:
        r"""
        See :meth:`Function.save_for_backward`.
        """
        self.to_save = tuple(_iter_tensors(args))
        self._to_save_nested = args

    @property
    def saved_tensors(self):  # type: ignore[override]
        r"""
        See :meth:`Function.saved_tensors`.
        """
        flat_tensors = super().saved_tensors  # type: ignore[misc]
        return _unflatten(flat_tensors, self._to_save_nested)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `saved_tensors`, which serializes or reconstructs state across a Python-visible boundary. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `saved_tensors`，其作用是在 Python 可见边界上序列化或重建状态。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 876-898 / 第 876-898 行
````python
    def mark_dirty(self, *args: Any, **kwargs: Any) -> None:
        r"""
        See :meth:`Function.mark_dirty`.
        """
        self.dirty_tensors = tuple(_iter_tensors((args, kwargs)))

    def mark_non_differentiable(self, *args: Any, **kwargs: Any) -> None:
        r"""
        See :meth:`Function.mark_non_differentiable`.
        """
        self.non_differentiable = tuple(_iter_tensors((args, kwargs)))

    def forward_extended(self, *input: Any) -> None:
        r"""
        User defined forward.
        """
        raise NotImplementedError

    def backward_extended(self, *grad_output: Any) -> None:
        r"""
        User defined backward.
        """
        raise NotImplementedError
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `backward_extended`, which implements differentiation-time behavior that complements the forward path. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `backward_extended`，其作用是实现与前向路径配套的求导期行为。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

## Key Concepts / 关键概念

- **Autograd engine**
  - EN: Carries saved tensors, gradient metadata, and APIs that connect forward execution to backward logic.
  - CN: 承载保存张量、梯度元数据以及连接前向与反向逻辑的 API。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **AUTOGRAD_FUNCTION_COUNTER**
  - EN: `AUTOGRAD_FUNCTION_COUNTER` is one of the main symbols declared or implemented in this file.
  - CN: `AUTOGRAD_FUNCTION_COUNTER` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._C`, `torch._functorch`, `torch.utils.hooks`, `torch._functorch.autograd_function`
- **Standard library / 标准库**: `functools`, `inspect`, `itertools`, `warnings`, `collections`, `collections.abc`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `AUTOGRAD_FUNCTION_COUNTER`, `_T`, `_R`, `_P`, `FunctionCtx`, `_HookMixin`, `BackwardCFunction`, `FunctionMeta`, `_SingleLevelFunction`
