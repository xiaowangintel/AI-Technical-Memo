# radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and setup / 导入与初始化
```python
from __future__ import annotations

from sglang.srt.mem_cache.cache_init_params import CacheInitParams

"""
Copyright 2023-2024 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

# ... omitted for brevity ...
from functools import lru_cache
from typing import TYPE_CHECKING, Any, Iterator, List, Optional, Tuple, Union

import torch
```
**EN:** Imports `__future__`, `sglang.srt.mem_cache.cache_init_params`, `hashlib`, `heapq`, `logging`, `sys` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `sglang.srt.mem_cache.cache_init_params`, `hashlib`, `heapq`, `logging`, `sys` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 35-35: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 37-60: Imports and setup / 导入与初始化
```python
from sglang.srt.mem_cache.base_prefix_cache import (
    BasePrefixCache,
    DecLockRefParams,
    DecLockRefResult,
    EvictParams,
    EvictResult,
    IncLockRefResult,
    InsertParams,
    InsertResult,
    MatchPrefixParams,
    MatchResult,
)
# ... omitted for brevity ...
    PriorityStrategy,
    SLRUStrategy,
)
from sglang.srt.mem_cache.utils import split_node_hash_value
```
**EN:** Imports `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.events`, `sglang.srt.mem_cache.evict_policy`, `sglang.srt.mem_cache.utils` and other helpers used by the surrounding scope.
**CN:** 导入 `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.events`, `sglang.srt.mem_cache.evict_policy`, `sglang.srt.mem_cache.utils` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 62-63: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 66-68: RadixKey declaration / RadixKey 声明
```python
class RadixKey:
    """is_bigram=True: token_ids holds raw tokens (N+1 for N bigrams); slices share one boundary token."""

```
**EN:** is_bigram=True: token_ids holds raw tokens (N+1 for N bigrams); slices share one boundary token. Declares the `RadixKey` class.
**CN:** 声明 `RadixKey` 类。

### Lines 69-69: Shared state definitions / 共享状态定义
```python
    __slots__ = ("token_ids", "extra_key", "is_bigram")
```
**EN:** Defines class-level variables such as `__slots__`.
**CN:** 定义类级变量，例如 `__slots__`。

### Lines 71-82: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        token_ids: List[int],
        extra_key: Optional[str] = None,
        is_bigram: bool = False,
    ):
        # token ids sequence (raw ints in both modes)
        self.token_ids = token_ids
        # extra key (e.g. lora_id, cache_salt)
        self.extra_key = extra_key
        # bigram view over token_ids: length = max(0, len(token_ids) - 1)
        self.is_bigram = is_bigram
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `RadixKey`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `RadixKey`。

### Lines 84-88: __len__ implementation / __len__ 实现
```python
    def __len__(self) -> int:
        if self.is_bigram:
            n = len(self.token_ids)
            return n - 1 if n > 0 else 0
        return len(self.token_ids)
```
**EN:** Returns the logical length exposed by the object. It belongs to `RadixKey`. It returns a computed result to its caller.
**CN:** 返回对象对外暴露的逻辑长度。 该方法属于 `RadixKey`。它会向调用方返回计算结果。

### Lines 90-96: __iter__ implementation / __iter__ 实现
```python
    def __iter__(self) -> Iterator:
        if self.is_bigram:
            t = self.token_ids
            for i in range(len(t) - 1):
                yield (t[i], t[i + 1])
        else:
            yield from self.token_ids
```
**EN:** Iterates through the logical items managed by the object. It belongs to `RadixKey`. The implementation iterates over inputs or managed entries.
**CN:** 遍历对象管理的逻辑元素。 该方法属于 `RadixKey`。实现过程中会遍历输入或受管条目。

