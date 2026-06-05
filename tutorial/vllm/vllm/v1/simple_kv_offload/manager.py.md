# manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/simple_kv_offload/manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Scheduler-side manager for SimpleCPUOffloadConnector. / 该模块位于 `simple_kv_offload` 子系统，主要围绕 `TransferMeta`, `LoadRequestState`, `StoreRequestState` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Scheduler-side manager for SimpleCPUOffloadConnector."""

import contextlib
from collections.abc import Iterable
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any

from vllm.config import VllmConfig
from vllm.distributed.kv_events import KVCacheEvent
from vllm.distributed.kv_transfer.kv_connector.utils import yield_req_data
from vllm.logger import init_logger
from vllm.utils.math_utils import cdiv
from vllm.v1.core.block_pool import BlockPool
from vllm.v1.core.kv_cache_coordinator import (
    KVCacheCoordinator,
    get_kv_cache_coordinator,
)
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    MambaSpec,
    SlidingWindowSpec,
)
from vllm.v1.outputs import KVConnectorOutput
from vllm.v1.simple_kv_offload.metadata import (
    SimpleCPUOffloadMetadata,
    SimpleCPUOffloadWorkerMetadata,
)

if TYPE_CHECKING:
    from vllm.v1.core.kv_cache_manager import KVCacheBlocks
    from vllm.v1.core.kv_cache_utils import KVCacheBlock
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.request import Request

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `TransferMeta` class / `TransferMeta` 类
```python
@dataclass
class TransferMeta:
    gpu_block_ids: list[int]
    cpu_block_ids: list[int]
```
**EN:** Uses `@dataclass` to package related state for `TransferMeta`. Typical fields include `gpu_block_ids`, `cpu_block_ids`.
**CN:** `TransferMeta` 使用 `@dataclass` 打包相关状态。典型字段包括 `gpu_block_ids`, `cpu_block_ids`。

### `LoadRequestState` class / `LoadRequestState` 类
```python
@dataclass
class LoadRequestState:
    request: "Request"
    transfer_meta: TransferMeta
    load_event: int | None = None
    finished: bool = False
```
**EN:** Uses `@dataclass` to package related state for `LoadRequestState`. Typical fields include `request`, `transfer_meta`, `load_event`, `finished`.
**CN:** `LoadRequestState` 使用 `@dataclass` 打包相关状态。典型字段包括 `request`, `transfer_meta`, `load_event`, `finished`。

### `StoreRequestState` class / `StoreRequestState` 类
```python
@dataclass
class StoreRequestState:
    request: "Request"
    # Accumulated block IDs from scheduler_output via yield_req_data.
    block_ids: tuple[list[int], ...]
    # Per-group cursors tracking how many blocks have been stored/skipped.
    num_stored_blocks: list[int]
    store_events: set[int] = field(default_factory=set)
    finished: bool = False
```
**EN:** Uses `@dataclass` to package related state for `StoreRequestState`. Typical fields include `request`, `block_ids`, `num_stored_blocks`, `store_events`, `finished`.
**CN:** `StoreRequestState` 使用 `@dataclass` 打包相关状态。典型字段包括 `request`, `block_ids`, `num_stored_blocks`, `store_events`, `finished`。

### `SimpleCPUOffloadScheduler` class / `SimpleCPUOffloadScheduler` 类
```python
class SimpleCPUOffloadScheduler:
    """Scheduler-side manager for CPU offloading."""
```
**EN:** Introduces the `SimpleCPUOffloadScheduler` class. Core methods include `__init__`, `_derive_cpu_config`, `_estimate_lazy_target_blocks`, `bind_gpu_block_pool`, `get_num_new_matched_tokens`, `update_state_after_alloc`. Docstring signal: Scheduler-side manager for CPU offloading.
**CN:** 这里定义 `SimpleCPUOffloadScheduler` 类。核心方法包括 `__init__`, `_derive_cpu_config`, `_estimate_lazy_target_blocks`, `bind_gpu_block_pool`, `get_num_new_matched_tokens`, `update_state_after_alloc`。

