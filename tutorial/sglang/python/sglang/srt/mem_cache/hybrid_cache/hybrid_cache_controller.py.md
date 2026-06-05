# hybrid_cache_controller.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hybrid_cache/hybrid_cache_controller.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hybrid cache controller logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的混合缓存控制器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
import threading
import time
from typing import TYPE_CHECKING, Any, Callable, List, Optional

import torch

from sglang.srt.managers.cache_controller import CacheOperation as BaseCacheOperation
from sglang.srt.managers.cache_controller import (
    HiCacheAck,
# ... omitted for brevity ...
    PoolTransferResult,
)
from sglang.srt.mem_cache.memory_pool_host import PoolEntry
from sglang.srt.utils import get_device_module
```
**EN:** Imports `__future__`, `logging`, `threading`, `time`, `typing`, `torch` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `threading`, `time`, `typing`, `torch` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 33-34: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 36-37: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
device_module = get_device_module()
```
**EN:** Defines module-level variables such as `logger`, `device_module`.
**CN:** 定义模块级变量，例如 `logger`, `device_module`。

### Lines 40-40: CacheOperation declaration / CacheOperation 声明
```python
class CacheOperation(BaseCacheOperation):
```
**EN:** Declares the `CacheOperation` class and connects it to `BaseCacheOperation`.
**CN:** 声明 `CacheOperation` 类，并将其关联到 `BaseCacheOperation`。

### Lines 41-50: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        host_indices: torch.Tensor,
        device_indices: torch.Tensor,
        node_id: int,
        priority: Optional[int] = None,
        pool_transfers: Optional[list[PoolTransfer]] = None,
    ):
        super().__init__(host_indices, device_indices, node_id, priority)
        self.pool_transfers = pool_transfers
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `CacheOperation`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `CacheOperation`。

### Lines 52-77: merge_pool_transfers implementation / merge_pool_transfers 实现
```python
    @staticmethod
    def merge_pool_transfers(
        ops: List[CacheOperation],
    ) -> Optional[list[PoolTransfer]]:
        grouped: dict[tuple[PoolName, Optional[PoolName]], list[PoolTransfer]] = {}
        for op in ops:
            for t in op.pool_transfers or []:
                grouped.setdefault((t.name, t.indices_from_pool), []).append(t)
        if not grouped:
            return None

        def cat_or_none(tensors):
# ... omitted for brevity ...
                indices_from_pool=ts[0].indices_from_pool,
            )
            for ts in grouped.values()
        ]
```
**EN:** Implements the merge pool transfers routine for this scope. It belongs to `CacheOperation`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的merge池transfers例程。 该方法属于 `CacheOperation`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 79-97: merge_ops implementation / merge_ops 实现
```python
    @staticmethod
    def merge_ops(ops: List[CacheOperation]) -> CacheOperation:
        if len(ops) == 1:
            return ops[0]
        host_indices = torch.cat([op.host_indices for op in ops])
        device_indices = torch.cat([op.device_indices for op in ops])
        node_ids = []
        priority = min(op.priority for op in ops)
        for op in ops:
            node_ids.extend(op.node_ids)
        merged = CacheOperation(
            host_indices,
# ... omitted for brevity ...
            pool_transfers=CacheOperation.merge_pool_transfers(ops),
        )
        merged.node_ids = node_ids
        return merged
```
**EN:** Implements the merge OPS routine for this scope. It belongs to `CacheOperation`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的merge ops例程。 该方法属于 `CacheOperation`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 100-100: StorageOperation declaration / StorageOperation 声明
```python
class StorageOperation(BaseStorageOperation):
```
**EN:** Declares the `StorageOperation` class and connects it to `BaseStorageOperation`.
**CN:** 声明 `StorageOperation` 类，并将其关联到 `BaseStorageOperation`。

### Lines 101-112: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        host_indices: torch.Tensor,
        token_ids: List[int],
        last_hash: Optional[str] = None,
        hash_value: Optional[List[str]] = None,
        prefix_keys: Optional[List[str]] = None,
        pool_transfers: Optional[list[PoolTransfer]] = None,
    ):
        super().__init__(host_indices, token_ids, last_hash, hash_value, prefix_keys)
        self.pool_transfers = pool_transfers
        self.pool_storage_result = PoolTransferResult.empty()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `StorageOperation`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `StorageOperation`。

