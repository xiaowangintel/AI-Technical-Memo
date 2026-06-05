# proton_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/runtime/proton_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module provides runtime helpers used after compilation. It exposes functions such as `_write_trace`, `_read_trace`, `_get_base_name`, `_group_events_by_sm`, `_group_events_per_cta_occupancy`, `_apply_grouping`, and `...+3`. Module note: Utilities for post-processing Triton Proton profiling trace files.
- **用途（中文）**: 该模块提供编译完成后使用的运行期辅助逻辑。同时提供 `_write_trace`、`_read_trace`、`_get_base_name`、`_group_events_by_sm`、`_group_events_per_cta_occupancy`、`_apply_grouping`、`另有3项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
"""
Utilities for post-processing Triton Proton profiling trace files.

This module provides functions to transform proton-generated Chrome trace files
for better visualization in Perfetto/Chrome tracing tools.
"""

import gzip
import json
import os
import re
from typing import Any

from torch.utils._ordered_set import OrderedSet


def _write_trace(path: str, data: dict[str, Any]) -> None:
    """Write trace data to a gzipped JSON file."""
    with gzip.open(path, "wt", encoding="utf-8") as f:
        json.dump(data, f)
````
- **EN**: Imports dependencies such as `gzip`, `json`, `os`, `re`, `typing`, and `torch.utils._ordered_set` for the logic in this range. Introduces function `_write_trace`. Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts.
- **CN**: 这里导入了 `gzip`、`json`、`os`、`re`、`typing`、`torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_write_trace`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。

### Lines 21-40 / 第 21-40 行
````python


def _read_trace(path: str) -> dict[str, Any]:
    """Read trace data from a JSON file (gzipped or not)."""
    if path.endswith(".gz"):
        with gzip.open(path, "rt", encoding="utf-8") as f:
            return json.load(f)
    else:
        with open(path) as f:
            return json.load(f)


def _get_base_name(trace_path: str) -> str:
    """Get the base name of a trace file, stripping known extensions."""
    base_name = os.path.basename(trace_path)
    for ext in [".trace.json.gz", ".chrome_trace", ".json.gz", ".json"]:
        if base_name.endswith(ext):
            return base_name[: -len(ext)]
    return os.path.splitext(base_name)[0]

````
- **EN**: Introduces function `_read_trace`, function `_get_base_name`. Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_read_trace`、函数`_get_base_name`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 41-60 / 第 41-60 行
````python

def _group_events_by_sm(events: list[dict[str, Any]]) -> list[dict[str, Any]]:
    """
    Group events so all CTAs on the same SM share one track.

    Transforms:
    - pid: "Core0 CTA6" -> "Core0"
    - tid: "warp0" -> "CTA6 warp0"
    """
    core_cta_pattern = re.compile(r"^(.*?)\s*(Core\d+)\s+(CTA\d+)$")

    for event in events:
        pid = event.get("pid", "")
        tid = event.get("tid", "")

        match = core_cta_pattern.match(str(pid))
        if match:
            prefix = match.group(1).strip()
            core = match.group(2)
            cta = match.group(3)
