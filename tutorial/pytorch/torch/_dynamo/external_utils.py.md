# external_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/external_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: This module contains utility functions that are explicitly allowed to be called during TorchDynamo compilation.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```python
"""
This module contains utility functions that are explicitly allowed to be called during
TorchDynamo compilation. These functions are carefully vetted to ensure they work
correctly within the TorchDynamo tracing and compilation process.

Key functionality groups:

- Compilation State:
  Functions for checking compilation state (is_compiling)

- Function Wrapping:
  Utilities for wrapping functions (wrap_inline, wrap_numpy) to work with
  TorchDynamo compilation

- Autograd Hooks:
  Functions and classes for handling autograd hooks and backward passes
  (call_hook, FakeBackwardCFunction, etc.)

- Tensor Operations:
  Utility functions for tensor operations and transformations
"""
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 23-47
```python
import functools
import warnings
from collections.abc import Callable
from typing import Any, TYPE_CHECKING, TypeVar
from typing_extensions import deprecated, ParamSpec

import torch
import torch.utils._pytree as pytree


try:
    import numpy as np
except ModuleNotFoundError:
    np = None  # type: ignore[assignment]

_P = ParamSpec("_P")
_R = TypeVar("_R")

if TYPE_CHECKING:
    # TorchScript does not support `@deprecated`
    # This is a workaround to avoid breaking TorchScript
    @deprecated(
        "`torch._dynamo.external_utils.is_compiling` is deprecated. Use `torch.compiler.is_compiling` instead.",
        category=FutureWarning,
    )
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 48-73
```python
    def is_compiling() -> bool:
        return torch.compiler.is_compiling()

else:

    def is_compiling() -> bool:
        """
        Indicates whether we are tracing/compiling with torch.compile() or torch.export().
        """
        # NOTE: With `@torch.compile(backend="eager")`, torch._dynamo.is_compiling() will get traced
        # and return true. torch.compiler.is_compiling() is skipped and will return false.
        return torch.compiler.is_compiling()


