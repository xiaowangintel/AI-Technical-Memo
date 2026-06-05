# unified_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/unified_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the unified radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的统一基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
import threading
import time
from collections import defaultdict
from functools import partial
from typing import TYPE_CHECKING, Any, Optional

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
# ... omitted for brevity ...
    TreeComponent,
    get_and_increase_time_counter,
)
from sglang.srt.session.streaming_session import StreamingSession
```
**EN:** Imports `__future__`, `logging`, `threading`, `time`, `collections`, `functools` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `threading`, `time`, `collections`, `functools` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 46-49: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.server_args import ServerArgs
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 52-52: UnifiedTreeNode declaration / UnifiedTreeNode 声明
```python
class UnifiedTreeNode:
```
**EN:** Declares the `UnifiedTreeNode` class.
**CN:** 声明 `UnifiedTreeNode` 类。

### Lines 53-53: Shared state definitions / 共享状态定义
```python
    counter = 0
```
**EN:** Defines class-level variables such as `counter`.
**CN:** 定义类级变量，例如 `counter`。

### Lines 55-74: __init__ implementation / __init__ 实现
```python
    def __init__(self, tree_components: tuple[ComponentType, ...]):
        self.children = defaultdict(partial(UnifiedTreeNode, tree_components))
        self.parent: UnifiedTreeNode | None = None
        self.key: Optional[RadixKey] = None
        self.tree_components = tree_components
        # list indexed by ComponentType (int enum 0..N-1)
        self.component_data: list[ComponentData] = [
            ComponentData() for _ in range(_NUM_COMPONENT_TYPES)
        ]
        self.last_access_time = get_and_increase_time_counter()
        self.hash_value = None
        self.hit_count = 0
# ... omitted for brevity ...
            _NUM_COMPONENT_TYPES * 2
        )
        self.id = UnifiedTreeNode.counter
        UnifiedTreeNode.counter += 1
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `UnifiedTreeNode`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `UnifiedTreeNode`。

### Lines 76-77: component implementation / component 实现
```python
    def component(self, component_type: ComponentType) -> ComponentData:
        return self.component_data[component_type]
```
**EN:** Implements the component routine for this scope. It belongs to `UnifiedTreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的组件例程。 该方法属于 `UnifiedTreeNode`。它会向调用方返回计算结果。

### Lines 79-82: backuped implementation / backuped 实现
```python
    @property
    def backuped(self) -> bool:
        """Tree-level: Full KV present on host."""
        return self.component_data[ComponentType.FULL].host_value is not None
```
**EN:** Tree-level: Full KV present on host. Implements the backuped routine for this scope. It belongs to `UnifiedTreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backuped例程。 该方法属于 `UnifiedTreeNode`。它会向调用方返回计算结果。

### Lines 84-90: evicted implementation / evicted 实现
```python
    @property
    def evicted(self) -> bool:
        """Tree-level: Full KV not on device (non-root with value=None)."""
        return (
            self.parent is not None
            and self.component_data[ComponentType.FULL].value is None
        )
```
**EN:** Tree-level: Full KV not on device (non-root with value=None). Implements the evicted routine for this scope. It belongs to `UnifiedTreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evicted例程。 该方法属于 `UnifiedTreeNode`。它会向调用方返回计算结果。

### Lines 92-93: __lt__ implementation / __lt__ 实现
```python
    def __lt__(self, other: UnifiedTreeNode):
        return self.last_access_time < other.last_access_time
```
**EN:** Implements the LT routine for this scope. It belongs to `UnifiedTreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的lt例程。 该方法属于 `UnifiedTreeNode`。它会向调用方返回计算结果。

### Lines 96-96: UnifiedLRUList declaration / UnifiedLRUList 声明
```python
class UnifiedLRUList:
```
**EN:** Declares the `UnifiedLRUList` class.
**CN:** 声明 `UnifiedLRUList` 类。

### Lines 97-111: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        component_type: ComponentType,
        tree_components: tuple[ComponentType, ...],
        use_host_ptr: bool = False,
    ):
        self.component_type = component_type
        # Pointer slot: host LRU uses offset slots so device/host pointers
        # never collide on the same node.
        self._pt: int = component_type + (_NUM_COMPONENT_TYPES if use_host_ptr else 0)
        self.head = UnifiedTreeNode(tree_components)
        self.tail = UnifiedTreeNode(tree_components)
        self.head.lru_next[self._pt] = self.tail
        self.tail.lru_prev[self._pt] = self.head
        self.cache: dict[int, UnifiedTreeNode] = {}
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `UnifiedLRUList`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `UnifiedLRUList`。

### Lines 113-118: _add_node_after implementation / _add_node_after 实现
```python
    def _add_node_after(self, prev_node: UnifiedTreeNode, new_node: UnifiedTreeNode):
        pt = self._pt
        new_node.lru_prev[pt] = prev_node
        new_node.lru_next[pt] = prev_node.lru_next[pt]
        prev_node.lru_next[pt].lru_prev[pt] = new_node
        prev_node.lru_next[pt] = new_node
```
**EN:** Implements the ADD node after routine for this scope. It belongs to `UnifiedLRUList`.
**CN:** 实现当前作用域中的add node after例程。 该方法属于 `UnifiedLRUList`。

### Lines 120-121: _add_node implementation / _add_node 实现
```python
    def _add_node(self, node: UnifiedTreeNode):
        self._add_node_after(self.head, node)
```
**EN:** Implements the ADD node routine for this scope. It belongs to `UnifiedLRUList`.
**CN:** 实现当前作用域中的add node例程。 该方法属于 `UnifiedLRUList`。

### Lines 123-126: _remove_node implementation / _remove_node 实现
```python
    def _remove_node(self, node: UnifiedTreeNode):
        pt = self._pt
        node.lru_prev[pt].lru_next[pt] = node.lru_next[pt]
        node.lru_next[pt].lru_prev[pt] = node.lru_prev[pt]
```
**EN:** Implements the remove node routine for this scope. It belongs to `UnifiedLRUList`.
**CN:** 实现当前作用域中的remove node例程。 该方法属于 `UnifiedLRUList`。

### Lines 128-131: insert_mru implementation / insert_mru 实现
```python
    def insert_mru(self, node: UnifiedTreeNode):
        assert node.id not in self.cache
        self.cache[node.id] = node
        self._add_node(node)
```
**EN:** Inserts new data into the managed structure. It belongs to `UnifiedLRUList`.
**CN:** 向受管结构中插入新数据。 该方法属于 `UnifiedLRUList`。

