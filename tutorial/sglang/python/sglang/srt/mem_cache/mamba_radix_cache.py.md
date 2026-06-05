# mamba_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/mamba_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the mamba radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的Mamba基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-52: Imports and setup / 导入与初始化
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
from sglang.srt.mem_cache.memory_pool import HybridReqToTokenPool
from sglang.srt.mem_cache.radix_cache import RadixKey
from sglang.srt.mem_cache.utils import split_node_hash_value
from sglang.srt.server_args import get_global_server_args
```
**EN:** Imports `__future__`, `heapq`, `collections`, `functools`, `typing`, `torch` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `heapq`, `collections`, `functools`, `typing`, `torch` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 54-56: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 58-58: Imports and setup / 导入与初始化
```python
import logging
```
**EN:** Imports `logging` and other helpers used by the surrounding scope.
**CN:** 导入 `logging` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 60-60: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 63-64: TreeNode declaration / TreeNode 声明
```python
class TreeNode:

```
**EN:** Declares the `TreeNode` class.
**CN:** 声明 `TreeNode` 类。

### Lines 65-66: Shared state definitions / 共享状态定义
```python
    counter = 0
    last_access_time_counter_float = float64(1.0)
```
**EN:** Defines class-level variables such as `counter`, `last_access_time_counter_float`.
**CN:** 定义类级变量，例如 `counter`, `last_access_time_counter_float`。

### Lines 68-104: __init__ implementation / __init__ 实现
```python
    def __init__(self, id: Optional[int] = None):
        self.children = defaultdict(TreeNode)
        self.parent: TreeNode = None
        self.key: RadixKey = None
        self.value: Optional[torch.Tensor] = None
        self.mamba_value: Optional[torch.Tensor] = None
        self.mamba_host_value: Optional[torch.Tensor] = None
        # invariant: for any node, if mamba_lock_ref is locked, full_lock_ref must be locked;
        # if full_lock_ref is locked, mamba_lock_ref doesn't need to be locked. So,
        # full_lock_ref is always >= mamba_lock_ref.
        # for full_lock, once it is locked, its parent must be locked as well
        # for mamba_lock, it only need lock node itself
# ... omitted for brevity ...
        self.host_mamba_next = None

        self.id = TreeNode.counter if id is None else id
        TreeNode.counter += 1
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `TreeNode`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `TreeNode`。

### Lines 106-108: evicted implementation / evicted 实现
```python
    @property
    def evicted(self):
        return self.value is None
```
**EN:** Implements the evicted routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evicted例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 110-112: mamba_evicted implementation / mamba_evicted 实现
```python
    @property
    def mamba_evicted(self):
        return self.mamba_value is None
```
**EN:** Implements the mamba evicted routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba evicted例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 114-116: backuped implementation / backuped 实现
```python
    @property
    def backuped(self):
        return self.host_value is not None
```
**EN:** Implements the backuped routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backuped例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 118-120: mamba_backuped implementation / mamba_backuped 实现
```python
    @property
    def mamba_backuped(self):
        return self.mamba_host_value is not None
```
**EN:** Implements the mamba backuped routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba backuped例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 122-124: protect_host implementation / protect_host 实现
```python
    def protect_host(self):
        """Protect the host KV value from eviction."""
        self.host_ref_counter += 1
```
**EN:** Protect the host KV value from eviction. Implements the protect host routine for this scope. It belongs to `TreeNode`.
**CN:** 实现当前作用域中的protect主机侧例程。 该方法属于 `TreeNode`。

### Lines 126-131: release_host implementation / release_host 实现
```python
    def release_host(self):
        """Release the host KV value, allowing it to be evicted."""
        if self.host_ref_counter > 0:
            self.host_ref_counter -= 1
        else:
            raise RuntimeError("Host reference counter is already zero.")
```
**EN:** Release the host KV value, allowing it to be evicted. Implements the release host routine for this scope. It belongs to `TreeNode`. It validates error cases explicitly.
**CN:** 实现当前作用域中的release主机侧例程。 该方法属于 `TreeNode`。它会显式处理错误场景。

### Lines 133-135: protect_host_mamba implementation / protect_host_mamba 实现
```python
    def protect_host_mamba(self):
        """Protect the host mamba value from eviction."""
        self.host_mamba_ref_counter += 1
```
**EN:** Protect the host mamba value from eviction. Implements the protect host mamba routine for this scope. It belongs to `TreeNode`.
**CN:** 实现当前作用域中的protect主机侧Mamba例程。 该方法属于 `TreeNode`。

