# profiler_legacy.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/autograd/profiler_legacy.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements automatic-differentiation helpers, gradient APIs, profiling hooks, and graph-facing utilities.
- **Purpose (CN)**: 实现自动求导辅助逻辑、梯度 API、profiling 钩子以及面向计算图的工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import itertools
import warnings
from typing_extensions import deprecated

import torch
import torch.cuda
from torch.autograd import (
    _disable_profiler_legacy,
    _enable_profiler_legacy,
    DeviceType,
    ProfilerConfig,
    ProfilerState,
)
from torch.autograd.profiler_util import (
    _filter_name,
    _filter_stack_entry,
    _rewrite_name,
    EventList,
    FunctionEvent,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.cuda, torch.autograd, ...; standard-library helpers such as itertools, warnings; other helper packages such as typing_extensions.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.cuda、torch.autograd、...；标准库辅助模块，如 itertools、warnings；其他辅助包，如 typing_extensions。

### Lines 21-34 / 第 21-34 行
````python
    MEMORY_EVENT_NAME,
)


__all__ = ["profile"]


@deprecated(
    "`torch.autograd.profiler_legacy.profile` is deprecated and will be removed in a future release. "
    "Please use `torch.profiler` instead.",
    category=None,  # TODO: change to `FutureWarning`
)
class profile:
    """DEPRECATED: use torch.profiler instead."""
````
- **EN**: `__all__` defines the public symbols that this module chooses to export. It introduces or extends `profile`, which hold the main object-oriented state for this portion of the file. This chunk continues `profile` and expands its internal control flow or state updates. Decorators such as `deprecated` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。 它引入或扩展了 `profile`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `profile`，进一步展开其内部控制流或状态更新。 像 `deprecated` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 36-55 / 第 36-55 行
````python
    def __init__(
        self,
        enabled=True,
        *,
        use_cuda=False,
        record_shapes=False,
        with_flops=False,
        profile_memory=False,
        with_stack=False,
        with_modules=False,
    ):
        self.enabled: bool = enabled
        if not self.enabled:
            return
        self.use_cuda = use_cuda
        self.function_events = None
        self.entered = False
        self.record_shapes = record_shapes
        self.with_flops = with_flops
        self.record_shapes |= self.with_flops
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 56-70 / 第 56-70 行
````python
        self.profile_memory = profile_memory
        self.with_stack = with_stack
        self.with_modules = with_modules

        if self.use_cuda and not torch.cuda.is_available():
            warnings.warn(
                "CUDA is not available, disabling CUDA profiling",
                stacklevel=2,
            )
            self.use_cuda = False

        if self.use_cuda:
            self.profiler_kind = ProfilerState.CUDA
        else:
            self.profiler_kind = ProfilerState.CPU
````
- **EN**: This chunk continues `__init__` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 72-91 / 第 72-91 行
````python
    def config(self):
        return ProfilerConfig(
            self.profiler_kind,
            self.record_shapes,
            self.profile_memory,
            self.with_stack,
            self.with_flops,
            self.with_modules,
            # avoid exposing _ExperimentalConfig this in legacy public API
            torch._C._profiler._ExperimentalConfig(),
        )

    def __enter__(self):
        if not self.enabled:
            return
        if self.entered:
            raise RuntimeError("Profiler context manager is not reentrant")
        self.entered = True
        self._start_trace()
        return self
````
- **EN**: This chunk defines `__enter__`, which implements context-manager boundaries around temporary runtime state. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__enter__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 93-112 / 第 93-112 行
````python
    def _start_trace(self):
        _enable_profiler_legacy(self.config())

    def __exit__(self, exc_type, exc_val, exc_tb):
        if not self.enabled:
            return
        if self.use_cuda:
            torch.cuda.synchronize()

        records = _disable_profiler_legacy()
        parsed_results = _parse_legacy_records(records)
        # pyrefly: ignore [bad-assignment]
        self.function_events = EventList(
            parsed_results,
            use_device="cuda" if self.use_cuda else None,
            profile_memory=self.profile_memory,
            with_flops=self.with_flops,
        )
        # pyrefly: ignore [missing-attribute]
        self.function_events._build_tree()