### Lines 133-136: remove_node implementation / remove_node 实现
```python
    def remove_node(self, node: UnifiedTreeNode):
        assert node.id in self.cache
        del self.cache[node.id]
        self._remove_node(node)
```
**EN:** Removes selected data from the managed structure. It belongs to `UnifiedLRUList`.
**CN:** 从受管结构中删除选定数据。 该方法属于 `UnifiedLRUList`。

### Lines 138-141: reset_node_mru implementation / reset_node_mru 实现
```python
    def reset_node_mru(self, node: UnifiedTreeNode):
        assert node.id in self.cache
        self._remove_node(node)
        self._add_node(node)
```
**EN:** Resets the component back to its starting state. It belongs to `UnifiedLRUList`.
**CN:** 将组件恢复到初始状态。 该方法属于 `UnifiedLRUList`。

### Lines 143-156: reset_node_and_parents_mru implementation / reset_node_and_parents_mru 实现
```python
    def reset_node_and_parents_mru(
        self,
        node: UnifiedTreeNode,
        root_node: UnifiedTreeNode,
        should_include,
    ):
        prev_node = self.head
        while node != root_node:
            if should_include(node):
                assert node.id in self.cache
                self._remove_node(node)
                self._add_node_after(prev_node, node)
                prev_node = node
            node = node.parent
```
**EN:** Resets the component back to its starting state. It belongs to `UnifiedLRUList`. The implementation iterates over inputs or managed entries.
**CN:** 将组件恢复到初始状态。 该方法属于 `UnifiedLRUList`。实现过程中会遍历输入或受管条目。

### Lines 158-159: in_list implementation / in_list 实现
```python
    def in_list(self, node: Optional[UnifiedTreeNode]):
        return node is not None and node.id in self.cache
```
**EN:** Implements the IN list routine for this scope. It belongs to `UnifiedLRUList`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的in list例程。 该方法属于 `UnifiedLRUList`。它会向调用方返回计算结果。

### Lines 161-171: get_prev_no_lock implementation / get_prev_no_lock 实现
```python
    def get_prev_no_lock(self, node: UnifiedTreeNode, check_id: bool = True):
        if check_id:
            assert node.id in self.cache
        pt = self._pt
        ct = self.component_type
        x = node.lru_prev[pt]
        while x.component_data[ct].lock_ref > 0:
            x = x.lru_prev[pt]
        if x == self.head:
            return None
        return x
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `UnifiedLRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `UnifiedLRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 173-183: get_prev_leaf_no_lock implementation / get_prev_leaf_no_lock 实现
```python
    def get_prev_leaf_no_lock(self, node: UnifiedTreeNode, check_id: bool = True):
        if check_id:
            assert node.id in self.cache
        pt = self._pt
        ct = self.component_type
        x = node.lru_prev[pt]
        while x.component_data[ct].lock_ref > 0 or len(x.children) > 0:
            x = x.lru_prev[pt]
        if x == self.head:
            return None
        return x
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `UnifiedLRUList`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `UnifiedLRUList`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 185-186: get_lru_no_lock implementation / get_lru_no_lock 实现
```python
    def get_lru_no_lock(self):
        return self.get_prev_no_lock(self.tail, check_id=False)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `UnifiedLRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `UnifiedLRUList`。它会向调用方返回计算结果。

### Lines 188-189: get_leaf_lru_no_lock implementation / get_leaf_lru_no_lock 实现
```python
    def get_leaf_lru_no_lock(self):
        return self.get_prev_leaf_no_lock(self.tail, check_id=False)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `UnifiedLRUList`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `UnifiedLRUList`。它会向调用方返回计算结果。

### Lines 192-198: Shared state definitions / 共享状态定义
```python
COMPONENT_REGISTRY: dict[ComponentType, type[TreeComponent]] = {
    ComponentType.FULL: FullComponent,
    ComponentType.MAMBA: MambaComponent,
    ComponentType.SWA: SWAComponent,
}

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `COMPONENT_REGISTRY`, `logger`.
**CN:** 定义模块级变量，例如 `COMPONENT_REGISTRY`, `logger`。

### Lines 201-201: UnifiedRadixCache declaration / UnifiedRadixCache 声明
```python
class UnifiedRadixCache(BasePrefixCache):
```
**EN:** Declares the `UnifiedRadixCache` class and connects it to `BasePrefixCache`.
**CN:** 声明 `UnifiedRadixCache` 类，并将其关联到 `BasePrefixCache`。

### Lines 202-249: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        params: CacheInitParams,
    ):
        self.req_to_token_pool = params.req_to_token_pool
        self.token_to_kv_pool_allocator = params.token_to_kv_pool_allocator
        self.page_size = params.page_size
        self.disable = params.disable
        self.is_eagle = params.is_eagle

        if self.token_to_kv_pool_allocator:
            self.device = self.token_to_kv_pool_allocator.device
# ... omitted for brevity ...
        self.write_through_threshold = 256

        self.reset()
        logger.info(f"Init Unified RadixTree with components {self.tree_components}")
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `UnifiedRadixCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `UnifiedRadixCache`。

### Lines 251-252: reset implementation / reset 实现
```python
    def reset(self) -> None:
        self._reset_full()
```
**EN:** Resets the component back to its starting state. It belongs to `UnifiedRadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `UnifiedRadixCache`。

### Lines 254-296: _reset_full implementation / _reset_full 实现
```python
    def _reset_full(self) -> None:
        """Full reset: destroy entire tree and all state."""
        self.root_node = UnifiedTreeNode(self.tree_components)
        self.root_node.key = RadixKey([], None)
        self.root_node.component_data[BASE_COMPONENT_TYPE].value = []
        for ct in self.tree_components:
            self.root_node.component_data[ct].lock_ref = 1
        self.component_evictable_size_ = {ct: 0 for ct in self.tree_components}
        self.component_protected_size_ = {ct: 0 for ct in self.tree_components}

        self.lru_lists = {
            ct: UnifiedLRUList(ct, self.tree_components) for ct in self.tree_components
# ... omitted for brevity ...
            last_device_node=self.root_node,
            last_host_node=self.root_node,
            best_match_node=self.root_node,
        )
```
**EN:** Full reset: destroy entire tree and all state. Implements the reset full routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的reset完整例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 298-336: init_hicache implementation / init_hicache 实现
```python
    def init_hicache(self, server_args: ServerArgs, params: CacheInitParams) -> None:
        """Initialize HiCache infrastructure."""
        from sglang.srt.mem_cache.hybrid_cache.hybrid_pool_assembler import (
            attach_hybrid_pool_to_unified_cache,
        )

        # Direct IO layout fixup (must happen before pool creation)
        if server_args.hicache_io_backend == "direct":
            if server_args.hicache_mem_layout == "page_first":
                server_args.hicache_mem_layout = "page_first_direct"
                logger.warning(
                    "Page first layout is not supported with direct IO backend, "
# ... omitted for brevity ...
            f"write_policy={server_args.hicache_write_policy}, "
            f"tp_world_size={self.tp_world_size}, "
            f"transfer_layer_num={self.cache_controller.layer_num}"
        )
```
**EN:** Initialize HiCache infrastructure. Initializes supporting state for later operations. It belongs to `UnifiedRadixCache`.
**CN:** 初始化后续操作所需的辅助状态。 该方法属于 `UnifiedRadixCache`。

