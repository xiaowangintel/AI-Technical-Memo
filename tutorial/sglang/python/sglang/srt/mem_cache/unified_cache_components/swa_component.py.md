# swa_component.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/unified_cache_components/swa_component.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the SWA component logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的SWA组件相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and setup / 导入与初始化
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Callable, Optional

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    DecLockRefParams,
    EvictParams,
    IncLockRefResult,
    InsertParams,
    InsertResult,
# ... omitted for brevity ...
    EvictLayer,
    TreeComponent,
    next_component_uuid,
)
```
**EN:** Imports `__future__`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.unified_cache_components.tree_component` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.unified_cache_components.tree_component` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 26-32: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.mem_cache.unified_radix_cache import (
        UnifiedRadixCache,
        UnifiedTreeNode,
    )
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 35-44: SWAComponent declaration / SWAComponent 声明
```python
class SWAComponent(TreeComponent):
    """Sliding window attention component.

    Each SWA node stores translated SWA pool indices as its component
    value, independent of the full attention indices on the same tree node.
    When SWA data is evicted from an internal node the node is tombstoned
    — its SWA component value becomes None while the full attention
    value stays intact.
    """

```
**EN:** Sliding window attention component. Declares the `SWAComponent` class and connects it to `TreeComponent`.
**CN:** 声明 `SWAComponent` 类，并将其关联到 `TreeComponent`。

### Lines 45-54: __init__ implementation / __init__ 实现
```python
    def __init__(self, cache: UnifiedRadixCache, params: CacheInitParams):
        from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator

        assert isinstance(
            cache.token_to_kv_pool_allocator, SWATokenToKVPoolAllocator
        ), f"SWAComponent requires SWATokenToKVPoolAllocator, got {type(cache.token_to_kv_pool_allocator)}"
        super().__init__(cache, params)
        self.sliding_window_size = params.sliding_window_size
        # HiCache state: set to host SWA pool when HiCache enabled
        self._swa_kv_pool_host = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SWAComponent`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SWAComponent`。

### Lines 56-56: Shared state definitions / 共享状态定义
```python
    component_type = ComponentType.SWA
```
**EN:** Defines class-level variables such as `component_type`.
**CN:** 定义类级变量，例如 `component_type`。

### Lines 58-61: _translate_full_to_swa implementation / _translate_full_to_swa 实现
```python
    def _translate_full_to_swa(self, full_indices: torch.Tensor) -> torch.Tensor:
        return self.cache.token_to_kv_pool_allocator.translate_loc_from_full_to_swa(
            full_indices
        )
```
**EN:** Implements the translate full TO SWA routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate完整to SWA例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 63-70: _restore_device_value implementation / _restore_device_value 实现
```python
    def _restore_device_value(self, node: UnifiedTreeNode, value: torch.Tensor) -> None:
        ct = self.component_type
        node.component_data[ct].value = value
        host_lru = self.cache.host_lru_lists[ct]
        if host_lru.in_list(node):
            host_lru.remove_node(node)
        self.cache.lru_lists[ct].insert_mru(node)
        self.cache.component_evictable_size_[ct] += len(value)
```
**EN:** Implements the restore device value routine for this scope. It belongs to `SWAComponent`.
**CN:** 实现当前作用域中的restore device value例程。 该方法属于 `SWAComponent`。

