# moriio_common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/moriio/moriio_common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements MoriIO KV-transfer adapters and supporting logic. / 实现 MoriIO KV 传输适配器及其配套逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import contextlib
import threading
import time
from collections.abc import Iterator
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any

import msgspec
import regex as re
import torch
import zmq

from vllm import envs
from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorMetadata,
)
from vllm.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.logger import init_logger
from vllm.utils.network_utils import (
    get_ip,
    get_open_port,
    make_zmq_socket,
)
```
**EN:** This block imports `contextlib`, `threading`, `time`, `collections.abc`, `dataclasses`, `typing` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `contextlib`, `threading`, `time`, `collections.abc`, `dataclasses`, `typing`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    pass
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Imports and setup / 导入与初始化
```python
from dataclasses import field
from enum import Enum
```
**EN:** This block imports `dataclasses`, `enum` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `enum`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)


Transfer = tuple[int, float]
EngineId = str
ReqId = str
TransferId = str
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, `Transfer`, `EngineId`, `ReqId`, `TransferId`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`, `Transfer`, `EngineId`, `ReqId`, `TransferId`，供后续代码复用。

### Class `WriteTask` / 类 `WriteTask`
```python
@dataclass
class WriteTask:
    request_id: ReqId
    transfer_id: TransferId
    dst_engine_id: str
    local_block_ids: list[int]
    remote_block_ids_hint: list[int] | None
    layer_name: str
    event: torch.cuda.Event
    remote_notify_port: int
    remote_ip: str
    enqueue_time: float = field(default_factory=time.perf_counter)
    retried: int = 0
```
**EN:** Declares `WriteTask`, a dataclass. It packages structured data fields such as `request_id`, `transfer_id`, `dst_engine_id`, `local_block_ids`, `remote_block_ids_hint`.
**CN:** 声明 `WriteTask`，它是一个数据类。 它封装了 `request_id`, `transfer_id`, `dst_engine_id`, `local_block_ids`, `remote_block_ids_hint` 等结构化字段。

### Class `LayerTransferPlan` / 类 `LayerTransferPlan`
```python
@dataclass
class LayerTransferPlan:
    """Plan for transferring a single layer."""

    request_id: ReqId
    transfer_id: TransferId
    layer_name: str
    sess_idx: int
    transfer_local_offsets: list[int]
    transfer_remote_offsets: list[int]
    transfer_sizes: list[int]
    use_batch: bool = True
```
**EN:** Declares `LayerTransferPlan`, a dataclass. It packages structured data fields such as `request_id`, `transfer_id`, `layer_name`, `sess_idx`, `transfer_local_offsets`. The docstring summarizes its role as: Plan for transferring a single layer.
**CN:** 声明 `LayerTransferPlan`，它是一个数据类。 它封装了 `request_id`, `transfer_id`, `layer_name`, `sess_idx`, `transfer_local_offsets` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `RemoteAllocInfo` / 类 `RemoteAllocInfo`
```python
@dataclass
class RemoteAllocInfo:
    """Information about remote block allocation."""

    block_ids: list[int]
    writes_done: int = 0
    decode_dp_rank: int = 0
    transfer_offset: tuple[list[int], list[int], list[int]] | None = None
```
**EN:** Declares `RemoteAllocInfo`, a dataclass. It packages structured data fields such as `block_ids`, `writes_done`, `decode_dp_rank`, `transfer_offset`. The docstring summarizes its role as: Information about remote block allocation.
**CN:** 声明 `RemoteAllocInfo`，它是一个数据类。 它封装了 `block_ids`, `writes_done`, `decode_dp_rank`, `transfer_offset` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `ROLE` / 类 `ROLE`
```python
class ROLE(Enum):
    PRODUCER = "producer"
    CONSUMER = "consumer"
    NOTINIT = "notinit"
```
**EN:** Declares `ROLE`, a enum derived from `Enum`. It enumerates values such as `PRODUCER`, `CONSUMER`, `NOTINIT` to model roles or states.
**CN:** 声明 `ROLE`，它是一个枚举，继承自 `Enum`。 它通过 `PRODUCER`, `CONSUMER`, `NOTINIT` 等枚举值表达角色或状态。

