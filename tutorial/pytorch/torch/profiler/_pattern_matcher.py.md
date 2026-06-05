# _pattern_matcher.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/profiler/_pattern_matcher.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements profiling APIs, trace utilities, and performance-analysis helpers.
- **Purpose (CN)**: 实现 profiling API、跟踪工具以及性能分析辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-26 / 第 1-26 行
````python
# mypy: allow-untyped-defs
import json
import math
import os
import re

import torch
import torch.utils.benchmark as benchmark
from torch._C._profiler import (
    _EventType,
    _ExtraFields_PyCall,
    _ExtraFields_PyCCall,
    _ExtraFields_TorchOp,
    _ProfilerEvent,
)
from torch.profiler import profile
from torch.profiler._utils import index_of_first_match, traverse_bfs, traverse_dfs


class Pattern:
    """
    Base class for all patterns, subclass this class and implement match()
    to define custom patterns.

    In subclass, define description and skip property.
    """
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.utils.benchmark, torch._C._profiler, ...; standard-library helpers such as json, math, os, .... It introduces or extends `Pattern`, which hold the main object-oriented state for this portion of the file.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.utils.benchmark、torch._C._profiler、...；标准库辅助模块，如 json、math、os、...。 它引入或扩展了 `Pattern`，这些类承载了本段涉及的主要面向对象状态。

### Lines 28-49 / 第 28-49 行
````python
    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        self.prof = prof
        self.should_benchmark = should_benchmark
        self.name = "Please specify a name for pattern"
        self.description = "Please specify a description for pattern"
        self.url = ""
        if prof.profiler is None or prof.profiler.kineto_results is None:
            raise AssertionError("profiler and kineto_results must not be None")
        self.event_tree = prof.profiler.kineto_results.experimental_event_tree()
        self.tid_root: dict[int, list[_ProfilerEvent]] = {}
        for event in self.event_tree:
            self.tid_root.setdefault(event.start_tid, []).append(event)

    @property
    def skip(self) -> bool:
        return False

    def report(self, event: _ProfilerEvent):
        msg = (
            f"{self.description}\n[Source Code Location] {source_code_location(event)}"
        )
        return msg
````
- **EN**: This chunk defines `report`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `report`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 51-76 / 第 51-76 行
````python
    def eventTreeTraversal(self):
        """
        Traverse the event tree and yield all events.
        Override this method in subclass to customize the traversal.
        """
        yield from traverse_dfs(self.event_tree)

    def summary(self, events: list[_ProfilerEvent]):
        default_summary = f"{self.name}: {len(events)} events matched."
        if self.should_benchmark:
            # If benchmark summary is not empty, use it.
            return (
                self.benchmark_summary(events)
                if hasattr(self, "benchmark")  # type: ignore[attr-defined]
                else default_summary
            )
        return default_summary

    def benchmark_summary(self, events: list[_ProfilerEvent]) -> str:
        def format_time(time_ns: int) -> str:
            unit_lst = ["ns", "us", "ms"]
            for unit in unit_lst:
                if time_ns < 1000:
                    return f"{time_ns:.2f} {unit}"
                time_ns //= 1000
            return f"{time_ns:.2f} s"
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `format_time`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `format_time`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 78-104 / 第 78-104 行
````python
        if not hasattr(self, "benchmark"):
            raise AssertionError("Please implement benchmark()")
        shapes_factor_map = self.benchmark(events)  # type: ignore[attr-defined]
        original_time = sum(event.duration_time_ns for event in events)
        new_time = sum(
            shapes_factor_map[input_shapes(event)] * event.duration_time_ns
            for event in events
        )
        return (
            f"{self.name}: {len(events)} events matched. "
            f"Total Estimated Speedup: {format_time(original_time - new_time)} ({round(original_time / new_time, 2)}X)"
        )

    def match(self, event: _ProfilerEvent):
        """
        Return True if the event matches the pattern.
        This method should be overridden in subclass.
        """
        raise NotImplementedError

    def matched_events(self):
        if self.skip:
            return []
        matched_events = [
            event for event in self.eventTreeTraversal() if self.match(event)
        ]
        return matched_events