### Lines 115-115: PrefetchOperation declaration / PrefetchOperation 声明
```python
class PrefetchOperation(StorageOperation):
```
**EN:** Declares the `PrefetchOperation` class and connects it to `StorageOperation`.
**CN:** 声明 `PrefetchOperation` 类，并将其关联到 `StorageOperation`。

### Lines 116-135: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        request_id: str,
        host_indices: torch.Tensor,
        token_ids: List[int],
        last_hash: Optional[str] = None,
        prefix_keys: Optional[List[str]] = None,
        pool_transfers: Optional[list[PoolTransfer]] = None,
    ):
        self.request_id = request_id
        self._lock = threading.Lock()
        self._terminated_flag = False
# ... omitted for brevity ...
            last_hash,
            prefix_keys=prefix_keys,
            pool_transfers=pool_transfers,
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `PrefetchOperation`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `PrefetchOperation`。

### Lines 137-142: increment implementation / increment 实现
```python
    def increment(self, num_tokens: int):
        with self._lock:
            if self._terminated_flag:
                return False
            self.completed_tokens += num_tokens
            return True
```
**EN:** Implements the increment routine for this scope. It belongs to `PrefetchOperation`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的increment例程。 该方法属于 `PrefetchOperation`。它会向调用方返回计算结果。

### Lines 144-146: mark_terminate implementation / mark_terminate 实现
```python
    def mark_terminate(self):
        with self._lock:
            self._terminated_flag = True
```
**EN:** Implements the mark terminate routine for this scope. It belongs to `PrefetchOperation`.
**CN:** 实现当前作用域中的mark terminate例程。 该方法属于 `PrefetchOperation`。

### Lines 148-149: is_terminated implementation / is_terminated 实现
```python
    def is_terminated(self) -> bool:
        return self._terminated_flag
```
**EN:** Checks whether a condition holds for the current state. It belongs to `PrefetchOperation`. It returns a computed result to its caller.
**CN:** 检查当前状态是否满足某个条件。 该方法属于 `PrefetchOperation`。它会向调用方返回计算结果。

### Lines 152-152: HybridCacheController declaration / HybridCacheController 声明
```python
class HybridCacheController(BaseHiCacheController):
```
**EN:** Declares the `HybridCacheController` class and connects it to `BaseHiCacheController`.
**CN:** 声明 `HybridCacheController` 类，并将其关联到 `BaseHiCacheController`。

### Lines 153-205: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        token_to_kv_pool_allocator: BaseTokenToKVPoolAllocator,
        mem_pool_host: Any,
        page_size: int,
        tp_group: torch.distributed.ProcessGroup,
        load_cache_event: threading.Event,
        attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
        attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
        write_policy: str = "write_through_selective",
        io_backend: str = "",
        storage_backend: Optional[str] = None,
# ... omitted for brevity ...
                model_name=model_name,
                storage_backend_extra_config=storage_backend_extra_config,
                host_pools=getattr(mem_pool_host, "entries", None),
            )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HybridCacheController`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HybridCacheController`。

### Lines 207-223: attach_storage_backend implementation / attach_storage_backend 实现
```python
    def attach_storage_backend(
        self,
        storage_backend: str,
        prefetch_threshold: int = 256,
        model_name: Optional[str] = None,
        storage_backend_extra_config: Optional[dict] = None,
        host_pools: Optional[list[PoolEntry]] = None,
    ):
        super().attach_storage_backend(
            storage_backend=storage_backend,
            prefetch_threshold=prefetch_threshold,
            model_name=model_name,
            storage_backend_extra_config=storage_backend_extra_config,
        )

        for entry in host_pools or []:
            self.storage_backend.register_mem_host_pool_v2(entry.host_pool, entry.name)
```
**EN:** Implements the attach storage backend routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的attach存储后端例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。