### Lines 137-142: release_host_mamba implementation / release_host_mamba 实现
```python
    def release_host_mamba(self):
        """Release the host mamba value, allowing it to be evicted."""
        if self.host_mamba_ref_counter > 0:
            self.host_mamba_ref_counter -= 1
        else:
            raise RuntimeError("Host mamba reference counter is already zero.")
```
**EN:** Release the host mamba value, allowing it to be evicted. Implements the release host mamba routine for this scope. It belongs to `TreeNode`. It validates error cases explicitly.
**CN:** 实现当前作用域中的release主机侧Mamba例程。 该方法属于 `TreeNode`。它会显式处理错误场景。

### Lines 144-148: get_last_hash_value implementation / get_last_hash_value 实现
```python
    def get_last_hash_value(self) -> Optional[str]:
        """Returns the hash value of the last page in this node."""
        if self.hash_value is None or len(self.hash_value) == 0:
            return None
        return self.hash_value[-1]
```
**EN:** Returns the hash value of the last page in this node. Retrieves the requested data or state from the current object. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 150-154: get_prefix_hash_values implementation / get_prefix_hash_values 实现
```python
    @lru_cache(maxsize=1)
    def get_prefix_hash_values(self, node: "TreeNode") -> List[str]:
        if node is None or node.hash_value is None:
            return []
        return node.get_prefix_hash_values(node.parent) + node.hash_value
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 156-157: __lt__ implementation / __lt__ 实现
```python
    def __lt__(self, other: "TreeNode"):
        return self.last_access_time < other.last_access_time
```
**EN:** Implements the LT routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的lt例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 160-163: get_last_access_time implementation / get_last_access_time 实现
```python
def get_last_access_time() -> float64:
    ret = TreeNode.last_access_time_counter_float
    TreeNode.last_access_time_counter_float += 1.0
    return ret
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 166-166: LRUList declaration / LRUList 声明
```python
class LRUList:
```
**EN:** Declares the `LRUList` class.
**CN:** 声明 `LRUList` 类。

### Lines 167-182: __init__ implementation / __init__ 实现
```python
    def __init__(self, mamba: bool = False):
        self.mamba = mamba
        if self.mamba:
            self.prv = "mamba_prev"
            self.nxt = "mamba_next"
            self.lock_ref = "mamba_lock_ref"
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

### Lines 184-186: _add_node implementation / _add_node 实现
```python
    def _add_node(self, node):
        """Helper to add node right after head (most recently used)"""
        self._add_node_after(self.head, node)
```
**EN:** Helper to add node right after head (most recently used) Implements the ADD node routine for this scope. It belongs to `LRUList`.
**CN:** 实现当前作用域中的add node例程。 该方法属于 `LRUList`。

### Lines 188-197: _add_node_after implementation / _add_node_after 实现
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

### Lines 199-206: _remove_node implementation / _remove_node 实现
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

### Lines 208-214: _get_lru implementation / _get_lru 实现
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

### Lines 216-225: reset_node_mru implementation / reset_node_mru 实现
```python
    def reset_node_mru(self, node):
        """
        Move a (existing) node to most recently used position
        """
        assert node.id in self.cache, f"Resetting node {node.id=} not in lru list"
        assert (
            not self.mamba or node.mamba_value is not None
        ), f"Resetting mamba tombstone node in mamba lru list: {node.id=}"
        self._remove_node(node)
        self._add_node(node)
```
**EN:** Move a (existing) node to most recently used position Resets the component back to its starting state. It belongs to `LRUList`.
**CN:** 将组件恢复到初始状态。 该方法属于 `LRUList`。

### Lines 227-241: reset_node_and_parents_mru implementation / reset_node_and_parents_mru 实现
```python
    def reset_node_and_parents_mru(self, node, root_node):
        """
        Move an (existing) node and its parents to most recently used position. Child node is
        more recently used than parent node.
        """
        prev_node = self.head
        while node != root_node:
            if not self.mamba or node.mamba_value is not None:
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

