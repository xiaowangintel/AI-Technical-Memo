# tracked_process_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/compile_worker/tracked_process_pool.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `_QueueStats`, and `TrackedProcessPoolExecutor`. It exposes functions such as `_queue_stats_report`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `_QueueStats`、`TrackedProcessPoolExecutor` 等类。同时提供 `_queue_stats_report` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import atexit
import concurrent
import dataclasses
import logging
import threading
from collections.abc import Callable
from concurrent.futures import Future, ProcessPoolExecutor
from dataclasses import dataclass
from multiprocessing.context import BaseContext
from time import time
from typing import Any, TypeVar
from typing_extensions import ParamSpec

# _thread_safe_fork is needed because the subprocesses in the pool can read
````
- **EN**: Imports dependencies such as `atexit`, `concurrent`, `dataclasses`, `logging`, `threading`, `collections.abc`, and `...+5` for the logic in this range.
- **CN**: 这里导入了 `atexit`、`concurrent`、`dataclasses`、`logging`、`threading`、`collections.abc`、`另有5项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python
# justknobs, e.g., in the Triton compiler. For internal, the import installs
# functionality to destroy singletons before forking and re-enable them after.
import torch._thread_safe_fork  # noqa: F401


_P = ParamSpec("_P")
_R = TypeVar("_R")


log = logging.getLogger(__name__)


@dataclass
class _QueueStats:
````
- **EN**: Imports dependencies such as `torch._thread_safe_fork  # noqa: F401` for the logic in this range. Introduces class `_QueueStats`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `torch._thread_safe_fork  # noqa: F401` 等依赖，为后续逻辑提供基础能力。这里定义了类`_QueueStats`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 29-42 / 第 29-42 行
````python
    # Mapping from id(future) -> start time
    pending: dict[int, float] = dataclasses.field(default_factory=dict)
    timing: list[float] = dataclasses.field(default_factory=list)
    enqueue_count: int = 0
    dequeue_count: int = 0
    max_queue_depth: int = 0
    pool_count: int = 0


# The queue statistics tracked by TrackedProcessPoolExecutor. Always grab
# _queue_stats_lock before touching.
_queue_stats = _QueueStats()
_queue_stats_lock = threading.Lock()

````
- **EN**: Initializes or updates values such as `pending`, `timing`, `enqueue_count`, `dequeue_count`, `max_queue_depth`, `pool_count`, and `...+2`. This range continues the implementation of class `_QueueStats`.
- **CN**: 初始化或更新了 `pending`、`timing`、`enqueue_count`、`dequeue_count`、`max_queue_depth`、`pool_count`、`另有2项` 等值。这一段延续了类`_QueueStats` 的具体实现。

### Lines 43-56 / 第 43-56 行
````python

class TrackedProcessPoolExecutor(ProcessPoolExecutor):
    def __init__(
        self,
        max_workers: int | None = None,
        mp_context: BaseContext | None = None,
        initializer: Callable[[], object] | None = None,
    ) -> None:
        with _queue_stats_lock:
            _queue_stats.pool_count += 1
        super().__init__(max_workers, mp_context, initializer)

    def _record_dequeue(self, f: Future[Any]) -> None:
        now = time()
````
- **EN**: Introduces class `TrackedProcessPoolExecutor`, function `__init__`, function `_record_dequeue`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_workers`, `mp_context`, `initializer`, and `now`.
- **CN**: 这里定义了类`TrackedProcessPoolExecutor`、函数`__init__`、函数`_record_dequeue`。包含分支、循环或上下文管理等控制流。初始化或更新了 `max_workers`、`mp_context`、`initializer`、`now` 等值。

### Lines 57-70 / 第 57-70 行
````python
        with _queue_stats_lock:
            stats = _queue_stats
            if (start_time := stats.pending.pop(id(f), None)) is None:
                return
            stats.dequeue_count += 1
            duration = now - start_time
            stats.timing.append(duration)

    def _record_enqueue(self, f: Future[Any]) -> None:
        # Monkeypatch the set_running_or_notify_cancel so we can track when the Future moves out of PENDING.
        saved_running_or_notify_cancel = f.set_running_or_notify_cancel

        def set_running_or_notify_cancel() -> Any:
            self._record_dequeue(f)
````
- **EN**: Introduces function `_record_enqueue`, function `set_running_or_notify_cancel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stats`, `duration`, and `saved_running_or_notify_cancel`.
- **CN**: 这里定义了函数`_record_enqueue`、函数`set_running_or_notify_cancel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `stats`、`duration`、`saved_running_or_notify_cancel` 等值。

### Lines 71-84 / 第 71-84 行
````python
            return saved_running_or_notify_cancel()

        now = time()
        with _queue_stats_lock:
            stats = _queue_stats
            stats.pending[id(f)] = now
            stats.enqueue_count += 1
            stats.max_queue_depth = max(stats.max_queue_depth, len(stats.pending))
            f.set_running_or_notify_cancel = set_running_or_notify_cancel  # type: ignore[method-assign]

        if f._state != concurrent.futures._base.PENDING:
            self._record_dequeue(f)

    def submit(
````
- **EN**: Introduces function `submit`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `now`, and `stats`.
- **CN**: 这里定义了函数`submit`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `now`、`stats` 等值。

### Lines 85-98 / 第 85-98 行
````python
        self, fn: Callable[_P, _R], /, *args: _P.args, **kwargs: _P.kwargs
    ) -> Future[_R]:
        # pyrefly: ignore [bad-argument-type]
        f = super().submit(fn, *args, **kwargs)
        self._record_enqueue(f)
        return f


@atexit.register
def _queue_stats_report() -> None:
    stats = _queue_stats
    if stats.pool_count == 0:
        return

````
- **EN**: Introduces function `_queue_stats_report`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_queue_stats_report`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 99-112 / 第 99-112 行
````python
    timing = stats.timing
    timing.sort()

    log.info("AsyncCompile Metrics:")
    log.info("  Pools %s", stats.pool_count)
    log.info(
        "  Items %d enqueued / %d dequeued", stats.enqueue_count, stats.dequeue_count
    )
    log.info("  Max Queue Depth: %d", stats.max_queue_depth)
    n = len(timing)
    if n > 0:
        log.info("  Longest queue time: %0.2fs", timing[-1])
        log.info("  P50: %0.2fs", timing[n // 2])
        if n >= 20:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `timing`, and `n`. This range continues the implementation of function `_queue_stats_report`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `timing`、`n` 等值。这一段延续了函数`_queue_stats_report` 的具体实现。

### Lines 113-113 / 第 113-113 行
````python
            log.info("  P95: %0.2fs", timing[n * 95 // 100])
````
- **EN**: This range continues the implementation of function `_queue_stats_report`.
- **CN**: 这一段延续了函数`_queue_stats_report` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Primary classes: `_QueueStats`, and `TrackedProcessPoolExecutor`  
  **CN**: 主要类：`_QueueStats`、`TrackedProcessPoolExecutor`
- **EN**: Primary functions: `_queue_stats_report`  
  **CN**: 主要函数：`_queue_stats_report`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `atexit`, `concurrent`, `dataclasses`, `logging`, `threading`, `collections.abc`, `concurrent.futures`, `multiprocessing.context`, `time`, `typing`
- **Third-party / 第三方**: `typing_extensions`
- **PyTorch/Internal / PyTorch 内部**: `torch._thread_safe_fork`
