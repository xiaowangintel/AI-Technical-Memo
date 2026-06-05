# custom_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/custom_op.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `custom_op` and the surrounding SGLang serving stack. / 提供围绕 `custom_op` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import inspect
from typing import Any, Callable, List, Optional, TypeVar, Union, overload

import torch
import torch.library

from sglang.kernel_api_logging import debug_torch_op

F = TypeVar("F", bound=Callable)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `inspect`, `typing`, `torch`, `torch.library`, `sglang.kernel_api_logging`. It also defines symbols such as `F` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `inspect`, `typing`, `torch`, `torch.library`, `sglang.kernel_api_logging`。 同时定义了 `F` 等符号，供后续逻辑使用。

### Lines 14-22: Function `register_custom_op` / 函数 `register_custom_op`
```python
@overload
def register_custom_op(
    fn: F,
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    out_shape: Optional[Union[int, str]] = None,
    eager: bool = True,
) -> F: ...
```
**EN:** This function implements `register_custom_op`.
**CN:** 该函数实现了 `register_custom_op`。

### Lines 25-33: Function `register_custom_op` / 函数 `register_custom_op`
```python
@overload
def register_custom_op(
    fn: F,
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    fake_impl: Optional[Callable],
    eager: bool = True,
) -> F: ...
```
**EN:** This function implements `register_custom_op`.
**CN:** 该函数实现了 `register_custom_op`。

### Lines 36-43: Function `register_custom_op` / 函数 `register_custom_op`
```python
@overload
def register_custom_op(
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    out_shape: Optional[Union[int, str]] = None,
    eager: bool = True,
) -> Callable[[F], F]: ...
```
**EN:** This function implements `register_custom_op`.
**CN:** 该函数实现了 `register_custom_op`。

### Lines 46-53: Function `register_custom_op` / 函数 `register_custom_op`
```python
@overload
def register_custom_op(
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    fake_impl: Optional[Callable],
    eager: bool = True,
) -> Callable[[F], F]: ...
```
**EN:** This function implements `register_custom_op`.
**CN:** 该函数实现了 `register_custom_op`。