### Lines 243-254: insert_mru implementation / insert_mru 实现
```python
    def insert_mru(self, node):
        """
        Insert a (new) node as most recently used
        """
        assert (
            not self.mamba or node.mamba_value is not None
        ), f"Inserting mamba tombstone node in mamba lru list: {node.id=}"
        assert (
            node.id not in self.cache
        ), f"Inserting node {node.id=} already in lru list, existing node: {self.cache[node.id].id=}"
        self.cache[node.id] = node
        self._add_node(node)
```
**EN:** Insert a (new) node as most recently used Inserts new data into the managed structure. It belongs to `LRUList`.
**CN:** 向受管结构中插入新数据。 该方法属于 `LRUList`。

### Lines 256-265: remove_node implementation / remove_node 实现
```python
    def remove_node(self, node: TreeNode):
        """
        Remove node from lru list
        """
        assert node.id in self.cache, f"Removing node {node.id=} not in lru list"
        assert (
            not self.mamba or node.mamba_value is not None
        ), f"Removing mamba tombstone node from mamba lru list: {node.id=}"
        del self.cache[node.id]
        self._remove_node(node)
```
**EN:** Remove node from lru list Removes selected data from the managed structure. It belongs to `LRUList`.
**CN:** 从受管结构中删除选定数据。 该方法属于 `LRUList`。

### Lines 267-271: get_lru_no_lock implementation / get_lru_no_lock 实现
```python
    def get_lru_no_lock(self) -> Optional[TreeNode]:
        """
        Get the least recently used node that is not locked
        """
        return self.get_prev_no_lock(self.tail, check_id=False)
```
**EN:** Get the least recently used node that is not locked Retrieves the requested data or state from the current object. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 273-277: get_leaf_lru_no_lock implementation / get_leaf_lru_no_lock 实现
```python
    def get_leaf_lru_no_lock(self) -> Optional[TreeNode]:
        """
        Get the least recently used leaf node that is not locked
        """
        return self.get_prev_leaf_no_lock(self.tail, check_id=False)
```
**EN:** Get the least recently used leaf node that is not locked Retrieves the requested data or state from the current object. It belongs to `LRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUList`。它会向调用方返回计算结果。

### Lines 279-295: get_prev_no_lock implementation / get_prev_no_lock 实现
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

### Lines 297-311: get_prev_leaf_no_lock implementation / get_prev_leaf_no_lock 实现
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

### Lines 313-319: in_list implementation / in_list 实现
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

### Lines 321-343: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self, tree_cache: Optional["MambaRadixCache"] = None):
        """
        Pretty print the lru list
        """
        msg = f"{self.mamba=} LRU list: "
        x_lru = self._get_lru()
        while x_lru is not None and x_lru.id in self.cache:
            msg += f"[{x_lru.id}] {x_lru.last_access_time:f} -> "
            x_lru = getattr(x_lru, self.prv)
        print(msg)

        if not tree_cache:
# ... omitted for brevity ...
        while len(nodes):
            x = heapq.heappop(nodes)
            msg += f"[{x.id}] {x.last_access_time:f} -> "
        print(msg)
```
**EN:** Pretty print the lru list Implements the pretty print routine for this scope. It belongs to `LRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 344-345: Comment block / 注释块
```python

    # Note: this is expensive, only use for debug
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 346-357: sanity_check_evictable_size implementation / sanity_check_evictable_size 实现
```python
    def sanity_check_evictable_size(self):
        """
        Check the evictable size (i.e. the size of the nodes that are not locked)
        """
        node = self.get_lru_no_lock()
        evictable_size = 0
        while self.in_list(node):
            evictable_size += (
                len(node.value) if not self.mamba else len(node.mamba_value)
            )
            node = self.get_prev_no_lock(node)
        return evictable_size
```
**EN:** Check the evictable size (i.e. Implements the sanity check evictable size routine for this scope. It belongs to `LRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的sanity check evictable size例程。 该方法属于 `LRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 358-359: Comment block / 注释块
```python

    # Note: this is expensive, only use for debug or idle check
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 360-417: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self, tree_cache: "MambaRadixCache"):
        """
        Check if the lru list is valid by rebuilding the lru list from the tree, heapifying it, and
        checking if the lru list is valid.
        """
        try:
            if self.mamba:
                nodes = tree_cache._collect_nontombstone_nodes()
            else:
                nodes = tree_cache._collect_all_nodes()
            total_nodes = len(nodes)
            total_lru = len(self.cache)
# ... omitted for brevity ...
                tree_cache.pretty_print()
                tree_cache.full_lru_list.pretty_print(tree_cache)
                tree_cache.mamba_lru_list.pretty_print(tree_cache)
                raise Exception(msg)