### Lines 98-115: __getitem__ implementation / __getitem__ 实现
```python
    def __getitem__(self, idx: Union[int, slice]) -> "RadixKey":
        # Normalize int -> 1-element slice so the rest handles one shape.
        if isinstance(idx, int):
            if idx < 0:
                idx += len(self)
            if idx < 0 or idx >= len(self):
                raise IndexError(f"RadixKey index out of range: {idx}")
            idx = slice(idx, idx + 1)
        start, stop, step = idx.indices(len(self))
        if step != 1:
            raise ValueError("RadixKey slice step must be 1")

        if self.is_bigram:
            # bigrams [start, stop) span raw tokens [start, stop + 1);
            # empty slice -> empty raw tokens (not a dangling boundary token).
            raw = self.token_ids[start : stop + 1] if stop > start else []
            return RadixKey(raw, self.extra_key, is_bigram=True)
        return RadixKey(self.token_ids[start:stop], self.extra_key)
```
**EN:** Supports indexed or sliced access to the underlying data. It belongs to `RadixKey`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 支持对底层数据进行索引或切片访问。 该方法属于 `RadixKey`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 117-119: __repr__ implementation / __repr__ 实现
```python
    def __repr__(self) -> str:
        preview = self.token_ids[:10]
        return f"RadixKey(extra_key={self.extra_key!r}, token_ids={preview}{'...' if len(self.token_ids) > 10 else ''}, is_bigram={self.is_bigram})"
```
**EN:** Builds a debug-friendly string representation for the object. It belongs to `RadixKey`. It returns a computed result to its caller.
**CN:** 生成便于调试的字符串表示。 该方法属于 `RadixKey`。它会向调用方返回计算结果。

### Lines 121-125: page_aligned implementation / page_aligned 实现
```python
    def page_aligned(self, page_size: int) -> "RadixKey":
        if page_size == 1:
            return self
        aligned_len = len(self) // page_size * page_size
        return self[:aligned_len]
```
**EN:** Implements the page aligned routine for this scope. It belongs to `RadixKey`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的页aligned例程。 该方法属于 `RadixKey`。它会向调用方返回计算结果。

### Lines 127-138: maybe_to_bigram_view implementation / maybe_to_bigram_view 实现
```python
    def maybe_to_bigram_view(
        self,
        is_eagle: bool,
        value: Optional[torch.Tensor] = None,
    ) -> Tuple["RadixKey", Optional[torch.Tensor]]:
        # O(1): flip the bigram flag instead of materializing a tuple list.
        # value is paired with raw tokens and gets truncated to the bigram count.
        if is_eagle and not self.is_bigram:
            self.is_bigram = True
            if value is not None:
                value = value[: len(self)]
        return self, value
```
**EN:** Implements the maybe TO bigram view routine for this scope. It belongs to `RadixKey`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe to bigram view例程。 该方法属于 `RadixKey`。它会向调用方返回计算结果。

### Lines 140-145: _check_compatible implementation / _check_compatible 实现
```python
    def _check_compatible(self, other: "RadixKey") -> None:
        if self.extra_key != other.extra_key:
            raise ValueError(
                f"RadixKey operations require matching extra_key, but got "
                f"{self.extra_key=} != {other.extra_key=}"
            )
```
**EN:** Implements the check compatible routine for this scope. It belongs to `RadixKey`. It validates error cases explicitly.
**CN:** 实现当前作用域中的check compatible例程。 该方法属于 `RadixKey`。它会显式处理错误场景。