### Lines 338-339: register_sidecar_pool implementation / register_sidecar_pool 实现
```python
    def register_sidecar_pool(self, spec: SidecarPoolSpec) -> None:
        self.sidecar_pool_specs.append(spec)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `UnifiedRadixCache`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `UnifiedRadixCache`。

### Lines 341-366: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        result = self.session.try_match_prefix(params)
        if result is not None:
            return result

        key = params.key
        key, _ = key.maybe_to_bigram_view(self.is_eagle)
        if self.disable or len(key) == 0:
            return self._empty_match_result
        key = key.page_aligned(self.page_size)
        if len(key) == 0:
            return self._empty_match_result
# ... omitted for brevity ...
            best_match_node,
            best_match_device_node,
            best_match_device_value_len,
        )
```
**EN:** Matches input data against cached state and returns the best fit. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 368-382: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        if self.disable:
            return InsertResult(prefix_len=0)

        key = params.key
        value = params.value
        key, value = key.maybe_to_bigram_view(self.is_eagle, value)
        key = key.page_aligned(self.page_size)
        if value is not None:
            value = value[: len(key)]
        else:
            value = torch.tensor(key.token_ids[: len(key)], dtype=torch.int64)

        result = self._insert_helper(self.root_node, key, value, params)
        return result
```
**EN:** Inserts new data into the managed structure. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 384-404: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        if self.disable:
            return EvictResult()
        start_time = time.perf_counter()
        tracker = {ct: 0 for ct in self.tree_components}

        for component in self._components_tuple:
            component.drive_eviction(params=params, tracker=tracker)

        if (
            self.cache_controller is not None
            and self.cache_controller.write_policy == "write_back"
# ... omitted for brevity ...
            num_tokens_evicted=tracker[BASE_COMPONENT_TYPE],
            swa_num_tokens_evicted=tracker.get(ComponentType.SWA, 0),
            mamba_num_evicted=tracker.get(ComponentType.MAMBA, 0),
        )
```
**EN:** Removes cache entries according to the active policy. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 406-417: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: Any) -> IncLockRefResult:
        result = self.session.try_inc_lock_ref(node)
        if result is not None:
            return result
        if self.disable:
            return IncLockRefResult()
        result = IncLockRefResult()
        for component in self._components_tuple:
            result = component.acquire_component_lock(node=node, result=result)

        self._update_evictable_leaf_sets(node)
        return result
```
**EN:** Implements the INC lock REF routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 419-432: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: Any, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        result = self.session.try_dec_lock_ref(node, params)
        if result is not None:
            return result
        if self.disable:
            return DecLockRefResult()
        for component in self._components_tuple:
            component.release_component_lock(node=node, params=params)

        self._update_evictable_leaf_sets(node)
        # TODO: delta is not aggregated from components; no caller uses it yet.
        return DecLockRefResult()
```
**EN:** Implements the DEC lock REF routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 434-503: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True, **kwargs) -> None:
        if self.session.try_cache_finished_req(req, is_insert=is_insert, **kwargs):
            return

        kv_committed_len = req.pop_committed_kv_cache()

        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, :kv_committed_len
            ]
            self.token_to_kv_pool_allocator.free(kv_indices)
            for comp in self._components_tuple:
# ... omitted for brevity ...
        for comp in self._components_tuple:
            comp.cleanup_after_caching_req(
                req, is_finished=True, insert_result=result, insert_params=insert_params
            )
```
**EN:** Implements the cache finished REQ routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 505-599: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False, **kwargs) -> None:
        if self.session.try_cache_unfinished_req(req, chunked=chunked, **kwargs):
            return

        token_ids = req.fill_ids

        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, : len(token_ids)
            ]
            req.prefix_indices = kv_indices
            return
# ... omitted for brevity ...
                is_finished=False,
                insert_result=result,
                insert_params=insert_params,
            )
```
**EN:** Implements the cache unfinished REQ routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 600-602: Comment block / 注释块
```python

    # ---- Internal Helpers ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 603-678: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(
        self, key: RadixKey
    ) -> tuple[list[torch.Tensor], UnifiedTreeNode, UnifiedTreeNode, int]:
        # Non-HiCache mode has only device-resident matches, so the scheduler
        # device anchor follows the best match. In HiCache mode, host-backed
        # nodes can also match, so we separately track the best device-resident
        # match for scheduler prefix indices and locking.
        node = self.root_node
        child_key = key.child_key(self.page_size)
        value: list[torch.Tensor] = []
        best_match_node = node
        best_match_device_node = node
# ... omitted for brevity ...
            best_match_node,
            best_match_device_node,
            best_match_device_value_len,
        )
```
**EN:** Implements the match prefix helper routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 680-729: _match_post_processor implementation / _match_post_processor 实现
```python
    def _match_post_processor(
        self,
        params: MatchPrefixParams,
        value: list[torch.Tensor],
        best_match_node: UnifiedTreeNode,
        best_match_device_node: UnifiedTreeNode,
        best_match_device_value_len: int,
    ) -> MatchResult:
        node_update = best_match_node
        for comp in self._components_tuple:
            if comp.component_type == BASE_COMPONENT_TYPE:
                continue  # Full uses last_access_time, not LRU
# ... omitted for brevity ...
                value_chunks=value,
                best_value_len=best_match_device_value_len,
            )
        return result
```
**EN:** Implements the match post processor routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match post processor例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 731-758: _split_node implementation / _split_node 实现
```python
    def _split_node(
        self, key: RadixKey, child: UnifiedTreeNode, split_len: int
    ) -> UnifiedTreeNode:
        new_node = UnifiedTreeNode(self.tree_components)
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.key = child.key[:split_len]

        self._for_each_component_lru(child, UnifiedLRUList.remove_node)

        child.parent = new_node
        child.key = child.key[split_len:]
# ... omitted for brevity ...

        self._update_evictable_leaf_sets(new_node)
        self._update_evictable_leaf_sets(child)
        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 760-763: _touch_node implementation / _touch_node 实现
```python
    def _touch_node(self, node: UnifiedTreeNode):
        node.last_access_time = get_and_increase_time_counter()
        if node != self.root_node:
            self._for_each_component_lru(node, UnifiedLRUList.reset_node_mru)
