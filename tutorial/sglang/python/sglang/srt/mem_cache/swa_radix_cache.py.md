# swa_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/swa_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the SWA radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的SWA基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47: Imports and setup / 导入与初始化
```python
from __future__ import annotations

"""
Copyright 2023-2024 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
# ... omitted for brevity ...
from sglang.srt.mem_cache.events import KVCacheEventMixin
from sglang.srt.mem_cache.radix_cache import RadixKey
from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator
from sglang.srt.mem_cache.utils import split_node_hash_value
```
**EN:** Imports `__future__`, `heapq`, `time`, `collections`, `typing`, `torch` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `heapq`, `time`, `collections`, `typing`, `torch` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 49-50: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 52-52: Imports and setup / 导入与初始化
```python
import logging
```
**EN:** Imports `logging` and other helpers used by the surrounding scope.
**CN:** 导入 `logging` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 54-54: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 57-58: TreeNode declaration / TreeNode 声明
```python
class TreeNode:

```
**EN:** Declares the `TreeNode` class.
**CN:** 声明 `TreeNode` 类。

### Lines 59-61: Shared state definitions / 共享状态定义
```python
    counter = 0
    swa_uuid_counter = 1
    last_access_time_counter_float = float64(1.0)
```
**EN:** Defines class-level variables such as `counter`, `swa_uuid_counter`, `last_access_time_counter_float`.
**CN:** 定义类级变量，例如 `counter`, `swa_uuid_counter`, `last_access_time_counter_float`。

### Lines 63-94: __init__ implementation / __init__ 实现
```python
    def __init__(self, id: Optional[int] = None):
        self.children = defaultdict(TreeNode)
        self.parent: TreeNode = None
        self.key: RadixKey = None
        self.value: Optional[torch.Tensor] = None
        # swa_tombstone is used to indicate the kv indices have been freed for swa layers
        self.swa_tombstone = False
        # invariant: for any node, if swa_lock_ref is locked, full_lock_ref must be locked;
        # if full_lock_ref is locked, swa_lock_ref doesn't need to be locked. So,
        # full_lock_ref is always >= swa_lock_ref.
        self.full_lock_ref = 0
        self.swa_lock_ref = 0
# ... omitted for brevity ...

        self.id = TreeNode.counter if id is None else id
        TreeNode.counter += 1
        self.swa_uuid = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `TreeNode`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `TreeNode`。

### Lines 96-98: evicted implementation / evicted 实现
```python
    @property
    def evicted(self):
        return self.value is None
```
**EN:** Implements the evicted routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evicted例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 100-102: backuped implementation / backuped 实现
```python
    @property
    def backuped(self):
        return self.host_value is not None
```
**EN:** Implements the backuped routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backuped例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 104-105: __lt__ implementation / __lt__ 实现
```python
    def __lt__(self, other: "TreeNode"):
        return self.last_access_time < other.last_access_time
```
**EN:** Implements the LT routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的lt例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 108-110: gen_swa_uuid implementation / gen_swa_uuid 实现
```python
def gen_swa_uuid() -> int:
    TreeNode.swa_uuid_counter += 1
    return TreeNode.swa_uuid_counter
```
**EN:** Implements the GEN SWA UUID routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的gen SWA UUID例程。它会向调用方返回计算结果。

### Lines 113-116: get_last_access_time implementation / get_last_access_time 实现
```python
def get_last_access_time() -> float64:
    ret = TreeNode.last_access_time_counter_float
    TreeNode.last_access_time_counter_float += 1.0
    return ret
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 119-119: LRUList declaration / LRUList 声明
```python
class LRUList:
```
**EN:** Declares the `LRUList` class.
**CN:** 声明 `LRUList` 类。

### Lines 120-135: __init__ implementation / __init__ 实现
```python
    def __init__(self, is_swa_list: bool = False):
        self.is_swa_list = is_swa_list
        if self.is_swa_list:
            self.prv = "swa_prev"
            self.nxt = "swa_next"
            self.lock_ref = "swa_lock_ref"
        else:
            self.prv = "prev"
            self.nxt = "next"
            self.lock_ref = "full_lock_ref"
        # Initialize dummy head and tail nodes
        self.head = TreeNode()  # Most recently used side
        self.tail = TreeNode()  # Least recently used side
        setattr(self.head, self.nxt, self.tail)  # self.head.next = self.tail
        setattr(self.tail, self.prv, self.head)  # self.tail.prev = self.head
        self.cache = {}
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `LRUList`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `LRUList`。

### Lines 137-139: _add_node implementation / _add_node 实现
```python
    def _add_node(self, node):
        """Helper to add node right after head (most recently used)"""
        self._add_node_after(self.head, node)
