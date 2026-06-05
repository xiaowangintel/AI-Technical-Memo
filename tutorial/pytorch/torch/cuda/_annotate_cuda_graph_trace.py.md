# _annotate_cuda_graph_trace.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/_annotate_cuda_graph_trace.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
````python
#!/usr/bin/env python3
"""Post-process a profiler trace to add CUDA graph kernel annotations.

Reads a profiler trace (gzipped or plain JSON) and a kernel annotations
pickle, matches kernel events by their graph node id, and writes an
annotated trace with the annotation fields added to each kernel event's
args (displayed alongside grid/block size in trace viewers).

The annotations pickle is auto-discovered from the trace file's parent
directory (one level up, matching the rank from the trace filename).

Usage:
    python -m torch.cuda._annotate_cuda_graph_trace <trace_file> [-a <annotations_pkl>] [-o <output_file>]

Examples:
    # Auto-discover annotations pickle from trace location
    python -m torch.cuda._annotate_cuda_graph_trace \\
        traces/step_000000000014/000000.*.pt.trace.json.gz
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 20-35 / 第 20-35 行
````python
    # Explicit annotations pickle
    python -m torch.cuda._annotate_cuda_graph_trace trace.json.gz -a annotations.pkl
"""

import argparse
import gzip
import json
import pickle
import re
import sys
from collections import defaultdict
from pathlib import Path
from typing import Any


_WORK_CATEGORIES = {"kernel", "gpu_memcpy", "gpu_memset"}
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as argparse, gzip, json, ....
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 argparse、gzip、json、...。

### Lines 38-57 / 第 38-57 行
````python
def _move_overlapping_to_stream(
    trace: dict, default_stream: int = 7, overlap_stream: int = 8
) -> int:
    """Move graphed kernels that overlap with their predecessor to a separate stream.

    Perfetto cannot display overlapping (non-nested) events on the same
    stream -- they get hidden.  This pass detects graphed kernel events on
    *default_stream* whose start timestamp falls before the previous
    kernel's end, and moves them to *overlap_stream* so they're visible.

    Returns the number of events moved.
    """
    graphed_on_default = [
        e
        for e in trace["traceEvents"]
        if e.get("cat") == "kernel"
        and e.get("tid") == default_stream
        and e.get("args", {}).get("graph node id", 0) != 0
    ]
    graphed_on_default.sort(key=lambda e: e["ts"])
