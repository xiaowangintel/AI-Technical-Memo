# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/offloading/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from dataclasses import dataclass, field

from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorMetadata,
    KVConnectorWorkerMetadata,
)
from vllm.v1.kv_offload.worker.worker import TransferSpec
```
**EN:** This block imports `dataclasses`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.kv_offload.worker.worker` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.kv_offload.worker.worker`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
ReqId = str
```
**EN:** This section defines module-level aliases, constants, or shared state such as `ReqId`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `ReqId`，供后续代码复用。

### Class `TransferJob` / 类 `TransferJob`
```python
@dataclass
class TransferJob:
    """A transfer job bundling request context with transfer spec.

    Used for both loads and stores, keyed by scheduler-assigned job ID.
    The worker reports the job ID back when the transfer finishes,
    and the scheduler processes the completion.
    """

    req_id: ReqId
    transfer_spec: TransferSpec
```
**EN:** Declares `TransferJob`, a dataclass. It packages structured data fields such as `req_id`, `transfer_spec`. The docstring summarizes its role as: A transfer job bundling request context with transfer spec.
**CN:** 声明 `TransferJob`，它是一个数据类。 它封装了 `req_id`, `transfer_spec` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `OffloadingConnectorMetadata` / 类 `OffloadingConnectorMetadata`
```python
@dataclass
class OffloadingConnectorMetadata(KVConnectorMetadata):
    # Keyed by scheduler-assigned job IDs.
    load_jobs: dict[int, TransferJob]
    store_jobs: dict[int, TransferJob]
    jobs_to_flush: set[int] | None = None
```
**EN:** Declares `OffloadingConnectorMetadata`, a dataclass derived from `KVConnectorMetadata`. It packages structured data fields such as `load_jobs`, `store_jobs`, `jobs_to_flush`.
**CN:** 声明 `OffloadingConnectorMetadata`，它是一个数据类，继承自 `KVConnectorMetadata`。 它封装了 `load_jobs`, `store_jobs`, `jobs_to_flush` 等结构化字段。

### Class `OffloadingWorkerMetadata` / 类 `OffloadingWorkerMetadata`
```python
@dataclass
class OffloadingWorkerMetadata(KVConnectorWorkerMetadata):
    """Worker -> Scheduler metadata for completed transfer jobs.

    Each worker reports {job_id: 1} for newly completed transfer jobs
    (load or store). aggregate() sums counts across workers within a step.
    The scheduler accumulates across steps and processes
    a transfer completion only when count reaches num_workers.
    """

    completed_jobs: dict[int, int] = field(default_factory=dict)

    def mark_completed(self, job_id: int) -> None:
        """Record a transfer job completion from this worker."""
        self.completed_jobs[job_id] = 1

    def aggregate(
        self, other: "KVConnectorWorkerMetadata"
    ) -> "KVConnectorWorkerMetadata":
        assert isinstance(other, OffloadingWorkerMetadata)

        merged = dict(self.completed_jobs)
        for job_id, v in other.completed_jobs.items():
            merged[job_id] = merged.get(job_id, 0) + v

        return OffloadingWorkerMetadata(completed_jobs=merged)
```
**EN:** Declares `OffloadingWorkerMetadata`, a dataclass derived from `KVConnectorWorkerMetadata`. It packages structured data fields such as `completed_jobs`. The docstring summarizes its role as: Worker -> Scheduler metadata for completed transfer jobs.
**CN:** 声明 `OffloadingWorkerMetadata`，它是一个数据类，继承自 `KVConnectorWorkerMetadata`。 它封装了 `completed_jobs` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `TransferJob`: dataclass interface or data carrier / `TransferJob`：数据类接口或数据载体
- `OffloadingConnectorMetadata`: dataclass interface or data carrier / `OffloadingConnectorMetadata`：数据类接口或数据载体
- `OffloadingWorkerMetadata`: dataclass interface or data carrier / `OffloadingWorkerMetadata`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.kv_offload.worker.worker`