### Lines 147-176: match implementation / match 实现
```python
    def match(self, other: "RadixKey", page_size: int = 1) -> int:
        """Logical-unit prefix length shared with ``other``. Result is rounded down to ``page_size``."""
        self._check_compatible(other)
        t0, t1 = self.token_ids, other.token_ids

        if self.is_bigram:
            # Walk raw tokens; L matching tokens imply L-1 matching bigrams.
            i = 0
            for a, b in zip(t0, t1):
                if a != b:
                    break
                i += 1
# ... omitted for brevity ...
            if t0[i : i + page_size] != t1[i : i + page_size]:
                break
            i += page_size
        return i
```
**EN:** Logical-unit prefix length shared with ``other``. Matches input data against cached state and returns the best fit. It belongs to `RadixKey`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `RadixKey`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 178-188: child_key implementation / child_key 实现
```python
    def child_key(self, page_size: int = 1):
        """Hashable dict-key for the first ``page_size`` logical units, namespaced by ``extra_key``."""
        t = self.token_ids
        if self.is_bigram:
            if page_size == 1:
                plain = (t[0], t[1])
            else:
                plain = tuple((t[j], t[j + 1]) for j in range(page_size))
        else:
            plain = t[0] if page_size == 1 else tuple(t[:page_size])
        return plain if self.extra_key is None else (self.extra_key, plain)
```
**EN:** Hashable dict-key for the first ``page_size`` logical units, namespaced by ``extra_key``. Implements the child KEY routine for this scope. It belongs to `RadixKey`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的child key例程。 该方法属于 `RadixKey`。它会向调用方返回计算结果。

### Lines 190-203: hash_page implementation / hash_page 实现
```python
    def hash_page(self, start: int, end: int, prior_hash: Optional[str] = None) -> str:
        """SHA256 for logical units [start, end); bigram mode feeds overlapping (t_i, t_{i+1}) byte pairs."""
        hasher = hashlib.sha256()
        if prior_hash:
            hasher.update(bytes.fromhex(prior_hash))
        t = self.token_ids
        if self.is_bigram:
            for j in range(start, end):
                hasher.update(t[j].to_bytes(4, byteorder="little", signed=False))
                hasher.update(t[j + 1].to_bytes(4, byteorder="little", signed=False))
        else:
            for j in range(start, end):
                hasher.update(t[j].to_bytes(4, byteorder="little", signed=False))
        return hasher.hexdigest()
```
**EN:** SHA256 for logical units [start, end); bigram mode feeds overlapping (t_i, t_{i+1}) byte pairs. Implements the hash page routine for this scope. It belongs to `RadixKey`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的hash页例程。 该方法属于 `RadixKey`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 206-207: TreeNode declaration / TreeNode 声明
```python
class TreeNode:

```
**EN:** Declares the `TreeNode` class.
**CN:** 声明 `TreeNode` 类。

### Lines 208-208: Shared state definitions / 共享状态定义
```python
    counter = 0
```
**EN:** Defines class-level variables such as `counter`.
**CN:** 定义类级变量，例如 `counter`。

### Lines 210-231: __init__ implementation / __init__ 实现
```python
    def __init__(self, id: Optional[int] = None, priority: int = 0):
        self.children = defaultdict(TreeNode)
        self.parent: TreeNode = None
        self.key: RadixKey = None
        self.value: Optional[torch.Tensor] = None
        self.lock_ref = 0
        self.last_access_time = time.monotonic()
        self.creation_time = time.monotonic()

        self.hit_count = 0
        # indicating the node is locked to protect from eviction
        # incremented when the node is referenced by a storage operation
# ... omitted for brevity ...
        self.priority = priority

        self.id = TreeNode.counter if id is None else id
        TreeNode.counter += 1
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `TreeNode`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `TreeNode`。

### Lines 233-235: evicted implementation / evicted 实现
```python
    @property
    def evicted(self):
        return self.value is None
```
**EN:** Implements the evicted routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evicted例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 237-239: backuped implementation / backuped 实现
```python
    @property
    def backuped(self):
        return self.host_value is not None
```
**EN:** Implements the backuped routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backuped例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 241-243: protect_host implementation / protect_host 实现
```python
    def protect_host(self):
        """Protect the host value from eviction."""
        self.host_ref_counter += 1
```
**EN:** Protect the host value from eviction. Implements the protect host routine for this scope. It belongs to `TreeNode`.
**CN:** 实现当前作用域中的protect主机侧例程。 该方法属于 `TreeNode`。

### Lines 245-250: release_host implementation / release_host 实现
```python
    def release_host(self):
        """Release the host value, allowing it to be evicted."""
        if self.host_ref_counter > 0:
            self.host_ref_counter -= 1
        else:
            raise RuntimeError("Host reference counter is already zero.")