```
**EN:** Helper to add node right after head (most recently used) Implements the ADD node routine for this scope. It belongs to `LRUList`.
**CN:** 实现当前作用域中的add node例程。 该方法属于 `LRUList`。

### Lines 141-150: _add_node_after implementation / _add_node_after 实现
```python
    def _add_node_after(self, old_node, new_node):
        """Helper to add node right after old_node"""
        setattr(new_node, self.prv, old_node)  # new_node.prev = old_node
        setattr(
            new_node, self.nxt, getattr(old_node, self.nxt)
        )  # new_node.next = old_node.next
        setattr(
            getattr(old_node, self.nxt), self.prv, new_node
        )  # old_node.next.prev = new_node
        setattr(old_node, self.nxt, new_node)  # old_node.next = new_node
```
**EN:** Helper to add node right after old_node Implements the ADD node after routine for this scope. It belongs to `LRUList`.
**CN:** 实现当前作用域中的add node after例程。 该方法属于 `LRUList`。

### Lines 152-159: _remove_node implementation / _remove_node 实现
```python
    def _remove_node(self, node):
        """Helper to remove node from linked list"""
        setattr(
            getattr(node, self.prv), self.nxt, getattr(node, self.nxt)
        )  # node.prev.next = node.next
        setattr(
            getattr(node, self.nxt), self.prv, getattr(node, self.prv)
        )  # node.next.prev = node.prev
```
**EN:** Helper to remove node from linked list Implements the remove node routine for this scope. It belongs to `LRUList`.
**CN:** 实现当前作用域中的remove node例程。 该方法属于 `LRUList`。

### Lines 161-167: _get_lru implementation / _get_lru 实现
```python
    def _get_lru(self) -> Optional[TreeNode]:
        """
        Get the least recently used node
        """
        if len(self.cache) == 0:
            return None
        return getattr(self.tail, self.prv)
```
**EN:** Get the least recently used node Implements the get LRU routine for this scope. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get lru例程。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 169-178: reset_node_mru implementation / reset_node_mru 实现
```python
    def reset_node_mru(self, node):
        """
        Move a (existing) node to most recently used position
        """
        assert node.id in self.cache, f"Resetting node {node.id=} not in lru list"
        assert (
            not self.is_swa_list or not node.swa_tombstone
        ), f"Resetting swa tombstone node in swa lru list: {node.id=}"
        self._remove_node(node)
        self._add_node(node)
```
**EN:** Move a (existing) node to most recently used position Resets the component back to its starting state. It belongs to `LRUList`.
**CN:** 将组件恢复到初始状态。 该方法属于 `LRUList`。

### Lines 180-195: reset_node_and_parents_mru implementation / reset_node_and_parents_mru 实现
```python
    def reset_node_and_parents_mru(self, node, root_node):
        """
        Move an (existing) node and its parents to most recently used position. Child node is
        more recently used than parent node.
        """
        prev_node = self.head
        while node != root_node:
            # for swa lru list, only reset non-tombstone nodes
            if not self.is_swa_list or not node.swa_tombstone:
                assert (
                    node.id in self.cache
                ), f"Resetting node {node.id=} not in lru list when resetting node and parents mru"
                self._remove_node(node)
                self._add_node_after(prev_node, node)
                prev_node = node
            node = node.parent
```
**EN:** Move an (existing) node and its parents to most recently used position. Resets the component back to its starting state. It belongs to `LRUList`. The implementation iterates over inputs or managed entries.
**CN:** 将组件恢复到初始状态。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。

### Lines 197-208: insert_mru implementation / insert_mru 实现
```python
    def insert_mru(self, node):
        """
        Insert a (new) node as most recently used
        """
        assert (
            not self.is_swa_list or not node.swa_tombstone
        ), f"Inserting swa tombstone node in swa lru list: {node.id=}"
        assert (
            node.id not in self.cache
        ), f"Inserting node {node.id=} already in lru list, existing node: {self.cache[node.id].id=}"
        self.cache[node.id] = node
        self._add_node(node)
```
**EN:** Insert a (new) node as most recently used Inserts new data into the managed structure. It belongs to `LRUList`.
**CN:** 向受管结构中插入新数据。 该方法属于 `LRUList`。

### Lines 210-219: remove_node implementation / remove_node 实现
```python
    def remove_node(self, node: TreeNode):
        """
        Remove node from lru list
        """
        assert node.id in self.cache, f"Removing node {node.id=} not in lru list"
        assert (
            not self.is_swa_list or not node.swa_tombstone
        ), f"Removing swa tombstone node from swa lru list: {node.id=}"
        del self.cache[node.id]
        self._remove_node(node)
