# await_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/await_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `await_sync`, `get_loop`, `_new_loop`, `_cancel_all_tasks`, and `_patch_loop`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `await_sync`、`get_loop`、`_new_loop`、`_cancel_all_tasks`、`_patch_loop` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import asyncio
import sys
import weakref
from asyncio import AbstractEventLoop, Future
from collections.abc import Awaitable, Callable, Coroutine, Generator, Iterator
from contextlib import contextmanager, ExitStack
from contextvars import Context
from typing import Any, Protocol, TypeVar

from torch.utils._ordered_set import OrderedSet


T = TypeVar("T")
TCoro = Generator[Any, None, T]
````
- **EN**: Imports dependencies such as `asyncio`, `sys`, `weakref`, `collections.abc`, `contextlib`, `contextvars`, and `...+2` for the logic in this range. Initializes or updates values such as `T`, and `TCoro`.
- **CN**: 这里导入了 `asyncio`、`sys`、`weakref`、`collections.abc`、`contextlib`、`contextvars`、`另有2项` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `T`、`TCoro` 等值。

### Lines 15-28 / 第 15-28 行
````python

if sys.version_info >= (3, 11):

    class TaskFactory(Protocol):
        def __call__(
            self,
            __loop: AbstractEventLoop,
            __factory: Coroutine[None, None, object] | Generator[None, None, object],
            __context: Context | None = None,
            /,
        ) -> asyncio.futures.Future[object]: ...

    TaskFactoryType = TaskFactory
else:
````
- **EN**: Introduces class `TaskFactory`, function `__call__`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `__loop`, `__factory`, `__context`, `TaskFactoryType`, and `else`.
- **CN**: 这里定义了类`TaskFactory`、函数`__call__`。包含分支、循环或上下文管理等控制流。初始化或更新了 `__loop`、`__factory`、`__context`、`TaskFactoryType`、`else` 等值。

### Lines 29-42 / 第 29-42 行
````python
    TaskFactoryType = Callable[[AbstractEventLoop, Generator[TCoro, None, T]], Future]  # type: ignore[valid-type]


def await_sync(awaitable: Awaitable[T]) -> T:
    with get_loop() as loop:
        return loop.run_until_complete(awaitable)


@contextmanager
def get_loop(
    always_create_new_loop: bool = False,
) -> Iterator[AbstractEventLoop]:
    try:
        loop = asyncio.get_event_loop()
````
- **EN**: Introduces function `await_sync`, function `get_loop`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`await_sync`、函数`get_loop`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 43-56 / 第 43-56 行
````python
    except RuntimeError as re:
        if "There is no current event loop in thread" in str(re):
            with _new_loop() as loop:
                yield loop
            return
        else:
            raise

    @contextmanager
    def _restore_loop(
        loop: asyncio.AbstractEventLoop,
    ) -> Iterator[None]:
        try:
            yield
````
- **EN**: Introduces function `_restore_loop`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_restore_loop`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python
        finally:
            asyncio.set_event_loop(loop)

    @contextmanager
    def _restore_running_loop() -> Iterator[None]:
        loop_from_events = asyncio.events._get_running_loop()
        asyncio.events._set_running_loop(None)
        try:
            yield
        finally:
            asyncio.events._set_running_loop(loop_from_events)

    with ExitStack() as stack:
        if loop.is_running():
````
- **EN**: Introduces function `_restore_running_loop`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_restore_running_loop`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python
            stack.enter_context(_restore_running_loop())
            stack.enter_context(_restore_loop(loop=loop))
            loop = stack.enter_context(_new_loop(loop.get_task_factory()))  # type: ignore[arg-type]
        elif loop.is_closed():
            loop = stack.enter_context(_new_loop())  # type: ignore[arg-type]
        elif always_create_new_loop:
            stack.enter_context(_restore_loop(loop=loop))
            loop = stack.enter_context(_new_loop())  # type: ignore[arg-type]
        yield loop


