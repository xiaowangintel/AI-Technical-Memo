# wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/profiler/wrapper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `wrapper`-related logic centered around `WorkerProfiler`, `TorchProfilerWrapper`. / 实现与 `wrapper` 相关的逻辑，核心符号包括 `WorkerProfiler`, `TorchProfilerWrapper`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from abc import ABC, abstractmethod
from collections.abc import Callable
from contextlib import nullcontext
from typing import Literal

import torch
from typing_extensions import override

from vllm.config import ProfilerConfig
from vllm.config.profiler import _is_uri_path
from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `abc`, `collections.abc`, `contextlib`, external packages such as `torch`, `typing_extensions`, `torch.cuda.profiler`, vLLM modules such as `vllm.config`, `vllm.config.profiler`, `vllm.logger`. It prepares the symbols later used by `WorkerProfiler`, `TorchProfilerWrapper`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.config`, `vllm.config.profiler`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `WorkerProfiler`, `TorchProfilerWrapper` 提供上下文。

### WorkerProfiler overview (lines 19-148)
```python
class WorkerProfiler(ABC):
    def __init__(self, profiler_config: ProfilerConfig) -> None:
        self._delay_iters = profiler_config.delay_iterations
        if self._delay_iters > 0:
            logger.info_once(
                "GPU profiling will start "
                f"{self._delay_iters} steps after start_profile."
            )

        self._max_iters = profiler_config.max_iterations
        if self._max_iters > 0:
            logger.info_once(
                "GPU profiling will stop "
                f"after {self._max_iters} worker steps, "
                "or when stop_profile is received."
            )

        # Track when the profiler gets triggered by start_profile
        self._active_iteration_count = 0
        self._active = False

        # Track when the profiler is actually running
        self._profiling_for_iters = 0
        self._running = False

    @abstractmethod
    def _start(self) -> None:
    # ...
```
**EN:** Defines the `WorkerProfiler` class used by this module. It extends `ABC`. Key methods include `__init__`, `start`, `step`, `stop`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `WorkerProfiler` 是该文件中的核心类，用于封装与 `WorkerProfiler` 相关的状态和行为。 它继承自 `ABC`。 关键方法包括 `__init__`, `start`, `step`, `stop`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### WorkerProfiler.__init__ (lines 20-42)
```python
    def __init__(self, profiler_config: ProfilerConfig) -> None:
        self._delay_iters = profiler_config.delay_iterations
        if self._delay_iters > 0:
            logger.info_once(
                "GPU profiling will start "
                f"{self._delay_iters} steps after start_profile."
            )

        self._max_iters = profiler_config.max_iterations
        if self._max_iters > 0:
            logger.info_once(
                "GPU profiling will stop "
                f"after {self._max_iters} worker steps, "
                "or when stop_profile is received."
            )

        # Track when the profiler gets triggered by start_profile
        self._active_iteration_count = 0
        self._active = False

        # Track when the profiler is actually running
        self._profiling_for_iters = 0
        self._running = False
```
**EN:** `__init__` initializes state required by the module. It mainly works with `profiler_config`. Inside the body, it relies on `logger.info_once` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `profiler_config` 等参数。 实现过程中会调用 `logger.info_once` 等函数完成关键步骤。

### WorkerProfiler.step (lines 83-114)
```python
    def step(self) -> None:
        """Update the profiler state at each worker step,
        to handle delayed starts and max iteration limits."""
        if not self._active:
            return

        self._active_iteration_count += 1

        if (
            not self._running
            and self._delay_iters > 0
            and self._active_iteration_count == self._delay_iters
        ):
            logger.info_once("Starting profiler after delay...")
            self._call_start()

        # Call profiler step for schedule-based profiling
        # Only count iterations where data is actually recorded (not warmup)
        if self._running and self._profiler_step():
            self._profiling_for_iters += 1
    # ...
            # Automatically stop the profiler after max iters
            # will be marked as not running, but leave as active so that stop
            # can clean up properly
            logger.info_once("Max profiling iterations reached. Stopping profiler...")
            self._call_stop()
            return
