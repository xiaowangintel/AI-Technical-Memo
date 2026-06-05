# hi_mamba_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hi_mamba_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the HI mamba radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的hi Mamba基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and setup / 导入与初始化
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
from typing import TYPE_CHECKING, Dict, List, Optional, Tuple

# ... omitted for brevity ...
    RadixKey,
)
from sglang.srt.mem_cache.utils import compute_node_hash_values, split_node_hash_value
from sglang.srt.observability.metrics_collector import StorageMetricsCollector
```
**EN:** Imports `__future__`, `atexit`, `heapq`, `json`, `logging`, `os` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `atexit`, `heapq`, `json`, `logging`, `os` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 51-53: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.server_args import ServerArgs
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 55-55: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 58-58: HostLRUList declaration / HostLRUList 声明
```python
class HostLRUList(LRUList):
```
**EN:** Declares the `HostLRUList` class and connects it to `LRUList`.
**CN:** 声明 `HostLRUList` 类，并将其关联到 `LRUList`。

### Lines 59-64: __init__ implementation / __init__ 实现
```python
    def __init__(self):
        super().__init__(mamba=True)
        self.prv = "host_mamba_prev"
        self.nxt = "host_mamba_next"
        setattr(self.head, self.nxt, self.tail)
        setattr(self.tail, self.prv, self.head)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HostLRUList`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HostLRUList`。

### Lines 66-72: reset_node_mru implementation / reset_node_mru 实现
```python
    def reset_node_mru(self, node):
        assert node.id in self.cache, f"Resetting node {node.id=} not in host mamba lru"
        assert (
            node.mamba_host_value is not None
        ), f"Resetting host mamba tombstone node in lru list: {node.id=}"
        self._remove_node(node)
        self._add_node(node)
```
**EN:** Resets the component back to its starting state. It belongs to `HostLRUList`.
**CN:** 将组件恢复到初始状态。 该方法属于 `HostLRUList`。

### Lines 74-82: insert_mru implementation / insert_mru 实现
```python
    def insert_mru(self, node):
        assert (
            node.mamba_host_value is not None
        ), f"Inserting host mamba tombstone node in lru list: {node.id=}"
        assert (
            node.id not in self.cache
        ), f"Inserting node {node.id=} already in host mamba lru list"
        self.cache[node.id] = node
        self._add_node(node)
```
**EN:** Inserts new data into the managed structure. It belongs to `HostLRUList`.
**CN:** 向受管结构中插入新数据。 该方法属于 `HostLRUList`。

### Lines 84-90: remove_node implementation / remove_node 实现
```python
    def remove_node(self, node: TreeNode):
        assert node.id in self.cache, f"Removing node {node.id=} not in host mamba lru"
        assert (
            node.mamba_host_value is not None
        ), f"Removing host mamba tombstone node from lru list: {node.id=}"
        del self.cache[node.id]
        self._remove_node(node)
```
**EN:** Removes selected data from the managed structure. It belongs to `HostLRUList`.
**CN:** 从受管结构中删除选定数据。 该方法属于 `HostLRUList`。

### Lines 93-95: HiMambaRadixCache declaration / HiMambaRadixCache 声明
```python
class HiMambaRadixCache(MambaRadixCache):
    """Hierarchical cache for hybrid Mamba models."""

```
**EN:** Hierarchical cache for hybrid Mamba models. Declares the `HiMambaRadixCache` class and connects it to `MambaRadixCache`.
**CN:** 声明 `HiMambaRadixCache` 类，并将其关联到 `MambaRadixCache`。

### Lines 96-183: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams, server_args: ServerArgs):
        self._enable_metrics_flag = params.enable_metrics
        if server_args.hicache_io_backend == "direct":
            if server_args.hicache_mem_layout == "page_first":
                server_args.hicache_mem_layout = "page_first_direct"
                logger.warning(
                    "Page first layout is not supported with direct IO backend, "
                    "switching to page first direct layout"
                )

        self.page_size = params.page_size
        self.hybrid_kv_cache = params.token_to_kv_pool_allocator.get_kvcache()
# ... omitted for brevity ...
        # Detach storage backend automatically on process shutdown
        atexit.register(self.shutdown)

        super().__init__(params=params)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiMambaRadixCache`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiMambaRadixCache`。它会显式处理错误场景。

### Lines 185-204: reset implementation / reset 实现
```python
    def reset(self) -> None:
        TreeNode.counter = 0
        self._flush_pending_storage_backups_before_reset()
        self.cache_controller.reset()
        self.full_kv_pool_host.clear()
        self.mamba_pool_host.clear()
        self.ongoing_write_through = {}
        self.ongoing_load_back = {}
        self.ongoing_prefetch = {}
        self.ongoing_backup = {}
        self.prefetch_loaded_tokens_by_reqid.clear()
        self.evictable_full_device_leaves.clear()
# ... omitted for brevity ...
            self.full_kv_pool_host.available_size(),
            self.mamba_pool_host.available_size(),
        )
        super().reset()
```
**EN:** Resets the component back to its starting state. It belongs to `HiMambaRadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `HiMambaRadixCache`。

### Lines 206-245: write_backup implementation / write_backup 实现
```python
    def write_backup(self, node: TreeNode, write_back=False) -> int:
        # Backup invariant (for write-through mode): backed-up nodes must form a
        # contiguous prefix from root — no gaps.  Skip if parent isn't backed
        # up yet;
        if not write_back and (
            node.parent != self.root_node and not node.parent.backuped
        ):
            return 0

        # If mamba host slot already exists, refresh its LRU position.
        if node.mamba_value is not None and node.mamba_host_value is not None:
            if self.mamba_host_lru_list.in_list(node):
# ... omitted for brevity ...
        else:
            return 0

        return len(host_indices)
```
**EN:** Implements the write backup routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write backup例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 247-342: load_back implementation / load_back 实现
```python
    def load_back(
        self, node: TreeNode, mem_quota: Optional[int] = None, req=None
    ) -> Optional[torch.Tensor]:
        """Load full KV back from host."""
        last_hit_node = node
        nodes_to_load = []

        while node.evicted:
            assert node.backuped, f"No backup on evicted node {node.id}"
            nodes_to_load.insert(0, node)
            node = node.parent
        else:
