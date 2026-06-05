# mamba_component.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/unified_cache_components/mamba_component.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the mamba component logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的Mamba组件相关逻辑。

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
    TreeComponent,
    get_and_increase_time_counter,
)
from sglang.srt.server_args import get_global_server_args
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

### Lines 35-35: MambaComponent declaration / MambaComponent 声明
```python
class MambaComponent(TreeComponent):
```
**EN:** Declares the `MambaComponent` class and connects it to `TreeComponent`.
**CN:** 声明 `MambaComponent` 类，并将其关联到 `TreeComponent`。

### Lines 36-36: Shared state definitions / 共享状态定义
```python
    component_type = ComponentType.MAMBA
```
**EN:** Defines class-level variables such as `component_type`.
**CN:** 定义类级变量，例如 `component_type`。

### Lines 38-51: __init__ implementation / __init__ 实现
```python
    def __init__(self, cache: UnifiedRadixCache, params: CacheInitParams):
        from sglang.srt.mem_cache.memory_pool import HybridReqToTokenPool

        assert isinstance(
            cache.req_to_token_pool, HybridReqToTokenPool
        ), f"MambaComponent requires HybridReqToTokenPool, got {type(cache.req_to_token_pool)}"
        if not params.enable_mamba_extra_buffer:
            assert (
                cache.page_size == 1
            ), f"MambaComponent requires page_size=1 when mamba_extra_buffer is disabled, got {cache.page_size}"
        super().__init__(cache, params)
        self.enable_mamba_extra_buffer = params.enable_mamba_extra_buffer
        # HiCache state
        self._mamba_pool_host = None  # set to host mamba pool when HiCache enabled
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MambaComponent`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MambaComponent`。

### Lines 53-64: create_match_validator implementation / create_match_validator 实现
```python
    def create_match_validator(
        self, match_device_only: bool = False
    ) -> Callable[[UnifiedTreeNode], bool]:
        ct = self.component_type
        if match_device_only:
            return lambda node: node.component_data[ct].value is not None

        # HiCache: evicted + backuped (host_value present) is also a valid match
        return lambda node: (
            node.component_data[ct].value is not None
            or node.component_data[ct].host_value is not None
        )
```
**EN:** Constructs a new object or resource with the requested configuration. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 66-116: finalize_match_result implementation / finalize_match_result 实现
```python
    def finalize_match_result(
        self,
        result: MatchResult,
        params: MatchPrefixParams,
        value_chunks: list[torch.Tensor],
        best_value_len: int,
    ) -> MatchResult:
        cow_mamba = params.cow_mamba
        req = params.req
        last_node = result.best_match_node

        # HiCache can still use prefix matches and load back host-backed Mamba
# ... omitted for brevity ...
        if cd.value is None and cd.host_value is not None:
            result = result._replace(host_hit_length=max(result.host_hit_length, 1))

        return result._replace(mamba_branching_seqlen=branching_seqlen)