```
**EN:** Release the host value, allowing it to be evicted. Implements the release host routine for this scope. It belongs to `TreeNode`. It validates error cases explicitly.
**CN:** 实现当前作用域中的release主机侧例程。 该方法属于 `TreeNode`。它会显式处理错误场景。

### Lines 252-256: get_last_hash_value implementation / get_last_hash_value 实现
```python
    def get_last_hash_value(self) -> Optional[str]:
        """Returns the hash value of the last page in this node."""
        if self.hash_value is None or len(self.hash_value) == 0:
            return None
        return self.hash_value[-1]
```
**EN:** Returns the hash value of the last page in this node. Retrieves the requested data or state from the current object. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 258-263: get_prefix_hash_values implementation / get_prefix_hash_values 实现
```python
    @lru_cache(maxsize=1)
    def get_prefix_hash_values(self, node: TreeNode) -> List[str]:
        if node is None or node.hash_value is None:
            return []

        return node.get_prefix_hash_values(node.parent) + node.hash_value
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 265-266: __lt__ implementation / __lt__ 实现
```python
    def __lt__(self, other: "TreeNode"):
        return self.last_access_time < other.last_access_time
```
**EN:** Implements the LT routine for this scope. It belongs to `TreeNode`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的lt例程。 该方法属于 `TreeNode`。它会向调用方返回计算结果。

### Lines 269-269: RadixCache declaration / RadixCache 声明
```python
class RadixCache(KVCacheEventMixin, BasePrefixCache):
```
**EN:** Declares the `RadixCache` class and connects it to `KVCacheEventMixin`, `BasePrefixCache`.
**CN:** 声明 `RadixCache` 类，并将其关联到 `KVCacheEventMixin`, `BasePrefixCache`。

### Lines 270-315: __init__ implementation / __init__ 实现
```python
    def __init__(self, params: CacheInitParams):
        self.disable = params.disable
        self.req_to_token_pool = params.req_to_token_pool
        self.token_to_kv_pool_allocator = params.token_to_kv_pool_allocator
        self.page_size = params.page_size
        self.enable_kv_cache_events = params.enable_kv_cache_events
        self.is_eagle = params.is_eagle
        self.disable_finished_insert = params.disable_finished_insert
        self.eviction_policy = params.eviction_policy.lower()

        self.kv_event_queue = []

# ... omitted for brevity ...
            )

        self.evictable_leaves = set()
        self.reset()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `RadixCache`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `RadixCache`。它会显式处理错误场景。

### Lines 317-333: create_simulated implementation / create_simulated 实现
```python
    @classmethod
    def create_simulated(
        self,
        disable: bool = False,
        mock_allocator: Optional[Any] = None,
        page_size: int = 1,
        enable_kv_cache_events: bool = False,
    ) -> RadixCache:
        """Init a radix cache without memory pools for simulation purpose."""
        params = CacheInitParams(
            disable=disable,
            req_to_token_pool=None,
            token_to_kv_pool_allocator=mock_allocator,
            page_size=page_size,
            enable_kv_cache_events=enable_kv_cache_events,
        )
        return RadixCache(params)
```
**EN:** Init a radix cache without memory pools for simulation purpose. Constructs a new object or resource with the requested configuration. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 334-336: Comment block / 注释块
```python

    ##### Public API #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 337-358: reset implementation / reset 实现