````
- **EN**: This chunk defines `matched_events`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `matched_events`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 106-132 / 第 106-132 行
````python
    def root_of(self, event: _ProfilerEvent):
        while event.parent:
            event = event.parent
        return event

    def siblings_of(self, event: _ProfilerEvent):
        if event.parent:
            children = event.parent.children
        else:
            children = self.tid_root[event.start_tid]
        index = children.index(event)
        return children[:index], children[index + 1 :]

    def next_of(self, event: _ProfilerEvent):
        _, next_events = self.siblings_of(event)
        return next_events[0] if next_events else None

    def prev_of(self, event: _ProfilerEvent):
        prev_events, _ = self.siblings_of(event)
        return prev_events[-1] if prev_events else None

    def go_up_until(self, event: _ProfilerEvent, predicate):
        if not event:
            return None
        while event.parent and not predicate(event):
            event = event.parent
        return event
````
- **EN**: This chunk defines `go_up_until`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `go_up_until`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 135-158 / 第 135-158 行
````python
# Patterns


class NamePattern(Pattern):
    def __init__(
        self, prof: profile, name: str, should_benchmark: bool = False
    ) -> None:
        super().__init__(prof, should_benchmark)
        self.description = f"Matched Name Event: {name}"
        self.name = name

    def match(self, event: _ProfilerEvent):
        return re.search(self.name, event.name) is not None


class ExtraCUDACopyPattern(Pattern):
    """
    This pattern identifies if we creates a constant tensor on CPU and immediately moves it to GPU.
    example: torch.zeros((100, 100)).to("cuda")

    Pattern:
    built-in method                 |built-in method
        ...                         |    aten::to
            aten::fill_/aten::zero_ |        aten::_to_copy
````
- **EN**: It introduces or extends `NamePattern`, `ExtraCUDACopyPattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `match`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `NamePattern`、`ExtraCUDACopyPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `match`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 160-181 / 第 160-181 行
````python
    Algorithm:
    We start at node aten::to, go parent events' previous events,
    and check if we have a aten::fill_/aten::zero_ as we keep going down the tree.
    We always select the last child in the children list when we go down the tree.
    If at any step we failed, it is not a match.
    """

    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Extra CUDA Copy Pattern"
        self.description = "Filled a CPU tensor and immediately moved it to GPU. Please initialize it on GPU."
        self.url = "https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html#create-tensors-directly-on-the-target-device"
        self.init_ops = {
            "aten::fill_",
            "aten::zero_",
            "aten::normal_",
            "aten::uniform_",
        }

    @property
    def skip(self) -> bool:
        return not self.prof.with_stack or not self.prof.record_shapes
````
- **EN**: This chunk defines `skip`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `skip`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 183-210 / 第 183-210 行
````python
    def match(self, event):
        # TODO: We should also check tensor identities
        if event.name != "aten::to":
            return False
        to_event = event
        if not event.children:
            return False
        event = event.children[-1]
        if event.name != "aten::_to_copy":
            return False
        if not event.children:
            return False
        event = event.children[-1]
        if event.name != "aten::copy_":
            return False
        # aten::copy_ should have the first 2 args dtype the same
        dtypes = input_dtypes(event)
        if len(dtypes) < 2:
            return False
        if dtypes[0] is None or dtypes[0] != dtypes[1]:
            return False
        event = to_event
        # Up one level
        event = event.parent
        if event is None:
            return False
        # Check if we have a aten::fill_ in previous leaf
        event = self.prev_of(event)
