# hiradix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hiradix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hiradix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的hiradix缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-59: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import atexit
import heapq
import json
import logging
import os
import threading
import time
from queue import Empty
from typing import TYPE_CHECKING, Dict, List, Optional

# ... omitted for brevity ...
    compute_node_hash_values,
    split_node_hash_value,
)
from sglang.srt.observability.metrics_collector import StorageMetricsCollector
```
**EN:** Imports `__future__`, `atexit`, `heapq`, `json`, `logging`, `os` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `atexit`, `heapq`, `json`, `logging`, `os` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 61-63: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.server_args import ServerArgs
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 65-65: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 68-69: HiRadixCache declaration / HiRadixCache 声明
```python
class HiRadixCache(RadixCache):

```
**EN:** Declares the `HiRadixCache` class and connects it to `RadixCache`.
**CN:** 声明 `HiRadixCache` 类，并将其关联到 `RadixCache`。

### Lines 70-187: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams, server_args: ServerArgs):
        self._enable_metrics_flag = params.enable_metrics

        self.page_size = params.page_size
        self.kv_cache = params.token_to_kv_pool_allocator.get_kvcache()

        if isinstance(self.kv_cache, MHATokenToKVPool):
            self.token_to_kv_pool_host = MHATokenToKVPoolHost(
                self.kv_cache,
                server_args.hicache_ratio,
                server_args.hicache_size,
                self.page_size,
# ... omitted for brevity ...

        self.evictable_host_leaves = set()

        super().__init__(params=params)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiRadixCache`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiRadixCache`。它会显式处理错误场景。

### Lines 189-196: _all_reduce_attn_groups implementation / _all_reduce_attn_groups 实现
```python
    def _all_reduce_attn_groups(self, tensor: torch.Tensor, op):
        reduced = False
        for group in (self.attn_cp_group, self.attn_tp_group):
            if group is not None and torch.distributed.get_world_size(group=group) > 1:
                torch.distributed.all_reduce(tensor, op=op, group=group)
                reduced = True
        if not reduced and self.tp_world_size > 1:
            torch.distributed.all_reduce(tensor, op=op, group=self.tp_group)
```
**EN:** Implements the ALL reduce attn groups routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的all reduce attn groups例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 198-205: _barrier_attn_groups implementation / _barrier_attn_groups 实现
```python
    def _barrier_attn_groups(self):
        waited = False
        for group in (self.attn_cp_group, self.attn_tp_group):
            if group is not None and torch.distributed.get_world_size(group=group) > 1:
                torch.distributed.barrier(group=group)
                waited = True
        if not waited and self.tp_world_size > 1:
            torch.distributed.barrier(group=self.tp_group)
```
**EN:** Implements the barrier attn groups routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的barrier attn groups例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 207-217: shutdown implementation / shutdown 实现
```python
    def shutdown(self):
        """Best-effort auto-detach of storage backend on process shutdown.

        This keeps startup and runtime behavior consistent: if a backend was attached
        (either via CLI args or via admin API), we attempt to detach it on exit.
        """
        try:
            if self.enable_storage:
                self.detach_storage_backend()
        except Exception:
            logger.exception("Failed to detach storage backend on process shutdown.")
```
**EN:** Best-effort auto-detach of storage backend on process shutdown. Implements the shutdown routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的shutdown例程。 该方法属于 `HiRadixCache`。

### Lines 219-262: _apply_storage_runtime_config implementation / _apply_storage_runtime_config 实现
```python
    def _apply_storage_runtime_config(
        self,
        *,
        storage_backend: Optional[str],
        prefetch_threshold: int,
        prefetch_timeout_config: PrefetchTimeoutConfig,
        hicache_storage_pass_prefix_keys: bool,
        enable_storage: bool,
        enable_storage_metrics: bool,
        extra_metric_labels: Optional[Dict[str, str]],
    ) -> None:
        self.enable_storage = enable_storage
