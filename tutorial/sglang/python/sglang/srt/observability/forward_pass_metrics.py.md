# forward_pass_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/forward_pass_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `forward_pass_metrics`. The module docstring frames it as: "Forward pass metrics for per-iteration scheduler telemetry." / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `forward_pass_metrics` 的逻辑。 它对外提供的主要入口包括 `WelfordAccumulator`, `ScheduledRequestMetrics`, `QueuedRequestMetrics`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Module imports, constants, and setup
```python
"""
Forward pass metrics for per-iteration scheduler telemetry.

Emits per-iteration scheduling metrics over ZMQ PUB so that external
consumers can observe scheduler behavior in real time without polling
Prometheus.

Uses msgspec.Struct for zero-copy serialization.

Data flow::

    Scheduler process:
        SchedulerMetricsMixin._emit_forward_pass_metrics()
          -> _FpmPublisherThread -> ZMQ PUB (localhost)

    External consumer:
        ZMQ SUB -> deserialize ForwardPassMetrics

"""

from __future__ import annotations

import logging
import queue
import threading
import time
from itertools import count

import msgspec

# Schema version. Must match the consumer (Dynamo's ForwardPassMetrics).
# Bump when the schema changes incompatibly.
FPM_VERSION: int = 1

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；记录指标或追踪信号。

### Lines 38-45: Class WelfordAccumulator
```python
class WelfordAccumulator:
    """Welford's online algorithm for count / total / population-variance.

    Numerically stable single-pass computation.
    """

    __slots__ = ("count", "total", "_mean", "_m2")

```
**EN:** This range introduces `WelfordAccumulator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Welford's online algorithm for count / total / population-variance."
**CN:** 这一段引入 `WelfordAccumulator`，并定义其后续方法依赖的结构或元数据。

### Lines 46-50: Method WelfordAccumulator.__init__
```python
    def __init__(self) -> None:
        self.count = 0
        self.total = 0
        self._mean = 0.0
        self._m2 = 0.0
```
**EN:** This callable implements `WelfordAccumulator.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `WelfordAccumulator.__init__`，主要用于初始化实例状态与默认值。

### Lines 52-58: Method WelfordAccumulator.add
```python
    def add(self, v: int) -> None:
        self.count += 1
        self.total += v
        delta = v - self._mean
        self._mean += delta / self.count
        delta2 = v - self._mean
        self._m2 += delta * delta2
```
**EN:** This callable implements `WelfordAccumulator.add`. It takes `v` and mainly adds configuration entries or arguments.
**CN:** 这一可调用对象实现了 `WelfordAccumulator.add`。它接收 `v`，主要用于添加配置项或参数。

### Lines 60-63: Method WelfordAccumulator.variance
```python
    def variance(self) -> float:
        if self.count == 0:
            return 0.0
        return self._m2 / self.count
```
**EN:** This callable implements `WelfordAccumulator.variance` and mainly implements variance.
**CN:** 这一可调用对象实现了 `WelfordAccumulator.variance`，主要用于实现 variance 相关逻辑。

### Lines 66-79: Class ScheduledRequestMetrics
```python
class ScheduledRequestMetrics(
    msgspec.Struct,
    frozen=True,
    gc=False,
):
    """Metrics for requests scheduled in this iteration."""

    num_prefill_requests: int = 0
    sum_prefill_tokens: int = 0
    var_prefill_length: float = 0.0
    sum_prefill_kv_tokens: int = 0
    num_decode_requests: int = 0
    sum_decode_kv_tokens: int = 0
    var_decode_kv_tokens: float = 0.0
```
**EN:** This range introduces `ScheduledRequestMetrics` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Metrics for requests scheduled in this iteration." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `ScheduledRequestMetrics`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 82-94: Class QueuedRequestMetrics
```python
class QueuedRequestMetrics(
    msgspec.Struct,
    frozen=True,
    gc=False,
):
    """Metrics for requests waiting in the queue."""

    num_prefill_requests: int = 0
    sum_prefill_tokens: int = 0
    var_prefill_length: float = 0.0
    num_decode_requests: int = 0
    sum_decode_kv_tokens: int = 0
    var_decode_kv_tokens: float = 0.0
```
**EN:** This range introduces `QueuedRequestMetrics` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Metrics for requests waiting in the queue." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `QueuedRequestMetrics`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 97-120: Class ForwardPassMetrics
```python
class ForwardPassMetrics(
    msgspec.Struct,
    frozen=True,
    gc=False,
):
    """Per-iteration metrics emitted by the scheduler.

    One message per scheduler iteration (one per forward pass).
    ``wall_time`` is the iteration duration in seconds.
    An idle heartbeat (all zeros, wall_time=0) is emitted when the
    engine transitions from active to idle.

    Field order must match Dynamo's ``ForwardPassMetrics`` in
    ``dynamo.common.forward_pass_metrics`` — msgspec uses positional
    encoding so any mismatch silently corrupts data.
    """

    version: int = FPM_VERSION
    worker_id: str = ""
    dp_rank: int = 0
    counter_id: int = 0
    wall_time: float = 0.0
    scheduled_requests: ScheduledRequestMetrics = ScheduledRequestMetrics()
    queued_requests: QueuedRequestMetrics = QueuedRequestMetrics()
