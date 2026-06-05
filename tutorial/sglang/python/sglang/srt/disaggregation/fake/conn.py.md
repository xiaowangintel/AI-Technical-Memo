# conn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/fake/conn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the fake connection backend for disaggregated serving. It manages handshakes, transfer state, and KV movement for that specific transport or runtime environment. / 该文件实现了解耦式服务中的 fake 连接后端，负责该特定传输/运行时环境下的握手、传输状态管理以及 KV 数据移动。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Module-level constants and helper logic
```python
import logging
from typing import List, Optional

import numpy as np
import numpy.typing as npt

from sglang.srt.disaggregation.base.conn import (
    BaseKVManager,
    BaseKVReceiver,
    BaseKVSender,
    KVArgs,
    KVPoll,
    KVTransferMetric,
)
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)


# For warmup reqs, we don't kv transfer, we use the fake manager, sender and receiver
```
**EN:** This block contains module-level constants, helpers, or documentation for fake backend connection and KV transfer management. It prepares shared state that later classes and functions build on. Notable operations include `import`, `getLogger`.
**CN:** 这一段包含与fake 后端连接与 KV 传输管理相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `import`、`getLogger`。

### Lines 22-22: Class `FakeKVManager` declaration
```python
class FakeKVManager(BaseKVManager):
```
**EN:** This block declares the class `FakeKVManager` and establishes its responsibility inside fake backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `FakeKVManager`.
**CN:** 这一段声明了类 `FakeKVManager`，并说明它在fake 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `FakeKVManager`。

### Lines 23-32: Method `__init__`
```python
    def __init__(
        self,
        args: KVArgs,
        disaggregation_mode: DisaggregationMode,
        server_args: ServerArgs,
        is_mla_backend: Optional[bool] = False,
    ):
        super().__init__(args, disaggregation_mode, server_args, is_mla_backend)
        self.kv_args = args
        self.req_to_decode_prefix_len = {}
```
**EN:** This block defines the method `__init__` on `FakeKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `FakeKVManager`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 34-35: Method `register_to_bootstrap`
```python
    def register_to_bootstrap(self):
        pass
```
**EN:** This block defines the method `register_to_bootstrap` on `FakeKVManager`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `register_to_bootstrap`.
**CN:** 这一段定义了method `register_to_bootstrap`（属于 `FakeKVManager`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `register_to_bootstrap`。

### Lines 38-38: Class `FakeKVSender` declaration
```python
class FakeKVSender(BaseKVSender):
```
**EN:** This block declares the class `FakeKVSender` and establishes its responsibility inside fake backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `FakeKVSender`.
**CN:** 这一段声明了类 `FakeKVSender`，并说明它在fake 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `FakeKVSender`。

### Lines 39-49: Method `__init__`
```python
    def __init__(
        self,
        mgr: BaseKVManager,
        bootstrap_addr: str,
        bootstrap_room: int,
        dest_tp_ranks: List[int],
        pp_rank: int,
    ):
        self.kv_mgr = mgr
        self.has_sent = False
        self.conclude_state: Optional[KVPoll] = None
```
**EN:** This block defines the method `__init__` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 51-61: Method `poll`
```python
    def poll(self) -> KVPoll:
        if self.conclude_state is not None:
            return self.conclude_state
        if not self.has_sent:
            # Assume handshake completed instantly
            return KVPoll.WaitingForInput

        # Assume transfer completed instantly
        logger.debug("FakeKVSender poll success")
        self.conclude_state = KVPoll.Success
        return KVPoll.Success
```
**EN:** This block defines the method `poll` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `poll`. Notable operations include `debug`.
**CN:** 这一段定义了method `poll`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。 值得注意的操作包括 `debug`。

### Lines 63-64: Method `get_transfer_metric`
```python
    def get_transfer_metric(self) -> KVTransferMetric:
        return KVTransferMetric()
```
**EN:** This block defines the method `get_transfer_metric` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `get_transfer_metric`. Notable operations include `KVTransferMetric`.
**CN:** 这一段定义了method `get_transfer_metric`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `get_transfer_metric`。 值得注意的操作包括 `KVTransferMetric`。

### Lines 66-74: Method `init`
```python
    def init(
        self,
        kv_indices: list[int],
        aux_index: Optional[int] = None,
    ):
        logger.debug(
            f"FakeKVSender init with kv_indices: {kv_indices}, aux_index: {aux_index}"
        )
        pass
```
**EN:** This block defines the method `init` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `init`. Notable operations include `debug`.
**CN:** 这一段定义了method `init`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。 值得注意的操作包括 `debug`。

### Lines 76-84: Method `send`
```python
    def send(
        self,
        kv_indices: npt.NDArray[np.int32],
        state_indices: Optional[List] = None,
    ):
        self.has_sent = True
        logger.debug(
            f"FakeKVSender send with kv_indices: {kv_indices}, state_indices: {state_indices}"
        )
