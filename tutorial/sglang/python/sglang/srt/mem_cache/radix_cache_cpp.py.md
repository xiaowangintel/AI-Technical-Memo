# radix_cache_cpp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/radix_cache_cpp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the radix cache CPP logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的基数缓存C++相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
import time
from typing import TYPE_CHECKING, List, Optional, Set

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    BasePrefixCache,
    DecLockRefParams,
    DecLockRefResult,
# ... omitted for brevity ...
    RadixTreeCpp,
    TreeNodeCpp,
)
from sglang.srt.mem_cache.radix_cache import RadixKey
```
**EN:** Imports `__future__`, `logging`, `time`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `time`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 26-29: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.server_args import ServerArgs
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 32-32: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 35-35: RadixCacheCpp declaration / RadixCacheCpp 声明
```python
class RadixCacheCpp(BasePrefixCache):
```
**EN:** Declares the `RadixCacheCpp` class and connects it to `BasePrefixCache`.
**CN:** 声明 `RadixCacheCpp` 类，并将其关联到 `BasePrefixCache`。

### Lines 36-78: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        params: CacheInitParams,
        server_args: ServerArgs,
        enable_write_cancel: bool = False,
    ):
        self.disable = params.disable
        self.enable_write_cancel = enable_write_cancel

        assert (
            params.enable_kv_cache_events is False
        ), "HiRadixCache does not support kv cache events yet"
# ... omitted for brevity ...
            self.cache_controller = None
            return  # early return if hicache is not used

        raise NotImplementedError("Host cache is not supported yet")
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `RadixCacheCpp`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `RadixCacheCpp`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 80-93: _merge_tensor implementation / _merge_tensor 实现
```python
    def _merge_tensor(self, l: List[torch.Tensor]) -> torch.Tensor:
        """
        Merge a list of tensors into a single tensor.
        Args:
            l (List[torch.Tensor]): List of tensors to merge.
        Returns:
            torch.Tensor: Merged tensor.
        """
        if len(l) == 0:
            return torch.empty(0, dtype=torch.int64, device=self.device)
        elif len(l) == 1:
            return l[0]
        else:
            return torch.cat(l)
```
**EN:** Merge a list of tensors into a single tensor. Implements the merge tensor routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的merge张量例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 95-99: reset implementation / reset 实现
```python
    def reset(self):
        if self.cache_controller is not None:
            # need to clear the acks before resetting the cache controller
            raise NotImplementedError("Host cache is not supported yet")
        self.tree.reset()
```
**EN:** Resets the component back to its starting state. It belongs to `RadixCacheCpp`. It validates error cases explicitly.
**CN:** 将组件恢复到初始状态。 该方法属于 `RadixCacheCpp`。它会显式处理错误场景。

### Lines 101-112: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        key = params.key
        device_indices_vec, host_indices_length, node_gpu, node_cpu = (
            self.tree.match_prefix(key.token_ids)
        )
        return MatchResult(
            device_indices=self._merge_tensor(device_indices_vec),
            last_device_node=node_gpu,
            last_host_node=node_cpu,
            best_match_node=node_cpu,
            host_hit_length=host_indices_length,
        )
```
**EN:** Matches input data against cached state and returns the best fit. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 114-128: _insert implementation / _insert 实现
```python
    def _insert(self, key: RadixKey, value: torch.Tensor) -> int:
        """
        Insert a key-value pair into the radix tree.
        Args:
            key (RadixKey): The key to insert, represented as a RadixKey.
            value (torch.Tensor): The value to associate with the key.
        Returns:
            int: Number of device indices that were already present in the tree before the insertion.
        """
        ongoing_write, length = self.tree.writing_through(key.token_ids, value)
        if self.cache_controller is None:
            assert len(ongoing_write) == 0, "Implementation error"
            return length

        raise NotImplementedError("Host cache is not supported yet")
```
**EN:** Insert a key-value pair into the radix tree. Implements the insert routine for this scope. It belongs to `RadixCacheCpp`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert例程。 该方法属于 `RadixCacheCpp`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 130-139: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: TreeNodeCpp, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        """
        Decrement the reference count of a node to root of the radix tree.
        Args:
            node (TreeNodeCpp): The handle of the node to decrement the reference count for.
        """
        self.tree.lock_ref(node, False)  # do not increment
        return DecLockRefResult()