```
**EN:** This range introduces `ForwardPassMetrics` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Per-iteration metrics emitted by the scheduler." In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一段引入 `ForwardPassMetrics`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 121-126: Module-level constants and helpers
```python


_encoder = msgspec.msgpack.Encoder()
_decoder = msgspec.msgpack.Decoder(ForwardPassMetrics)


```
**EN:** This range organizes module-level state and shared setup. In this range it records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 127-128: Function encode
```python
def encode(metrics: ForwardPassMetrics) -> bytes:
    return _encoder.encode(metrics)
```
**EN:** This callable implements `encode`. It takes `metrics` and mainly implements encode. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `encode`。它接收 `metrics`，主要用于实现 encode 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 131-132: Function decode
```python
def decode(data: bytes) -> ForwardPassMetrics:
    return _decoder.decode(data)
```
**EN:** This callable implements `decode`. It takes `data` and mainly implements decode. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `decode`。它接收 `data`，主要用于实现 decode 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 135-143: Class _FpmPublisherThread
```python
class _FpmPublisherThread:
    """Background thread that serializes and sends ForwardPassMetrics over ZMQ.

    Also emits periodic heartbeats when idle.
    """

    SHUTDOWN_TIMEOUT: float = 1.0
    HEARTBEAT_INTERVAL: float = 1.0

```
**EN:** This range introduces `_FpmPublisherThread` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Background thread that serializes and sends ForwardPassMetrics over ZMQ." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `_FpmPublisherThread`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 144-169: Method _FpmPublisherThread.__init__
```python
    def __init__(
        self,
        endpoint: str,
        worker_id: str,
        dp_rank: int,
        max_queue_size: int = 10_000,
    ) -> None:
        import zmq

        self._queue: queue.Queue[ForwardPassMetrics | None] = queue.Queue(
            maxsize=max_queue_size
        )
        self._seq = count()
        self._worker_id = worker_id
        self._dp_rank = dp_rank

        self._ctx = zmq.Context()
        self._pub = self._ctx.socket(zmq.PUB)
        self._pub.bind(endpoint)
        self._zmq = zmq

        self._running = True
        self._thread = threading.Thread(
            target=self._run, daemon=True, name="fpm-zmq-publisher"
        )
        self._thread.start()
```
**EN:** This callable implements `_FpmPublisherThread.__init__`. It takes `endpoint`, `worker_id`, `dp_rank`, `max_queue_size` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_FpmPublisherThread.__init__`。它接收 `endpoint`, `worker_id`, `dp_rank`, `max_queue_size`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 171-177: Method _FpmPublisherThread.publish
```python
    def publish(self, metrics: ForwardPassMetrics) -> None:
        if not self._running:
            return
        try:
            self._queue.put_nowait(metrics)
        except queue.Full:
            pass
```
**EN:** This callable implements `_FpmPublisherThread.publish`. It takes `metrics` and mainly implements publish. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_FpmPublisherThread.publish`。它接收 `metrics`，主要用于实现 publish 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 179-190: Method _FpmPublisherThread.shutdown
```python
    def shutdown(self) -> None:
        self._running = False
        try:
            self._queue.put_nowait(None)
        except queue.Full:
            pass
        self._thread.join(timeout=self.SHUTDOWN_TIMEOUT)
        try:
            self._pub.close(linger=0)
            self._ctx.term()
        except Exception:
            pass
```
**EN:** This callable implements `_FpmPublisherThread.shutdown` and mainly implements shutdown.
**CN:** 这一可调用对象实现了 `_FpmPublisherThread.shutdown`，主要用于实现 shutdown 相关逻辑。

### Lines 192-221: Method _FpmPublisherThread._run
```python
    def _run(self) -> None:
        zmq = self._zmq
        topic = b""
        last_publish = time.monotonic()

        while self._running or not self._queue.empty():
            try:
                metrics = self._queue.get(timeout=self.HEARTBEAT_INTERVAL)
                if metrics is None:
                    break
            except queue.Empty:
                if time.monotonic() - last_publish >= self.HEARTBEAT_INTERVAL:
                    metrics = ForwardPassMetrics(
                        worker_id=self._worker_id,
                        dp_rank=self._dp_rank,
                    )
                else:
                    continue

            try:
                seq = next(self._seq)
                metrics = msgspec.structs.replace(metrics, counter_id=seq)
                payload = encode(metrics)
                seq_bytes = seq.to_bytes(8, "big")
                self._pub.send_multipart((topic, seq_bytes, payload), flags=zmq.NOBLOCK)
                last_publish = time.monotonic()
            except zmq.Again:
                pass
            except Exception:
                logger.warning("FPM publisher send failed", exc_info=True)
```
**EN:** This callable implements `_FpmPublisherThread._run` and mainly executes the main workflow. In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_FpmPublisherThread._run`，主要用于执行主要流程。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

## Key Concepts / 关键概念
- `WelfordAccumulator`: core class or state container / 核心类或状态容器
- `ScheduledRequestMetrics`: core class or state container / 核心类或状态容器
- `QueuedRequestMetrics`: core class or state container / 核心类或状态容器
- `ForwardPassMetrics`: core class or state container / 核心类或状态容器
- `encode`: implements encode / 实现 encode 相关逻辑
- `decode`: implements decode / 实现 decode 相关逻辑
- `_FpmPublisherThread`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `queue`, `threading`, `time`, `itertools`
- **Third-party / 第三方**: `msgspec`, `zmq`