# ... omitted for brevity ...
        self.inc_lock_ref(last_hit_node)
        self.ongoing_load_back[last_hit_node.id] = last_hit_node

        return full_device_indices
```
**EN:** Load full KV back from host. Loads state from an external or serialized representation. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 344-367: init_load_back implementation / init_load_back 实现
```python
    def init_load_back(
        self,
        params: InitLoadBackParams,
    ):
        last_node = params.best_match_node
        mem_quota = params.mem_quota
        req = params.req
        if last_node.evicted or (last_node.mamba_evicted and last_node.mamba_backuped):
            loading_values = self.load_back(last_node, mem_quota, req=req)
            if loading_values is not None:
                logger.debug(
                    f"loading back {len(loading_values)} tokens for node {last_node.id}"
# ... omitted for brevity ...
        return (
            torch.empty((0,), dtype=torch.int64, device=self.device),
            last_node,
        )
```
**EN:** Initializes supporting state for later operations. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 初始化后续操作所需的辅助状态。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 369-376: _inc_hit_count implementation / _inc_hit_count 实现
```python
    def _inc_hit_count(self, node: TreeNode, chunked=False):
        if self.cache_controller.write_policy == "write_back" or chunked:
            return
        node.hit_count += 1

        if not node.backuped and node.hit_count >= self.write_through_threshold:
            # write to host if the node is not backuped
            self.write_backup(node)
```
**EN:** Implements the INC HIT count routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc hit count例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 378-422: writing_check implementation / writing_check 实现
```python
    def writing_check(self, write_back=False):
        if write_back:
            # blocking till all write back complete
            while len(self.ongoing_write_through) > 0:
                for _, finish_event, ack_list in self.cache_controller.ack_write_queue:
                    finish_event.synchronize()
                    for ack_id in ack_list:
                        backuped_node = self.ongoing_write_through.pop(ack_id)
                        self._record_store_event(
                            backuped_node, medium=StorageMedium.CPU
                        )
                        if self.enable_storage:
# ... omitted for brevity ...
                self.dec_lock_ref(backuped_node)
                if self.enable_storage:
                    self.write_backup_storage(backuped_node)
            finish_count -= 1
```
**EN:** Implements the writing check routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的writing check例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 424-435: loading_check implementation / loading_check 实现
```python
    def loading_check(self):
        finish_count = 0
        for _, finish_event, ack_list in self.cache_controller.ack_load_queue:
            if not finish_event.query():
                # the KV cache loading is still ongoing
                break
            finish_count += 1
            for ack_id in ack_list:
                end_node = self.ongoing_load_back.pop(ack_id)
                self.dec_lock_ref(end_node)

        del self.cache_controller.ack_load_queue[:finish_count]
```
**EN:** Implements the loading check routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的loading check例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 437-438: ready_to_load_host_cache implementation / ready_to_load_host_cache 实现
```python
    def ready_to_load_host_cache(self) -> int:
        return self.cache_controller.start_loading()
```
**EN:** Implements the ready TO load host cache routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的ready to load主机侧缓存例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 440-441: flush_write_through_acks implementation / flush_write_through_acks 实现
```python
    def flush_write_through_acks(self) -> None:
        self.writing_check()
```
**EN:** Flushes buffered state to the next storage layer. It belongs to `HiMambaRadixCache`.
**CN:** 将缓冲状态刷新到下一层存储。 该方法属于 `HiMambaRadixCache`。

### Lines 443-452: check_hicache_events implementation / check_hicache_events 实现
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
**EN:** Implements the check hicache events routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的check hicache事件例程。 该方法属于 `HiMambaRadixCache`。

### Lines 454-460: _protect_host_node implementation / _protect_host_node 实现
```python
    def _protect_host_node(self, node: TreeNode, protect_mamba: bool = True):
        node.protect_host()
        self.evictable_full_host_leaves.discard(node)
        if protect_mamba:
            node.protect_host_mamba()
            if self.mamba_host_lru_list.in_list(node):
                self.mamba_host_lru_list.remove_node(node)
```
**EN:** Implements the protect host node routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的protect主机侧node例程。 该方法属于 `HiMambaRadixCache`。

### Lines 462-470: _release_host_node implementation / _release_host_node 实现
```python
    def _release_host_node(self, node: TreeNode, release_mamba: bool = True):
        node.release_host()
        if release_mamba:
            node.release_host_mamba()
            if node.host_mamba_ref_counter == 0 and node.mamba_host_value is not None:
                if not self.mamba_host_lru_list.in_list(node):
                    self.mamba_host_lru_list.insert_mru(node)
        if node.host_ref_counter == 0 and node.host_mamba_ref_counter == 0:
            self._update_full_host_leaf_status(node)
```
**EN:** Implements the release host node routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的release主机侧node例程。 该方法属于 `HiMambaRadixCache`。

### Lines 472-474: _discard_from_leaf_sets implementation / _discard_from_leaf_sets 实现
```python
    def _discard_from_leaf_sets(self, node: TreeNode):
        self.evictable_full_device_leaves.discard(node)
        self.evictable_full_host_leaves.discard(node)
```
**EN:** Implements the discard from leaf sets routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的discard from leaf sets例程。 该方法属于 `HiMambaRadixCache`。

### Lines 476-478: _update_leaf_status implementation / _update_leaf_status 实现
```python
    def _update_leaf_status(self, node: TreeNode):
        self._update_full_device_leaf_status(node)
        self._update_full_host_leaf_status(node)
```
**EN:** Implements the update leaf status routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的update leaf status例程。 该方法属于 `HiMambaRadixCache`。

### Lines 480-488: _update_full_device_leaf_status implementation / _update_full_device_leaf_status 实现
```python
    def _update_full_device_leaf_status(self, node: TreeNode):
        if node == self.root_node or node.evicted or node.full_lock_ref > 0:
            self.evictable_full_device_leaves.discard(node)
            return
        for child in node.children.values():
            if not child.evicted:
                self.evictable_full_device_leaves.discard(node)
                return
        self.evictable_full_device_leaves.add(node)
```
**EN:** Implements the update full device leaf status routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的update完整device leaf status例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 490-501: _update_full_host_leaf_status implementation / _update_full_host_leaf_status 实现
```python
    def _update_full_host_leaf_status(self, node: TreeNode):
        if (
            not node.evicted
            or not node.backuped
            or node == self.root_node
            or node.host_ref_counter > 0
            or node.host_mamba_ref_counter > 0
            or len(node.children) > 0
        ):
            self.evictable_full_host_leaves.discard(node)
            return
        self.evictable_full_host_leaves.add(node)
```
**EN:** Implements the update full host leaf status routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的update完整主机侧leaf status例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 503-516: _free_device_mamba implementation / _free_device_mamba 实现
```python
    def _free_device_mamba(self, node: TreeNode) -> int:
        if node.mamba_value is None:
            return 0
        mamba_num = len(node.mamba_value)
        self.req_to_token_pool.mamba_pool.free(node.mamba_value)
        if node.mamba_lock_ref > 0:
            self.mamba_protected_size_ -= mamba_num
            node.mamba_lock_ref = 0
        else:
            self.mamba_evictable_size_ -= mamba_num
        if self.mamba_lru_list.in_list(node):
            self.mamba_lru_list.remove_node(node)
        node.mamba_value = None
        return mamba_num
```
**EN:** Implements the free device mamba routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的free device Mamba例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 518-536: _evict_to_host implementation / _evict_to_host 实现
```python
    def _evict_to_host(self, node: TreeNode) -> Tuple[int, int]:
        # GPU -> CPU demotion: node stays in tree as evicted+backuped
        assert not node.evicted, f"already evicted, {node.id=}"
        assert node.backuped, f"not backuped, {node.id=}"

        num_full = len(node.value)

        self._record_remove_event(node, medium=StorageMedium.GPU)
        self.cache_controller.evict_device(node.value)
        self.full_evictable_size_ -= num_full
        if self.full_lru_list.in_list(node):
            self.full_lru_list.remove_node(node)
# ... omitted for brevity ...
        node.value = None
        self._update_leaf_status(node)
        self._update_full_device_leaf_status(node.parent)
        return num_full, mamba_num
```
**EN:** Implements the evict TO host routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict to主机侧例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 538-575: _evict_regular implementation / _evict_regular 实现
```python
    def _evict_regular(self, node: TreeNode) -> Tuple[int, int]:
        # evict a non-backuped device leaf — free GPU KV + mamba, delete from tree
        assert not node.evicted, f"already evicted, {node.id=}"
        assert not node.backuped, f"backuped node, {node.id=}"
        assert len(node.children) == 0, f"non-leaf, {node.id=}"

        full_num_evicted = len(node.value)

        self._record_remove_event(node, medium=StorageMedium.GPU)
        self.cache_controller.evict_device(node.value)
        self.full_evictable_size_ -= full_num_evicted
        if self.full_lru_list.in_list(node):
# ... omitted for brevity ...
        return (
            full_num_evicted + cascade_full_num_evicted,
            mamba_num_evicted + cascade_mamba_num_evicted,
        )
```
**EN:** Implements the evict regular routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict regular例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 577-608: _evict_host_leaf implementation / _evict_host_leaf 实现
```python
    def _evict_host_leaf(self, node: TreeNode) -> int:
        # evict a host-resident leaf: free host KV + mamba, delete from tree, cascade
        assert node.evicted, f"not evicted, {node.id=}"
        assert node.backuped, f"not backuped, {node.id=}"
        assert node.mamba_value is None, f"has device mamba, {node.id=}"
        assert (
            node.host_ref_counter == 0
        ), f"host kv in use, {node.id=} {node.host_ref_counter=}"
        assert (
            node.host_mamba_ref_counter == 0
        ), f"host mamba in use, {node.id=} {node.host_mamba_ref_counter=}"

# ... omitted for brevity ...
        self._update_leaf_status(parent)
        _, cascade_full_num_evicted, _ = self._iteratively_delete_tombstone_leaf(node)

        return full_num_evicted + cascade_full_num_evicted
```
**EN:** Implements the evict host leaf routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict主机侧leaf例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 610-630: _delete_tombstone_leaf implementation / _delete_tombstone_leaf 实现
```python
    def _delete_tombstone_leaf(self, node: TreeNode) -> None:
        assert node.mamba_value is None, f"has mamba value, {node.id=}"
        assert node.mamba_host_value is None, f"has mamba host value, {node.id=}"
        assert len(node.children) == 0, f"leaf node has children, {node.id=}"
        parent = node.parent
        key = node.key.child_key(self.page_size)
        v = parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"

        self._discard_from_leaf_sets(node)

        if (
# ... omitted for brevity ...
            self.cache_controller.evict_host(node.host_value)
            node.host_value = None

        self._update_leaf_status(parent)
```
**EN:** Implements the delete tombstone leaf routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的delete tombstone leaf例程。 该方法属于 `HiMambaRadixCache`。

### Lines 632-667: _iteratively_delete_tombstone_leaf implementation / _iteratively_delete_tombstone_leaf 实现
```python
    def _iteratively_delete_tombstone_leaf(
        self, node: TreeNode
    ) -> Tuple[TreeNode, int, int]:
        full_num_evicted = 0
        mamba_num_evicted = 0

        while len(node.parent.children) == 0:
            if node.parent == self.root_node:
                break
            if node.parent.mamba_value is not None:
                break
            if node.parent.mamba_host_value is not None:
# ... omitted for brevity ...
            self._delete_tombstone_leaf(parent)
            node = parent

        return node, full_num_evicted, mamba_num_evicted
```
**EN:** Implements the iteratively delete tombstone leaf routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的iteratively delete tombstone leaf例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 669-683: _evict_device_leaf implementation / _evict_device_leaf 实现
```python
    def _evict_device_leaf(self, x: TreeNode) -> Tuple[int, int]:
        """Evict a device leaf node, choosing the right strategy:

        - backuped: demote to host via _evict_to_host (node stays in tree)
        - not backuped + write_back: write_backup first, then demote
        - not backuped + write_through: _evict_regular (delete from tree)
        """
        if not x.backuped:
            if self.cache_controller.write_policy == "write_back":
                self.write_backup(x, write_back=True)
                self.writing_check(write_back=True)
                return self._evict_to_host(x)
            else:
                return self._evict_regular(x)
        return self._evict_to_host(x)
```
**EN:** Evict a device leaf node, choosing the right strategy: - backuped: demote to host via _evict_to_host (node stays in tree) - not backuped + write_back: write_backup first, then... Implements the evict device leaf routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict device leaf例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 685-717: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        if self.disable:
            return EvictResult()

        full_num_tokens = params.num_tokens
        full_num_evicted = 0
        mamba_num_evicted = 0

        if full_num_tokens > 0:
            leaves = list(self.evictable_full_device_leaves)
            eviction_heap = [(n.last_access_time, n) for n in leaves]
            heapq.heapify(eviction_heap)
# ... omitted for brevity ...
        return EvictResult(
            num_tokens_evicted=full_num_evicted,
            mamba_num_evicted=mamba_num_evicted,
        )
```
**EN:** Removes cache entries according to the active policy. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 719-733: evict_host implementation / evict_host 实现
```python
    def evict_host(self, num_tokens: int):
        """Evict host-resident leaf nodes: free host KV + mamba, delete from tree, cascade."""
        heap = [(n.last_access_time, n) for n in self.evictable_full_host_leaves]
        heapq.heapify(heap)

        num_evicted = 0
        while num_evicted < num_tokens and heap:
            _, x = heapq.heappop(heap)
            if x not in self.evictable_full_host_leaves:
                continue

            num_evicted += self._evict_host_leaf(x)

            if x.parent in self.evictable_full_host_leaves:
                heapq.heappush(heap, (x.parent.last_access_time, x.parent))
```
**EN:** Evict host-resident leaf nodes: free host KV + mamba, delete from tree, cascade. Removes cache entries according to the active policy. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 735-770: evict_mamba_host implementation / evict_mamba_host 实现
```python
    def evict_mamba_host(self, num_mamba_hosts: int) -> int:
        """Evict host mamba states.

        Internal host node: free host mamba only (tombstone).
        Host leaf node: same as Full host evict — _evict_host_leaf_node frees
                        host KV + mamba, deletes from tree, cascades.
        """
        if self.disable or num_mamba_hosts <= 0:
            return 0

        x = self.mamba_host_lru_list.get_lru_no_lock()
        num_evicted = 0
# ... omitted for brevity ...
                num_evicted += 1

            x = x_next
        return num_evicted
```
**EN:** Evict host mamba states. Removes cache entries according to the active policy. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 772-814: evict_mamba implementation / evict_mamba 实现
```python
    def evict_mamba(self, mamba_num: int) -> int:
        """Evict mamba states.

        Internal node: tombstone — free GPU mamba only, KV stays on GPU.
        Leaf node: same as Full evict — _evict_to_host moves KV+mamba to host,
                   node stays in tree, then cascade tombstone parent device leaves.
        """
        if self.disable or mamba_num <= 0:
            return 0

        x = self.mamba_lru_list.get_lru_no_lock()
        mamba_num_evicted = 0
# ... omitted for brevity ...

            x = x_next

        return mamba_num_evicted
```
**EN:** Evict mamba states. Removes cache entries according to the active policy. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 816-828: _unevict_node implementation / _unevict_node 实现
```python
    def _unevict_node(self, node: TreeNode, fresh_value: torch.Tensor):
        assert node.evicted, f"not evicted, {node.id=}"
        assert node.mamba_value is None, f"evicted node has device mamba, {node.id=}"
        n = len(fresh_value)

        node.value = fresh_value.clone()
        self.full_lru_list.insert_mru(node)
        self.full_evictable_size_ += n
        self._record_store_event(node, medium=StorageMedium.GPU)

        self._update_leaf_status(node)
        if node.parent is not None:
            self._update_leaf_status(node.parent)
```
**EN:** Implements the unevict node routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的unevict node例程。 该方法属于 `HiMambaRadixCache`。

### Lines 830-900: _insert_helper implementation / _insert_helper 实现
```python
    def _insert_helper(
        self,
        node: TreeNode,
        key: RadixKey,
        value,
        mamba_value,
        chunked: bool = False,
        prev_prefix_len: int = 0,
    ) -> Tuple[int, bool]:
        assert mamba_value is not None, "Mamba value should not be None here."
        node.last_access_time = get_last_access_time()
        if node != self.root_node:
# ... omitted for brevity ...
            self.mamba_lru_list.reset_node_mru(node)
            node.last_access_time = get_last_access_time()

        return total_prefix_length, mamba_value_exist
```
**EN:** Implements the insert helper routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 902-925: _add_new_node implementation / _add_new_node 实现
```python
    def _add_new_node(
        self,
        parent: TreeNode,
        key: RadixKey,
        value: torch.Tensor,
        mamba_value: torch.Tensor,
    ) -> TreeNode:
        child_key = key.child_key(self.page_size)
        new_node = TreeNode()
        new_node.parent = parent
        new_node.key = key
        new_node.value = value.clone()
# ... omitted for brevity ...
        self._record_store_event(new_node, medium=StorageMedium.GPU)
        self._update_full_device_leaf_status(new_node)
        self._update_full_device_leaf_status(parent)
        return new_node
```
**EN:** Implements the ADD NEW node routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的add new node例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 927-944: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        key = params.key

        if self.disable or len(key) == 0:
            return MatchResult(
                device_indices=torch.empty((0,), dtype=torch.int64, device=self.device),
                last_device_node=self.root_node,
                last_host_node=self.root_node,
                best_match_node=self.root_node,
                host_hit_length=0,
            )

        if self.page_size != 1:
            page_aligned_len = len(key) // self.page_size * self.page_size
            key = key[:page_aligned_len]

        value, best_last_node, best_value_len = self._match_prefix_helper(key)
        return self._match_post_processor(params, value, best_last_node, best_value_len)
```
**EN:** Matches input data against cached state and returns the best fit. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 946-986: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(
        self, key: RadixKey
    ) -> Tuple[List[torch.Tensor], TreeNode, int]:
        """Walk tree to find best_last_node (mamba boundary)."""
        node = self.root_node
        child_key = key.child_key(self.page_size)

        value: List[torch.Tensor] = []
        best_value_len = 0
        best_last_node = node

        while len(key) > 0 and child_key in node.children.keys():
# ... omitted for brevity ...
            best_value_len = len(value)
            best_last_node = node

        return value, best_last_node, best_value_len
```
**EN:** Walk tree to find best_last_node (mamba boundary). Implements the match prefix helper routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 988-1074: _match_post_processor implementation / _match_post_processor 实现
```python
    def _match_post_processor(
        self,
        params: MatchPrefixParams,
        value: List[torch.Tensor],
        best_last_node: TreeNode,
        best_value_len: int,
    ) -> MatchResult:
        cow_mamba = params.cow_mamba
        req = params.req

        # Full LRU: skip evicted nodes for full_lru_list
        lru_node = best_last_node
# ... omitted for brevity ...
            best_match_node=last_host_node,
            host_hit_length=host_hit_length,
            mamba_branching_seqlen=mamba_branching_seqlen,
        )
```
**EN:** Implements the match post processor routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match post processor例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1076-1091: _split_node implementation / _split_node 实现
```python
    def _split_node(self, key: RadixKey, child: TreeNode, split_len: int) -> TreeNode:
        if child.evicted:
            return self._split_evicted_node(key, child, split_len)

        self.evictable_full_device_leaves.discard(child)

        new_node = super()._split_node(key, child, split_len)

        if child.backuped:
            new_node.host_value = child.host_value[:split_len].clone()
            child.host_value = child.host_value[split_len:].clone()

        self._update_leaf_status(new_node)
        self._update_leaf_status(child)

        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1093-1127: _split_evicted_node implementation / _split_evicted_node 实现
```python
    def _split_evicted_node(
        self, key: RadixKey, child: TreeNode, split_len: int
    ) -> TreeNode:
        self.evictable_full_host_leaves.discard(child)

        new_node = TreeNode()
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.value = None
        new_node.mamba_value = None
        new_node.full_lock_ref = child.full_lock_ref
        new_node.mamba_lock_ref = 0
# ... omitted for brevity ...
        self._update_full_host_leaf_status(new_node)
        self._update_full_host_leaf_status(child)

        return new_node
```
**EN:** Implements the split evicted node routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split evicted node例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1129-1137: _collect_all_nodes implementation / _collect_all_nodes 实现
```python
    def _collect_all_nodes(self) -> list:
        ret = []
        stack = [self.root_node]
        while stack:
            cur = stack.pop()
            if not cur.evicted:
                ret.append(cur)
            stack.extend(cur.children.values())
        return ret
```
**EN:** Implements the collect ALL nodes routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect all nodes例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1139-1147: _collect_mamba_nontombstone_nodes implementation / _collect_mamba_nontombstone_nodes 实现
```python
    def _collect_mamba_nontombstone_nodes(self) -> list:
        ret = []
        stack = [self.root_node]
        while stack:
            cur = stack.pop()
            if cur.mamba_value is not None:
                ret.append(cur)
            stack.extend(cur.children.values())
        return ret
```
**EN:** Implements the collect mamba nontombstone nodes routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect Mamba nontombstone nodes例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1149-1159: all_values_flatten implementation / all_values_flatten 实现
```python
    def all_values_flatten(self) -> torch.Tensor:
        values = []

        def _dfs(node: TreeNode):
            for child in node.children.values():
                if not child.evicted:
                    values.append(child.value)
                _dfs(child)

        _dfs(self.root_node)
        return torch.cat(values) if values else torch.tensor([])
```
**EN:** Implements the ALL values flatten routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all values flatten例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1161-1166: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self):
        """Skip if async operations are pending (those nodes are still locked)."""
        self.loading_check()
        if self.ongoing_load_back or self.ongoing_write_through:
            return
        super().sanity_check()
```
**EN:** Skip if async operations are pending (those nodes are still locked). Implements the sanity check routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1168-1194: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: TreeNode) -> IncLockRefResult:
        if self.disable:
            return IncLockRefResult(delta=0)

        delta = 0
        if node.mamba_value is not None:
            if node.mamba_lock_ref == 0:
                self.mamba_evictable_size_ -= len(node.mamba_value)
                self.mamba_protected_size_ += len(node.mamba_value)
            node.mamba_lock_ref += 1

        while node != self.root_node:
# ... omitted for brevity ...
                self.evictable_full_device_leaves.discard(node)
            node.full_lock_ref += 1
            node = node.parent
        return IncLockRefResult(delta=delta)
```
**EN:** Implements the INC lock REF routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1196-1226: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: TreeNode, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        if self.disable:
            return DecLockRefResult(delta=0)

        delta = 0

        if node.mamba_value is not None and node.mamba_lock_ref > 0:
            if node.mamba_lock_ref == 1:
                self.mamba_evictable_size_ += len(node.mamba_value)
                self.mamba_protected_size_ -= len(node.mamba_value)
# ... omitted for brevity ...
            if node.full_lock_ref == 0:
                self._update_full_device_leaf_status(node)
            node = node.parent
        return DecLockRefResult(delta=delta)
```
**EN:** Implements the DEC lock REF routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1227-1229: Comment block / 注释块
```python

    # ---- L3 Support ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1230-1235: shutdown implementation / shutdown 实现
```python
    def shutdown(self):
        try:
            if self.enable_storage:
                self.detach_storage_backend()
        except Exception:
            logger.exception("Failed to detach storage backend on process shutdown.")
```
**EN:** Implements the shutdown routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的shutdown例程。 该方法属于 `HiMambaRadixCache`。

### Lines 1237-1269: _apply_storage_runtime_config implementation / _apply_storage_runtime_config 实现
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
        storage_metrics_collector = None
# ... omitted for brevity ...
        if self.enable_storage_metrics:
            self.storage_metrics_collector = storage_metrics_collector
        else:
            self.storage_metrics_collector = None
```
**EN:** Implements the apply storage runtime config routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的apply存储runtime config例程。 该方法属于 `HiMambaRadixCache`。

### Lines 1271-1366: attach_storage_backend implementation / attach_storage_backend 实现
```python
    def attach_storage_backend(
        self,
        storage_backend: str,
        storage_backend_extra_config_json: Optional[str] = None,
        served_model_name: Optional[str] = None,
        hicache_storage_prefetch_policy: Optional[str] = None,
        hicache_write_policy: Optional[str] = None,
    ) -> tuple[bool, str]:
        if hicache_storage_prefetch_policy is not None:
            allowed = ["best_effort", "wait_complete", "timeout"]
            if hicache_storage_prefetch_policy not in allowed:
                return (
# ... omitted for brevity ...
            enable_storage_metrics=self._enable_metrics_flag,
            extra_metric_labels=self.extra_metric_labels,
        )
        return True, "Attached HiCache storage backend successfully."
```
**EN:** Implements the attach storage backend routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的attach存储后端例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1368-1383: detach_storage_backend implementation / detach_storage_backend 实现
```python
    def detach_storage_backend(self) -> tuple:
        try:
            self._drain_storage_control_queues_local()
            self.cache_controller.detach_storage_backend()
        except Exception as e:
            logger.exception("Failed to detach storage backend.")
            return False, f"Failed to detach HiCache storage backend: {e}"

        self._drain_storage_control_queues_local()
        self._force_release_pending_storage_ops()

        self.enable_storage = False
        self.enable_storage_metrics = False
        if hasattr(self, "storage_metrics_collector"):
            self.storage_metrics_collector = None
        return True, "Detached HiCache storage backend successfully."
```
**EN:** Implements the detach storage backend routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的detach存储后端例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1385-1391: prefetch_abort implementation / prefetch_abort 实现
```python
    def prefetch_abort(self, pool_transfers: Optional[list[PoolTransfer]]) -> None:
        """Free any allocated mamba host slots on prefetch abort/revoke."""
        for transfer in pool_transfers or []:
            if transfer.name == PoolName.MAMBA:
                if transfer.host_indices is not None:
                    self.mamba_pool_host.free(transfer.host_indices)
                break
```
**EN:** Free any allocated mamba host slots on prefetch abort/revoke. Implements the prefetch abort routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的prefetch abort例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1393-1443: _force_release_pending_storage_ops implementation / _force_release_pending_storage_ops 实现
```python
    def _force_release_pending_storage_ops(self):
        cc = self.cache_controller

        try:
            for req_id, info in list(self.ongoing_prefetch.items()):
                try:
                    last_host_node, token_ids, host_indices, _operation = info
                except Exception:
                    self.ongoing_prefetch.pop(req_id, None)
                    continue
                try:
                    if host_indices is not None:
# ... omitted for brevity ...
                    )
                self.ongoing_backup.pop(ack_id, None)
        except Exception:
            logger.exception("Force release pending backup ops failed.")
```
**EN:** Implements the force release pending storage OPS routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的force release pending存储ops例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1445-1451: _drain_storage_control_queues_local implementation / _drain_storage_control_queues_local 实现
```python
    def _drain_storage_control_queues_local(self):
        self._drain_storage_control_queues_impl(
            n_revoke=None,
            n_backup=None,
            n_release=None,
            log_metrics=False,
        )
```
**EN:** Implements the drain storage control queues local routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的drain存储control queues local例程。 该方法属于 `HiMambaRadixCache`。

### Lines 1453-1505: _drain_storage_control_queues_impl implementation / _drain_storage_control_queues_impl 实现
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
**EN:** Implements the drain storage control queues impl routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drain存储control queues impl例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1507-1583: _parse_storage_backend_extra_config implementation / _parse_storage_backend_extra_config 实现
```python
    def _parse_storage_backend_extra_config(
        self, storage_backend_extra_config: Optional[str]
    ):
        extra_config = {}
        if storage_backend_extra_config:
            try:
                if storage_backend_extra_config.startswith("@"):
                    path = storage_backend_extra_config[1:]
                    ext = os.path.splitext(path)[1].lower()
                    with open(path, "rb" if ext == ".toml" else "r") as f:
                        if ext == ".json":
                            extra_config = json.load(f)
# ... omitted for brevity ...
            prefetch_threshold,
            prefetch_timeout_config,
            hicache_storage_pass_prefix_keys,
        )
```
**EN:** Implements the parse storage backend extra config routine for this scope. It belongs to `HiMambaRadixCache`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的parse存储后端extra config例程。 该方法属于 `HiMambaRadixCache`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 1585-1606: clear_storage_backend implementation / clear_storage_backend 实现
```python
    def clear_storage_backend(self) -> bool:
        if self.enable_storage:
            try:
                if hasattr(self.cache_controller.storage_backend, "clear"):
                    self.cache_controller.storage_backend.clear()
                    logger.info(
                        "Hierarchical cache storage backend cleared successfully!"
                    )
                    return True
                else:
                    logger.warning(
                        f"Storage backend "
# ... omitted for brevity ...
                return False
        else:
            logger.warning("Hierarchical cache storage backend is not enabled.")
            return False
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1608-1630: drain_storage_control_queues implementation / drain_storage_control_queues 实现
```python
    def drain_storage_control_queues(self):
        cc = self.cache_controller

        qsizes = torch.tensor(
            [
                cc.prefetch_revoke_queue.qsize(),
                cc.ack_backup_queue.qsize(),
                cc.host_mem_release_queue.qsize(),
            ],
            dtype=torch.int,
        )
        if self.tp_world_size > 1:
# ... omitted for brevity ...
            n_backup=n_backup,
            n_release=n_release,
            log_metrics=True,
        )
