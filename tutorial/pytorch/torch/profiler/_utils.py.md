# _utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/profiler/_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements profiling APIs, trace utilities, and performance-analysis helpers.
- **Purpose (CN)**: 实现 profiling API、跟踪工具以及性能分析辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
import functools
import operator
import re
from collections import deque
from dataclasses import dataclass
from typing import Any, Literal, TYPE_CHECKING

from torch.autograd.profiler import profile
from torch.profiler import DeviceType


if TYPE_CHECKING:
    from torch.autograd import _KinetoEvent
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.autograd.profiler, torch.profiler, torch.autograd; standard-library helpers such as functools, operator, re, .... Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.autograd.profiler、torch.profiler、torch.autograd；标准库辅助模块，如 functools、operator、re、...。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 17-30 / 第 17-30 行
````python
def _traverse(tree, next_fn, children_fn=lambda x: x.children, reverse: bool = False):
    order = reversed if reverse else lambda x: x
    remaining = deque(order(tree))
    while remaining:
        curr_event = next_fn(remaining)
        yield curr_event
        for child_event in order(children_fn(curr_event)):
            remaining.append(child_event)


traverse_dfs = functools.partial(_traverse, next_fn=lambda x: x.pop(), reverse=True)
traverse_bfs = functools.partial(
    _traverse, next_fn=lambda x: x.popleft(), reverse=False
)
````
- **EN**: This chunk defines `_traverse`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_traverse`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 33-51 / 第 33-51 行
````python
@dataclass
class EventMetrics:
    duration_time_ns: int = 0
    self_time_ns: int = 0
    idle_time_ns: int = 0
    queue_depth: int = 0

    @property
    def fraction_idle_time(self):
        if self.duration_time_ns == 0:
            return 0.0
        return self.idle_time_ns / self.duration_time_ns


@dataclass
class Interval:
    start: int
    end: int
    queue_depth: int = 0
````
- **EN**: It introduces or extends `EventMetrics`, `Interval`, which hold the main object-oriented state for this portion of the file. This chunk defines `fraction_idle_time`, which implements a focused helper used by the surrounding module. Decorators such as `dataclass`, `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `EventMetrics`、`Interval`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `fraction_idle_time`，其作用是实现周边模块使用的关键辅助逻辑。 像 `dataclass`、`property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 54-69 / 第 54-69 行
````python
class EventKey:
    def __init__(self, event) -> None:
        self.event = event

    def __hash__(self):
        return hash(self.event.id)

    def __eq__(self, other):
        return self.event.id == other.event.id

    def __repr__(self) -> str:
        return f"{self.event.name}"

    def intervals_overlap(self, intervals: list[Interval]):
        overlap_time = 0
        intervals = sorted(intervals, key=lambda x: x.start)
````
- **EN**: It introduces or extends `EventKey`, which hold the main object-oriented state for this portion of the file. This chunk defines `intervals_overlap`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `EventKey`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `intervals_overlap`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 71-90 / 第 71-90 行
````python
        if intervals:
            overlap_start = max(self.event.start_time_ns, intervals[0].start)
            overlap_end = min(self.event.end_time_ns, intervals[0].end)

            if overlap_start < overlap_end:
                overlap_time += overlap_end - overlap_start

        i, j = 0, 1
        while j < len(intervals):
            prev_interval = intervals[i]
            curr_interval = intervals[j]
            j += 1
            if prev_interval.end > curr_interval.start:
                # Completely subsumed by previous interval
                if prev_interval.end > curr_interval.end:
                    j += 1
                    continue
                else:
                    curr_interval.start = prev_interval.end
                    i = j
````
- **EN**: This chunk continues `EventKey` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `EventKey`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 92-111 / 第 92-111 行
````python
            overlap_start = max(self.event.start_time_ns, curr_interval.start)
            overlap_end = min(self.event.end_time_ns, curr_interval.end)
            if overlap_start < overlap_end:
                overlap_time += overlap_end - overlap_start

        return overlap_time


class BasicEvaluation:
    def __init__(self, prof: profile) -> None:
        self.profile = prof
        self.metrics: dict[EventKey, EventMetrics] = {}
        self.compute_self_time()
        self.event_keys = sorted(
            self.metrics.keys(), key=lambda x: x.event.start_time_ns
        )
        self.events = [e.event for e in self.event_keys]
        self.cuda_events: list[_KinetoEvent] = []
        self.queue_depth_list = self.compute_queue_depth()
        self.compute_idle_time()
````
- **EN**: It introduces or extends `BasicEvaluation`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `BasicEvaluation`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 113-132 / 第 113-132 行
````python
    def compute_self_time(self) -> None:
        """
        Computes event's self time(total time - time in child ops).
        """
        if self.profile.kineto_results is None:
            raise AssertionError("kineto_results must not be None")
        stack = deque(self.profile.kineto_results.experimental_event_tree())

        # standard iterating dfs
        while stack:
            curr_event = stack.pop()
            self_time = curr_event.duration_time_ns
            for child_event in curr_event.children:
                self_time -= child_event.duration_time_ns
                stack.append(child_event)
            if EventKey(curr_event) in self.metrics:
                raise AssertionError(
                    f"Duplicate id: {curr_event.id}, {curr_event.name}"
                )
            self.metrics[EventKey(curr_event)] = EventMetrics(self_time_ns=self_time)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `compute_self_time`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `compute_self_time`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 133-145 / 第 133-145 行
````python
            self.metrics[
                EventKey(curr_event)
            ].duration_time_ns = curr_event.duration_time_ns

    def compute_queue_depth(self):
        """
        Computes queue_depth at each event. This will calculate the queue depth data for
        All the events in the tree.
        This will return a list of Interval of queue depth data of cuda launch and kernels.
        """
        if self.profile.kineto_results is None:
            raise AssertionError("kineto_results must not be None")
        cuda_event_list = self.profile.kineto_results.events()
````
- **EN**: This chunk defines `compute_queue_depth`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `compute_queue_depth`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 147-165 / 第 147-165 行
````python
        def is_cuda_launch_kernel(e):
            """Check if the event is a CUDA launch kernel."""
            launch_patterns = {
                "cudaLaunchKernel",  # Standard CUDA
                "cudaLaunchKernelExC",  # Extended C
                "__cudaLaunchKernel",  # Internal
                "cudaLaunchCooperativeKernel",  # Collaborative (single-device)
                "cudaLaunchCooperativeKernelMultiDevice",  # Collaborative (multi-devices)
            }
            name = str(getattr(e, "name", e))
            return any(name.startswith(pattern) for pattern in launch_patterns)

        def is_cuda_kernel(e):
            """Check if the event is a CUDA runtime kernel."""
            # Check if the kernel is CUDA
            if e.device_type() != DeviceType.CUDA:
                return False

            name = str(getattr(e, "name", e)).lower()
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `is_cuda_kernel`, which checks a capability or invariant before later code relies on it. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `is_cuda_kernel`，其作用是检查某项能力或不变量，供后续逻辑依赖。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 167-183 / 第 167-183 行
````python
            # Exclude memory operations
            exclude_patterns = {"mem", "cpy", "alloc", "free"}

            return not any(pattern in name for pattern in exclude_patterns)

        cuda_launch_events = sorted(
            (e for e in cuda_event_list if is_cuda_launch_kernel(e)),
            key=lambda x: x.start_ns(),
        )
        cuda_kernel_events = sorted(
            (e for e in cuda_event_list if is_cuda_kernel(e)),
            key=lambda x: x.start_ns(),
        )

        self.cuda_events = sorted(
            cuda_launch_events + cuda_kernel_events, key=lambda x: x.start_ns()
        )
````
- **EN**: This chunk continues `is_cuda_kernel` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `is_cuda_kernel`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 185-200 / 第 185-200 行
````python
        kernel_mapping: dict[_KinetoEvent, int] = {}
        last_mapped_kernel = 0
        for cuda_launch_event in cuda_launch_events:
            index = index_of_first_match(
                cuda_kernel_events,
                lambda x: x.linked_correlation_id()
                == cuda_launch_event.linked_correlation_id(),
                start=last_mapped_kernel,
            )
            kernel_mapping[cuda_launch_event] = index
            last_mapped_kernel = index if index is not None else last_mapped_kernel

        current_kernel_index = 0
        spawned_kernel_index = -1

        all_events = cuda_launch_events + cuda_kernel_events + self.events
````
- **EN**: This chunk continues `is_cuda_kernel` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `is_cuda_kernel`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 202-221 / 第 202-221 行
````python
        def new_old_event_comparator(event):
            if hasattr(event, "start_us"):
                return event.start_us() * 1000
            if hasattr(event, "start_ns"):
                return event.start_ns()
            if hasattr(event, "start_time_ns"):
                return event.start_time_ns
            raise Exception("Unknown Event Type")  # noqa: TRY002

        queue_depth_list: list[Interval] = []
        all_events.sort(key=new_old_event_comparator)
        for event in all_events:
            # Find latest cuda kernel event
            if hasattr(event, "start_us"):
                start_time = event.start_us() * 1000
                # pyrefly: ignore [missing-attribute]
                end_time = (event.start_us() + event.duration_us()) * 1000
                # Find current spawned cuda kernel event
                if event in kernel_mapping and kernel_mapping[event] is not None:
                    spawned_kernel_index = kernel_mapping[event]
````
- **EN**: This chunk defines `new_old_event_comparator`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `new_old_event_comparator`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 222-238 / 第 222-238 行
````python
            if hasattr(event, "start_ns"):
                start_time = event.start_ns()
                end_time = event.start_ns() + event.duration_ns()
                # Find current spawned cuda kernel event
                if event in kernel_mapping and kernel_mapping[event] is not None:
                    spawned_kernel_index = kernel_mapping[event]
            elif hasattr(event, "start_time_ns"):
                start_time = event.start_time_ns  # type: ignore[attr-defined]
                end_time = event.end_time_ns  # type: ignore[attr-defined]

            while (
                current_kernel_index < len(cuda_kernel_events)
                and (cuda_kernel_events[current_kernel_index].start_ns()) <= start_time  # type: ignore[possibly-undefined]
            ):
                current_kernel_index += 1
            current_queue_depth = spawned_kernel_index - current_kernel_index + 1
            current_queue_depth = max(current_queue_depth, 0)
````
- **EN**: This chunk continues `new_old_event_comparator` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `new_old_event_comparator`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 240-259 / 第 240-259 行
````python
            if hasattr(event, "start_us") or hasattr(event, "start_ns"):
                queue_depth_list.append(
                    Interval(start_time, end_time, current_queue_depth)  # type: ignore[possibly-undefined]
                )
            elif hasattr(event, "start_time_ns"):
                self.metrics[EventKey(event)].queue_depth = current_queue_depth

        return queue_depth_list

    def compute_idle_time(self) -> None:
        """
        Computes idle time of the profile.
        """
        # Based on queue_depth_list, we can calculate idle time for all the events
        idle = False
        idle_start = 0
        idle_intervals: list[Interval] = []
        if self.queue_depth_list and self.events:
            idle_intervals += [
                Interval(self.events[0].start_time_ns, self.queue_depth_list[0].start),
````
- **EN**: This chunk defines `compute_idle_time`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `compute_idle_time`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 260-275 / 第 260-275 行
````python
                Interval(self.queue_depth_list[-1].end, self.events[-1].end_time_ns),
            ]

        for data_point in self.queue_depth_list:
            if data_point.queue_depth == 0 and not idle:
                idle_start = data_point.end
                idle = True
            if data_point.queue_depth > 0 and idle:
                idle_intervals.append(Interval(idle_start, data_point.start))
                idle = False

        event_list = [e.event for e in self.metrics]
        for event in event_list:
            self.metrics[EventKey(event)].idle_time_ns = EventKey(
                event
            ).intervals_overlap(idle_intervals)
````
- **EN**: This chunk continues `compute_idle_time` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `compute_idle_time`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 277-291 / 第 277-291 行
````python
    def rank_events(self, length):
        """
        Filter and Rank the events based on some heuristics:
        1) Events that are in the falling phase of the queue depth.
        2) Events that have a high idle_time, self_time difference.

        Parameters:
            length: The number of events to return.
        """

        # Find the interval when qd is falling to 0
        import torch

        queue_depth_list = list(reversed(self.queue_depth_list))
        qd_values = [e.queue_depth for e in queue_depth_list]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `rank_events`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `rank_events`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 293-307 / 第 293-307 行
````python
        bottom_threashold = 0
        top_threashold = 4
        decrease_interval = []
        i = 0
        while i < len(qd_values):
            if qd_values[i] > bottom_threashold:
                i += 1
                continue
            for j in range(i + 1, len(qd_values)):
                # Find next zero and if the max value between them exceeds
                # the threshold, then we have a falling interval
                next_minimum_idx = index_of_first_match(
                    qd_values, lambda x: x <= bottom_threashold, start=j
                )
                peak_idx = argmax(qd_values, start=j, end=next_minimum_idx)
````
- **EN**: This chunk continues `rank_events` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `rank_events`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 309-328 / 第 309-328 行
````python
                # if is a valid peak, we add to list and continue
                if peak_idx is not None and qd_values[peak_idx] >= top_threashold:
                    decrease_interval.append(
                        Interval(
                            queue_depth_list[peak_idx].start, queue_depth_list[i].start
                        )
                    )
                    i = next_minimum_idx if next_minimum_idx is not None else i
                    break
            i += 1
        # Filter out events that are not in the decrease interval
        event_list = [
            event
            for event in self.metrics
            if event.intervals_overlap(decrease_interval)
        ]
        if event_list:
            self_time = torch.tensor(
                [self.metrics[event].self_time_ns for event in event_list],
                dtype=torch.float32,
````
- **EN**: This chunk continues `rank_events` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `rank_events`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 329-348 / 第 329-348 行
````python
            )
            idle_time = torch.tensor(
                [self.metrics[event].fraction_idle_time for event in event_list],
                dtype=torch.float32,
            )
            normalized_gain = (idle_time - torch.mean(idle_time)) / torch.std(idle_time)
            normalized_self = (self_time - torch.mean(self_time)) / torch.std(self_time)
            heuristic_score_list = normalized_gain + 0.6 * normalized_self

            # Sort events by heuristic
            event_list = [
                event
                for _, event in sorted(
                    zip(heuristic_score_list, event_list, strict=True),
                    key=operator.itemgetter(0),
                    reverse=True,
                )
            ]
            event_list = event_list[:length]
        return event_list
````
- **EN**: This chunk continues `rank_events` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `rank_events`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 350-368 / 第 350-368 行
````python
    def get_optimizable_events(self, length: int = 1, print_enable: bool = True):
        event_list = self.rank_events(length)
        if not print_enable:
            return event_list
        output = "Optimizable events:\n" if event_list else "No events to optimize\n"

        output += "\n".join(
            [
                f"""{"-" * 80}
Event:                {event}
Source code location: {source_code_location(event.event)}
Percentage idle time: {self.metrics[event].fraction_idle_time * 100:.2f}%
{"-" * 80}"""
                for event in event_list
            ]
        )
        if print_enable:
            print(output)
        return event_list
````
- **EN**: This chunk defines `get_optimizable_events`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_optimizable_events`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 371-384 / 第 371-384 行
````python
def index_of_first_match(seq, predicate, start=0, end=None):
    if end is None or end >= len(seq):
        end = len(seq)
    for i in range(start, end):
        if predicate(seq[i]):
            return i
    return None


