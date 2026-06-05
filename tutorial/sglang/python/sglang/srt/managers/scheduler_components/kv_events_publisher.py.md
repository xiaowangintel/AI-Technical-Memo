# kv_events_publisher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/kv_events_publisher.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements kv events publisher logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 kv events publisher 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-11: Provide supporting module logic / 提供辅助模块逻辑
```python
import dataclasses
import time
from dataclasses import dataclass
from typing import (
    TYPE_CHECKING,
    Any,
    Callable,
    Optional,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-13: Import runtime dependencies / 导入运行时依赖
```python
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 15-18: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.disaggregation.kv_events import (
    EventPublisherFactory,
    KVEventBatch,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-22: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.distributed.parallel_state_wrapper import ParallelState
    from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 25-25: Define class SchedulerStats / 定义类 SchedulerStats
```python
class SchedulerStats: ...  # type: ignore[no-redef]
```
**EN:** This block declares the class `SchedulerStats`. It centers on coordinating kv events publisher behavior.
**CN:** 该代码块声明类 `SchedulerStats`。它负责承载与 kv events publisher 相关的核心状态与行为。

### Lines 26-28: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclasses.dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 29-37: Define class KvMetrics / 定义类 KvMetrics
```python
class KvMetrics:
    request_active_slots: int = 0
    request_total_slots: int = 0
    kv_active_blocks: int = 0
    kv_total_blocks: int = 0
    num_requests_waiting: int = 0
    gpu_cache_usage_perc: float = 0.0
    gpu_prefix_cache_hit_rate: float = 0.0
    data_parallel_rank: int = 0
```
**EN:** This block declares the class `KvMetrics`. It centers on coordinating kv events publisher behavior.
**CN:** 该代码块声明类 `KvMetrics`。它负责承载与 kv events publisher 相关的核心状态与行为。

### Lines 38-40: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass(kw_only=True, slots=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 41-97: Define class SchedulerKvEventsPublisher / 定义类 SchedulerKvEventsPublisher
```python
class SchedulerKvEventsPublisher:
    kv_events_config: Optional[str]
    ps: "ParallelState"
    attn_tp_rank: int
    attn_cp_rank: int
    attn_dp_rank: int
    dp_rank: Optional[int]
    tree_cache: "BasePrefixCache"
    send_metrics_from_scheduler: Optional["zmq.Socket"]
    max_running_requests: int
    max_total_num_tokens: int
    get_stats: Callable
    enable_kv_cache_events: bool = False
    kv_event_publisher: Any = None

    def __post_init__(self) -> None:
        self.init_kv_events(self.kv_events_config)

    def init_kv_events(self, kv_events_config: Optional[str]):
        self.enable_kv_cache_events = bool(
            kv_events_config and self.ps.attn_tp_rank == 0 and self.ps.attn_cp_rank == 0
        )

        if self.enable_kv_cache_events:
            self.kv_event_publisher = EventPublisherFactory.create(
                kv_events_config, self.ps.attn_dp_rank
            )

    def emit_kv_metrics(self):
        if not self.enable_kv_cache_events:
            return

        kv_metrics = KvMetrics()
        kv_metrics.request_active_slots = self.get_stats().num_running_reqs.total
        kv_metrics.request_total_slots = self.max_running_requests
        kv_metrics.kv_active_blocks = int(
            self.get_stats().token_usage * self.max_total_num_tokens
        )
        kv_metrics.kv_total_blocks = self.max_total_num_tokens
        kv_metrics.num_requests_waiting = self.get_stats().num_queue_reqs.total
        kv_metrics.gpu_cache_usage_perc = self.get_stats().token_usage
        kv_metrics.gpu_prefix_cache_hit_rate = self.get_stats().cache_hit_rate
        kv_metrics.data_parallel_rank = (
            self.ps.dp_rank if self.ps.dp_rank is not None else 0
        )

        if not self.send_metrics_from_scheduler.closed:
            self.send_metrics_from_scheduler.send_pyobj(kv_metrics)

    def publish_kv_events(self):
        if not self.enable_kv_cache_events:
            return

        events = self.tree_cache.take_events()
        if events:
            batch = KVEventBatch(ts=time.time(), events=events)
            self.kv_event_publisher.publish(batch)
```
**EN:** This block declares the class `SchedulerKvEventsPublisher`. It centers on coordinating kv events publisher behavior, with methods such as __post_init__, init_kv_events, emit_kv_metrics, publish_kv_events.
**CN:** 该代码块声明类 `SchedulerKvEventsPublisher`。它负责承载与 kv events publisher 相关的核心状态与行为，并通过 __post_init__, init_kv_events, emit_kv_metrics, publish_kv_events 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerStats, KvMetrics, SchedulerKvEventsPublisher
- **Domain focus / 领域焦点**: kv events publisher / kv events publisher
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, time, typing
- **Third-party / 第三方库**: __future__, zmq
- **Local Modules / 本地模块**: sglang.srt.disaggregation.kv_events, sglang.srt.distributed.parallel_state_wrapper, sglang.srt.mem_cache.base_prefix_cache