```
**EN:** Implements the drain storage control queues routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的drain存储control queues例程。 该方法属于 `HiMambaRadixCache`。

### Lines 1632-1636: _prefetch_timeout_check_linear_func implementation / _prefetch_timeout_check_linear_func 实现
```python
    def _prefetch_timeout_check_linear_func(self, operation: PrefetchOperation):
        cfg = self.prefetch_timeout_config
        num_tokens = len(operation.hash_value) * self.page_size
        timeout = min(cfg.max, cfg.base + cfg.per_ki_token * num_tokens / 1024)
        return time.monotonic() - operation.start_time > timeout
```
**EN:** Implements the prefetch timeout check linear func routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch timeout check linear func例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1638-1672: can_terminate_prefetch implementation / can_terminate_prefetch 实现
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
            can_terminate = states[0].item() == 0
            operation_terminated = states[1].item() == 1
        can_terminate = can_terminate or operation_terminated
        return can_terminate
```
**EN:** Implements the CAN terminate prefetch routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的can terminate prefetch例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1674-1681: terminate_prefetch implementation / terminate_prefetch 实现
```python
    def terminate_prefetch(self, req_id: str):
        if req_id not in self.ongoing_prefetch:
            return

        _, _, _, operation = self.ongoing_prefetch[req_id]
        if operation.host_indices is None:
            return
        operation.mark_terminate()
```
**EN:** Implements the terminate prefetch routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的terminate prefetch例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1683-1684: pop_prefetch_loaded_tokens implementation / pop_prefetch_loaded_tokens 实现
```python
    def pop_prefetch_loaded_tokens(self, req_id: str) -> int:
        return self.prefetch_loaded_tokens_by_reqid.pop(req_id, 0)
```
**EN:** Removes and returns a selected value. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 移除并返回指定值。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1686-1702: write_backup_storage implementation / write_backup_storage 实现
```python
    def write_backup_storage(self, node: TreeNode):
        prefix_keys = (
            node.get_prefix_hash_values(node.parent)
            if self.hicache_storage_pass_prefix_keys
            else None
        )
        extra_pools = self.mamba_archive_transfers(node)
        operation_id = self.cache_controller.write_storage(
            node.host_value,
            node.key,
            node.hash_value,
            prefix_keys,
            extra_pools=extra_pools,
        )
        mamba_host_protected = extra_pools is not None
        self.ongoing_backup[operation_id] = (node, mamba_host_protected)
        self._protect_host_node(node, protect_mamba=mamba_host_protected)
```
**EN:** Implements the write backup storage routine for this scope. It belongs to `HiMambaRadixCache`.
**CN:** 实现当前作用域中的write backup存储例程。 该方法属于 `HiMambaRadixCache`。