### Class `MoRIIOAgentMetadata` / 类 `MoRIIOAgentMetadata`
```python
class MoRIIOAgentMetadata(
    msgspec.Struct,
    omit_defaults=True,  # type: ignore[call-arg]
    # required for @cached_property.d
    dict=True,
):
    engine_id: str
    agent_metadata: bytes
    kv_caches_base_addr: list[int]
    num_blocks: int
    block_len: int
    attn_backend_name: str
```
**EN:** Declares `MoRIIOAgentMetadata`, a class derived from `msgspec.Struct`.
**CN:** 声明 `MoRIIOAgentMetadata`，它是一个类，继承自 `msgspec.Struct`。

### Class `RoleManager` / 类 `RoleManager`
```python
class RoleManager:
    """Manages role state across the connector."""

    _instance: "RoleManager | None" = None
    _lock = threading.Lock()

    def __init__(self) -> None:
        self._role: ROLE = ROLE.NOTINIT

    @classmethod
    def get_instance(cls) -> "RoleManager":
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = cls()
        return cls._instance

    def set_role(self, role: ROLE) -> None:
        """Set the current role."""
        with self._lock:
            self._role = role

    def get_role(self) -> ROLE:
        """Get the current role."""
        return self._role
```
**EN:** Declares `RoleManager`, a class. Key methods include `__init__`, `get_instance`, `set_role`, `get_role`. The docstring summarizes its role as: Manages role state across the connector.
**CN:** 声明 `RoleManager`，它是一个类。 关键方法包括 `__init__`, `get_instance`, `set_role`, `get_role`。 文档字符串概括了它在整体流程中的职责。

### Function `set_role` / 函数 `set_role`
```python
def set_role(role: ROLE):
    """Set the global role."""
    RoleManager.get_instance().set_role(role)
```
**EN:** `set_role` updates state or configuration for this module. The docstring frames it as: Set the global role. It primarily works with arguments like `role`. Key calls include `RoleManager.get_instance().set_role`, `RoleManager.get_instance`.
**CN:** `set_role` 负责更新状态或配置。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `role` 这样的参数。 关键调用包括 `RoleManager.get_instance().set_role`, `RoleManager.get_instance`。

### Function `get_role` / 函数 `get_role`
```python
def get_role() -> ROLE:
    """Get the global role."""
    return RoleManager.get_instance().get_role()
```
**EN:** `get_role` is a thin wrapper around `RoleManager.get_instance().get_role`, exposing that operation through a module-level helper. The docstring frames it as: Get the global role. Key calls include `RoleManager.get_instance().get_role`, `RoleManager.get_instance`.
**CN:** `get_role` 是对 `RoleManager.get_instance().get_role` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 关键调用包括 `RoleManager.get_instance().get_role`, `RoleManager.get_instance`。

### Class `MoRIIOMode` / 类 `MoRIIOMode`
```python
class MoRIIOMode(Enum):
    READ = "read"
    WRITE = "write"
```
**EN:** Declares `MoRIIOMode`, a enum derived from `Enum`. It enumerates values such as `READ`, `WRITE` to model roles or states.
**CN:** 声明 `MoRIIOMode`，它是一个枚举，继承自 `Enum`。 它通过 `READ`, `WRITE` 等枚举值表达角色或状态。

### Class `MoRIIOError` / 类 `MoRIIOError`
```python
class MoRIIOError(Exception):
    """Base exception for MoRIIO operations."""

    pass
```
**EN:** Declares `MoRIIOError`, a class derived from `Exception`. The docstring summarizes its role as: Base exception for MoRIIO operations.
**CN:** 声明 `MoRIIOError`，它是一个类，继承自 `Exception`。 文档字符串概括了它在整体流程中的职责。

### Class `HandshakeError` / 类 `HandshakeError`
```python
class HandshakeError(MoRIIOError):
    """Exception raised when handshake fails."""

    pass
```
**EN:** Declares `HandshakeError`, a class derived from `MoRIIOError`. The docstring summarizes its role as: Exception raised when handshake fails.
**CN:** 声明 `HandshakeError`，它是一个类，继承自 `MoRIIOError`。 文档字符串概括了它在整体流程中的职责。

### Class `TransferError` / 类 `TransferError`
```python
class TransferError(MoRIIOError):
    """Exception raised when transfer fails."""

    pass
```
**EN:** Declares `TransferError`, a class derived from `MoRIIOError`. The docstring summarizes its role as: Exception raised when transfer fails.
**CN:** 声明 `TransferError`，它是一个类，继承自 `MoRIIOError`。 文档字符串概括了它在整体流程中的职责。