def argmax(seq, key=lambda x: x, start=0, end=None):
    seq = seq[start:end]
    if len(seq) == 0:
        return None
    return seq.index(max(seq, key=key)) + start
````
- **EN**: This chunk defines `argmax`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `argmax`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 387-405 / 第 387-405 行
````python
def source_code_location(event):
    while event is not None:
        match = re.search(r"\.py\(.*\)", event.name)
        if match is None:
            event = event.parent
            continue
        return event.name
    return "No source code location found"


# Provide an OSS workaround for cudagraphs + CUPTI issue
# https://github.com/pytorch/pytorch/issues/75504
# TODO(dberard) - deprecate / remove workaround for CUDA >= 12, when
# we stop supporting older CUDA versions.
def _init_for_cuda_graphs() -> None:
    from torch.autograd.profiler import profile

    with profile():
        pass
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.autograd.profiler. This chunk defines `_init_for_cuda_graphs`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.autograd.profiler。 这一段定义了 `_init_for_cuda_graphs`，其作用是协调 tracing、捕获或编译所需的图相关状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 408-426 / 第 408-426 行
````python
@dataclass
class TimelineEvent:
    """Represents an event in the profiler timeline."""

    timestamp: int
    event_type: Literal["start", "end", "regular"]
    marker_type: Literal["filename", "node"] | None
    identifier: str | int | None
    event: dict[str, Any]


