# scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/offloading/scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from collections.abc import Iterable, Sequence
from dataclasses import dataclass, field
from itertools import islice
from typing import Any, NamedTuple

from vllm.distributed.kv_events import BlockRemoved, BlockStored, KVCacheEvent
from vllm.distributed.kv_transfer.kv_connector.utils import yield_req_data
from vllm.distributed.kv_transfer.kv_connector.v1.base import KVConnectorMetadata
from vllm.distributed.kv_transfer.kv_connector.v1.offloading.common import (
    OffloadingConnectorMetadata,
    OffloadingWorkerMetadata,
    ReqId,
    TransferJob,
)
from vllm.logger import init_logger
from vllm.utils.math_utils import cdiv
from vllm.v1.core.kv_cache_manager import KVCacheBlocks
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheSpec,
    MambaSpec,
    SlidingWindowSpec,
)
from vllm.v1.kv_offload.base import (
    GPULoadStoreSpec,
    OffloadingManager,
    OffloadingSpec,
    OffloadKey,
    ReqContext,
    get_offload_block_hash,
    make_offload_key,
)
from vllm.v1.outputs import KVConnectorOutput
from vllm.v1.request import Request
```
**EN:** This block imports `collections.abc`, `dataclasses`, `itertools`, `typing`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.utils` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `collections.abc`, `dataclasses`, `itertools`, `typing`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.utils`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `TransferJobStatus` / 类 `TransferJobStatus`
```python
@dataclass(slots=True)
class TransferJobStatus:
    """Tracks scheduler-side state for a single transfer job."""

    req_id: ReqId
    # Number of workers still pending. Starts at num_workers,
    # decremented as each worker reports completion. Job is done at 0.
    pending_count: int
    # Offload keys this job covers; passed to manager.complete_*().
    keys: set[OffloadKey]
    is_store: bool
    # Store src block IDs whose ref_cnt protects them while the request
    # runs. Only registered in _block_id_to_pending_jobs on request_finished.
    non_sliding_window_block_ids: list[int] | None = None
    # Store src block IDs that may be freed before the request finishes.
    # Registered in _block_id_to_pending_jobs at store creation time.
    sliding_window_block_ids: list[int] | None = None
```
**EN:** Declares `TransferJobStatus`, a class. The docstring summarizes its role as: Tracks scheduler-side state for a single transfer job.
**CN:** 声明 `TransferJobStatus`，它是一个类。 文档字符串概括了它在整体流程中的职责。

### Class `GroupOffloadConfig` / 类 `GroupOffloadConfig`
```python
class GroupOffloadConfig(NamedTuple):
    group_idx: int
    gpu_block_size: int
    offloaded_block_size: int
    hash_block_size_factor: int
    # None below means full attention
    sliding_window_size_in_blocks: int | None
```
**EN:** Declares `GroupOffloadConfig`, a class derived from `NamedTuple`.
**CN:** 声明 `GroupOffloadConfig`，它是一个类，继承自 `NamedTuple`。

### Function `get_sliding_window_size_in_blocks` / 函数 `get_sliding_window_size_in_blocks`
```python
def get_sliding_window_size_in_blocks(
    kv_cache_spec: KVCacheSpec, offloaded_block_size: int
) -> int | None:
    if isinstance(kv_cache_spec, SlidingWindowSpec):
        assert kv_cache_spec.sliding_window > 0
        return cdiv(kv_cache_spec.sliding_window, offloaded_block_size)

    if isinstance(kv_cache_spec, MambaSpec):
        # Mamba depends on a single state
        return 1

    assert isinstance(kv_cache_spec, FullAttentionSpec)
    return None