```
**EN:** This block defines the method `send` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `send`. Notable operations include `debug`.
**CN:** 这一段定义了method `send`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send`。 值得注意的操作包括 `debug`。

### Lines 86-87: Method `failure_exception`
```python
    def failure_exception(self):
        raise Exception("Fake KVSender Exception")
```
**EN:** This block defines the method `failure_exception` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `failure_exception`. Notable operations include `Exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。 值得注意的操作包括 `Exception`。

### Lines 89-90: Method `abort`
```python
    def abort(self):
        self.conclude_state = KVPoll.Failed
```
**EN:** This block defines the method `abort` on `FakeKVSender`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `abort`.
**CN:** 这一段定义了method `abort`（属于 `FakeKVSender`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `abort`。

### Lines 93-93: Class `FakeKVReceiver` declaration
```python
class FakeKVReceiver(BaseKVReceiver):
```
**EN:** This block declares the class `FakeKVReceiver` and establishes its responsibility inside fake backend connection and KV transfer management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `FakeKVReceiver`.
**CN:** 这一段声明了类 `FakeKVReceiver`，并说明它在fake 后端连接与 KV 传输管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `FakeKVReceiver`。

### Lines 94-103: Method `__init__`
```python
    def __init__(
        self,
        mgr: BaseKVManager,
        bootstrap_addr: str,
        bootstrap_room: Optional[int] = None,
    ):
        self.bootstrap_done = False
        self.has_sent_metadata = False
        self.require_staging: bool = False
        self.conclude_state: Optional[KVPoll] = None
```
**EN:** This block defines the method `__init__` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 105-114: Method `poll`
```python
    def poll(self) -> KVPoll:
        if self.conclude_state is not None:
            return self.conclude_state
        if not self.bootstrap_done:
            return KVPoll.Bootstrapping
        if not self.has_sent_metadata:
            return KVPoll.WaitingForInput
        logger.debug("FakeKVReceiver poll success")
        self.conclude_state = KVPoll.Success
        return KVPoll.Success
```
**EN:** This block defines the method `poll` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `poll`. Notable operations include `debug`.
**CN:** 这一段定义了method `poll`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `poll`。 值得注意的操作包括 `debug`。

### Lines 116-120: Method `init`
```python
    def init(
        self,
        prefill_dp_rank: int,
    ):
        self.bootstrap_done = True
```
**EN:** This block defines the method `init` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `init`.
**CN:** 这一段定义了method `init`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `init`。

### Lines 122-132: Method `send_metadata`
```python
    def send_metadata(
        self,
        kv_indices: list[int],
        aux_index: Optional[int] = None,
        state_indices: Optional[List] = None,
        decode_prefix_len: Optional[int] = None,
    ):
        self.has_sent_metadata = True
        logger.debug(
            f"FakeKVReceiver send_metadata with kv_indices: {kv_indices}, aux_index: {aux_index}, state_indices: {state_indices}"
        )
```
**EN:** This block defines the method `send_metadata` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `send_metadata`. Notable operations include `debug`.
**CN:** 这一段定义了method `send_metadata`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `send_metadata`。 值得注意的操作包括 `debug`。

### Lines 134-135: Method `failure_exception`
```python
    def failure_exception(self):
        raise Exception("Fake KVReceiver Exception")
```
**EN:** This block defines the method `failure_exception` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `failure_exception`. Notable operations include `Exception`.
**CN:** 这一段定义了method `failure_exception`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `failure_exception`。 值得注意的操作包括 `Exception`。

### Lines 137-138: Method `abort`
```python
    def abort(self):
        self.conclude_state = KVPoll.Failed
```
**EN:** This block defines the method `abort` on `FakeKVReceiver`. It introduces the parameters, setup steps, and the main entry point for this piece of fake backend connection and KV transfer management. Definitions introduced here include `abort`.
**CN:** 这一段定义了method `abort`（属于 `FakeKVReceiver`），介绍了参数、初始化步骤，以及这部分fake 后端连接与 KV 传输管理逻辑的主要入口。 此处引入的定义包括 `abort`。

## Key Concepts / 关键概念
- `FakeKVManager`: Class that encapsulates fake kvmanager behavior in this module. / `FakeKVManager`：封装与“模拟kvmanager”相关行为的类。
- `FakeKVSender`: Class that encapsulates fake kvsender behavior in this module. / `FakeKVSender`：封装与“模拟kvsender”相关行为的类。
- `FakeKVReceiver`: Class that encapsulates fake kvreceiver behavior in this module. / `FakeKVReceiver`：封装与“模拟kvreceiver”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **External packages / 外部依赖**: `numpy`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.base.conn`, `sglang.srt.disaggregation.utils`, `sglang.srt.server_args`
