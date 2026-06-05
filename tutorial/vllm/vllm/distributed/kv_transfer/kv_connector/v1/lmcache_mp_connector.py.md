# lmcache_mp_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/lmcache_mp_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import enum
import os
from collections.abc import Iterable
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any, Literal

import torch
import zmq
from lmcache.integration.vllm.utils import mla_enabled
from lmcache.utils import init_logger as lmcache_init_logger

from vllm.config import VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorBase_V1,
    KVConnectorMetadata,
    KVConnectorRole,
)
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.outputs import KVConnectorOutput
from vllm.v1.request import RequestStatus
from vllm.v1.utils import ConstantList
```
**EN:** This block imports `enum`, `os`, `collections.abc`, `dataclasses`, `typing`, `torch` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `enum`, `os`, `collections.abc`, `dataclasses`, `typing`, `torch`，为后续实现准备运行时、类型与辅助 API。

### Guarded block / 保护代码块
```python
try:
    from lmcache.integration.vllm.vllm_multi_process_adapter import (
        LMCacheMPSchedulerAdapter,
        LMCacheMPWorkerAdapter,
        LoadStoreOp,
        ParallelStrategy,
    )

    try:
        from lmcache.v1.multiprocess.custom_types import RequestAllocationRecord
    except ImportError:
        from lmcache.v1.multiprocess.custom_types import (
            BlockAllocationRecord as RequestAllocationRecord,
        )
except ImportError:
    from lmcache.v1.multiprocess.custom_types import (
        BlockAllocationRecord as RequestAllocationRecord,
    )

    from vllm.distributed.kv_transfer.kv_connector.v1.lmcache_integration import (
        LMCacheMPSchedulerAdapter,
        LMCacheMPWorkerAdapter,
        LoadStoreOp,
        ParallelStrategy,
    )
```
**EN:** This guarded block attempts optional imports such as `lmcache.integration.vllm.vllm_multi_process_adapter`, `lmcache.v1.multiprocess.custom_types`, `vllm.distributed.kv_transfer.kv_connector.v1.lmcache_integration` and falls back when those dependencies are unavailable.
**CN:** 该保护代码块会尝试导入 `lmcache.integration.vllm.vllm_multi_process_adapter`, `lmcache.v1.multiprocess.custom_types`, `vllm.distributed.kv_transfer.kv_connector.v1.lmcache_integration` 等可选依赖，并在依赖不存在时回退。

### Conditional block / 条件代码块
```python
if TYPE_CHECKING:
    from vllm.distributed.kv_events import KVCacheEvent
    from vllm.distributed.kv_transfer.kv_connector.v1.metrics import (
        KVConnectorPromMetrics,
        KVConnectorStats,
        PromMetric,
        PromMetricT,
    )
    from vllm.forward_context import ForwardContext
    from vllm.v1.core.kv_cache_manager import KVCacheBlocks
    from vllm.v1.core.kv_cache_utils import BlockHash
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.request import Request
```
**EN:** This conditional keeps imports and declarations that are only needed for static type checking out of the runtime path.
**CN:** 该条件块将仅用于静态类型检查的导入与声明隔离在运行时路径之外。

### Module constants / 模块常量
```python
logger = lmcache_init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Function `reformat_block_ids` / 函数 `reformat_block_ids`
```python
def reformat_block_ids(block_ids: tuple[list[int], ...] | None) -> list[int]:
    if block_ids is None:
        return []
    assert isinstance(block_ids, tuple), (
        f"Expected block_ids to be a tuple of lists, but got {type(block_ids)}"
    )

    if len(block_ids) > 1:
        raise RuntimeError(
            "LMCacheMPConnector only works without hybrid kv cache manager. "
            "Please pass --disable-hybrid-kv-cache-manager when starting vllm"
        )

    return block_ids[0]
```
**EN:** `reformat_block_ids` implements a focused helper routine for this module. It primarily works with arguments like `block_ids`. Key calls include `isinstance`, `len`, `RuntimeError`.
**CN:** `reformat_block_ids` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `block_ids` 这样的参数。 关键调用包括 `isinstance`, `len`, `RuntimeError`。

