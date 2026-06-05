# metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/nixl/metadata.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Metadata dataclasses and helpers for the NIXL connector / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Metadata dataclasses and helpers for the NIXL connector."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Metadata dataclasses and helpers for the NIXL connector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
from dataclasses import dataclass
from typing import Any

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.utils import BlockIds, EngineId
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorHandshakeMetadata,
    KVConnectorMetadata,
)
from vllm.logger import init_logger
```
**EN:** This block imports `dataclasses`, `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `typing`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)

TransferHandle = int
ReqId = str

GET_META_MSG = b"get_meta_msg"
#
# NIXL Connector Version
#
# Increment this version whenever there is an incompatible change to:
#   - NixlAgentMetadata schema
#   - kv_transfer_params schema or semantics
#   - NIXL transfer protocol or wire format
#   - KV cache memory layout or block organization
#   - Any other change that breaks P/D interoperability
#
# Version History:
#   1: Initial version with compatibility checking
#   2: Add remote_request_id to kv_transfer_params
#   3: Add physical_blocks_per_logical_kv_block to NixlAgentMetadata
#   4: Add KV block lease renewal through heartbeats
#
NIXL_CONNECTOR_VERSION: int = 4
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, `TransferHandle`, `ReqId`, `GET_META_MSG`, `NIXL_CONNECTOR_VERSION`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`, `TransferHandle`, `ReqId`, `GET_META_MSG`, `NIXL_CONNECTOR_VERSION`，供后续代码复用。

### Class `NixlAgentMetadata` / 类 `NixlAgentMetadata`
```python
@dataclass
class NixlAgentMetadata:
    engine_id: str
    agent_metadata: bytes
    kv_caches_base_addr: list[int]
    device_id: int
    num_blocks: int
    block_lens: list[int]
    kv_cache_layout: str
    block_size: int
    ssm_sizes: tuple[int, int]
    attn_backend_name: str
    physical_blocks_per_logical_kv_block: int
```
**EN:** Declares `NixlAgentMetadata`, a dataclass. It packages structured data fields such as `engine_id`, `agent_metadata`, `kv_caches_base_addr`, `device_id`, `num_blocks`.
**CN:** 声明 `NixlAgentMetadata`，它是一个数据类。 它封装了 `engine_id`, `agent_metadata`, `kv_caches_base_addr`, `device_id`, `num_blocks` 等结构化字段。

### Class `NixlHandshakePayload` / 类 `NixlHandshakePayload`
```python
@dataclass
class NixlHandshakePayload(KVConnectorHandshakeMetadata):
    """
    Wrapper for NIXL handshake sent over the wire.

    Enables two-phase decoding for graceful compatibility checking:
    1. Decode NixlHandshakePayload to get compatibility_hash
    2. Compute local hash and compare
    3. Only if hashes match, decode agent_metadata_bytes

    This prevents decoder errors when NixlAgentMetadata schema is
    incompatible, allowing graceful failure with clear error message.
    """

    compatibility_hash: str
    agent_metadata_bytes: bytes  # NixlAgentMetadata encoded
```
**EN:** Declares `NixlHandshakePayload`, a dataclass derived from `KVConnectorHandshakeMetadata`. It packages structured data fields such as `compatibility_hash`, `agent_metadata_bytes`. The docstring summarizes its role as: Wrapper for NIXL handshake sent over the wire.
**CN:** 声明 `NixlHandshakePayload`，它是一个数据类，继承自 `KVConnectorHandshakeMetadata`。 它封装了 `compatibility_hash`, `agent_metadata_bytes` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Function `compute_nixl_compatibility_hash` / 函数 `compute_nixl_compatibility_hash`
```python
def compute_nixl_compatibility_hash(
    vllm_config: VllmConfig, attn_backend_name: str, cross_layers_blocks: bool
) -> str:
    """
    Compute compatibility hash for NIXL KV transfer.

    Hash only the factors that affect whether two NIXL instances can
    successfully transfer KV cache data.

    Factors included:
    - vLLM version and NIXL connector version
    - Model architecture (name, dtype, KV heads, layers)
    - KV cache format (dtype, sliding window)
    - Attention backend

    Note: Factors like tensor_parallel_size, block_size, and kv_cache_layout
    are validated at runtime in _validate_remote_agent_handshake and are not
    included in this hash to support heterogeneous deployments.

    Note - the set of factors are likely to evolve significantly over
    time to be more or less permissive.

    Returns:
        SHA-256 hex digest
    """
    from vllm import __version__ as vllm_version
    from vllm.config.utils import hash_factors

    model_config = vllm_config.model_config
    cache_config = vllm_config.cache_config
    is_hma_enabled = not vllm_config.scheduler_config.disable_hybrid_kv_cache_manager

    factors = {
        # Version compatibility
        "vllm_version": vllm_version,
        "nixl_connector_version": NIXL_CONNECTOR_VERSION,
        # Model architecture - affects KV cache shape
        "model": model_config.model,
        "dtype": str(model_config.dtype),
        "num_kv_heads": model_config.get_total_num_kv_heads(),
        "head_size": model_config.get_head_size(),
        "num_hidden_layers": model_config.get_total_num_hidden_layers(),
        # Attention backend and KV cache dtype affect memory layout
        "attn_backend_name": attn_backend_name,
        "cache_dtype": str(cache_config.cache_dtype),
# ... truncated for analysis ...
        "NIXL compatibility hash: %s (model=%s, dtype=%s, num_kv_heads=%d, "
        "cache_dtype=%s, attn_backend=%s)",
        compat_hash,
        factors["model"],
        factors["dtype"],
        factors["num_kv_heads"],
        factors["cache_dtype"],
        attn_backend_name,
    )
    return compat_hash