```python
    def reset(self):
        # Initialize root with minimum priority so any real priority overrides it
        self.root_node = TreeNode(priority=-sys.maxsize)
        self.root_node.key = RadixKey(token_ids=[], extra_key=None)
        self.root_node.value = []
        self.root_node.host_value = []
        self.root_node.lock_ref = 1
        self.root_node.hash_value = []
        self.evictable_size_ = 0
        self.protected_size_ = 0
        self.evictable_leaves.clear()
        self._empty_match_result = MatchResult(
# ... omitted for brevity ...
            last_host_node=self.root_node,
            best_match_node=self.root_node,
        )
        self._record_all_cleared_event()
```
**EN:** Resets the component back to its starting state. It belongs to `RadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `RadixCache`。

### Lines 360-418: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:
        """Find the longest cached prefix of ``key`` in the radix tree.

        The logical namespace for prefix matching is determined by both the
        token id sequence and the optional ``extra_key`` carried by ``RadixKey``.
        Entries that share identical leading token ids but have *different*
        ``extra_key`` values are intentionally kept disjoint and never share
        prefix nodes. This is useful to:

        * Isolate KV cache lines for different LoRA / adapter IDs.
        * Separate requests that intentionally should not share state (e.g.,
          different sampling salt, cache version, or retrieval augmentation
# ... omitted for brevity ...
            last_device_node=last_node,
            last_host_node=last_node,
            best_match_node=last_node,
        )
```
**EN:** Find the longest cached prefix of ``key`` in the radix tree. Matches input data against cached state and returns the best fit. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 420-438: insert implementation / insert 实现
```python
    def insert(self, params: InsertParams) -> InsertResult:
        if self.disable:
            return InsertResult(prefix_len=0)

        key = params.key
        value = params.value
        priority = params.priority
        chunked = params.chunked

        key, value = key.maybe_to_bigram_view(self.is_eagle, value)
        key = key.page_aligned(self.page_size)
        if value is not None:
# ... omitted for brevity ...
            value = torch.tensor(key.token_ids[: len(key)], dtype=torch.int64)

        prefix_len = self._insert_helper(self.root_node, key, value, priority, chunked)
        return InsertResult(prefix_len=prefix_len)
```
**EN:** Inserts new data into the managed structure. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 向受管结构中插入新数据。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 440-485: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True):
        """Cache request when it finishes."""
        # In deterministic mode, disable finished request insertion to radix cache
        if self.disable_finished_insert:
            is_insert = False

        kv_committed_len = req.pop_committed_kv_cache()
        if self.disable:
            kv_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, :kv_committed_len
            ]
            self.token_to_kv_pool_allocator.free(kv_indices)
# ... omitted for brevity ...

        # Remove req slot release the cache lock
        if req.last_node is not None:
            self.dec_lock_ref(req.last_node)