### Function `extract_world_size_and_kv_rank` / 函数 `extract_world_size_and_kv_rank`
```python
def extract_world_size_and_kv_rank(
    world_size: int,
    rank: int,
    vllm_config: VllmConfig,
) -> tuple[int, int]:
    """
    Convert the rank for the MLA.
    """
    use_mla = mla_enabled(vllm_config.model_config)
    if not use_mla:
        return world_size, rank
    else:
        # Tensor parallel does not change the KV caches for MLA models.
        # So we need to "exclude" the effect of TP on rank and world size
        tp_size = vllm_config.parallel_config.tensor_parallel_size
        # vLLM constructs TP groups first, and then construct other
        # parallel groups on top of TP groups.
        # for example, TP=4, PP=2,
        # PP group: [0, 1, 2, 3], [4, 5, 6, 7]
        # TP group: [0, 4], [1, 5], [2, 6], [3, 7]
        # So we can "exclude" the effect of TP by rank // tp_size.
        return world_size // tp_size, rank // tp_size
```
**EN:** `extract_world_size_and_kv_rank` implements a focused helper routine for this module. The docstring frames it as: Convert the rank for the MLA. It primarily works with arguments like `world_size`, `rank`, `vllm_config`. Key calls include `mla_enabled`.
**CN:** `extract_world_size_and_kv_rank` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `world_size`, `rank`, `vllm_config` 这样的参数。 关键调用包括 `mla_enabled`。

### Function `create_scheduler_adapter` / 函数 `create_scheduler_adapter`
```python
def create_scheduler_adapter(
    server_url: str,
    zmq_context: zmq.Context,
    vllm_config: VllmConfig,
    mq_timeout: float,
    heartbeat_interval: float,
) -> LMCacheMPSchedulerAdapter:
    world_size, kv_rank = extract_world_size_and_kv_rank(
        vllm_config.parallel_config.world_size,
        vllm_config.parallel_config.rank,
        vllm_config,
    )
    parallel_strategy = ParallelStrategy(
        mla_enabled(vllm_config.model_config),
        world_size,
        kv_rank,
        vllm_config.parallel_config.world_size,
        vllm_config.parallel_config.rank,
        vllm_config.parallel_config.tensor_parallel_size,
        vllm_config.parallel_config.pipeline_parallel_size,
    )

    return LMCacheMPSchedulerAdapter(
        server_url=server_url,
        context=zmq_context,
        model_name=vllm_config.model_config.model,
        vllm_block_size=vllm_config.cache_config.block_size,
        parallel_strategy=parallel_strategy,
        mq_timeout=mq_timeout,
        heartbeat_interval=heartbeat_interval,
    )
```
**EN:** `create_scheduler_adapter` constructs the required object or payload for this module. It primarily works with arguments like `server_url`, `zmq_context`, `vllm_config`, `mq_timeout`. Key calls include `extract_world_size_and_kv_rank`, `ParallelStrategy`, `LMCacheMPSchedulerAdapter`.
**CN:** `create_scheduler_adapter` 负责构建所需对象或负载。 它主要处理诸如 `server_url`, `zmq_context`, `vllm_config`, `mq_timeout` 这样的参数。 关键调用包括 `extract_world_size_and_kv_rank`, `ParallelStrategy`, `LMCacheMPSchedulerAdapter`。

### Function `create_worker_adapter` / 函数 `create_worker_adapter`
```python
def create_worker_adapter(
    server_url: str,
    zmq_context: zmq.Context,
    vllm_config: VllmConfig,
    mq_timeout: float,
    heartbeat_interval: float,
) -> LMCacheMPWorkerAdapter:
    world_size, kv_rank = extract_world_size_and_kv_rank(
        vllm_config.parallel_config.world_size,
        vllm_config.parallel_config.rank,
        vllm_config,
    )
    parallel_strategy = ParallelStrategy(
        mla_enabled(vllm_config.model_config),
        world_size,
        kv_rank,
        vllm_config.parallel_config.world_size,
        vllm_config.parallel_config.rank,
        vllm_config.parallel_config.tensor_parallel_size,
        vllm_config.parallel_config.pipeline_parallel_size,
    )

    return LMCacheMPWorkerAdapter(
        server_url=server_url,
        context=zmq_context,
        model_name=vllm_config.model_config.model,
        vllm_block_size=vllm_config.cache_config.block_size,
        parallel_strategy=parallel_strategy,
        mq_timeout=mq_timeout,
        heartbeat_interval=heartbeat_interval,
    )
```
**EN:** `create_worker_adapter` constructs the required object or payload for this module. It primarily works with arguments like `server_url`, `zmq_context`, `vllm_config`, `mq_timeout`. Key calls include `extract_world_size_and_kv_rank`, `ParallelStrategy`, `LMCacheMPWorkerAdapter`.
**CN:** `create_worker_adapter` 负责构建所需对象或负载。 它主要处理诸如 `server_url`, `zmq_context`, `vllm_config`, `mq_timeout` 这样的参数。 关键调用包括 `extract_world_size_and_kv_rank`, `ParallelStrategy`, `LMCacheMPWorkerAdapter`。

