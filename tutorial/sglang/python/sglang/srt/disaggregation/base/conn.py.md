# conn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/base/conn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the base connection backend for disaggregated serving. It manages handshakes, transfer state, and KV movement for that specific transport or runtime environment. / 该文件实现了解耦式服务中的 base 连接后端，负责该特定传输/运行时环境下的握手、传输状态管理以及 KV 数据移动。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup
```python
from __future__ import annotations

import dataclasses
import enum
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, List, Optional

import numpy as np
import numpy.typing as npt

from sglang.srt.server_args import ServerArgs

if TYPE_CHECKING:
    from sglang.srt.disaggregation.utils import DisaggregationMode


```
**EN:** This block gathers the imports and module-level setup for base backend connection and KV transfer management. The imported modules show which runtime services, schemas, or backend components this file depends on.
**CN:** 这一段汇集了与base 后端连接与 KV 传输管理相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。

### Lines 17-17: Class `StateType` declaration
```python
class StateType(str, enum.Enum):
```
**EN:** This block declares the class `StateType` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StateType`.
**CN:** 这一段声明了类 `StateType`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StateType`。

### Lines 18-20: Supporting state inside `StateType`
```python
    MAMBA = "mamba"
    SWA = "swa"
    NSA = "nsa"
```
**EN:** This block adds supporting state or helper logic inside `StateType`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `StateType` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 23-25: Class `KVTransferMetric` declaration
```python
@dataclasses.dataclass
class KVTransferMetric:
    # Backends that cannot isolate transfer latency can leave this as None.
```
**EN:** This block declares the class `KVTransferMetric` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVTransferMetric`.
**CN:** 这一段声明了类 `KVTransferMetric`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVTransferMetric`。

### Lines 26-27: Supporting state inside `KVTransferMetric`
```python
    transfer_latency_s: Optional[float] = None
    transfer_total_bytes: Optional[int] = None
```
**EN:** This block adds supporting state or helper logic inside `KVTransferMetric`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVTransferMetric` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 30-30: Class `KVArgs` declaration
```python
class KVArgs:
```
**EN:** This block declares the class `KVArgs` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVArgs`.
**CN:** 这一段声明了类 `KVArgs`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVArgs`。

### Lines 31-54: Supporting state inside `KVArgs`
```python
    engine_rank: int
    kv_data_ptrs: List[int]
    kv_data_lens: List[int]
    kv_item_lens: List[int]
    aux_data_ptrs: List[int]
    aux_data_lens: List[int]
    aux_item_lens: List[int]
    state_types: List[StateType]
    state_data_ptrs: List[List[int]]
    state_data_lens: List[List[int]]
    state_item_lens: List[List[int]]
    # Per-tensor TP slice dim, used when prefill/decode attn_tp_size differ.
    state_dim_per_tensor: List[List[int]]
    ib_device: str
    ib_traffic_class: str
    gpu_id: int
    kv_head_num: int
    total_kv_head_num: int
    page_size: int
    # for system dp
    system_dp_rank: int
    # for pp prefill
    pp_rank: int
    prefill_start_layer: int
```
**EN:** This block adds supporting state or helper logic inside `KVArgs`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVArgs` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 55-68: Supporting state inside `KVArgs`
```python
    # Absolute end layer (exclusive) for this prefill PP stage. Needed to
    # reconstruct PP sub-ranges when kv_data_ptrs does not use a flat
    # layer-indexed layout (e.g. DeepSeek V4's buffer-type-organized flat
    # list).
    prefill_end_layer: Optional[int]
    # For DeepSeek V4 (and other compressed-MLA) memory pools only.
    # Full-model compression ratio per layer (entries are 0/4/128). Used by
    # the connection layer to slice the buffer-type-organized flat list in a
    # PP-aware manner.
    mla_compression_ratios: Optional[List[int]]
    # Only used of npu, for kv buf groups
    kv_buf_groups: int
    # Only used of npu, for decode total kv layers
    total_kv_layers: int
```
**EN:** This block adds supporting state or helper logic inside `KVArgs`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `layer`, `layout`, `V4`.
**CN:** 这一段为 `KVArgs` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `layer`、`layout`、`V4`。

### Lines 71-71: Class `KVPoll` declaration
```python
class KVPoll:
```
**EN:** This block declares the class `KVPoll` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVPoll`.
**CN:** 这一段声明了类 `KVPoll`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVPoll`。