### Function `get_moriio_mode` / 函数 `get_moriio_mode`
```python
def get_moriio_mode() -> MoRIIOMode:
    read_mode = envs.VLLM_MORIIO_CONNECTOR_READ_MODE
    logger.debug("MoRIIO Connector read_mode: %s", read_mode)
    if read_mode:
        return MoRIIOMode.READ
    else:
        return MoRIIOMode.WRITE
```
**EN:** `get_moriio_mode` retrieves state or computed results for this module. Key calls include `logger.debug`.
**CN:** `get_moriio_mode` 负责获取状态或计算结果。 关键调用包括 `logger.debug`。

### Function `get_port_offset` / 函数 `get_port_offset`
```python
def get_port_offset(dp_rank: int, tp_rank: int, tp_size: int = 1) -> int:
    return (dp_rank) * tp_size + tp_rank
```
**EN:** `get_port_offset` retrieves state or computed results for this module. It primarily works with arguments like `dp_rank`, `tp_rank`, `tp_size`.
**CN:** `get_port_offset` 负责获取状态或计算结果。 它主要处理诸如 `dp_rank`, `tp_rank`, `tp_size` 这样的参数。

### Class `MoRIIOConfig` / 类 `MoRIIOConfig`
```python
@dataclass
class MoRIIOConfig:
    local_ip: str
    local_kv_port: int
    proxy_ip: str
    local_ping_port: int
    proxy_ping_port: int
    http_port: int
    handshake_port: int
    notify_port: int
    tp_rank: int
    dp_rank: int
    dp_size: int
    tp_size: int

    @classmethod
    def from_vllm_config(cls, vllm_config: VllmConfig) -> "MoRIIOConfig":
        # Port Configuration:
        # local_ping_port   -> Outgoing heartbeat to proxy
        # proxy_ping_port   -> Remote proxy's heartbeat ingress port
        # http_port         -> Instance's HTTP service endpoint
        # local_kv_port     -> service port for mori engine
        # notify_port       -> For synchronizing stages between prefill and decode
        # handshake_port    -> For initial handshake between mori engine

        # TODO : merge notify_port and handshake_port to simplify port management
        #        supports non-contiguous ports
        assert vllm_config.kv_transfer_config is not None, (
            "kv_transfer_config must be set for MoRIIOConnector"
        )
        kv_transfer_config = vllm_config.kv_transfer_config
        extra_config = kv_transfer_config.kv_connector_extra_config
        tp_rank = get_tensor_model_parallel_rank()
        dp_rank = vllm_config.parallel_config.data_parallel_rank
        base_notify_port = int(extra_config["notify_port"])
        dp_size = vllm_config.parallel_config.data_parallel_size
        tp_size = get_tensor_model_parallel_world_size()
        port_offset = get_port_offset(dp_rank, tp_rank)

        return cls(
            local_ip=get_ip(),
            local_kv_port=get_open_port(),
            proxy_ip=extra_config["proxy_ip"],
            local_ping_port=get_open_port(),
            proxy_ping_port=int(extra_config["proxy_ping_port"]),
            http_port=int(extra_config["http_port"]),
            handshake_port=int(extra_config["handshake_port"]),
            notify_port=base_notify_port + port_offset,
            tp_rank=tp_rank,
            dp_rank=dp_rank,
            dp_size=dp_size,
            tp_size=tp_size,
        )
```
**EN:** Declares `MoRIIOConfig`, a dataclass. It packages structured data fields such as `local_ip`, `local_kv_port`, `proxy_ip`, `local_ping_port`, `proxy_ping_port`.
**CN:** 声明 `MoRIIOConfig`，它是一个数据类。 它封装了 `local_ip`, `local_kv_port`, `proxy_ip`, `local_ping_port`, `proxy_ping_port` 等结构化字段。

### Class `MoRIIOConstants` / 类 `MoRIIOConstants`
```python
class MoRIIOConstants:
    """Constants for MoRIIO connector."""

    # ZMQ message types
    GET_META_MSG = b"get_meta_msg"
    POP_DONE_RECV = b"pop_done_recv"
    OVER = b"OVER"
    COMPLETION_PREFIX = "cmpl"
    TRANSFER_PREFIX = "tx"

    PING_INTERVAL = 3
    MAX_PING_RETRIES = 100
    DEFAULT_HANDSHAKE_PORT = "6301"
    DEFAULT_NOTIFY_PORT = "61005"

    VLLM_MORI_READ_ABORT_REQUEST_TIMEOUT = 3600
```
**EN:** Declares `MoRIIOConstants`, a class. The docstring summarizes its role as: Constants for MoRIIO connector.
**CN:** 声明 `MoRIIOConstants`，它是一个类。 文档字符串概括了它在整体流程中的职责。

