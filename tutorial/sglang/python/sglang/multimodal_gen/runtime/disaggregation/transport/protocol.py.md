# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `TransferMsgType`, `TransferStagedMsg`, and `TransferAllocMsg`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Transfer protocol messages for disaggregated diffusion. All messages are sent as ZMQ multipart with a b"__transfer__" discriminator in frame[0] and JSON payload in frame[1]. / 该文件属于解耦运行时层。它围绕 `TransferMsgType`、`TransferStagedMsg` 和 `TransferAllocMsg` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-11: module setup and imports / 模块初始化与导入
```python
"""Transfer protocol messages for disaggregated diffusion.

All messages are sent as ZMQ multipart with a b"__transfer__" discriminator
in frame[0] and JSON payload in frame[1].
"""

import json
import logging
from dataclasses import asdict, dataclass, field
from typing import Any
```
**EN:** This block establishes the module context and imports `json`, `logging`, `dataclasses`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`logging`、`dataclasses` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 13-15: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

TRANSFER_MAGIC = b"__transfer__"
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `TRANSFER_MAGIC`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `TRANSFER_MAGIC` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 18-19: `TransferMsgType` class overview / `TransferMsgType` 类概览
```python
class TransferMsgType:
    # Instance → DiffusionServer
```
**EN:** This block defines class `TransferMsgType`. It encapsulates transfer msg type behavior.
**CN:** 该代码块定义了类 `TransferMsgType`。 它用于封装 transfer msg type 相关行为。

### Lines 20-32: supporting statements / 辅助语句
```python
    STAGED = "transfer_staged"
    ALLOCATED = "transfer_allocated"
    PUSHED = "transfer_pushed"
    DONE = "transfer_done"

    # DiffusionServer → Instance
    ALLOC = "transfer_alloc"
    PUSH = "transfer_push"
    READY = "transfer_ready"

    # Registration
    REGISTER = "transfer_register"
    REGISTER_ACK = "transfer_register_ack"
```
**EN:** This block gathers supporting statements inside `TransferMsgType`. It updates names such as `STAGED`, `ALLOCATED`, `PUSHED`, `DONE`, `ALLOC`, and `PUSH`.
**CN:** 该代码块汇集了位于 `TransferMsgType` 内部的辅助语句。 它会更新 `STAGED`、`ALLOCATED`、`PUSHED`、`DONE`、`ALLOC` 和 `PUSH` 等名称。

### Lines 36-36: `TransferStagedMsg` class overview / `TransferStagedMsg` 类概览
```python
class TransferStagedMsg:
```
**EN:** This block defines class `TransferStagedMsg`. It encapsulates transfer staged msg behavior.
**CN:** 该代码块定义了类 `TransferStagedMsg`。 它用于封装 transfer staged msg 相关行为。

### Lines 37-43: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.STAGED
    request_id: str = ""
    data_size: int = 0
    manifest: dict = None
    session_id: str = ""
    pool_ptr: int = 0
    slot_offset: int = 0
```
**EN:** This block gathers supporting statements inside `TransferStagedMsg`. It updates names such as `msg_type`, `request_id`, `data_size`, `manifest`, `session_id`, and `pool_ptr`.
**CN:** 该代码块汇集了位于 `TransferStagedMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id`、`data_size`、`manifest`、`session_id` 和 `pool_ptr` 等名称。

### Lines 45-47: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        if self.manifest is None:
            self.manifest = {}
```
**EN:** This block defines method `__post_init__` on `TransferStagedMsg`. It post-processes init. The implementation branches on conditions.
**CN:** 该代码块定义了 `TransferStagedMsg` 的方法 `__post_init__`。 它用于后处理init。 实现中包含条件分支。

### Lines 51-51: `TransferAllocMsg` class overview / `TransferAllocMsg` 类概览
```python
class TransferAllocMsg:
```
**EN:** This block defines class `TransferAllocMsg`. It encapsulates transfer alloc msg behavior.
**CN:** 该代码块定义了类 `TransferAllocMsg`。 它用于封装 transfer alloc msg 相关行为。

### Lines 52-55: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.ALLOC
    request_id: str = ""
    data_size: int = 0
    source_role: str = ""