```
**EN:** Remove node from lru list Removes selected data from the managed structure. It belongs to `LRUList`.
**CN:** 从受管结构中删除选定数据。 该方法属于 `LRUList`。

### Lines 221-225: get_lru_no_lock implementation / get_lru_no_lock 实现
```python
    def get_lru_no_lock(self) -> Optional[TreeNode]:
        """
        Get the least recently used node that is not locked
        """
        return self.get_prev_no_lock(self.tail, check_id=False)
```
**EN:** Get the least recently used node that is not locked Retrieves the requested data or state from the current object. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 227-231: get_leaf_lru_no_lock implementation / get_leaf_lru_no_lock 实现
```python
    def get_leaf_lru_no_lock(self) -> Optional[TreeNode]:
        """
        Get the least recently used leaf node that is not locked
        """
        return self.get_prev_leaf_no_lock(self.tail, check_id=False)
```
**EN:** Get the least recently used leaf node that is not locked Retrieves the requested data or state from the current object. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 233-249: get_prev_no_lock implementation / get_prev_no_lock 实现
```python
    def get_prev_no_lock(
        self, node: TreeNode, check_id: bool = True
    ) -> Optional[TreeNode]:
        """
        Get the previous (i.e. more recently used) node that is not locked
        """
        if check_id:
            assert (
                node.id in self.cache
            ), f"Getting prev of node {node.id=} not in lru list"
        x = getattr(node, self.prv)  # x = node.prev
        while getattr(x, self.lock_ref) > 0:
            x = getattr(x, self.prv)  # x = x.prev
        # if x is the head, it means there is no node in the lru list without lock
        if x == self.head:
            return None
        return x