````
- **EN**: This chunk defines `match`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `match`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 211-234 / 第 211-234 行
````python
        if event is None:
            return False
        while event.children:
            event = event.children[-1]
            # aten::zero_ is a special optimization case where fill_ is not called
            if event.name in self.init_ops:
                return True
        return event.name in self.init_ops
        # TODO: Check if tensor is reused

    def benchmark(self, events: list[_ProfilerEvent]):
        shapes_factor_map = {input_shapes(event): 0.0 for event in events}
        for shape in shapes_factor_map:
            size = shape[0]
            to_timer = benchmark.Timer(
                stmt='torch.ones(size).to("cuda")', globals={"size": size}
            )
            de_timer = benchmark.Timer(
                stmt='torch.ones(size, device="cuda")', globals={"size": size}
            )
            to_time = to_timer.timeit(10).mean
            de_time = de_timer.timeit(10).mean
            shapes_factor_map[shape] = de_time / to_time
        return shapes_factor_map
````
- **EN**: This chunk defines `benchmark`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `benchmark`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 237-258 / 第 237-258 行
````python
class ForLoopIndexingPattern(Pattern):
    """
    This pattern identifies if we use a for loop to index a tensor that
    can be vectorized.
    example:
    tensor = torch.empty((100, 100))
    for i in range(100):
        tensor[i] = i

    Pattern:
    aten::select | ... | aten::select | ... (Repeat)

    Algorithm:
    We start at node aten::select, and we check if we can find this alternating patterns.
    We also keep a dictionary to avoid duplicate match in the for loop.
    """

    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "For Loop Indexing Pattern"
        self.description = "For loop indexing detected. Vectorization recommended."
        self.visited: set[int] = set()
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `ForLoopIndexingPattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `ForLoopIndexingPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 260-283 / 第 260-283 行
````python
    def eventTreeTraversal(self):
        """
        We need to use BFS traversal order to avoid duplicate match.
        """
        yield from traverse_bfs(self.event_tree)

    def match(self, event: _ProfilerEvent):
        if event.name != "aten::select":
            return False
        if event.id in self.visited:
            return False
        repeat_count = 1
        _, next = self.siblings_of(event)
        if len(next) <= 1:
            return False

        # Custom event list matching
        def same_ops(list1, list2) -> bool:
            if len(list1) != len(list2):
                return False
            for op1, op2 in zip(list1, list2, strict=True):
                if op1.name != op2.name:
                    return False
            return True
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `same_ops`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `same_ops`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 285-308 / 第 285-308 行
````python
        # Record the ops between two aten::select
        next_select_idx = index_of_first_match(next, lambda e: e.name == "aten::select")
        if next_select_idx is None:
            return False
        indexing_ops = [event] + next[:next_select_idx]
        next = next[len(indexing_ops) - 1 :]
        for i in range(0, len(next), len(indexing_ops)):
            if same_ops(indexing_ops, next[i : i + len(indexing_ops)]):
                repeat_count += 1
                self.visited.add(next[i].id)
            else:
                break
        return repeat_count >= 10


class FP32MatMulPattern(Pattern):
    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "FP32 MatMul Pattern"
        self.description = (
            "You are currently using GPU that supports TF32. "
            "Please enable TF32 by setting 'torch.backends.cuda.matmul.allow_tf32 = True'"
        )
        self.url = "https://pytorch.org/docs/stable/notes/cuda.html#tensorfloat-32-tf32-on-ampere-devices"
````
- **EN**: It introduces or extends `FP32MatMulPattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `FP32MatMulPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 310-336 / 第 310-336 行
````python
    @property
    def skip(self):
        if torch.version.hip is not None:
            has_tf32 = False
        else:
            # Anything less than sm_80 is not Ampere which doesn't support TF32
            has_tf32 = all(
                int(re.sub("sm_|compute_", "", arch)) >= 80
                for arch in torch.cuda.get_arch_list()
            )
        return has_tf32 is False or super().skip or not self.prof.record_shapes

    def match(self, event: _ProfilerEvent) -> bool:
        # If we saw this pattern once, we don't need to match it again
        if event.tag != _EventType.TorchOp:
            return False
        if not isinstance(event.extra_fields, _ExtraFields_TorchOp):
            raise AssertionError(
                f"expected _ExtraFields_TorchOp, got {type(event.extra_fields).__name__}"
            )
        if event.name == "aten::mm":
            if event.extra_fields.allow_tf32_cublas is False:
                return True
        return False

    def report(self, event: _ProfilerEvent):
        return self.description