### `SimpleCPUOffloadScheduler.__init__` method / `SimpleCPUOffloadScheduler.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        kv_cache_config: "KVCacheConfig | None",
        cpu_capacity_bytes: int,
        lazy_offload: bool = False,
    ):
        self.vllm_config = vllm_config
        self.kv_cache_config = kv_cache_config
        self.enable_kv_cache_events = (
            vllm_config.kv_events_config is not None
            and vllm_config.kv_events_config.enable_kv_cache_events
        )
        # NOTE: We use the same block size for both GPU and CPU.
        self.block_size = vllm_config.cache_config.block_size
        # Derive a CPU KVCacheConfig from the GPU config and build a coordinator
        assert kv_cache_config is not None
        self.cpu_kv_cache_config = self._derive_cpu_config(
            kv_cache_config, cpu_capacity_bytes
        )
        self.num_cpu_blocks = self.cpu_kv_cache_config.num_blocks
        # Find the full attention kv group for prefix cache matching.
        self.fa_gidx = -1
        for g_idx, g in enumerate(self.cpu_kv_cache_config.kv_cache_groups):
            if isinstance(g.kv_cache_spec, FullAttentionSpec):
                self.fa_gidx = g_idx
                break
        assert 0 <= self.fa_gidx < len(self.cpu_kv_cache_config.kv_cache_groups)

        logger.info(
            "SimpleCPUOffloadScheduler: Allocating %d CPU blocks (%.2f GB, mode=%s)",
            self.num_cpu_blocks,
            cpu_capacity_bytes / (1024**3),
            "lazy" if lazy_offload else "eager",
        )

        # TODO (yifan): maybe need to enable kv_cache_events and metrics_collector here.
        dcp_world_size = vllm_config.parallel_config.decode_context_parallel_size
        pcp_world_size = vllm_config.parallel_config.prefill_context_parallel_size
        assert dcp_world_size == 1 and pcp_world_size == 1
        self.cpu_coordinator: KVCacheCoordinator = get_kv_cache_coordinator(
            kv_cache_config=self.cpu_kv_cache_config,
            max_model_len=vllm_config.model_config.max_model_len,
            max_num_batched_tokens=(
                vllm_config.scheduler_config.max_num_batched_tokens
            ),
            use_eagle=False,
            enable_caching=True,
            enable_kv_cache_events=self.enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=self.block_size,
        )
        self.cpu_block_pool: BlockPool = self.cpu_coordinator.block_pool

        # GPU block pool reference - bound after scheduler builds kv_cache_manager
        self._gpu_block_pool: BlockPool | None = None

        # Load metadata
        self._reqs_to_load: dict[str, LoadRequestState] = {}
        # Inverse map: load_event_idx -> req_ids. Keyed by load_event_idx because
        # the worker reports completions by event index, not request id.
        self._load_event_to_reqs: dict[int, list[str]] = {}

        # Store metadata
        self._lazy_mode = lazy_offload
        # Lazy mode: use a cursor to track the last scanned block in the GPU free queue.
        self._cursor: KVCacheBlock | None = None
        if self._lazy_mode:
            self._target_free = self._estimate_lazy_target_blocks(
                kv_cache_config,
                vllm_config.scheduler_config.max_num_batched_tokens,
            )
        else:
            self._target_free = 0
        self._store_event_to_blocks: dict[int, TransferMeta] = {}
        # Eager mode only
        self._reqs_to_store: dict[str, StoreRequestState] = {}
        self._store_event_to_reqs: dict[int, list[str]] = {}
        self._in_flight_store_gpu_blocks: set[int] = set()

        # Event counters
        self._load_event_counter: int = 0
        self._store_event_counter: int = 0

        # For TP/PP: track partial store completions across steps.
        # Events must be reported by all world_size workers before considered complete.
        self._expected_worker_count = vllm_config.parallel_config.world_size
        self._store_event_pending_counts: dict[int, int] = {}
