# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/futures/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
````python
# mypy: allow-untyped-defs
# pylint: disable=useless-parent-delegation
from __future__ import annotations

from typing import cast, Generic, TYPE_CHECKING, TypeVar

import torch


if TYPE_CHECKING:
    from collections.abc import Callable


__all__ = ["Future", "collect_all", "wait_all"]


T = TypeVar("T")
S = TypeVar("S")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as __future__, typing, collections.abc. The future import postpones annotation evaluation, keeping type hints lightweight at import time. `__all__` defines the public symbols that this module chooses to export. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 __future__、typing、collections.abc。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 `__all__` 定义了本模块选择导出的公共符号。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 21-37 / 第 21-37 行
````python
class Future(torch._C.Future, Generic[T]):
    r"""
    Wrapper around a ``torch._C.Future`` which encapsulates an asynchronous
    execution of a callable, e.g. :meth:`~torch.distributed.rpc.rpc_async`. It
    also exposes a set of APIs to add callback functions and set results.

    .. warning:: GPU support is a beta feature, subject to changes.
    """

    def __init__(self, *, devices: list[int | str | torch.device] | None = None):
        r"""
        Create an empty unset ``Future``. If the future is intended to hold
        values containing CUDA tensors, (a superset of) their CUDA devices must
        be specified at construction. (This is only supported if
        ``torch.cuda.is_available()`` returns ``True``). This is needed to
        ensure proper CUDA stream synchronization. The child futures, returned
        by the ``then`` method, will inherit these devices.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `Future`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `Future`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 39-51 / 第 39-51 行
````python
        Args:
            devices(``List[Union[int, str, torch.device]]``, optional): the set
                of devices on which tensors contained in this future's value are
                allowed to reside and on which callbacks are allowed to operate.
        """
        if devices is None:
            devices = []
        super().__init__([torch.device(d) for d in devices])

    def done(self) -> bool:
        r"""
        Return ``True`` if this ``Future`` is done. A ``Future`` is done if it
        has a result or an exception.
````
- **EN**: This chunk defines `done`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `done`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 53-72 / 第 53-72 行
````python
        If the value contains tensors that reside on GPUs, ``Future.done()``
        will return ``True`` even if the asynchronous kernels that are
        populating those tensors haven't yet completed running on the device,
        because at such stage the result is already usable, provided one
        performs the appropriate synchronizations (see :meth:`wait`).
        """
        return super().done()

    def wait(self) -> T:
        r"""
        Block until the value of this ``Future`` is ready.

        If the value contains tensors that reside on GPUs, then an additional
        synchronization is performed with the kernels (executing on the device)
        which may be asynchronously populating those tensors. Such sync is
        non-blocking, which means that ``wait()`` will insert the necessary
        instructions in the current streams to ensure that further operations
        enqueued on those streams will be properly scheduled after the async
        kernels but, once that is done, ``wait()`` will return, even if those
        kernels are still running. No further synchronization is required when
````
- **EN**: This chunk defines `wait`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wait`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 73-89 / 第 73-89 行
````python
        accessing and using the values, as long as one doesn't change streams.

        Returns:
            The value held by this ``Future``. If the function (callback or RPC)
            creating the value has thrown an error, this ``wait`` method will
            also throw an error.
        """
        return super().wait()

    def value(self) -> T:
        r"""
        Obtain the value of an already-completed future.

        This method should only be called after a call to :meth:`wait` has
        completed, or inside a callback function passed to :meth:`then`. In
        other cases this ``Future`` may not yet hold a value and calling
        ``value()`` could fail.
````
- **EN**: This chunk defines `value`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 这一段定义了 `value`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 91-110 / 第 91-110 行
````python
        If the value contains tensors that reside on GPUs, then this method will
        *not* perform any additional synchronization. This should be done
        beforehand, separately, through a call to :meth:`wait` (except within
        callbacks, for which it's already being taken care of by :meth:`then`).

        Returns:
            The value held by this ``Future``. If the function (callback or RPC)
            creating the value has thrown an error, this ``value()`` method will
            also throw an error.
        """
        return super().value()

    def then(self, callback: Callable[[Future[T]], S]) -> Future[S]:
        r"""
        Append the given callback function to this ``Future``, which will be run
        when the ``Future`` is completed.  Multiple callbacks can be added to
        the same ``Future``, but the order in which they will be executed cannot
        be guaranteed (to enforce a certain order consider chaining:
        ``fut.then(cb1).then(cb2)``). The callback must take one argument, which
        is the reference to this ``Future``. The callback function can use the
````
- **EN**: This chunk defines `then`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `then`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 111-123 / 第 111-123 行
````python
        :meth:`value` method to get the value. Note that if this ``Future`` is
        already completed, the given callback will be run immediately inline.

        If the ``Future``'s value contains tensors that reside on GPUs, the
        callback might be invoked while the async kernels that are populating
        those tensors haven't yet finished executing on the device. However, the
        callback will be invoked with some dedicated streams set as current
        (fetched from a global pool) which will be synchronized with those
        kernels. Hence any operation performed by the callback on these tensors
        will be scheduled on the device after the kernels complete. In other
        words, as long as the callback doesn't switch streams, it can safely
        manipulate the result without any additional synchronization. This is
        similar to the non-blocking behavior of :meth:`wait`.
````
- **EN**: This chunk continues `then` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `then`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 125-139 / 第 125-139 行
````python
        Similarly, if the callback returns a value that contains tensors that
        reside on a GPU, it can do so even if the kernels that are producing
        these tensors are still running on the device, as long as the callback
        didn't change streams during its execution. If one wants to change
        streams, one must be careful to re-synchronize them with the original
        streams, that is, those that were current when the callback was invoked.

        Args:
            callback(``Callable``): a ``Callable`` that takes this ``Future`` as
                                    the only argument.

        Returns:
            A new ``Future`` object that holds the return value of the
            ``callback`` and will be marked as completed when the given
            ``callback`` finishes.
````
- **EN**: This chunk continues `then` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `then`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 141-160 / 第 141-160 行
````python
        .. note:: Note that if the callback function throws, either
            through the original future being completed with an exception and
            calling ``fut.wait()``, or through other code in the callback, the
            future returned by ``then`` will be marked appropriately with the
            encountered error. However, if this callback later completes
            additional futures, those futures are not marked as completed with
            an error and the user is responsible for handling completion/waiting
            on those futures independently.

        Example::

            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_FUTURES)
            >>> def callback(fut):
            ...     print(f"RPC return value is {fut.wait()}.")
            >>> fut = torch.futures.Future()
            >>> # The inserted callback will print the return value when
            >>> # receiving the response from "worker1"
            >>> cb_fut = fut.then(callback)
            >>> chain_cb_fut = cb_fut.then(
            ...     lambda x : print(f"Chained cb done. {x.wait()}")
````
- **EN**: This chunk continues `then` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `then`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 161-176 / 第 161-176 行
````python
            ... )
            >>> fut.set_result(5)
            RPC return value is 5.
            Chained cb done. None
        """
        return cast(Future[S], super().then(callback))

    def add_done_callback(self, callback: Callable[[Future[T]], None]) -> None:
        r"""
        Append the given callback function to this ``Future``, which will be run
        when the ``Future`` is completed.  Multiple callbacks can be added to
        the same ``Future``, but the order in which they will be executed cannot
        be guaranteed. The callback must take one argument, which is the
        reference to this ``Future``. The callback function can use the
        :meth:`value` method to get the value. Note that if this ``Future`` is
        already completed, the given callback will be run inline.
````
- **EN**: This chunk defines `add_done_callback`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `add_done_callback`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 178-197 / 第 178-197 行
````python
        We recommend that you use the :meth:`then` method as it provides a way
        to synchronize after your callback has completed. ``add_done_callback``
        can be cheaper if your callback does not return anything. But both
        :meth:`then` and ``add_done_callback`` use the same callback
        registration API under the hood.

        With respect to GPU tensors, this method behaves in the same way as
        :meth:`then`.

        Args:
            callback(``Future``): a ``Callable`` that takes in one argument,
                which is the reference to this ``Future``.

        .. note:: Note that if the callback function throws, either
            through the original future being completed with an exception and
            calling ``fut.wait()``, or through other code in the callback,
            error handling must be carefully taken care of. For example, if
            this callback later completes additional futures, those futures are
            not marked as completed with an error and the user is responsible
            for handling completion/waiting on those futures independently.
````
- **EN**: This chunk continues `add_done_callback` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `add_done_callback`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 199-217 / 第 199-217 行
````python
        Example::

            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_FUTURES)
            >>> def callback(fut):
            ...     print("This will run after the future has finished.")
            ...     print(fut.wait())
            >>> fut = torch.futures.Future()
            >>> fut.add_done_callback(callback)
            >>> fut.set_result(5)
            This will run after the future has finished.
            5
        """
        super().add_done_callback(callback)

    def set_result(self, result: T) -> None:
        r"""
        Set the result for this ``Future``, which will mark this ``Future`` as
        completed and trigger all attached callbacks. Note that a ``Future``
        cannot be marked completed twice.
````
- **EN**: This chunk defines `set_result`, which mutates configuration or backend state that affects later execution. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `set_result`，其作用是修改会影响后续执行的配置或后端状态。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 219-233 / 第 219-233 行
````python
        If the result contains tensors that reside on GPUs, this method can be
        called even if the asynchronous kernels that are populating those
        tensors haven't yet completed running on the device, provided that the
        streams on which those kernels were enqueued are set as the current ones
        when this method is called. Put simply, it's safe to call this method
        immediately after launching those kernels, without any additional
        synchronization, as long as one doesn't change streams in between. This
        method will record events on all the relevant current streams and will
        use them to ensure proper scheduling for all the consumers of this
        ``Future``.

        Args:
            result (object): the result object of this ``Future``.

        Example::
````
- **EN**: This chunk continues `set_result` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `set_result`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 235-251 / 第 235-251 行
````python
            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_FUTURES)
            >>> import threading
            >>> import time
            >>> def slow_set_future(fut, value):
            ...     time.sleep(0.5)
            ...     fut.set_result(value)
            >>> fut = torch.futures.Future()
            >>> t = threading.Thread(
            ...     target=slow_set_future,
            ...     args=(fut, torch.ones(2) * 3)
            ... )
            >>> t.start()
            >>> print(fut.wait())
            tensor([3., 3.])
            >>> t.join()
        """
        super().set_result(result)