````
- **EN**: This chunk defines `report`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `report`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 338-356 / 第 338-356 行
````python
    def benchmark(self, events: list[_ProfilerEvent]):
        shapes_factor_map = {input_shapes(event): 0.0 for event in events}
        for shape in shapes_factor_map:
            matrixA = torch.randn(shape[0], device="cuda", dtype=torch.float32)
            matrixB = torch.randn(shape[1], device="cuda", dtype=torch.float32)
            fp32_timer = benchmark.Timer(
                stmt="torch.mm(matrixA, matrixB)",
                globals={"matrixA": matrixA, "matrixB": matrixB},
            )
            tf32_timer = benchmark.Timer(
                stmt="torch.mm(matrixA, matrixB)",
                setup="torch.backends.cuda.matmul.allow_tf32 = True",
                globals={"matrixA": matrixA, "matrixB": matrixB},
            )
            torch.backends.cuda.matmul.allow_tf32 = False
            fp32_time = fp32_timer.timeit(10).mean
            tf32_time = tf32_timer.timeit(10).mean
            shapes_factor_map[shape] = tf32_time / fp32_time
        return shapes_factor_map
````
- **EN**: This chunk defines `benchmark`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `benchmark`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 359-382 / 第 359-382 行
````python
class OptimizerSingleTensorPattern(Pattern):
    """
    This pattern identifies if we are using the single-tensor version of an optimizer.
    example:
    optimizer = torch.optim.SGD(model.parameters(), lr=0.1)
    By adding foreach=True to enable multi-tensor optimizer, we can gain speedup when
    the kernels are relatively small.

    Pattern:
    XXXXX: _single_tenser_<OPTIMIZER_NAME>

    Algorithm:
    String match
    """

    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Optimizer Single Tensor Pattern"
        self.optimizers_with_foreach = ["adam", "sgd", "adamw"]
        self.description = (
            "Detected optimizer running with single tensor implementation. "
            "Please enable multi tensor implementation by passing 'foreach=True' into optimizer."
        )
        self.url = ""
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `OptimizerSingleTensorPattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `OptimizerSingleTensorPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 384-407 / 第 384-407 行
````python
    def match(self, event: _ProfilerEvent) -> bool:
        for optimizer in self.optimizers_with_foreach:
            if event.name.endswith(f"_single_tensor_{optimizer}"):
                return True
        return False


class SynchronizedDataLoaderPattern(Pattern):
    """
    This pattern identifies if we are using num_workers=0 in DataLoader.
    example:
    torch.utils.data.DataLoader(dataset, batch_size=batch_size)
    Add num_workers=N to the arguments. N depends on system configuration.

    Pattern:
    dataloader.py(...): __iter__
        dataloader.py(...): _get_iterator
            NOT dataloader.py(...): check_worker_number_rationality

    Algorithm:
    If we don't see check_worker_number_rationality call in the dataloader __iter__,
    It is not an asynchronous dataloader.

    """
````
- **EN**: It introduces or extends `SynchronizedDataLoaderPattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `match`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `SynchronizedDataLoaderPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `match`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 409-433 / 第 409-433 行
````python
    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Synchronized DataLoader Pattern"
        self.description = (
            "Detected DataLoader running with synchronized implementation. "
            "Please enable asynchronous dataloading by setting num_workers > 0 when initializing DataLoader."
        )
        self.url = (
            "https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html"
            "#enable-async-data-loading-and-augmentation"
        )

    def match(self, event: _ProfilerEvent) -> bool:
        def is_dataloader_function(name: str, function_name: str):
            return name.startswith(
                os.path.join("torch", "utils", "data", "dataloader.py")
            ) and name.endswith(function_name)

        # TODO: fixme! Due to lifetime issues of the function name, this field might
        # actually point to an already freed string when the even is a PyCall.
        # Just silently skip this to unblock testing.
        try:
            event.name
        except UnicodeDecodeError:
            return False