````
- **EN**: This chunk defines `_move_overlapping_to_stream`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_move_overlapping_to_stream`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 59-75 / 第 59-75 行
````python
    moved = 0
    prev_end = 0.0
    for event in graphed_on_default:
        ts = event["ts"]
        dur = event.get("dur", 0)
        if ts < prev_end:
            event["tid"] = overlap_stream
            event.get("args", {})["stream"] = overlap_stream
            moved += 1
        else:
            prev_end = ts + dur

    return moved


def _fix_overlapping_timestamps(trace: dict, max_adjust_us: float = 1.0) -> int:
    """Clamp graphed kernel/memcpy timestamps so they don't overlap on the same stream.
````
- **EN**: This chunk defines `_fix_overlapping_timestamps`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_fix_overlapping_timestamps`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 77-94 / 第 77-94 行
````python
    CUPTI can produce slightly overlapping timestamps for consecutive graphed
    events, causing Perfetto to hide events that sit entirely "under" their
    neighbours.  This pass sorts graphed work events per stream and ensures
    each event starts at or after the previous event's end.

    Overlaps larger than *max_adjust_us* are flagged as warnings and left
    unchanged, since they likely indicate a real issue rather than CUPTI
    timestamp jitter.

    Returns the number of events adjusted.
    """
    per_stream: dict[int, list[dict]] = defaultdict(list)
    for event in trace["traceEvents"]:
        if (
            event.get("cat") in _WORK_CATEGORIES
            and event.get("args", {}).get("graph node id", 0) != 0
        ):
            per_stream[event.get("tid")].append(event)
````
- **EN**: This chunk continues `_fix_overlapping_timestamps` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_fix_overlapping_timestamps`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 96-113 / 第 96-113 行
````python
    adjusted = 0
    for tid, events in per_stream.items():
        events.sort(key=lambda e: e["ts"])
        prev_end = 0.0
        for event in events:
            ts = event["ts"]
            dur = event.get("dur", 0)
            if ts < prev_end:
                overlap = prev_end - ts
                if overlap > max_adjust_us:
                    print(
                        f"WARNING: large overlap {overlap:.3f}us on stream {tid} "
                        f"for {event.get('name', '?')[:60]}, skipping adjustment"
                    )
                else:
                    event["ts"] = prev_end
                    adjusted += 1
            prev_end = event["ts"] + dur
````
- **EN**: This chunk continues `_fix_overlapping_timestamps` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_fix_overlapping_timestamps`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 115-133 / 第 115-133 行
````python
    return adjusted


def annotate_trace(
    trace: dict,
    annotations: dict[int, list[Any]],
    default_stream: int = 7,
) -> int:
    """Add annotation fields to kernel events matching the annotations dict.

    Each annotation entry is a list (from nested ``mark_kernels`` scopes).
    Fields from all annotations are merged into the event args; if multiple
    annotations define the same key, later entries in the list win.

    For graphed events (graph_node_id != 0), reassigns ``tid`` and
    ``args["stream"]`` to the stream recorded in annotations, or to
    *default_stream* if there is no annotation.  Also moves the
    corresponding ``ac2g`` flow-finish events to the new tid so that
    CPU-to-GPU correlation arrows are preserved.