````
- **EN**: This chunk continues `set_result` and expands its internal control flow or state updates. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 这一段延续了 `set_result`，进一步展开其内部控制流或状态更新。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 253-272 / 第 253-272 行
````python
    def set_exception(self, result: T) -> None:
        r"""
        Set an exception for this ``Future``, which will mark this ``Future`` as
        completed with an error and trigger all attached callbacks. Note that
        when calling wait()/value() on this ``Future``, the exception set here
        will be raised inline.

        Args:
            result (BaseException): the exception for this ``Future``.

        Example::

            >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_FUTURES)
            >>> fut = torch.futures.Future()
            >>> fut.set_exception(ValueError("foo"))
            >>> fut.wait()
            Traceback (most recent call last):
            ...
            ValueError: foo
        """
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `set_exception`, which mutates configuration or backend state that affects later execution.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `set_exception`，其作用是修改会影响后续执行的配置或后端状态。

### Lines 273-289 / 第 273-289 行
````python
        if not isinstance(result, Exception):
            raise AssertionError(
                f"{result} is of type {type(result)}, not an Exception."
            )

        def raise_error(fut_result):
            raise fut_result

        super()._set_unwrap_func(raise_error)
        self.set_result(result)  # type: ignore[arg-type]


def collect_all(futures: list[Future]) -> Future[list[Future]]:
    r"""
    Collects the provided :class:`~torch.futures.Future` objects into a single
    combined :class:`~torch.futures.Future` that is completed when all of the
    sub-futures are completed.
````
- **EN**: This chunk defines `collect_all`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `collect_all`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 291-310 / 第 291-310 行
````python
    Args:
        futures (list): a list of :class:`~torch.futures.Future` objects.

    Returns:
        Returns a :class:`~torch.futures.Future` object to a list of the passed
        in Futures.

    Example::
        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_FUTURES)
        >>> fut0 = torch.futures.Future()
        >>> fut1 = torch.futures.Future()
        >>> fut = torch.futures.collect_all([fut0, fut1])
        >>> fut0.set_result(0)
        >>> fut1.set_result(1)
        >>> fut_list = fut.wait()
        >>> print(f"fut0 result = {fut_list[0].wait()}")
        fut0 result = 0
        >>> print(f"fut1 result = {fut_list[1].wait()}")
        fut1 result = 1
    """