### Module constants / 模块常量
```python
_PREFILL_ZMQ_RE = re.compile(r"___prefill_addr_(.+?)___decode_addr_")
# vLLM wraps the router's X-Request-Id as "cmpl-<id>-<seq>-<hex>" so there may
# be a trailing "-<seq>-<hex>" suffix after the 32-char UUID.  Allow it.
_DECODE_ZMQ_RE = re.compile(r"___decode_addr_(.+)_[0-9a-f]{32}(?:-.*)?$")
```
**EN:** This section defines module-level aliases, constants, or shared state such as `_PREFILL_ZMQ_RE`, `_DECODE_ZMQ_RE`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `_PREFILL_ZMQ_RE`, `_DECODE_ZMQ_RE`，供后续代码复用。

### Function `parse_moriio_zmq_address` / 函数 `parse_moriio_zmq_address`
```python
def parse_moriio_zmq_address(
    zmq_address: str,
) -> tuple[str, int, int]:
    """Parse the MoRI-IO zmq address into its components.

    Parses ``"host:IP,handshake:PORT,notify:PORT"`` into
        (host, handshake_port, notify_port).

    Each key-value pair is split on the *first* colon so that IPv6 addresses
    (e.g. ``host:::1``) are handled correctly.  Raises ``ValueError`` if any
    of ``host``, ``handshake``, or ``notify`` keys are absent or if the port
    values are non-numeric.
    """
    parts: dict[str, str] = {}
    for segment in zmq_address.split(","):
        key, _, val = segment.partition(":")
        parts[key.strip()] = val.strip()
    try:
        host = parts["host"]
        handshake_port = int(parts["handshake"])
        notify_port = int(parts["notify"])
    except (KeyError, ValueError) as e:
        raise ValueError(
            f"Malformed zmq_address {zmq_address!r}: expected "
            f"'host:IP,handshake:PORT,notify:PORT' format"
        ) from e
    return host, handshake_port, notify_port
```
**EN:** `parse_moriio_zmq_address` implements a focused helper routine for this module. The docstring frames it as: Parse the MoRI-IO zmq address into its components. It primarily works with arguments like `zmq_address`. Key calls include `zmq_address.split`, `segment.partition`, `val.strip`.
**CN:** `parse_moriio_zmq_address` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `zmq_address` 这样的参数。 关键调用包括 `zmq_address.split`, `segment.partition`, `val.strip`。

### Function `get_peer_zmq_from_request_id` / 函数 `get_peer_zmq_from_request_id`
```python
def get_peer_zmq_from_request_id(request_id: str, is_producer: bool) -> str:
    """Extract the *peer's* zmq_address from the vLLM router request_id.

    The producer (prefill) needs the decode's address; the consumer (decode)
    needs the prefill's address.
    """
    if is_producer:
        m = _DECODE_ZMQ_RE.search(request_id)
    else:
        m = _PREFILL_ZMQ_RE.search(request_id)
    if m is None:
        raise ValueError(
            f"Cannot parse peer zmq_address from request_id: {request_id!r}"
        )
    return m.group(1)
```
**EN:** `get_peer_zmq_from_request_id` retrieves state or computed results for this module. The docstring frames it as: Extract the *peer's* zmq_address from the vLLM router request_id. It primarily works with arguments like `request_id`, `is_producer`. Key calls include `m.group`, `_DECODE_ZMQ_RE.search`, `_PREFILL_ZMQ_RE.search`.
**CN:** `get_peer_zmq_from_request_id` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `request_id`, `is_producer` 这样的参数。 关键调用包括 `m.group`, `_DECODE_ZMQ_RE.search`, `_PREFILL_ZMQ_RE.search`。

