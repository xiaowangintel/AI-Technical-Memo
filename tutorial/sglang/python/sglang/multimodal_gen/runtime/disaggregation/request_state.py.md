# request_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/request_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `RequestState`, `RequestRecord`, and `RequestTracker`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Request state machine for disaggregated diffusion pipelines. / 该文件属于解耦运行时层。它围绕 `RequestState`、`RequestRecord` 和 `RequestTracker` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-8: module setup and imports / 模块初始化与导入
```python
"""Request state machine for disaggregated diffusion pipelines."""

import enum
import logging
import threading
import time
from dataclasses import dataclass, field
```
**EN:** This block establishes the module context and imports `enum`, `logging`, `threading`, `time`, and `dataclasses`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `enum`、`logging`、`threading`、`time` 和 `dataclasses`。这些依赖为后续实现提供所需符号。

### Lines 10-10: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 13-19: `RequestState` class overview / `RequestState` 类概览
```python
class RequestState(enum.Enum):
    """Lifecycle states for a disagg pipeline request.

    *_WAITING: request queued, awaiting a free buffer slot.
    *_RUNNING: request dispatched to a specific instance.
    """
```
**EN:** This block defines class `RequestState`. Lifecycle states for a disagg pipeline request. *_WAITING: request queued, awaiting a free buffer slot. It inherits from `enum.Enum`.
**CN:** 该代码块定义了类 `RequestState`。 它用于封装 request state 相关行为。 它继承自 `enum.Enum`。

### Lines 20-31: supporting statements / 辅助语句
```python
    PENDING = "pending"
    ENCODER_WAITING = "encoder_waiting"
    ENCODER_RUNNING = "encoder_running"
    ENCODER_DONE = "encoder_done"
    DENOISING_WAITING = "denoising_waiting"
    DENOISING_RUNNING = "denoising_running"
    DENOISING_DONE = "denoising_done"
    DECODER_WAITING = "decoder_waiting"
    DECODER_RUNNING = "decoder_running"
    DONE = "done"
    FAILED = "failed"
    TIMED_OUT = "timed_out"
```
**EN:** This block gathers supporting statements inside `RequestState`. It updates names such as `PENDING`, `ENCODER_WAITING`, `ENCODER_RUNNING`, `ENCODER_DONE`, `DENOISING_WAITING`, and `DENOISING_RUNNING`.
**CN:** 该代码块汇集了位于 `RequestState` 内部的辅助语句。 它会更新 `PENDING`、`ENCODER_WAITING`、`ENCODER_RUNNING`、`ENCODER_DONE`、`DENOISING_WAITING` 和 `DENOISING_RUNNING` 等名称。

### Lines 34-55: supporting statements / 辅助语句
```python
_TERMINAL_STATES = {RequestState.DONE, RequestState.FAILED, RequestState.TIMED_OUT}
_ACTIVE_STATES = set(RequestState) - _TERMINAL_STATES

# Normal (non-failure) transitions.  FAILED and TIMED_OUT are handled
# separately in transition() — any active state can reach them.
_VALID_TRANSITIONS: dict[RequestState, set[RequestState]] = {
    RequestState.PENDING: {RequestState.ENCODER_WAITING, RequestState.ENCODER_RUNNING},
    RequestState.ENCODER_WAITING: {RequestState.ENCODER_RUNNING},
    RequestState.ENCODER_RUNNING: {RequestState.ENCODER_DONE},
    RequestState.ENCODER_DONE: {
        RequestState.DENOISING_WAITING,
        RequestState.DENOISING_RUNNING,
    },
    RequestState.DENOISING_WAITING: {RequestState.DENOISING_RUNNING},
    RequestState.DENOISING_RUNNING: {RequestState.DENOISING_DONE},
    RequestState.DENOISING_DONE: {
        RequestState.DECODER_WAITING,
        RequestState.DECODER_RUNNING,
    },
    RequestState.DECODER_WAITING: {RequestState.DECODER_RUNNING},
    RequestState.DECODER_RUNNING: {RequestState.DONE},
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_TERMINAL_STATES`, `_ACTIVE_STATES`, and `_VALID_TRANSITIONS`. The code collaborates with `set`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_TERMINAL_STATES`、`_ACTIVE_STATES` 和 `_VALID_TRANSITIONS` 等名称。 代码会与 `set` 协同工作。

### Lines 59-59: `RequestRecord` class overview / `RequestRecord` 类概览
```python
class RequestRecord:
```
**EN:** This block defines class `RequestRecord`. It encapsulates request record behavior.
**CN:** 该代码块定义了类 `RequestRecord`。 它用于封装 request record 相关行为。

### Lines 60-67: supporting statements / 辅助语句
```python
    request_id: str
    state: RequestState = RequestState.PENDING
    submit_time: float = field(default_factory=time.monotonic)
    last_transition_time: float = field(default_factory=time.monotonic)
    encoder_instance: int | None = None
    denoiser_instance: int | None = None
    decoder_instance: int | None = None
    error: str | None = None