### Lines 225-229: reset implementation / reset 实现
```python
    def reset(self):
        super().reset()
        if self.enable_storage:
            self.host_mem_release_queue.queue.clear()
            self.prefetch_tokens_occupied = 0
```
**EN:** Resets the component back to its starting state. It belongs to `HybridCacheController`.
**CN:** 将组件恢复到初始状态。 该方法属于 `HybridCacheController`。

### Lines 231-261: write implementation / write 实现
```python
    def write(
        self,
        device_indices: torch.Tensor,
        priority: Optional[int] = None,
        node_id: int = -1,
        extra_pools: Optional[list[PoolTransfer]] = None,
    ) -> Optional[torch.Tensor]:
        host_indices = self.mem_pool_host.alloc(len(device_indices))
        if host_indices is None:
            return None
        pool_transfers = self._resolve_pool_transfers_allocation(
            extra_pools,
# ... omitted for brevity ...
            )
        )
        self.start_writing()
        return host_indices
```
**EN:** Implements the write routine for this scope. It belongs to `HybridCacheController`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write例程。 该方法属于 `HybridCacheController`。它会向调用方返回计算结果。

### Lines 263-290: start_writing implementation / start_writing 实现
```python
    def start_writing(self) -> None:
        if not self.write_queue:
            return
        op = CacheOperation.merge_ops(self.write_queue)
        host_indices, device_indices, resolved_pool_transfers = (
            self.move_hybrid_indices(op)
        )
        self.write_queue.clear()
        start_event = device_module.Event()
        finish_event = device_module.Event()
        start_event.record()
        with device_module.stream(self.write_stream):
# ... omitted for brevity ...
                device_indices,
                resolved_pool_transfers,
            )
        self.ack_write_queue.append(HiCacheAck(start_event, finish_event, op.node_ids))
```
**EN:** Starts the workflow or background activity. It belongs to `HybridCacheController`. It returns a computed result to its caller.
**CN:** 启动相应工作流或后台活动。 该方法属于 `HybridCacheController`。它会向调用方返回计算结果。

### Lines 292-333: load implementation / load 实现
```python
    def load(
        self,
        host_indices: torch.Tensor,
        priority: Optional[int] = None,
        node_id: int = -1,
        extra_pools: Optional[list[PoolTransfer]] = None,
    ) -> Optional[torch.Tensor]:
        need_load_kv = host_indices.numel() > 0

        full_allocator = getattr(
            self.mem_pool_device_allocator,
            "full_attn_allocator",
# ... omitted for brevity ...
                pool_transfers=pool_transfers or None,
            )
        )
        return device_indices
```
**EN:** Loads state from an external or serialized representation. It belongs to `HybridCacheController`. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HybridCacheController`。它会向调用方返回计算结果。

### Lines 335-371: start_loading implementation / start_loading 实现
```python
    def start_loading(self) -> int:
        if not self.load_queue:
            return -1
        producer_id = self.layer_done_counter.update_producer()
        op = CacheOperation.merge_ops(self.load_queue)
        host_indices, device_indices, resolved_pool_transfers = (
            self.move_hybrid_indices(op)
        )
        self.load_queue.clear()
        producer_event = self.layer_done_counter.events[producer_id]
        producer_event.start_event.record()
        with device_module.stream(self.load_stream):
# ... omitted for brevity ...
                op.node_ids,
            )
        )
        return producer_id
```
**EN:** Starts the workflow or background activity. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 启动相应工作流或后台活动。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 373-388: _record_transfer_indices_on_stream implementation / _record_transfer_indices_on_stream 实现
```python
    def _record_transfer_indices_on_stream(
        self,
        stream: torch.Stream,
        host_indices: torch.Tensor,
        device_indices: torch.Tensor,
        pool_transfers: Optional[list[PoolTransfer]] = None,
    ) -> None:
        if host_indices.is_cuda:
            host_indices.record_stream(stream)
        if device_indices.is_cuda:
            device_indices.record_stream(stream)
        for transfer in pool_transfers or []:
            if transfer.host_indices is not None and transfer.host_indices.is_cuda:
                transfer.host_indices.record_stream(stream)
            if transfer.device_indices is not None and transfer.device_indices.is_cuda:
                transfer.device_indices.record_stream(stream)