```
**EN:** `compute_nixl_compatibility_hash` implements a focused helper routine for this module. The docstring frames it as: Compute compatibility hash for NIXL KV transfer. It primarily works with arguments like `vllm_config`, `attn_backend_name`, `cross_layers_blocks`. Key calls include `hash_factors`, `logger.debug`, `str`.
**CN:** `compute_nixl_compatibility_hash` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `vllm_config`, `attn_backend_name`, `cross_layers_blocks` 这样的参数。 关键调用包括 `hash_factors`, `logger.debug`, `str`。

### Class `HeartbeatInfo` / 类 `HeartbeatInfo`
```python
@dataclass
class HeartbeatInfo:
    """Heartbeat data for a single remote engine, sent from D worker to P."""

    req_ids: set[ReqId]
    host: str
    port: int
    tp_size: int
```
**EN:** Declares `HeartbeatInfo`, a dataclass. It packages structured data fields such as `req_ids`, `host`, `port`, `tp_size`. The docstring summarizes its role as: Heartbeat data for a single remote engine, sent from D worker to P.
**CN:** 声明 `HeartbeatInfo`，它是一个数据类。 它封装了 `req_ids`, `host`, `port`, `tp_size` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `RemoteMeta` / 类 `RemoteMeta`
```python
@dataclass
class RemoteMeta:
    block_ids: BlockIds
    host: str
    port: int
    engine_id: str
    request_id: str
```
**EN:** Declares `RemoteMeta`, a dataclass. It packages structured data fields such as `block_ids`, `host`, `port`, `engine_id`, `request_id`.
**CN:** 声明 `RemoteMeta`，它是一个数据类。 它封装了 `block_ids`, `host`, `port`, `engine_id`, `request_id` 等结构化字段。

### Class `ReqMeta` / 类 `ReqMeta`
```python
@dataclass
class ReqMeta:
    local_block_ids: BlockIds
    # To be used when logical block size does not match the kernel block size
    local_physical_block_ids: BlockIds
    tp_size: int
    remote: RemoteMeta | None = None
```
**EN:** Declares `ReqMeta`, a dataclass. It packages structured data fields such as `local_block_ids`, `local_physical_block_ids`, `tp_size`, `remote`.
**CN:** 声明 `ReqMeta`，它是一个数据类。 它封装了 `local_block_ids`, `local_physical_block_ids`, `tp_size`, `remote` 等结构化字段。

### Class `NixlConnectorMetadata` / 类 `NixlConnectorMetadata`
```python
class NixlConnectorMetadata(KVConnectorMetadata):
    def __init__(self):
        self.reqs_to_recv: dict[ReqId, ReqMeta] = {}
        self.reqs_to_save: dict[ReqId, ReqMeta] = {}
        self.reqs_to_send: dict[ReqId, float] = {}
        self.reqs_in_batch: set[ReqId] = set()
        self.reqs_not_processed: set[ReqId] = set()
        # Heartbeat data grouped by remote engine, sent by D worker to P.
        self.heartbeat_by_engine: dict[EngineId, HeartbeatInfo] = {}

    def _add_new_req(
        self,
        local_block_ids: BlockIds,
        kv_transfer_params: dict[str, Any],
    ) -> ReqMeta:
        return ReqMeta(
            local_block_ids=local_block_ids,
            local_physical_block_ids=local_block_ids,
            # P workers don't need to receive tp_size from proxy here.
            tp_size=kv_transfer_params.get("tp_size", 1),
        )

    def add_new_req_to_save(
        self,
        request_id: ReqId,
        local_block_ids: BlockIds,
        kv_transfer_params: dict[str, Any],
    ):
        self.reqs_to_save[request_id] = self._add_new_req(
            local_block_ids, kv_transfer_params
        )

    def add_new_req_to_recv(
        self,
        request_id: ReqId,
        local_block_ids: BlockIds,
        kv_transfer_params: dict[str, Any],
    ):
        req = self._add_new_req(local_block_ids, kv_transfer_params)
        req.remote = RemoteMeta(
            block_ids=kv_transfer_params["remote_block_ids"],
            engine_id=kv_transfer_params["remote_engine_id"],
            request_id=kv_transfer_params["remote_request_id"],
            host=kv_transfer_params["remote_host"],
            port=kv_transfer_params["remote_port"],
        )
        self.reqs_to_recv[request_id] = req
```
**EN:** Declares `NixlConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `_add_new_req`, `add_new_req_to_save`, `add_new_req_to_recv`.
**CN:** 声明 `NixlConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `_add_new_req`, `add_new_req_to_save`, `add_new_req_to_recv`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `NixlAgentMetadata`: dataclass interface or data carrier / `NixlAgentMetadata`：数据类接口或数据载体
- `NixlHandshakePayload`: dataclass interface or data carrier / `NixlHandshakePayload`：数据类接口或数据载体
- `compute_nixl_compatibility_hash`: module-level helper or API entry / `compute_nixl_compatibility_hash`：模块级辅助函数或 API 入口
- `HeartbeatInfo`: dataclass interface or data carrier / `HeartbeatInfo`：数据类接口或数据载体
- `RemoteMeta`: dataclass interface or data carrier / `RemoteMeta`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`, `vllm`, `vllm.config.utils`