````
- **EN**: This chunk continues `collect_all` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `collect_all`，进一步展开其内部控制流或状态更新。

### Lines 311-325 / 第 311-325 行
````python
    return cast(
        Future[list[Future]],
        torch._C._collect_all(cast(list[torch._C.Future], futures)),
    )


def wait_all(futures: list[Future]) -> list:
    r"""
    Waits for all provided futures to be complete, and returns
    the list of completed values. If any of the futures encounters an error,
    the method will exit early and report the error not waiting for other
    futures to complete.

    Args:
        futures (list): a list of :class:`~torch.futures.Future` object.
````
- **EN**: This chunk defines `wait_all`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wait_all`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 327-335 / 第 327-335 行
````python
    Returns:
        A list of the completed :class:`~torch.futures.Future` results. This
        method will throw an error if ``wait`` on any
        :class:`~torch.futures.Future` throws.
    """
    return [
        fut.wait()
        for fut in torch._C._collect_all(cast(list[torch._C.Future], futures)).wait()
    ]
````
- **EN**: This chunk continues `wait_all` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `wait_all`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **T**
  - EN: `T` is one of the main symbols declared or implemented in this file.
  - CN: `T` 是本文件声明或实现的主要符号之一。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
- **Package wiring**
  - EN: The file shapes the public namespace by re-exporting symbols and applying package-level initialization.
  - CN: 该文件通过重新导出符号并执行包级初始化来塑造公共命名空间。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `T`, `S`, `Future`, `collect_all`, `wait_all`