````
- **EN**: This chunk defines `annotate_trace`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `annotate_trace`，其作用是协调 tracing、捕获或编译所需的图相关状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 135-147 / 第 135-147 行
````python
    Removes ``gpu_user_annotation`` events and orphaned ``ac2g`` events
    from streams that have no kernel or memcpy events after reassignment,
    since CUPTI replicates these onto every stream during graph replay.

    Returns the number of events annotated.
    """
    # Build an index of ac2g 'f' events keyed by (tid, ts) so we can
    # move them together with the kernel events they correspond to.
    ac2g_f_index: dict[tuple, list] = {}
    for event in trace["traceEvents"]:
        if event.get("cat") == "ac2g" and event.get("ph") == "f":
            key = (event.get("tid"), event.get("ts"))
            ac2g_f_index.setdefault(key, []).append(event)
````
- **EN**: This chunk continues `annotate_trace` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `annotate_trace`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 149-165 / 第 149-165 行
````python
    annotated = 0
    for event in trace.get("traceEvents", []):
        args = event.get("args", {})
        graph_node_id = args.get("graph node id")
        if graph_node_id is None or graph_node_id == 0:
            continue
        stream_id = None
        if graph_node_id in annotations:
            for ann in annotations[graph_node_id]:
                if isinstance(ann, dict):
                    for key, value in ann.items():
                        args[key] = str(value)
                    if "stream" in ann:
                        stream_id = int(ann["stream"])
                else:
                    args["annotation"] = str(ann)
            annotated += 1
````
- **EN**: This chunk continues `annotate_trace` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `annotate_trace`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 167-184 / 第 167-184 行
````python
        # Reassign stream: use annotated stream if available, else default
        if stream_id is None:
            stream_id = default_stream
        old_key = (event.get("tid"), event.get("ts"))
        event["tid"] = stream_id
        args["stream"] = stream_id

        # Move the matching ac2g 'f' event(s) to the same new tid
        for ac2g_event in ac2g_f_index.get(old_key, ()):
            ac2g_event["tid"] = stream_id

    # Remove gpu_user_annotation events and ac2g flow-finish events from
    # streams that have no real kernel/memcpy/memset work -- these are
    # noise replicated by CUPTI onto every stream during graph replay.
    tids_with_work = set()
    for event in trace["traceEvents"]:
        if event.get("cat") in _WORK_CATEGORIES:
            tids_with_work.add(event.get("tid"))
````
- **EN**: This chunk continues `annotate_trace` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `annotate_trace`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 186-200 / 第 186-200 行
````python
    def _is_noise(event: dict) -> bool:
        cat = event.get("cat")
        if cat == "gpu_user_annotation":
            return event.get("tid") not in tids_with_work
        if cat == "ac2g" and event.get("ph") == "f":
            return event.get("tid") not in tids_with_work
        return False

    original_count = len(trace["traceEvents"])
    trace["traceEvents"] = [
        event for event in trace["traceEvents"] if not _is_noise(event)
    ]
    removed = original_count - len(trace["traceEvents"])
    if removed:
        print(f"Removed {removed} noise events from empty streams")
````
- **EN**: This chunk defines `_is_noise`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_is_noise`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 202-217 / 第 202-217 行
````python
    # Clean up metadata: remove thread_name / thread_sort_index entries
    # for noise streams that have no real (non-metadata) events, and add
    # thread_name entries for our new annotation streams.
    all_tids_in_trace = {
        e.get("tid") for e in trace["traceEvents"] if e.get("ph") != "M"
    }
    # Find the GPU process pid from existing thread_name metadata
    gpu_pid = 0
    for event in trace["traceEvents"]:
        if (
            event.get("ph") == "M"
            and event.get("name") == "thread_name"
            and str(event.get("args", {}).get("name", "")).startswith("stream ")
        ):
            gpu_pid = event.get("pid", 0)
            break
````
- **EN**: This chunk continues `_is_noise` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_is_noise`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 219-238 / 第 219-238 行
````python
    # Remove metadata entries for tids with no non-metadata events
    trace["traceEvents"] = [
        event
        for event in trace["traceEvents"]
        if event.get("ph") != "M" or event.get("tid") in all_tids_in_trace
    ]

    # Add thread_name metadata for new annotation tids that lack one
    existing_thread_names = {
        e.get("tid")
        for e in trace["traceEvents"]
        if e.get("ph") == "M" and e.get("name") == "thread_name"
    }
    for tid in sorted(tids_with_work - existing_thread_names):
        trace["traceEvents"].append(
            {
                "ph": "M",
                "pid": gpu_pid,
                "tid": tid,
                "name": "thread_name",
````
- **EN**: This chunk continues `_is_noise` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_is_noise`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 239-252 / 第 239-252 行
````python
                "args": {"name": f"stream {tid}"},
            }
        )

    return annotated


def load_trace(path: Path) -> dict:
    if path.suffix == ".gz" or path.name.endswith(".json.gz"):
        with gzip.open(path, "rt") as f:
            return json.load(f)
    else:
        with open(path) as f:
            return json.load(f)
````
- **EN**: This chunk defines `load_trace`, which serializes or reconstructs state across a Python-visible boundary. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `load_trace`，其作用是在 Python 可见边界上序列化或重建状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 255-274 / 第 255-274 行
````python
def save_trace(trace: dict, path: Path) -> None:
    if path.suffix == ".gz" or path.name.endswith(".json.gz"):
        with gzip.open(path, "wt") as f:
            json.dump(trace, f)
    else:
        with open(path, "w") as f:
            json.dump(trace, f)


def _find_annotations_pkl(trace_file: Path) -> Path | None:
    """Auto-discover the annotations pickle from the trace file location.

    Trace files live in e.g. ``traces/step_000000000014/000000.<id>.pt.trace.json.gz``
    where the leading digits are the rank. The pickle lives one level up:
    ``traces/kernel_annotations_rank0_*.pkl``.
    """
    match = re.match(r"^(\d+)", trace_file.name)
    if not match:
        return None
    rank = int(match.group(1))
````
- **EN**: This chunk defines `_find_annotations_pkl`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_find_annotations_pkl`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 276-295 / 第 276-295 行
````python
    traces_dir = trace_file.parent.parent
    candidates = sorted(traces_dir.glob(f"kernel_annotations_rank{rank}_*.pkl"))
    if candidates:
        return candidates[0]
    return None


def main() -> None:
    parser = argparse.ArgumentParser(
        description="Annotate a profiler trace with CUDA graph kernel annotations."
    )
    parser.add_argument(
        "trace_file", type=Path, help="Input trace file (.json or .json.gz)"
    )
    parser.add_argument(
        "-a",
        "--annotations",
        type=Path,
        default=None,
        help="Kernel annotations pickle file. Auto-discovered if omitted.",
````
- **EN**: This chunk defines `main`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `main`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 296-310 / 第 296-310 行
````python
    )
    parser.add_argument(
        "-o",
        "--output",
        type=Path,
        default=None,
        help="Output file path. Defaults to <trace_file>.annotated.<ext>",
    )
    parser.add_argument(
        "--default-stream",
        type=int,
        default=7,
        help="Stream ID to assign to unannotated graphed events (default: 7).",
    )
    args = parser.parse_args()
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。

### Lines 312-330 / 第 312-330 行
````python
    annotations_pkl = args.annotations
    if annotations_pkl is None:
        annotations_pkl = _find_annotations_pkl(args.trace_file)
        if annotations_pkl is None:
            print(
                f"Could not auto-discover annotations pickle for {args.trace_file}. "
                f"Use -a to specify it explicitly.",
                file=sys.stderr,
            )
            sys.exit(1)
        print(f"Auto-discovered annotations: {annotations_pkl}")

    with open(annotations_pkl, "rb") as f:
        annotations = pickle.load(f)
    print(f"Loaded {len(annotations)} kernel annotations")

    trace = load_trace(args.trace_file)
    total_events = len(trace.get("traceEvents", []))
    print(f"Loaded trace with {total_events} events")
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 332-351 / 第 332-351 行
````python
    count = annotate_trace(trace, annotations, default_stream=args.default_stream)
    print(f"Annotated {count} kernel events")

    overlap_moved = _move_overlapping_to_stream(
        trace, default_stream=args.default_stream
    )
    if overlap_moved:
        print(f"Moved {overlap_moved} overlapping events to stream 8")

    ts_fixed = _fix_overlapping_timestamps(trace)
    if ts_fixed:
        print(f"Fixed {ts_fixed} overlapping graphed event timestamps")

    output = args.output
    if output is None:
        name = args.trace_file.name
        if name.endswith(".json.gz"):
            output = args.trace_file.with_name(
                name.replace(".json.gz", ".annotated.json.gz")
            )
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 352-362 / 第 352-362 行
````python
        elif name.endswith(".json"):
            output = args.trace_file.with_suffix(".annotated.json")
        else:
            output = args.trace_file.with_suffix(args.trace_file.suffix + ".annotated")

    save_trace(trace, output)
    print(f"Saved annotated trace to {output}")


if __name__ == "__main__":
    main()
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **_WORK_CATEGORIES**
  - EN: `_WORK_CATEGORIES` is one of the main symbols declared or implemented in this file.
  - CN: `_WORK_CATEGORIES` 是本文件声明或实现的主要符号之一。
- **_move_overlapping_to_stream**
  - EN: `_move_overlapping_to_stream` is one of the main symbols declared or implemented in this file.
  - CN: `_move_overlapping_to_stream` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
## Dependencies / 依赖关系

- **Standard library / 标准库**: `argparse`, `gzip`, `json`, `pickle`, `re`, `sys`, `collections`, `pathlib`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `_WORK_CATEGORIES`, `_move_overlapping_to_stream`, `_fix_overlapping_timestamps`, `annotate_trace`, `load_trace`, `save_trace`, `_find_annotations_pkl`, `main`