```
**EN:** This method initializes the object state within `SimpleCPUOffloadScheduler`. Key calls include `_derive_cpu_config`, `enumerate`, `info`, `get_kv_cache_coordinator`, `set`, `isinstance`. It touches state such as `vllm_config`, `kv_cache_config`, `enable_kv_cache_events`, `block_size`, `cpu_kv_cache_config`, `num_cpu_blocks`, `fa_gidx`, `cpu_coordinator`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SimpleCPUOffloadScheduler`。 关键调用包括 `_derive_cpu_config`, `enumerate`, `info`, `get_kv_cache_coordinator`, `set`, `isinstance`。 它会读写 `vllm_config`, `kv_cache_config`, `enable_kv_cache_events`, `block_size`, `cpu_kv_cache_config`, `num_cpu_blocks`, `fa_gidx`, `cpu_coordinator` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SimpleCPUOffloadScheduler.update_state_after_alloc` method / `SimpleCPUOffloadScheduler.update_state_after_alloc` 方法
```python
    def update_state_after_alloc(
        self,
        request: "Request",
        blocks: "KVCacheBlocks",
        num_external_tokens: int,
    ) -> None:
        req_id = request.request_id
        block_ids_by_group = blocks.get_block_ids()
        num_groups = len(block_ids_by_group)

        # Store tracking (eager mode only). Register the request;
        # block IDs are accumulated from scheduler_output in
        # _prepare_eager_store_specs via yield_req_data.
        if not self._lazy_mode and req_id not in self._reqs_to_store:
            self._reqs_to_store[req_id] = StoreRequestState(
                request=request,
                block_ids=tuple([] for _ in range(num_groups)),
                num_stored_blocks=[0] * num_groups,
            )

        if num_external_tokens == 0:
            return

        num_blocks_to_load = num_external_tokens // self.block_size
        assert num_blocks_to_load > 0

        skipped = sum(blk.block_hash is not None for blk in blocks.blocks[self.fa_gidx])
        num_computed_tokens = skipped * self.block_size
        hashes_to_load = request.block_hashes[skipped : skipped + num_blocks_to_load]

        # Find CPU cached blocks across all groups.
        max_hit_len = len(hashes_to_load) * self.block_size
        cpu_hit_blocks, hit_length = self.cpu_coordinator.find_longest_cache_hit(
            hashes_to_load, max_hit_len
        )
        assert hit_length == num_external_tokens, (
            f"Expected {num_external_tokens} hit tokens, got {hit_length}"
        )

        # Build transfer pairs across all groups.
        total_computed_tokens = num_computed_tokens + num_external_tokens
        kv_cache_groups = self.cpu_kv_cache_config.kv_cache_groups

        gpu_block_ids: list[int] = []
        cpu_block_ids: list[int] = []
        cpu_blocks_to_touch: list[KVCacheBlock] = []

        for g in range(num_groups):
            cpu_blocks_g = cpu_hit_blocks[g]
            n_ext_g = len(cpu_blocks_g)
            if n_ext_g == 0:
                continue

            # Number of blocks in the computed range for this group.
            g_block_size = kv_cache_groups[g].kv_cache_spec.block_size
            n_computed_g = cdiv(total_computed_tokens, g_block_size)

            # Back-trace: ext blocks sit at the tail of the computed range.
            gpu_ext_start = n_computed_g - n_ext_g
            group_gpu_ids = block_ids_by_group[g]

            for i, cpu_blk in enumerate(cpu_blocks_g):
                # Skip null blocks (e.g. sliding window or mamba padding).
                if cpu_blk.is_null:
                    continue
                gpu_block_ids.append(group_gpu_ids[gpu_ext_start + i])
                cpu_block_ids.append(cpu_blk.block_id)
                cpu_blocks_to_touch.append(cpu_blk)

        # Touch CPU blocks to prevent eviction during async load.
        self.cpu_block_pool.touch(cpu_blocks_to_touch)

        # Touch GPU blocks to prevent freeing during async load
        assert self._gpu_block_pool is not None
        self._gpu_block_pool.touch(
            [self._gpu_block_pool.blocks[bid] for bid in gpu_block_ids]
        )

        assert self._reqs_to_load.get(req_id) is None
        self._reqs_to_load[req_id] = LoadRequestState(
            request=request, transfer_meta=TransferMeta(gpu_block_ids, cpu_block_ids)
        )
