# chunk_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/chunk_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the chunk cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的块缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and setup / 导入与初始化
```python
from __future__ import annotations

"""Cache for chunked prefill, used when RadixCache is disabled."""

import logging
from typing import TYPE_CHECKING, Any, Optional

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    BasePrefixCache,
    DecLockRefParams,
# ... omitted for brevity ...
from sglang.srt.mem_cache.hisparse_memory_pool import (
    DeepSeekV4HiSparseTokenToKVPoolAllocator,
)
from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator
```
**EN:** Imports `__future__`, `logging`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hisparse_memory_pool` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hisparse_memory_pool` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 27-29: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 32-32: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 35-42: ChunkCache declaration / ChunkCache 声明
```python
class ChunkCache(BasePrefixCache):
    """
    ChunkCache is used when radix cache is disabled.

    That includes standard chunked-prefill setups and the decode side of P/D
    disaggregation when decode radix cache is not enabled.
    """

```
**EN:** ChunkCache is used when radix cache is disabled. Declares the `ChunkCache` class and connects it to `BasePrefixCache`.
**CN:** 声明 `ChunkCache` 类，并将其关联到 `BasePrefixCache`。

### Lines 43-52: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams):
        self.req_to_token_pool = params.req_to_token_pool
        self.token_to_kv_pool_allocator = params.token_to_kv_pool_allocator
        self.page_size = params.page_size
        if self.token_to_kv_pool_allocator:
            self.device = self.token_to_kv_pool_allocator.device
        else:
            self.device = torch.device("cpu")

        self.protected_size_ = 0
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `ChunkCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `ChunkCache`。

### Lines 54-55: is_chunk_cache implementation / is_chunk_cache 实现
```python
    def is_chunk_cache(self) -> bool:
        return True
```
**EN:** Checks whether a condition holds for the current state. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 检查当前状态是否满足某个条件。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 56-59: Comment block / 注释块
```python

    # NOTE (csy): this is to determine if a cache has prefix matching feature.
    # Chunk cache always return True to indicate no prefix matching.
    # TODO (csy): Using a prefix cache trait to replace this
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 60-62: disable implementation / disable 实现
```python
    @property
    def disable(self):
        return True
```
**EN:** Implements the disable routine for this scope. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的disable例程。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 64-65: reset implementation / reset 实现
```python
    def reset(self):
        pass
```
**EN:** Resets the component back to its starting state. It belongs to `ChunkCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `ChunkCache`。

### Lines 67-73: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        return MatchResult(
            device_indices=torch.empty((0,), dtype=torch.int64),
            last_device_node=None,
            last_host_node=None,
            best_match_node=None,
        )
```
**EN:** Matches input data against cached state and returns the best fit. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 75-77: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        # ChunkCache does not support prefix caching, so insert is a no-op
        return InsertResult(prefix_len=0)
```
**EN:** Inserts new data into the managed structure. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 79-85: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True):
        kv_committed_len = req.pop_committed_kv_cache()
        # For decode server: if req.output_ids is empty, we want to free all req.origin_input_ids
        kv_indices = self.req_to_token_pool.req_to_token[
            req.req_pool_idx, :kv_committed_len
        ]
        self.token_to_kv_pool_allocator.free(kv_indices)
```
**EN:** Implements the cache finished REQ routine for this scope. It belongs to `ChunkCache`.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `ChunkCache`。

### Lines 87-92: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False):
        kv_indices = self.req_to_token_pool.req_to_token[
            req.req_pool_idx, : len(req.fill_ids)
        ]
        # `req.prefix_indices` will be used in `PrefillAdder::add_chunked_req` later
        req.prefix_indices = kv_indices.to(dtype=torch.int64, copy=True)
```
**EN:** Implements the cache unfinished REQ routine for this scope. It belongs to `ChunkCache`.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `ChunkCache`。

### Lines 94-95: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        return EvictResult()
```
**EN:** Removes cache entries according to the active policy. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 97-98: inc_lock_ref implementation / inc_lock_ref 实现
```python
    def inc_lock_ref(self, node: Any) -> IncLockRefResult:
        return IncLockRefResult(delta=0)
```
**EN:** Implements the INC lock REF routine for this scope. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 100-103: dec_lock_ref implementation / dec_lock_ref 实现
```python
    def dec_lock_ref(
        self, node: Any, params: Optional[DecLockRefParams] = None
    ) -> DecLockRefResult:
        return DecLockRefResult(delta=0)
```
**EN:** Implements the DEC lock REF routine for this scope. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 105-107: protected_size implementation / protected_size 实现
```python
    def protected_size(self):
        # NOTE: no protected size in chunk cache. Chunk cache's eviction is the same with request's lifecycle.
        return 0
```
**EN:** Implements the protected size routine for this scope. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 109-110: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self):
        return ""
```
**EN:** Implements the pretty print routine for this scope. It belongs to `ChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `ChunkCache`。它会向调用方返回计算结果。

### Lines 113-115: SWAChunkCache declaration / SWAChunkCache 声明
```python
class SWAChunkCache(ChunkCache):
    """ChunkCache with support for sliding window attention."""

```
**EN:** ChunkCache with support for sliding window attention. Declares the `SWAChunkCache` class and connects it to `ChunkCache`.
**CN:** 声明 `SWAChunkCache` 类，并将其关联到 `ChunkCache`。

### Lines 116-128: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams):
        # DeepSeek V4 HiSparse wraps SWATokenToKVPoolAllocator and exposes the same API.
        assert isinstance(
            params.token_to_kv_pool_allocator,
            (
                SWATokenToKVPoolAllocator,
                DeepSeekV4HiSparseTokenToKVPoolAllocator,
            ),
        )
        super().__init__(params)

        self.sliding_window_size = params.sliding_window_size
        self.chunked_prefill_size = params.chunked_prefill_size
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SWAChunkCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SWAChunkCache`。

### Lines 130-134: supports_swa implementation / supports_swa 实现
```python
    def supports_swa(self) -> bool:
        assert (
            self.sliding_window_size is not None
        ), "sliding_window_size must be set for SWAChunkCache"
        return True
```
**EN:** Implements the supports SWA routine for this scope. It belongs to `SWAChunkCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的supports SWA例程。 该方法属于 `SWAChunkCache`。它会向调用方返回计算结果。

### Lines 136-137: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        return EvictResult()
```
**EN:** Removes cache entries according to the active policy. It belongs to `SWAChunkCache`. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `SWAChunkCache`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`ChunkCache`**: Defines the `ChunkCache` type and its core responsibilities. / 定义 `ChunkCache` 类型及其核心职责。
- **`SWAChunkCache`**: Defines the `SWAChunkCache` type and its core responsibilities. / 定义 `SWAChunkCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.hisparse_memory_pool`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`