### Class `LMCacheMPRequestState` / 类 `LMCacheMPRequestState`
```python
class LMCacheMPRequestState(enum.Enum):
    """
    State machine:
    PREFETCHING -- update_state_after_alloc --> WAITING_FOR_LOAD
    WAITING_FOR_LOAD -- process_loading_requests --> READY
    """

    PREFETCHING = enum.auto()
    WAITING_FOR_LOAD = enum.auto()
    READY = enum.auto()
```
**EN:** Declares `LMCacheMPRequestState`, a enum derived from `enum.Enum`. It enumerates values such as `PREFETCHING`, `WAITING_FOR_LOAD`, `READY` to model roles or states. The docstring summarizes its role as: State machine: PREFETCHING -- update_state_after_alloc --> WAITING_FOR_LOAD WAITING_FOR_LOAD -- process_loading_requests --> READY.
**CN:** 声明 `LMCacheMPRequestState`，它是一个枚举，继承自 `enum.Enum`。 它通过 `PREFETCHING`, `WAITING_FOR_LOAD`, `READY` 等枚举值表达角色或状态。 文档字符串概括了它在整体流程中的职责。

### Class `LMCacheMPRequestTracker` / 类 `LMCacheMPRequestTracker`
```python
@dataclass
class LMCacheMPRequestTracker:
    # NOTE: this class used vLLM data structures, should be part of
    # vLLM integration code

    request_id: str

    # Read-only lists to track the token ids and block hashes
    all_token_ids: ConstantList[int]
    block_hashes: ConstantList["BlockHash"]

    # Block ids and hashes will be updated at update_states_after_alloc and
    # during the generation
    allocated_block_ids: list[int] = field(default_factory=list)

    # Number of scheduled tokens in this request. We keep tracking this to
    # avoid saving half-full blocks.
    num_scheduled_tokens: int = 0

    # Number of blocks stored will be initialized when lookup the external
    # hit tokens and will be updated when processing new requests and cached
    # requests.
    num_stored_blocks: int = 0

    # Staging load operation -- save vllm and lmcache hit tokens during lookup
    num_vllm_hit_blocks: int = 0
    num_lmcache_hit_blocks: int = 0

    # Main state
    state: LMCacheMPRequestState = LMCacheMPRequestState.PREFETCHING

    cache_salt: str = ""

    def __init__(self, request: "Request"):
        self.request_id = request.request_id
        self.cache_salt: str = request.cache_salt or ""
        self.all_token_ids = request.all_token_ids
        self.block_hashes = ConstantList(request.block_hashes)
        self.allocated_block_ids = []
        self.num_stored_blocks = 0
        self.num_vllm_hit_blocks = 0
        self.num_lmcache_hit_blocks = 0
        self.state = LMCacheMPRequestState.PREFETCHING

    ####
# ... truncated for analysis ...
            f"num_block_hashes={len(self.block_hashes)}, "
            f"num_allocated_blocks={len(self.allocated_block_ids)}, "
            f"num_stored_blocks={self.num_stored_blocks}, "
            f"vllm_hit_blocks={self.num_vllm_hit_blocks}, "
            f"lmcache_hit_blocks={self.num_lmcache_hit_blocks}, "
            f"state={self.state})"
        )

    def __str__(self) -> str:
        return self.__repr__()
```
**EN:** Declares `LMCacheMPRequestTracker`, a dataclass. It packages structured data fields such as `request_id`, `all_token_ids`, `block_hashes`, `allocated_block_ids`, `num_scheduled_tokens`.
**CN:** 声明 `LMCacheMPRequestTracker`，它是一个数据类。 它封装了 `request_id`, `all_token_ids`, `block_hashes`, `allocated_block_ids`, `num_scheduled_tokens` 等结构化字段。