```
**EN:** Implements the touch node routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的touch node例程。 该方法属于 `UnifiedRadixCache`。

### Lines 765-780: _add_new_node implementation / _add_new_node 实现
```python
    def _add_new_node(
        self,
        parent: UnifiedTreeNode,
        key: RadixKey,
        value: torch.Tensor,
    ) -> UnifiedTreeNode:
        new_node = UnifiedTreeNode(self.tree_components)
        new_node.parent = parent
        new_node.key = key
        new_node.component_data[BASE_COMPONENT_TYPE].value = value.clone()
        parent.children[key.child_key(self.page_size)] = new_node
        self.component_evictable_size_[BASE_COMPONENT_TYPE] += len(value)

        self._update_evictable_leaf_sets(new_node)
        self._update_evictable_leaf_sets(parent)
        return new_node
```
**EN:** Implements the ADD NEW node routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的add new node例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 782-795: _unevict_node_on_insert implementation / _unevict_node_on_insert 实现
```python
    def _unevict_node_on_insert(
        self, node: UnifiedTreeNode, fresh_value: torch.Tensor
    ) -> None:
        """Restore an evicted node's Full device value from fresh KV indices
        during insert."""
        ct = BASE_COMPONENT_TYPE
        cd = node.component_data[ct]
        assert cd.value is None
        n = len(fresh_value)
        cd.value = fresh_value.clone()
        self.component_evictable_size_[ct] += n
        self._update_evictable_leaf_sets(node)
        if node.parent is not None:
            self._update_evictable_leaf_sets(node.parent)
```
**EN:** Restore an evicted node's Full device value from fresh KV indices during insert. Implements the unevict node ON insert routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的unevict node on insert例程。 该方法属于 `UnifiedRadixCache`。

### Lines 797-891: _insert_helper implementation / _insert_helper 实现
```python
    def _insert_helper(
        self,
        node: UnifiedTreeNode,
        key: RadixKey,
        value: torch.Tensor,
        params: InsertParams,
    ) -> InsertResult:
        self._touch_node(node)
        if len(key) == 0:
            return InsertResult(prefix_len=0, mamba_exist=True)

        child_key = key.child_key(self.page_size)
# ... omitted for brevity ...
            )
        if is_new_leaf:
            self._inc_hit_count(target_node, params.chunked)
        return result
```
**EN:** Implements the insert helper routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 892-894: Comment block / 注释块
```python

    # ---- Evict Helpers ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 895-933: _cascade_evict implementation / _cascade_evict 实现
```python
    def _cascade_evict(
        self,
        node: UnifiedTreeNode,
        trigger: TreeComponent,
        tracker: dict[ComponentType, int],
        target: EvictLayer = EvictLayer.DEVICE,
    ):
        """Cascade eviction from trigger to lower-or-equal priority components."""

        is_leaf = False
        if target == EvictLayer.DEVICE:
            is_leaf = node in self.evictable_device_leaves
# ... omitted for brevity ...
        ):
            node.component_data[trigger.component_type].value = None

        self._update_evictable_leaf_sets(node)
```
**EN:** Cascade eviction from trigger to lower-or-equal priority components. Implements the cascade evict routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的cascade evict例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 935-938: _remove_leaf_from_parent implementation / _remove_leaf_from_parent 实现
```python
    def _remove_leaf_from_parent(self, node: UnifiedTreeNode):
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node
```
**EN:** Implements the remove leaf from parent routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的remove leaf from parent例程。 该方法属于 `UnifiedRadixCache`。