### Lines 72-89: create_match_validator implementation / create_match_validator 实现
```python
    def create_match_validator(
        self, match_device_only: bool = False
    ) -> Callable[[UnifiedTreeNode], bool]:
        sliding_window_size = self.sliding_window_size
        ct = self.component_type
        state = {"len": float("inf")}

        def validator(node: UnifiedTreeNode) -> bool:
            cd = node.component_data[ct]
            # HiCache: a host-only tombstone is a valid match boundary too
            # — load_back will restore SWA from host before use.
            if cd.value is None and (match_device_only or cd.host_value is None):
                state["len"] = 0
                return False
            state["len"] += len(node.key)
            return state["len"] >= sliding_window_size

        return validator
```
**EN:** Constructs a new object or resource with the requested configuration. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 91-114: finalize_match_result implementation / finalize_match_result 实现
```python
    def finalize_match_result(
        self,
        result: MatchResult,
        params: MatchPrefixParams,
        value_chunks: list[torch.Tensor],
        best_value_len: int,
    ) -> MatchResult:
        ct = self.component_type
        n_swa = 0
        node = result.best_match_node
        root = self.cache.root_node
        while node is not root and n_swa < self.sliding_window_size:
# ... omitted for brevity ...
            else:
                break
            node = node.parent
        return result
```
**EN:** Implements the finalize match result routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的finalize match result例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 116-167: update_component_on_insert_overlap implementation / update_component_on_insert_overlap 实现
```python
    def update_component_on_insert_overlap(
        self,
        node: UnifiedTreeNode,
        prefix_len: int,
        total_prefix_len: int,
        value_slice: torch.Tensor,
        params: InsertParams,
    ) -> int:
        if params.prev_prefix_len >= total_prefix_len + prefix_len:
            return prefix_len

        is_tombstone = node.component_data[self.component_type].value is None
# ... omitted for brevity ...
            return start_idx
        else:
            # Branch 3: entire value_slice is outside SWA window — not consumed
            return prefix_len
```
**EN:** Updates existing state to reflect new inputs. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 根据新输入更新已有状态。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 169-172: should_skip_leaf_creation implementation / should_skip_leaf_creation 实现
```python
    def should_skip_leaf_creation(
        self, total_prefix_len: int, key_len: int, params: InsertParams
    ) -> bool:
        return params.swa_evicted_seqlen >= total_prefix_len + key_len
```
**EN:** Implements the should skip leaf creation routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的should skip leaf creation例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 174-205: recover_after_unevict implementation / recover_after_unevict 实现
```python
    def recover_after_unevict(
        self,
        node: UnifiedTreeNode,
        prefix_len: int,
        total_prefix_len: int,
        params: InsertParams,
    ) -> None:
        # _unevict_node_on_insert already wrote the request's fresh KV slice
        # into the base value. We just need to rebuild SWA from that slice for
        # the in-window portion. There is no old SWA slot to free here.
        ct = self.component_type
        if node.component_data[ct].value is not None:
# ... omitted for brevity ...
            swa_value = self._translate_full_to_swa(full_value)
        else:
            return
        self._restore_device_value(node, swa_value)
```
**EN:** Implements the recover after unevict routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的recover after unevict例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 207-237: commit_insert_component_data implementation / commit_insert_component_data 实现
```python
    def commit_insert_component_data(
        self,
        node: UnifiedTreeNode,
        is_new_leaf: bool,
        params: InsertParams,
        result: InsertResult,
    ) -> None:
        if not is_new_leaf:
            return

        node_start = result.prefix_len
        split_pos = params.swa_evicted_seqlen - node_start
# ... omitted for brevity ...
            )
            node.component_data[self.component_type].value = swa_value
            self.cache.lru_lists[self.component_type].insert_mru(node)
            self.cache.component_evictable_size_[self.component_type] += len(swa_value)
```
**EN:** Implements the commit insert component data routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的commit insert组件data例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 239-279: redistribute_on_node_split implementation / redistribute_on_node_split 实现
```python
    def redistribute_on_node_split(
        self, new_parent: UnifiedTreeNode, child: UnifiedTreeNode
    ):
        new_parent.component_data[self.component_type].lock_ref = child.component_data[
            self.component_type
        ].lock_ref

        child_swa_value = child.component_data[self.component_type].value
        if child_swa_value is not None:
            split_len = len(new_parent.key)
            new_parent.component_data[self.component_type].value = child_swa_value[
                :split_len
# ... omitted for brevity ...
        new_parent.component_data[self.component_type].metadata["uuid"] = (
            child.component_data[self.component_type].metadata.get("uuid")
        )
        child.component_data[self.component_type].metadata.pop("uuid", None)
```
**EN:** Implements the redistribute ON node split routine for this scope. It belongs to `SWAComponent`.
**CN:** 实现当前作用域中的redistribute on node split例程。 该方法属于 `SWAComponent`。

### Lines 281-322: evict_component implementation / evict_component 实现
```python
    def evict_component(
        self,
        node: UnifiedTreeNode,
        target: EvictLayer = EvictLayer.DEVICE,
    ) -> tuple[int, int]:
        ct = self.component_type
        cd = node.component_data[ct]
        freed = 0
        host_freed = 0

        # Device layer
        if EvictLayer.DEVICE in target and cd.value is not None:
# ... omitted for brevity ...
            if not host_lru.in_list(node):
                host_lru.insert_mru(node)

        return freed, host_freed
```
**EN:** Removes cache entries according to the active policy. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 324-325: eviction_priority implementation / eviction_priority 实现
```python
    def eviction_priority(self, is_leaf: bool) -> int:
        return 0 if is_leaf else 1
```
**EN:** Implements the eviction priority routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的eviction priority例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 327-350: drive_eviction implementation / drive_eviction 实现
```python
    def drive_eviction(
        self, params: EvictParams, tracker: dict[ComponentType, int]
    ) -> None:
        request = params.swa_num_tokens
        ct = self.component_type
        lru = self.cache.lru_lists[ct]
        x = lru.get_lru_no_lock()
        while tracker[ct] < request and x is not None and lru.in_list(x):
            assert x.component_data[ct].value is not None
            if x in self.cache.evictable_device_leaves:
                # D-leaf: atomic eviction of all components
                x_next = lru.get_prev_no_lock(x)
# ... omitted for brevity ...
                    x, self, target=EvictLayer.DEVICE, tracker=tracker
                )
                self.cache._cascade_evict(x, self, tracker)
                x = x_next
```
**EN:** Implements the drive eviction routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive eviction例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。

