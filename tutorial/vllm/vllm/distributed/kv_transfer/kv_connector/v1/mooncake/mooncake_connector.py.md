# mooncake_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/mooncake_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Mooncake-based KV-transfer connectors, stores, or runtime helpers. / 实现基于 Mooncake 的 KV 传输连接器、存储层或运行时辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import asyncio
import logging
import threading
import time
from collections import defaultdict
from concurrent.futures import ThreadPoolExecutor
from dataclasses import dataclass
from enum import IntEnum
from typing import TYPE_CHECKING, Any

import httpx
import msgspec
import numpy as np
import torch
import zmq
import zmq.asyncio

from vllm import envs
from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.utils import (
    EngineId,
    TransferTopology,
    get_current_attn_backend,
    get_current_attn_backends,
)
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorBase_V1,
    KVConnectorMetadata,
    KVConnectorRole,
    SupportsHMA,
)
from vllm.distributed.kv_transfer.kv_connector.v1.metrics import KVConnectorStats
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils import (
    MooncakeBootstrapServer,
    RegisterWorkerPayload,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats import (
    MooncakeKVConnectorStats,
)
from vllm.distributed.parallel_state import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    is_local_first_rank,
)
from vllm.forward_context import ForwardContext
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.math_utils import cdiv
from vllm.utils.network_utils import get_ip, make_zmq_path, make_zmq_socket
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.utils import get_kv_cache_layout
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.kv_cache_interface import FullAttentionSpec, SlidingWindowSpec
from vllm.v1.request import RequestStatus
from vllm.v1.worker.utils import select_common_block_size
```
**EN:** This block imports `asyncio`, `logging`, `threading`, `time`, `collections`, `concurrent.futures` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `asyncio`, `logging`, `threading`, `time`, `collections`, `concurrent.futures`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Guarded block / 保护代码块
```python
try:
    from mooncake.engine import TransferEngine
except ImportError:
    logger.warning(
        "Please install mooncake by following the instructions at "
        "https://github.com/kvcache-ai/Mooncake/blob/main/doc/en/build.md "
        "to run VLLM with MooncakeTransferEngine."
    )
    TransferEngine = None
```
**EN:** This guarded block attempts optional imports such as `mooncake.engine` and falls back when those dependencies are unavailable.
**CN:** 该保护代码块会尝试导入 `mooncake.engine` 等可选依赖，并在依赖不存在时回退。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.v1.core.kv_cache_manager import KVCacheBlocks
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.request import Request
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
ReqId = str  # Internal scheduler request ID
TransferId = str  # KV transfer coordination ID (shared by P/D)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `ReqId`, `TransferId`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `ReqId`, `TransferId`，供后续代码复用。

### Class `TransferRegion` / 类 `TransferRegion`
```python
@dataclass(frozen=True)
class TransferRegion:
    base_addr: int
    block_len: int
    kv_block_len: int