```
**EN:** Decrement the reference count of a node to root of the radix tree. Implements the DEC lock REF routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 141-148: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: TreeNodeCpp) -> IncLockRefResult:
        """
        Increment the reference count of from a node to root of the radix tree.
        Args:
            node (TreeNodeCpp): The handle of the node to increment the reference count for.
        """
        self.tree.lock_ref(node, True)
        return IncLockRefResult()
```
**EN:** Increment the reference count of from a node to root of the radix tree. Implements the INC lock REF routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 150-161: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        start_time = time.perf_counter()
        num_tokens = params.num_tokens
        evicted_device_indices = self.tree.evict(num_tokens)

        num_evicted = 0
        for indice in evicted_device_indices:
            num_evicted += len(indice)
            self.token_to_kv_pool_allocator.free(indice)

        self.update_eviction_metrics(num_evicted, start_time)
        return EvictResult(num_tokens_evicted=num_evicted)
```
**EN:** Removes cache entries according to the active policy. It belongs to `RadixCacheCpp`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `RadixCacheCpp`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 163-164: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self):
        return self.tree.evictable_size()
```
**EN:** Implements the evictable size routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 166-167: protected_size implementation / protected_size 实现
```python
    def protected_size(self):
        return self.tree.protected_size()
```
**EN:** Implements the protected size routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 169-170: total_size implementation / total_size 实现
```python
    def total_size(self):
        return self.tree.total_size()
```
**EN:** Implements the total size routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

### Lines 172-208: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True):
        """Cache request when it finishes."""
        assert req.req_pool_idx is not None
        kv_committed_len = req.pop_committed_kv_cache()
        token_ids = (req.origin_input_ids + req.output_ids)[:kv_committed_len]
        kv_indices = self.req_to_token_pool.req_to_token[
            req.req_pool_idx, :kv_committed_len
        ].to(dtype=torch.int64, copy=True)

        # NOTE: our C++ implementation don't need `token_ids` and `kv_indices` to be page-aligned
        # it will automatically align them, but length of them should be equal
        old_prefix_len = len(req.prefix_indices) // self.page_size * self.page_size
# ... omitted for brevity ...
            self.token_to_kv_pool_allocator.free(kv_indices[page_aligned_overall_len:])

        # Remove req slot release the cache lock
        self.dec_lock_ref(req.last_node)
```
**EN:** Cache request when it finishes. Implements the cache finished REQ routine for this scope. It belongs to `RadixCacheCpp`.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `RadixCacheCpp`。

### Lines 210-258: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False):
        """Cache request when it is unfinished."""
        assert req.req_pool_idx is not None
        token_ids = req.fill_ids
        prefill_len = len(token_ids)  # prefill only (maybe chunked)
        kv_indices = self.req_to_token_pool.req_to_token[
            req.req_pool_idx, :prefill_len
        ].to(dtype=torch.int64, copy=True)

        # NOTE: our C++ implementation don't need `token_ids` and `kv_indices` to be page-aligned
        # it will automatically align them, but length of them should be equal
        old_prefix_len = len(req.prefix_indices) // self.page_size * self.page_size
# ... omitted for brevity ...
            )
        else:
            req.prefix_indices = new_indices
        req.last_node = new_last_node
```
**EN:** Cache request when it is unfinished. Implements the cache unfinished REQ routine for this scope. It belongs to `RadixCacheCpp`.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `RadixCacheCpp`。

### Lines 260-261: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self):
        return self.tree.debug_print()
```
**EN:** Implements the pretty print routine for this scope. It belongs to `RadixCacheCpp`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `RadixCacheCpp`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`RadixCacheCpp`**: Defines the `RadixCacheCpp` type and its core responsibilities. / 定义 `RadixCacheCpp` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `time`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cpp_radix_tree.radix_tree`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.server_args`