```
**EN:** `get_sliding_window_size_in_blocks` retrieves state or computed results for this module. It primarily works with arguments like `kv_cache_spec`, `offloaded_block_size`. Key calls include `isinstance`, `cdiv`.
**CN:** `get_sliding_window_size_in_blocks` 负责获取状态或计算结果。 它主要处理诸如 `kv_cache_spec`, `offloaded_block_size` 这样的参数。 关键调用包括 `isinstance`, `cdiv`。

### Class `SchedulerOffloadConfig` / 类 `SchedulerOffloadConfig`
```python
class SchedulerOffloadConfig(NamedTuple):
    kv_group_configs: tuple[GroupOffloadConfig, ...]
    block_size_factor: int
    num_workers: int

    @classmethod
    def from_spec(cls, spec: OffloadingSpec) -> "SchedulerOffloadConfig":
        return cls(
            num_workers=spec.vllm_config.parallel_config.world_size,
            kv_group_configs=tuple(
                GroupOffloadConfig(
                    group_idx=idx,
                    gpu_block_size=gpu_block_size,
                    offloaded_block_size=gpu_block_size * spec.block_size_factor,
                    hash_block_size_factor=(
                        (gpu_block_size * spec.block_size_factor)
                        // spec.hash_block_size
                    ),
                    sliding_window_size_in_blocks=get_sliding_window_size_in_blocks(
                        spec.kv_cache_config.kv_cache_groups[idx].kv_cache_spec,
                        gpu_block_size * spec.block_size_factor,
                    ),
                )
                for idx, gpu_block_size in enumerate(spec.gpu_block_size)
            ),
            block_size_factor=spec.block_size_factor,
        )
```
**EN:** Declares `SchedulerOffloadConfig`, a class derived from `NamedTuple`. Key methods include `from_spec`.
**CN:** 声明 `SchedulerOffloadConfig`，它是一个类，继承自 `NamedTuple`。 关键方法包括 `from_spec`。

### Class `RequestGroupState` / 类 `RequestGroupState`
```python
@dataclass
class RequestGroupState:
    offload_keys: list[OffloadKey] = field(default_factory=list)
    block_ids: list[int] = field(default_factory=list)
    # index of next block (of size offloaded_block_size) to offload
    next_stored_block_idx: int = 0
    # number of offloaded blocks hit (including GPU prefix cache)
    # when the request first started
    num_hit_blocks: int = 0
```
**EN:** Declares `RequestGroupState`, a dataclass. It packages structured data fields such as `offload_keys`, `block_ids`, `next_stored_block_idx`, `num_hit_blocks`.
**CN:** 声明 `RequestGroupState`，它是一个数据类。 它封装了 `offload_keys`, `block_ids`, `next_stored_block_idx`, `num_hit_blocks` 等结构化字段。

### Class `RequestOffloadState` / 类 `RequestOffloadState`
```python
@dataclass(slots=True)
class RequestOffloadState:
    config: SchedulerOffloadConfig
    req: Request
    group_states: tuple[RequestGroupState, ...] = field(init=False)
    req_context: ReqContext = field(init=False)
    # number of hits in the GPU cache
    num_locally_computed_tokens: int = 0
    # In-flight job IDs. Per the connector's invariant, at any given time
    # this contains either a single load job, or one or more store jobs.
    transfer_jobs: set[int] = field(default_factory=set)

    def __post_init__(self) -> None:
        self.group_states = tuple(
            RequestGroupState() for _ in self.config.kv_group_configs
        )
        self.req_context = ReqContext(
            req_id=self.req.request_id,
            kv_transfer_params=self.req.kv_transfer_params,
        )

    def update_offload_keys(self) -> None:
        for group_config, group_state in zip(
            self.config.kv_group_configs, self.group_states
        ):
            for req_block_hash in islice(
                self.req.block_hashes,
                group_config.hash_block_size_factor * len(group_state.offload_keys)
                + group_config.hash_block_size_factor
                - 1,
                None,
                group_config.hash_block_size_factor,
            ):
                group_state.offload_keys.append(
                    make_offload_key(req_block_hash, group_config.group_idx)
                )

    def update_block_id_groups(
        self, new_block_id_groups: tuple[list[int], ...] | None
    ) -> None:
        if new_block_id_groups is None:
            return

        assert len(new_block_id_groups) == len(self.group_states)
        for group_state, new_blocks in zip(self.group_states, new_block_id_groups):
# ... truncated for analysis ...
            num_blocks = num_offloadable_tokens // group_config.offloaded_block_size
            group_state.next_stored_block_idx = num_blocks

    def update_num_hit_blocks(self, num_cached_tokens: int) -> None:
        for group_config, group_state in zip(
            self.config.kv_group_configs, self.group_states
        ):
            group_state.num_hit_blocks = (
                num_cached_tokens // group_config.offloaded_block_size
            )