```
**EN:** This method updates existing state within `SimpleCPUOffloadScheduler`. Key calls include `get_block_ids`, `len`, `sum`, `find_longest_cache_hit`, `range`, `touch`. It touches state such as `_reqs_to_load`, `_reqs_to_store`. The control flow contains 4 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`SimpleCPUOffloadScheduler`。 关键调用包括 `get_block_ids`, `len`, `sum`, `find_longest_cache_hit`, `range`, `touch`。 它会读写 `_reqs_to_load`, `_reqs_to_store` 等状态。 控制流包含 4 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `SimpleCPUOffloadScheduler._prepare_eager_store_specs` method / `SimpleCPUOffloadScheduler._prepare_eager_store_specs` 方法
```python
    def _prepare_eager_store_specs(
        self, scheduler_output: SchedulerOutput
    ) -> tuple[list[int], list[int], list[str]]:
        """Identify newly computed blocks to offload from scheduler requests.

        Only considers blocks whose KV data has been **confirmed computed** by
        the GPU. This means blocks from the current step are NOT stored until the
        next step. If a request finishes in the same step as its last full block,
        that block may be missed. (TODO: flush on finish.)

        Returns:
            (gpu_block_ids, cpu_block_ids, req_ids) for the store event.
        """

        merged_gpu_block_ids: list[int] = []
        merged_cpu_block_ids: list[int] = []
        req_ids: list[str] = []

        gpu_block_pool = self._gpu_block_pool
        if gpu_block_pool is None:
            return [], [], []
        cpu_block_pool = self.cpu_block_pool
        num_free = cpu_block_pool.get_num_free_blocks()
        kv_cache_groups = self.cpu_kv_cache_config.kv_cache_groups
        num_groups = len(kv_cache_groups)
        # Dedup against blocks already scheduled.
        in_flight = self._in_flight_store_gpu_blocks

        for req_id, new_block_id_groups, preempted in yield_req_data(scheduler_output):
            state = self._reqs_to_store.get(req_id)
            if state is None or state.finished:
                continue

            # Accumulate new block IDs.
            if preempted:
                state.block_ids = tuple([] for _ in range(num_groups))
                state.num_stored_blocks = [0] * num_groups
            if new_block_id_groups:
                for g in range(min(num_groups, len(new_block_id_groups))):
                    if new_block_id_groups[g] is not None:
                        state.block_ids[g].extend(new_block_id_groups[g])

            num_new_tokens = scheduler_output.num_scheduled_tokens.get(req_id, 0)
            if num_new_tokens == 0:
                continue

            block_ids_by_group = state.block_ids
            if not block_ids_by_group:
                continue

    # ... omitted for brevity ...
                cpu_block_ids = []

            if cpu_block_ids:
                req_ids.append(req_id)
                merged_gpu_block_ids.extend(gpu_block_ids)
                merged_cpu_block_ids.extend(cpu_block_ids)
                in_flight.update(gpu_block_ids)

                # Touch GPU blocks to prevent freeing during async copy
                gpu_block_pool.touch(
                    [gpu_block_pool.blocks[bid] for bid in gpu_block_ids]
                )

                logger.debug(
                    "Request %s: Scheduling store of %d blocks to CPU (%d groups)",
                    req_id,
                    len(cpu_block_ids),
                    num_groups,
                )

            # Advance per-group cursors (includes cached hits + newly stored)
            for g in range(num_groups):
                state.num_stored_blocks[g] += advanced_per_group[g]

        return merged_gpu_block_ids, merged_cpu_block_ids, req_ids
```
**EN:** This method implements `_prepare_eager_store_specs` within `SimpleCPUOffloadScheduler`. The docstring frames it as: Identify newly computed blocks to offload from scheduler requests. Key calls include `get_num_free_blocks`, `len`, `yield_req_data`, `get`, `range`, `tuple`. The control flow contains 14 branch(es) and 6 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_prepare_eager_store_specs`，其作用域位于`SimpleCPUOffloadScheduler`。 关键调用包括 `get_num_free_blocks`, `len`, `yield_req_data`, `get`, `range`, `tuple`。 控制流包含 14 个分支和 6 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `TransferMeta`: central class or interface in this module. / `TransferMeta`：本模块中的核心类或接口。
- `LoadRequestState`: central class or interface in this module. / `LoadRequestState`：本模块中的核心类或接口。
- `StoreRequestState`: central class or interface in this module. / `StoreRequestState`：本模块中的核心类或接口。
- `SimpleCPUOffloadScheduler`: central class or interface in this module. / `SimpleCPUOffloadScheduler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`, `collections`, `dataclasses`, `typing`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.logger`, `vllm.utils.math_utils`, `vllm.v1.core.block_pool`, `vllm.v1.core.kv_cache_coordinator`, `vllm.v1.core.sched.output`, `vllm.v1.kv_cache_interface`, `vllm.v1.outputs`, `vllm.v1.simple_kv_offload.metadata`, `vllm.v1.core.kv_cache_manager`
