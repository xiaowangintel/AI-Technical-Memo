# _trace_validator.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/profiler/_trace_validator.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements profiling APIs, trace utilities, and performance-analysis helpers.
- **Purpose (CN)**: 实现 profiling API、跟踪工具以及性能分析辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
````python
# mypy: allow-untyped-defs
"""
Validates Chrome traces emitted by ``torch.profiler`` against rules derived
from production issues.

Usage::

    from torch.profiler._trace_validator import validate_trace

    passed, violations = validate_trace("trace.pt.trace.json")
    for v in violations:
        print(v)
"""

from __future__ import annotations
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.profiler._trace_validator; standard-library helpers such as __future__. The future import postpones annotation evaluation, keeping type hints lightweight at import time. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.profiler._trace_validator；标准库辅助模块，如 __future__。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 17-33 / 第 17-33 行
````python
import dataclasses
import gzip
import json
from collections import defaultdict
from typing import TYPE_CHECKING


if TYPE_CHECKING:
    from collections.abc import Callable


@dataclasses.dataclass
class Violation:
    """A single rule violation found in a trace."""

    rule_name: str
    message: str
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as dataclasses, gzip, json, .... It introduces or extends `Violation`, which hold the main object-oriented state for this portion of the file. Decorators such as `dataclasses.dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 dataclasses、gzip、json、...。 它引入或扩展了 `Violation`，这些类承载了本段涉及的主要面向对象状态。 像 `dataclasses.dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 35-51 / 第 35-51 行
````python
    def __str__(self) -> str:
        return f"{self.rule_name}: {self.message}"


def _load_events(path: str) -> list[dict]:
    opener = gzip.open if path.endswith(".gz") else open
    with opener(path, "rt", encoding="utf-8") as fh:
        data = json.load(fh)
    events = data if isinstance(data, list) else data.get("traceEvents", [])
    return [e for e in events if isinstance(e, dict)]


def _check_gpu_kernel_causality(events: list[dict]) -> list[Violation]:
    """For each (cudaLaunchKernel, GPU kernel) pair matched by External id,
    the GPU kernel must start at or after its cudaLaunchKernel."""
    cpu_launches: dict[int, dict] = {}
    gpu_kernels: dict[int, dict] = {}
````
- **EN**: This chunk defines `_check_gpu_kernel_causality`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_gpu_kernel_causality`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 53-69 / 第 53-69 行
````python
    for ev in events:
        if ev.get("ph") != "X":
            continue
        args = ev.get("args", {})
        ext_id = args.get("External id")
        if ext_id is None:
            continue
        ext_id = int(ext_id)
        cat, name, ts = ev.get("cat", ""), ev.get("name", ""), float(ev.get("ts", 0))
        corr = args.get("correlation")

        if cat == "cuda_runtime" and name == "cudaLaunchKernel":
            if ext_id not in cpu_launches or ts < cpu_launches[ext_id]["ts"]:
                cpu_launches[ext_id] = {"ts": ts, "name": name, "corr": corr}
        elif cat == "kernel":
            if ext_id not in gpu_kernels or ts < gpu_kernels[ext_id]["ts"]:
                gpu_kernels[ext_id] = {"ts": ts, "name": name, "corr": corr}
````
- **EN**: This chunk continues `_check_gpu_kernel_causality` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_check_gpu_kernel_causality`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 71-90 / 第 71-90 行
````python
    violations = []
    for ext_id, gpu in gpu_kernels.items():
        launch = cpu_launches.get(ext_id)
        if launch is None:
            continue
        if gpu["ts"] < launch["ts"]:
            skew = launch["ts"] - gpu["ts"]
            violations.append(
                Violation(
                    rule_name="_check_gpu_kernel_causality",
                    message=(
                        f"GPU kernel '{gpu['name']}' (External id={ext_id}, "
                        f"correlation={gpu['corr']}) starts {skew:.1f}us before "
                        f"its cudaLaunchKernel (External id={ext_id}, "
                        f"correlation={launch['corr']}), "
                        f"gpu_ts={gpu['ts']:.1f}, cpu_ts={launch['ts']:.1f}"
                    ),
                )
            )
    return violations