### Class `LMCacheMPRequestMetadata` / 类 `LMCacheMPRequestMetadata`
```python
@dataclass
class LMCacheMPRequestMetadata:
    request_id: str
    direction: Literal["STORE", "RETRIEVE"]
    op: LoadStoreOp
    cache_salt: str = ""

    @staticmethod
    def GetStoreMetadata(
        tracker: LMCacheMPRequestTracker,
        blocks_in_chunk: int,
        vllm_block_size: int,
    ) -> "LMCacheMPRequestMetadata | None":
        """
        Generate the store metadata for the current request tracker.

        Args:
            tracker: The request tracker to generate the metadata from.
            blocks_in_chunk: the number of blocks in a LMCache data chunk
            vllm_block_size: the block size used in vLLM
        """
        # Store the blocks that has block hashes
        # NOTE: the invariant here is that `num_stored_blocks` should
        # always be a multiple of `blocks_in_chunk`
        # TODO: This should be checked everytime we update the num_stored_blocks
        #
        # Why computed_blocks uses max(num_vllm_hit_blocks, num_lmcache_hit_blocks):
        #
        # Both values represent a prefix of blocks whose KV data is already
        # available (either from vLLM APC or from LMCache), so they must NOT
        # be summed (that would double-count the overlapping prefix).
        #
        # * num_lmcache_hit_blocks: LMCache-hit blocks are already counted in
        #   num_stored_blocks (set during lookup), so they must be included
        #   here to keep the upper bound consistent.  They are NOT re-stored.
        # * num_vllm_hit_blocks: LMCache stores in units of chunks (N blocks),
        #   so num_lmcache_hit_blocks is rounded DOWN to the nearest chunk
        #   boundary.  When vLLM APC hits more blocks than that rounded value
        #   (e.g. APC=44 blocks, LMCache=32 blocks after chunk alignment),
        #   using only num_lmcache_hit_blocks would set the upper bound too
        #   low and silently skip the APC-hit blocks that fall between the
        #   two values, causing under-storing.  Taking the max ensures we
        #   always use the tighter (larger) of the two hit counts.
        computed_blocks = tracker.num_scheduled_tokens // vllm_block_size + max(
            tracker.num_vllm_hit_blocks, tracker.num_lmcache_hit_blocks
# ... truncated for analysis ...

            ret = LMCacheMPRequestMetadata(
                request_id=tracker.request_id,
                direction="RETRIEVE",
                op=op,
                cache_salt=tracker.cache_salt,
            )
            return ret

        return None
```
**EN:** Declares `LMCacheMPRequestMetadata`, a dataclass. It packages structured data fields such as `request_id`, `direction`, `op`, `cache_salt`.
**CN:** 声明 `LMCacheMPRequestMetadata`，它是一个数据类。 它封装了 `request_id`, `direction`, `op`, `cache_salt` 等结构化字段。

### Class `LMCacheMPConnectorMetadata` / 类 `LMCacheMPConnectorMetadata`
```python
class LMCacheMPConnectorMetadata(KVConnectorMetadata):
    def __init__(self):
        super().__init__()
        self.requests: list[LMCacheMPRequestMetadata] = []

    def add_request_metadata(self, request_metadata: LMCacheMPRequestMetadata):
        self.requests.append(request_metadata)

    def __len__(self):
        return len(self.requests)

    # For debugging
    def __str__(self):
        request_strs = []
        for req_meta in self.requests:
            request_strs.append(
                f"RequestMetadata(request_id={req_meta.request_id}, "
                f"direction={req_meta.direction}, "
                f"num_blocks={len(req_meta.op)}, "
                f"block_ids={req_meta.op.block_ids})"
            )
        return "[" + "\n".join(request_strs) + "]"

    def __repr__(self):
        return self.__str__()
```
**EN:** Declares `LMCacheMPConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `add_request_metadata`, `__len__`, `__str__`, `__repr__`.
**CN:** 声明 `LMCacheMPConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `add_request_metadata`, `__len__`, `__str__`, `__repr__`。

