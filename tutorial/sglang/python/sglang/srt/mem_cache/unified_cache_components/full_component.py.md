# full_component.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/unified_cache_components/full_component.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the full component logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的完整组件相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import heapq
from typing import TYPE_CHECKING, Callable, Optional

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    DecLockRefParams,
    EvictParams,
    IncLockRefResult,
    MatchPrefixParams,
# ... omitted for brevity ...
    ComponentType,
    EvictLayer,
    TreeComponent,
)
```
**EN:** Imports `__future__`, `heapq`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `heapq`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 23-26: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.unified_radix_cache import (
        UnifiedTreeNode,
    )
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 29-29: FullComponent declaration / FullComponent 声明
```python
class FullComponent(TreeComponent):
```
**EN:** Declares the `FullComponent` class and connects it to `TreeComponent`.
**CN:** 声明 `FullComponent` 类，并将其关联到 `TreeComponent`。

### Lines 30-30: Shared state definitions / 共享状态定义
```python
    component_type = ComponentType.FULL
```
**EN:** Defines class-level variables such as `component_type`.
**CN:** 定义类级变量，例如 `component_type`。

### Lines 32-42: __init__ implementation / __init__ 实现
```python
    def __init__(self, cache, params):
        super().__init__(cache, params)
        allocator = cache.token_to_kv_pool_allocator
        # When SWA is present, only free full-attention KV here;
        # SWA KV will be freed by cascade via SWAComponent.evict_component.
        if ComponentType.SWA in cache.tree_components:
            self._free_full = allocator.full_attn_allocator.free
        else:
            self._free_full = allocator.free
        # HiCache state: set to host KV pool when HiCache enabled
        self._full_kv_pool_host = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `FullComponent`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `FullComponent`。

### Lines 44-55: create_match_validator implementation / create_match_validator 实现
```python
    def create_match_validator(
        self, match_device_only: bool = False
    ) -> Callable[[UnifiedTreeNode], bool]:
        if match_device_only:
            return (
                lambda node: node.component_data[self.component_type].value is not None
            )

        # HiCache: evicted + backuped nodes are valid match boundaries.
        return lambda node: (
            node.component_data[self.component_type].value is not None or node.backuped
        )
```
**EN:** Constructs a new object or resource with the requested configuration. It belongs to `FullComponent`. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `FullComponent`。它会向调用方返回计算结果。

### Lines 57-79: finalize_match_result implementation / finalize_match_result 实现
```python
    def finalize_match_result(
        self,
        result: MatchResult,
        params: MatchPrefixParams,
        value_chunks: list[torch.Tensor],
        best_value_len: int,
    ) -> MatchResult:
        # Compute Full KV host hit length: walk from last_host_node up to
        # last_device_node, summing host_value lengths of evicted nodes.
        ct = self.component_type
        kv_host_hit = 0
        node = result.last_host_node
# ... omitted for brevity ...
            return result._replace(
                host_hit_length=max(result.host_hit_length, kv_host_hit)
            )
        return result
```
**EN:** Implements the finalize match result routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的finalize match result例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 81-95: redistribute_on_node_split implementation / redistribute_on_node_split 实现
```python
    def redistribute_on_node_split(
        self, new_parent: UnifiedTreeNode, child: UnifiedTreeNode
    ):
        ct = self.component_type
        new_parent.component_data[ct].lock_ref = child.component_data[ct].lock_ref
        child_cd = child.component_data[ct]
        split_len = len(new_parent.key)
        if child_cd.value is not None:
            new_parent.component_data[ct].value = child_cd.value[:split_len].clone()
            child_cd.value = child_cd.value[split_len:].clone()
        if child_cd.host_value is not None:
            new_parent.component_data[ct].host_value = child_cd.host_value[
                :split_len
            ].clone()
            child_cd.host_value = child_cd.host_value[split_len:].clone()
```
**EN:** Implements the redistribute ON node split routine for this scope. It belongs to `FullComponent`.
**CN:** 实现当前作用域中的redistribute on node split例程。 该方法属于 `FullComponent`。

### Lines 97-121: evict_component implementation / evict_component 实现
```python
    def evict_component(
        self,
        node: UnifiedTreeNode,
        target: EvictLayer = EvictLayer.DEVICE,
    ) -> tuple[int, int]:
        cd = node.component_data[self.component_type]
        freed = 0
        host_freed = 0

        # Device layer
        if EvictLayer.DEVICE in target and cd.value is not None:
            self._free_full(cd.value)
# ... omitted for brevity ...
            if self._full_kv_pool_host is not None:
                self._full_kv_pool_host.free(cd.host_value)
            cd.host_value = None
        return freed, host_freed
```
**EN:** Removes cache entries according to the active policy. It belongs to `FullComponent`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `FullComponent`。它会向调用方返回计算结果。

