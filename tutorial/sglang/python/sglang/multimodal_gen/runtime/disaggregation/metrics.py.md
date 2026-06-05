# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `_RequestTiming`, `RoleStats`, and `DisaggMetrics`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Observability metrics for disaggregated diffusion pipelines. / 该文件属于解耦运行时层。它围绕 `_RequestTiming`、`RoleStats` 和 `DisaggMetrics` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-6: module setup and imports / 模块初始化与导入
```python
"""Observability metrics for disaggregated diffusion pipelines."""

import threading
import time
from dataclasses import dataclass
```
**EN:** This block establishes the module context and imports `threading`, `time`, and `dataclasses`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `threading`、`time` 和 `dataclasses`。这些依赖为后续实现提供所需符号。

### Lines 10-10: `_RequestTiming` class overview / `_RequestTiming` 类概览
```python
class _RequestTiming:
```
**EN:** This block defines class `_RequestTiming`. It encapsulates request timing behavior.
**CN:** 该代码块定义了类 `_RequestTiming`。 它用于封装 request timing 相关行为。

### Lines 11-12: supporting statements / 辅助语句
```python
    start_time: float
    stage_start: float = 0.0
```
**EN:** This block gathers supporting statements inside `_RequestTiming`. It updates names such as `start_time`, and `stage_start`.
**CN:** 该代码块汇集了位于 `_RequestTiming` 内部的辅助语句。 它会更新 `start_time` 和 `stage_start` 等名称。

### Lines 16-16: `RoleStats` class overview / `RoleStats` 类概览
```python
class RoleStats:
```
**EN:** This block defines class `RoleStats`. It encapsulates role stats behavior.
**CN:** 该代码块定义了类 `RoleStats`。 它用于封装 role stats 相关行为。

### Lines 17-27: supporting statements / 辅助语句
```python
    role: str
    requests_completed: int = 0
    requests_failed: int = 0
    requests_in_flight: int = 0
    requests_timed_out: int = 0
    queue_depth: int = 0
    last_latency_s: float = 0.0
    avg_latency_s: float = 0.0
    max_latency_s: float = 0.0
    throughput_rps: float = 0.0
    uptime_s: float = 0.0
```
**EN:** This block gathers supporting statements inside `RoleStats`. It updates names such as `role`, `requests_completed`, `requests_failed`, `requests_in_flight`, `requests_timed_out`, and `queue_depth`.
**CN:** 该代码块汇集了位于 `RoleStats` 内部的辅助语句。 它会更新 `role`、`requests_completed`、`requests_failed`、`requests_in_flight`、`requests_timed_out` 和 `queue_depth` 等名称。

### Lines 29-42: `to_dict` implementation / `to_dict` 实现
```python
    def to_dict(self) -> dict:
        return {
            "role": self.role,
            "requests_completed": self.requests_completed,
            "requests_failed": self.requests_failed,
            "requests_in_flight": self.requests_in_flight,
            "requests_timed_out": self.requests_timed_out,
            "queue_depth": self.queue_depth,
            "last_latency_s": round(self.last_latency_s, 4),
            "avg_latency_s": round(self.avg_latency_s, 4),
            "max_latency_s": round(self.max_latency_s, 4),
            "throughput_rps": round(self.throughput_rps, 4),
            "uptime_s": round(self.uptime_s, 1),
        }
```
**EN:** This block defines method `to_dict` on `RoleStats`. It converts to dict. Key calls include `round`.
**CN:** 该代码块定义了 `RoleStats` 的方法 `to_dict`。 它用于转换为dict。 关键调用包括 `round`。

### Lines 45-47: `DisaggMetrics` class overview / `DisaggMetrics` 类概览
```python
class DisaggMetrics:
    """Thread-safe metrics collector for a single disagg role."""
```
**EN:** This block defines class `DisaggMetrics`. Thread-safe metrics collector for a single disagg role.
**CN:** 该代码块定义了类 `DisaggMetrics`。 它用于封装 disagg metrics 相关行为。

### Lines 48-66: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, role: str):
        self._role = role
        self._lock = threading.Lock()
        self._start_time = time.monotonic()

        self._completed = 0
        self._failed = 0
        self._timed_out = 0

        self._in_flight: dict[str, _RequestTiming] = {}

        self._last_latency = 0.0
        self._max_latency = 0.0
        self._total_latency = 0.0

        self._completion_times: list[float] = []
        self._throughput_window_s = 60.0

        self._queue_depth = 0
```
**EN:** This block defines method `__init__` on `DisaggMetrics`. It initializes the instance state. Key calls include `threading.Lock`, and `time.monotonic`. Parameters such as `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `threading.Lock` 和 `time.monotonic`。 本段逻辑主要由 `role` 等参数驱动。

### Lines 68-70: `role` implementation / `role` 实现
```python
    @property
    def role(self) -> str:
        return self._role
```
**EN:** This block defines method `role` on `DisaggMetrics`. It handles role logic.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `role`。 它用于处理 role 相关逻辑。