### Lines 940-964: _evict_component_and_detach_lru implementation / _evict_component_and_detach_lru 实现
```python
    def _evict_component_and_detach_lru(
        self,
        node: UnifiedTreeNode,
        comp: TreeComponent,
        target: EvictLayer = EvictLayer.DEVICE,
        tracker: dict[ComponentType, int] = None,
    ) -> tuple[int, int]:
        device_freed, host_freed = comp.evict_component(node, target=target)
        if tracker is not None:
            if EvictLayer.DEVICE in target:
                tracker[comp.component_type] += device_freed
            elif EvictLayer.HOST in target:
# ... omitted for brevity ...
                lru = lru_lists[ct]
                if lru.in_list(node):
                    lru.remove_node(node)
        return device_freed, host_freed
```
**EN:** Implements the evict component AND detach LRU routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict组件and detach lru例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 966-1013: _iteratively_delete_tombstone_leaf implementation / _iteratively_delete_tombstone_leaf 实现
```python
    def _iteratively_delete_tombstone_leaf(
        self, deleted_node: UnifiedTreeNode, tracker: dict[ComponentType, int]
    ):
        """Walk up from *deleted_node* and cascade-delete childless ancestors.

        Only the Full (base) component decides whether a node survives:
          - Full device present  → keep as D-leaf
          - Full host present    → keep as H-leaf
          - neither              → evict all remaining data, delete, continue up
        """
        ct = BASE_COMPONENT_TYPE
        cur = deleted_node.parent
# ... omitted for brevity ...
            self._remove_leaf_from_parent(cur)
            parent = cur.parent
            self._update_evictable_leaf_sets(parent)
            cur = parent
```
**EN:** Walk up from *deleted_node* and cascade-delete childless ancestors. Implements the iteratively delete tombstone leaf routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的iteratively delete tombstone leaf例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1015-1033: _for_each_component_lru implementation / _for_each_component_lru 实现
```python
    def _for_each_component_lru(
        self,
        node: UnifiedTreeNode,
        lru_op,
        target: EvictLayer = EvictLayer.DEVICE,
        skip_existing: bool = False,
    ):
        """Apply lru_op to each aux component's LRU that has data on this node.
        If skip_existing=True, skip components already in the target LRU list."""
        lru_dict = self.host_lru_lists if target is EvictLayer.HOST else self.lru_lists
        for ct in self.tree_components:
            if ct == BASE_COMPONENT_TYPE:
# ... omitted for brevity ...
                lru = lru_dict[ct]
                if skip_existing and lru.in_list(node):
                    continue
                lru_op(lru, node)
```
**EN:** Apply lru_op to each aux component's LRU that has data on this node. Implements the for each component LRU routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的for each组件lru例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1035-1043: evict_host implementation / evict_host 实现
```python
    def evict_host(
        self, num_tokens: int, component_type: ComponentType = BASE_COMPONENT_TYPE
    ) -> int:
        """Evict host resources for a specific component to free host pool space."""
        tracker: dict[ComponentType, int] = {ct: 0 for ct in self.tree_components}
        comp = self.components.get(component_type)
        if comp is not None:
            comp.drive_host_eviction(num_tokens, tracker)
        return tracker[component_type]
```
**EN:** Evict host resources for a specific component to free host pool space. Removes cache entries according to the active policy. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1045-1061: _is_device_leaf implementation / _is_device_leaf 实现
```python
    def _is_device_leaf(self, node: UnifiedTreeNode) -> bool:
        """D-leaf: Full device value present, no child with Full KV on device,
        unlocked, not root.

        Only the Full (base) component is required; auxiliary components
        (Mamba, SWA) are not mandatory for D-leaf membership."""
        ct = BASE_COMPONENT_TYPE
        if node is self.root_node or node.evicted:
            return False
        if any(cd.lock_ref > 0 for cd in node.component_data):
            return False
        if any(
            child.component_data[ct].value is not None
            for child in node.children.values()
        ):
            return False
        return True
```
**EN:** D-leaf: Full device value present, no child with Full KV on device, unlocked, not root. Implements the IS device leaf routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的is device leaf例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1063-1076: _is_host_leaf implementation / _is_host_leaf 实现
```python
    def _is_host_leaf(self, node: UnifiedTreeNode) -> bool:
        """H-leaf: evicted, Full host value present, no children, unlocked, not root.

        Only the Full (base) component host_value is required; auxiliary
        components are not mandatory for H-leaf membership."""
        if node is self.root_node or not node.evicted:
            return False
        if not node.backuped:
            return False
        if any(cd.host_lock_ref > 0 for cd in node.component_data):
            return False
        if len(node.children) > 0:
            return False
        return True
```
**EN:** H-leaf: evicted, Full host value present, no children, unlocked, not root. Implements the IS host leaf routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的is主机侧leaf例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1078-1088: _update_evictable_leaf_sets implementation / _update_evictable_leaf_sets 实现
```python
    def _update_evictable_leaf_sets(self, node: UnifiedTreeNode) -> None:
        """Update both device and host leaf sets for a node."""
        if self._is_device_leaf(node):
            self.evictable_device_leaves.add(node)
        else:
            self.evictable_device_leaves.discard(node)

        if self._is_host_leaf(node):
            self.evictable_host_leaves.add(node)
        else:
            self.evictable_host_leaves.discard(node)
```
**EN:** Update both device and host leaf sets for a node. Implements the update evictable leaf sets routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的update evictable leaf sets例程。 该方法属于 `UnifiedRadixCache`。

### Lines 1090-1105: _evict_to_host implementation / _evict_to_host 实现
```python
    def _evict_to_host(
        self, node: UnifiedTreeNode, tracker: dict[ComponentType, int] = None
    ) -> None:
        """GPU→CPU demotion: release all device resources, node stays in tree."""
        assert not node.evicted and node.backuped
        trigger = self.components[BASE_COMPONENT_TYPE]
        self._evict_component_and_detach_lru(
            node, trigger, target=EvictLayer.DEVICE, tracker=tracker
        )
        self._cascade_evict(node, trigger, tracker)

        # after device eviction, insert aux components into host LRU.
        self._for_each_component_lru(
            node, UnifiedLRUList.insert_mru, target=EvictLayer.HOST, skip_existing=True
        )
        self._update_evictable_leaf_sets(node.parent)
```
**EN:** GPU→CPU demotion: release all device resources, node stays in tree. Implements the evict TO host routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的evict to主机侧例程。 该方法属于 `UnifiedRadixCache`。

### Lines 1107-1139: _evict_device_leaf implementation / _evict_device_leaf 实现
```python
    def _evict_device_leaf(
        self, node: UnifiedTreeNode, tracker: dict[ComponentType, int]
    ) -> None:
        """Evict a device leaf node, choosing the right strategy:

        - backuped: demote to host via _evict_to_host (node stays in tree)
        - not backuped + write_back: write_backup first, then demote
        - not backuped + write_through: Cascade evict all components

        All freed device tokens are accumulated into *tracker*.
        """
        assert self._is_device_leaf(node), f"node {node.id} is not a D-leaf"
# ... omitted for brevity ...
                self._update_evictable_leaf_sets(parent)
                self._iteratively_delete_tombstone_leaf(node, tracker)
                return
        self._evict_to_host(node, tracker)
```
**EN:** Evict a device leaf node, choosing the right strategy: - backuped: demote to host via _evict_to_host (node stays in tree) - not backuped + write_back: write_backup first, then... Implements the evict device leaf routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evict device leaf例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1141-1156: _evict_host_leaf implementation / _evict_host_leaf 实现
```python
    def _evict_host_leaf(
        self, node: UnifiedTreeNode, tracker: dict[ComponentType, int]
    ) -> None:
        """Atomically evict all components on a host leaf.

        All freed tokens are accumulated into *tracker*."""
        assert self._is_host_leaf(node), f"node {node.id} is not an H-leaf"

        for comp in self._components_tuple:
            _, hf = self._evict_component_and_detach_lru(
                node, comp, target=EvictLayer.ALL, tracker=None
            )
            tracker[comp.component_type] += hf
        self.evictable_host_leaves.discard(node)
        self._remove_leaf_from_parent(node)
        self._iteratively_delete_tombstone_leaf(node, tracker)
```
**EN:** Atomically evict all components on a host leaf. Implements the evict host leaf routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的evict主机侧leaf例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1157-1159: Comment block / 注释块
```python

    # ---- HiCache: Backup / LoadBack ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1160-1221: write_backup implementation / write_backup 实现
```python
    def write_backup(self, node: UnifiedTreeNode, write_back: bool = False) -> int:
        """Backup a node's data from device to host (D->H)."""
        if self.cache_controller is None:
            return 0

        # Backup invariant (write-through): parent must be backuped first
        if not write_back and (
            node.parent is not self.root_node and not node.parent.backuped
        ):
            return 0

        device_value = node.component_data[BASE_COMPONENT_TYPE].value
