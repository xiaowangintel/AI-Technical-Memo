# manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/tiering/manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TieringOffloadingManager: Multi-tier KV cache offloading orchestrator. / 该模块位于 `kv_offload/tiering` 子系统，主要围绕 `PendingPromotion`, `CPUPrimaryTierOffloadingManager`, `TieringOffloadingManager` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
TieringOffloadingManager: Multi-tier KV cache offloading orchestrator.

This manager coordinates between a CPU primary tier (with direct GPU access)
and zero or more secondary tiers (Storage, Network, etc.) to provide
hierarchical KV cache offloading.

Key Design Principles:
1. Always offload to all tiers — When a block is stored to the primary tier,
   it is cascaded to ALL secondary tiers
2. Primary tier is the gateway — Secondary tiers cannot access GPU memory
   directly; all data flows through the CPU primary tier
3. Staged promotion — Blocks in secondary tiers must be promoted to the
   primary tier before GPU can access them
4. Transparent retry mechanism — Return None from lookup() to signal
   "data is being promoted, try later"
5. ref_cnt as eviction protection — primary.prepare_read() increments ref_cnt,
   protecting blocks from eviction until complete_read() is called
"""

from collections.abc import Collection, Iterable
from dataclasses import dataclass, field

import numpy as np

from vllm.logger import init_logger
from vllm.v1.kv_offload.base import (
    LoadStoreSpec,
    OffloadingEvent,
    OffloadingManager,
    OffloadKey,
    PrepareStoreOutput,
    ReqContext,
)
from vllm.v1.kv_offload.cpu.common import CPULoadStoreSpec
from vllm.v1.kv_offload.cpu.manager import CPUOffloadingManager
from vllm.v1.kv_offload.cpu.shared_offload_region import SharedOffloadRegion
from vllm.v1.kv_offload.tiering.base import (
    JobId,
    JobMetadata,
    SecondaryTierManager,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `PendingPromotion` class / `PendingPromotion` 类
```python
@dataclass
class PendingPromotion:
    """Accumulator for blocks awaiting submit_load() for one (tier, request)."""

    req_context: ReqContext
    keys: list[OffloadKey] = field(default_factory=list)
    block_ids: list[int] = field(default_factory=list)
```
**EN:** Uses `@dataclass` to package related state for `PendingPromotion`. Typical fields include `req_context`, `keys`, `block_ids`.
**CN:** `PendingPromotion` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_context`, `keys`, `block_ids`。

### `CPUPrimaryTierOffloadingManager` class / `CPUPrimaryTierOffloadingManager` 类
```python
class CPUPrimaryTierOffloadingManager(CPUOffloadingManager):
    """CPUOffloadingManager with a primary/secondary transfer interface.

    The inherited prepare_store/complete_store/prepare_load/complete_load are the
    GPU-facing OffloadingManager interface. These aliases expose the same operations
    from the secondary tier perspective, where read/write refers to secondary
    accessing primary. This avoids confusion when reading TieringOffloadingManager
    code (e.g. calling prepare_load inside a cascade/store path would be misleading).
    """
```
**EN:** Introduces the `CPUPrimaryTierOffloadingManager` class on top of `CPUOffloadingManager`. Core methods include `__init__`, `get_kv_memoryview`, `shutdown`. Docstring signal: CPUOffloadingManager with a primary/secondary transfer interface.
**CN:** 这里定义 `CPUPrimaryTierOffloadingManager` 类，其基类包括 `CPUOffloadingManager`。核心方法包括 `__init__`, `get_kv_memoryview`, `shutdown`。

### `CPUPrimaryTierOffloadingManager.__init__` method / `CPUPrimaryTierOffloadingManager.__init__` 方法
```python
    def __init__(
        self,
        num_blocks: int,
        mmap_region: SharedOffloadRegion,
        cache_policy: str = "lru",
        enable_events: bool = False,
    ):
        super().__init__(
            num_blocks=num_blocks,
            cache_policy=cache_policy,  # type: ignore[arg-type]
            enable_events=enable_events,
        )
        self._mmap_region = mmap_region
        # read/write is for CPU<->secondary transfers,
        # load/store is for CPU<->GPU transfers.
        # These aliases avoid calling prepare_load inside a store path.
        self.prepare_read = self.prepare_load
        self.complete_read = self.complete_load
        self.prepare_write = self.prepare_store
        self.complete_write = self.complete_store

        self._kv_memoryview = mmap_region.create_kv_memoryview()
```
**EN:** This method initializes the object state within `CPUPrimaryTierOffloadingManager`. Key calls include `__init__`, `create_kv_memoryview`, `super`. It touches state such as `_mmap_region`, `prepare_read`, `complete_read`, `prepare_write`, `complete_write`, `_kv_memoryview`.
**CN:** 该方法会初始化对象状态，其作用域位于`CPUPrimaryTierOffloadingManager`。 关键调用包括 `__init__`, `create_kv_memoryview`, `super`。 它会读写 `_mmap_region`, `prepare_read`, `complete_read`, `prepare_write`, `complete_write`, `_kv_memoryview` 等状态。

### `CPUPrimaryTierOffloadingManager.get_kv_memoryview` method / `CPUPrimaryTierOffloadingManager.get_kv_memoryview` 方法
```python
    def get_kv_memoryview(self) -> memoryview:
        """Return the memoryview over the primary tier's KV cache buffer.

        The view has shape (num_blocks, row_stride_bytes) and is backed by the
        SharedOffloadRegion mmap.  Secondary tiers address block *b* as
        ``view[b]``.
        """
        return self._kv_memoryview
```
**EN:** This method returns or derives a value within `CPUPrimaryTierOffloadingManager`. The docstring frames it as: Return the memoryview over the primary tier's KV cache buffer.
**CN:** 该方法会返回或推导一个值，其作用域位于`CPUPrimaryTierOffloadingManager`。

### `CPUPrimaryTierOffloadingManager.shutdown` method / `CPUPrimaryTierOffloadingManager.shutdown` 方法
```python
    def shutdown(self) -> None:
        super().shutdown()
        self._kv_memoryview.release()
        self._mmap_region.cleanup()
```
**EN:** This method implements `shutdown` within `CPUPrimaryTierOffloadingManager`. Key calls include `shutdown`, `release`, `cleanup`, `super`.
**CN:** 该方法会实现 `shutdown`，其作用域位于`CPUPrimaryTierOffloadingManager`。 关键调用包括 `shutdown`, `release`, `cleanup`, `super`。

### `TieringOffloadingManager` class / `TieringOffloadingManager` 类
```python
class TieringOffloadingManager(OffloadingManager):
    """
    Orchestrates multi-tier KV cache offloading.

    This manager coordinates between a CPU primary tier (with direct GPU access)
    and zero or more secondary tiers (Storage, Network, etc.) to provide
    hierarchical KV cache offloading.

    Key internal state:
      - Minimal state tracking; relies on secondary tiers to report completion
        via get_finished()
      - Secondary tiers return JobResult objects containing all necessary
        information
      - job_id_counter: monotonically increasing counter for job IDs
    """
```
**EN:** Introduces the `TieringOffloadingManager` class on top of `OffloadingManager`. Core methods include `__init__`, `_next_job_id`, `_maybe_process_finished_jobs`, `_process_finished_jobs`, `lookup`, `_initiate_promotion`. Docstring signal: Orchestrates multi-tier KV cache offloading.
**CN:** 这里定义 `TieringOffloadingManager` 类，其基类包括 `OffloadingManager`。核心方法包括 `__init__`, `_next_job_id`, `_maybe_process_finished_jobs`, `_process_finished_jobs`, `lookup`, `_initiate_promotion`。

### `TieringOffloadingManager.__init__` method / `TieringOffloadingManager.__init__` 方法
```python
    def __init__(
        self,
        primary_tier: CPUPrimaryTierOffloadingManager,
        secondary_tiers: list[SecondaryTierManager] | None = None,
        enable_events: bool = False,
    ):
        """
        Initialize the TieringOffloadingManager.

        Args:
            primary_tier: The primary tier manager (CPU-based).
            secondary_tiers: List of secondary tier managers (e.g., Storage,
                            Network). Can be None or empty list.
            enable_events: Whether to track offloading events
        """
        self.primary_tier: CPUPrimaryTierOffloadingManager = primary_tier
        self.secondary_tiers = secondary_tiers or []

        self._job_id_counter: int = 0
        self.events: list[OffloadingEvent] | None = [] if enable_events else None

        # Job tracking: maps job_id to metadata for all in-flight transfers.
        # JobMetadata.is_promotion distinguishes direction:
        #   True:  secondary → primary (promotion)
        #   False: primary → secondary (cascade)
        self._transfer_jobs: dict[JobId, JobMetadata] = {}

        # Pending promotion requests accumulated during lookup() calls; flushed
        # as one batched submit_load() per (tier, request) in take_events().
        # Outer key: tier. Inner key: req_context.req_id — the same ReqContext
        # object is reused for all block lookups of a given request per engine step.
        self._pending_load_submissions: dict[
            SecondaryTierManager, dict[str, PendingPromotion]
        ] = {}

        # Gate for once-per-step execution of _maybe_process_finished_jobs().
        # Reset at the end of each step in take_events().
        self._processed_jobs_this_step: bool = False
```
**EN:** This method initializes the object state within `TieringOffloadingManager`. The docstring frames it as: Initialize the TieringOffloadingManager. It touches state such as `primary_tier`, `secondary_tiers`, `_job_id_counter`, `events`, `_transfer_jobs`, `_pending_load_submissions`, `_processed_jobs_this_step`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TieringOffloadingManager`。 它会读写 `primary_tier`, `secondary_tiers`, `_job_id_counter`, `events`, `_transfer_jobs`, `_pending_load_submissions`, `_processed_jobs_this_step` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TieringOffloadingManager.prepare_load` method / `TieringOffloadingManager.prepare_load` 方法
```python
    def prepare_load(
        self, keys: Collection[OffloadKey], req_context: ReqContext
    ) -> LoadStoreSpec:
        """
        Prepare blocks to be loaded from primary tier to GPU.

        CRITICAL: This method calls _maybe_process_finished_jobs() FIRST to ensure
        that any completed promotions have been finalized and blocks are ready.

        This increments ref_cnt on the blocks in the primary tier, protecting
        them from eviction during the transfer.

        Args:
            keys: Blocks to prepare for loading.
            req_context: Per-request context.

        Returns:
            LoadStoreSpec for reading from primary tier.
        """
        # Process completed promotions to ensure blocks are ready
        self._maybe_process_finished_jobs()

        return self.primary_tier.prepare_load(keys, req_context)
```
**EN:** This method prepares inputs and state within `TieringOffloadingManager`. The docstring frames it as: Prepare blocks to be loaded from primary tier to GPU. Key calls include `_maybe_process_finished_jobs`, `prepare_load`.
**CN:** 该方法会准备输入与状态，其作用域位于`TieringOffloadingManager`。 关键调用包括 `_maybe_process_finished_jobs`, `prepare_load`。

### `TieringOffloadingManager.prepare_store` method / `TieringOffloadingManager.prepare_store` 方法
```python
    def prepare_store(
        self, keys: Collection[OffloadKey], req_context: ReqContext
    ) -> PrepareStoreOutput | None:
        """
        Prepare blocks to be stored from GPU to primary tier.

        CRITICAL: This method calls _maybe_process_finished_jobs() FIRST to ensure
        that any completed async transfers have their ref_cnt decremented
        before the primary tier makes eviction decisions.

        Args:
            keys: Blocks to prepare for storing.
            req_context: Per-request context.

        Returns:
            PrepareStoreOutput describing where to store blocks and what was
            evicted, or None if store cannot proceed.
        """
        # Step 1: Poll for completed async jobs FIRST
        # This decrements ref_cnt on primary blocks that have been
        # successfully transferred to secondary tiers.
        self._maybe_process_finished_jobs()

        # Step 2: Store to primary tier
        primary_result = self.primary_tier.prepare_store(keys, req_context)

        # Note: Secondary tier cascading will happen in complete_store()
        # after the GPU→Primary transfer completes and blocks are ready.

        return primary_result
```
**EN:** This method prepares inputs and state within `TieringOffloadingManager`. The docstring frames it as: Prepare blocks to be stored from GPU to primary tier. Key calls include `_maybe_process_finished_jobs`, `prepare_store`.
**CN:** 该方法会准备输入与状态，其作用域位于`TieringOffloadingManager`。 关键调用包括 `_maybe_process_finished_jobs`, `prepare_store`。

## Key Concepts / 关键概念
- `PendingPromotion`: central class or interface in this module. / `PendingPromotion`：本模块中的核心类或接口。
- `CPUPrimaryTierOffloadingManager`: central class or interface in this module. / `CPUPrimaryTierOffloadingManager`：本模块中的核心类或接口。
- `TieringOffloadingManager`: central class or interface in this module. / `TieringOffloadingManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `dataclasses`
- External / 外部依赖: `numpy`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.v1.kv_offload.base`, `vllm.v1.kv_offload.cpu.common`, `vllm.v1.kv_offload.cpu.manager`, `vllm.v1.kv_offload.cpu.shared_offload_region`, `vllm.v1.kv_offload.tiering.base`
