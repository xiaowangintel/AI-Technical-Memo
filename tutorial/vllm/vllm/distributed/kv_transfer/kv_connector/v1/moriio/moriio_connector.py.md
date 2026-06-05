# moriio_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/moriio/moriio_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements MoriIO KV-transfer adapters and supporting logic. / 实现 MoriIO KV 传输适配器及其配套逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import logging
import math
import queue
import threading
import time
from collections import defaultdict
from concurrent.futures import Future, ThreadPoolExecutor
from typing import TYPE_CHECKING, Any

import msgpack
import msgspec
import numpy as np
import torch
import zmq

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorBase_V1,
    KVConnectorMetadata,
    KVConnectorRole,
)
from vllm.distributed.kv_transfer.kv_connector.v1.moriio.moriio_common import (
    ROLE,
    EngineId,
    HandshakeError,
    MoRIIOAgentMetadata,
    MoRIIOConfig,
    MoRIIOConnectorMetadata,
    MoRIIOConstants,
    MoRIIOMode,
    ReqId,
    ReqMeta,
    TransferId,
    WriteTask,
    get_moriio_mode,
    get_peer_zmq_from_request_id,
    get_port_offset,
    get_role,
    parse_moriio_zmq_address,
    set_role,
    zmq_ctx,
)
from vllm.distributed.kv_transfer.kv_connector.v1.moriio.moriio_engine import (
    MoRIIOWrapper,
    MoRIIOWriter,
# ... truncated for analysis ...
from vllm.forward_context import ForwardContext
from vllm.logger import init_logger
from vllm.utils.network_utils import (
    get_ip,
    make_zmq_path,
    make_zmq_socket,
)
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.request import RequestStatus
```
**EN:** This block imports `logging`, `math`, `queue`, `threading`, `time`, `collections` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `logging`, `math`, `queue`, `threading`, `time`, `collections`，为后续实现准备运行时、类型与辅助 API。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.v1.attention.backend import AttentionMetadata
    from vllm.v1.core.kv_cache_manager import KVCacheBlocks
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.request import Request
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Guarded block / 保护代码块
```python
try:
    from mori.io import (
        BackendType,
        IOEngine,
        IOEngineConfig,
    )

    logger.info("MoRIIO is available")
    MoRIIO_enabled = True
except ImportError:
    logger.error("MoRIIO is not available")
    MoRIIO_enabled = False
```
**EN:** This guarded block attempts optional imports such as `mori.io` and falls back when those dependencies are unavailable.
**CN:** 该保护代码块会尝试导入 `mori.io` 等可选依赖，并在依赖不存在时回退。

### Function `is_moriio_available` / 函数 `is_moriio_available`
```python
def is_moriio_available() -> bool:
    return MoRIIO_enabled
```
**EN:** `is_moriio_available` checks a boolean property or state for this module.
**CN:** `is_moriio_available` 负责检查布尔属性或状态。

### Class `MoRIIOConnector` / 类 `MoRIIOConnector`
```python
class MoRIIOConnector(KVConnectorBase_V1):
    def __init__(
        self,
        vllm_config: VllmConfig,
        role: KVConnectorRole,
        kv_cache_config: "KVCacheConfig",
    ):
        super().__init__(vllm_config, role, kv_cache_config)
        assert vllm_config.kv_transfer_config is not None, (
            "kv_transfer_config must be set for MoRIIOConnector"
        )

        self.kv_transfer_config = vllm_config.kv_transfer_config
        self._set_port_defaults(vllm_config)

        self.engine_id = (
            str(get_ip())
            + ":"
            + str(self.kv_transfer_config.kv_connector_extra_config["handshake_port"])
        )
        self.mode = get_moriio_mode()
        if role == KVConnectorRole.SCHEDULER:
            self.connector_scheduler: MoRIIOConnectorScheduler | None = (
                MoRIIOConnectorScheduler(vllm_config, self.engine_id)
            )
            self.connector_worker: MoRIIOConnectorWorker | None = None
        elif role == KVConnectorRole.WORKER:
            self.connector_scheduler = None
            self.connector_worker = MoRIIOConnectorWorker(vllm_config, self.engine_id)
        logger.info(
            "Initialized MoRIIO Connector,engine_id:%s,role: %s",
            self.engine_id,
            role.value,
        )

    ############################################################
    # Scheduler Side Methods
    ############################################################

    def _set_port_defaults(self, vllm_config: VllmConfig):
        assert vllm_config.kv_transfer_config is not None, (
            "kv_transfer_config must be set for MoRIIOConnector"
        )
        kv_transfer_config = vllm_config.kv_transfer_config
        extra_config = kv_transfer_config.kv_connector_extra_config
# ... truncated for analysis ...
    def has_connector_metadata(self) -> bool:
        """Check whether the connector metadata is currently set.

        Returns:
            bool: True if connector metadata exists, False otherwise.
        """
        try:
            return self._connector_metadata is not None
        except AttributeError:
            return False
```
**EN:** Declares `MoRIIOConnector`, a class derived from `KVConnectorBase_V1`. Key methods include `__init__`, `_set_port_defaults`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`.
**CN:** 声明 `MoRIIOConnector`，它是一个类，继承自 `KVConnectorBase_V1`。 关键方法包括 `__init__`, `_set_port_defaults`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`。

### Class `MoRIIOConnectorScheduler` / 类 `MoRIIOConnectorScheduler`
```python
class MoRIIOConnectorScheduler:
    """Implementation of Scheduler side methods"""

    def __init__(self, vllm_config: VllmConfig, engine_id: str):
        self.vllm_config = vllm_config

        assert vllm_config.kv_transfer_config is not None, (
            "kv_transfer_config must be set for MoRIIOConnector"
        )
        self.kv_transfer_config = vllm_config.kv_transfer_config
        self.block_size = vllm_config.cache_config.block_size
        self.engine_id: EngineId = engine_id
        self.mode = get_moriio_mode()
        self.host_ip = get_ip()
        self.handshake_port = self.kv_transfer_config.kv_connector_extra_config[
            "handshake_port"
        ]
        logger.info("Initializing MoRIIO Scheduler engine_id = %s", engine_id)

        self.side_notify_port = self.kv_transfer_config.kv_connector_extra_config[
            "notify_port"
        ]
        self.tp_size = self.vllm_config.parallel_config.tensor_parallel_size
        self.dp_rank = self.vllm_config.parallel_config.data_parallel_rank
        self.is_producer = self.kv_transfer_config.kv_role == "kv_producer"
        # Requests that need to start recv/send.
        # New requests are added by update_state_after_alloc in
        # the scheduler. Used to make metadata passed to Worker.
        self._reqs_need_recv: dict[ReqId, tuple[Request, list[int]]] = {}
        self._reqs_need_save: dict[ReqId, tuple[Request, list[int]]] = {}

        # For chunked prefill, we perform layer-wise access within the final chunk.
        # TODO: Perform transfer at end chunk.
        self._reqs_need_pending_save: dict[ReqId, tuple[Request, list[int]]] = {}

        if self.is_producer:
            set_role(ROLE.PRODUCER)
        else:
            set_role(ROLE.CONSUMER)
        # Reqs to send and their expiration time
        self._reqs_need_send: dict[ReqId, float] = {}
        self.paths: dict[str, zmq.Socket] = {}
        self.transfer_id_to_request_id: dict[TransferId, ReqId] = {}
        self.request_id_to_transfer_id: dict[ReqId, TransferId] = {}
# ... truncated for analysis ...
        # Return KV transfer params forwarded verbatim to the decode instance by
        # the router.
        return delay_free_blocks, dict(
            do_remote_prefill=True,
            do_remote_decode=False,
            remote_block_ids=computed_block_ids,
            remote_engine_id=self.engine_id,
            tp_size=self.vllm_config.parallel_config.tensor_parallel_size,
            transfer_id=params["transfer_id"],
        )
```
**EN:** Declares `MoRIIOConnectorScheduler`, a class. Key methods include `__init__`, `map_request_id`, `unmap_request_id`, `get_num_new_matched_tokens`, `send_notify_block`. The docstring summarizes its role as: Implementation of Scheduler side methods.
**CN:** 声明 `MoRIIOConnectorScheduler`，它是一个类。 关键方法包括 `__init__`, `map_request_id`, `unmap_request_id`, `get_num_new_matched_tokens`, `send_notify_block`。 文档字符串概括了它在整体流程中的职责。

### Class `MoRIIOConnectorWorker` / 类 `MoRIIOConnectorWorker`
```python
class MoRIIOConnectorWorker:
    """Implementation of Worker side methods"""

    def __init__(self, vllm_config: VllmConfig, engine_id: str):
        if not is_moriio_available():
            raise RuntimeError(
                "MoRIIO is not available. Please ensure the 'mori' package "
                "is installed and properly configured."
            )

        self.moriio_config = MoRIIOConfig.from_vllm_config(vllm_config)
        self.mode = get_moriio_mode()

        logger.info("Initializing MoRIIO worker %s", engine_id)

        logging.getLogger("aiter").disabled = True

        # Config.
        self.vllm_config = vllm_config
        assert vllm_config.kv_transfer_config is not None, (
            "kv_transfer_config must be set for MoRIIOConnector"
        )
        self.kv_transfer_config = vllm_config.kv_transfer_config
        self.is_producer = self.kv_transfer_config.is_kv_producer

        if self.is_producer:
            set_role(ROLE.PRODUCER)
        else:
            set_role(ROLE.CONSUMER)
        # mori engine
        self._rank = get_world_group().rank
        self._local_rank = get_world_group().local_rank
        self.tp_rank = self.moriio_config.tp_rank
        self.dp_rank = self.moriio_config.dp_rank

        self.local_ip = self.moriio_config.local_ip
        self.local_kv_port = self.moriio_config.local_kv_port
        self.proxy_ip = self.moriio_config.proxy_ip
        self.local_ping_port = self.moriio_config.local_ping_port
        self.proxy_ping_port = self.moriio_config.proxy_ping_port
        self.http_port = self.moriio_config.http_port
        self.handshake_port = self.moriio_config.handshake_port
        self.notify_port = self.moriio_config.notify_port

        self.zmq_context = zmq.Context()
# ... truncated for analysis ...
            # TODO : apply multi-session batch-read when moriio support it
            transfer_status = self.moriio_wrapper.read_remote_data(
                offs[2], offs[0], offs[1], sessions[sess_idx]
            )
            with self.moriio_wrapper.lock:
                self._recving_transfers[request_id].append(transfer_status)
                self._recving_transfers_callback_addr[request_id] = (
                    remote_host,
                    str(remote_notify_port + self.tp_rank),
                )
```
**EN:** Declares `MoRIIOConnectorWorker`, a class. Key methods include `__init__`, `schedule_write_blocks`, `_get_built_session`, `_ping`, `shutdown`. The docstring summarizes its role as: Implementation of Worker side methods.
**CN:** 声明 `MoRIIOConnectorWorker`，它是一个类。 关键方法包括 `__init__`, `schedule_write_blocks`, `_get_built_session`, `_ping`, `shutdown`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `is_moriio_available`: module-level helper or API entry / `is_moriio_available`：模块级辅助函数或 API 入口
- `MoRIIOConnector`: class interface or data carrier / `MoRIIOConnector`：类接口或数据载体
- `MoRIIOConnectorScheduler`: class interface or data carrier / `MoRIIOConnectorScheduler`：类接口或数据载体
- `MoRIIOConnectorWorker`: class interface or data carrier / `MoRIIOConnectorWorker`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `math`, `queue`, `threading`, `time`, `collections`, `concurrent.futures`, `typing`
- **Third-party / 第三方**: `msgpack`, `msgspec`, `numpy`, `torch`, `zmq`, `mori.io`, `transformers`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.moriio.moriio_common`, `vllm.distributed.kv_transfer.kv_connector.v1.moriio.moriio_engine`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.logger`, `vllm.utils.network_utils`, `vllm.v1.attention.selector`, `vllm.v1.core.sched.output`, `vllm.v1.request`, `vllm.v1.attention.backend`