# ... omitted for brevity ...
        if not write_back:
            lock_params = self.inc_lock_ref(node).to_dec_params()
        self.ongoing_write_through[node.id] = (node, lock_params)
        return len(host_indices)
```
**EN:** Backup a node's data from device to host (D->H). Implements the write backup routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write backup例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1223-1306: load_back implementation / load_back 实现
```python
    def load_back(
        self,
        best_match_node: UnifiedTreeNode,
        mem_quota: Optional[int] = None,
        req=None,
    ) -> bool:
        """Load evicted KV data from host back to device (H→D)."""
        if self.cache_controller is None:
            return False

        # Build KV transfer
        kv_xfer = self.components[BASE_COMPONENT_TYPE].build_hicache_transfers(
# ... omitted for brevity ...
            best_match_node,
            self.inc_lock_ref(best_match_node).to_dec_params(),
        )
        return True
```
**EN:** Load evicted KV data from host back to device (H→D). Loads state from an external or serialized representation. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1308-1352: _build_sidecar_transfers implementation / _build_sidecar_transfers 实现
```python
    def _build_sidecar_transfers(
        self,
        phase: CacheTransferPhase,
        kv_xfer: PoolTransfer,
        comp_xfers: dict[ComponentType, list[PoolTransfer]],
    ) -> list[PoolTransfer]:
        transfers: list[PoolTransfer] = []
        for spec in self.sidecar_pool_specs:
            if spec.indices_from_pool == PoolName.KV:
                indices_source = kv_xfer
            else:
                source_component = {
# ... omitted for brevity ...
                    indices_from_pool=spec.indices_from_pool,
                )
            )
        return transfers
```
**EN:** Implements the build sidecar transfers routine for this scope. It belongs to `UnifiedRadixCache`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的build sidecar transfers例程。 该方法属于 `UnifiedRadixCache`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 1354-1364: _inc_hit_count implementation / _inc_hit_count 实现
```python
    def _inc_hit_count(self, node: UnifiedTreeNode, chunked: bool = False) -> None:
        """Increment hit count; trigger write_backup when threshold reached."""
        if self.cache_controller is None:
            return
        if node.evicted or chunked:
            return
        if self.cache_controller.write_policy == "write_back":
            return
        node.hit_count += 1
        if not node.backuped and node.hit_count >= self.write_through_threshold:
            self.write_backup(node)
```
**EN:** Increment hit count; trigger write_backup when threshold reached. Implements the INC HIT count routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc hit count例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1365-1367: Comment block / 注释块
```python

    # ---- HiCache: Async Event Management ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1368-1413: writing_check implementation / writing_check 实现
```python
    def writing_check(self, write_back: bool = False) -> None:
        """Poll write-through completions."""
        cc = self.cache_controller
        if cc is None:
            return

        if write_back:
            # Blocking: wait for all pending write-backs
            while self.ongoing_write_through:
                for _, finish_event, ack_list in cc.ack_write_queue:
                    finish_event.synchronize()
                    for ack_id in ack_list:
# ... omitted for brevity ...
            for ack_id in ack_list:
                node, params = self.ongoing_write_through.pop(ack_id)
                self.dec_lock_ref(node, params)
            finish_count -= 1
```
**EN:** Poll write-through completions. Implements the writing check routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的writing check例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1415-1428: loading_check implementation / loading_check 实现
```python
    def loading_check(self) -> None:
        """Poll load-back completions."""
        cc = self.cache_controller
        if cc is None or not self.ongoing_load_back:
            return
        finish_count = 0
        for _, finish_event, ack_list in cc.ack_load_queue:
            if not finish_event.query():
                break
            finish_count += 1
            for ack_id in ack_list:
                node, lock_params = self.ongoing_load_back.pop(ack_id)
                self.dec_lock_ref(node, lock_params)
        del cc.ack_load_queue[:finish_count]
```
**EN:** Poll load-back completions. Implements the loading check routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的loading check例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1429-1431: Comment block / 注释块
```python

    # ---- HiCache: Scheduler Entry Points ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1432-1476: init_load_back implementation / init_load_back 实现
```python
    def init_load_back(
        self,
        params: InitLoadBackParams,
    ) -> tuple[torch.Tensor, UnifiedTreeNode]:
        """Prepare KV cache loading from host to device.
        Returns (device_indices, last_node) tuple."""
        best_match_node = params.best_match_node
        mem_quota = params.mem_quota
        req = params.req
        assert req is not None
        last_best_match_device_node = req.last_node

# ... omitted for brevity ...
        return (
            self._empty_match_result.device_indices,
            last_best_match_device_node,
        )
```
**EN:** Prepare KV cache loading from host to device. Initializes supporting state for later operations. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 初始化后续操作所需的辅助状态。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1478-1481: check_hicache_events implementation / check_hicache_events 实现
```python
    def check_hicache_events(self) -> None:
        """Called per scheduler step to poll async HiCache events."""
        self.writing_check()
        self.loading_check()
```
**EN:** Called per scheduler step to poll async HiCache events. Implements the check hicache events routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的check hicache事件例程。 该方法属于 `UnifiedRadixCache`。

### Lines 1483-1485: flush_write_through_acks implementation / flush_write_through_acks 实现
```python
    def flush_write_through_acks(self) -> None:
        """Flush pending write-through acknowledgements."""
        self.writing_check()
```
**EN:** Flush pending write-through acknowledgements. Flushes buffered state to the next storage layer. It belongs to `UnifiedRadixCache`.
**CN:** 将缓冲状态刷新到下一层存储。 该方法属于 `UnifiedRadixCache`。

### Lines 1487-1491: ready_to_load_host_cache implementation / ready_to_load_host_cache 实现
```python
    def ready_to_load_host_cache(self) -> int:
        """Notify the cache controller to start the KV cache loading."""
        if self.cache_controller is not None:
            return self.cache_controller.start_loading()
        return 0
```
**EN:** Notify the cache controller to start the KV cache loading. Implements the ready TO load host cache routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的ready to load主机侧缓存例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1492-1496: Comment block / 注释块
```python

    # ---- Query / Inspection APIs ----
    # These APIs exist for compatibility with other RadixTree implementations.
    # TODO: simplify and consolidate in a future refactor.

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1497-1500: sliding_window_size implementation / sliding_window_size 实现
```python
    @property
    def sliding_window_size(self):
        swa = self.components.get(ComponentType.SWA)
        return swa.sliding_window_size if swa else None
```
**EN:** Implements the sliding window size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的sliding window size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1502-1503: supports_swa implementation / supports_swa 实现
```python
    def supports_swa(self) -> bool:
        return ComponentType.SWA in self.components
```
**EN:** Implements the supports SWA routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports SWA例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1505-1506: supports_mamba implementation / supports_mamba 实现
```python
    def supports_mamba(self) -> bool:
        return ComponentType.MAMBA in self.components
```
**EN:** Implements the supports mamba routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports Mamba例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1507-1509: Comment block / 注释块
```python

    # ---- Streaming session API (delegates to composed StreamingSession) ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 1510-1511: supports_streaming_session implementation / supports_streaming_session 实现
```python
    def supports_streaming_session(self) -> bool:
        return True
