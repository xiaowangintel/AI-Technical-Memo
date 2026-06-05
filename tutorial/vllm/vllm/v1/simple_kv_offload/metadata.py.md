# metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/simple_kv_offload/metadata.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Metadata for SimpleCPUOffloadConnector. / 该模块位于 `simple_kv_offload` 子系统，主要围绕 `SimpleCPUOffloadMetadata`, `SimpleCPUOffloadWorkerMetadata` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Metadata for SimpleCPUOffloadConnector."""

from dataclasses import dataclass, field

from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorMetadata,
    KVConnectorWorkerMetadata,
)

INVALID_JOB_ID = -1
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `INVALID_JOB_ID`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `INVALID_JOB_ID`。

### `SimpleCPUOffloadMetadata` class / `SimpleCPUOffloadMetadata` 类
```python
@dataclass
class SimpleCPUOffloadMetadata(KVConnectorMetadata):
    """
    Metadata passed from scheduler to worker for CPU offload operations.

    The worker receives flat block lists keyed by a monotonic event_idx.
    Job->req_id translation is handled by the scheduler-side manager
    (via inverse maps), so the worker never knows about request identities.
    """

    # Load event per step. INVALID_JOB_ID means no blocks to load this step.
    load_event: int = INVALID_JOB_ID
    load_gpu_blocks: list[int] = field(default_factory=list)
    load_cpu_blocks: list[int] = field(default_factory=list)
    # Reverse map: load_event->req_ids, for tracking requests with finished load events
    load_event_to_reqs: dict[int, list[str]] = field(default_factory=dict)

    # Store event per step. INVALID_JOB_ID means no blocks to store this step.
    store_event: int = INVALID_JOB_ID
    store_gpu_blocks: list[int] = field(default_factory=list)
    store_cpu_blocks: list[int] = field(default_factory=list)

    # Whether any requests were preempted this step and need flush pending transfers.
    need_flush: bool = False
```
**EN:** Uses `@dataclass` to package related state for `SimpleCPUOffloadMetadata`. Typical fields include `load_event`, `load_gpu_blocks`, `load_cpu_blocks`, `load_event_to_reqs`, `store_event`, `store_gpu_blocks`.
**CN:** `SimpleCPUOffloadMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `load_event`, `load_gpu_blocks`, `load_cpu_blocks`, `load_event_to_reqs`, `store_event`, `store_gpu_blocks`。

### `SimpleCPUOffloadWorkerMetadata` class / `SimpleCPUOffloadWorkerMetadata` 类
```python
@dataclass
class SimpleCPUOffloadWorkerMetadata(KVConnectorWorkerMetadata):
    """Worker -> Scheduler metadata for completed store events.

    Each worker reports {event_idx: 1} for newly completed stores.
    ``aggregate()`` sums counts across workers within a step.
    The scheduler-side manager accumulates across steps and processes
    a store completion only when count reaches ``world_size``.
    """

    completed_store_events: dict[int, int]
```
**EN:** Uses `@dataclass` to package related state for `SimpleCPUOffloadWorkerMetadata`. Typical fields include `completed_store_events`.
**CN:** `SimpleCPUOffloadWorkerMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `completed_store_events`。

### `SimpleCPUOffloadWorkerMetadata.aggregate` method / `SimpleCPUOffloadWorkerMetadata.aggregate` 方法
```python
    def aggregate(
        self, other: "KVConnectorWorkerMetadata"
    ) -> "KVConnectorWorkerMetadata":
        assert isinstance(other, SimpleCPUOffloadWorkerMetadata)
        merged = dict(self.completed_store_events)
        for k, v in other.completed_store_events.items():
            merged[k] = merged.get(k, 0) + v
        return SimpleCPUOffloadWorkerMetadata(completed_store_events=merged)
```
**EN:** This method implements `aggregate` within `SimpleCPUOffloadWorkerMetadata`. Key calls include `isinstance`, `dict`, `items`, `SimpleCPUOffloadWorkerMetadata`, `get`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `aggregate`，其作用域位于`SimpleCPUOffloadWorkerMetadata`。 关键调用包括 `isinstance`, `dict`, `items`, `SimpleCPUOffloadWorkerMetadata`, `get`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `SimpleCPUOffloadMetadata`: central class or interface in this module. / `SimpleCPUOffloadMetadata`：本模块中的核心类或接口。
- `SimpleCPUOffloadWorkerMetadata`: central class or interface in this module. / `SimpleCPUOffloadWorkerMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- Internal vLLM / 内部依赖: `vllm.distributed.kv_transfer.kv_connector.v1.base`