````
- **EN**: This chunk defines `is_dataloader_function`, which checks a capability or invariant before later code relies on it. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_dataloader_function`，其作用是检查某项能力或不变量，供后续逻辑依赖。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 435-459 / 第 435-459 行
````python
        if not is_dataloader_function(event.name, "__iter__"):
            return False
        if not event.children:
            return False
        event = event.children[0]
        if not is_dataloader_function(event.name, "_get_iterator"):
            return False
        if not event.children:
            return False
        event = event.children[0]
        return not is_dataloader_function(event.name, "check_worker_number_rationality")
        # TODO: We should also check if the loader is bottleneck.


class GradNotSetToNonePattern(Pattern):
    """
    This pattern identifies if we are not setting grad to None in zero_grad.
    example:
    optimizer.zero_grad()
    By setting set_to_none=True, we can gain speedup

    Pattern:
    XXXXX: _zero_grad
        NOT aten::zeros
            aten::zero_
````
- **EN**: It introduces or extends `GradNotSetToNonePattern`, which hold the main object-oriented state for this portion of the file. This chunk continues `GradNotSetToNonePattern` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `GradNotSetToNonePattern`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `GradNotSetToNonePattern`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 461-484 / 第 461-484 行
````python
    aten::zero_ is called on each parameter in the model.
    We also want to make sure it is not called by aten::zeros.

    Algorithm:
    String match
    """

    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Gradient Set To Zero Instead of None Pattern"
        self.description = (
            "Detected gradient set to zero instead of None. "
            "Please add 'set_to_none=True' when calling zero_grad()."
        )
        self.url = (
            "https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html"
            "#disable-gradient-calculation-for-validation-or-inference"
        )

    def match(self, event: _ProfilerEvent) -> bool:
        if not event.name.endswith(": zero_grad"):
            return False
        if not event.children:
            return False
````
- **EN**: This chunk defines `match`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `match`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 486-507 / 第 486-507 行
````python
        for sub_event in traverse_dfs(event.children):
            if (
                sub_event.name == "aten::zero_"
                and sub_event.parent.name != "aten::zeros"
            ):
                return True
        # TODO: We should also check if the optimizer's numerical behavior will change.
        return False


class Conv2dBiasFollowedByBatchNorm2dPattern(Pattern):
    """
    This pattern identifies if we are enabling bias in Conv2d which is followed by BatchNorm2d.
    Bias doesn't do anything when followed by batchnorm.
    Pattern:
    nn.Module: Conv2d            | nn.Module: BatchNorm2d
        ...
            aten::conv2d AND dtype of third argument is not null
    The third argument is the bias
    Algorithm:
    String match
    """
````
- **EN**: It introduces or extends `Conv2dBiasFollowedByBatchNorm2dPattern`, which hold the main object-oriented state for this portion of the file. This chunk continues `Conv2dBiasFollowedByBatchNorm2dPattern` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `Conv2dBiasFollowedByBatchNorm2dPattern`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `Conv2dBiasFollowedByBatchNorm2dPattern`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 509-536 / 第 509-536 行
````python
    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Enabling Bias in Conv2d Followed By BatchNorm Pattern"
        self.description = "Detected bias enabled in Conv2d that is followed by BatchNorm2d. Please set 'bias=False' in Conv2d."
        self.url = (
            "https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html"
            "#disable-bias-for-convolutions-directly-followed-by-a-batch-norm"
        )

    @property
    def skip(self):
        return self.prof.record_shapes is False or super().skip

    def match(self, event: _ProfilerEvent):
        if event.name != "aten::conv2d":
            return False
        if len(input_dtypes(event)) < 3 or input_dtypes(event)[2] is None:
            return False
        # This means bias=True
        event = self.go_up_until(
            event, lambda e: e.name.startswith("nn.Module: Conv2d")
        )
        if not event:
            return False
        event = self.next_of(event)
        if not event:
            return False
        return event.name.startswith("nn.Module: BatchNorm2d")
````
- **EN**: This chunk defines `match`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `match`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 539-563 / 第 539-563 行
````python
class MatMulDimInFP16Pattern(Pattern):
    def __init__(self, prof: profile, should_benchmark: bool = False) -> None:
        super().__init__(prof, should_benchmark)
        self.name = "Matrix Multiplication Dimension Not Aligned Pattern"
        self.description = "Detected matmul with dimension not aligned. Please use matmul with aligned dimension."
        self.url = "https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html#use-mixed-precision-and-amp"

    @property
    def skip(self) -> bool:
        return not self.prof.with_stack or not self.prof.record_shapes

    def match(self, event: _ProfilerEvent) -> bool:
        def mutiple_of(shapes, multiple):
            return all(dim % multiple == 0 for shape in shapes for dim in shape[-2:])

        if event.name not in ("aten::mm", "aten::bmm", "aten::addmm"):
            return False
        if not input_dtypes(event):
            return False
        arg_dtype = input_dtypes(event)[0]
        if arg_dtype in (torch.bfloat16, torch.half) and not mutiple_of(
            input_shapes(event), 8
        ):
            return True
        return False
````
- **EN**: It introduces or extends `MatMulDimInFP16Pattern`, which hold the main object-oriented state for this portion of the file. This chunk defines `mutiple_of`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `MatMulDimInFP16Pattern`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `mutiple_of`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 565-590 / 第 565-590 行
````python
    def benchmark(self, events: list[_ProfilerEvent]):
        def closest_multiple(shapes, multiple):
            return [multiple * math.ceil(shape / multiple) for shape in shapes]

        shapes_factor_map = {input_shapes(event): 0.0 for event in events}
        for shape in shapes_factor_map:
            matrixA = torch.randn(shape[0], device="cuda", dtype=torch.float16)
            matrixB = torch.randn(shape[1], device="cuda", dtype=torch.float16)
            not_aligned_dim_timer = benchmark.Timer(
                stmt="torch.mm(matrixA, matrixB)",
                globals={"matrixA": matrixA, "matrixB": matrixB},
            )
            matrixA = torch.randn(
                closest_multiple(shape[0], 8), device="cuda", dtype=torch.float16
            )
            matrixB = torch.randn(
                closest_multiple(shape[1], 8), device="cuda", dtype=torch.float16
            )
            aligned_dim_timer = benchmark.Timer(
                stmt="torch.mm(matrixA, matrixB)",
                globals={"matrixA": matrixA, "matrixB": matrixB},
            )
            not_aligned_dim_time = not_aligned_dim_timer.timeit(10).mean
            aligned_dim_time = aligned_dim_timer.timeit(10).mean
            shapes_factor_map[shape] = aligned_dim_time / not_aligned_dim_time
        return shapes_factor_map
````
- **EN**: This chunk defines `closest_multiple`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `closest_multiple`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 593-614 / 第 593-614 行
````python
def source_code_location(event: _ProfilerEvent | None) -> str:
    while event:
        if event.tag == _EventType.PyCall or event.tag == _EventType.PyCCall:
            if not isinstance(
                event.extra_fields, (_ExtraFields_PyCall, _ExtraFields_PyCCall)
            ):
                raise AssertionError(
                    f"expected _ExtraFields_PyCall or _ExtraFields_PyCCall, "
                    f"got {type(event.extra_fields).__name__}"
                )
            if not event.extra_fields.caller.file_name.startswith("torch" + os.sep):
                return f"{event.extra_fields.caller.file_name}:{event.extra_fields.caller.line_number}"
        event = event.parent
    return "No source code location found"


def input_shapes(event: _ProfilerEvent):
    if not isinstance(event.extra_fields, _ExtraFields_TorchOp):
        raise AssertionError(
            f"expected _ExtraFields_TorchOp, got {type(event.extra_fields).__name__}"
        )
    return tuple(tuple(getattr(i, "sizes", ())) for i in event.extra_fields.inputs)
````
- **EN**: This chunk defines `input_shapes`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `input_shapes`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 617-644 / 第 617-644 行
````python
def input_dtypes(event: _ProfilerEvent):
    if not isinstance(event.extra_fields, _ExtraFields_TorchOp):
        raise AssertionError(
            f"expected _ExtraFields_TorchOp, got {type(event.extra_fields).__name__}"
        )
    return tuple(getattr(i, "dtype", None) for i in event.extra_fields.inputs)


def report_all_anti_patterns(
    prof,
    should_benchmark: bool = False,
    print_enable: bool = True,
    json_report_dir: str | None = None,
) -> None:
    report_dict: dict = {}
    anti_patterns = [
        ExtraCUDACopyPattern(prof, should_benchmark),
        # ForLoopIndexingPattern(prof, should_benchmark),
        FP32MatMulPattern(prof, should_benchmark),
        OptimizerSingleTensorPattern(prof, should_benchmark),
        SynchronizedDataLoaderPattern(prof, should_benchmark),
        GradNotSetToNonePattern(prof, should_benchmark),
        Conv2dBiasFollowedByBatchNorm2dPattern(prof, should_benchmark),
        MatMulDimInFP16Pattern(prof, should_benchmark),
    ]
    reported = set()
    summaries = []
    message_list = [f"{'-' * 40}TorchTidy Report{'-' * 40}"]
````
- **EN**: This chunk defines `report_all_anti_patterns`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `report_all_anti_patterns`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 645-665 / 第 645-665 行
````python
    message_list.append("Matched Events:")

    for anti_pattern in anti_patterns:
        matched_events = anti_pattern.matched_events()
        if not matched_events:
            continue
        summaries.append(anti_pattern.summary(matched_events))
        for event in matched_events:
            report_msg = anti_pattern.report(event)
            if report_msg not in reported:
                message_list.append(report_msg)
                reported.add(report_msg)
                src_location, line_no = source_code_location(event).split(":")
                report_dict.setdefault(src_location, []).append(
                    {
                        "line_number": int(line_no),
                        "name": anti_pattern.name,
                        "url": anti_pattern.url,
                        "message": anti_pattern.description,
                    }
                )
````
- **EN**: This chunk continues `report_all_anti_patterns` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `report_all_anti_patterns`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 667-681 / 第 667-681 行
````python
    if json_report_dir is not None:
        json_report_path = os.path.join(json_report_dir, "torchtidy_report.json")
        if os.path.exists(json_report_path):
            with open(json_report_path) as f:
                exisiting_report = json.load(f)
                exisiting_report.update(report_dict)
                report_dict = exisiting_report
        with open(json_report_path, "w") as f:
            json.dump(report_dict, f, indent=4)

    message_list.append("Summary:")
    message_list += summaries
    message_list.append(f"{'-' * 40}TorchTidy Report{'-' * 40}")
    if print_enable:
        print("\n".join(message_list))
````
- **EN**: This chunk continues `report_all_anti_patterns` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `report_all_anti_patterns`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Performance tracing**
  - EN: Captures operator-level timing and trace metadata for performance diagnosis.
  - CN: 捕获算子级时序与跟踪元数据，用于性能诊断。
- **Pattern**
  - EN: `Pattern` is one of the main symbols declared or implemented in this file.
  - CN: `Pattern` 是本文件声明或实现的主要符号之一。
- **NamePattern**
  - EN: `NamePattern` is one of the main symbols declared or implemented in this file.
  - CN: `NamePattern` 是本文件声明或实现的主要符号之一。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils.benchmark`, `torch._C._profiler`, `torch.profiler`, `torch.profiler._utils`
- **Standard library / 标准库**: `json`, `math`, `os`, `re`
- **Primary symbols in this file / 本文件核心符号**: `Pattern`, `NamePattern`, `ExtraCUDACopyPattern`, `ForLoopIndexingPattern`, `FP32MatMulPattern`, `OptimizerSingleTensorPattern`, `SynchronizedDataLoaderPattern`, `GradNotSetToNonePattern`, `Conv2dBiasFollowedByBatchNorm2dPattern`, `MatMulDimInFP16Pattern`