### Lines 72-76: Supporting state inside `KVPoll`
```python
    Failed = 0
    Bootstrapping = 1
    WaitingForInput = 2
    Transferring = 3
    Success = 4
```
**EN:** This block adds supporting state or helper logic inside `KVPoll`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVPoll` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 79-79: Class `BaseKVManager` declaration
```python
class BaseKVManager(ABC):
```
**EN:** This block declares the class `BaseKVManager` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BaseKVManager`.
**CN:** 这一段声明了类 `BaseKVManager`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BaseKVManager`。

### Lines 80-81: Supporting state inside `BaseKVManager`
```python
    """Base class for managing transfer states"""

```
**EN:** This block adds supporting state or helper logic inside `BaseKVManager`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `BaseKVManager` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 82-89: Method `__init__`
```python
    @abstractmethod
    def __init__(
        self,
        args: KVArgs,
        disaggregation_mode: DisaggregationMode,
        server_args: ServerArgs,
        is_mla_backend: Optional[bool] = False,
    ): ...
```
**EN:** This block defines the method `__init__` on `BaseKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `BaseKVManager`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 91-94: Method `register_to_bootstrap`
```python
    @abstractmethod
    def register_to_bootstrap(self):
        """Register prefill server info to the bootstrap server."""
        ...
```
**EN:** This block defines the method `register_to_bootstrap` on `BaseKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `register_to_bootstrap`.
**CN:** 这一段定义了method `register_to_bootstrap`（属于 `BaseKVManager`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `register_to_bootstrap`。

### Lines 97-98: Class `BaseKVSender` declaration
```python
class BaseKVSender(ABC):

```
**EN:** This block declares the class `BaseKVSender` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BaseKVSender`.
**CN:** 这一段声明了类 `BaseKVSender`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BaseKVSender`。

### Lines 99-107: Method `__init__`
```python
    @abstractmethod
    def __init__(
        self,
        mgr: BaseKVManager,
        bootstrap_addr: str,
        bootstrap_room: int,
        dest_tp_ranks: List[int],
        pp_rank: int,
    ): ...
```
**EN:** This block defines the method `__init__` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 109-114: Method `init`
```python
    @abstractmethod
    def init(self, num_kv_indices: int, aux_index: Optional[int] = None):
        """
        Set req's index metadata locally or notify the decoder server about the kv indices length and aux index.
        """
        ...
```
**EN:** This block defines the method `init` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `init`.
**CN:** 这一段定义了method `init`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。

### Lines 116-125: Method `send`
```python
    @abstractmethod
    def send(
        self,
        kv_indices: npt.NDArray[np.int32],
        state_indices: Optional[List] = None,
    ):
        """
        Send the kv cache at the given kv indices and the extra cache/state at the given indices to the decoder server.
        """
        ...
```
**EN:** This block defines the method `send` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `send`.
**CN:** 这一段定义了method `send`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send`。

### Lines 127-128: Method `pop_decode_prefix_len`
```python
    def pop_decode_prefix_len(self) -> int:
        return 0
```
**EN:** This block defines the method `pop_decode_prefix_len` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `pop_decode_prefix_len`.
**CN:** 这一段定义了method `pop_decode_prefix_len`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `pop_decode_prefix_len`。

### Lines 130-131: Method `should_send_kv_chunk`
```python
    def should_send_kv_chunk(self, num_pages: int, last_chunk: bool) -> bool:
        return num_pages > 0
```
**EN:** This block defines the method `should_send_kv_chunk` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `should_send_kv_chunk`.
**CN:** 这一段定义了method `should_send_kv_chunk`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `should_send_kv_chunk`。

### Lines 133-136: Method `get_transfer_metric`
```python
    @abstractmethod
    def get_transfer_metric(self) -> KVTransferMetric:
        """Return backend-specific transfer metrics for this sender."""
        ...
```
**EN:** This block defines the method `get_transfer_metric` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `get_transfer_metric`.
**CN:** 这一段定义了method `get_transfer_metric`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_transfer_metric`。

### Lines 138-143: Method `poll`
```python
    @abstractmethod
    def poll(self) -> KVPoll:
        """
        Check the status of the kv cache transfer.
        """
        ...
```
**EN:** This block defines the method `poll` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `poll`.
**CN:** 这一段定义了method `poll`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。

### Lines 145-150: Method `failure_exception`
```python
    @abstractmethod
    def failure_exception(self):
        """
        Raise an exception if the kv cache transfer fails.
        """
        ...
```
**EN:** This block defines the method `failure_exception` on `BaseKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `failure_exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `BaseKVSender`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。

### Lines 153-154: Class `BaseKVReceiver` declaration
```python
class BaseKVReceiver(ABC):