```
**EN:** This block gathers supporting statements inside `RequestRecord`. It updates names such as `request_id`, `state`, `submit_time`, `last_transition_time`, `encoder_instance`, and `denoiser_instance`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `RequestRecord` 内部的辅助语句。 它会更新 `request_id`、`state`、`submit_time`、`last_transition_time`、`encoder_instance` 和 `denoiser_instance` 等名称。 代码会与 `field` 协同工作。

### Lines 69-70: `elapsed_s` implementation / `elapsed_s` 实现
```python
    def elapsed_s(self) -> float:
        return time.monotonic() - self.submit_time
```
**EN:** This block defines method `elapsed_s` on `RequestRecord`. It handles elapsed s logic. Key calls include `time.monotonic`.
**CN:** 该代码块定义了 `RequestRecord` 的方法 `elapsed_s`。 它用于处理 elapsed s 相关逻辑。 关键调用包括 `time.monotonic`。

### Lines 72-73: `is_terminal` implementation / `is_terminal` 实现
```python
    def is_terminal(self) -> bool:
        return self.state in _TERMINAL_STATES
```
**EN:** This block defines method `is_terminal` on `RequestRecord`. It handles is terminal logic.
**CN:** 该代码块定义了 `RequestRecord` 的方法 `is_terminal`。 它用于处理 is terminal 相关逻辑。

### Lines 76-78: `RequestTracker` class overview / `RequestTracker` 类概览
```python
class RequestTracker:
    """Thread-safe tracker for request state machines."""
```
**EN:** This block defines class `RequestTracker`. Thread-safe tracker for request state machines.
**CN:** 该代码块定义了类 `RequestTracker`。 它用于封装 request tracker 相关行为。

### Lines 79-81: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        self._lock = threading.Lock()
        self._requests: dict[str, RequestRecord] = {}
```
**EN:** This block defines method `__init__` on `RequestTracker`. It initializes the instance state. Key calls include `threading.Lock`.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `threading.Lock`。

### Lines 83-89: `submit` implementation / `submit` 实现
```python
    def submit(self, request_id: str) -> RequestRecord:
        with self._lock:
            if request_id in self._requests:
                raise ValueError(f"Duplicate request_id: {request_id}")
            record = RequestRecord(request_id=request_id)
            self._requests[request_id] = record
            return record
```
**EN:** This block defines method `submit` on `RequestTracker`. It handles submit logic. Key calls include `RequestRecord`, and `ValueError`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `submit`。 它用于处理 submit 相关逻辑。 关键调用包括 `RequestRecord` 和 `ValueError`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 91-135: `transition` implementation / `transition` 实现
```python
    def transition(
        self,
        request_id: str,
        new_state: RequestState,
        *,
        error: str | None = None,
        encoder_instance: int | None = None,
        denoiser_instance: int | None = None,
        decoder_instance: int | None = None,
    ) -> RequestRecord:
        with self._lock:
            record = self._requests.get(request_id)
            if record is None:
                raise ValueError(f"Unknown request_id: {request_id}")

            old_state = record.state

            if new_state in _TERMINAL_STATES and new_state != RequestState.DONE:
                # FAILED / TIMED_OUT: allowed from any active state
                if old_state not in _ACTIVE_STATES:
                    raise ValueError(
                        f"Cannot transition {request_id} from terminal state "
                        f"{old_state.value} to {new_state.value}"
                    )
            elif new_state not in _VALID_TRANSITIONS.get(old_state, set()):
                raise ValueError(
                    f"Invalid transition for {request_id}: "
                    f"{old_state.value} -> {new_state.value}"
                )

            record.state = new_state
            record.last_transition_time = time.monotonic()
            if error is not None:
                record.error = error
            if encoder_instance is not None:
                record.encoder_instance = encoder_instance
            if denoiser_instance is not None:
                record.denoiser_instance = denoiser_instance
            if decoder_instance is not None:
                record.decoder_instance = decoder_instance

            logger.debug(
                "Request %s: %s -> %s", request_id, old_state.value, new_state.value
            )
            return record
```
**EN:** This block defines method `transition` on `RequestTracker`. It handles transition logic. Key calls include `self._requests.get`, `time.monotonic`, `logger.debug`, `ValueError`, and `_VALID_TRANSITIONS.get`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id`, and `new_state` drive the behavior in this section.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `transition`。 它用于处理 transition 相关逻辑。 关键调用包括 `self._requests.get`、`time.monotonic`、`logger.debug`、`ValueError` 和 `_VALID_TRANSITIONS.get`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 和 `new_state` 等参数驱动。

### Lines 137-139: `get` implementation / `get` 实现
```python
    def get(self, request_id: str) -> RequestRecord | None:
        with self._lock:
            return self._requests.get(request_id)