```
**EN:** Cache request when it finishes. Implements the cache finished REQ routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 487-551: cache_unfinished_req implementation / cache_unfinished_req 实现
```python
    def cache_unfinished_req(self, req: Req, chunked=False):
        """Cache request when it is unfinished."""
        if self.disable:
            return

        token_ids = req.fill_ids
        kv_indices = self.req_to_token_pool.req_to_token[
            req.req_pool_idx, : len(token_ids)
        ]

        radix_key = RadixKey(
            token_ids, req.extra_key, is_bigram=self.is_eagle
# ... omitted for brevity ...
        else:
            req.prefix_indices = new_indices

        req.last_node = new_last_node
```
**EN:** Cache request when it is unfinished. Implements the cache unfinished REQ routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存unfinished req例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 553-555: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self):
        self._print_helper(self.root_node, 0)
        print(f"#tokens: {self.total_size()}")
```
**EN:** Implements the pretty print routine for this scope. It belongs to `RadixCache`.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `RadixCache`。

### Lines 557-558: total_size implementation / total_size 实现
```python
    def total_size(self):
        return self._total_size_helper()
```
**EN:** Implements the total size routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 560-587: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        if self.disable:
            return EvictResult()

        start_time = time.perf_counter()
        num_tokens = params.num_tokens
        leaves = list(self.evictable_leaves)
        eviction_heap = [
            (self.eviction_strategy.get_priority(node), node) for node in leaves
        ]
        heapq.heapify(eviction_heap)

# ... omitted for brevity ...
            self._record_remove_event(x)

        self.update_eviction_metrics(num_evicted, start_time)
        return EvictResult(num_tokens_evicted=num_evicted)
```
**EN:** Removes cache entries according to the active policy. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 589-602: inc_lock_ref implementation / inc_lock_ref 实现
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
            node = node.parent
        return IncLockRefResult(delta=delta)
```
**EN:** Implements the INC lock REF routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc lock ref例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 604-623: dec_lock_ref implementation / dec_lock_ref 实现
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
**EN:** Implements the DEC lock REF routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的dec lock ref例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 625-626: evictable_size implementation / evictable_size 实现
```python
    def evictable_size(self):
        return self.evictable_size_
```
**EN:** Implements the evictable size routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的evictable size例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 628-630: protected_size implementation / protected_size 实现
```python
    def protected_size(self):
        # protected size refers to the size of the cache that is locked
        return self.protected_size_
```
**EN:** Implements the protected size routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的protected size例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 632-641: all_values_flatten implementation / all_values_flatten 实现
```python
    def all_values_flatten(self):
        values = []

        def _dfs_helper(node: TreeNode):
            for _, child in node.children.items():
                values.append(child.value)
                _dfs_helper(child)

        _dfs_helper(self.root_node)
        return torch.cat(values)
```
**EN:** Implements the ALL values flatten routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的all values flatten例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 642-644: Comment block / 注释块
```python

    ##### Internal Helper Functions #####

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 645-669: _match_prefix_helper implementation / _match_prefix_helper 实现
```python
    def _match_prefix_helper(self, node: TreeNode, key: RadixKey):
        access_time = time.monotonic()
        node.last_access_time = access_time

        child_key = key.child_key(self.page_size)

        value = []
        while len(key) > 0 and child_key in node.children.keys():
            child = node.children[child_key]
            child.last_access_time = access_time
            prefix_len = child.key.match(key, page_size=self.page_size)
            if prefix_len < len(child.key):
# ... omitted for brevity ...
                if len(key):
                    child_key = key.child_key(self.page_size)

        return value, node
```
**EN:** Implements the match prefix helper routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的match前缀helper例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 671-691: _split_node implementation / _split_node 实现
```python
    def _split_node(self, key: RadixKey, child: TreeNode, split_len: int):
        # new_node -> child
        # New node inherits child's priority (represents shared prefix)
        new_node = TreeNode(priority=child.priority)
        new_node.hit_count = child.hit_count
        new_node.children = {key[split_len:].child_key(self.page_size): child}
        new_node.parent = child.parent
        new_node.lock_ref = child.lock_ref
        new_node.key = child.key[:split_len]
        new_node.value = child.value[:split_len].clone()
        child.parent = new_node
        child.key = child.key[split_len:]
# ... omitted for brevity ...
            child.hash_value, split_len, self.page_size
        )

        return new_node
```
**EN:** Implements the split node routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 693-699: _inc_hit_count implementation / _inc_hit_count 实现
```python
    def _inc_hit_count(self, node: TreeNode, chunked: bool = False):
        # Skip the hit count update for chunked requests to avoid self-referencing
        # inflation where a chunked request increments hit_count on nodes it created
        # in previous chunks.
        if chunked:
            return
        node.hit_count += 1
```
**EN:** Implements the INC HIT count routine for this scope. It belongs to `RadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的inc hit count例程。 该方法属于 `RadixCache`。它会向调用方返回计算结果。

### Lines 701-753: _insert_helper implementation / _insert_helper 实现
```python
    def _insert_helper(
        self,
        node: TreeNode,
        key: RadixKey,
        value,
        priority: int = 0,
        chunked: bool = False,
    ):
        # Convert None priority to 0
        if priority is None:
            priority = 0
        access_time = time.monotonic()
# ... omitted for brevity ...
            self._update_leaf_status(new_node)
            # Hash will be computed lazily during event emission
            self._record_store_event(new_node)
        return total_prefix_length
```
**EN:** Implements the insert helper routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的insert helper例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 755-771: _print_helper implementation / _print_helper 实现
```python
    def _print_helper(self, node: TreeNode, indent: int):
        """Prints the radix tree in a human-readable format."""
        stack = [(node, indent)]
        while stack:
            current_node, current_indent = stack.pop()
            print(
                " " * current_indent,
                len(current_node.key),
                current_node.key.token_ids[:10],
                f"r={current_node.lock_ref}",
            )
            for key, child in current_node.children.items():
                stack.append((child, current_indent + 2))

                assert key == child.key.child_key(
                    self.page_size
                ), f"{key=}, {child.key.child_key(self.page_size)=}"
```
**EN:** Prints the radix tree in a human-readable format. Implements the print helper routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的print helper例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。