```
**EN:** Check if the lru list is valid by rebuilding the lru list from the tree, heapifying it, and checking if the lru list is valid. Implements the sanity check routine for this scope. It belongs to `LRUList`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `LRUList`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 420-420: MambaRadixCache declaration / MambaRadixCache 声明
```python
class MambaRadixCache(KVCacheEventMixin, BasePrefixCache):
```
**EN:** Declares the `MambaRadixCache` class and connects it to `KVCacheEventMixin`, `BasePrefixCache`.
**CN:** 声明 `MambaRadixCache` 类，并将其关联到 `KVCacheEventMixin`, `BasePrefixCache`。

### Lines 421-447: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams):
        assert isinstance(
            params.token_to_kv_pool_allocator, TokenToKVPoolAllocator
        ) or isinstance(params.token_to_kv_pool_allocator, PagedTokenToKVPoolAllocator)
        self.req_to_token_pool: HybridReqToTokenPool = params.req_to_token_pool
        self.token_to_kv_pool_allocator = params.token_to_kv_pool_allocator

        self.page_size = params.page_size
        self.disable = params.disable
        self.enable_kv_cache_events = params.enable_kv_cache_events
        self.enable_mamba_extra_buffer = params.enable_mamba_extra_buffer
        self.kv_event_queue = []
# ... omitted for brevity ...
        if params.enable_metrics:
            self.init_metrics_collector()

        self.reset()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MambaRadixCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MambaRadixCache`。

### Lines 448-450: Comment block / 注释块
```python

    ##### Public API #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 451-452: supports_mamba implementation / supports_mamba 实现
```python
    def supports_mamba(self) -> bool:
        return True
```
**EN:** Implements the supports mamba routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports Mamba例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 454-468: reset implementation / reset 实现
```python
    def reset(self) -> None:
        self.root_node = TreeNode()
        self.root_node.key = RadixKey([], None)
        self.root_node.value = []
        self.root_node.hash_value = []
        self.root_node.full_lock_ref = 1
        self.root_node.mamba_lock_ref = 1
        self.full_evictable_size_ = 0
        self.mamba_evictable_size_ = 0
        self.full_protected_size_ = 0
        self.mamba_protected_size_ = 0
        # LRU lists are used to maintain the order of eviction of the nodes in the tree
        self.full_lru_list = LRUList(mamba=False)
        self.mamba_lru_list = LRUList(mamba=True)
        self._record_all_cleared_event()
```
**EN:** Resets the component back to its starting state. It belongs to `MambaRadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `MambaRadixCache`。