```
**EN:** This block defines method `get` on `RequestTracker`. It retrieves function. Key calls include `self._requests.get`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `get`。 它用于获取函数。 关键调用包括 `self._requests.get`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 141-143: `remove` implementation / `remove` 实现
```python
    def remove(self, request_id: str) -> RequestRecord | None:
        with self._lock:
            return self._requests.pop(request_id, None)
```
**EN:** This block defines method `remove` on `RequestTracker`. It handles remove logic. Key calls include `self._requests.pop`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `remove`。 它用于处理 remove 相关逻辑。 关键调用包括 `self._requests.pop`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 145-152: `find_timed_out` implementation / `find_timed_out` 实现
```python
    def find_timed_out(self, timeout_s: float) -> list[str]:
        now = time.monotonic()
        with self._lock:
            return [
                r.request_id
                for r in self._requests.values()
                if r.state in _ACTIVE_STATES and (now - r.submit_time) > timeout_s
            ]
```
**EN:** This block defines method `find_timed_out` on `RequestTracker`. It handles find timed out logic. Key calls include `time.monotonic`, and `self._requests.values`. The implementation uses context-managed resources. Parameters such as `timeout_s` drive the behavior in this section.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `find_timed_out`。 它用于处理 find timed out 相关逻辑。 关键调用包括 `time.monotonic` 和 `self._requests.values`。 实现中使用上下文管理资源。 本段逻辑主要由 `timeout_s` 等参数驱动。

### Lines 154-165: `snapshot` implementation / `snapshot` 实现
```python
    def snapshot(self) -> dict:
        with self._lock:
            state_counts = {}
            for r in self._requests.values():
                state_counts[r.state.value] = state_counts.get(r.state.value, 0) + 1
            return {
                "total": len(self._requests),
                "active": sum(
                    1 for r in self._requests.values() if not r.is_terminal()
                ),
                "by_state": state_counts,
            }
```
**EN:** This block defines method `snapshot` on `RequestTracker`. It handles snapshot logic. Key calls include `self._requests.values`, `len`, `sum`, `state_counts.get`, and `r.is_terminal`. The implementation iterates over collections or steps, uses context-managed resources.
**CN:** 该代码块定义了 `RequestTracker` 的方法 `snapshot`。 它用于处理 snapshot 相关逻辑。 关键调用包括 `self._requests.values`、`len`、`sum`、`state_counts.get` 和 `r.is_terminal`。 实现中会遍历集合或步骤，使用上下文管理资源。

## Key Concepts / 关键概念
- `RequestState`: Lifecycle states for a disagg pipeline request. / 核心类，用于封装 request state 相关行为。
- `RequestRecord`: Primary class that encapsulates request record behavior. / 核心类，用于封装 request record 相关行为。
- `RequestTracker`: Thread-safe tracker for request state machines. / 核心类，用于封装 request tracker 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`, `logging`, `threading`, `time`, `dataclasses`

- **Total lines / 总行数**: 165