```
**EN:** Implements the record transfer indices ON stream routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的record transfer indices on stream例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。

### Lines 390-408: prefetch implementation / prefetch 实现
```python
    def prefetch(
        self,
        request_id: str,
        host_indices: torch.Tensor,
        new_input_tokens: List[int],
        last_hash: Optional[str] = None,
        prefix_keys: Optional[List[str]] = None,
        extra_pools: Optional[list[PoolTransfer]] = None,
    ) -> PrefetchOperation:
        operation = PrefetchOperation(
            request_id,
            host_indices,
# ... omitted for brevity ...
            pool_transfers=extra_pools,
        )
        self.prefetch_queue.put(operation)
        return operation
```
**EN:** Implements the prefetch routine for this scope. It belongs to `HybridCacheController`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch例程。 该方法属于 `HybridCacheController`。它会向调用方返回计算结果。

### Lines 410-426: write_storage implementation / write_storage 实现
```python
    def write_storage(
        self,
        host_indices: torch.Tensor,
        token_ids: List[int],
        hash_value: Optional[List[str]] = None,
        prefix_keys: Optional[List[str]] = None,
        extra_pools: Optional[list[PoolTransfer]] = None,
    ) -> int:
        operation = StorageOperation(
            host_indices,
            token_ids,
            hash_value=hash_value,
            prefix_keys=prefix_keys,
            pool_transfers=extra_pools,
        )
        self.backup_queue.put(operation)
        return operation.id
```
**EN:** Implements the write storage routine for this scope. It belongs to `HybridCacheController`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write存储例程。 该方法属于 `HybridCacheController`。它会向调用方返回计算结果。

### Lines 428-459: _storage_hit_query implementation / _storage_hit_query 实现
```python
    def _storage_hit_query(self, operation) -> tuple[list[str], int]:
        last_hash = operation.last_hash
        hash_value = []
        for start in range(0, len(operation.token_ids), self.page_size):
            last_hash = self.get_hash_str(
                operation.token_ids[start : start + self.page_size], last_hash
            )
            hash_value.append(last_hash)

        extra_info = HiCacheStorageExtraInfo(
            prefix_keys=operation.prefix_keys.copy() if operation.prefix_keys else None
        )
# ... omitted for brevity ...
        return (
            hash_value[:kv_hit_pages],
            kv_hit_pages * self.page_size,
        )
```
**EN:** Implements the storage HIT query routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的存储hit query例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 461-487: move_hybrid_indices implementation / move_hybrid_indices 实现
```python
    def move_hybrid_indices(
        self, operation: CacheOperation
    ) -> tuple[torch.Tensor, torch.Tensor, Optional[list[PoolTransfer]]]:
        host_indices, device_indices = self.move_indices(
            operation.host_indices, operation.device_indices
        )
        resolved_pool_transfers = None
        if operation.pool_transfers:
            resolved_pool_transfers = []
            for transfer in operation.pool_transfers:
                transfer_host_indices, transfer_device_indices = self.move_indices(
                    transfer.host_indices, transfer.device_indices
# ... omitted for brevity ...
                        indices_from_pool=transfer.indices_from_pool,
                    )
                )
        return host_indices, device_indices, resolved_pool_transfers
```
**EN:** Implements the move hybrid indices routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的move混合indices例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 489-497: _page_transfer implementation / _page_transfer 实现
```python
    def _page_transfer(self, operation):
        # Transfer extra pools
        if operation.pool_transfers and not operation.is_terminated():
            self._resolve_sidecar_derived_pool_transfers(operation)
            results = self.storage_backend.batch_get_v2(operation.pool_transfers)
            operation.pool_storage_result.update_extra_pool_hit_pages(results)

        # Transfer kv pools
        super()._page_transfer(operation)
```
**EN:** Implements the page transfer routine for this scope. It belongs to `HybridCacheController`.
**CN:** 实现当前作用域中的页transfer例程。 该方法属于 `HybridCacheController`。

### Lines 499-507: _page_backup implementation / _page_backup 实现
```python
    def _page_backup(self, operation):
        # Backup extra pools
        if operation.pool_transfers:
            self._resolve_sidecar_derived_pool_transfers(operation)
            results = self.storage_backend.batch_set_v2(operation.pool_transfers)
            operation.pool_storage_result.update_extra_pool_hit_pages(results)

        # Backup kv pools
        super()._page_backup(operation)