```
**EN:** Get the previous (i.e. Retrieves the requested data or state from the current object. It belongs to `LRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 251-265: get_prev_leaf_no_lock implementation / get_prev_leaf_no_lock 实现
```python
    def get_prev_leaf_no_lock(self, node: TreeNode, check_id: bool = True):
        """
        Get the previous (i.e. more recently used) leaf node that is not locked
        """
        if check_id:
            assert (
                node.id in self.cache
            ), f"Getting prev of node {node.id=} not in lru list"
        x = getattr(node, self.prv)  # x = node.prev
        while getattr(x, self.lock_ref) > 0 or len(x.children) > 0:
            x = getattr(x, self.prv)  # x = x.prev
        # if x is the head, it means there is no leaf node in the lru list without lock
        if x == self.head:
            return None
        return x
```
**EN:** Get the previous (i.e. Retrieves the requested data or state from the current object. It belongs to `LRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 267-273: in_list implementation / in_list 实现
```python
    def in_list(self, node: Optional[TreeNode]):
        """
        Check if the node is in the lru list
        """
        if not node:
            return False
        return node.id in self.cache
```
**EN:** Check if the node is in the lru list Implements the IN list routine for this scope. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的in list例程。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 274-275: Comment block / 注释块
```python

    # Note: this is expensive, only use for debug
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 276-285: sanity_check_evictable_size implementation / sanity_check_evictable_size 实现
```python
    def sanity_check_evictable_size(self):
        """
        Check the evictable size (i.e. the size of the nodes that are not locked)
        """
        node = self.get_lru_no_lock()
        evictable_size = 0
        while self.in_list(node):
            evictable_size += len(node.value)
            node = self.get_prev_no_lock(node)
        return evictable_size
```
**EN:** Check the evictable size (i.e. Implements the sanity check evictable size routine for this scope. It belongs to `LRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的sanity check evictable size例程。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 286-287: Comment block / 注释块
```python

    # Note: this is expensive, only use for debug or idle check
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 288-337: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self, tree_cache: "SWARadixCache"):
        """
        Check if the lru list is valid by rebuilding the lru list from the tree, heapifying it, and
        checking if the lru list is valid.
        """
        try:
            if self.is_swa_list:
                nodes = tree_cache._collect_nontombstone_nodes()
            else:
                nodes = tree_cache._collect_all_nodes()
            total_nodes = len(nodes)
            total_lru_plus_1 = len(self.cache) + 1
# ... omitted for brevity ...
        except Exception as e:
            msg = f"SWA Radix tree sanity check failed, ping @hanming-lu: {e}"
            logger.error(msg)
            raise Exception(msg)
```
**EN:** Check if the lru list is valid by rebuilding the lru list from the tree, heapifying it, and checking if the lru list is valid. Implements the sanity check routine for this scope. It belongs to `LRUList`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `LRUList`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 340-340: SWARadixCache declaration / SWARadixCache 声明
```python
class SWARadixCache(KVCacheEventMixin, BasePrefixCache):
```
**EN:** Declares the `SWARadixCache` class and connects it to `KVCacheEventMixin`, `BasePrefixCache`.
**CN:** 声明 `SWARadixCache` 类，并将其关联到 `KVCacheEventMixin`, `BasePrefixCache`。

### Lines 341-360: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams):
        assert isinstance(params.token_to_kv_pool_allocator, SWATokenToKVPoolAllocator)
        self.req_to_token_pool = params.req_to_token_pool
        self.token_to_kv_pool_allocator = params.token_to_kv_pool_allocator
        self.page_size = params.page_size
        self.disable = params.disable
        self.is_eagle = params.is_eagle
        self.enable_kv_cache_events = params.enable_kv_cache_events
        self.kv_event_queue = []

        if self.token_to_kv_pool_allocator:
            self.device = self.token_to_kv_pool_allocator.device
# ... omitted for brevity ...
            self.init_metrics_collector()

        self.sliding_window_size = params.sliding_window_size
        self.reset()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SWARadixCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SWARadixCache`。

### Lines 361-363: Comment block / 注释块
```python

    ##### Public API #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 364-368: supports_swa implementation / supports_swa 实现
```python
    def supports_swa(self) -> bool:
        assert (
            self.sliding_window_size is not None
        ), "sliding_window_size must be set for SWARadixCache"
        return True
```
**EN:** Implements the supports SWA routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports SWA例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 370-384: reset implementation / reset 实现
```python
    def reset(self) -> None:
        self.root_node = TreeNode()
        self.root_node.key = []
        self.root_node.value = []
        self.root_node.hash_value = []
        self.root_node.full_lock_ref = 1
        self.root_node.swa_lock_ref = 1
        self.full_evictable_size_ = 0
        self.swa_evictable_size_ = 0
        self.full_protected_size_ = 0
        self.swa_protected_size_ = 0
        # LRU lists are used to maintain the order of eviction of the nodes in the tree
        self.full_lru_list = LRUList(is_swa_list=False)
        self.swa_lru_list = LRUList(is_swa_list=True)
        self._record_all_cleared_event()
```
**EN:** Resets the component back to its starting state. It belongs to `SWARadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `SWARadixCache`。

### Lines 386-412: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        """Find the matching prefix from the radix tree.
        Args:
            params: MatchPrefixParams containing key.
        Returns:
            A tuple of a tensor of matching prefix token IDs and
            the last node that contains the prefix values. Note that
            this API can modify the internal state of the Radix tree.
            The last node create a new child if the prefix is shorter
            than the last node's value.
        """

# ... omitted for brevity ...
            )

        value, last_node, best_value_len = self._match_prefix_helper(key)
        return self._match_post_processor(params, value, last_node, best_value_len)
```
**EN:** Find the matching prefix from the radix tree. Matches input data against cached state and returns the best fit. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 414-433: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        if self.disable:
            return InsertResult(prefix_len=0)

        key = params.key
        value = params.value
        prev_prefix_len = params.prev_prefix_len
        swa_evicted_seqlen = params.swa_evicted_seqlen

        key, value = key.maybe_to_bigram_view(self.is_eagle, value)
        key = key.page_aligned(self.page_size)
        if value is not None:
# ... omitted for brevity ...
        prefix_len = self._insert_helper(
            self.root_node, key, value, prev_prefix_len, swa_evicted_seqlen
        )
        return InsertResult(prefix_len=prefix_len)
```
**EN:** Inserts new data into the managed structure. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 435-482: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True) -> None:
        """Cache request when it finishes."""
        kv_committed_len = req.pop_committed_kv_cache()
        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, :kv_committed_len
            ]
            self.token_to_kv_pool_allocator.free(kv_indices)
            return

        token_ids = (req.origin_input_ids + req.output_ids)[:kv_committed_len]
        kv_indices = self.req_to_token_pool.req_to_token[
# ... omitted for brevity ...
            DecLockRefParams(swa_uuid_for_lock=req.swa_uuid_for_lock),
            skip_swa=req.swa_prefix_lock_released,
        )
        req.swa_prefix_lock_released = False
```
**EN:** Cache request when it finishes. Implements the cache finished REQ routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 484-550: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False) -> None:
        """Cache request when it is unfinished."""
        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, : len(req.fill_ids)
            ]

            # `req.prefix_indices` will be used in `PrefillAdder::add_chunked_req` later
            req.prefix_indices = kv_indices
            return

        token_ids = req.fill_ids
# ... omitted for brevity ...
        else:
            req.prefix_indices = new_indices
        req.last_node = new_last_node
        req.swa_uuid_for_lock = swa_uuid_for_lock
```
**EN:** Cache request when it is unfinished. Implements the cache unfinished REQ routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 552-555: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self) -> None:
        self._print_helper(self.root_node, 0)
        total_size, total_swa_size = self._total_size_helper()
        print(f"#full_tokens: {total_size}, #swa_tokens: {total_swa_size}")
```
**EN:** Implements the pretty print routine for this scope. It belongs to `SWARadixCache`.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `SWARadixCache`。

### Lines 557-558: total_size implementation / total_size 实现
```python
    def total_size(self) -> Tuple[int, int]:
        return self._total_size_helper()
```
**EN:** Implements the total size routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 560-665: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        if self.disable:
            return EvictResult()
        start_time = time.perf_counter()
        full_num_tokens = params.num_tokens
        swa_num_tokens = params.swa_num_tokens
        full_num_evicted = 0
        swa_num_evicted = 0
        if full_num_tokens > 0:
            # get the least recently used leaf node that is not locked
            x = self.full_lru_list.get_leaf_lru_no_lock()

# ... omitted for brevity ...
        self.update_eviction_metrics(full_num_evicted + swa_num_evicted, start_time)
        return EvictResult(
            num_tokens_evicted=full_num_evicted, swa_num_tokens_evicted=swa_num_evicted
        )
```
**EN:** Removes cache entries according to the active policy. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 667-706: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: TreeNode) -> IncLockRefResult:
        """
        Increment the lock reference count for the node. Returns the swa_uuid_for_lock, which needs
        to be passed to dec_lock_ref.
        It locks the full_lock_ref for nodes between the [last node, root), exclusive.
        It locks the swa_lock_ref for nodes between the [last node, swa_uuid_for_lock], inclusive.
        """
        if self.disable:
            return IncLockRefResult()

        swa_lock_size = 0
        swa_uuid_for_lock = None
# ... omitted for brevity ...
                        node.swa_uuid = gen_swa_uuid()
                    swa_uuid_for_lock = node.swa_uuid
            node = node.parent
        return IncLockRefResult(swa_uuid_for_lock=swa_uuid_for_lock)
```
**EN:** Increment the lock reference count for the node. Implements the INC lock REF routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 708-755: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self,
        node: TreeNode,
        params: Optional[DecLockRefParams] = None,
        skip_swa: bool = False,
    ) -> DecLockRefResult:
        """
        Decrement the lock reference count for the node.
        It unlocks the full_lock_ref for nodes between the [last node, root), exclusive.
        It unlocks the swa_lock_ref for nodes between the [last node, swa_uuid_for_lock], inclusive.
        If swa_uuid_for_lock is None, it unlocks to the root, exclusive.

# ... omitted for brevity ...

            node = node.parent

        return DecLockRefResult()
```
**EN:** Decrement the lock reference count for the node. Implements the DEC lock REF routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 757-810: dec_swa_lock_only implementation / dec_swa_lock_only 实现
```python
    def dec_swa_lock_only(
        self, node: TreeNode, swa_uuid_for_lock: Optional[int] = None
    ):
        """
        Decrement only the swa_lock_ref (and swa_protected_size_) along the chain
        [node, swa_uuid_for_lock], inclusive. The full_lock_ref is left untouched
        so the caller's full-cache protection is preserved.

        Used to early-release the SWA portion of a request's tree lock once the
        request's decode position has advanced past the sliding window, so the
        protected window can be reclaimed.

# ... omitted for brevity ...

            if swa_uuid_for_lock and node.swa_uuid == swa_uuid_for_lock:
                break
            node = node.parent
```
**EN:** Decrement only the swa_lock_ref (and swa_protected_size_) along the chain [node, swa_uuid_for_lock], inclusive. Implements the DEC SWA lock only routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec SWA lock only例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 812-814: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self):
        self.full_lru_list.sanity_check(self)
        self.swa_lru_list.sanity_check(self)
```
**EN:** Implements the sanity check routine for this scope. It belongs to `SWARadixCache`.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `SWARadixCache`。

### Lines 816-818: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self) -> Tuple[int, int]:
        # Note: use full_evictable_size() and swa_evictable_size() instead.
        raise NotImplementedError
```
**EN:** Implements the evictable size routine for this scope. It belongs to `SWARadixCache`. It validates error cases explicitly.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `SWARadixCache`。它会显式处理错误场景。

### Lines 820-821: full_evictable_size implementation / full_evictable_size 实现
```python
    def full_evictable_size(self) -> int:
        return self.full_evictable_size_
```
**EN:** Implements the full evictable size routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整evictable size例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 823-824: swa_evictable_size implementation / swa_evictable_size 实现
```python
    def swa_evictable_size(self) -> int:
        return self.swa_evictable_size_
```
**EN:** Implements the SWA evictable size routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA evictable size例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 826-828: protected_size implementation / protected_size 实现
```python
    def protected_size(self) -> Tuple[int, int]:
        # Note: use full_protected_size() and swa_protected_size() instead.
        raise NotImplementedError
```
**EN:** Implements the protected size routine for this scope. It belongs to `SWARadixCache`. It validates error cases explicitly.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `SWARadixCache`。它会显式处理错误场景。

### Lines 830-832: full_protected_size implementation / full_protected_size 实现
```python
    def full_protected_size(self) -> int:
        # protected size refers to the size of the full cache that is locked
        return self.full_protected_size_
```
**EN:** Implements the full protected size routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整protected size例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 834-836: swa_protected_size implementation / swa_protected_size 实现
```python
    def swa_protected_size(self) -> int:
        # protected size refers to the size of the swa cache that is locked
        return self.swa_protected_size_
```
**EN:** Implements the SWA protected size routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA protected size例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 838-847: all_values_flatten implementation / all_values_flatten 实现
```python
    def all_values_flatten(self) -> torch.Tensor:
        values = []

        def _dfs_helper(node: TreeNode):
            for _, child in node.children.items():
                values.append(child.value)
                _dfs_helper(child)

        _dfs_helper(self.root_node)
        return torch.cat(values)
```
**EN:** Implements the ALL values flatten routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all values flatten例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 849-859: available_and_evictable_str implementation / available_and_evictable_str 实现
```python
    def available_and_evictable_str(self) -> str:
        full_available_size = self.token_to_kv_pool_allocator.full_available_size()
        swa_available_size = self.token_to_kv_pool_allocator.swa_available_size()
        full_evictable_size = self.full_evictable_size()
        swa_evictable_size = self.swa_evictable_size()
        return (
            f"Available full tokens: {full_available_size + full_evictable_size} ({full_available_size=} + {full_evictable_size=})\n"
            f"Available swa tokens: {swa_available_size + swa_evictable_size} ({swa_available_size=} + {swa_evictable_size=})\n"
            f"Full LRU list evictable size: {self.full_lru_list.sanity_check_evictable_size()}\n"
            f"SWA LRU list evictable size: {self.swa_lru_list.sanity_check_evictable_size()}\n"
        )
```
**EN:** Implements the available AND evictable STR routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available and evictable str例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 860-862: Comment block / 注释块
```python

    ##### Internal Helper Functions #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 863-918: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(
        self, key: RadixKey
    ) -> Tuple[List[torch.Tensor], TreeNode, int]:
        """
        SWA prefix matching helper. It factors in the sliding window size such that
        the matched node is guaranteed to either 1. connected to root without swa tombstone,
        or 2. the number of matching tokens from the matched node to the last swa tombstone
        node is greater than or equal to the sliding window size.
        """
        node = self.root_node
        child_key = key.child_key(self.page_size)

# ... omitted for brevity ...
            best_value_len = len(value)
            best_last_node = node

        return value, best_last_node, best_value_len
```
**EN:** SWA prefix matching helper. Implements the match prefix helper routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 920-929: _match_pre_processor implementation / _match_pre_processor 实现
```python
    def _match_pre_processor(self, params: MatchPrefixParams) -> Optional[RadixKey]:
        """Preprocess the key before matching."""
        key = params.key
        key, _ = key.maybe_to_bigram_view(self.is_eagle)
        if self.disable or len(key) == 0:
            return None
        key = key.page_aligned(self.page_size)
        if len(key) == 0:
            return None
        return key
```
**EN:** Preprocess the key before matching. Implements the match PRE processor routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match pre processor例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 931-965: _match_post_processor implementation / _match_post_processor 实现
```python
    def _match_post_processor(
        self,
        params: MatchPrefixParams,
        value: List[torch.Tensor],
        last_node: TreeNode,
        best_value_len: int,
    ) -> MatchResult:
        """Post-process the matched result."""
        node_update = last_node
        # update time for matched nodes, and make nodes closer to root to be least recently used
        # this allows swa to evict nodes closer to root first
        self.full_lru_list.reset_node_and_parents_mru(node_update, self.root_node)
# ... omitted for brevity ...
            last_device_node=last_node,
            last_host_node=last_node,
            best_match_node=last_node,
        )
```
**EN:** Post-process the matched result. Implements the match post processor routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match post processor例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 967-1011: _compact_single_child_chain implementation / _compact_single_child_chain 实现
```python
    def _compact_single_child_chain(self, node: TreeNode) -> None:
        # FIXME(ispobock): drifts retract pool accounting (commit 6348cb506);
        # also overwrites active swa_uuid when window > page_size. Off by
        # default via SGLANG_OPT_SWA_RADIX_CACHE_COMPACT.
        while len(node.children) == 1:
            child = next(iter(node.children.values()))
            if len(child.children) == 0:
                break
            sum_gc_full_lock_ref = sum(
                gc.full_lock_ref for gc in child.children.values()
            )
            if child.full_lock_ref > sum_gc_full_lock_ref:
# ... omitted for brevity ...

            self.full_lru_list.remove_node(child)
            if not child.swa_tombstone:
                self.swa_lru_list.remove_node(child)
```
**EN:** Implements the compact single child chain routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的compact single child chain例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1013-1048: _maybe_split_leaf_for_swa_lock implementation / _maybe_split_leaf_for_swa_lock 实现
```python
    def _maybe_split_leaf_for_swa_lock(self, leaf: TreeNode) -> TreeNode:
        """``inc_lock_ref`` protects ``len(leaf.value)`` SWA tokens for the
        leaf even though SWA only actually needs the last
        ``sliding_window_size`` tokens. With chunked prefill, leaves can be
        thousands of tokens long, which inflates ``swa_protected_size_`` by
        ~``chunked_prefill_size / sliding_window_size`` and causes premature
        SWA pool exhaustion / retract thrashing.
        """
        if (
            leaf is self.root_node
            or leaf.swa_lock_ref > 0
            or leaf.swa_tombstone
# ... omitted for brevity ...
            return leaf

        self._split_node(leaf.key, leaf, split_at)
        return leaf
```
**EN:** ``inc_lock_ref`` protects ``len(leaf.value)`` SWA tokens for the leaf even though SWA only actually needs the last ``sliding_window_size`` tokens. Implements the maybe split leaf for SWA lock routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe split leaf for SWA lock例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 1050-1087: _split_node implementation / _split_node 实现
```python
    def _split_node(self, key: RadixKey, child: TreeNode, split_len: int) -> TreeNode:
        # new_node -> child
        new_node = TreeNode()
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.swa_tombstone = child.swa_tombstone
        new_node.full_lock_ref = child.full_lock_ref
        new_node.swa_lock_ref = child.swa_lock_ref
        new_node.key = child.key[:split_len]
        assert len(new_node.key) > 0, f"new_node.key should not be empty"
        new_node.value = child.value[:split_len].clone()
        # parent inherits the swa_uuid from child for swa lock ref
# ... omitted for brevity ...
        if not new_node.swa_tombstone:
            self.swa_lru_list.insert_mru(new_node)
            self.swa_lru_list.insert_mru(child)
        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 1089-1215: _insert_helper implementation / _insert_helper 实现
```python
    def _insert_helper(
        self,
        node: TreeNode,
        key: RadixKey,
        value,
        update_kv_after_len: int,
        swa_evicted_seqlen: int = 0,
    ) -> int:
        # Update the last access time from root to leaf, so that
        # swa will tombstone the node closer to root first
        node.last_access_time = get_last_access_time()
        if node != self.root_node:
# ... omitted for brevity ...
                # inc_lock_ref only protects `sliding_window_size` tokens of SWA pool.
                self._maybe_split_leaf_for_swa_lock(new_leaf)

        return total_prefix_length
```
**EN:** Implements the insert helper routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1217-1237: _add_new_node implementation / _add_new_node 实现
```python
    def _add_new_node(
        self,
        parent: TreeNode,
        key: RadixKey,
        value: torch.Tensor,
        swa_tombstone: bool = False,
    ) -> TreeNode:
        assert len(key) > 0, f"key should not be empty"
        new_node = TreeNode()
        new_node.parent = parent
        new_node.key = key
        new_node.value = value.clone()
# ... omitted for brevity ...
            self.swa_lru_list.insert_mru(new_node)
            self.swa_evictable_size_ += len(value)
        self._record_store_event(new_node)
        return new_node
```
**EN:** Implements the ADD NEW node routine for this scope. It belongs to `SWARadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的add new node例程。 该方法属于 `SWARadixCache`。它会向调用方返回计算结果。

### Lines 1239-1261: _iteratively_delete_tombstone_leaf implementation / _iteratively_delete_tombstone_leaf 实现
```python
    def _iteratively_delete_tombstone_leaf(
        self, node: TreeNode
    ) -> Tuple[TreeNode, int]:
        full_num_evicted = 0
        while node.parent.swa_tombstone and len(node.parent.children) == 0:
            # root node is not evictable
            if node.parent == self.root_node:
                break
            # if locked, means node is in use, skip
            if node.parent.full_lock_ref > 0:
                break
            assert (
# ... omitted for brevity ...
            self._delete_tombstone_leaf(node.parent)
            node = node.parent

        return node, full_num_evicted
```
**EN:** Implements the iteratively delete tombstone leaf routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的iteratively delete tombstone leaf例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1263-1272: _delete_leaf implementation / _delete_leaf 实现
```python
    def _delete_leaf(self, node: TreeNode) -> None:
        assert len(node.children) == 0, f"leaf node has children, {node.id=}"
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"
        self.full_evictable_size_ -= len(node.key)
        # Tombstoned leaves were never (re-)added to swa_lru_list and were
        # already removed from swa_evictable_size_ when they were tombstoned.
        if not node.swa_tombstone:
            self.swa_evictable_size_ -= len(node.key)
```
**EN:** Implements the delete leaf routine for this scope. It belongs to `SWARadixCache`.
**CN:** 实现当前作用域中的delete leaf例程。 该方法属于 `SWARadixCache`。

### Lines 1274-1277: _tombstone_internal_node implementation / _tombstone_internal_node 实现
```python
    def _tombstone_internal_node(self, node: TreeNode) -> None:
        assert len(node.children) != 0, f"Cannot tombstone a leaf node, {node.id=}"
        node.swa_tombstone = True
        self.swa_evictable_size_ -= len(node.key)
```
**EN:** Implements the tombstone internal node routine for this scope. It belongs to `SWARadixCache`.
**CN:** 实现当前作用域中的tombstone internal node例程。 该方法属于 `SWARadixCache`。

### Lines 1279-1288: _delete_tombstone_leaf implementation / _delete_tombstone_leaf 实现
```python
    def _delete_tombstone_leaf(self, node: TreeNode) -> None:
        assert (
            node.swa_tombstone
        ), f"Deleting a unexpected non-tombstone leaf node, {node.id=}"
        assert len(node.children) == 0, f"leaf node has children, {node.id=}"
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"

        self.full_evictable_size_ -= len(node.key)
```
**EN:** Implements the delete tombstone leaf routine for this scope. It belongs to `SWARadixCache`.
**CN:** 实现当前作用域中的delete tombstone leaf例程。 该方法属于 `SWARadixCache`。

### Lines 1290-1300: _collect_nontombstone_nodes implementation / _collect_nontombstone_nodes 实现
```python
    def _collect_nontombstone_nodes(self) -> List[TreeNode]:
        ret_list = []
        stack = [self.root_node]

        while stack:
            cur_node = stack.pop()
            if not cur_node.swa_tombstone:
                ret_list.append(cur_node)
            stack.extend(cur_node.children.values())

        return ret_list
```
**EN:** Implements the collect nontombstone nodes routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect nontombstone nodes例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1302-1309: _collect_all_nodes implementation / _collect_all_nodes 实现
```python
    def _collect_all_nodes(self) -> List[TreeNode]:
        ret_list = []
        stack = [self.root_node]
        while stack:
            cur_node = stack.pop()
            ret_list.append(cur_node)
            stack.extend(cur_node.children.values())
        return ret_list
```
**EN:** Implements the collect ALL nodes routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect all nodes例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1311-1331: _print_helper implementation / _print_helper 实现
```python
    def _print_helper(self, node: TreeNode, indent: int) -> None:
        """Prints the radix tree in a human-readable format."""
        stack = [(node, indent)]
        while stack:
            current_node, current_indent = stack.pop()
            print(
                " " * current_indent,
                current_node.id,
                len(current_node.key),
                f"fr={current_node.full_lock_ref}",
                f"sr={current_node.swa_lock_ref}",
                f"fll={self.full_lru_list.in_list(current_node)}",
# ... omitted for brevity ...

                assert key == child.key.child_key(
                    self.page_size
                ), f"{key=}, {child.key.child_key(self.page_size)=}"
```
**EN:** Prints the radix tree in a human-readable format. Implements the print helper routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的print helper例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1333-1346: _total_size_helper implementation / _total_size_helper 实现
```python
    def _total_size_helper(self) -> Tuple[int, int]:
        total_size = 0
        total_swa_size = 0
        stack = [self.root_node]
        while stack:
            current_node = stack.pop()
            total_size += len(current_node.value)
            if not current_node.swa_tombstone:
                total_swa_size += len(current_node.value)
            for child in current_node.children.values():
                if child.evicted:
                    continue
                stack.append(child)
        return total_size, total_swa_size
```
**EN:** Implements the total size helper routine for this scope. It belongs to `SWARadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size helper例程。 该方法属于 `SWARadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`TreeNode`**: Defines the `TreeNode` type and its core responsibilities. / 定义 `TreeNode` 类型及其核心职责。
- **`gen_swa_uuid`**: Provides the `gen_swa_uuid` entry point for module-level behavior. / 提供模块级行为的 `gen_swa_uuid` 入口。
- **`get_last_access_time`**: Provides the `get_last_access_time` entry point for module-level behavior. / 提供模块级行为的 `get_last_access_time` 入口。
- **`LRUList`**: Defines the `LRUList` type and its core responsibilities. / 定义 `LRUList` 类型及其核心职责。
- **`SWARadixCache`**: Defines the `SWARadixCache` type and its core responsibilities. / 定义 `SWARadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `heapq`, `time`, `collections`, `typing`, `torch`, `numpy`, `logging`
- **Internal / 内部**: `sglang.srt.environ`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.events`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.srt.mem_cache.utils`, `sglang.srt.managers.schedule_batch`
