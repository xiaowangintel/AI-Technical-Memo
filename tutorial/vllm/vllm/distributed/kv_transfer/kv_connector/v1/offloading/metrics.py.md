# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/offloading/metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from dataclasses import dataclass
from typing import Any

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.metrics import (
    KVConnectorPromMetrics,
    KVConnectorStats,
    PromMetric,
    PromMetricT,
)
from vllm.logger import init_logger
from vllm.v1.kv_offload.worker.worker import TransferType
```
**EN:** This block imports `dataclasses`, `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.logger`, `vllm.v1.kv_offload.worker.worker` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.logger`, `vllm.v1.kv_offload.worker.worker`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `OffloadingOperationMetrics` / 类 `OffloadingOperationMetrics`
```python
@dataclass
class OffloadingOperationMetrics:
    op_size: int
    op_time: float
```
**EN:** Declares `OffloadingOperationMetrics`, a dataclass. It packages structured data fields such as `op_size`, `op_time`.
**CN:** 声明 `OffloadingOperationMetrics`，它是一个数据类。 它封装了 `op_size`, `op_time` 等结构化字段。

### Class `OffloadingConnectorStats` / 类 `OffloadingConnectorStats`
```python
@dataclass
class OffloadingConnectorStats(KVConnectorStats):
    def __post_init__(self):
        if not self.data:
            # Empty container init, no data is passed in.
            self.reset()

    def reset(self):
        self.data: dict[str, list[OffloadingOperationMetrics]] = {}

    def aggregate(self, other: KVConnectorStats) -> KVConnectorStats:
        if not other.is_empty():
            for k, v in other.data.items():
                if k not in self.data:
                    self.data[k] = v
                else:
                    accumulator = self.data[k]
                    assert isinstance(accumulator, list)
                    accumulator.extend(v)
        return self

    def reduce(self) -> dict[str, int | float]:
        """
        Reduce the observations collected during a time interval to one or
        more representative values (eg avg/median/sum of the series).
        This is meant to be called by the logger to produce a summary of the
        stats for the last time interval.
        """
        return_dict: dict[str, int | float] = {}
        for transfer_type, ops_list in self.data.items():
            assert isinstance(ops_list, list)
            total_bytes = 0
            total_time = 0.0
            for op in ops_list:
                assert isinstance(op, dict)
                total_bytes += op["op_size"]
                total_time += op["op_time"]
            return_dict[f"{transfer_type}_total_bytes"] = total_bytes
            return_dict[f"{transfer_type}_total_time"] = total_time
        return return_dict

    def is_empty(self) -> bool:
        return not self.data

    def record_transfer(self, num_bytes: int, time: float, transfer_type: TransferType):
        src, dst = transfer_type
        transfer_type_key = src + "_to_" + dst
        op = OffloadingOperationMetrics(num_bytes, time)
        if transfer_type_key in self.data:
            self.data[transfer_type_key].append(op)
        else:
            self.data[transfer_type_key] = [op]
```
**EN:** Declares `OffloadingConnectorStats`, a dataclass derived from `KVConnectorStats`. Key methods include `__post_init__`, `reset`, `aggregate`, `reduce`, `is_empty`.
**CN:** 声明 `OffloadingConnectorStats`，它是一个数据类，继承自 `KVConnectorStats`。 关键方法包括 `__post_init__`, `reset`, `aggregate`, `reduce`, `is_empty`。

### Class `OffloadPromMetrics` / 类 `OffloadPromMetrics`
```python
class OffloadPromMetrics(KVConnectorPromMetrics):
    def __init__(
        self,
        vllm_config: VllmConfig,
        metric_types: dict[type[PromMetric], type[PromMetricT]],
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        super().__init__(vllm_config, metric_types, labelnames, per_engine_labelvalues)
        # (engine_idx, transfer_type) -> (metric with bounded labels)
        self.histogram_transfer_size: dict[tuple[int, str], PromMetricT] = {}
        self.counter_kv_bytes: dict[tuple[int, str], PromMetricT] = {}
        self.counter_kv_transfer_time: dict[tuple[int, str], PromMetricT] = {}
        buckets = [  # In bytes
            1e6,
            5e6,
            10e6,
            20e6,
            40e6,
            60e6,
            80e6,
            100e6,
            150e6,
            200e6,
        ]

        self._counter_kv_bytes = self._counter_cls(
            name="vllm:kv_offload_total_bytes",
            documentation="Number of bytes offloaded by KV connector",
            labelnames=labelnames + ["transfer_type"],
        )

        self._counter_kv_transfer_time = self._counter_cls(
            name="vllm:kv_offload_total_time",
            documentation="Total time measured by all KV offloading operations",
            labelnames=labelnames + ["transfer_type"],
        )

        self._histogram_transfer_size = self._histogram_cls(
            name="vllm:kv_offload_size",
            documentation="Histogram of KV offload transfer size, in bytes.",
            buckets=buckets[:],
            labelnames=labelnames + ["transfer_type"],
        )
# ... truncated for analysis ...
                self.histogram_transfer_size[(engine_idx, transfer_type)].observe(
                    op["op_size"]
                )

                # Increment byte and time counters
                self.counter_kv_bytes[(engine_idx, transfer_type)].inc(op["op_size"])

                self.counter_kv_transfer_time[(engine_idx, transfer_type)].inc(
                    op["op_time"]
                )
```
**EN:** Declares `OffloadPromMetrics`, a class derived from `KVConnectorPromMetrics`. Key methods include `__init__`, `observe`.
**CN:** 声明 `OffloadPromMetrics`，它是一个类，继承自 `KVConnectorPromMetrics`。 关键方法包括 `__init__`, `observe`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `OffloadingOperationMetrics`: dataclass interface or data carrier / `OffloadingOperationMetrics`：数据类接口或数据载体
- `OffloadingConnectorStats`: dataclass interface or data carrier / `OffloadingConnectorStats`：数据类接口或数据载体
- `OffloadPromMetrics`: class interface or data carrier / `OffloadPromMetrics`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.logger`, `vllm.v1.kv_offload.worker.worker`