```
**EN:** This block declares the class `BaseKVReceiver` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BaseKVReceiver`.
**CN:** 这一段声明了类 `BaseKVReceiver`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BaseKVReceiver`。

### Lines 155-161: Method `__init__`
```python
    @abstractmethod
    def __init__(
        self,
        mgr: BaseKVManager,
        bootstrap_addr: str,
        bootstrap_room: Optional[int] = None,
    ): ...
```
**EN:** This block defines the method `__init__` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 163-171: Method `init`
```python
    @abstractmethod
    def init(
        self,
        prefill_dp_rank: int,
    ):
        """
        Resolve bootstrap metadata and mark the receiver ready for transfer metadata.
        """
        ...
```
**EN:** This block defines the method `init` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `init`.
**CN:** 这一段定义了method `init`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。

### Lines 173-184: Method `send_metadata`
```python
    @abstractmethod
    def send_metadata(
        self,
        kv_indices: npt.NDArray[np.int32],
        aux_index: Optional[int] = None,
        state_indices: Optional[List] = None,
        decode_prefix_len: Optional[int] = None,
    ):
        """
        Notify the prefill server about the kv indices, aux index, and state_indices.
        """
        ...
```
**EN:** This block defines the method `send_metadata` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `send_metadata`.
**CN:** 这一段定义了method `send_metadata`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send_metadata`。

### Lines 186-191: Method `poll`
```python
    @abstractmethod
    def poll(self) -> KVPoll:
        """
        Check the status of the kv cache transfer.
        """
        ...
```
**EN:** This block defines the method `poll` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `poll`.
**CN:** 这一段定义了method `poll`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。

### Lines 193-198: Method `failure_exception`
```python
    @abstractmethod
    def failure_exception(self):
        """
        Raise an exception if the kv cache transfer fails.
        """
        ...
```
**EN:** This block defines the method `failure_exception` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `failure_exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。

### Lines 200-204: Method `clear`
```python
    def clear(self):
        """
        Clear any internal states.
        """
        pass
```
**EN:** This block defines the method `clear` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `clear`.
**CN:** 这一段定义了method `clear`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `clear`。

### Lines 206-210: Method `abort`
```python
    def abort(self):
        """
        Abort the current transfer.
        """
        pass
```
**EN:** This block defines the method `abort` on `BaseKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `abort`.
**CN:** 这一段定义了method `abort`（属于 `BaseKVReceiver`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `abort`。

### Lines 213-213: Class `BaseKVBootstrapServer` declaration
```python
class BaseKVBootstrapServer(ABC):
```
**EN:** This block declares the class `BaseKVBootstrapServer` and establishes its responsibility inside base backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BaseKVBootstrapServer`.
**CN:** 这一段声明了类 `BaseKVBootstrapServer`，并说明它在base 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BaseKVBootstrapServer`。

### Lines 214-215: Method `__init__`
```python
    @abstractmethod
    def __init__(self, host: str, port: int): ...
```
**EN:** This block defines the method `__init__` on `BaseKVBootstrapServer`. It introduces the parameters, setup steps, and the main entry point for this piece of base backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `BaseKVBootstrapServer`），介绍了参数、初始化步骤，以及这部分base 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

## Key Concepts / 关键概念
- `StateType`: Class that encapsulates state type behavior in this module. / `StateType`：封装与“状态类型”相关行为的类。
- `KVTransferMetric`: Class that encapsulates kvtransfer metric behavior in this module. / `KVTransferMetric`：封装与“kvtransfermetric”相关行为的类。
- `KVArgs`: Class that encapsulates kvargs behavior in this module. / `KVArgs`：封装与“kvargs”相关行为的类。
- `KVPoll`: Class that encapsulates kvpoll behavior in this module. / `KVPoll`：封装与“kvpoll”相关行为的类。
- `BaseKVManager`: Class that encapsulates base kvmanager behavior in this module. / `BaseKVManager`：封装与“基础kvmanager”相关行为的类。
- `BaseKVSender`: Class that encapsulates base kvsender behavior in this module. / `BaseKVSender`：封装与“基础kvsender”相关行为的类。
- `BaseKVReceiver`: Class that encapsulates base kvreceiver behavior in this module. / `BaseKVReceiver`：封装与“基础kvreceiver”相关行为的类。
- `BaseKVBootstrapServer`: Class that encapsulates base kvbootstrap server behavior in this module. / `BaseKVBootstrapServer`：封装与“基础kvbootstrap服务端”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `enum`, `abc`, `typing`
- **External packages / 外部依赖**: `numpy`
- **Internal modules / 内部模块**: `sglang.srt.server_args`, `sglang.srt.disaggregation.utils`