```
**EN:** `step`: Update the profiler state at each worker step, to handle delayed starts and max iteration limits. Inside the body, it relies on `logger.info_once`, `self._call_start`, `self._profiler_step` to complete the main steps.
**CN:** `step` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `logger.info_once`, `self._call_start`, `self._profiler_step` 等函数完成关键步骤。

### WorkerProfiler.stop (lines 126-138)
```python
    def stop(self) -> None:
        """Attempt to stop the profiler, accounting for overlapped calls."""
        if not self._active:
            logger.debug(
                "stop_profile received when profiler is not active. Ignoring request."
            )
            return
        self._active = False
        self._active_iteration_count = 0
        self._profiling_for_iters = 0

        if self._running:
            self._call_stop()
```
**EN:** `stop`: Attempt to stop the profiler, accounting for overlapped calls. Inside the body, it relies on `logger.debug`, `self._call_stop` to complete the main steps.
**CN:** `stop` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `logger.debug`, `self._call_stop` 等函数完成关键步骤。

### WorkerProfiler.start (lines 71-81)
```python
    def start(self) -> None:
        """Attempt to start the profiler, accounting for delayed starts."""
        if self._active:
            logger.debug(
                "start_profile received when profiler is already active. "
                "Ignoring request."
            )
            return
        self._active = True
        if self._delay_iters == 0:
            self._call_start()
```
**EN:** `start`: Attempt to start the profiler, accounting for delayed starts. Inside the body, it relies on `logger.debug`, `self._call_start` to complete the main steps.
**CN:** `start` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `logger.debug`, `self._call_start` 等函数完成关键步骤。

### TorchProfilerWrapper overview (lines 159-307)
```python
class TorchProfilerWrapper(WorkerProfiler):
    def __init__(
        self,
        profiler_config: ProfilerConfig,
        worker_name: str,
        local_rank: int,
        activities: list[TorchProfilerActivity],
        on_trace_ready: Callable[[torch.profiler.profile], None] | None = None,
    ) -> None:
        super().__init__(profiler_config)

        self.local_rank = local_rank
        self.profiler_config = profiler_config
        torch_profiler_trace_dir = profiler_config.torch_profiler_dir
        if local_rank in (None, 0):
            logger.info_once(
                "Torch profiling enabled. Traces will be saved to: %s",
                torch_profiler_trace_dir,
            )
            logger.debug(
                "Profiler config: record_shapes=%s,"
                "profile_memory=%s,with_stack=%s,with_flops=%s",
                profiler_config.torch_profiler_record_shapes,
                profiler_config.torch_profiler_with_memory,
                profiler_config.torch_profiler_with_stack,
                profiler_config.torch_profiler_with_flops,
            )
    # ...