### Class `ReqMeta` / 类 `ReqMeta`
```python
@dataclass
class ReqMeta:
    """Metadata for a single request."""

    transfer_id: TransferId
    local_block_ids: list[int]
    remote_block_ids: list[int]
    remote_host: str
    remote_port: int
    remote_handshake_port: int
    remote_notify_port: int
    remote_engine_id: str
    tp_size: int
    remote_dp_size: int
```
**EN:** Declares `ReqMeta`, a dataclass. It packages structured data fields such as `transfer_id`, `local_block_ids`, `remote_block_ids`, `remote_host`, `remote_port`. The docstring summarizes its role as: Metadata for a single request.
**CN:** 声明 `ReqMeta`，它是一个数据类。 它封装了 `transfer_id`, `local_block_ids`, `remote_block_ids`, `remote_host`, `remote_port` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `MoRIIOConnectorMetadata` / 类 `MoRIIOConnectorMetadata`
```python
class MoRIIOConnectorMetadata(KVConnectorMetadata):
    def __init__(self):
        self.reqs_to_recv: dict[ReqId, ReqMeta] = {}
        self.reqs_to_save: dict[ReqId, ReqMeta] = {}
        self.reqs_to_send: dict[ReqId, float] = {}
        self.transfer_id_to_request_id: dict[TransferId, ReqId] = {}

    def __repr__(self):
        return (
            f"MoRIIOConnectorMetadata: reqs_to_recv={self.reqs_to_recv}, "
            f"reqs_to_save={self.reqs_to_save}, "
            f"reqs_to_send={self.reqs_to_send}, "
            f"transfer_id_to_request_id={self.transfer_id_to_request_id}"
        )

    def add_new_req(
        self,
        request_id: ReqId,
        local_block_ids: list[int],
        kv_transfer_params: dict[str, Any],
        write_mode=False,
    ):
        transfer_id = kv_transfer_params["transfer_id"]

        # Parse host/ports from the request_id. The router embeds both zmq_addresses
        # in the request_id
        peer_zmq = get_peer_zmq_from_request_id(request_id, is_producer=write_mode)
        remote_host, remote_handshake_port, remote_notify_port = (
            parse_moriio_zmq_address(peer_zmq)
        )

        _req = ReqMeta(
            transfer_id=transfer_id,
            local_block_ids=local_block_ids,
            remote_block_ids=kv_transfer_params["remote_block_ids"],
            remote_engine_id=kv_transfer_params["remote_engine_id"],
            remote_host=remote_host,
            remote_port=remote_handshake_port,
            remote_handshake_port=remote_handshake_port,
            remote_notify_port=remote_notify_port,
            tp_size=kv_transfer_params.get("tp_size", 1),
            remote_dp_size=kv_transfer_params.get("remote_dp_size", 1),
        )
        if write_mode:
            self.reqs_to_save[request_id] = _req
        else:
            self.reqs_to_recv[request_id] = _req
```
**EN:** Declares `MoRIIOConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `__repr__`, `add_new_req`.
**CN:** 声明 `MoRIIOConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `__repr__`, `add_new_req`。

### Function `zmq_ctx` / 函数 `zmq_ctx`
```python
@contextlib.contextmanager
def zmq_ctx(socket_type: Any, addr: str) -> Iterator[zmq.Socket]:
    """Context manager for a ZMQ socket"""

    if socket_type not in (zmq.ROUTER, zmq.REQ, zmq.DEALER):
        raise ValueError(f"Unexpected socket type: {socket_type}")

    ctx: zmq.Context | None = None
    try:
        ctx = zmq.Context()  # type: ignore[attr-defined]
        yield make_zmq_socket(
            ctx=ctx, path=addr, socket_type=socket_type, bind=socket_type == zmq.ROUTER
        )
    finally:
        if ctx is not None:
            ctx.destroy(linger=0)
```
**EN:** `zmq_ctx` implements a focused helper routine for this module. The docstring frames it as: Context manager for a ZMQ socket. It primarily works with arguments like `socket_type`, `addr`. Key calls include `ValueError`, `zmq.Context`, `make_zmq_socket`.
**CN:** `zmq_ctx` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `socket_type`, `addr` 这样的参数。 关键调用包括 `ValueError`, `zmq.Context`, `make_zmq_socket`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `WriteTask`: dataclass interface or data carrier / `WriteTask`：数据类接口或数据载体
- `LayerTransferPlan`: dataclass interface or data carrier / `LayerTransferPlan`：数据类接口或数据载体
- `RemoteAllocInfo`: dataclass interface or data carrier / `RemoteAllocInfo`：数据类接口或数据载体
- `ROLE`: enum interface or data carrier / `ROLE`：枚举接口或数据载体
- `MoRIIOAgentMetadata`: class interface or data carrier / `MoRIIOAgentMetadata`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `threading`, `time`, `collections.abc`, `dataclasses`, `typing`, `enum`
- **Third-party / 第三方**: `msgspec`, `regex`, `torch`, `zmq`
- **Internal modules / 内部模块**: `vllm`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.utils.network_utils`