````
- **EN**: Introduces function `_group_events_by_sm`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Transforms`, `core_cta_pattern`, `pid`, `tid`, `match`, `prefix`, and `...+2`.
- **CN**: 这里定义了函数`_group_events_by_sm`。包含分支、循环或上下文管理等控制流。初始化或更新了 `Transforms`、`core_cta_pattern`、`pid`、`tid`、`match`、`prefix`、`另有2项` 等值。

### Lines 61-80 / 第 61-80 行
````python
            event["pid"] = f"{prefix} {core}" if prefix else core
            event["tid"] = f"{cta} {tid}" if tid else cta

    return events


def _group_events_per_cta_occupancy(
    events: list[dict[str, Any]],
) -> list[dict[str, Any]]:
    """
    Process warp events into CTA-level events with slot assignment.

    Each CTA gets: min warp start, max warp end.
    CTAs are assigned to slots per SM such that non-overlapping CTAs share slots.
    """
    core_cta_pattern = re.compile(r"^(.*?)\s*(Core\d+)\s+(CTA\d+)$")

    # Group events by (prefix, SM, CTA)
    cta_events: dict[tuple[str, str, str], list[dict[str, Any]]] = {}

````
- **EN**: Introduces function `_group_events_per_cta_occupancy`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `events`, `core_cta_pattern`, and `cta_events`.
- **CN**: 这里定义了函数`_group_events_per_cta_occupancy`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `events`、`core_cta_pattern`、`cta_events` 等值。

### Lines 81-100 / 第 81-100 行
````python
    for event in events:
        pid = event.get("pid", "")
        match = core_cta_pattern.match(str(pid))
        if match:
            prefix = match.group(1).strip()
            core = match.group(2)
            cta = match.group(3)
            key = (prefix, core, cta)
            if key not in cta_events:
                cta_events[key] = []
            cta_events[key].append(event)

    # For each CTA, compute min start and max end
    cta_intervals: dict[tuple[str, str, str], tuple[float, float]] = {}
    for key, evts in cta_events.items():
        min_start = float("inf")
        max_end = float("-inf")
        for evt in evts:
            ts = evt.get("ts", 0)
            dur = evt.get("dur", 0)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pid`, `match`, `prefix`, `core`, `cta`, `key`, and `...+5`. This range continues the implementation of function `_group_events_per_cta_occupancy`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `pid`、`match`、`prefix`、`core`、`cta`、`key`、`另有5项` 等值。这一段延续了函数`_group_events_per_cta_occupancy` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
            min_start = min(min_start, ts)
            max_end = max(max_end, ts + dur)
        if min_start != float("inf"):
            cta_intervals[key] = (min_start, max_end)

    # Group CTAs by (prefix, SM) and assign to slots
    sm_ctas: dict[tuple[str, str], list[tuple[str, float, float]]] = {}
    for (prefix, core, cta), (start, end) in cta_intervals.items():
        sm_key = (prefix, core)
        if sm_key not in sm_ctas:
            sm_ctas[sm_key] = []
        sm_ctas[sm_key].append((cta, start, end))

    # Assign CTAs to slots using interval scheduling (greedy)
    cta_slot_assignments: dict[tuple[str, str, str], int] = {}
    for sm_key, ctas in sm_ctas.items():
        prefix, core = sm_key
        sorted_ctas = sorted(ctas, key=lambda x: x[1])
        slots: list[float] = []
        for cta, start, end in sorted_ctas:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_start`, `max_end`, `sm_ctas`, `sm_key`, `cta_slot_assignments`, `sorted_ctas`, and `...+1`. This range continues the implementation of function `_group_events_per_cta_occupancy`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `min_start`、`max_end`、`sm_ctas`、`sm_key`、`cta_slot_assignments`、`sorted_ctas`、`另有1项` 等值。这一段延续了函数`_group_events_per_cta_occupancy` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
            assigned_slot = None
            for i, slot_end in enumerate(slots):
                if start >= slot_end:
                    assigned_slot = i
                    slots[i] = end
                    break
            if assigned_slot is None:
                assigned_slot = len(slots)
                slots.append(end)
            cta_slot_assignments[(prefix, core, cta)] = assigned_slot

    # Build numeric ID mappings for pids and tids
    # Chrome trace format requires numeric pid/tid values
    pid_names: dict[str, int] = {}
    tid_names: dict[tuple[int, str], int] = {}  # (pid, tid_name) -> tid_num

    for key in cta_intervals:
        prefix, core, cta = key
        pid_name = f"{prefix} {core}" if prefix else core
        if pid_name not in pid_names:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `assigned_slot`, `pid_names`, `tid_names`, and `pid_name`. This range continues the implementation of function `_group_events_per_cta_occupancy`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `assigned_slot`、`pid_names`、`tid_names`、`pid_name` 等值。这一段延续了函数`_group_events_per_cta_occupancy` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
            pid_names[pid_name] = len(pid_names)

    new_events: list[dict[str, Any]] = []

    # Add process name metadata events
    for pid_name, pid_num in pid_names.items():
        new_events.append(
            {
                "name": "process_name",
                "ph": "M",
                "pid": pid_num,
                "args": {"name": pid_name},
            }
        )

    # Create CTA events with numeric IDs and collect tid mappings
    for key, (start, end) in cta_intervals.items():
        prefix, core, cta = key
        slot = cta_slot_assignments[key]
        pid_name = f"{prefix} {core}" if prefix else core
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_events`, `slot`, and `pid_name`. This range continues the implementation of function `_group_events_per_cta_occupancy`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `new_events`、`slot`、`pid_name` 等值。这一段延续了函数`_group_events_per_cta_occupancy` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
        tid_name = f"slot{slot}"
        pid_num = pid_names[pid_name]

        if (pid_num, tid_name) not in tid_names:
            tid_num = len(tid_names)
            tid_names[(pid_num, tid_name)] = tid_num
            # Add thread name metadata event
            new_events.append(
                {
                    "name": "thread_name",
                    "ph": "M",
                    "pid": pid_num,
                    "tid": tid_num,
                    "args": {"name": tid_name},
                }
            )
        else:
            tid_num = tid_names[(pid_num, tid_name)]

        new_events.append(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tid_name`, `pid_num`, `tid_num`, and `else`. This range continues the implementation of function `_group_events_per_cta_occupancy`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `tid_name`、`pid_num`、`tid_num`、`else` 等值。这一段延续了函数`_group_events_per_cta_occupancy` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            {
                "name": cta,
                "cat": "cta",
                "ph": "X",
                "ts": start,
                "dur": end - start,
                "pid": pid_num,
                "tid": tid_num,
            }
        )

    return new_events


def _apply_grouping(
    events: list[dict[str, Any]], group_by_sm: bool, per_cta_occupancy: bool
) -> list[dict[str, Any]]:
    """Apply grouping transformation to events."""
    if per_cta_occupancy:
        return _group_events_per_cta_occupancy(events)
````
- **EN**: Introduces function `_apply_grouping`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `events`.
- **CN**: 这里定义了函数`_apply_grouping`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `events` 等值。

### Lines 201-220 / 第 201-220 行
````python
    elif group_by_sm:
        return _group_events_by_sm(events)
    return events


def _split_events_by_invocation(
    events: list[dict[str, Any]],
    gap_threshold_ns: float = 1000.0,
) -> list[list[dict[str, Any]]]:
    """Split events into separate invocations based on time gaps."""
    if not events:
        return []

    events_sorted = sorted(events, key=lambda e: e.get("ts", 0))
    invocations: list[list[dict[str, Any]]] = [[]]
    prev_end = events_sorted[0].get("ts", 0)

    for event in events_sorted:
        ts = event.get("ts", 0)
        dur = event.get("dur", 0)
````
- **EN**: Introduces function `_split_events_by_invocation`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `events`, `gap_threshold_ns`, `events_sorted`, `invocations`, `prev_end`, `ts`, and `...+1`.
- **CN**: 这里定义了函数`_split_events_by_invocation`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `events`、`gap_threshold_ns`、`events_sorted`、`invocations`、`prev_end`、`ts`、`另有1项` 等值。

### Lines 221-240 / 第 221-240 行
````python
        if ts - prev_end > gap_threshold_ns and invocations[-1]:
            invocations.append([])
        invocations[-1].append(event)
        prev_end = max(prev_end, ts + dur)

    return [inv for inv in invocations if inv]


def _normalize_timestamps(
    events: list[dict[str, Any]], scale_factor: float = 1.0
) -> list[dict[str, Any]]:
    """Normalize timestamps to start at 0 and optionally scale."""
    if not events:
        return events

    # Only consider duration events (ph=X) for min timestamp calculation
    duration_events = [e for e in events if e.get("ph") == "X"]
    if not duration_events:
        return events

````
- **EN**: Introduces function `_normalize_timestamps`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `prev_end`, `events`, and `duration_events`.
- **CN**: 这里定义了函数`_normalize_timestamps`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `prev_end`、`events`、`duration_events` 等值。

### Lines 241-260 / 第 241-260 行
````python
    min_ts = min(e.get("ts", 0) for e in duration_events)
    for event in events:
        if event.get("ph") == "X":
            event["ts"] = (event.get("ts", 0) - min_ts) * scale_factor
            if "dur" in event:
                event["dur"] = event["dur"] * scale_factor

    return events


def process_proton_trace(
    trace_path: str,
    output_dir: str | None = None,
    group_by_sm: bool = True,
    split_invocations: bool = True,
    scale_factor: float = 1.0,
    gap_threshold_ns: float = 1000.0,
    per_cta_occupancy: bool = True,
) -> list[str]:
    """
````
- **EN**: Introduces function `process_proton_trace`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_ts`, `trace_path`, `output_dir`, `group_by_sm`, `split_invocations`, `scale_factor`, and `...+2`.
- **CN**: 这里定义了函数`process_proton_trace`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `min_ts`、`trace_path`、`output_dir`、`group_by_sm`、`split_invocations`、`scale_factor`、`另有2项` 等值。

### Lines 261-280 / 第 261-280 行
````python
    Process a proton trace file with various transformations.

    Always produces 1 + N files: one complete trace and N per-invocation traces.
    Grouping options (group_by_sm, per_cta_occupancy) apply uniformly to all outputs.

    Args:
        trace_path: Path to the input Chrome trace file
        output_dir: Directory to write output files. If None, uses same directory as input.
        group_by_sm: If True, group CTAs by SM into single tracks.
        split_invocations: If True, also produce per-invocation trace files.
        scale_factor: Factor to scale durations by (helps visibility in Perfetto).
        gap_threshold_ns: Time gap (in nanoseconds) that indicates a new invocation.
        per_cta_occupancy: If True, process warp tracks into CTA tracks and assign
            CTAs to slots per SM such that CTAs do not overlap.

    Returns:
        List of paths to the output files.
    """
    if output_dir is None:
        output_dir = os.path.dirname(trace_path) or "."
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `trace_path`, `output_dir`, `group_by_sm`, `split_invocations`, `scale_factor`, and `...+3`. This range continues the implementation of function `process_proton_trace`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`trace_path`、`output_dir`、`group_by_sm`、`split_invocations`、`scale_factor`、`另有3项` 等值。这一段延续了函数`process_proton_trace` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python

    os.makedirs(output_dir, exist_ok=True)
    base_name = _get_base_name(trace_path)

    data = _read_trace(trace_path)
    events = data.get("traceEvents", [])

    # Split into invocations first (before grouping, to avoid merging across invocations)
    invocations = _split_events_by_invocation(events, gap_threshold_ns)

    # Apply grouping transformation to each invocation
    invocations = [
        _apply_grouping(inv_events, group_by_sm, per_cta_occupancy)
        for inv_events in invocations
    ]

    output_files = []

    # Write complete trace (dedupe metadata events)
    complete_events = []
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `base_name`, `data`, `events`, `invocations`, `output_files`, and `complete_events`. This range continues the implementation of function `process_proton_trace`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `base_name`、`data`、`events`、`invocations`、`output_files`、`complete_events` 等值。这一段延续了函数`process_proton_trace` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
    seen_metadata: OrderedSet[tuple[str | None, int | None, int | None]] = OrderedSet()
    for inv_events in invocations:
        for event in inv_events:
            if event.get("ph") == "M":
                # Dedupe metadata events by (name, pid, tid)
                key = (event.get("name"), event.get("pid"), event.get("tid"))
                if key in seen_metadata:
                    continue
                seen_metadata.add(key)
            complete_events.append(event)
    complete_path = os.path.join(output_dir, f"{base_name}.trace.json.gz")
    _write_trace(complete_path, {"traceEvents": complete_events})
    output_files.append(complete_path)

    # Write per-invocation traces
    if split_invocations:
        for i, inv_events in enumerate(invocations):
            inv_events = _normalize_timestamps(list(inv_events), scale_factor)
            inv_path = os.path.join(
                output_dir, f"{base_name}_invocation_{i}.trace.json.gz"
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `seen_metadata`, `key`, `complete_path`, `inv_events`, and `inv_path`. This range continues the implementation of function `process_proton_trace`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `seen_metadata`、`key`、`complete_path`、`inv_events`、`inv_path` 等值。这一段延续了函数`process_proton_trace` 的具体实现。

### Lines 321-325 / 第 321-325 行
````python
            )
            _write_trace(inv_path, {"traceEvents": inv_events})
            output_files.append(inv_path)

    return output_files
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `process_proton_trace`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`process_proton_trace` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Provides runtime helpers used after compilation  
  **CN**: 提供编译完成后使用的运行期辅助逻辑
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Primary functions: `_write_trace`, `_read_trace`, `_get_base_name`, `_group_events_by_sm`, `_group_events_per_cta_occupancy`, `_apply_grouping`, and `...+3`  
  **CN**: 主要函数：`_write_trace`、`_read_trace`、`_get_base_name`、`_group_events_by_sm`、`_group_events_per_cta_occupancy`、`_apply_grouping`、`另有3项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gzip`, `json`, `os`, `re`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch.utils._ordered_set`