```
**EN:** This block gathers supporting statements inside `TransferAllocMsg`. It updates names such as `msg_type`, `request_id`, `data_size`, and `source_role`.
**CN:** 该代码块汇集了位于 `TransferAllocMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id`、`data_size` 和 `source_role` 等名称。

### Lines 59-59: `TransferAllocatedMsg` class overview / `TransferAllocatedMsg` 类概览
```python
class TransferAllocatedMsg:
```
**EN:** This block defines class `TransferAllocatedMsg`. It encapsulates transfer allocated msg behavior.
**CN:** 该代码块定义了类 `TransferAllocatedMsg`。 它用于封装 transfer allocated msg 相关行为。

### Lines 60-65: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.ALLOCATED
    request_id: str = ""
    session_id: str = ""
    pool_ptr: int = 0
    slot_offset: int = 0
    slot_size: int = 0
```
**EN:** This block gathers supporting statements inside `TransferAllocatedMsg`. It updates names such as `msg_type`, `request_id`, `session_id`, `pool_ptr`, `slot_offset`, and `slot_size`.
**CN:** 该代码块汇集了位于 `TransferAllocatedMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id`、`session_id`、`pool_ptr`、`slot_offset` 和 `slot_size` 等名称。

### Lines 69-69: `TransferPushMsg` class overview / `TransferPushMsg` 类概览
```python
class TransferPushMsg:
```
**EN:** This block defines class `TransferPushMsg`. It encapsulates transfer push msg behavior.
**CN:** 该代码块定义了类 `TransferPushMsg`。 它用于封装 transfer push msg 相关行为。

### Lines 70-74: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.PUSH
    request_id: str = ""
    dest_session_id: str = ""
    dest_addr: int = 0
    transfer_size: int = 0
```
**EN:** This block gathers supporting statements inside `TransferPushMsg`. It updates names such as `msg_type`, `request_id`, `dest_session_id`, `dest_addr`, and `transfer_size`.
**CN:** 该代码块汇集了位于 `TransferPushMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id`、`dest_session_id`、`dest_addr` 和 `transfer_size` 等名称。

### Lines 78-78: `TransferPushedMsg` class overview / `TransferPushedMsg` 类概览
```python
class TransferPushedMsg:
```
**EN:** This block defines class `TransferPushedMsg`. It encapsulates transfer pushed msg behavior.
**CN:** 该代码块定义了类 `TransferPushedMsg`。 它用于封装 transfer pushed msg 相关行为。

### Lines 79-80: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.PUSHED
    request_id: str = ""
```
**EN:** This block gathers supporting statements inside `TransferPushedMsg`. It updates names such as `msg_type`, and `request_id`.
**CN:** 该代码块汇集了位于 `TransferPushedMsg` 内部的辅助语句。 它会更新 `msg_type` 和 `request_id` 等名称。

### Lines 84-84: `TransferReadyMsg` class overview / `TransferReadyMsg` 类概览
```python
class TransferReadyMsg:
```
**EN:** This block defines class `TransferReadyMsg`. It encapsulates transfer ready msg behavior.
**CN:** 该代码块定义了类 `TransferReadyMsg`。 它用于封装 transfer ready msg 相关行为。

### Lines 85-89: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.READY
    request_id: str = ""
    manifest: dict = None
    slot_offset: int = 0
    scalar_fields: dict = None
```
**EN:** This block gathers supporting statements inside `TransferReadyMsg`. It updates names such as `msg_type`, `request_id`, `manifest`, `slot_offset`, and `scalar_fields`.
**CN:** 该代码块汇集了位于 `TransferReadyMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id`、`manifest`、`slot_offset` 和 `scalar_fields` 等名称。

### Lines 91-95: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        if self.manifest is None:
            self.manifest = {}
        if self.scalar_fields is None:
            self.scalar_fields = {}
```
**EN:** This block defines method `__post_init__` on `TransferReadyMsg`. It post-processes init. The implementation branches on conditions.
**CN:** 该代码块定义了 `TransferReadyMsg` 的方法 `__post_init__`。 它用于后处理init。 实现中包含条件分支。

### Lines 99-99: `TransferDoneMsg` class overview / `TransferDoneMsg` 类概览
```python
class TransferDoneMsg:
```
**EN:** This block defines class `TransferDoneMsg`. It encapsulates transfer done msg behavior.
**CN:** 该代码块定义了类 `TransferDoneMsg`。 它用于封装 transfer done msg 相关行为。

### Lines 100-102: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.DONE
    request_id: str = ""
    error: str | None = None
```
**EN:** This block gathers supporting statements inside `TransferDoneMsg`. It updates names such as `msg_type`, `request_id`, and `error`.
**CN:** 该代码块汇集了位于 `TransferDoneMsg` 内部的辅助语句。 它会更新 `msg_type`、`request_id` 和 `error` 等名称。

### Lines 106-106: `TransferRegisterMsg` class overview / `TransferRegisterMsg` 类概览
```python
class TransferRegisterMsg:
```
**EN:** This block defines class `TransferRegisterMsg`. It encapsulates transfer register msg behavior.
**CN:** 该代码块定义了类 `TransferRegisterMsg`。 它用于封装 transfer register msg 相关行为。

### Lines 107-119: supporting statements / 辅助语句
```python
    msg_type: str = TransferMsgType.REGISTER
    role: str = ""
    session_id: str = ""
    pool_ptr: int = 0
    pool_size: int = 0
    # The instance's own work endpoint (e.g. tcp://host:port). Used by the
    # DiffusionServer to key peer info by URL index (i.e. the same index used
    # to build the PUSH work-socket list), so the control plane and the RDMA
    # data plane cannot drift when instances register in a different order
    # than --*-urls.
    work_endpoint: str = ""
    # Pre-allocated receive slots: [{"offset": int, "size": int, "slot_id": int, "addr": int}]
    preallocated_slots: list = field(default_factory=list)