# ... omitted for brevity ...
                    "avoid duplicate metric registration.",
                    sorted(existing_collector.labels.keys()),
                    sorted(labels.keys()),
                )
```
**EN:** Implements the apply storage runtime config routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的apply存储runtime config例程。 该方法属于 `HiRadixCache`。

### Lines 264-380: attach_storage_backend implementation / attach_storage_backend 实现
```python
    def attach_storage_backend(
        self,
        storage_backend: str,
        storage_backend_extra_config_json: Optional[str] = None,
        served_model_name: Optional[str] = None,
        hicache_storage_prefetch_policy: Optional[str] = None,
        hicache_write_policy: Optional[str] = None,
    ) -> tuple[bool, str]:
        """Attach (enable) storage backend at runtime.

        This will start storage threads inside `HiCacheController` and enable
        prefetch/backup paths. Caller must ensure there are no running/queued
# ... omitted for brevity ...
            enable_storage_metrics=self._enable_metrics_flag,
            extra_metric_labels=self.extra_metric_labels,
        )
        return True, "Attached HiCache storage backend successfully."
```
**EN:** Attach (enable) storage backend at runtime. Implements the attach storage backend routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的attach存储后端例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 382-409: detach_storage_backend implementation / detach_storage_backend 实现
```python
    def detach_storage_backend(self) -> tuple[bool, str]:
        """Detach (disable) storage backend at runtime.

        Caller must ensure there are no running/queued requests to avoid races.
        """
        try:
            # Drain any pending control queues before tearing down storage threads/backend.
            # IMPORTANT: this must happen before we clear `ongoing_*`, otherwise acks/releases
            # cannot be matched to nodes and may leak host pages / locks.
            self._drain_storage_control_queues_local()
            # Idempotent detach: always ask controller to best-effort cleanup, even if
            # `self.enable_storage` is already False (may be leftover state from a
# ... omitted for brevity ...

        self.enable_storage = False
        self.enable_storage_metrics = False
        return True, "Detached HiCache storage backend successfully."
```
**EN:** Detach (disable) storage backend at runtime. Implements the detach storage backend routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的detach存储后端例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 411-468: _force_release_pending_storage_ops implementation / _force_release_pending_storage_ops 实现
```python
    def _force_release_pending_storage_ops(self):
        """Force release any leftover pending prefetch/backup bookkeeping.

        This is a safety net for detach/shutdown paths. It assumes storage threads
        have been stopped already (via controller.detach), so no concurrent access
        to these structures should happen.
        """
        cc = self.cache_controller

        # Force release leftover prefetch ops: free pre-allocated host pages and
        # drop the host protection on the matched prefix node.
        try:
# ... omitted for brevity ...
                    )
                self.ongoing_backup.pop(ack_id, None)
        except Exception:
            logger.exception("Force release pending backup ops failed.")
```
**EN:** Force release any leftover pending prefetch/backup bookkeeping. Implements the force release pending storage OPS routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的force release pending存储ops例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 470-481: _drain_storage_control_queues_local implementation / _drain_storage_control_queues_local 实现
```python
    def _drain_storage_control_queues_local(self):
        """Drain storage control queues without TP synchronization.

        This is intended for shutdown/detach paths where we want to make best-effort
        cleanup even if queue sizes temporarily differ across ranks.
        """
        self._drain_storage_control_queues_impl(
            n_revoke=None,
            n_backup=None,
            n_release=None,
            log_metrics=False,
        )
```
**EN:** Drain storage control queues without TP synchronization. Implements the drain storage control queues local routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的drain存储control queues local例程。 该方法属于 `HiRadixCache`。

### Lines 483-533: _drain_storage_control_queues_impl implementation / _drain_storage_control_queues_impl 实现
```python
    def _drain_storage_control_queues_impl(
        self,
        n_revoke: Optional[int],
        n_backup: Optional[int],
        n_release: Optional[int],
        log_metrics: bool,
    ):
        cc = self.cache_controller

        def _drain_queue(q, limit: Optional[int]):
            drained = 0
            while limit is None or drained < limit:
# ... omitted for brevity ...

        _drain_revoke()
        _drain_backup()
        _drain_release()
```
**EN:** Implements the drain storage control queues impl routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drain存储control queues impl例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 535-625: _parse_storage_backend_extra_config implementation / _parse_storage_backend_extra_config 实现
```python
    def _parse_storage_backend_extra_config(
        self, storage_backend_extra_config: Optional[str]
    ):
        """
        Parse storage backend extra config JSON and extract specific parameters.

        Args:
            storage_backend_extra_config: JSON string containing extra configuration

        Returns:
            tuple: (extra_config_dict, prefetch_threshold, prefetch_timeout_config, hicache_storage_pass_prefix_keys)
        """
# ... omitted for brevity ...
            prefetch_threshold,
            prefetch_timeout_config,
            hicache_storage_pass_prefix_keys,
        )
```
**EN:** Parse storage backend extra config JSON and extract specific parameters. Implements the parse storage backend extra config routine for this scope. It belongs to `HiRadixCache`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的parse存储后端extra config例程。 该方法属于 `HiRadixCache`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 627-634: reset implementation / reset 实现
```python
    def reset(self):
        TreeNode.counter = 0
        self.cache_controller.reset()
        self.token_to_kv_pool_host.clear()
        # Clear per-request tracking dicts
        self.prefetch_loaded_tokens_by_reqid.clear()
        self.evictable_host_leaves.clear()
        super().reset()
```
**EN:** Resets the component back to its starting state. It belongs to `HiRadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `HiRadixCache`。

### Lines 636-641: get_height implementation / get_height 实现
```python
    def get_height(self, node: TreeNode):
        height = 0
        while node != self.root_node:
            node = node.parent
            height += 1
        return height
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 643-654: _get_extra_pools implementation / _get_extra_pools 实现
```python
    def _get_extra_pools(self) -> dict:
        if not isinstance(self.cache_controller, HybridCacheController):
            return {}
        if isinstance(self.kv_cache, NSATokenToKVPool):
            pool = PoolTransfer(
                name=PoolName.INDEXER,
                hit_policy=PoolHitPolicy.ALL_PAGES,
                indices_from_pool=PoolName.KV,
            )
            return {"extra_pools": [pool]}
        else:
            return {}
```
**EN:** Implements the get extra pools routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get extra pools例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 656-660: _get_hybrid_storage_attach_kwargs implementation / _get_hybrid_storage_attach_kwargs 实现
```python
    def _get_hybrid_storage_attach_kwargs(self) -> dict:
        """Extra kwargs for attach_storage_backend when controller is HybridCacheController."""
        if isinstance(self.cache_controller, HybridCacheController):
            return {"host_pools": self.cache_controller.mem_pool_host.entries}
        return {}
```
**EN:** Extra kwargs for attach_storage_backend when controller is HybridCacheController. Implements the get hybrid storage attach kwargs routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get混合存储attach kwargs例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 662-682: clear_storage_backend implementation / clear_storage_backend 实现
```python
    def clear_storage_backend(self) -> bool:
        if self.enable_storage:
            try:
                # Check if the storage backend has a clear method (for nixl backends)
                if hasattr(self.cache_controller.storage_backend, "clear"):
                    self.cache_controller.storage_backend.clear()
                    logger.info(
                        "Hierarchical cache storage backend cleared successfully!"
                    )
                    return True
                else:
                    logger.warning(
# ... omitted for brevity ...
                return False
        else:
            logger.warning("Hierarchical cache storage backend is not enabled.")
            return False
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 684-717: write_backup implementation / write_backup 实现
```python
    def write_backup(self, node: TreeNode, write_back=False) -> int:
        # Backup invariant (for write-through mode): backed-up nodes must form a
        # contiguous prefix from root — no gaps.  Skip if parent isn't backed
        # up yet;
        if not write_back and (
            node.parent != self.root_node and not node.parent.backuped
        ):
            return 0

        host_indices = self.cache_controller.write(
            device_indices=node.value,
            node_id=node.id,
# ... omitted for brevity ...
        else:
            return 0

        return len(host_indices)
```
**EN:** Implements the write backup routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write backup例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 719-734: write_backup_storage implementation / write_backup_storage 实现
```python
    def write_backup_storage(self, node: TreeNode):
        prefix_keys = (
            node.get_prefix_hash_values(node.parent)
            if self.hicache_storage_pass_prefix_keys
            else None
        )

        operation_id = self.cache_controller.write_storage(
            node.host_value,
            node.key,
            node.hash_value,
            prefix_keys,
            **self._get_extra_pools(),
        )
        self.ongoing_backup[operation_id] = node
        node.protect_host()
```
**EN:** Implements the write backup storage routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的write backup存储例程。 该方法属于 `HiRadixCache`。

### Lines 736-745: _inc_hit_count implementation / _inc_hit_count 实现
```python
    def _inc_hit_count(self, node: TreeNode, chunked=False):
        # skip the hit count update for chunked requests
        if self.cache_controller.write_policy == "write_back" or chunked:
            return
        node.hit_count += 1

        if not node.backuped:
            if node.hit_count >= self.write_through_threshold:
                # write to host if the node is not backuped
                self.write_backup(node)
```
**EN:** Implements the INC HIT count routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc hit count例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 747-789: writing_check implementation / writing_check 实现
```python
    def writing_check(self, write_back=False):
        if write_back:
            # blocking till all write back complete
            while len(self.ongoing_write_through) > 0:
                for _, finish_event, ack_list in self.cache_controller.ack_write_queue:
                    finish_event.synchronize()
                    for ack_id in ack_list:
                        backuped_node = self.ongoing_write_through.pop(ack_id)
                        # DMA confirmed -- block is now on host.
                        self._record_store_event(
                            backuped_node, medium=StorageMedium.CPU
                        )
# ... omitted for brevity ...
                self.dec_lock_ref(backuped_node)
                if self.enable_storage:
                    self.write_backup_storage(backuped_node)
            finish_count -= 1
```
**EN:** Implements the writing check routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的writing check例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 791-804: loading_check implementation / loading_check 实现
```python
    def loading_check(self):
        finish_count = 0
        for _, finish_event, ack_list in self.cache_controller.ack_load_queue:
            if not finish_event.query():
                # the KV cache loading is still ongoing
                break
            finish_count += 1
            # no need to sync across TP workers as batch forwarding is synced
            for ack_id in ack_list:
                end_node = self.ongoing_load_back.pop(ack_id)
                self.dec_lock_ref(end_node)

        # ACK until all events are processed
        del self.cache_controller.ack_load_queue[:finish_count]
```
**EN:** Implements the loading check routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的loading check例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 806-807: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self):
        return self.evictable_size_
```
**EN:** Implements the evictable size routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 809-811: _to_radix_key implementation / _to_radix_key 实现
```python
    def _to_radix_key(self, token_ids: List[int]) -> RadixKey:
        """Convert raw token_ids to a RadixKey; must be list (not tuple) for paged match."""
        return RadixKey(token_ids=list(token_ids))
```
**EN:** Convert raw token_ids to a RadixKey; must be list (not tuple) for paged match. Implements the TO radix KEY routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的to基数key例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 813-827: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: TreeNode) -> IncLockRefResult:
        if self.disable:
            return IncLockRefResult(delta=0)

        delta = 0
        while node != self.root_node:
            if node.lock_ref == 0:
                self.evictable_size_ -= len(node.key)
                self.protected_size_ += len(node.key)
                delta -= len(node.key)
            node.lock_ref += 1
            self._update_leaf_status(node)
            self._update_host_leaf_status(node)
            node = node.parent
        return IncLockRefResult(delta=delta)
```
**EN:** Implements the INC lock REF routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 829-849: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: TreeNode, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        if self.disable:
            return DecLockRefResult(delta=0)

        delta = 0
        while node != self.root_node:
            if node.lock_ref == 1:
                self.evictable_size_ += len(node.key)
                self.protected_size_ -= len(node.key)
                delta += len(node.key)
# ... omitted for brevity ...
                    node is self.root_node
                ), f"This request holds the node from another tree"
            node = node.parent
        return DecLockRefResult(delta=delta)
```
**EN:** Implements the DEC lock REF routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 851-864: _update_host_leaf_status implementation / _update_host_leaf_status 实现
```python
    def _update_host_leaf_status(self, node: TreeNode):
        if not node.evicted or node.lock_ref > 0:
            if node in self.evictable_host_leaves:
                self.evictable_host_leaves.remove(node)
            return

        for child in node.children.values():
            if child.backuped:
                if node in self.evictable_host_leaves:
                    self.evictable_host_leaves.remove(node)
                return

        if node not in self.evictable_host_leaves:
            self.evictable_host_leaves.add(node)
```
**EN:** Implements the update host leaf status routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的update主机侧leaf status例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 866-912: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        start_time = time.perf_counter()
        num_tokens = params.num_tokens
        leaves = list(self.evictable_leaves)
        eviction_heap = [
            (self.eviction_strategy.get_priority(node), node) for node in leaves
        ]
        heapq.heapify(eviction_heap)

        num_evicted = 0
        write_back_nodes = []
        while num_evicted < num_tokens and len(eviction_heap):
# ... omitted for brevity ...
                self._evict_backuped(node)

        self.update_eviction_metrics(num_evicted, start_time)
        return EvictResult(num_tokens_evicted=num_evicted)
```
**EN:** Removes cache entries according to the active policy. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 914-927: _evict_backuped implementation / _evict_backuped 实现
```python
    def _evict_backuped(self, node: TreeNode):
        # GPU -> CPU demotion: block moves from device to host.
        # Emit remove(GPU) so downstream indexers stop scoring it as device-local.
        # The matching store(CPU) was emitted when write_backup() copied to host.
        self._record_remove_event(node, medium=StorageMedium.GPU)
        num_evicted = self.cache_controller.evict_device(node.value)
        assert num_evicted > 0
        self.evictable_size_ -= num_evicted
        node.value = None
        self._update_leaf_status(node)
        self._update_host_leaf_status(node)
        # update leaf status for the parent because the node is evicted
        self._update_leaf_status(node.parent)
        return num_evicted
```
**EN:** Implements the evict backuped routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict backuped例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 929-937: _evict_regular implementation / _evict_regular 实现
```python
    def _evict_regular(self, node: TreeNode):
        # evict a node not initiated write to host -- emit BlockRemoved
        assert len(node.children) == 0, f"non-leaf, {node.id=}"

        self._record_remove_event(node)
        self.cache_controller.mem_pool_device_allocator.free(node.value)
        num_evicted = len(node.value)
        self._delete_leaf(node)
        return num_evicted
```
**EN:** Implements the evict regular routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict regular例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 939-972: evict_host implementation / evict_host 实现
```python
    def evict_host(self, num_tokens: int):
        leaves = list(self.evictable_host_leaves)
        eviction_heap = [
            (self.eviction_strategy.get_priority(node), node) for node in leaves
        ]
        heapq.heapify(eviction_heap)

        num_evicted = 0
        while num_evicted < num_tokens and len(eviction_heap):
            _priority, x = heapq.heappop(eviction_heap)
            if x == self.root_node:
                break
# ... omitted for brevity ...

            if len(x.parent.children) == 0 and x.parent.evicted:
                new_priority = self.eviction_strategy.get_priority(x.parent)
                heapq.heappush(eviction_heap, (new_priority, x.parent))
```
**EN:** Removes cache entries according to the active policy. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 974-1044: load_back implementation / load_back 实现
```python
    def load_back(
        self, node: TreeNode, mem_quota: Optional[int] = None
    ) -> Optional[torch.Tensor]:

        start_time = time.perf_counter()
        last_hit_node = node
        nodes_to_load = []
        while node.evicted:
            assert (
                node.backuped
            ), "No backup available on evicted nodes, should not happen"
            nodes_to_load.insert(0, node)
# ... omitted for brevity ...
            )
            self.metrics_collector.increment_load_back_num_tokens(len(device_indices))

        return device_indices
```
**EN:** Loads state from an external or serialized representation. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1046-1066: init_load_back implementation / init_load_back 实现
```python
    def init_load_back(
        self,
        params: InitLoadBackParams,
    ):
        last_node = params.best_match_node
        mem_quota = params.mem_quota
        if last_node.evicted:
            loading_values = self.load_back(last_node, mem_quota)
            if loading_values is not None:
                logger.debug(
                    f"loading back {len(loading_values)} tokens for node {last_node.id}"
                )
# ... omitted for brevity ...
        return (
            self._empty_match_result.device_indices,
            last_node,
        )
```
**EN:** Initializes supporting state for later operations. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 初始化后续操作所需的辅助状态。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1068-1073: ready_to_load_host_cache implementation / ready_to_load_host_cache 实现
```python
    def ready_to_load_host_cache(self) -> int:
        """
        Notify the cache controller to start the KV cache loading.
        Return the consumer index for the schedule batch manager to track.
        """
        return self.cache_controller.start_loading()
```
**EN:** Notify the cache controller to start the KV cache loading. Implements the ready TO load host cache routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的ready to load主机侧缓存例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1075-1076: flush_write_through_acks implementation / flush_write_through_acks 实现
```python
    def flush_write_through_acks(self) -> None:
        self.writing_check()
```
**EN:** Flushes buffered state to the next storage layer. It belongs to `HiRadixCache`.
**CN:** 将缓冲状态刷新到下一层存储。 该方法属于 `HiRadixCache`。

### Lines 1078-1086: check_hicache_events implementation / check_hicache_events 实现
```python
    def check_hicache_events(self):
        self.writing_check()
        self.loading_check()
        if self.enable_storage:
            self.drain_storage_control_queues()
        if self.enable_storage_metrics:
            self.storage_metrics_collector.log_storage_metrics(
                self.cache_controller.storage_backend.get_stats()
            )
```
**EN:** Implements the check hicache events routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的check hicache事件例程。 该方法属于 `HiRadixCache`。

### Lines 1088-1111: drain_storage_control_queues implementation / drain_storage_control_queues 实现
```python
    def drain_storage_control_queues(self):
        """
        Combine prefetch revoke, backup ack, and host mem release checks
        to minimize TP synchronization and Python overhead.
        """
        cc = self.cache_controller

        qsizes = torch.tensor(
            [
                cc.prefetch_revoke_queue.qsize(),
                cc.ack_backup_queue.qsize(),
                cc.host_mem_release_queue.qsize(),
# ... omitted for brevity ...
            n_backup=n_backup,
            n_release=n_release,
            log_metrics=True,
        )
```
**EN:** Combine prefetch revoke, backup ack, and host mem release checks to minimize TP synchronization and Python overhead. Implements the drain storage control queues routine for this scope. It belongs to `HiRadixCache`.
**CN:** 实现当前作用域中的drain存储control queues例程。 该方法属于 `HiRadixCache`。

### Lines 1112-1113: Comment block / 注释块
```python

    # Timeout is linearly increasing with the number of pages
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1114-1118: _prefetch_timeout_check_linear_func implementation / _prefetch_timeout_check_linear_func 实现
```python
    def _prefetch_timeout_check_linear_func(self, operation: PrefetchOperation):
        cfg = self.prefetch_timeout_config
        num_tokens = len(operation.hash_value) * self.page_size
        timeout = min(cfg.max, cfg.base + cfg.per_ki_token * num_tokens / 1024)
        return time.monotonic() - operation.start_time > timeout
```
**EN:** Implements the prefetch timeout check linear func routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch timeout check linear func例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1120-1152: can_terminate_prefetch implementation / can_terminate_prefetch 实现
```python
    def can_terminate_prefetch(self, operation: PrefetchOperation):
        can_terminate = True

        if self.prefetch_stop_policy == "best_effort":
            return can_terminate

        if len(operation.hash_value) == 0:
            completed = False
        else:
            completed = (
                operation.completed_tokens == len(operation.hash_value) * self.page_size
            )
# ... omitted for brevity ...
        # the operation should be terminated if it is already terminated on any TP worker
        # or it meets the termination condition on all TP workers
        can_terminate = can_terminate or operation_terminated
        return can_terminate
```
**EN:** Implements the CAN terminate prefetch routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的can terminate prefetch例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1154-1208: check_prefetch_progress implementation / check_prefetch_progress 实现
```python
    def check_prefetch_progress(self, req_id: str) -> bool:
        if req_id not in self.ongoing_prefetch:
            # there is no ongoing prefetch for this request or it has been revoked
            return True

        # todo: more policies for prefetch progress such as timeout
        # the current policy is to prefetch with best effort and terminate when queuing is over
        last_host_node, prefetch_key, host_indices, operation = self.ongoing_prefetch[
            req_id
        ]

        if operation.host_indices is None:
# ... omitted for brevity ...
        if self.enable_storage_metrics:
            self.storage_metrics_collector.log_prefetched_tokens(loaded_from_storage)

        return True
```
**EN:** Implements the check prefetch progress routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的check prefetch progress例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1210-1217: terminate_prefetch implementation / terminate_prefetch 实现
```python
    def terminate_prefetch(self, req_id: str):
        if req_id not in self.ongoing_prefetch:
            return

        _, _, _, operation = self.ongoing_prefetch[req_id]
        if operation.host_indices is None:
            return
        operation.mark_terminate()
```
**EN:** Implements the terminate prefetch routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的terminate prefetch例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1219-1225: pop_prefetch_loaded_tokens implementation / pop_prefetch_loaded_tokens 实现
```python
    def pop_prefetch_loaded_tokens(self, req_id: str) -> int:
        """
        Pop and return the number of tokens loaded from storage for a request.
        Returns 0 if no prefetch was done or was revoked.
        This should be called after check_prefetch_progress() returns True.
        """
        return self.prefetch_loaded_tokens_by_reqid.pop(req_id, 0)
```
**EN:** Pop and return the number of tokens loaded from storage for a request. Removes and returns a selected value. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 移除并返回指定值。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1227-1258: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams):
        if self.disable:
            return self._empty_match_result

        key = params.key
        key, _ = key.maybe_to_bigram_view(self.is_eagle)
        key = key.page_aligned(self.page_size)
        if len(key) == 0:
            return self._empty_match_result

        value, last_node = self._match_prefix_helper(self.root_node, key)
        if value:
# ... omitted for brevity ...
            # TODO(ispobock): use best_match_node as start node for load_back
            best_match_node=last_host_node,
            host_hit_length=host_hit_length,
        )
```
**EN:** Matches input data against cached state and returns the best fit. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1260-1314: prefetch_from_storage implementation / prefetch_from_storage 实现
```python
    def prefetch_from_storage(
        self,
        req_id: str,
        last_host_node: TreeNode,
        new_input_tokens: List[int],
        last_hash: Optional[str] = None,
        prefix_keys: Optional[List[str]] = None,
    ):
        prefetch_key = RadixKey(
            new_input_tokens,
            extra_key=last_host_node.key.extra_key,
            is_bigram=self.is_eagle,
# ... omitted for brevity ...
            host_indices,
            operation,
        )
        self.cache_controller.prefetch_tokens_occupied += len(prefetch_key)
```
**EN:** Implements the prefetch from storage routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch from存储例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1316-1357: _insert_helper_host implementation / _insert_helper_host 实现
```python
    def _insert_helper_host(
        self, node: TreeNode, key: RadixKey, host_value, hash_value
    ):
        node.last_access_time = time.monotonic()
        if len(key) == 0:
            return 0

        child_key = key.child_key(self.page_size)

        matched_length = 0
        while len(key) > 0 and child_key in node.children.keys():
            node = node.children[child_key]
# ... omitted for brevity ...
            # cache indexers can resolve descendants that extend this L2-only prefix.
            self._record_store_event(new_node, medium=StorageMedium.CPU)

        return matched_length
```
**EN:** Implements the insert helper host routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper主机侧例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1359-1383: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(self, node: TreeNode, key: RadixKey):
        node.last_access_time = time.monotonic()
        child_key = key.child_key(self.page_size)
        value = []

        while len(key) > 0 and child_key in node.children.keys():
            child = node.children[child_key]
            child.last_access_time = time.monotonic()
            prefix_len = child.key.match(key, page_size=self.page_size)
            if prefix_len < len(child.key):
                new_node = self._split_node(child.key, child, prefix_len)
                if not new_node.evicted:
# ... omitted for brevity ...
                if len(key):
                    child_key = key.child_key(self.page_size)

        return value, node
```
**EN:** Implements the match prefix helper routine for this scope. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1385-1411: _split_node implementation / _split_node 实现
```python
    def _split_node(self, key: RadixKey, child: TreeNode, split_len: int):
        # child node split into new_node -> child
        new_node = TreeNode(priority=child.priority)
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.lock_ref = child.lock_ref
        new_node.key = child.key[:split_len]
        new_node.hit_count = child.hit_count

        # split value and host value if exists
        if child.evicted:
            new_node.value = None
# ... omitted for brevity ...
        child.key = child.key[split_len:]
        new_node.parent.children[key.child_key(self.page_size)] = new_node

        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

### Lines 1413-1495: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        key = params.key
        value = params.value
        chunked = params.chunked
        priority = params.priority

        if priority is None:
            priority = 0

        key, value = key.maybe_to_bigram_view(self.is_eagle, value)
        key = key.page_aligned(self.page_size)
        if value is not None:
# ... omitted for brevity ...

            if self.cache_controller.write_policy != "write_back":
                self._inc_hit_count(new_node, chunked)
        return InsertResult(prefix_len=total_prefix_length)
```
**EN:** Inserts new data into the managed structure. It belongs to `HiRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `HiRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1497-1515: release_aborted_request implementation / release_aborted_request 实现
```python
    def release_aborted_request(self, rid: str):
        # Clean up storage hit tracking for aborted request
        self.prefetch_loaded_tokens_by_reqid.pop(rid, None)

        if rid not in self.ongoing_prefetch:
            return

        last_host_node, prefetch_key, host_indices, operation = self.ongoing_prefetch[
            rid
        ]
        if operation.host_indices is None:
            return
# ... omitted for brevity ...
        last_host_node.release_host()
        del self.ongoing_prefetch[rid]
        self.cache_controller.append_host_mem_release(host_indices[:completed_tokens])
        self.cache_controller.prefetch_tokens_occupied -= len(prefetch_key)
```
**EN:** Implements the release aborted request routine for this scope. It belongs to `HiRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的release aborted请求例程。 该方法属于 `HiRadixCache`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`HiRadixCache`**: Defines the `HiRadixCache` type and its core responsibilities. / 定义 `HiRadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `atexit`, `heapq`, `json`, `logging`, `os`, `threading`, `time`, `queue`, `typing`, `torch`, `tomllib`
- **Internal / 内部**: `sglang.srt.disaggregation.kv_events`, `sglang.srt.managers.cache_controller`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller`, `sglang.srt.mem_cache.hybrid_cache.hybrid_pool_assembler`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.utils`, `sglang.srt.observability.metrics_collector`, `sglang.srt.mem_cache.cache_init_params`