### Lines 352-384: acquire_component_lock implementation / acquire_component_lock 实现
```python
    def acquire_component_lock(
        self, node: UnifiedTreeNode, result: IncLockRefResult
    ) -> IncLockRefResult:
        ct = self.component_type
        root = self.cache.root_node
        sliding_window_size = self.sliding_window_size
        swa_lock_size = 0
        swa_uuid_for_lock = None

        # Tombstoned nodes (cd.value is None) have no SWA chunk to protect
        # skip them and keep walking up. This path is hit when HiCache
        # backs up a FULL present internal node whose SWA was already evicted.
# ... omitted for brevity ...
            cur = cur.parent

        result.swa_uuid_for_lock = swa_uuid_for_lock
        return result
```
**EN:** Implements the acquire component lock routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的acquire组件lock例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 386-412: release_component_lock implementation / release_component_lock 实现
```python
    def release_component_lock(
        self, node: UnifiedTreeNode, params: Optional[DecLockRefParams]
    ) -> None:
        ct = self.component_type
        root = self.cache.root_node
        swa_uuid_for_lock = params.swa_uuid_for_lock if params else None
        skip_lock_node_ids = params.skip_lock_node_ids.get(ct, ()) if params else ()
        dec_swa = True

        # A node in skip_lock_node_ids was a tombstone when this lock was acquired.
        cur = node
        while cur != root and dec_swa:
# ... omitted for brevity ...
            comp.lock_ref -= 1
            if swa_uuid_for_lock and comp.metadata.get("uuid") == swa_uuid_for_lock:
                dec_swa = False
            cur = cur.parent
```
**EN:** Implements the release component lock routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的release组件lock例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。

### Lines 414-423: prepare_for_caching_req implementation / prepare_for_caching_req 实现
```python
    def prepare_for_caching_req(
        self,
        req: Req,
        insert_params: InsertParams,
        token_ids_len: int,
        is_finished: bool,
    ) -> Optional[int]:
        if is_finished:
            insert_params.swa_evicted_seqlen = req.swa_evicted_seqlen
        return None
```
**EN:** Implements the prepare for caching REQ routine for this scope. It belongs to `SWAComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prepare for caching req例程。 该方法属于 `SWAComponent`。它会向调用方返回计算结果。

### Lines 424-426: Comment block / 注释块
```python

    # ---- HiCache Hooks ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 427-480: build_hicache_transfers implementation / build_hicache_transfers 实现
```python
    def build_hicache_transfers(
        self, node: UnifiedTreeNode, phase: CacheTransferPhase, **kw
    ) -> Optional[list[PoolTransfer]]:
        ct = self.component_type

        if phase == CacheTransferPhase.BACKUP_HOST:
            cd = node.component_data[ct]
            if cd.value is None:
                return None
            # cd.value already holds SWA-pool indices (translated at insert time).
            # Host pool indexing wants int64.
            return [
# ... omitted for brevity ...
                )
            ]

        return None
```
**EN:** Builds a derived structure from the available configuration or inputs. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 482-515: commit_hicache_transfer implementation / commit_hicache_transfer 实现
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
                cd = node.component_data[ct]
                if cd.host_value is None:
# ... omitted for brevity ...
                allocator.set_full_to_swa_mapping(cd_full_n.value, swa_chunk)
                offset += n_tokens
            assert offset == len(xfer.host_indices)
            return
```
**EN:** Implements the commit hicache transfer routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的commit hicache transfer例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 517-537: drive_host_eviction implementation / drive_host_eviction 实现
```python
    def drive_host_eviction(
        self, num_tokens: int, tracker: dict[ComponentType, int]
    ) -> None:
        """Evict SWA host resources.
        Internal nodes: private tombstone (free SWA host only).
        Host leaves: atomic eviction via _evict_host_leaf."""
        ct = self.component_type
        host_lru = self.cache.host_lru_lists[ct]
        x = host_lru.get_lru_no_lock()
        while tracker[ct] < num_tokens and x is not None and host_lru.in_list(x):
            x_next = host_lru.get_prev_no_lock(x)
            cd = x.component_data[ct]
# ... omitted for brevity ...
                    x, self, target=EvictLayer.HOST, tracker=tracker
                )
                self.cache._cascade_evict(x, self, tracker, target=EvictLayer.HOST)
            x = x_next
```
**EN:** Evict SWA host resources. Implements the drive host eviction routine for this scope. It belongs to `SWAComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive主机侧eviction例程。 该方法属于 `SWAComponent`。实现过程中会遍历输入或受管条目。

## Key Concepts / 关键概念
- **`SWAComponent`**: Defines the `SWAComponent` type and its core responsibilities. / 定义 `SWAComponent` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.unified_cache_components.tree_component`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.unified_radix_cache`, `sglang.srt.mem_cache.swa_memory_pool`