### Lines 72-74: `record_request_start` implementation / `record_request_start` 实现
```python
    def record_request_start(self, request_id: str) -> None:
        with self._lock:
            self._in_flight[request_id] = _RequestTiming(start_time=time.monotonic())
```
**EN:** This block defines method `record_request_start` on `DisaggMetrics`. It handles record request start logic. Key calls include `_RequestTiming`, and `time.monotonic`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `record_request_start`。 它用于处理 record request start 相关逻辑。 关键调用包括 `_RequestTiming` 和 `time.monotonic`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 76-88: `record_request_complete` implementation / `record_request_complete` 实现
```python
    def record_request_complete(self, request_id: str) -> None:
        now = time.monotonic()
        with self._lock:
            timing = self._in_flight.pop(request_id, None)
            if timing is not None:
                latency = now - timing.start_time
                self._last_latency = latency
                self._max_latency = max(self._max_latency, latency)
                self._total_latency += latency

            self._completed += 1
            self._completion_times.append(now)
            self._prune_completion_times(now)
```
**EN:** This block defines method `record_request_complete` on `DisaggMetrics`. It handles record request complete logic. Key calls include `time.monotonic`, `self._in_flight.pop`, `self._completion_times.append`, `self._prune_completion_times`, and `max`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `record_request_complete`。 它用于处理 record request complete 相关逻辑。 关键调用包括 `time.monotonic`、`self._in_flight.pop`、`self._completion_times.append`、`self._prune_completion_times` 和 `max`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 90-93: `record_request_failed` implementation / `record_request_failed` 实现
```python
    def record_request_failed(self, request_id: str) -> None:
        with self._lock:
            self._in_flight.pop(request_id, None)
            self._failed += 1
```
**EN:** This block defines method `record_request_failed` on `DisaggMetrics`. It handles record request failed logic. Key calls include `self._in_flight.pop`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `record_request_failed`。 它用于处理 record request failed 相关逻辑。 关键调用包括 `self._in_flight.pop`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 95-98: `record_request_timeout` implementation / `record_request_timeout` 实现
```python
    def record_request_timeout(self, request_id: str) -> None:
        with self._lock:
            self._in_flight.pop(request_id, None)
            self._timed_out += 1
```
**EN:** This block defines method `record_request_timeout` on `DisaggMetrics`. It handles record request timeout logic. Key calls include `self._in_flight.pop`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `record_request_timeout`。 它用于处理 record request timeout 相关逻辑。 关键调用包括 `self._in_flight.pop`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 100-102: `update_queue_depth` implementation / `update_queue_depth` 实现
```python
    def update_queue_depth(self, depth: int) -> None:
        with self._lock:
            self._queue_depth = depth
```
**EN:** This block defines method `update_queue_depth` on `DisaggMetrics`. It updates queue depth. The implementation uses context-managed resources. Parameters such as `depth` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `update_queue_depth`。 它用于更新queue depth。 实现中使用上下文管理资源。 本段逻辑主要由 `depth` 等参数驱动。

### Lines 104-128: `snapshot` implementation / `snapshot` 实现
```python
    def snapshot(self) -> RoleStats:
        now = time.monotonic()
        with self._lock:
            self._prune_completion_times(now)
            total = self._completed + self._failed
            avg_latency = self._total_latency / total if total > 0 else 0.0
            rps = (
                len(self._completion_times) / self._throughput_window_s
                if self._completion_times
                else 0.0
            )

            return RoleStats(
                role=self._role,
                requests_completed=self._completed,
                requests_failed=self._failed,
                requests_in_flight=len(self._in_flight),
                requests_timed_out=self._timed_out,
                queue_depth=self._queue_depth,
                last_latency_s=self._last_latency,
                avg_latency_s=avg_latency,
                max_latency_s=self._max_latency,
                throughput_rps=rps,
                uptime_s=now - self._start_time,
            )
```
**EN:** This block defines method `snapshot` on `DisaggMetrics`. It handles snapshot logic. Key calls include `time.monotonic`, `self._prune_completion_times`, `RoleStats`, and `len`. The implementation uses context-managed resources.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `snapshot`。 它用于处理 snapshot 相关逻辑。 关键调用包括 `time.monotonic`、`self._prune_completion_times`、`RoleStats` 和 `len`。 实现中使用上下文管理资源。

### Lines 130-133: `_prune_completion_times` implementation / `_prune_completion_times` 实现
```python
    def _prune_completion_times(self, now: float) -> None:
        cutoff = now - self._throughput_window_s
        while self._completion_times and self._completion_times[0] < cutoff:
            self._completion_times.pop(0)
```
**EN:** This block defines method `_prune_completion_times` on `DisaggMetrics`. It handles prune completion times logic. Key calls include `self._completion_times.pop`. The implementation iterates over collections or steps. Parameters such as `now` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggMetrics` 的方法 `_prune_completion_times`。 它用于处理 prune completion times 相关逻辑。 关键调用包括 `self._completion_times.pop`。 实现中会遍历集合或步骤。 本段逻辑主要由 `now` 等参数驱动。

## Key Concepts / 关键概念
- `_RequestTiming`: Primary class that encapsulates request timing behavior. / 核心类，用于封装 request timing 相关行为。
- `RoleStats`: Primary class that encapsulates role stats behavior. / 核心类，用于封装 role stats 相关行为。
- `DisaggMetrics`: Thread-safe metrics collector for a single disagg role. / 核心类，用于封装 disagg metrics 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `time`, `dataclasses`

- **Total lines / 总行数**: 133