```
**EN:** Implements the finalize match result routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的finalize match result例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 118-147: commit_insert_component_data implementation / commit_insert_component_data 实现
```python
    def commit_insert_component_data(
        self,
        node: UnifiedTreeNode,
        is_new_leaf: bool,
        params: InsertParams,
        result: InsertResult,
    ) -> None:
        assert params.mamba_value is not None
        if is_new_leaf:
            node.component_data[self.component_type].value = params.mamba_value
            self.cache.lru_lists[self.component_type].insert_mru(node)
            self.cache.component_evictable_size_[self.component_type] += len(
# ... omitted for brevity ...
            return
        self.cache.lru_lists[self.component_type].reset_node_mru(node)
        node.last_access_time = get_and_increase_time_counter()
        result.mamba_exist = True
```
**EN:** Implements the commit insert component data routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的commit insert组件data例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 149-157: redistribute_on_node_split implementation / redistribute_on_node_split 实现
```python
    def redistribute_on_node_split(
        self, new_parent: UnifiedTreeNode, child: UnifiedTreeNode
    ):
        ct = self.component_type
        new_parent.component_data[ct].value = None
        new_parent.component_data[ct].lock_ref = 0
        # HiCache: mamba host_value stays on child (mamba = leaf-only data)
        new_parent.component_data[ct].host_value = None
        new_parent.component_data[ct].host_lock_ref = 0
```
**EN:** Implements the redistribute ON node split routine for this scope. It belongs to `MambaComponent`.
**CN:** 实现当前作用域中的redistribute on node split例程。 该方法属于 `MambaComponent`。

### Lines 159-194: evict_component implementation / evict_component 实现
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
            self.cache.req_to_token_pool.mamba_pool.free(cd.value)
# ... omitted for brevity ...
            if not host_lru.in_list(node):
                host_lru.insert_mru(node)

        return freed, host_freed
```
**EN:** Removes cache entries according to the active policy. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 196-219: drive_eviction implementation / drive_eviction 实现
```python
    def drive_eviction(
        self, params: EvictParams, tracker: dict[ComponentType, int]
    ) -> None:
        request = params.mamba_num
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
**EN:** Implements the drive eviction routine for this scope. It belongs to `MambaComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive eviction例程。 该方法属于 `MambaComponent`。实现过程中会遍历输入或受管条目。

### Lines 221-237: acquire_component_lock implementation / acquire_component_lock 实现
```python
    def acquire_component_lock(
        self, node: UnifiedTreeNode, result: IncLockRefResult
    ) -> IncLockRefResult:
        ct = self.component_type
        cd = node.component_data[ct]
        value = cd.value
        # A node in skip_lock_node_ids was a tombstone when this lock was acquired.
        if value is None:
            result.skip_lock_node_ids.setdefault(ct, set()).add(node.id)
            return result

        if cd.lock_ref == 0:
            vlen = len(value)
            self.cache.component_evictable_size_[ct] -= vlen
            self.cache.component_protected_size_[ct] += vlen
        cd.lock_ref += 1
        return result
```
**EN:** Implements the acquire component lock routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的acquire组件lock例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 239-254: release_component_lock implementation / release_component_lock 实现
```python
    def release_component_lock(
        self, node: UnifiedTreeNode, params: Optional[DecLockRefParams]
    ) -> None:
        ct = self.component_type
        cd = node.component_data[ct]
        skip_lock_node_ids = params.skip_lock_node_ids.get(ct, ()) if params else ()
        if node.id in skip_lock_node_ids:
            return

        value = cd.value
        if value is not None and cd.lock_ref > 0:
            if cd.lock_ref == 1:
                vlen = len(value)
                self.cache.component_evictable_size_[ct] += vlen
                self.cache.component_protected_size_[ct] -= vlen
            cd.lock_ref -= 1
```
**EN:** Implements the release component lock routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的release组件lock例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 256-306: prepare_for_caching_req implementation / prepare_for_caching_req 实现
```python
    def prepare_for_caching_req(
        self,
        req: Req,
        insert_params: InsertParams,
        token_ids_len: int,
        is_finished: bool,
    ) -> Optional[int]:
        cache_len = (
            req.mamba_last_track_seqlen
            if self.enable_mamba_extra_buffer
            else token_ids_len
        )
# ... omitted for brevity ...
                )
                assert mamba_value_forked is not None, "Can not alloc mamba cache"
            insert_params.mamba_value = mamba_value_forked
            return cache_len
```
**EN:** Implements the prepare for caching REQ routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prepare for caching req例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 308-337: cleanup_after_caching_req implementation / cleanup_after_caching_req 实现
```python
    def cleanup_after_caching_req(
        self,
        req: Req,
        is_finished: bool,
        insert_result: Optional[InsertResult] = None,
        insert_params: Optional[InsertParams] = None,
    ) -> None:
        if is_finished:
            mamba_exist = (
                insert_result.mamba_exist if insert_result is not None else True
            )
            if self.enable_mamba_extra_buffer:
# ... omitted for brevity ...
                insert_result is None or insert_result.mamba_exist
            ):
                self.cache.req_to_token_pool.mamba_pool.free(insert_params.mamba_value)
            req.mamba_last_track_seqlen = None
```
**EN:** Implements the cleanup after caching REQ routine for this scope. It belongs to `MambaComponent`.
**CN:** 实现当前作用域中的cleanup after caching req例程。 该方法属于 `MambaComponent`。

### Lines 338-340: Comment block / 注释块
```python

    # ---- HiCache Hooks ----

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 341-395: build_hicache_transfers implementation / build_hicache_transfers 实现
```python
    def build_hicache_transfers(
        self, node: UnifiedTreeNode, phase: CacheTransferPhase, **kw
    ) -> Optional[list[PoolTransfer]]:
        ct = self.component_type

        if phase == CacheTransferPhase.BACKUP_HOST:
            cd = node.component_data[ct]
            if cd.value is None:
                return None
            return [
                PoolTransfer(
                    name=PoolName.MAMBA,
# ... omitted for brevity ...

            return transfers if transfers else None

        return None
```
**EN:** Builds a derived structure from the available configuration or inputs. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 397-424: commit_hicache_transfer implementation / commit_hicache_transfer 实现
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
                if host_lru.in_list(node):
                    host_lru.remove_node(node)
                self.cache.lru_lists[ct].insert_mru(node)
                self.cache.component_evictable_size_[ct] += count
```
**EN:** Implements the commit hicache transfer routine for this scope. It belongs to `MambaComponent`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的commit hicache transfer例程。 该方法属于 `MambaComponent`。它会向调用方返回计算结果。

### Lines 426-448: drive_host_eviction implementation / drive_host_eviction 实现
```python
    def drive_host_eviction(
        self, num_tokens: int, tracker: dict[ComponentType, int]
    ) -> None:
        """Evict mamba host resources.
        Internal nodes: private tombstone (free host mamba only).
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
**EN:** Evict mamba host resources. Implements the drive host eviction routine for this scope. It belongs to `MambaComponent`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的drive主机侧eviction例程。 该方法属于 `MambaComponent`。实现过程中会遍历输入或受管条目。

## Key Concepts / 关键概念
- **`MambaComponent`**: Defines the `MambaComponent` type and its core responsibilities. / 定义 `MambaComponent` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.unified_cache_components.tree_component`, `sglang.srt.server_args`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.unified_radix_cache`, `sglang.srt.mem_cache.memory_pool`