@dataclass
class ContextStackEntry:
    """Represents a context (filename or node) in the stack."""

    context_type: Literal["filename", "node"]
    identifier: str | int
    metadata: dict | None
    tid: int | None = None  # Thread ID associated with this context
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. It introduces or extends `TimelineEvent`, `ContextStackEntry`, which hold the main object-oriented state for this portion of the file. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 它引入或扩展了 `TimelineEvent`、`ContextStackEntry`，这些类承载了本段涉及的主要面向对象状态。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 429-446 / 第 429-446 行
````python
def map_recorded_events_to_aten_ops_with_stack_trace(traced_data):
    """
    Maps recorded profiler events to their corresponding fx nodes and adds stack traces.

    Builds a timeline of all events (regular ops and FX markers for filenames/nodes),
    sorts by timestamp, then processes chronologically while maintaining a context stack of active
    filename/node scopes. Regular events are augmented with stack traces and node names from the
    innermost active context. Runtime is O(n log n) for n events.

    Args:
        traced_data: Json of profiler events from Chrome trace

    Returns:
        Dict mapping recorded event names to their aten operations with added stack traces
    """
    from torch.fx.traceback import _FX_METADATA_REGISTRY

    trace_events = traced_data.get("traceEvents", [])
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.traceback. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `map_recorded_events_to_aten_ops_with_stack_trace`, which coordinates graph-oriented state needed for tracing, capture, or compilation.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.traceback。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `map_recorded_events_to_aten_ops_with_stack_trace`，其作用是协调 tracing、捕获或编译所需的图相关状态。

