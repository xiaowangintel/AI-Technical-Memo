# stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/nixl/stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Stats and Prometheus metrics for the NIXL connector / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Stats and Prometheus metrics for the NIXL connector."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Stats and Prometheus metrics for the NIXL connector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import copy
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any

import numpy as np

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.metrics import (
    KVConnectorPromMetrics,
    KVConnectorStats,
    PromMetric,
    PromMetricT,
)
from vllm.v1.metrics.utils import create_metric_per_engine
```
**EN:** This block imports `copy`, `dataclasses`, `typing`, `numpy`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `copy`, `dataclasses`, `typing`, `numpy`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.distributed.nixl_utils import nixlXferTelemetry
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Class `NixlKVConnectorStats` / 类 `NixlKVConnectorStats`
```python
@dataclass
class NixlKVConnectorStats(KVConnectorStats):
    """Container for transfer performance metrics"""

    def __post_init__(self):
        if not self.data:
            # Empty container init, no data is passed in.
            self.reset()

    def reset(self):
        # Must be serializable
        self.data: dict[str, list[float | int]] = {
            "transfer_duration": [],
            "post_duration": [],
            "bytes_transferred": [],
            "num_descriptors": [],
            "num_failed_transfers": [],
            "num_failed_notifications": [],
            "num_kv_expired_reqs": [],
        }

    def record_transfer(self, res: "nixlXferTelemetry"):
        # Keep metrics units consistent with rest of the code: time us->s
        self.data["transfer_duration"].append(res.xferDuration / 1e6)
        self.data["post_duration"].append(res.postDuration / 1e6)
        self.data["bytes_transferred"].append(res.totalBytes)
        self.data["num_descriptors"].append(res.descCount)

    def record_failed_transfer(self):
        """Record a failed NIXL transfer operation."""
        self.data["num_failed_transfers"].append(1)

    def record_failed_notification(self):
        """Record a failed NIXL notification (send_notif)."""
        self.data["num_failed_notifications"].append(1)

    def record_kv_expired_req(self):
        """Record a request that had its KV blocks expire."""
        self.data["num_kv_expired_reqs"].append(1)

    def clone_and_reset(self) -> "NixlKVConnectorStats":
        old = copy.copy(self)
        self.reset()
        return old
# ... truncated for analysis ...
            "Avg post time (ms)": round(post_time.mean() * 1e3, 3),
            "P90 post time (ms)": round(np.percentile(post_time, 90).item() * 1e3, 3),
            "Avg MB per transfer": round(avg_mb, 3),
            "Throughput (MB/s)": round(throughput_mb_s, 3),
            "Avg number of descriptors": round(descs.mean(), 1),
        }

    @property
    def num_successful_transfers(self) -> int:
        return len(self.data["transfer_duration"])
```
**EN:** Declares `NixlKVConnectorStats`, a dataclass derived from `KVConnectorStats`. Key methods include `__post_init__`, `reset`, `record_transfer`, `record_failed_transfer`, `record_failed_notification`. The docstring summarizes its role as: Container for transfer performance metrics.
**CN:** 声明 `NixlKVConnectorStats`，它是一个数据类，继承自 `KVConnectorStats`。 关键方法包括 `__post_init__`, `reset`, `record_transfer`, `record_failed_transfer`, `record_failed_notification`。 文档字符串概括了它在整体流程中的职责。

### Class `NixlPromMetrics` / 类 `NixlPromMetrics`
```python
class NixlPromMetrics(KVConnectorPromMetrics):
    def __init__(
        self,
        vllm_config: VllmConfig,
        metric_types: dict[type[PromMetric], type[PromMetricT]],
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        super().__init__(vllm_config, metric_types, labelnames, per_engine_labelvalues)

        buckets = [
            0.001,
            0.005,
            0.01,
            0.025,
            0.05,
            0.075,
            0.1,
            0.2,
            0.3,
            0.5,
            0.75,
            1.0,
            5.0,
        ]
        nixl_histogram_xfer_time = self._histogram_cls(
            name="vllm:nixl_xfer_time_seconds",
            documentation="Histogram of transfer duration for NIXL KV Cache transfers.",
            buckets=buckets[1:],
            labelnames=labelnames,
        )
        self.nixl_histogram_xfer_time = create_metric_per_engine(
            nixl_histogram_xfer_time, self.per_engine_labelvalues
        )
        nixl_histogram_post_time = self._histogram_cls(
            name="vllm:nixl_post_time_seconds",
            documentation="Histogram of transfer post time for NIXL KV"
            " Cache transfers.",
            buckets=buckets,
            labelnames=labelnames,
        )
        self.nixl_histogram_post_time = create_metric_per_engine(
            nixl_histogram_post_time, self.per_engine_labelvalues
        )
        # uniform 2kb to 16gb range
# ... truncated for analysis ...
        for counter_obj, counter_item_key in zip(
            [
                self.counter_nixl_num_failed_transfers,
                self.counter_nixl_num_failed_notifications,
                self.counter_nixl_num_kv_expired_reqs,
            ],
            ["num_failed_transfers", "num_failed_notifications", "num_kv_expired_reqs"],
        ):
            for list_item in transfer_stats_data[counter_item_key]:
                counter_obj[engine_idx].inc(list_item)
```
**EN:** Declares `NixlPromMetrics`, a class derived from `KVConnectorPromMetrics`. Key methods include `__init__`, `observe`.
**CN:** 声明 `NixlPromMetrics`，它是一个类，继承自 `KVConnectorPromMetrics`。 关键方法包括 `__init__`, `observe`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `NixlKVConnectorStats`: dataclass interface or data carrier / `NixlKVConnectorStats`：数据类接口或数据载体
- `NixlPromMetrics`: class interface or data carrier / `NixlPromMetrics`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `dataclasses`, `typing`
- **Third-party / 第三方**: `numpy`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.v1.metrics.utils`, `vllm.distributed.nixl_utils`