````
- **EN**: This chunk defines `__exit__`, which implements context-manager boundaries around temporary runtime state. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__exit__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 113-127 / 第 113-127 行
````python
        return False

    def __repr__(self):
        if self.function_events is None:
            return "<unfinished profiler_legacy.profile>"
        return repr(self.function_events)

    def __str__(self):
        if self.function_events is None:
            return "<unfinished profile.profiler_legacy.profile>"
        return str(self.function_events)

    def _check_finish(self):
        if self.function_events is None:
            raise RuntimeError("Profiler didn't finish running")
````
- **EN**: This chunk defines `_check_finish`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_finish`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-148 / 第 129-148 行
````python
    def table(
        self,
        sort_by=None,
        row_limit=100,
        max_src_column_width=75,
        max_name_column_width=55,
        max_shapes_column_width=80,
        header=None,
        top_level_events_only=False,
    ):
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        return self.function_events.table(
            sort_by=sort_by,
            row_limit=row_limit,
            max_src_column_width=max_src_column_width,
            max_name_column_width=max_name_column_width,
            max_shapes_column_width=max_shapes_column_width,
            header=header,
````
- **EN**: This chunk defines `table`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `table`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 149-168 / 第 149-168 行
````python
            top_level_events_only=top_level_events_only,
        )

    table.__doc__ = EventList.table.__doc__

    def export_chrome_trace(self, path):
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        return self.function_events.export_chrome_trace(path)

    export_chrome_trace.__doc__ = EventList.export_chrome_trace.__doc__

    def export_stacks(self, path: str, metric: str = "self_cpu_time_total"):
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        if not self.with_stack:
            raise AssertionError("export_stacks() requires with_stack=True")
        return self.function_events.export_stacks(path, metric)
````
- **EN**: This chunk defines `export_stacks`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `export_stacks`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 170-184 / 第 170-184 行
````python
    def key_averages(self, group_by_input_shape=False, group_by_stack_n=0):
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        return self.function_events.key_averages(group_by_input_shape, group_by_stack_n)

    key_averages.__doc__ = EventList.key_averages.__doc__

    def total_average(self):
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        return self.function_events.total_average()

    total_average.__doc__ = EventList.total_average.__doc__
````
- **EN**: This chunk defines `total_average`, which implements a focused step in autograd bookkeeping or gradient propagation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `total_average`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 186-201 / 第 186-201 行
````python
    @property
    def self_cpu_time_total(self):
        """Return CPU time as the sum of self times across all events."""
        self._check_finish()
        if self.function_events is None:
            raise AssertionError("Expected profiling results")
        return self.function_events.self_cpu_time_total


def _parse_legacy_records(thread_records):
    def _get_record_key(record):
        """Return a tuple for correlating start and end records in `_parse_legacy_records`."""
        return (record.handle(), record.node_id())

    start_record = None
    functions = []
````
- **EN**: This chunk defines `_get_record_key`, which implements a focused step in autograd bookkeeping or gradient propagation. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_record_key`，其作用是实现自动求导记账或梯度传播中的一个关键步骤。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 203-217 / 第 203-217 行
````python
    # '__start_profile' is not guaranteed to be first, so we must find it here
    for record in itertools.chain.from_iterable(thread_records):
        name = record.name()
        if start_record is None and name == "__start_profile":
            start_record = record

    if start_record is None or start_record.is_remote():
        raise AssertionError("Expected a valid local start_record")

    for thread_record_list in thread_records:
        # accumulated memory allocations per handle
        cpu_memory_allocs = {}
        cuda_memory_allocs = {}
        # ranges per handle
        range_starts = {}
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 219-238 / 第 219-238 行
````python
        filtered_handles = set()
        prev_record = None
        for record in thread_record_list:
            record_key = _get_record_key(record)
            if _filter_name(record.name()) or record_key in filtered_handles:
                filtered_handles.add(record_key)
                continue

            if record.kind() == "push":
                # workaround to reduce double logging from operator
                # wrappers and redispatch
                if prev_record is not None:
                    duplicate = (
                        prev_record.name() == record.name()
                        and prev_record.kind() == record.kind()
                        and prev_record.node_id() == record.node_id()
                    )
                    if duplicate:
                        filtered_handles.add(record_key)
                        continue
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 240-256 / 第 240-256 行
````python
                range_starts[record_key] = record
                cpu_memory_allocs[record_key] = 0
                cuda_memory_allocs[record_key] = 0
            elif record.kind() == "pop":
                if record_key not in range_starts:
                    raise AssertionError(
                        f"Expected record with key {record_key} to exist in range_starts. "
                        "This means that the pop event did not have a corresponding push."
                    )

                start = range_starts[record_key]

                cpu_memory_usage = cpu_memory_allocs[record_key]
                cuda_memory_usage = cuda_memory_allocs[record_key]
                is_async = start.is_async() or (start.thread_id() != record.thread_id())
                is_remote_event = record.is_remote()
                start_flops = start.flops()
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 258-277 / 第 258-277 行
````python
                fe = FunctionEvent(
                    id=record.handle(),
                    node_id=record.node_id(),
                    name=_rewrite_name(name=start.name(), with_wildcard=True),
                    trace_name=_rewrite_name(name=start.name(), with_wildcard=False),
                    thread=start.thread_id(),
                    start_us=start_record.cpu_elapsed_us(start),
                    end_us=start_record.cpu_elapsed_us(record),
                    fwd_thread=start.fwd_thread_id(),
                    input_shapes=start.shapes(),
                    stack=[
                        entry for entry in start.stack() if _filter_stack_entry(entry)
                    ],
                    scope=start.scope(),
                    use_device="cuda" if start.has_cuda() else None,
                    cpu_memory_usage=cpu_memory_usage,
                    device_memory_usage=cuda_memory_usage,
                    is_async=is_async,
                    is_remote=is_remote_event,
                    sequence_nr=start.sequence_nr(),
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 278-297 / 第 278-297 行
````python
                    device_type=DeviceType.CPU,
                    is_legacy=True,
                    flops=start_flops,
                )
                # note: async events have only cpu total time
                if not is_async and start.has_cuda():
                    duration = start.cuda_elapsed_us(record)
                    if duration > 0:
                        fe.append_kernel(start.name(), start.device(), duration)
                functions.append(fe)
                del range_starts[record_key]
                del cpu_memory_allocs[record_key]
                del cuda_memory_allocs[record_key]
            elif record.kind() == "memory_alloc":
                num_open_handles_cpu = len(cpu_memory_allocs)
                num_open_handles_cuda = len(cuda_memory_allocs)
                if num_open_handles_cpu != num_open_handles_cuda:
                    raise AssertionError(
                        f"Expected CPU and CUDA memory allocation handles to match, "
                        f"but got {num_open_handles_cpu} CPU and {num_open_handles_cuda} CUDA"
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 298-317 / 第 298-317 行
````python
                    )
                for handle in cpu_memory_allocs:
                    cpu_memory_allocs[handle] += record.cpu_memory_usage()
                for handle in cuda_memory_allocs:
                    cuda_memory_allocs[handle] += record.cuda_memory_usage()
                if num_open_handles_cpu == 0:
                    # output event as a top-level memory event
                    fe = FunctionEvent(
                        id=0,
                        name=MEMORY_EVENT_NAME,
                        trace_name=None,
                        thread=0,
                        start_us=0,
                        end_us=0,
                        stack=[],
                        cpu_memory_usage=record.cpu_memory_usage(),
                        device_memory_usage=record.cuda_memory_usage(),
                        is_legacy=True,
                    )
                    functions.append(fe)
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 318-327 / 第 318-327 行
````python
            prev_record = record

    # Sort functions by start time then by end time ascending.
    # This ensures that--in the case of nested events which
    # have the same start time (which may happen due to the
    # granularity of the given clock tick)--we always show
    # the outermost nested call first. This adds stability
    # in how FunctionEvents appear
    functions.sort(key=lambda evt: [evt.time_range.start, -evt.time_range.end])
    return functions
````
- **EN**: This chunk continues `_get_record_key` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_get_record_key`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Autograd engine**
  - EN: Carries saved tensors, gradient metadata, and APIs that connect forward execution to backward logic.
  - CN: 承载保存张量、梯度元数据以及连接前向与反向逻辑的 API。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **profile**
  - EN: `profile` is one of the main symbols declared or implemented in this file.
  - CN: `profile` 是本文件声明或实现的主要符号之一。
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

- **Internal torch modules / torch 内部模块**: `torch`, `torch.cuda`, `torch.autograd`, `torch.autograd.profiler_util`
- **Standard library / 标准库**: `itertools`, `warnings`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `profile`, `_parse_legacy_records`