```
**EN:** This block gathers supporting statements inside `TransferRegisterMsg`. It updates names such as `msg_type`, `role`, `session_id`, `pool_ptr`, `pool_size`, and `work_endpoint`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TransferRegisterMsg` 内部的辅助语句。 它会更新 `msg_type`、`role`、`session_id`、`pool_ptr`、`pool_size` 和 `work_endpoint` 等名称。 代码会与 `field` 协同工作。

### Lines 122-131: `encode_transfer_msg` implementation / `encode_transfer_msg` 实现
```python
def encode_transfer_msg(msg: Any) -> list[bytes]:
    """Encode as [TRANSFER_MAGIC, json_payload_bytes]."""
    if hasattr(msg, "__dataclass_fields__"):
        d = asdict(msg)
    elif isinstance(msg, dict):
        d = msg
    else:
        raise TypeError(f"Cannot encode transfer message: {type(msg)}")

    return [TRANSFER_MAGIC, json.dumps(d, separators=(",", ":")).encode("utf-8")]
```
**EN:** This block defines function `encode_transfer_msg`. Encode as [TRANSFER_MAGIC, json_payload_bytes]. Key calls include `hasattr`, `asdict`, `isinstance`, `json.dumps.encode`, and `TypeError`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了函数 `encode_transfer_msg`。 它用于编码transfer msg。 关键调用包括 `hasattr`、`asdict`、`isinstance`、`json.dumps.encode` 和 `TypeError`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 134-137: `decode_transfer_msg` implementation / `decode_transfer_msg` 实现
```python
def decode_transfer_msg(frames: list[bytes]) -> dict:
    if len(frames) < 2 or frames[0] != TRANSFER_MAGIC:
        raise ValueError(f"Not a transfer message: frame[0]={frames[0]!r}")
    return json.loads(frames[1])
```
**EN:** This block defines function `decode_transfer_msg`. It decodes transfer msg. Key calls include `json.loads`, `ValueError`, and `len`. The implementation branches on conditions. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了函数 `decode_transfer_msg`。 它用于解码transfer msg。 关键调用包括 `json.loads`、`ValueError` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 140-145: `is_transfer_message` implementation / `is_transfer_message` 实现
```python
def is_transfer_message(frames: list) -> bool:
    return len(frames) >= 2 and (
        frames[0] == TRANSFER_MAGIC
        or (isinstance(frames[0], memoryview) and bytes(frames[0]) == TRANSFER_MAGIC)
        or (hasattr(frames[0], "bytes") and frames[0].bytes == TRANSFER_MAGIC)
    )
```
**EN:** This block defines function `is_transfer_message`. It handles is transfer message logic. Key calls include `len`, `isinstance`, `hasattr`, and `bytes`. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了函数 `is_transfer_message`。 它用于处理 is transfer message 相关逻辑。 关键调用包括 `len`、`isinstance`、`hasattr` 和 `bytes`。 本段逻辑主要由 `frames` 等参数驱动。

## Key Concepts / 关键概念
- `TransferMsgType`: Primary class that encapsulates transfer msg type behavior. / 核心类，用于封装 transfer msg type 相关行为。
- `TransferStagedMsg`: Primary class that encapsulates transfer staged msg behavior. / 核心类，用于封装 transfer staged msg 相关行为。
- `TransferAllocMsg`: Primary class that encapsulates transfer alloc msg behavior. / 核心类，用于封装 transfer alloc msg 相关行为。
- `TransferAllocatedMsg`: Primary class that encapsulates transfer allocated msg behavior. / 核心类，用于封装 transfer allocated msg 相关行为。
- `TransferPushMsg`: Primary class that encapsulates transfer push msg behavior. / 核心类，用于封装 transfer push msg 相关行为。
- `TransferPushedMsg`: Primary class that encapsulates transfer pushed msg behavior. / 核心类，用于封装 transfer pushed msg 相关行为。
- `TransferReadyMsg`: Primary class that encapsulates transfer ready msg behavior. / 核心类，用于封装 transfer ready msg 相关行为。
- `TransferDoneMsg`: Primary class that encapsulates transfer done msg behavior. / 核心类，用于封装 transfer done msg 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `dataclasses`, `typing`

- **Total lines / 总行数**: 145