### Lines 448-466 / 第 448-466 行
````python
    # Create event timeline
    event_timeline: list[TimelineEvent] = []

    def is_fx_marker_event(event):
        return (
            event.get("cat") == "cpu_op"
            and event.get("name", "").startswith("## ")
            and event.get("name", "").endswith(" ##")
        )

    def append_fx_marker_event(event_type, identifier, event):
        start_ts = event["ts"]
        end_ts = start_ts + event["dur"]
        event_timeline.append(
            TimelineEvent(start_ts, "start", event_type, identifier, event)
        )
        event_timeline.append(
            TimelineEvent(end_ts, "end", event_type, identifier, event)
        )
````
- **EN**: This chunk defines `append_fx_marker_event`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `append_fx_marker_event`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 468-482 / 第 468-482 行
````python
    for event in trace_events:
        if "ts" not in event or "dur" not in event:
            continue

        if is_fx_marker_event(event):
            content = event["name"][3:-3]

            if content.endswith(".py"):
                append_fx_marker_event("filename", content, event)
            else:
                try:
                    node_index = int(content)
                except ValueError:
                    pass
                append_fx_marker_event("node", node_index, event)  # type: ignore[possibly-undefined]
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 484-500 / 第 484-500 行
````python
        else:
            # Regular event that needs augmentation
            start_ts = event["ts"]
            event_timeline.append(TimelineEvent(start_ts, "regular", None, None, event))

    # Sort by timestamp
    event_timeline.sort(key=lambda x: x.timestamp)

    # Process events in chronological order with a stack
    context_stack: list[ContextStackEntry] = []

    # Invariant: all start event has a corresponding end event
    for timeline_event in event_timeline:
        match timeline_event.event_type:
            case "start":
                if timeline_event.identifier is None:
                    raise AssertionError("identifier must not be None for start event")
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 502-521 / 第 502-521 行
````python
                if timeline_event.marker_type == "filename":
                    if not isinstance(timeline_event.identifier, str):
                        raise AssertionError(
                            f"identifier must be str for filename marker, "
                            f"got {type(timeline_event.identifier).__name__}"
                        )
                    # Push filename context - query metadata registry on-demand
                    metadata = _FX_METADATA_REGISTRY.get(timeline_event.identifier)
                    tid = timeline_event.event.get("tid")
                    context_stack.append(
                        ContextStackEntry(
                            "filename", timeline_event.identifier, metadata, tid
                        )
                    )
                elif timeline_event.marker_type == "node":
                    # Find the current filename from stack
                    current_file_metadata = None
                    tid = timeline_event.event.get("tid")
                    for ctx_entry in reversed(context_stack):
                        if (
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 522-538 / 第 522-538 行
````python
                            ctx_entry.context_type == "filename"
                            and ctx_entry.tid == tid
                        ):
                            current_file_metadata = ctx_entry.metadata
                            break

                    if current_file_metadata:
                        node_metadata = current_file_metadata.get("node_metadata", {})
                        if timeline_event.identifier in node_metadata:
                            node_meta: dict | None = node_metadata[
                                timeline_event.identifier
                            ]
                            context_stack.append(
                                ContextStackEntry(
                                    "node", timeline_event.identifier, node_meta, tid
                                )
                            )
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 540-557 / 第 540-557 行
````python
            case "end":
                # Pop from stack - search backwards to find matching context
                for i in range(len(context_stack) - 1, -1, -1):
                    ctx_entry = context_stack[i]
                    if (
                        timeline_event.marker_type == ctx_entry.context_type
                        and timeline_event.identifier == ctx_entry.identifier
                    ):
                        context_stack.pop(i)
                        break

            case "regular":
                # Apply metadata from current context stack
                # Find the most specific context (node takes precedence over filename)
                # Only augment events with the same tid as the file/node event matched
                current_stack_trace = None
                current_node_name = None
                event_tid = timeline_event.event.get("tid")
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 559-577 / 第 559-577 行
````python
                for ctx_entry in reversed(context_stack):
                    # Only apply metadata from contexts with matching tid
                    if ctx_entry.tid == event_tid:
                        if ctx_entry.context_type == "node" and ctx_entry.metadata:
                            current_stack_trace = ctx_entry.metadata.get(
                                "stack_trace", "No model stack trace available"
                            )
                            current_node_name = ctx_entry.metadata.get("name", "")
                            # Do we want to only attach the stack trace of the lowest node or stack trace of all nodes
                            # if nodes are nested, e.g. in nested graph modules
                            break

                # Augment the event
                if current_stack_trace or current_node_name:
                    args = timeline_event.event.setdefault("args", {})
                    if current_stack_trace:
                        args["stack_trace"] = current_stack_trace
                    if current_node_name:
                        args["node_name"] = current_node_name
````
- **EN**: This chunk continues `append_fx_marker_event` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `append_fx_marker_event`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Performance tracing**
  - EN: Captures operator-level timing and trace metadata for performance diagnosis.
  - CN: 捕获算子级时序与跟踪元数据，用于性能诊断。
- **_traverse**
  - EN: `_traverse` is one of the main symbols declared or implemented in this file.
  - CN: `_traverse` 是本文件声明或实现的主要符号之一。
- **EventMetrics**
  - EN: `EventMetrics` is one of the main symbols declared or implemented in this file.
  - CN: `EventMetrics` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch.autograd.profiler`, `torch.profiler`, `torch.autograd`, `torch`, `torch.fx.traceback`
- **Standard library / 标准库**: `functools`, `operator`, `re`, `collections`, `dataclasses`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `_traverse`, `EventMetrics`, `Interval`, `EventKey`, `BasicEvaluation`, `index_of_first_match`, `argmax`, `source_code_location`, `_init_for_cuda_graphs`, `TimelineEvent`