```
**EN:** Defines the `TorchProfilerWrapper` class used by this module. It extends `WorkerProfiler`. Key methods include `__init__`, `annotate_context_manager`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `TorchProfilerWrapper` 是该文件中的核心类，用于封装与 `TorchProfilerWrapper` 相关的状态和行为。 它继承自 `WorkerProfiler`。 关键方法包括 `__init__`, `annotate_context_manager`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### TorchProfilerWrapper.__init__ (lines 160-237)
```python
    def __init__(
        self,
        profiler_config: ProfilerConfig,
        worker_name: str,
        local_rank: int,
        activities: list[TorchProfilerActivity],
        on_trace_ready: Callable[[torch.profiler.profile], None] | None = None,
    ) -> None:
        super().__init__(profiler_config)

        self.local_rank = local_rank
        self.profiler_config = profiler_config
        torch_profiler_trace_dir = profiler_config.torch_profiler_dir
        if local_rank in (None, 0):
            logger.info_once(
                "Torch profiling enabled. Traces will be saved to: %s",
                torch_profiler_trace_dir,
            )
            logger.debug(
                "Profiler config: record_shapes=%s,"
    # ...
        # (WAIT or WARMUP), so only wait + warmup - 1 non-active steps
        # remain to be advanced through via profiler.step() calls.
        self._warmup_steps_remaining = max(
            profiler_config.wait_iterations + profiler_config.warmup_iterations - 1,
            0,
        )
```
**EN:** `__init__` initializes state required by the module. It mainly works with `profiler_config`, `worker_name`, `local_rank`, `activities`. Inside the body, it relies on `super.__init__`, `torch.profiler.profile`, `max` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `profiler_config`, `worker_name`, `local_rank`, `activities` 等参数。 实现过程中会调用 `super.__init__`, `torch.profiler.profile`, `max` 等函数完成关键步骤。

### TorchProfilerWrapper.annotate_context_manager (lines 306-307)
```python
    def annotate_context_manager(self, name: str):
        return torch.profiler.record_function(name)
```
**EN:** `annotate_context_manager` implements helper logic used by this module. It mainly works with `name`. Inside the body, it relies on `torch.profiler.record_function` to complete the main steps.
**CN:** `annotate_context_manager` 负责实现本模块使用的辅助逻辑。 它主要处理 `name` 等参数。 实现过程中会调用 `torch.profiler.record_function` 等函数完成关键步骤。

### TorchProfilerWrapper._profiler_step (lines 290-303)
```python
    def _profiler_step(self) -> bool:
        """Call profiler.step() when using schedule-based profiling.

        Returns:
            True if the step was an active profiling step (data recorded),
            False if the step was a warmup step (data discarded).
        """
        if self._uses_schedule:
            self.profiler.step()
            # Track warmup steps - only count active steps toward max_iterations
            if self._warmup_steps_remaining > 0:
                self._warmup_steps_remaining -= 1
                return False
        return True
```
**EN:** `_profiler_step`: Call profiler.step() when using schedule-based profiling. Inside the body, it relies on `self.profiler.step` to complete the main steps.
**CN:** `_profiler_step` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `self.profiler.step` 等函数完成关键步骤。

### TorchProfilerWrapper._build_profiler_table (lines 239-249)
```python
    def _build_profiler_table(
        self,
        sort_key: str,
        row_limit: int | None = None,
    ) -> str:
        if row_limit is None:  # use profiler default row limit of 100
            return self.profiler.key_averages().table(sort_by=sort_key)
        return self.profiler.key_averages().table(
            sort_by=sort_key,
            row_limit=row_limit,
        )
```
**EN:** `_build_profiler_table` constructs objects for downstream execution. It mainly works with `sort_key`, `row_limit`. Inside the body, it relies on `self.profiler.key_averages.table`, `self.profiler.key_averages` to complete the main steps.
**CN:** `_build_profiler_table` 负责为下游执行构造对象。 它主要处理 `sort_key`, `row_limit` 等参数。 实现过程中会调用 `self.profiler.key_averages.table`, `self.profiler.key_averages` 等函数完成关键步骤。

### CudaProfilerWrapper (lines 310-328)
```python
class CudaProfilerWrapper(WorkerProfiler):
    def __init__(self, profiler_config: ProfilerConfig) -> None:
        super().__init__(profiler_config)
        # Note: lazy import to avoid dependency issues if CUDA is not available.
        import torch.cuda.profiler as cuda_profiler

        self._cuda_profiler = cuda_profiler

    @override
    def _start(self) -> None:
        self._cuda_profiler.start()

    @override
    def _stop(self) -> None:
        self._cuda_profiler.stop()

    @override
    def annotate_context_manager(self, name: str):
        return torch.cuda.nvtx.range(name)
```
**EN:** Defines the `CudaProfilerWrapper` class used by this module. It extends `WorkerProfiler`. Key methods include `__init__`, `annotate_context_manager`.
**CN:** `CudaProfilerWrapper` 是该文件中的核心类，用于封装与 `CudaProfilerWrapper` 相关的状态和行为。 它继承自 `WorkerProfiler`。 关键方法包括 `__init__`, `annotate_context_manager`。

## Key Concepts / 关键概念
- **`WorkerProfiler`**: Core class that organizes module behavior. / **`WorkerProfiler`**：组织模块行为的核心类。
- **`TorchProfilerWrapper`**: Core class that organizes module behavior. / **`TorchProfilerWrapper`**：组织模块行为的核心类。
- **`CudaProfilerWrapper`**: Core class that organizes module behavior. / **`CudaProfilerWrapper`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: abc, collections.abc, contextlib, typing
- **Third-party / 第三方**: torch, typing_extensions, torch.cuda.profiler
- **Internal vLLM / vLLM 内部依赖**: vllm.config, vllm.config.profiler, vllm.logger