def wrap_inline(fn: Callable[_P, _R]) -> Callable[_P, _R]:
    """
    Create an extra frame around fn that is not in skipfiles.
    """

    @functools.wraps(fn)
    def inner(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        return fn(*args, **kwargs)

    return inner
```
- **EN**: These decorators register or transform the following definition so it can trace Python execution into an intermediate graph representation.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将 Python 执行过程跟踪为中间图表示。

### Lines 74-94
```python
def call_hook(
    hook: Callable[..., torch.Tensor | None], *args: Any, **kwargs: Any
) -> torch.Tensor:
    """
    Used by compiled autograd to handle hook returning None.
    """
    result = hook(*args)
    if result is None:
        return args[0]
    elif kwargs.get("hook_type") == "post_acc_grad_hook":
        raise RuntimeError("Tensor post accumulate grad hooks should return None.")
    return result


def wrap_numpy(f: Callable[_P, _R]) -> Callable[_P, _R]:
    r"""Decorator that turns a function from ``np.ndarray``s to ``np.ndarray``s into a function
    from ``torch.Tensor``s to ``torch.Tensor``s.
    """
    if not np:
        return f
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 95-115
```python
    @functools.wraps(f)
    def wrap(*args: _P.args, **kwargs: _P.kwargs) -> pytree.PyTree:
        args, kwargs = pytree.tree_map_only(
            torch.Tensor, lambda x: x.numpy(), (args, kwargs)
        )
        out = f(*args, **kwargs)
        # pyrefly: ignore [missing-attribute]
        return pytree.tree_map_only(np.ndarray, lambda x: torch.as_tensor(x), out)

    return wrap


class FakeBackwardCFunction:
    def __init__(
        self,
        real: torch.autograd.function.BackwardCFunction,
        saved_tensors: list[torch.Tensor],
    ) -> None:
        self.real = real
        self.saved_tensors = saved_tensors
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 116-140
```python
    def __getattr__(self, name: str) -> Any:
        if name == "saved_variables":
            warnings.warn(
                "'saved_variables' is deprecated; use 'saved_tensors'",
                DeprecationWarning,
            )
            return self.saved_tensors

        return getattr(self.real, name)


def call_backward(
    backward_c_function: torch.autograd.function.BackwardCFunction,
    saved_tensors: list[torch.Tensor],
    *args: Any,
) -> torch.Tensor | tuple[torch.Tensor, ...]:
    fake = FakeBackwardCFunction(backward_c_function, saved_tensors)
    grads = fake._forward_cls.backward(fake, *args)  # type: ignore[attr-defined]

    if not isinstance(grads, tuple):
        grads = (grads,)

    return grads
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 141-159
```python
def normalize_as_list(x: Any) -> list[Any]:
    if isinstance(x, tuple):
        return list(x)
    elif isinstance(x, list):
        return x
    return [x]


def untyped_storage_size(x: torch.Tensor) -> int:
    return x.untyped_storage().size()


class FakeCompiledAutogradEngine:
    @staticmethod
    def queue_callback(
        final_callbacks: list[Callable[[], None]], cb: Callable[[], None]
    ) -> None:
        final_callbacks.append(cb)
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 160-182
```python
    @staticmethod
    def exec_final_callbacks(final_callbacks: list[Callable[[], None]]) -> None:
        i = 0
        while i < len(final_callbacks):
            cb = final_callbacks[i]
            cb()
            i += 1
        final_callbacks.clear()

    @staticmethod
    def _exec_final_callbacks_stub() -> None:
        pass


def call_hook_from_backward_state(
    *args: Any, bw_state: Any, hook_name: str, **kwargs: Any
) -> Any:
    return getattr(bw_state, hook_name)(*args, **kwargs)


class _ApplyBackwardHook(torch.autograd.Function):
    """Custom autograd function that applies a hook during backward.
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 183-203
```python
    This is used to implement register_hook on intermediate tensors without
    requiring compiled autograd. The hook function is captured in the context
    and applied during the backward pass.
    """

    @staticmethod
    # pyre-ignore[14]: Inconsistent override is expected for autograd.Function
    def forward(
        ctx: Any, tensor: torch.Tensor, hook_fn: Callable[..., Any]
    ) -> torch.Tensor:  # type: ignore[override]
        ctx.hook_fn = hook_fn
        return tensor.view_as(tensor)

    @staticmethod
    def backward(ctx: Any, grad: torch.Tensor) -> tuple[torch.Tensor, None]:  # type: ignore[override]
        result = ctx.hook_fn(grad)
        if result is None:
            result = grad
        return result, None
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 204-230
```python
def call_module_hooks_from_backward_state(
    _: Any, result: Any, *args: Any, bw_state: Any, hooks_name: str, module_name: str
) -> Any:
    module = getattr(bw_state, module_name)
    hooks = getattr(bw_state, hooks_name)
    for hook in hooks:
        new_result = hook(module, result, *args)
        if new_result is not None:
            result = new_result
    return result


# used for torch._dynamo.disable(recursive=False)
def get_nonrecursive_disable_wrapper(fn: Callable[_P, _R]) -> Callable[_P, _R]:
    # wrap function to get the right error message
    # this function is in external_utils so that convert_frame doesn't skip it.
    @functools.wraps(fn)
    def nonrecursive_disable_wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        if torch.compiler.is_exporting():
            raise RuntimeError(
                "Non-recursive torch.compiler.disable is not supported with torch.export."
            )
        return fn(*args, **kwargs)

    return nonrecursive_disable_wrapper
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 231-252
```python
def wrap_dunder_call_ctx_manager(self: Any, func: Callable[_P, _R]) -> Callable[_P, _R]:
    """
    Apply self as a ctx manager around a call to func
    """

    # NOTE: do not functools.wraps(func) because we don't ever want this frame to be skipped!
    def inner(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        with self:
            return func(*args, **kwargs)

    return inner


# Use only on ints marked dynamic via torch.empty(0, integer)
# Currently only way to mark ints as dynamic: https://github.com/pytorch/pytorch/issues/129623
def unwrap_maybe_dynamic_int(x: torch.Tensor | int) -> int:
    if isinstance(x, torch.Tensor):
        # x.size() is expected to be [0, dynamic_int]
        return x.size(1)
    return x
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 253-279
```python
def call_accumulate_grad(
    variable: torch.Tensor, grad: torch.Tensor, has_post_hooks: bool
) -> None:
    updated_grad = torch._dynamo.compiled_autograd.ops.AccumulateGrad(  # type: ignore[attr-defined]
        [grad], variable, variable.grad, has_post_hooks
    )
    variable.grad = updated_grad[0]


def wrap_inline_with_error_on_graph_break(
    fn: Callable[_P, _R], error_on_graph_break: bool
) -> Callable[_P, _R]:
    # NB: need multiple definitions in order to prevent `fullgraph` from
    # being a freevar of wrapper
    # NOTE: do not functools.wraps(fn) because we don't ever want these wrappers to be skipped!
    if error_on_graph_break:

        def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
            with torch._dynamo.error_on_graph_break(True):
                return fn(*args, **kwargs)

    else:

        def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
            with torch._dynamo.error_on_graph_break(False):
                return fn(*args, **kwargs)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 280-297
```python
    return wrapper


def filter_out_const_values(tup: tuple[Any, ...], masks: list[bool]) -> tuple[Any, ...]:
    """
    masks is a list of bools, where True means the corresponding element in tup
    is a const value. Filter out the const values.
    """
    out = []
    for mask_idx, mask in enumerate(masks):
        if not mask:
            out.append(tup[mask_idx])
    return tuple(out)


def insert_const_values_with_mask(
    tup: tuple[Any, ...], masks: list[bool], values: tuple[Any, ...]
) -> tuple[Any, ...]:
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 298-310
```python
    """
    masks and values are of same length. For indices where the mask is True, use
    the const_values to fill in.
    """
    out = []
    idx = 0
    for mask_idx, mask in enumerate(masks):
        if mask:
            out.append(values[mask_idx])
        else:
            out.append(tup[idx])
            idx += 1
    return tuple(out)
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.utils._pytree`
- **Standard library / 标准库**: `functools`, `warnings`, `collections.abc`, `typing`, `typing_extensions`
- **Third-party packages / 第三方包**: `numpy`
- **Primary symbols / 核心符号**: `_P`, `_R`, `wrap_inline`, `call_hook`, `wrap_numpy`, `FakeBackwardCFunction`, `call_backward`, `normalize_as_list`, `untyped_storage_size`, `FakeCompiledAutogradEngine`, `call_hook_from_backward_state`, `_ApplyBackwardHook`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