### Lines 57-130: Function `register_custom_op` / 函数 `register_custom_op`
```python
def register_custom_op(
    fn: Optional[Callable] = None,
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    eager: bool = True,
    **extra_kwargs,
) -> Any:
    """
    A decorator to register a custom operator.

    Example usage:
    ```python
    # inplace operator, out_shape is None by default
    @register_custom_op(mutates_args=["x"])
    def add_1_(x: torch.Tensor) -> None:
        x.add_(1)

    # operator with output, out_shape indicates the position of output
    @register_custom_op(mutates_args=["x"], out_shape=0)
    def add(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        return x.add_(y)
    ```

    :param fn: The function to be registered as a custom operator.
               If None, return a decorator.
    :type fn: Callable
    :param op_name: The name of the operator. If None, use the function name
# ... omitted for brevity ...
        )
        return wrapper.real_impl if eager else wrapper

    if fn is not None:
        return decorator(fn)
    return decorator
```
**EN:** This function implements `register_custom_op`. It primarily calls `set`, `extra_kwargs.keys`, `CustomOpWrapper`, `decorator` to complete its work. State updates are written into `extra_kwarg_keys`, `expected_kwarg_keys`, `has_out_shape`, `has_fake_impl`, `extra_kwargs`, `wrapper`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `register_custom_op`。 它主要通过调用 `set`, `extra_kwargs.keys`, `CustomOpWrapper`, `decorator` 来完成任务。 状态更新主要写入 `extra_kwarg_keys`, `expected_kwarg_keys`, `has_out_shape`, `has_fake_impl`, `extra_kwargs`, `wrapper`。 实现中使用了条件分支。

### Lines 133-133: Class `CustomOpWrapper` declaration / 类 `CustomOpWrapper` 声明
```python
class CustomOpWrapper:
```
**EN:** This class establishes `CustomOpWrapper` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__call__`, `real_impl`, `fake_impl`.
**CN:** 该类将 `CustomOpWrapper` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__call__`, `real_impl`, `fake_impl` 等方法。

### Lines 134-145: Method `CustomOpWrapper.__init__` / 方法 `CustomOpWrapper.__init__`
```python
    def __init__(
        self,
        op_name: str,
        op_func: Callable,
        mutates_args: List[str],
        **extra_kwargs,
    ):
        self.op_name = op_name
        self.op_func = op_func
        self.mutates_args = mutates_args
        self.extra_kwargs = extra_kwargs
        self._impl: Optional[Callable] = None
```
**EN:** This method implements `__init__` on `CustomOpWrapper`. State updates are written into `self.op_name`, `self.op_func`, `self.mutates_args`, `self.extra_kwargs`, `self._impl`.
**CN:** 该方法（属于 `CustomOpWrapper`）实现了 `__init__`。 状态更新主要写入 `self.op_name`, `self.op_func`, `self.mutates_args`, `self.extra_kwargs`, `self._impl`。

### Lines 147-148: Method `CustomOpWrapper.__call__` / 方法 `CustomOpWrapper.__call__`
```python
    def __call__(self, *args, **kwargs):
        return self.real_impl(*args, **kwargs)
```
**EN:** This method implements `__call__` on `CustomOpWrapper`. It primarily calls `self.real_impl` to complete its work.
**CN:** 该方法（属于 `CustomOpWrapper`）实现了 `__call__`。 它主要通过调用 `self.real_impl` 来完成任务。

### Lines 150-166: Method `CustomOpWrapper.real_impl` / 方法 `CustomOpWrapper.real_impl`
```python
    @property
    def real_impl(self) -> Callable:
        if self._impl is None:
            if not hasattr(torch.ops.sglang, self.op_name):
                from sglang.srt.utils.common import direct_register_custom_op

                # NOTE(dark): if torch compile fail here, mark the decorator as eager
                # lazy registration does not work with torch compile
                direct_register_custom_op(
                    op_name=self.op_name,
                    op_func=self.op_func,
                    mutates_args=self.mutates_args,
                    fake_impl=self.fake_impl,
                )
            self._impl = debug_torch_op(self.op_func, self.op_name)
            assert self._impl is not None
        return self._impl
```
**EN:** This method implements `real_impl` on `CustomOpWrapper`. It primarily calls `debug_torch_op`, `hasattr`, `direct_register_custom_op` to complete its work. State updates are written into `self._impl`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `CustomOpWrapper`）实现了 `real_impl`。 它主要通过调用 `debug_torch_op`, `hasattr`, `direct_register_custom_op` 来完成任务。 状态更新主要写入 `self._impl`。 实现中使用了条件分支。

### Lines 168-194: Method `CustomOpWrapper.fake_impl` / 方法 `CustomOpWrapper.fake_impl`
```python
    @property
    def fake_impl(self) -> Callable:
        if "fake_impl" in self.extra_kwargs:
            return self.extra_kwargs["fake_impl"]
        assert "out_shape" in self.extra_kwargs
        signature = inspect.signature(self.op_func)
        out_shape = self.extra_kwargs["out_shape"]
        # check out_shape in signature

        def fake_impl(*args, **kwargs):
            if out_shape is None:
                return None
            bound = signature.bind(*args, **kwargs)
            bound.apply_defaults()
            try:
                return torch.empty_like(
                    bound.args[out_shape]
                    if isinstance(out_shape, int)
                    else bound.arguments[out_shape]
                )
            except (IndexError, KeyError):
                raise RuntimeError(
                    f"Cannot find output argument at position `{out_shape}` for "
                    f"custom operator `{self.op_name}` with signature `{signature}`."
                )

        return fake_impl
```
**EN:** This method implements `fake_impl` on `CustomOpWrapper`. It primarily calls `inspect.signature`, `signature.bind`, `bound.apply_defaults`, `torch.empty_like`, `RuntimeError`, `isinstance` to complete its work. State updates are written into `signature`, `out_shape`, `bound`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `CustomOpWrapper`）实现了 `fake_impl`。 它主要通过调用 `inspect.signature`, `signature.bind`, `bound.apply_defaults`, `torch.empty_like`, `RuntimeError`, `isinstance` 来完成任务。 状态更新主要写入 `signature`, `out_shape`, `bound`。 实现中使用了条件分支、错误处理。

### Lines 197-337: Function `register_custom_op_from_extern` / 函数 `register_custom_op_from_extern`
```python
def register_custom_op_from_extern(
    fn: Callable,
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    out_shape: Optional[Union[int, str]] = None,
    out_dtype: Optional[torch.dtype] = None,
    fake_impl: Optional[Callable] = None,
    computed_args: Optional[dict] = None,
) -> Callable:
    """Wrap an external library function as a custom op for torch.compile compatibility.

    Use this to wrap functions from external libraries (e.g. flashinfer kernels) that
    perform operations incompatible with torch.compile/dynamo tracing, such as JIT
    compilation, file I/O, or dynamic module loading.

    The wrapped function becomes an opaque node in the compiled graph. Dynamo will
    not trace inside it, avoiding tracing failures. A fake implementation is used
    for shape/dtype propagation during compilation.

    The external function must have type annotations compatible with
    ``torch.library.infer_schema`` (``torch.Tensor``, ``int``, ``float``, ``bool``,
    ``Optional[torch.Tensor]``, etc.).

    This function is idempotent: calling it multiple times with the same ``op_name``
    (or ``fn.__name__``) safely skips re-registration.

    Example usage::
# ... omitted for brevity ...
        op_func=fn,
        mutates_args=mutates_args or [],
        fake_impl=fake_impl,
    )

    return debug_torch_op(fn, name)
```
**EN:** This function implements `register_custom_op_from_extern`. It primarily calls `inspect.signature`, `direct_register_custom_op`, `debug_torch_op`, `original_sig.replace`, `new_sig.bind`, `bound.apply_defaults` to complete its work. State updates are written into `name`, `computed_args`, `fake_sig`, `original_fn`, `original_sig`, `new_params`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `register_custom_op_from_extern`。 它主要通过调用 `inspect.signature`, `direct_register_custom_op`, `debug_torch_op`, `original_sig.replace`, `new_sig.bind`, `bound.apply_defaults` 来完成任务。 状态更新主要写入 `name`, `computed_args`, `fake_sig`, `original_fn`, `original_sig`, `new_params`。 实现中使用了条件分支、迭代逻辑、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `CustomOpWrapper`
- **Functions / 函数**: `register_custom_op`, `register_custom_op`, `register_custom_op`, `register_custom_op`, `register_custom_op`, `register_custom_op_from_extern`, `decorator`, `__init__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.kernel_api_logging`, `sglang.srt.utils.common`
- **External / 外部依赖**: `torch`, `torch.library`
- **Standard library / 标准库**: `__future__`, `inspect`, `typing`