```
**EN:** Implements the supports streaming session routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports streaming会话例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1513-1514: release_session implementation / release_session 实现
```python
    def release_session(self, session_id: str) -> None:
        self.session.release_session(session_id)
```
**EN:** Implements the release session routine for this scope. It belongs to `UnifiedRadixCache`.
**CN:** 实现当前作用域中的release会话例程。 该方法属于 `UnifiedRadixCache`。

### Lines 1516-1517: session_held_tokens implementation / session_held_tokens 实现
```python
    def session_held_tokens(self, active_pool_idxs: Optional[set] = None) -> int:
        return self.session.session_held_tokens(active_pool_idxs)
```
**EN:** Implements the session held tokens routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的会话held tokens例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1519-1520: session_held_full_tokens implementation / session_held_full_tokens 实现
```python
    def session_held_full_tokens(self, active_pool_idxs: Optional[set] = None) -> int:
        return self.session.session_held_full_tokens(active_pool_idxs)
```
**EN:** Implements the session held full tokens routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的会话held完整tokens例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1522-1523: session_held_swa_tokens implementation / session_held_swa_tokens 实现
```python
    def session_held_swa_tokens(self, active_pool_idxs: Optional[set] = None) -> int:
        return self.session.session_held_swa_tokens(active_pool_idxs)
```
**EN:** Implements the session held SWA tokens routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的会话held SWA tokens例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1525-1526: session_held_req_count implementation / session_held_req_count 实现
```python
    def session_held_req_count(self, active_pool_idxs: Optional[set] = None) -> int:
        return self.session.session_held_req_count(active_pool_idxs)
```
**EN:** Implements the session held REQ count routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的会话held req count例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1528-1529: session_held_mamba_slots implementation / session_held_mamba_slots 实现
```python
    def session_held_mamba_slots(self, active_pool_idxs: Optional[set] = None) -> int:
        return self.session.session_held_mamba_slots(active_pool_idxs)
```
**EN:** Implements the session held mamba slots routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的会话held Mamba slots例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1531-1532: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self) -> int:
        return self.component_evictable_size_.get(BASE_COMPONENT_TYPE, 0)
```
**EN:** Implements the evictable size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1534-1535: protected_size implementation / protected_size 实现
```python
    def protected_size(self) -> int:
        return self.component_protected_size_.get(BASE_COMPONENT_TYPE, 0)
```
**EN:** Implements the protected size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1537-1538: full_evictable_size implementation / full_evictable_size 实现
```python
    def full_evictable_size(self) -> int:
        return self.evictable_size()
```
**EN:** Implements the full evictable size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整evictable size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1540-1541: full_protected_size implementation / full_protected_size 实现
```python
    def full_protected_size(self) -> int:
        return self.protected_size()
```
**EN:** Implements the full protected size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整protected size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1543-1544: swa_evictable_size implementation / swa_evictable_size 实现
```python
    def swa_evictable_size(self) -> int:
        return self.component_evictable_size_.get(ComponentType.SWA, 0)
```
**EN:** Implements the SWA evictable size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA evictable size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1546-1547: mamba_evictable_size implementation / mamba_evictable_size 实现
```python
    def mamba_evictable_size(self) -> int:
        return self.component_evictable_size_.get(ComponentType.MAMBA, 0)
```
**EN:** Implements the mamba evictable size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba evictable size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1549-1550: swa_protected_size implementation / swa_protected_size 实现
```python
    def swa_protected_size(self) -> int:
        return self.component_protected_size_.get(ComponentType.SWA, 0)
```
**EN:** Implements the SWA protected size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA protected size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1552-1553: mamba_protected_size implementation / mamba_protected_size 实现
```python
    def mamba_protected_size(self) -> int:
        return self.component_protected_size_.get(ComponentType.MAMBA, 0)
```
**EN:** Implements the mamba protected size routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的Mamba protected size例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1555-1572: total_size implementation / total_size 实现
```python
    def total_size(self):
        total_size = 0
        total_aux_size = 0
        stack = [self.root_node]
        while stack:
            node = stack.pop()
            full_value = node.component_data[BASE_COMPONENT_TYPE].value
            if full_value is not None:
                total_size += len(full_value)
            for ct in self.tree_components:
                if ct == BASE_COMPONENT_TYPE:
                    continue
                value = node.component_data[ct].value
                if value is not None:
                    total_aux_size += len(value)
            for child in node.children.values():
                stack.append(child)
        return total_size, total_aux_size
```
**EN:** Implements the total size routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1574-1587: all_values_flatten implementation / all_values_flatten 实现
```python
    def all_values_flatten(self) -> torch.Tensor:
        values = []

        def _dfs(node: UnifiedTreeNode):
            for child in node.children.values():
                v = child.component_data[BASE_COMPONENT_TYPE].value
                if v is not None:
                    values.append(v)
                _dfs(child)

        _dfs(self.root_node)
        if values:
            return torch.cat(values)
        return torch.tensor([], dtype=torch.int64, device=self.device)
```
**EN:** Implements the ALL values flatten routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all values flatten例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1589-1607: _all_component_values_flatten implementation / _all_component_values_flatten 实现
```python
    def _all_component_values_flatten(
        self, component_type: ComponentType
    ) -> torch.Tensor:
        if component_type not in self.components:
            return torch.tensor([], dtype=torch.int64, device=self.device)

        values = []

        def _dfs(node: UnifiedTreeNode):
            value = node.component_data[component_type].value
            if value is not None:
                values.append(value)
# ... omitted for brevity ...
        _dfs(self.root_node)
        if values:
            return torch.cat(values)
        return torch.tensor([], dtype=torch.int64, device=self.device)
```
**EN:** Implements the ALL component values flatten routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all组件values flatten例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1609-1610: all_mamba_values_flatten implementation / all_mamba_values_flatten 实现
```python
    def all_mamba_values_flatten(self) -> torch.Tensor:
        return self._all_component_values_flatten(ComponentType.MAMBA)
```
**EN:** Implements the ALL mamba values flatten routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all Mamba values flatten例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1612-1613: all_swa_values_flatten implementation / all_swa_values_flatten 实现
```python
    def all_swa_values_flatten(self) -> torch.Tensor:
        return self._all_component_values_flatten(ComponentType.SWA)
```
**EN:** Implements the ALL SWA values flatten routine for this scope. It belongs to `UnifiedRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all SWA values flatten例程。 该方法属于 `UnifiedRadixCache`。它会向调用方返回计算结果。

### Lines 1615-1639: available_and_evictable_str implementation / available_and_evictable_str 实现
```python
    def available_and_evictable_str(self) -> str:
        if self.supports_swa():
            full_available_size = self.token_to_kv_pool_allocator.full_available_size()
        else:
            full_available_size = self.token_to_kv_pool_allocator.available_size()
        full_evictable = self.component_evictable_size_[BASE_COMPONENT_TYPE]
        lines = [
            f"Available full tokens: {full_available_size + full_evictable} "
            f"(full_available_size={full_available_size} + full_evictable_size_={full_evictable})"
        ]
        for ct in self.tree_components:
            if ct == BASE_COMPONENT_TYPE:
# ... omitted for brevity ...
                f"Available {ct}: {available_size + self.component_evictable_size_[ct]} "
                f"(available_size={available_size} + component_evictable_size_={self.component_evictable_size_[ct]})"
            )
        return "\n".join(lines) + "\n"
```
**EN:** Implements the available AND evictable STR routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available and evictable str例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1641-1648: _collect_all_nodes implementation / _collect_all_nodes 实现
```python
    def _collect_all_nodes(self) -> list[UnifiedTreeNode]:
        nodes = []
        stack = [self.root_node]
        while stack:
            node = stack.pop()
            nodes.append(node)
            stack.extend(node.children.values())
        return nodes
```
**EN:** Implements the collect ALL nodes routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect all nodes例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1650-1884: sanity_check implementation / sanity_check 实现
```python
    def sanity_check(self):
        """Verify tree invariants.

        TODO(hzh): This method has relatively high latency; simplify the
        check logic once the tree implementation stabilizes.
        """
        # Skip when streaming sessions hold tree locks: the check asserts
        # all nodes are unlocked during idle, which streaming sessions break
        # by design (they hold a first-turn lock across turns).
        if self.session.any_holding_kv():
            return

# ... omitted for brevity ...
        logger.debug(
            f"Sanity check PASSED: {len(all_nodes)} nodes, "
            f"{len(self.tree_components)} components"
        )
```
**EN:** Verify tree invariants. Implements the sanity check routine for this scope. It belongs to `UnifiedRadixCache`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的sanity check例程。 该方法属于 `UnifiedRadixCache`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 1886-1917: _check_lru_linked_list implementation / _check_lru_linked_list 实现
```python
    def _check_lru_linked_list(
        self,
        lru: "UnifiedLRUList",
        ct: ComponentType,
        label: str,
        errors: list[str],
    ) -> None:
        """Walk a LRU doubly-linked list, collect integrity errors."""
        pt = lru._pt  # use LRU's own pointer slot
        visited: set[int] = set()
        x = lru.head.lru_next[pt]
        prev = lru.head
# ... omitted for brevity ...
        if len(visited) != len(lru.cache):
            errors.append(
                f"[{label}][{ct}] list={len(visited)} != cache={len(lru.cache)}"
            )
```
**EN:** Walk a LRU doubly-linked list, collect integrity errors. Implements the check LRU linked list routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的check lru linked list例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1919-1935: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self) -> None:
        stack = [(self.root_node, 0)]
        while stack:
            node, indent = stack.pop()
            component_str = " ".join(
                f"{ct}={'yes' if node.component_data[ct].value is not None else 'no'}"
                for ct in self.tree_components
            )
            print(
                " " * indent,
                f"[{node.id}]",
                len(node.key),
                f"full_lock={node.component_data[BASE_COMPONENT_TYPE].lock_ref}",
                component_str,
            )
            for child in node.children.values():
                stack.append((child, indent + 2))
```
**EN:** Implements the pretty print routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1937-1944: _rebuild_host_leaf_sets implementation / _rebuild_host_leaf_sets 实现
```python
    def _rebuild_host_leaf_sets(self) -> None:
        """Rebuild evictable_host_leaves after L1-only reset."""
        stack = [self.root_node]
        while stack:
            node = stack.pop()
            if node is not self.root_node:
                self._update_evictable_leaf_sets(node)
            stack.extend(node.children.values())
```
**EN:** Rebuild evictable_host_leaves after L1-only reset. Implements the rebuild host leaf sets routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的rebuild主机侧leaf sets例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

### Lines 1946-1960: _rebuild_host_lru_lists implementation / _rebuild_host_lru_lists 实现
```python
    def _rebuild_host_lru_lists(self) -> None:
        """Rebuild host_lru_lists for extra components after L1-only reset.
        Walks the tree and adds nodes with host component data to the
        appropriate host LRU list."""
        stack = [self.root_node]
        while stack:
            node = stack.pop()
            if node is not self.root_node:
                for ct in self.tree_components:
                    if ct == BASE_COMPONENT_TYPE:
                        continue  # Full uses evictable_host_leaves, not host LRU
                    cd = node.component_data[ct]
                    if cd.host_value is not None:
                        self.host_lru_lists[ct].insert_mru(node)
            stack.extend(node.children.values())
```
**EN:** Rebuild host_lru_lists for extra components after L1-only reset. Implements the rebuild host LRU lists routine for this scope. It belongs to `UnifiedRadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的rebuild主机侧lru lists例程。 该方法属于 `UnifiedRadixCache`。实现过程中会遍历输入或受管条目。

## Key Concepts / 关键概念
- **`UnifiedTreeNode`**: Defines the `UnifiedTreeNode` type and its core responsibilities. / 定义 `UnifiedTreeNode` 类型及其核心职责。
- **`UnifiedLRUList`**: Defines the `UnifiedLRUList` type and its core responsibilities. / 定义 `UnifiedLRUList` 类型及其核心职责。
- **`UnifiedRadixCache`**: Defines the `UnifiedRadixCache` type and its core responsibilities. / 定义 `UnifiedRadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `threading`, `time`, `collections`, `functools`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.unified_cache_components`, `sglang.srt.session.streaming_session`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.server_args`, `sglang.srt.mem_cache.hybrid_cache.hybrid_pool_assembler`