@contextmanager
def _new_loop(
    task_factory: TaskFactoryType | None = None,
````
- **EN**: Introduces function `_new_loop`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_new_loop`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 85-98 / 第 85-98 行
````python
) -> Iterator[asyncio.AbstractEventLoop]:
    loop = asyncio.new_event_loop()
    tasks = _patch_loop(loop)

    if task_factory:
        # pyre-ignore[6]
        loop.set_task_factory(task_factory)  # type: ignore[arg-type]

    asyncio.set_event_loop(loop)
    try:
        yield loop
    finally:
        try:
            _cancel_all_tasks(loop, tasks)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `loop`, `tasks`, `try`, and `finally`. This range continues the implementation of function `_new_loop`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `loop`、`tasks`、`try`、`finally` 等值。这一段延续了函数`_new_loop` 的具体实现。

### Lines 99-112 / 第 99-112 行
````python
        finally:
            asyncio.set_event_loop(None)
            loop.close()


def _cancel_all_tasks(
    loop: AbstractEventLoop,
    tasks: OrderedSet[Future],  # type: ignore[type-arg]
) -> None:
    to_cancel = [task for task in tasks if not task.done()]

    if not to_cancel:
        return

````
- **EN**: Introduces function `_cancel_all_tasks`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `finally`, `loop`, `tasks`, and `to_cancel`.
- **CN**: 这里定义了函数`_cancel_all_tasks`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `finally`、`loop`、`tasks`、`to_cancel` 等值。

### Lines 113-126 / 第 113-126 行
````python
    # pyre-fixme[1001]: Awaitable assigned to `task` is never awaited.
    for task in to_cancel:
        task.cancel()

    # pyrefly: ignore [bad-argument-type]
    loop.run_until_complete(asyncio.gather(*to_cancel, return_exceptions=True))

    for task in to_cancel:
        if task.cancelled():
            continue
        if task.exception() is not None:
            loop.call_exception_handler(
                {
                    "message": "unhandled exception during asyncio.run() shutdown",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `_cancel_all_tasks`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`_cancel_all_tasks` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
                    "exception": task.exception(),
                    "task": task,
                }
            )


def _patch_loop(loop: AbstractEventLoop) -> OrderedSet[Future]:  # type: ignore[type-arg]
    tasks: weakref.WeakSet[Future] = weakref.WeakSet()  # type: ignore[type-arg]

    task_factories: list[TaskFactoryType | None] = [None]

    def _set_task_factory(factory: TaskFactoryType | None) -> None:
        task_factories[0] = factory

````
- **EN**: Introduces function `_patch_loop`, function `_set_task_factory`. Initializes or updates values such as `tasks`, and `task_factories`.
- **CN**: 这里定义了函数`_patch_loop`、函数`_set_task_factory`。初始化或更新了 `tasks`、`task_factories` 等值。

### Lines 141-154 / 第 141-154 行
````python
    def _get_task_factory() -> TaskFactoryType | None:
        return task_factories[0]

    def _safe_task_factory(
        loop: AbstractEventLoop,
        coro: TCoro,  # type: ignore[type-arg]
        *,
        context: Context | None = None,
    ) -> asyncio.Future:  # type: ignore[valid-type, type-arg]
        task_factory = task_factories[0]
        if task_factory is None:
            if sys.version_info >= (3, 11):
                # pyrefly: ignore [bad-argument-type]
                task = asyncio.Task(coro, loop=loop, context=context)
````
- **EN**: Introduces function `_get_task_factory`, function `_safe_task_factory`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `loop`, `coro`, `context`, `task_factory`, and `task`.
- **CN**: 这里定义了函数`_get_task_factory`、函数`_safe_task_factory`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `loop`、`coro`、`context`、`task_factory`、`task` 等值。

### Lines 155-168 / 第 155-168 行
````python
            else:
                task = asyncio.Task(coro, loop=loop)
            # pyre-ignore[16]: `Task` has no attribute `_source_traceback`.
            if task._source_traceback:  # type: ignore[attr-defined]
                del task._source_traceback[  # type: ignore[attr-defined]
                    -1
                ]  # pragma: no cover  # type: ignore[attr-defined]
        else:
            if sys.version_info >= (3, 11):
                task = task_factory(loop, coro, context=context)  # type: ignore[arg-type, call-arg, assignment]
            else:
                task = task_factory(loop, coro)  # type: ignore[arg-type]
        #  `Union[Task[Any], Future[Any]]`.
        tasks.add(task)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `task`. This range continues the implementation of function `_patch_loop._safe_task_factory`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`task` 等值。这一段延续了函数`_patch_loop._safe_task_factory` 的具体实现。

### Lines 169-178 / 第 169-178 行
````python
        return task

    # pyre-ignore[6]
    loop.set_task_factory(_safe_task_factory)  # type: ignore[method-assign, arg-type]
    # pyre-ignore[8]
    loop.set_task_factory = _set_task_factory  # type: ignore[method-assign, assignment]
    # pyre-ignore[8]
    loop.get_task_factory = _get_task_factory  # type: ignore[method-assign, assignment]

    return tasks  # type: ignore[return-value]
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `_patch_loop`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`_patch_loop` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Primary functions: `await_sync`, `get_loop`, `_new_loop`, `_cancel_all_tasks`, and `_patch_loop`  
  **CN**: 主要函数：`await_sync`、`get_loop`、`_new_loop`、`_cancel_all_tasks`、`_patch_loop`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `sys`, `weakref`, `collections.abc`, `contextlib`, `contextvars`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch.utils._ordered_set`
