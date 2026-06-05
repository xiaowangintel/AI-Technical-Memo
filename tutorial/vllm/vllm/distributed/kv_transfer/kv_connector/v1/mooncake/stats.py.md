# stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Stats container for the Mooncake connector / 实现基于 Mooncake 的 KV 传输连接器、存储层或运行时辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Stats container for the Mooncake connector."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Stats container for the Mooncake connector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import threading
from dataclasses import dataclass
from typing import Any

import numpy as np

from vllm.distributed.kv_transfer.kv_connector.v1.metrics import (
    KVConnectorStats,
)
```
**EN:** This block imports `threading`, `dataclasses`, `typing`, `numpy`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `threading`, `dataclasses`, `typing`, `numpy`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`，为后续实现准备运行时、类型与辅助 API。

### Class `MooncakeKVConnectorStats` / 类 `MooncakeKVConnectorStats`
```python
@dataclass
class MooncakeKVConnectorStats(KVConnectorStats):
    """Container for Mooncake KV transfer performance metrics.

    `_lock` serializes record_* against clone_and_reset so each row's
    appends are atomic and column lengths stay aligned. Writers run on
    the sender pool / receiver loop / sender loop; reader runs on the
    main worker thread.
    """

    def __post_init__(self):
        self._lock = threading.Lock()
        if not self.data:
            self.reset()

    # threading.Lock is not picklable; strip it from the wire form and
    # rebuild a fresh per-process lock on the receiver side.
    def __getstate__(self) -> dict[str, Any]:
        state = self.__dict__.copy()
        state.pop("_lock", None)
        return state

    def __setstate__(self, state: dict[str, Any]) -> None:
        self.__dict__.update(state)
        self._lock = threading.Lock()

    def reset(self):
        self.data: dict[str, list[float | int]] = {
            "transfer_duration": [],
            "bytes_transferred": [],
            "num_descriptors": [],
            "num_failed_transfers": [],
            "num_failed_recvs": [],
            "num_kv_expired_reqs": [],
        }

    def record_transfer(self, duration_s: float, total_bytes: int, num_descs: int):
        with self._lock:
            self.data["transfer_duration"].append(duration_s)
            self.data["bytes_transferred"].append(total_bytes)
            self.data["num_descriptors"].append(num_descs)

    # Failure counters store a list of 1s so a future Prom counter can iterate
    # with .inc(list_item), mirroring NIXL's NixlPromMetrics.observe.
    def record_failed_transfer(self):
# ... truncated for analysis ...
            "Throughput (MB/s)": round(throughput_mb_s, 3),
            "Avg number of descriptors": round(descs.mean(), 1),
            "Num failed transfers": num_failed_transfers,
            "Num failed recvs": num_failed_recvs,
            "Num KV expired reqs": num_kv_expired_reqs,
        }

    @property
    def num_successful_transfers(self) -> int:
        return len(self.data["transfer_duration"])
```
**EN:** Declares `MooncakeKVConnectorStats`, a dataclass derived from `KVConnectorStats`. Key methods include `__post_init__`, `__getstate__`, `__setstate__`, `reset`, `record_transfer`. The docstring summarizes its role as: Container for Mooncake KV transfer performance metrics.
**CN:** 声明 `MooncakeKVConnectorStats`，它是一个数据类，继承自 `KVConnectorStats`。 关键方法包括 `__post_init__`, `__getstate__`, `__setstate__`, `reset`, `record_transfer`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `MooncakeKVConnectorStats`: dataclass interface or data carrier / `MooncakeKVConnectorStats`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `dataclasses`, `typing`
- **Third-party / 第三方**: `numpy`
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.v1.metrics`