### Lines 1704-1761: prefetch_from_storage implementation / prefetch_from_storage 实现
```python
    def prefetch_from_storage(
        self,
        req_id: str,
        last_host_node: TreeNode,
        new_input_tokens: List[int],
        last_hash: Optional[str] = None,
        prefix_keys: Optional[List[str]] = None,
    ):
        prefetch_length = len(new_input_tokens) - (
            len(new_input_tokens) % self.page_size
        )
        new_input_tokens = new_input_tokens[:prefetch_length]
# ... omitted for brevity ...
            host_indices,
            operation,
        )
        self.cache_controller.prefetch_tokens_occupied += len(new_input_tokens)
```
**EN:** Implements the prefetch from storage routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch from存储例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1763-1849: check_prefetch_progress implementation / check_prefetch_progress 实现
```python
    def check_prefetch_progress(self, req_id: str) -> bool:
        if req_id not in self.ongoing_prefetch:
            return True

        last_host_node, token_ids, host_indices, operation = self.ongoing_prefetch[
            req_id
        ]

        if operation.host_indices is None:
            return True

        if not self.can_terminate_prefetch(operation):
# ... omitted for brevity ...
                int(mamba_loaded),
            )

        return True
```
**EN:** Implements the check prefetch progress routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的check prefetch progress例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1851-1906: _insert_helper_host implementation / _insert_helper_host 实现
```python
    def _insert_helper_host(
        self,
        node: TreeNode,
        key: RadixKey,
        host_value,
        hash_value,
        mamba_host_value: Optional[torch.Tensor] = None,
        mamba_loaded: bool = False,
    ):
        node.last_access_time = get_last_access_time()
        if len(key) == 0:
            return 0
# ... omitted for brevity ...
            leaf_node.mamba_host_value = mamba_host_value.clone()
            if not self.mamba_host_lru_list.in_list(leaf_node):
                self.mamba_host_lru_list.insert_mru(leaf_node)
        return matched_length
```
**EN:** Implements the insert helper host routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper主机侧例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1908-1925: release_aborted_request implementation / release_aborted_request 实现
```python
    def release_aborted_request(self, rid: str):
        self.prefetch_loaded_tokens_by_reqid.pop(rid, None)

        if rid not in self.ongoing_prefetch:
            return

        last_host_node, token_ids, host_indices, operation = self.ongoing_prefetch[rid]
        if operation.host_indices is None:
            return

        completed_tokens, _ = self.cache_controller.terminate_prefetch(operation)
        if self.tp_world_size > 1:
            torch.distributed.barrier(group=self.tp_group)
        self._release_host_node(last_host_node)
        del self.ongoing_prefetch[rid]
        self.cache_controller.append_host_mem_release(host_indices[:completed_tokens])
        self.prefetch_abort(operation.pool_transfers)
        self.cache_controller.prefetch_tokens_occupied -= len(token_ids)
```
**EN:** Implements the release aborted request routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的release aborted请求例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1927-1955: _flush_pending_storage_backups_before_reset implementation / _flush_pending_storage_backups_before_reset 实现
```python
    def _flush_pending_storage_backups_before_reset(self) -> None:
        if not self.enable_storage:
            return

        self.writing_check(write_back=True)
        deadline = time.monotonic() + 30.0
        while time.monotonic() < deadline:
            self.drain_storage_control_queues()
            backup_qsize = self.cache_controller.backup_queue.qsize()
            ack_backup_qsize = self.cache_controller.ack_backup_queue.qsize()
            ongoing_backup = len(self.ongoing_backup)
            ongoing_write = len(self.ongoing_write_through)
# ... omitted for brevity ...
            len(self.ongoing_backup),
            self.cache_controller.backup_queue.qsize(),
            self.cache_controller.ack_backup_queue.qsize(),
        )
```
**EN:** Implements the flush pending storage backups before reset routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的刷新pending存储backups before reset例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1957-1975: _alloc_with_evict implementation / _alloc_with_evict 实现
```python
    def _alloc_with_evict(
        self,
        pool,
        size: int,
        evict_fn,
        lock_node: Optional[TreeNode] = None,
        error_message: Optional[str] = None,
    ) -> Optional[torch.Tensor]:
        indices = pool.alloc(size)
        if indices is None:
            if lock_node is not None:
                self.inc_lock_ref(lock_node)
# ... omitted for brevity ...
                self.dec_lock_ref(lock_node)
        if indices is None and error_message is not None:
            raise RuntimeError(error_message)
        return indices
```
**EN:** Implements the alloc with evict routine for this scope. It belongs to `HiMambaRadixCache`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的alloc with evict例程。 该方法属于 `HiMambaRadixCache`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 1976-1978: Comment block / 注释块
```python

    # -- mamba PoolTransfer builders (D↔H↔S) ----------------------------------

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1979-1989: mamba_backup_transfers implementation / mamba_backup_transfers 实现
```python
    def mamba_backup_transfers(self, node: TreeNode) -> Optional[list[PoolTransfer]]:
        # build D→H transfer descriptor for mamba state
        if node.mamba_value is None:
            return None
        return [
            PoolTransfer(
                name=PoolName.MAMBA,
                host_indices=node.mamba_host_value,
                device_indices=node.mamba_value,
            )
        ]