### Lines 773-781: _delete_leaf implementation / _delete_leaf 实现
```python
    def _delete_leaf(self, node):
        key = node.key.child_key(self.page_size)
        v = node.parent.children.pop(key, None)
        assert v == node, f"parent does not have child key, {key}"

        self.evictable_size_ -= len(node.key)
        if node in self.evictable_leaves:
            self.evictable_leaves.remove(node)
        self._update_leaf_status(node.parent)
```
**EN:** Implements the delete leaf routine for this scope. It belongs to `RadixCache`.
**CN:** 实现当前作用域中的delete leaf例程。 该方法属于 `RadixCache`。

### Lines 783-796: _update_leaf_status implementation / _update_leaf_status 实现
```python
    def _update_leaf_status(self, node: TreeNode):
        if node.evicted or node.lock_ref > 0:
            if node in self.evictable_leaves:
                self.evictable_leaves.remove(node)
            return

        for child in node.children.values():
            if not child.evicted:
                if node in self.evictable_leaves:
                    self.evictable_leaves.remove(node)
                return

        if node not in self.evictable_leaves:
            self.evictable_leaves.add(node)
```
**EN:** Implements the update leaf status routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的update leaf status例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 798-808: _total_size_helper implementation / _total_size_helper 实现
```python
    def _total_size_helper(self):
        total_size = 0
        stack = [self.root_node]
        while stack:
            current_node = stack.pop()
            total_size += len(current_node.value)
            for child in current_node.children.values():
                if child.evicted:
                    continue
                stack.append(child)
        return total_size
```
**EN:** Implements the total size helper routine for this scope. It belongs to `RadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的total size helper例程。 该方法属于 `RadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 811-830: Script entry point / 脚本入口
```python
if __name__ == "__main__":
    tree = RadixCache.create_simulated()

    # Example token id sequences (as lists of ints)
    tree.insert(InsertParams(key=RadixKey(token_ids=[1, 2, 3], extra_key=None)))
    tree.insert(InsertParams(key=RadixKey(token_ids=[1, 2, 3], extra_key=None)))
    tree.insert(InsertParams(key=RadixKey(token_ids=[1, 2, 4, 5], extra_key=None)))
    tree.insert(
        InsertParams(key=RadixKey(token_ids=[1, 2, 4, 5, 6, 7], extra_key=None))
    )
    tree.insert(
        InsertParams(key=RadixKey(token_ids=[8, 9, 10, 11, 12], extra_key=None))
# ... omitted for brevity ...
        tree.match_prefix(
            MatchPrefixParams(key=RadixKey(token_ids=[1, 2, 3, 13, 14], extra_key=None))
        )
    )
```
**EN:** Provides the executable entry point for running this module directly.
**CN:** 为直接运行该模块提供可执行入口。

## Key Concepts / 关键概念
- **`RadixKey`**: Defines the `RadixKey` type and its core responsibilities. / 定义 `RadixKey` 类型及其核心职责。
- **`TreeNode`**: Defines the `TreeNode` type and its core responsibilities. / 定义 `TreeNode` 类型及其核心职责。
- **`RadixCache`**: Defines the `RadixCache` type and its core responsibilities. / 定义 `RadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `hashlib`, `heapq`, `logging`, `sys`, `time`, `collections`, `functools`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.events`, `sglang.srt.mem_cache.evict_policy`, `sglang.srt.mem_cache.utils`, `sglang.srt.managers.schedule_batch`