```
**EN:** Declares `TransferRegion`, a class.
**CN:** 声明 `TransferRegion`，它是一个类。

### Function `_get_tp_ratio` / 函数 `_get_tp_ratio`
```python
def _get_tp_ratio(local_tp_size: int, remote_tp_size: int) -> int:
    """Return the TP ratio used by heterogeneous TP transfer planning.

    Positive values mean one local rank maps into a larger remote KV region.
    Negative values mean one local rank must gather from multiple remote KV
    regions.
    """
    if local_tp_size >= remote_tp_size:
        assert local_tp_size % remote_tp_size == 0, (
            f"Local tensor parallel size {local_tp_size} is not divisible "
            f"by remote tensor parallel size {remote_tp_size}."
        )
        return local_tp_size // remote_tp_size

    assert remote_tp_size % local_tp_size == 0, (
        f"Remote tensor parallel size {remote_tp_size} is not divisible "
        f"by local tensor parallel size {local_tp_size}."
    )
    return -(remote_tp_size // local_tp_size)
```
**EN:** `_get_tp_ratio` implements a focused helper routine for this module. The docstring frames it as: Return the TP ratio used by heterogeneous TP transfer planning. It primarily works with arguments like `local_tp_size`, `remote_tp_size`.
**CN:** `_get_tp_ratio` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `local_tp_size`, `remote_tp_size` 这样的参数。

### Function `_expand_transfer_regions` / 函数 `_expand_transfer_regions`
```python
def _expand_transfer_regions(
    base_addrs: list[int],
    block_lens: list[int],
    is_kv_layout_blocks_first: bool,
) -> list[TransferRegion]:
    """Expand registered KV tensors into the regions transferred by Mooncake."""
    assert len(base_addrs) == len(block_lens), (
        "Mooncake transfer regions require matching numbers of base addresses "
        f"and block lengths, got {len(base_addrs)} and {len(block_lens)}."
    )
    regions: list[TransferRegion] = []
    for base_addr, block_len in zip(base_addrs, block_lens):
        kv_block_len = block_len // 2 if is_kv_layout_blocks_first else block_len
        regions.append(
            TransferRegion(
                base_addr=base_addr,
                block_len=block_len,
                kv_block_len=kv_block_len,
            )
        )
        if is_kv_layout_blocks_first:
            regions.append(
                TransferRegion(
                    base_addr=base_addr + kv_block_len,
                    block_len=block_len,
                    kv_block_len=kv_block_len,
                )
            )
    return regions
```
**EN:** `_expand_transfer_regions` implements a focused helper routine for this module. The docstring frames it as: Expand registered KV tensors into the regions transferred by Mooncake. It primarily works with arguments like `base_addrs`, `block_lens`, `is_kv_layout_blocks_first`. Key calls include `zip`, `len`, `regions.append`.
**CN:** `_expand_transfer_regions` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `base_addrs`, `block_lens`, `is_kv_layout_blocks_first` 这样的参数。 关键调用包括 `zip`, `len`, `regions.append`。

### Function `_compute_sender_transfer_plan` / 函数 `_compute_sender_transfer_plan`
```python
def _compute_sender_transfer_plan(
    local_tp_rank: int,
    local_tp_size: int,
    remote_tp_rank: int,
    remote_tp_size: int,
    local_kv_block_len: int,
    remote_kv_block_len: int,
    producer_cache_replicated: bool,
) -> tuple[bool, int, int, int]:
    """Plan one producer-rank to one consumer-rank copy for heterogeneous TP."""
    tp_ratio = _get_tp_ratio(local_tp_size, remote_tp_size)

    if tp_ratio == 1:
        return True, 0, 0, local_kv_block_len

    if tp_ratio > 0:
        if producer_cache_replicated:
            return local_tp_rank % tp_ratio == 0, 0, 0, local_kv_block_len
        return (
            True,
            0,
            (local_tp_rank % tp_ratio) * local_kv_block_len,
            local_kv_block_len,
        )

    if producer_cache_replicated:
        return True, 0, 0, local_kv_block_len

    ratio_abs = -tp_ratio
    return (
        True,
        (remote_tp_rank % ratio_abs) * remote_kv_block_len,
        0,
        remote_kv_block_len,
    )
```
**EN:** `_compute_sender_transfer_plan` implements a focused helper routine for this module. The docstring frames it as: Plan one producer-rank to one consumer-rank copy for heterogeneous TP. It primarily works with arguments like `local_tp_rank`, `local_tp_size`, `remote_tp_rank`, `remote_tp_size`. Key calls include `_get_tp_ratio`.
**CN:** `_compute_sender_transfer_plan` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `local_tp_rank`, `local_tp_size`, `remote_tp_rank`, `remote_tp_size` 这样的参数。 关键调用包括 `_get_tp_ratio`。

### Function `_can_coalesce_block_transfers` / 函数 `_can_coalesce_block_transfers`
```python
def _can_coalesce_block_transfers(
    local_region_block_len: int,
    remote_region_block_len: int,
    src_region_offset: int,
    dst_region_offset: int,
    transfer_len: int,
) -> bool:
    """Whether a contiguous block group can be emitted as one larger copy."""
    return (
        src_region_offset == 0
        and dst_region_offset == 0
        and transfer_len == local_region_block_len
        and transfer_len == remote_region_block_len
    )
```
**EN:** `_can_coalesce_block_transfers` implements a focused helper routine for this module. The docstring frames it as: Whether a contiguous block group can be emitted as one larger copy. It primarily works with arguments like `local_region_block_len`, `remote_region_block_len`, `src_region_offset`, `dst_region_offset`.
**CN:** `_can_coalesce_block_transfers` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `local_region_block_len`, `remote_region_block_len`, `src_region_offset`, `dst_region_offset` 这样的参数。

### Function `_validate_asymmetric_region_lengths` / 函数 `_validate_asymmetric_region_lengths`
```python
def _validate_asymmetric_region_lengths(
    local_regions: list[TransferRegion],
    remote_regions: list[TransferRegion],
    local_tp_size: int,
    remote_tp_size: int,
    producer_cache_replicated: bool,
) -> str | None:
    """Validate transfer-region metadata for a fixed producer/consumer pair.

    This checks registered KV regions, not per-request block counts. A region
    corresponds to one registered KV tensor, or one K/V half after expansion
    for layouts that store K and V together.
    """
    if len(local_regions) != len(remote_regions):
        return (
            "Mooncake asymmetric TP requires matching KV region counts between "
            "producer and consumer."
        )

    if producer_cache_replicated:
        return None

    tp_ratio = _get_tp_ratio(local_tp_size, remote_tp_size)
    for idx, (local_region, remote_region) in enumerate(
        zip(local_regions, remote_regions)
    ):
        if tp_ratio == 1:
            if local_region.kv_block_len != remote_region.kv_block_len:
                return (
                    "Mooncake KV region length mismatch for homogeneous TP at "
                    f"region {idx}: local={local_region.kv_block_len}, "
                    f"remote={remote_region.kv_block_len}."
                )
        elif tp_ratio > 0:
            if remote_region.kv_block_len != local_region.kv_block_len * tp_ratio:
                return (
                    "Mooncake destination KV region length does not match the "
                    "producer TP ratio at region "
                    f"{idx}: local={local_region.kv_block_len}, "
                    f"remote={remote_region.kv_block_len}, tp_ratio={tp_ratio}."
                )
        else:
            ratio_abs = -tp_ratio
            if local_region.kv_block_len != remote_region.kv_block_len * ratio_abs:
                return (
                    "Mooncake source KV region length does not match the "
                    "consumer TP ratio at region "
                    f"{idx}: local={local_region.kv_block_len}, "
                    f"remote={remote_region.kv_block_len}, tp_ratio={tp_ratio}."
                )

    return None
```
**EN:** `_validate_asymmetric_region_lengths` implements a focused helper routine for this module. The docstring frames it as: Validate transfer-region metadata for a fixed producer/consumer pair. It primarily works with arguments like `local_regions`, `remote_regions`, `local_tp_size`, `remote_tp_size`. Key calls include `_get_tp_ratio`, `enumerate`, `len`.
**CN:** `_validate_asymmetric_region_lengths` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `local_regions`, `remote_regions`, `local_tp_size`, `remote_tp_size` 这样的参数。 关键调用包括 `_get_tp_ratio`, `enumerate`, `len`。

### Function `_get_tensor_dense_flag` / 函数 `_get_tensor_dense_flag`
```python
def _get_tensor_dense_flag(tensor: torch.Tensor) -> bool | None:
    is_dense = getattr(tensor, "is_non_overlapping_and_dense", None)
    if callable(is_dense):
        return bool(is_dense())
    return None
```
**EN:** `_get_tensor_dense_flag` implements a focused helper routine for this module. It primarily works with arguments like `tensor`. Key calls include `getattr`, `callable`, `bool`.
**CN:** `_get_tensor_dense_flag` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `tensor` 这样的参数。 关键调用包括 `getattr`, `callable`, `bool`。

### Class `MooncakeXferMetadata` / 类 `MooncakeXferMetadata`
```python
class MooncakeXferMetadata(
    msgspec.Struct,
    omit_defaults=True,  # type: ignore[call-arg]
):
    remote_hostname: str
    remote_port: int
    remote_tp_size: int
    remote_tp_rank: int
    req_blocks: dict[ReqId, tuple[TransferId, list[list[int]]]]
    kv_caches_base_addr: list[int]
    block_lens: list[int]
```
**EN:** Declares `MooncakeXferMetadata`, a class derived from `msgspec.Struct`.
**CN:** 声明 `MooncakeXferMetadata`，它是一个类，继承自 `msgspec.Struct`。

### Class `MooncakeXferResponseStatus` / 类 `MooncakeXferResponseStatus`
```python
class MooncakeXferResponseStatus(IntEnum):
    # Transfer finished
    FINISH = 0
    # Continue to receive
    CONTINUE = 1
    # Something wrong, see err_msg
    ERROR = 2
```
**EN:** Declares `MooncakeXferResponseStatus`, a enum derived from `IntEnum`. It enumerates values such as `FINISH`, `CONTINUE`, `ERROR` to model roles or states.
**CN:** 声明 `MooncakeXferResponseStatus`，它是一个枚举，继承自 `IntEnum`。 它通过 `FINISH`, `CONTINUE`, `ERROR` 等枚举值表达角色或状态。

### Class `MooncakeXferResponse` / 类 `MooncakeXferResponse`
```python
class MooncakeXferResponse(
    msgspec.Struct,
    omit_defaults=True,  # type: ignore[call-arg]
):
    status: MooncakeXferResponseStatus
    ok_reqs: list[ReqId] | None = None
    err_reqs: list[ReqId] | None = None
    err_msg: str | None = None
```
**EN:** Declares `MooncakeXferResponse`, a class derived from `msgspec.Struct`.
**CN:** 声明 `MooncakeXferResponse`，它是一个类，继承自 `msgspec.Struct`。

### Class `PullReqMeta` / 类 `PullReqMeta`
```python
@dataclass
class PullReqMeta:
    d_req_id: ReqId
    transfer_id: TransferId
    local_block_ids: list[list[int]]
    remote_engine_id: EngineId
    remote_bootstrap_addr: str
    # Set expire time to avoid infinitely sending requests.
    expire_time: float = float("inf")
    # Designed for one D pairing to multiple P
    pull_tasks_count: int = 0
```
**EN:** Declares `PullReqMeta`, a dataclass. It packages structured data fields such as `d_req_id`, `transfer_id`, `local_block_ids`, `remote_engine_id`, `remote_bootstrap_addr`.
**CN:** 声明 `PullReqMeta`，它是一个数据类。 它封装了 `d_req_id`, `transfer_id`, `local_block_ids`, `remote_engine_id`, `remote_bootstrap_addr` 等结构化字段。

### Class `SendBlockMeta` / 类 `SendBlockMeta`
```python
@dataclass
class SendBlockMeta:
    p_req_id: ReqId
    transfer_id: TransferId
    local_block_ids: list[list[int]]
    ready: asyncio.Event
    expire_time: float = float("inf")
    need_send: int = 0
    sent: int = 0
    sending: int = 0
```
**EN:** Declares `SendBlockMeta`, a dataclass. It packages structured data fields such as `p_req_id`, `transfer_id`, `local_block_ids`, `ready`, `expire_time`.
**CN:** 声明 `SendBlockMeta`，它是一个数据类。 它封装了 `p_req_id`, `transfer_id`, `local_block_ids`, `ready`, `expire_time` 等结构化字段。

### Class `MooncakeConnectorMetadata` / 类 `MooncakeConnectorMetadata`
```python
class MooncakeConnectorMetadata(KVConnectorMetadata):
    def __init__(self):
        # Use (engine_id, dp_rank) to group reqs with same dp.
        # See comments in MooncakeBootstrapServer.
        self.reqs_to_recv: dict[EngineId, dict[ReqId, PullReqMeta]] = defaultdict(dict)
        self.reqs_to_send: dict[ReqId, tuple[TransferId, list[list[int]]]] = {}
        self.reqs_not_processed: set[TransferId] = set()

    def add_new_req(
        self,
        request_id: ReqId,
        local_block_ids: list[list[int]],
        kv_transfer_params: dict[str, Any],
        load_remote_cache: bool = True,
    ):
        transfer_id = kv_transfer_params["transfer_id"]
        if load_remote_cache:
            remote_engine_id = kv_transfer_params["remote_engine_id"]
            self.reqs_to_recv[remote_engine_id][request_id] = PullReqMeta(
                d_req_id=request_id,
                local_block_ids=local_block_ids,
                remote_engine_id=remote_engine_id,
                remote_bootstrap_addr=kv_transfer_params["remote_bootstrap_addr"],
                transfer_id=transfer_id,
            )
        else:
            self.reqs_to_send[request_id] = (transfer_id, local_block_ids)
```
**EN:** Declares `MooncakeConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `add_new_req`.
**CN:** 声明 `MooncakeConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `add_new_req`。

### Class `MooncakeConnector` / 类 `MooncakeConnector`
```python
class MooncakeConnector(KVConnectorBase_V1, SupportsHMA):
    def __init__(
        self,
        vllm_config: VllmConfig,
        role: KVConnectorRole,
        kv_cache_config: "KVCacheConfig",
    ):
        super().__init__(vllm_config, role, kv_cache_config)

        assert vllm_config.kv_transfer_config is not None
        assert vllm_config.kv_transfer_config.engine_id is not None
        self.engine_id: EngineId = vllm_config.kv_transfer_config.engine_id

        if role == KVConnectorRole.SCHEDULER:
            assert kv_cache_config is not None, (
                "kv_cache_config is required for SCHEDULER role"
            )
            self.connector_scheduler: MooncakeConnectorScheduler | None = (
                MooncakeConnectorScheduler(vllm_config, self.engine_id, kv_cache_config)
            )
            self.connector_worker: MooncakeConnectorWorker | None = None
        elif role == KVConnectorRole.WORKER:
            self.connector_scheduler = None
            self.connector_worker = MooncakeConnectorWorker(
                vllm_config, self.engine_id, kv_cache_config
            )

    @classmethod
    def get_required_kvcache_layout(cls, vllm_config: VllmConfig):
        if vllm_config.model_config is None:
            # This fallback mostly exists for unit tests that instantiate the
            # connector without a fully populated model config.
            logger.warning_once(
                "Unable to detect current VLLM config. "
                "Fallback to default kv cache layout."
            )
            return None
        if vllm_config.model_config.use_mla:
            return None
        logger.info_once(
            "MooncakeConnector setting KV cache layout to HND for "
            "heterogeneous TP-safe KV transfer."
        )
        return "HND"
# ... truncated for analysis ...
        """
        if self.connector_worker is None:
            return None
        return self.connector_worker.get_kv_connector_stats()

    @classmethod
    def build_kv_connector_stats(
        cls, data: dict[str, Any] | None = None
    ) -> KVConnectorStats | None:
        return MooncakeKVConnectorStats(data=data or {})
```
**EN:** Declares `MooncakeConnector`, a class derived from `KVConnectorBase_V1`, `SupportsHMA`. Key methods include `__init__`, `get_required_kvcache_layout`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`.
**CN:** 声明 `MooncakeConnector`，它是一个类，继承自 `KVConnectorBase_V1`, `SupportsHMA`。 关键方法包括 `__init__`, `get_required_kvcache_layout`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`。

### Class `MooncakeConnectorScheduler` / 类 `MooncakeConnectorScheduler`
```python
class MooncakeConnectorScheduler:
    """Implementation of Scheduler side methods"""

    def __init__(
        self,
        vllm_config: VllmConfig,
        engine_id: str,
        kv_cache_config: "KVCacheConfig",
    ):
        self.vllm_config = vllm_config
        self.block_size = vllm_config.cache_config.block_size

        assert vllm_config.kv_transfer_config
        self.is_kv_producer: bool = (
            vllm_config.kv_transfer_config.kv_role == "kv_producer"
        )
        self.is_kv_consumer: bool = (
            vllm_config.kv_transfer_config.kv_role == "kv_consumer"
        )
        logger.info("Initializing Mooncake Transfer Engine Scheduler %s", engine_id)

        self._is_hma_required = (
            not vllm_config.scheduler_config.disable_hybrid_kv_cache_manager
            and any(
                not isinstance(g.kv_cache_spec, FullAttentionSpec)
                for g in kv_cache_config.kv_cache_groups
            )
        )

        # Requests that need to start recv/send.
        # New requests are added by update_state_after_alloc in
        # the scheduler. Used to make metadata passed to Worker.
        self._reqs_need_recv: dict[ReqId, tuple[Request, list[list[int]]]] = {}
        self._reqs_need_send: dict[ReqId, tuple[Request, list[list[int]]]] = {}
        # Reqs to remove from processed set because they're not to send after
        # remote prefill or aborted.
        self._reqs_not_processed: set[TransferId] = set()

        # Compute sliding window block counts per KV cache group.
        sw_sizes_tokens: list[tuple[int, int]] = [
            (g.kv_cache_spec.sliding_window, g.kv_cache_spec.block_size)
            if isinstance(g.kv_cache_spec, SlidingWindowSpec)
            else (0, self.block_size)
            for g in kv_cache_config.kv_cache_groups
        ]
# ... truncated for analysis ...
        # remove the conditional below
        delay_free_blocks = any(len(group) > 0 for group in block_ids)

        if delay_free_blocks:
            self._reqs_need_send[request.request_id] = (
                request,
                self.get_sw_clipped_blocks(block_ids),
            )

        return delay_free_blocks, None
```
**EN:** Declares `MooncakeConnectorScheduler`, a class. Key methods include `__init__`, `get_sw_clipped_blocks`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`. The docstring summarizes its role as: Implementation of Scheduler side methods.
**CN:** 声明 `MooncakeConnectorScheduler`，它是一个类。 关键方法包括 `__init__`, `get_sw_clipped_blocks`, `get_num_new_matched_tokens`, `update_state_after_alloc`, `build_connector_meta`。 文档字符串概括了它在整体流程中的职责。

### Class `MooncakeConnectorWorker` / 类 `MooncakeConnectorWorker`
```python
class MooncakeConnectorWorker:
    """Implementation of Worker side methods"""

    def __init__(
        self,
        vllm_config: VllmConfig,
        engine_id: str,
        kv_cache_config: "KVCacheConfig | None" = None,
    ):
        if TransferEngine is None:
            logger.error("Mooncake is not available")
            raise RuntimeError("Mooncake is not available")
        logger.info("Initializing Mooncake Transfer Engine worker %s", engine_id)

        self.vllm_config = vllm_config
        # Capture device BEFORE TransferEngine init — MNNVL's NVLink allocator
        # may change the current CUDA device during engine.initialize().
        self.device_id = torch.accelerator.current_device_index()
        current_platform.set_device(self.device_id)

        self.engine = TransferEngine()
        self.hostname = get_ip()

        assert (kv_transfer_config := vllm_config.kv_transfer_config)
        self.is_kv_producer: bool = kv_transfer_config.kv_role == "kv_producer"
        self.is_kv_consumer: bool = kv_transfer_config.kv_role == "kv_consumer"
        self.num_sender_workers = kv_transfer_config.kv_connector_extra_config.get(
            "num_workers", 10
        )
        # Create more tasks than workers to keep the thread pool saturated.
        # Tasks can await async events, so a surplus (2x is a robust heuristic)
        # prevents workers from idling.
        self.num_sender_tasks = self.num_sender_workers * 2
        protocol = kv_transfer_config.kv_connector_extra_config.get(  # type: ignore[union-attr]
            "mooncake_protocol", "rdma"
        )
        logger.info(
            "The Mooncake Transfer Engine is using %s as its protocol.", protocol
        )
        ret_value = self.engine.initialize(self.hostname, "P2PHANDSHAKE", protocol, "")
        if ret_value != 0:
            raise RuntimeError("Mooncake Transfer Engine initialization failed.")

        self.rpc_port = self.engine.get_rpc_port()
# ... truncated for analysis ...
            "Mooncake register view layer=%s shape=%s stride=%s "
            "storage_offset=%d contiguous=%s dense=%s data_ptr=%d",
            layer_name,
            tuple(cache.shape),
            tuple(cache.stride()),
            cache.storage_offset(),
            cache.is_contiguous(),
            _get_tensor_dense_flag(cache),
            cache.data_ptr(),
        )
```
**EN:** Declares `MooncakeConnectorWorker`, a class. Key methods include `__init__`, `_sync_block_size_with_kernel`, `__del__`, `shutdown`, `register_worker_with_bootstrap`. The docstring summarizes its role as: Implementation of Worker side methods.
**CN:** 声明 `MooncakeConnectorWorker`，它是一个类。 关键方法包括 `__init__`, `_sync_block_size_with_kernel`, `__del__`, `shutdown`, `register_worker_with_bootstrap`。 文档字符串概括了它在整体流程中的职责。

### Function `group_concurrent_contiguous` / 函数 `group_concurrent_contiguous`
```python
def group_concurrent_contiguous(
    src_indices: list[int], dst_indices: list[int]
) -> tuple[list[list[int]], list[list[int]]]:
    """Vectorised NumPy implementation."""
    if len(src_indices) == 0:
        return [], []

    brk = np.where((np.diff(src_indices) != 1) | (np.diff(dst_indices) != 1))[0] + 1
    src_groups = np.split(src_indices, brk)
    dst_groups = np.split(dst_indices, brk)

    src_groups = [g.tolist() for g in src_groups]
    dst_groups = [g.tolist() for g in dst_groups]

    return src_groups, dst_groups
```
**EN:** `group_concurrent_contiguous` implements a focused helper routine for this module. The docstring frames it as: Vectorised NumPy implementation. It primarily works with arguments like `src_indices`, `dst_indices`. Key calls include `np.split`, `len`, `g.tolist`.
**CN:** `group_concurrent_contiguous` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `src_indices`, `dst_indices` 这样的参数。 关键调用包括 `np.split`, `len`, `g.tolist`。

### Function `get_mooncake_side_channel_port` / 函数 `get_mooncake_side_channel_port`
```python
def get_mooncake_side_channel_port(vllm_config: VllmConfig) -> int:
    # This logic is now centralized
    return (
        envs.VLLM_MOONCAKE_BOOTSTRAP_PORT
        + vllm_config.parallel_config.data_parallel_index
        * vllm_config.parallel_config.tensor_parallel_size
    )
```
**EN:** `get_mooncake_side_channel_port` retrieves state or computed results for this module. It primarily works with arguments like `vllm_config`.
**CN:** `get_mooncake_side_channel_port` 负责获取状态或计算结果。 它主要处理诸如 `vllm_config` 这样的参数。

### Function `_async_loop` / 函数 `_async_loop`
```python
def _async_loop(loop: asyncio.AbstractEventLoop):
    asyncio.set_event_loop(loop)
    loop.run_forever()
```
**EN:** `_async_loop` implements a focused helper routine for this module. It primarily works with arguments like `loop`. Key calls include `asyncio.set_event_loop`, `loop.run_forever`.
**CN:** `_async_loop` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `loop` 这样的参数。 关键调用包括 `asyncio.set_event_loop`, `loop.run_forever`。

### Function `should_launch_bootstrap_server` / 函数 `should_launch_bootstrap_server`
```python
def should_launch_bootstrap_server(vllm_config: VllmConfig) -> bool:
    assert (parallel_config := vllm_config.parallel_config)
    # In hybrid or external LB mode,
    # each instance should have its own bootstrap server.
    #
    # In internal LB mode,
    # only the real global first rank need to launch the bootstrap server.
    return is_local_first_rank() and (
        parallel_config.local_engines_only or parallel_config.data_parallel_index == 0
    )
```
**EN:** `should_launch_bootstrap_server` implements a focused helper routine for this module. It primarily works with arguments like `vllm_config`. Key calls include `is_local_first_rank`.
**CN:** `should_launch_bootstrap_server` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `vllm_config` 这样的参数。 关键调用包括 `is_local_first_rank`。

### Function `get_mooncake_bootstrap_addr` / 函数 `get_mooncake_bootstrap_addr`
```python
def get_mooncake_bootstrap_addr(vllm_config: VllmConfig) -> tuple[str, int]:
    """
    Returns the address of the Mooncake bootstrap server.
    This is only used by prefillers to register workers.
    Decoders should get addr from kv_transfer_params.
    """
    assert (parallel_config := vllm_config.parallel_config)
    if parallel_config.local_engines_only:
        # In hybrid or external LB mode, connect to local server.
        host = "127.0.0.1"
    else:
        host = parallel_config.data_parallel_master_ip
    port = envs.VLLM_MOONCAKE_BOOTSTRAP_PORT
    return (host, port)
```
**EN:** `get_mooncake_bootstrap_addr` retrieves state or computed results for this module. The docstring frames it as: Returns the address of the Mooncake bootstrap server. This is only used by prefillers to register workers. Decoders should get addr from.... It primarily works with arguments like `vllm_config`.
**CN:** `get_mooncake_bootstrap_addr` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `vllm_config` 这样的参数。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `TransferRegion`: class interface or data carrier / `TransferRegion`：类接口或数据载体
- `_get_tp_ratio`: module-level helper or API entry / `_get_tp_ratio`：模块级辅助函数或 API 入口
- `_expand_transfer_regions`: module-level helper or API entry / `_expand_transfer_regions`：模块级辅助函数或 API 入口
- `_compute_sender_transfer_plan`: module-level helper or API entry / `_compute_sender_transfer_plan`：模块级辅助函数或 API 入口
- `_can_coalesce_block_transfers`: module-level helper or API entry / `_can_coalesce_block_transfers`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `logging`, `threading`, `time`, `collections`, `concurrent.futures`, `dataclasses`, `enum`, `typing`
- **Third-party / 第三方**: `httpx`, `msgspec`, `numpy`, `torch`, `zmq`, `zmq.asyncio`, `mooncake.engine`
- **Internal modules / 内部模块**: `vllm`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.stats`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.logger`, `vllm.platforms`, `vllm.utils.math_utils`