```
**EN:** Implements the mamba backup transfers routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba backup transfers例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 1991-2000: mamba_backup_commit implementation / mamba_backup_commit 实现
```python
    def mamba_backup_commit(
        self, node: TreeNode, transfers: list[PoolTransfer]
    ) -> None:
        # store auto-allocated mamba host indices into the node after D→H backup
        if not transfers:
            return
        host_indices = transfers[0].host_indices
        if node.mamba_host_value is None and host_indices is not None:
            node.mamba_host_value = host_indices.clone()
            self.mamba_host_lru_list.insert_mru(node)
```
**EN:** Implements the mamba backup commit routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba backup commit例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 2002-2013: mamba_archive_transfers implementation / mamba_archive_transfers 实现
```python
    def mamba_archive_transfers(self, node: TreeNode) -> Optional[list[PoolTransfer]]:
        # build H→Storage transfer descriptor for mamba state
        if node.mamba_host_value is None or not node.hash_value:
            return None
        return [
            PoolTransfer(
                name=PoolName.MAMBA,
                host_indices=node.mamba_host_value,
                keys=[node.hash_value[-1]],
                hit_policy=PoolHitPolicy.TRAILING_PAGES,
            )
        ]
```
**EN:** Implements the mamba archive transfers routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba archive transfers例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 2015-2036: mamba_prefetch_alloc implementation / mamba_prefetch_alloc 实现
```python
    def mamba_prefetch_alloc(
        self,
        token_ids: List[int],
        last_hash: Optional[str],
    ) -> Optional[list[PoolTransfer]]:
        # allocate a mamba host slot and build Storage→H transfer descriptor
        if not token_ids:
            return None
        host_indices = self._alloc_with_evict(
            self.mamba_pool_host, 1, self.evict_mamba_host
        )
        if host_indices is None:
# ... omitted for brevity ...
                keys=["__placeholder__"],
                hit_policy=PoolHitPolicy.TRAILING_PAGES,
            )
        ]
```
**EN:** Implements the mamba prefetch alloc routine for this scope. It belongs to `HiMambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba prefetch alloc例程。 该方法属于 `HiMambaRadixCache`。它会向调用方返回计算结果。

### Lines 2038-2082: mamba_restore_transfers implementation / mamba_restore_transfers 实现
```python
    def mamba_restore_transfers(
        self,
        last_hit_node: TreeNode,
        nodes_to_restore: list[TreeNode],
        req,
    ) -> Optional[list[PoolTransfer]]:
        # build H→D transfer descriptors for mamba state
        backed_up_host_indices: list[torch.Tensor] = []
        for node in nodes_to_restore:
            if not node.mamba_backuped:
                continue
            backed_up_host_indices.append(node.mamba_host_value)
# ... omitted for brevity ...
                )
            )

        return transfers if transfers else None
```
**EN:** Implements the mamba restore transfers routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba restore transfers例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 2084-2097: mamba_restore_commit implementation / mamba_restore_commit 实现
```python
    def mamba_restore_commit(
        self,
        restored_nodes: list[TreeNode],
        transfers: Optional[list[PoolTransfer]],
    ) -> None:
        # write back controller-allocated device indices after H→D restore
        if not restored_nodes or not transfers or transfers[0].device_indices is None:
            return
        device_indices = transfers[0].device_indices
        offset = 0
        for node in restored_nodes:
            count = len(node.mamba_host_value)
            node.mamba_value = device_indices[offset : offset + count].clone()
            offset += count
```
**EN:** Implements the mamba restore commit routine for this scope. It belongs to `HiMambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba restore commit例程。 该方法属于 `HiMambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`HostLRUList`**: Defines the `HostLRUList` type and its core responsibilities. / 定义 `HostLRUList` 类型及其核心职责。
- **`HiMambaRadixCache`**: Defines the `HiMambaRadixCache` type and its core responsibilities. / 定义 `HiMambaRadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `atexit`, `heapq`, `json`, `logging`, `os`, `threading`, `time`, `queue`, `typing`, `torch`, `tomllib`
- **Internal / 内部**: `sglang.srt.disaggregation.kv_events`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller`, `sglang.srt.mem_cache.hybrid_cache.hybrid_pool_assembler`, `sglang.srt.mem_cache.mamba_radix_cache`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.utils`, `sglang.srt.observability.metrics_collector`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.server_args`