### Lines 470-495: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        """Find the matching prefix from the radix tree.
        Args:
            params: MatchPrefixParams containing key and optional Mamba-specific parameters.
        Returns:
            A tuple of a tensor of matching prefix token IDs and
            the last node that contains the prefix values. Note that
            this API can modify the internal state of the Radix tree.
            The last node create a new child if the prefix is shorter
            than the last node's value.
        """
        key = self._match_pre_processor(params)
# ... omitted for brevity ...
            )

        value, last_node, best_value_len = self._match_prefix_helper(key)
        return self._match_post_processor(params, value, last_node, best_value_len)
```
**EN:** Find the matching prefix from the radix tree. Matches input data against cached state and returns the best fit. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 497-511: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        if self.disable:
            return InsertResult(prefix_len=0, mamba_exist=False)

        key = params.key
        value = params.value
        mamba_value = params.mamba_value
        prev_prefix_len = params.prev_prefix_len

        if value is None:
            value = torch.tensor([x for x in key.token_ids], dtype=torch.int64)
        prefix_len, mamba_exist = self._insert_helper(
            self.root_node, key, value, mamba_value, params.chunked, prev_prefix_len
        )
        return InsertResult(prefix_len=prefix_len, mamba_exist=mamba_exist)
```
**EN:** Inserts new data into the managed structure. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 513-599: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True) -> None:
        """Cache request when it finishes."""
        kv_committed_len = req.pop_committed_kv_cache()
        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, :kv_committed_len
            ]
            self.token_to_kv_pool_allocator.free(kv_indices)
            self.req_to_token_pool.free_mamba_cache(req)
            return

        token_ids = (req.origin_input_ids + req.output_ids)[:kv_committed_len]
# ... omitted for brevity ...
                mamba_ping_pong_track_buffer_to_keep=mamba_ping_pong_track_buffer_to_keep,
            )

        self.dec_lock_ref(req.last_node)
```
**EN:** Cache request when it finishes. Implements the cache finished REQ routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 601-716: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False) -> None:
        """Cache request when it is unfinished."""

        def _skip_cache_unfinished_req(req: Req) -> None:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, : len(req.fill_ids)
            ]

            # `req.prefix_indices` will be used in `PrefillAdder::add_chunked_req` later
            req.prefix_indices = kv_indices.to(dtype=torch.int64, copy=True)
            return

# ... omitted for brevity ...
        )
        req.cache_protected_len = len(new_indices)
        req.mamba_last_track_seqlen = None
        req.last_node = new_last_node
```
**EN:** Cache request when it is unfinished. Implements the cache unfinished REQ routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 718-721: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self) -> None:
        self._print_helper(self.root_node, 0)
        total_size, total_mamba_size = self._total_size_helper()
        print(f"#full_tokens: {total_size}, #mamba_num: {total_mamba_size}")
```
**EN:** Implements the pretty print routine for this scope. It belongs to `MambaRadixCache`.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `MambaRadixCache`。

### Lines 723-724: total_size implementation / total_size 实现
```python
    def total_size(self) -> Tuple[int, int]:
        return self._total_size_helper()
```
**EN:** Implements the total size routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 726-755: _evict_leaf_node implementation / _evict_leaf_node 实现
```python
    def _evict_leaf_node(
        self, x: TreeNode, is_evict_mamba: bool
    ) -> Tuple[int, int, TreeNode, TreeNode]:
        assert (
            x.full_lock_ref == 0 and x.mamba_lock_ref == 0
        ), f"evict leaf node invalid with {x.id=} {x.full_lock_ref=} {x.mamba_lock_ref=}"

        assert x.mamba_value is not None, f"leaf node mamba value is not None, {x.id=}"
        # 1. a leaf node, free full tokens and mamba
        self._record_remove_event(x)
        self.token_to_kv_pool_allocator.free(x.value)
        full_num_evicted = len(x.value)
# ... omitted for brevity ...
        # 4. Iteratively delete tombstone leaves to maintain invariant that leaf nodes are not tombstone
        x, leaf_full_num_evicted = self._iteratively_delete_tombstone_leaf(x)
        full_num_evicted += leaf_full_num_evicted
        return full_num_evicted, mamba_num_evicted, x, x_next
```
**EN:** Implements the evict leaf node routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict leaf node例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 757-771: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        if self.disable:
            return EvictResult()

        full_num_evicted = 0
        mamba_num_evicted = 0

        if params.num_tokens > 0:
            full_num_evicted = self.evict_full(params.num_tokens)
        if params.mamba_num > 0:
            mamba_num_evicted = self.evict_mamba(params.mamba_num)

        return EvictResult(
            num_tokens_evicted=full_num_evicted, mamba_num_evicted=mamba_num_evicted
        )
```
**EN:** Removes cache entries according to the active policy. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 773-806: evict_mamba implementation / evict_mamba 实现
```python
    def evict_mamba(self, mamba_num: int) -> int:
        """Evict mamba states. Returns the number of mamba states evicted."""
        if self.disable or mamba_num <= 0:
            return 0
        # get the least recently used node that is not locked, doesn't have to be a leaf
        x = self.mamba_lru_list.get_lru_no_lock()
        mamba_num_evicted = 0
        # evict lru leaf nodes until mamba_num_tokens is reached
        while mamba_num_evicted < mamba_num and (self.mamba_lru_list.in_list(x)):
            assert x.mamba_value is not None, f"node has no mamba value, {x.id=}"
            assert (
                len(x.mamba_value) == 1
# ... omitted for brevity ...

            x = x_next

        return mamba_num_evicted
```
**EN:** Evict mamba states. Removes cache entries according to the active policy. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 808-831: evict_full implementation / evict_full 实现
```python
    def evict_full(self, full_num_tokens: int) -> int:
        """Evict full KV cache. Returns the number of tokens evicted."""
        if self.disable or full_num_tokens <= 0:
            return 0

        full_num_evicted = 0
        # get the least recently used leaf node that is not locked
        x = self.full_lru_list.get_leaf_lru_no_lock()

        while full_num_evicted < full_num_tokens and self.full_lru_list.in_list(x):
            assert (
                x != self.root_node
# ... omitted for brevity ...

            x = x_next

        return full_num_evicted
```
**EN:** Evict full KV cache. Removes cache entries according to the active policy. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 833-859: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: TreeNode) -> IncLockRefResult:
        """
        Increment the lock reference count for the node.
        It locks the full_lock_ref for nodes between the [last node, root), exclusive.
        It locks the mamba_lock_ref for current node if its mamba_value exists.
        """
        if self.disable:
            return IncLockRefResult()

        # protect mamba value in current node if it exists
        if node.mamba_value is not None:
            if node.mamba_lock_ref == 0:
# ... omitted for brevity ...
                self.full_protected_size_ += len(node.value)
            node.full_lock_ref += 1
            node = node.parent
        return IncLockRefResult()
```
**EN:** Increment the lock reference count for the node. Implements the INC lock REF routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 861-891: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: TreeNode, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        """
        Decrement the lock reference count for the node.
        It unlocks the full_lock_ref for nodes between the [last node, root), exclusive.
        It unlocks the mamba_lock_ref for current node if its mamba_value exists.
        """
        if self.disable:
            return DecLockRefResult()

        if node.mamba_value is not None:
# ... omitted for brevity ...
            node.full_lock_ref -= 1
            node = node.parent

        return DecLockRefResult()
```
**EN:** Decrement the lock reference count for the node. Implements the DEC lock REF routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 893-897: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self):
        if self.disable:
            return
        self.full_lru_list.sanity_check(self)
        self.mamba_lru_list.sanity_check(self)
```
**EN:** Implements the sanity check routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 899-901: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self) -> Tuple[int, int]:
        # Note: use full_evictable_size() and mamba_evictable_size() instead.
        raise NotImplementedError
```
**EN:** Implements the evictable size routine for this scope. It belongs to `MambaRadixCache`. It validates error cases explicitly.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `MambaRadixCache`。它会显式处理错误场景。

### Lines 903-904: full_evictable_size implementation / full_evictable_size 实现
```python
    def full_evictable_size(self) -> int:
        return self.full_evictable_size_
```
**EN:** Implements the full evictable size routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整evictable size例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 906-907: mamba_evictable_size implementation / mamba_evictable_size 实现
```python
    def mamba_evictable_size(self) -> int:
        return self.mamba_evictable_size_
```
**EN:** Implements the mamba evictable size routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba evictable size例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 909-911: protected_size implementation / protected_size 实现
```python
    def protected_size(self) -> Tuple[int, int]:
        # Note: use full_protected_size() and mamba_protected_size() instead.
        raise NotImplementedError
```
**EN:** Implements the protected size routine for this scope. It belongs to `MambaRadixCache`. It validates error cases explicitly.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `MambaRadixCache`。它会显式处理错误场景。

### Lines 913-915: full_protected_size implementation / full_protected_size 实现
```python
    def full_protected_size(self) -> int:
        # protected size refers to the size of the full cache that is locked
        return self.full_protected_size_
```
**EN:** Implements the full protected size routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整protected size例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 917-919: mamba_protected_size implementation / mamba_protected_size 实现
```python
    def mamba_protected_size(self) -> int:
        # protected size refers to the size of the mamba cache that is locked
        return self.mamba_protected_size_
```
**EN:** Implements the mamba protected size routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba protected size例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 921-930: all_values_flatten implementation / all_values_flatten 实现
```python
    def all_values_flatten(self) -> torch.Tensor:
        values = []

        def _dfs_helper(node: TreeNode):
            for _, child in node.children.items():
                values.append(child.value)
                _dfs_helper(child)

        _dfs_helper(self.root_node)
        return torch.cat(values) if len(values) > 0 else torch.tensor([])
```
**EN:** Implements the ALL values flatten routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all values flatten例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 932-942: all_mamba_values_flatten implementation / all_mamba_values_flatten 实现
```python
    def all_mamba_values_flatten(self) -> torch.Tensor:
        values = []

        def _dfs_helper(node: TreeNode):
            if node.mamba_value is not None:
                values.append(node.mamba_value)
            for _, child in node.children.items():
                _dfs_helper(child)

        _dfs_helper(self.root_node)
        return torch.cat(values) if len(values) > 0 else torch.tensor([])
```
**EN:** Implements the ALL mamba values flatten routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all Mamba values flatten例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 944-950: available_and_evictable_str implementation / available_and_evictable_str 实现
```python
    def available_and_evictable_str(self) -> str:
        full_available_size = self.token_to_kv_pool_allocator.available_size()
        full_evictable_size = self.full_evictable_size()
        return (
            f"Available full tokens: {full_available_size + full_evictable_size} ({full_available_size=} + {full_evictable_size=})\n"
            f"Full LRU list evictable size: {self.full_lru_list.sanity_check_evictable_size()}\n"
        )
```
**EN:** Implements the available AND evictable STR routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available and evictable str例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 951-953: Comment block / 注释块
```python

    ##### Internal Helper Functions #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 954-994: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(
        self, key: RadixKey
    ) -> Tuple[List[torch.Tensor], TreeNode, int]:
        """
        Mamba prefix matching helper. It factors in the sliding window size such that
        the matched node is guaranteed to either 1. connected to root without mamba tombstone,
        or 2. the number of matching tokens from the matched node to the last mamba tombstone
        node is greater than or equal to the sliding window size.
        """
        node = self.root_node
        child_key = key.child_key(self.page_size)

# ... omitted for brevity ...
            best_value_len = len(value)
            best_last_node = node

        return value, best_last_node, best_value_len
```
**EN:** Mamba prefix matching helper. Implements the match prefix helper routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 996-1003: _match_pre_processor implementation / _match_pre_processor 实现
```python
    def _match_pre_processor(self, params: MatchPrefixParams) -> Optional[RadixKey]:
        """Preprocess the key before matching."""
        key = params.key

        if self.disable or len(key) == 0:
            return None

        return key
```
**EN:** Preprocess the key before matching. Implements the match PRE processor routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match pre processor例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 1005-1078: _match_post_processor implementation / _match_post_processor 实现
```python
    def _match_post_processor(
        self,
        params: MatchPrefixParams,
        value: List[torch.Tensor],
        last_node: TreeNode,
        best_value_len: int,
    ) -> MatchResult:
        """Post-process the matched result."""
        cow_mamba = params.cow_mamba
        req = params.req

        # update time for matched nodes, and make nodes closer to root to be least recently used
# ... omitted for brevity ...
            last_host_node=last_node,
            best_match_node=last_node,
            mamba_branching_seqlen=mamba_branching_seqlen,
        )
```
**EN:** Post-process the matched result. Implements the match post processor routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match post processor例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1080-1111: _split_node implementation / _split_node 实现
```python
    def _split_node(self, key: RadixKey, child: TreeNode, split_len: int) -> TreeNode:
        # new_node -> child
        new_node = TreeNode()
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.mamba_value = None  # mamba cache can not be split
        new_node.full_lock_ref = child.full_lock_ref
        new_node.mamba_lock_ref = 0
        new_node.key = child.key[:split_len]
        new_node.value = child.value[:split_len].clone()

        # child time should be later than parent's time for mamba tombstone
# ... omitted for brevity ...
        self.full_lru_list.insert_mru(child)
        if child.mamba_value is not None:
            self.mamba_lru_list.insert_mru(child)
        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `MambaRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `MambaRadixCache`。它会向调用方返回计算结果。

### Lines 1113-1184: _insert_helper implementation / _insert_helper 实现
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
        # Update the last access time from root to leaf, so that
        # mamba will tombstone the node closer to root first
        assert mamba_value is not None, "Mamba value should not be None here."
# ... omitted for brevity ...
            self.mamba_lru_list.reset_node_mru(node)
            node.last_access_time = get_last_access_time()

        return total_prefix_length, mamba_value_exist
```
**EN:** Implements the insert helper routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1186-1208: _iteratively_delete_tombstone_leaf implementation / _iteratively_delete_tombstone_leaf 实现
```python
    def _iteratively_delete_tombstone_leaf(
        self, node: TreeNode
    ) -> Tuple[TreeNode, int]:
        full_num_evicted = 0
        while node.parent.mamba_value is None and len(node.parent.children) == 0:
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
**EN:** Implements the iteratively delete tombstone leaf routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的iteratively delete tombstone leaf例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1210-1220: _delete_leaf implementation / _delete_leaf 实现
```python
    def _delete_leaf(self, node: TreeNode) -> None:
        assert (
            node.mamba_value is not None
        ), f"Invariant violated: leaf node is a tombstone, {node.id=}"
        assert len(node.children) == 0, f"leaf node has children, {node.id=}"
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"

        self.full_evictable_size_ -= len(node.key)
        self.mamba_evictable_size_ -= len(node.mamba_value)
```
**EN:** Implements the delete leaf routine for this scope. It belongs to `MambaRadixCache`.
**CN:** 实现当前作用域中的delete leaf例程。 该方法属于 `MambaRadixCache`。

### Lines 1222-1225: _tombstone_internal_node implementation / _tombstone_internal_node 实现
```python
    def _tombstone_internal_node(self, node: TreeNode) -> None:
        assert len(node.children) != 0, f"Cannot tombstone a leaf node, {node.id=}"
        self.mamba_evictable_size_ -= len(node.mamba_value)
        node.mamba_value = None
```
**EN:** Implements the tombstone internal node routine for this scope. It belongs to `MambaRadixCache`.
**CN:** 实现当前作用域中的tombstone internal node例程。 该方法属于 `MambaRadixCache`。

### Lines 1227-1236: _delete_tombstone_leaf implementation / _delete_tombstone_leaf 实现
```python
    def _delete_tombstone_leaf(self, node: TreeNode) -> None:
        assert (
            node.mamba_value is None
        ), f"Deleting a unexpected non-tombstone leaf node, {node.id=}"
        assert len(node.children) == 0, f"leaf node has children, {node.id=}"
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"

        self.full_evictable_size_ -= len(node.key)
```
**EN:** Implements the delete tombstone leaf routine for this scope. It belongs to `MambaRadixCache`.
**CN:** 实现当前作用域中的delete tombstone leaf例程。 该方法属于 `MambaRadixCache`。

### Lines 1238-1248: _collect_nontombstone_nodes implementation / _collect_nontombstone_nodes 实现
```python
    def _collect_nontombstone_nodes(self) -> List[TreeNode]:
        ret_list = []
        stack = [self.root_node]

        while stack:
            cur_node = stack.pop()
            if cur_node.mamba_value is not None:
                ret_list.append(cur_node)
            stack.extend(cur_node.children.values())

        return ret_list
```
**EN:** Implements the collect nontombstone nodes routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect nontombstone nodes例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1250-1257: _collect_all_nodes implementation / _collect_all_nodes 实现
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
**EN:** Implements the collect ALL nodes routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect all nodes例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1259-1279: _print_helper implementation / _print_helper 实现
```python
    def _print_helper(self, node: TreeNode, indent: int) -> None:
        """Prints the radix tree in a human-readable format."""
        stack = [(node, indent)]
        while stack:
            current_node, current_indent = stack.pop()
            print(
                " " * current_indent,
                f"[{current_node.id}]",
                len(current_node.key),
                f"fr={current_node.full_lock_ref}",
                f"mr={current_node.mamba_lock_ref}",
                f"fll={self.full_lru_list.in_list(current_node)}",
# ... omitted for brevity ...

                assert key == child.key.child_key(
                    self.page_size
                ), f"{key=}, {child.key.child_key(self.page_size)=}"
```
**EN:** Prints the radix tree in a human-readable format. Implements the print helper routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的print helper例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1281-1294: _total_size_helper implementation / _total_size_helper 实现
```python
    def _total_size_helper(self) -> Tuple[int, int]:
        total_size = 0
        total_mamba_size = 0
        stack = [self.root_node]
        while stack:
            current_node = stack.pop()
            total_size += len(current_node.value)
            if current_node.mamba_value is not None:
                total_mamba_size += len(current_node.mamba_value)
            for child in current_node.children.values():
                if child.evicted:
                    continue
                stack.append(child)
        return total_size, total_mamba_size
```
**EN:** Implements the total size helper routine for this scope. It belongs to `MambaRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size helper例程。 该方法属于 `MambaRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`TreeNode`**: Defines the `TreeNode` type and its core responsibilities. / 定义 `TreeNode` 类型及其核心职责。
- **`get_last_access_time`**: Provides the `get_last_access_time` entry point for module-level behavior. / 提供模块级行为的 `get_last_access_time` 入口。
- **`LRUList`**: Defines the `LRUList` type and its core responsibilities. / 定义 `LRUList` 类型及其核心职责。
- **`MambaRadixCache`**: Defines the `MambaRadixCache` type and its core responsibilities. / 定义 `MambaRadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `heapq`, `collections`, `functools`, `typing`, `torch`, `numpy`, `logging`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.layers.attention.fla.chunk_delta_h`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.events`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.utils`, `sglang.srt.server_args`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`