### Lines 123-124: eviction_priority implementation / eviction_priority 实现
```python
    def eviction_priority(self, is_leaf: bool) -> int:
        return 0 if is_leaf else 2
```
**EN:** Implements the eviction priority routine for this scope. It belongs to `FullComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的eviction priority例程。 该方法属于 `FullComponent`。它会向调用方返回计算结果。

### Lines 126-140: drive_eviction implementation / drive_eviction 实现
```python
    def drive_eviction(
        self, params: EvictParams, tracker: dict[ComponentType, int]
    ) -> None:
        request = params.num_tokens
        # Heap-based eviction from evictable_device_leaves, ordered by LRU.
        heap = [(n.last_access_time, n) for n in self.cache.evictable_device_leaves]
        heapq.heapify(heap)
        ct = self.component_type
        while tracker[ct] < request and heap:
            _, x = heapq.heappop(heap)
            if x not in self.cache.evictable_device_leaves:
                continue
            self.cache._evict_device_leaf(x, tracker)
            if x.parent is not None and x.parent in self.cache.evictable_device_leaves:
                heapq.heappush(heap, (x.parent.last_access_time, x.parent))
```
**EN:** Implements the drive eviction routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive eviction例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。

### Lines 142-155: drive_host_eviction implementation / drive_host_eviction 实现
```python
    def drive_host_eviction(
        self, num_tokens: int, tracker: dict[ComponentType, int]
    ) -> None:
        """Evict host leaves to free KV host pool space."""
        heap = [(n.last_access_time, n) for n in self.cache.evictable_host_leaves]
        heapq.heapify(heap)
        ct = self.component_type
        while tracker[ct] < num_tokens and heap:
            _, x = heapq.heappop(heap)
            if x not in self.cache.evictable_host_leaves:
                continue
            self.cache._evict_host_leaf(x, tracker)
            if x.parent is not None and x.parent in self.cache.evictable_host_leaves:
                heapq.heappush(heap, (x.parent.last_access_time, x.parent))
```
**EN:** Evict host leaves to free KV host pool space. Implements the drive host eviction routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive主机侧eviction例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。

### Lines 157-185: acquire_component_lock implementation / acquire_component_lock 实现
```python
    def acquire_component_lock(
        self, node: UnifiedTreeNode, result: IncLockRefResult
    ) -> IncLockRefResult:
        ct = self.component_type
        root = self.cache.root_node
        cur = node

        # Skip the bottom evicted segment
        while cur is not root and cur.component_data[ct].value is None:
            result.skip_lock_node_ids.setdefault(ct, set()).add(cur.id)
            cur = cur.parent

# ... omitted for brevity ...
            self.cache.evictable_device_leaves.discard(cur)
            cur = cur.parent
        result.delta = delta
        return result
```
**EN:** Implements the acquire component lock routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的acquire组件lock例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 187-209: release_component_lock implementation / release_component_lock 实现
```python
    def release_component_lock(
        self, node: UnifiedTreeNode, params: Optional[DecLockRefParams]
    ) -> None:
        ct = self.component_type
        root = self.cache.root_node
        skip_lock_node_ids = params.skip_lock_node_ids.get(ct, ()) if params else ()
        cur = node
        while cur != root:
            if cur.id in skip_lock_node_ids:
                cur = cur.parent
                continue
            cd = cur.component_data[ct]
# ... omitted for brevity ...
            cd.lock_ref -= 1
            if cd.lock_ref == 0:
                self.cache._update_evictable_leaf_sets(cur)
            cur = cur.parent
```
**EN:** Implements the release component lock routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的release组件lock例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。

### Lines 210-212: Comment block / 注释块
```python

    # ---- HiCache Hooks ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 213-251: build_hicache_transfers implementation / build_hicache_transfers 实现
```python
    def build_hicache_transfers(
        self, node: UnifiedTreeNode, phase: CacheTransferPhase, **kw
    ) -> Optional[list[PoolTransfer]]:
        ct = self.component_type

        if phase == CacheTransferPhase.BACKUP_HOST:
            # Full KV backup is handled by the main flow
            # (write_backup → cache_controller.write on host_value directly).
            # No extra PoolTransfer needed.
            return None

        if phase == CacheTransferPhase.LOAD_BACK:
# ... omitted for brevity ...
                )
            ]

        return None
```
**EN:** Builds a derived structure from the available configuration or inputs. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 253-282: commit_hicache_transfer implementation / commit_hicache_transfer 实现
```python
    def commit_hicache_transfer(
        self,
        node: UnifiedTreeNode,
        phase: CacheTransferPhase,
        transfers: list[PoolTransfer] = (),
    ) -> None:
        ct = self.component_type

        if phase == CacheTransferPhase.BACKUP_HOST:
            if transfers and transfers[0].host_indices is not None:
                node.component_data[ct].host_value = transfers[0].host_indices.clone()

# ... omitted for brevity ...
                self.cache.component_evictable_size_[ct] += n_len
                self.cache._update_evictable_leaf_sets(n)

            self.cache._update_evictable_leaf_sets(node)
```
**EN:** Implements the commit hicache transfer routine for this scope. It belongs to `FullComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的commit hicache transfer例程。 该方法属于 `FullComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`FullComponent`**: Defines the `FullComponent` type and its core responsibilities. / 定义 `FullComponent` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `heapq`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.unified_cache_components.tree_component`, `sglang.srt.mem_cache.unified_radix_cache`