### Class `LMCacheMPConnectorUpstream` / 类 `LMCacheMPConnectorUpstream`
```python
class LMCacheMPConnectorUpstream(KVConnectorBase_V1):
    """
    The connector for LMCache multi-process mode.

    Extra configs (kv_transfer_config.extra_config):
    - lmcache.mp.host: the host of the LMCache server.
    - lmcache.mp.port: the port of the LMCache server.
    - lmcache.mp.mq_timeout: timeout (seconds) for message queue requests.
    - lmcache.mp.heartbeat_interval: interval (seconds) between server
      heartbeat pings.
    """

    def __init__(
        self,
        vllm_config: "VllmConfig",
        role: KVConnectorRole,
        kv_cache_config: "KVCacheConfig",
    ):
        super().__init__(vllm_config, role, kv_cache_config)

        assert vllm_config.kv_transfer_config is not None
        server_host = vllm_config.kv_transfer_config.get_from_extra_config(
            "lmcache.mp.host", "tcp://localhost"
        )
        server_port = vllm_config.kv_transfer_config.get_from_extra_config(
            "lmcache.mp.port", 5555
        )
        mq_timeout = float(
            vllm_config.kv_transfer_config.get_from_extra_config(
                "lmcache.mp.mq_timeout", 300.0
            )
        )
        heartbeat_interval = float(
            vllm_config.kv_transfer_config.get_from_extra_config(
                "lmcache.mp.heartbeat_interval", 10.0
            )
        )

        server_url = f"{server_host}:{server_port}"
        zmq_context = zmq.Context.instance()
        if self.role == KVConnectorRole.SCHEDULER:
            self.scheduler_adapter = create_scheduler_adapter(
                server_url,
                zmq_context,
                vllm_config,
# ... truncated for analysis ...
        """
        Clean up request tracker and associated lookup future for a request.
        This should be called when a request is finished to prevent memory leak.
        """
        # Clean up request tracker
        if self.request_trackers.pop(request_id, None):
            logger.debug(
                "[KVConnector] Cleaned up request_tracker for request %s",
                request_id,
            )
```
**EN:** Declares `LMCacheMPConnectorUpstream`, a class derived from `KVConnectorBase_V1`. Key methods include `__init__`, `role`, `_get_connector_metadata`, `register_kv_caches`, `start_load_kv`. The docstring summarizes its role as: The connector for LMCache multi-process mode.
**CN:** 声明 `LMCacheMPConnectorUpstream`，它是一个类，继承自 `KVConnectorBase_V1`。 关键方法包括 `__init__`, `role`, `_get_connector_metadata`, `register_kv_caches`, `start_load_kv`。 文档字符串概括了它在整体流程中的职责。

### Function `_resolve_lmcache_mp_connector` / 函数 `_resolve_lmcache_mp_connector`
```python
def _resolve_lmcache_mp_connector() -> type[KVConnectorBase_V1]:
    if os.environ.get("LMCACHE_USE_UPSTREAM_MP"):
        logger.info(
            "Force use builtin LMCacheMPConnectorUpstream in vLLM.",
        )
        return LMCacheMPConnectorUpstream

    try:
        from lmcache.integration.vllm.lmcache_mp_connector import (
            LMCacheMPConnector as _ExternalLMCacheMPConnector,
        )

        logger.info(
            "Using external LMCacheMPConnector from "
            "lmcache.integration.vllm.lmcache_mp_connector"
        )
        return _ExternalLMCacheMPConnector
    except ImportError as e:
        logger.info(
            "External LMCacheMPConnector is not available (%s), "
            "falling back to builtin implementation in vLLM.",
            e,
        )
        return LMCacheMPConnectorUpstream
```
**EN:** `_resolve_lmcache_mp_connector` implements a focused helper routine for this module. Key calls include `os.environ.get`, `logger.info`.
**CN:** `_resolve_lmcache_mp_connector` 实现了一个面向当前模块的辅助例程。 关键调用包括 `os.environ.get`, `logger.info`。

### Module constants / 模块常量
```python
LMCacheMPConnector = _resolve_lmcache_mp_connector()
```
**EN:** This section defines module-level aliases, constants, or shared state such as `LMCacheMPConnector`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `LMCacheMPConnector`，供后续代码复用。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `reformat_block_ids`: module-level helper or API entry / `reformat_block_ids`：模块级辅助函数或 API 入口
- `extract_world_size_and_kv_rank`: module-level helper or API entry / `extract_world_size_and_kv_rank`：模块级辅助函数或 API 入口
- `create_scheduler_adapter`: module-level helper or API entry / `create_scheduler_adapter`：模块级辅助函数或 API 入口
- `create_worker_adapter`: module-level helper or API entry / `create_worker_adapter`：模块级辅助函数或 API 入口
- `LMCacheMPRequestState`: enum interface or data carrier / `LMCacheMPRequestState`：枚举接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`, `os`, `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`, `zmq`, `lmcache.integration.vllm.utils`, `lmcache.utils`, `lmcache.integration.vllm.vllm_multi_process_adapter`, `lmcache.v1.multiprocess.custom_types`, `lmcache.integration.vllm.lmcache_mp_connector`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.attention.backend`, `vllm.v1.core.sched.output`, `vllm.v1.outputs`, `vllm.v1.request`, `vllm.v1.utils`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.forward_context`, `vllm.v1.core.kv_cache_manager`, `vllm.v1.core.kv_cache_utils`