````
- **EN**: This chunk continues `_check_gpu_kernel_causality` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_check_gpu_kernel_causality`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 93-112 / 第 93-112 行
````python
def _check_stream_wait_corr_id_populated(events: list[dict]) -> list[Violation]:
    """Stream Wait Events and Event Synchronize must have
    wait_on_cuda_event_record_corr_id >= 0."""
    TARGET_KINDS = {"Stream Wait Event", "Event Sync"}

    violations = []
    for ev in events:
        if ev.get("ph") != "X":
            continue
        args = ev.get("args", {})
        sync_kind = args.get("cuda_sync_kind")
        if sync_kind not in TARGET_KINDS:
            continue
        raw_corr = args.get("wait_on_cuda_event_record_corr_id")
        if raw_corr is None or int(raw_corr) < 0:
            ts = float(ev.get("ts", 0))
            violations.append(
                Violation(
                    rule_name="_check_stream_wait_corr_id_populated",
                    message=(
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_check_stream_wait_corr_id_populated`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_check_stream_wait_corr_id_populated`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 113-132 / 第 113-132 行
````python
                        f"'{sync_kind}' event at ts={ts:.1f}us on "
                        f"device={args.get('device')} stream={args.get('stream')} "
                        f"has invalid wait_on_cuda_event_record_corr_id={raw_corr!r}"
                    ),
                )
            )
    return violations


def _check_stream_sync_overlap(events: list[dict]) -> list[Violation]:
    """For each Stream Synchronize on (device, stream), no kernel on that
    stream should still be running when the sync starts."""
    stream_syncs = []
    for ev in events:
        if (
            ev.get("ph") == "X"
            and ev.get("cat") == "cuda_sync"
            and ev.get("args", {}).get("cuda_sync_kind") == "Stream Sync"
        ):
            args = ev.get("args", {})
````
- **EN**: This chunk defines `_check_stream_sync_overlap`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_stream_sync_overlap`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 133-152 / 第 133-152 行
````python
            stream_syncs.append(
                {
                    "ts": float(ev.get("ts", 0)),
                    "dur": float(ev.get("dur", 0)),
                    "stream": args.get("stream"),
                    "device": args.get("device"),
                }
            )
    if not stream_syncs:
        return []

    kernels_by_stream: dict[tuple, list[dict]] = defaultdict(list)
    for ev in events:
        if ev.get("ph") == "X" and ev.get("cat") == "kernel":
            args = ev.get("args", {})
            key = (args.get("device"), args.get("stream"))
            ts = float(ev.get("ts", 0))
            kernels_by_stream[key].append(
                {
                    "ts": ts,
````
- **EN**: This chunk continues `_check_stream_sync_overlap` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_check_stream_sync_overlap`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 153-172 / 第 153-172 行
````python
                    "end": ts + float(ev.get("dur", 0)),
                    "name": ev.get("name", ""),
                }
            )

    violations = []
    for sync in stream_syncs:
        key = (sync["device"], sync["stream"])
        for k in kernels_by_stream.get(key, []):
            if k["ts"] < sync["ts"] < k["end"]:
                overlap = k["end"] - sync["ts"]
                violations.append(
                    Violation(
                        rule_name="_check_stream_sync_overlap",
                        message=(
                            f"StreamSynchronize on device={sync['device']} "
                            f"stream={sync['stream']} at ts={sync['ts']:.1f}us "
                            f"but kernel '{k['name']}' (ends {k['end']:.1f}us) "
                            f"is still running ({overlap:.1f}us overlap)"
                        ),
````
- **EN**: This chunk continues `_check_stream_sync_overlap` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_check_stream_sync_overlap`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 173-192 / 第 173-192 行
````python
                    )
                )
    return violations