```
**EN:** Declares `RequestOffloadState`, a class. Key methods include `__post_init__`, `update_offload_keys`, `update_block_id_groups`, `advance_stored_idx`, `update_num_hit_blocks`.
**CN:** 声明 `RequestOffloadState`，它是一个类。 关键方法包括 `__post_init__`, `update_offload_keys`, `update_block_id_groups`, `advance_stored_idx`, `update_num_hit_blocks`。

### Class `OffloadingConnectorScheduler` / 类 `OffloadingConnectorScheduler`
```python
class OffloadingConnectorScheduler:
    """Implementation of Scheduler side methods"""

    def __init__(self, spec: OffloadingSpec):
        self.config = SchedulerOffloadConfig.from_spec(spec)
        self.manager: OffloadingManager = spec.get_manager()

        full_attention_groups: list[int] = []
        sliding_window_groups: list[int] = []
        for group_config in self.config.kv_group_configs:
            if group_config.sliding_window_size_in_blocks is None:
                full_attention_groups.append(group_config.group_idx)
            else:
                sliding_window_groups.append(group_config.group_idx)

        # sort sliding window groups by window size in decreasing order
        def _sliding_window_sort_key(i: int) -> int:
            val = self.config.kv_group_configs[i].sliding_window_size_in_blocks
            assert val is not None
            return val

        sliding_window_groups.sort(key=_sliding_window_sort_key, reverse=True)

        # used by _lookup
        self._sliding_window_groups: tuple[int, ...] = tuple(sliding_window_groups)
        self._lookup_groups = tuple(full_attention_groups) + self._sliding_window_groups

        self._req_status: dict[ReqId, RequestOffloadState] = {}
        self._current_batch_load_jobs: dict[int, TransferJob] = {}
        self._current_batch_jobs_to_flush: set[int] = set()
        # if GPU prefix caching is enabled,
        # track loaded blocks to avoid redundant loads
        self._blocks_being_loaded: set[OffloadKey] | None = (
            set() if spec.vllm_config.cache_config.enable_prefix_caching else None
        )

        # Job ID counter shared by loads and stores.
        self._job_counter: int = 0
        # Threshold value for stale jobs. All job ids >= _stale_job_threshold are
        # active jobs.
        self._stale_job_threshold: int = 0
        self._jobs: dict[int, TransferJobStatus] = {}

        # block_id -> pending store job_ids. Used to track jobs that needs
        # flushing in case a block is re-allocated by the KV cache manager.
# ... truncated for analysis ...
        self._jobs.clear()
        self._block_id_to_pending_jobs.clear()

        # Note: _current_batch_jobs_to_flush is intentionally NOT cleared.
        # The load flush IDs collected above must be delivered to workers.
        if self._blocks_being_loaded is not None:
            self._blocks_being_loaded.clear()

    def shutdown(self) -> None:
        self.manager.shutdown()
```
**EN:** Declares `OffloadingConnectorScheduler`, a class. Key methods include `__init__`, `_generate_job_id`, `_remove_pending_job`, `_maximal_prefix_lookup`, `_sliding_window_lookup`. The docstring summarizes its role as: Implementation of Scheduler side methods.
**CN:** 声明 `OffloadingConnectorScheduler`，它是一个类。 关键方法包括 `__init__`, `_generate_job_id`, `_remove_pending_job`, `_maximal_prefix_lookup`, `_sliding_window_lookup`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `TransferJobStatus`: class interface or data carrier / `TransferJobStatus`：类接口或数据载体
- `GroupOffloadConfig`: class interface or data carrier / `GroupOffloadConfig`：类接口或数据载体
- `get_sliding_window_size_in_blocks`: module-level helper or API entry / `get_sliding_window_size_in_blocks`：模块级辅助函数或 API 入口
- `SchedulerOffloadConfig`: class interface or data carrier / `SchedulerOffloadConfig`：类接口或数据载体
- `RequestGroupState`: dataclass interface or data carrier / `RequestGroupState`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`, `itertools`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.distributed.kv_transfer.kv_connector.v1.offloading.common`, `vllm.logger`, `vllm.utils.math_utils`, `vllm.v1.core.kv_cache_manager`, `vllm.v1.core.sched.output`, `vllm.v1.kv_cache_interface`, `vllm.v1.kv_offload.base`, `vllm.v1.outputs`, `vllm.v1.request`