```
**EN:** Implements the page backup routine for this scope. It belongs to `HybridCacheController`.
**CN:** 实现当前作用域中的页backup例程。 该方法属于 `HybridCacheController`。

### Lines 509-521: _resolve_sidecar_derived_pool_transfers implementation / _resolve_sidecar_derived_pool_transfers 实现
```python
    def _resolve_sidecar_derived_pool_transfers(self, operation):
        for transfer in operation.pool_transfers:
            if transfer.indices_from_pool is None:
                continue
            if transfer.indices_from_pool != PoolName.KV:
                # TODO(hzh): Support storage sidecar derived pools from other sources
                raise AssertionError(
                    "Storage sidecar derived pool currently only supports KV-shared "
                    f"indices, got {transfer.name} from {transfer.indices_from_pool}."
                )
            transfer.host_indices = operation.host_indices
            if transfer.keys is None:
                transfer.keys = operation.hash_value
```
**EN:** Implements the resolve sidecar derived pool transfers routine for this scope. It belongs to `HybridCacheController`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的resolve sidecar derived池transfers例程。 该方法属于 `HybridCacheController`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 523-541: _sync_trailing_keys implementation / _sync_trailing_keys 实现
```python
    def _sync_trailing_keys(
        self,
        pool_transfers: list[PoolTransfer],
        all_hashes: list[str],
        kv_hit_pages: int,
    ) -> None:
        """Re-align trailing-page sidecar keys after KV hit truncation.

        When the storage hit is shorter than the original target prefix, each
        pool transfer's keys must be updated to the last N hashes of the actual
        hit range instead of the last N hashes of the original target range.
        For mamba (N=1) this is just the last hit page hash; for SWA (N>1) it
# ... omitted for brevity ...
            if transfer.hit_policy != PoolHitPolicy.TRAILING_PAGES:
                continue
            trailing_n = len(transfer.keys) if transfer.keys else 1
            transfer.keys = all_hashes[max(0, kv_hit_pages - trailing_n) : kv_hit_pages]
```
**EN:** Re-align trailing-page sidecar keys after KV hit truncation. Implements the sync trailing keys routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的sync trailing keys例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。

### Lines 543-624: _resolve_pool_transfers_allocation implementation / _resolve_pool_transfers_allocation 实现
```python
    def _resolve_pool_transfers_allocation(
        self,
        extra_pools: Optional[list[PoolTransfer]],
        alloc_host: bool,
        kv_device_indices: Optional[torch.Tensor] = None,
        kv_host_indices: Optional[torch.Tensor] = None,
    ) -> Optional[list[PoolTransfer]]:
        """Auto-alloc host or device indices for PoolTransfers where they are None."""
        if not extra_pools:
            return None
        # (pool, free_fn, indices) for atomic rollback on failure.
        newly_allocated: list[tuple[PoolTransfer, Callable, torch.Tensor]] = []
# ... omitted for brevity ...
                return None
            pool.host_indices = source.host_indices
            pool.device_indices = source.device_indices
        return extra_pools
```
**EN:** Auto-alloc host or device indices for PoolTransfers where they are None. Implements the resolve pool transfers allocation routine for this scope. It belongs to `HybridCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的resolve池transfers allocation例程。 该方法属于 `HybridCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`CacheOperation`**: Defines the `CacheOperation` type and its core responsibilities. / 定义 `CacheOperation` 类型及其核心职责。
- **`StorageOperation`**: Defines the `StorageOperation` type and its core responsibilities. / 定义 `StorageOperation` 类型及其核心职责。
- **`PrefetchOperation`**: Defines the `PrefetchOperation` type and its core responsibilities. / 定义 `PrefetchOperation` 类型及其核心职责。
- **`HybridCacheController`**: Defines the `HybridCacheController` type and its core responsibilities. / 定义 `HybridCacheController` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `threading`, `time`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.managers.cache_controller`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.utils`, `sglang.srt.mem_cache.allocator`