_CUDA_EVENT_RECORD_NAMES = {
    "cudaEventRecord",
    "cudaEventRecord_ptsz",
    "cudaEventRecordWithFlags",
    "cudaEventRecordWithFlags_ptsz",
}


def _check_stream_wait_corr_id_in_past(events: list[dict]) -> list[Violation]:
    """wait_on_cuda_event_record_corr_id must point to a cudaEventRecord
    with cudaEventRecord.ts <= stream_wait.ts."""
    event_record_ts: dict[int, float] = {}
    for ev in events:
        if (
            ev.get("ph") == "X"
````
- **EN**: This chunk defines `_check_stream_wait_corr_id_in_past`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_check_stream_wait_corr_id_in_past`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 193-212 / 第 193-212 行
````python
            and ev.get("cat") in ("cuda_runtime", "cuda_driver")
            and ev.get("name") in _CUDA_EVENT_RECORD_NAMES
        ):
            args = ev.get("args", {})
            ts = float(ev.get("ts", 0))
            for field in ("External id", "correlation"):
                cid = args.get(field)
                if cid is not None:
                    cid = int(cid)
                    if cid not in event_record_ts or ts < event_record_ts[cid]:
                        event_record_ts[cid] = ts

    violations = []
    for ev in events:
        if ev.get("ph") != "X":
            continue
        args = ev.get("args", {})
        if args.get("cuda_sync_kind") != "Stream Wait Event":
            continue
        ref = args.get("wait_on_cuda_event_record_corr_id")
````
- **EN**: This chunk continues `_check_stream_wait_corr_id_in_past` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_check_stream_wait_corr_id_in_past`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 213-232 / 第 213-232 行
````python
        if ref is None or int(ref) < 0:
            continue
        ref = int(ref)
        sw_ts = float(ev.get("ts", 0))
        record_ts = event_record_ts.get(ref)

        if record_ts is None:
            violations.append(
                Violation(
                    rule_name="_check_stream_wait_corr_id_in_past",
                    message=(
                        f"Stream Wait Event at ts={sw_ts:.1f}us references "
                        f"corr_id={ref} but no matching cudaEventRecord in trace"
                    ),
                )
            )
        elif record_ts > sw_ts:
            lag = record_ts - sw_ts
            violations.append(
                Violation(
````
- **EN**: This chunk continues `_check_stream_wait_corr_id_in_past` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_check_stream_wait_corr_id_in_past`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 233-250 / 第 233-250 行
````python
                    rule_name="_check_stream_wait_corr_id_in_past",
                    message=(
                        f"Stream Wait Event at ts={sw_ts:.1f}us references "
                        f"cudaEventRecord (corr_id={ref}) {lag:.1f}us in the future "
                        f"(event_record_ts={record_ts:.1f})"
                    ),
                )
            )
    return violations


_NCCL_REQUIRED_FIELDS = {
    "Collective name",
    "dtype",
    "In msg nelems",
    "Out msg nelems",
    "Group size",
}
````
- **EN**: This chunk continues `_check_stream_wait_corr_id_in_past` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_check_stream_wait_corr_id_in_past`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 253-272 / 第 253-272 行
````python
def _check_nccl_metadata(events: list[dict]) -> list[Violation]:
    """record_param_comms events must carry: Collective name, dtype,
    In msg nelems, Out msg nelems, Group size."""
    violations = []
    for ev in events:
        if ev.get("ph") != "X" or ev.get("name") != "record_param_comms":
            continue
        args = ev.get("args", {})
        missing = _NCCL_REQUIRED_FIELDS - set(args.keys())
        if missing:
            violations.append(
                Violation(
                    rule_name="_check_nccl_metadata",
                    message=(
                        f"'record_param_comms' at ts={float(ev.get('ts', 0)):.1f}us "
                        f"missing metadata: {sorted(missing)}"
                    ),
                )
            )
    return violations
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_check_nccl_metadata`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_check_nccl_metadata`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 275-291 / 第 275-291 行
````python
def _check_backward_seq_id_uniqueness(events: list[dict]) -> list[Violation]:
    """Per Sequence number, at most one distinct backward op name."""
    seq_to_ops: dict[int, list[str]] = defaultdict(list)
    for ev in events:
        if ev.get("ph") != "X":
            continue
        name = ev.get("name", "")
        if "autograd::engine::evaluate_function:" not in name:
            continue
        args = ev.get("args", {})
        seq = args.get("Sequence number") or args.get("seq_num")
        if seq is None:
            continue
        seq = int(seq)
        op = name.split(":", 1)[-1].strip() if ":" in name else name
        if op not in seq_to_ops[seq]:
            seq_to_ops[seq].append(op)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_check_backward_seq_id_uniqueness`, which implements differentiation-time behavior that complements the forward path. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_check_backward_seq_id_uniqueness`，其作用是实现与前向路径配套的求导期行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 293-305 / 第 293-305 行
````python
    violations = []
    for seq, ops in seq_to_ops.items():
        if len(ops) > 1:
            violations.append(
                Violation(
                    rule_name="_check_backward_seq_id_uniqueness",
                    message=(
                        f"Sequence number {seq} shared by {len(ops)} backward "
                        f"ops: {ops}"
                    ),
                )
            )
    return violations
````
- **EN**: This chunk continues `_check_backward_seq_id_uniqueness` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_check_backward_seq_id_uniqueness`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 308-323 / 第 308-323 行
````python
_RULES: list[Callable[[list[dict]], list[Violation]]] = [
    _check_gpu_kernel_causality,
    _check_stream_wait_corr_id_populated,
    _check_stream_sync_overlap,
    _check_stream_wait_corr_id_in_past,
    _check_nccl_metadata,
    _check_backward_seq_id_uniqueness,
]


def validate_trace(trace_path: str) -> tuple[bool, list[Violation]]:
    """
    Run all validation rules against a Chrome trace JSON file.

    Args:
        trace_path: Path to ``.pt.trace.json`` or ``.pt.trace.json.gz``.
````
- **EN**: This chunk defines `validate_trace`, which coordinates graph-oriented state needed for tracing, capture, or compilation.
- **CN**: 这一段定义了 `validate_trace`，其作用是协调 tracing、捕获或编译所需的图相关状态。

### Lines 325-333 / 第 325-333 行
````python
    Returns:
        A ``(passed, violations)`` tuple.  ``passed`` is ``True`` when no
        violations were found.
    """
    events = _load_events(trace_path)
    all_violations: list[Violation] = []
    for rule in _RULES:
        all_violations.extend(rule(events))
    return len(all_violations) == 0, all_violations
````
- **EN**: This chunk continues `validate_trace` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `validate_trace`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Performance tracing**
  - EN: Captures operator-level timing and trace metadata for performance diagnosis.
  - CN: 捕获算子级时序与跟踪元数据，用于性能诊断。
- **Violation**
  - EN: `Violation` is one of the main symbols declared or implemented in this file.
  - CN: `Violation` 是本文件声明或实现的主要符号之一。
- **_load_events**
  - EN: `_load_events` is one of the main symbols declared or implemented in this file.
  - CN: `_load_events` 是本文件声明或实现的主要符号之一。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch.profiler._trace_validator`
- **Standard library / 标准库**: `__future__`, `dataclasses`, `gzip`, `json`, `collections`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `Violation`, `_load_events`, `_check_gpu_kernel_causality`, `_check_stream_wait_corr_id_populated`, `_check_stream_sync_overlap`, `_CUDA_EVENT_RECORD_NAMES`, `_check_stream_wait_corr_id_in_past`, `_NCCL_REQUIRED_FIELDS`, `_check_nccl_metadata`, `_check_backward_seq_id_uniqueness`
