# lmc_radix_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/lmcache/lmc_radix_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the LMC radix cache logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的lmc基数缓存相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
import threading
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    EvictParams,
    EvictResult,
    MatchPrefixParams,
    MatchResult,
)
from sglang.srt.mem_cache.radix_cache import RadixCache, RadixKey, TreeNode
```
**EN:** Imports `__future__`, `logging`, `threading`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `threading`, `typing`, `torch`, `sglang.srt.mem_cache.base_prefix_cache` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 17-26: Control flow block / 控制流代码块
```python
try:
    from lmcache.integration.sglang.sglang_adapter import (
        LMCacheLayerwiseConnector,
        LoadMetadata,
        StoreMetadata,
    )
except ImportError as e:
    raise RuntimeError(
        "LMCache is not installed. Please install it by running `pip install lmcache`"
    ) from e
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 29-32: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 34-34: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 37-44: LayerTransferCounter declaration / LayerTransferCounter 声明
```python
class LayerTransferCounter:
    """Minimal adapter that lets the memory pool notify LMCache per-layer.

    The KV pool calls `wait_until(layer_id)` after finishing a layer, which we
    translate into a `load_kv_layerwise(layer_id)` call on the LMCache connector
    within the provided CUDA stream.
    """

```
**EN:** Minimal adapter that lets the memory pool notify LMCache per-layer. Declares the `LayerTransferCounter` class.
**CN:** 声明 `LayerTransferCounter` 类。

### Lines 45-54: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        num_layers: int,
        load_stream: torch.cuda.Stream,
        lmc_connector: LMCacheLayerwiseConnector,
        printable: bool = False,
    ):
        self.num_layers = num_layers
        self.load_stream = load_stream
        self.lmc_connector = lmc_connector
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `LayerTransferCounter`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `LayerTransferCounter`。

### Lines 56-60: wait_until implementation / wait_until 实现
```python
    def wait_until(self, layer_id: int):
        # Ensure ordering of the async loads wrt compute stream(s).
        self.load_stream.synchronize()
        with self.load_stream:
            self.lmc_connector.load_kv_layerwise(layer_id)
```
**EN:** Implements the wait until routine for this scope. It belongs to `LayerTransferCounter`.
**CN:** 实现当前作用域中的wait until例程。 该方法属于 `LayerTransferCounter`。

### Lines 63-74: LMCRadixCache declaration / LMCRadixCache 声明
```python
class LMCRadixCache(RadixCache):
    """RadixCache + LMCache IO.

    This subclass adds:
      - LMCache connector setup (device/host buffers, TP rank/size)
      - Two CUDA streams for async load/store
      - Layer-wise transfer executor wiring to the KV cache
      - Overridden `match_prefix` to fetch missing prefix chunks from LMCache
      - Extended cache_finalization paths to store back into LMCache
      - Eviction barrier that respects any in-flight host->device stores
    """

```
**EN:** RadixCache + LMCache IO. Declares the `LMCRadixCache` class and connects it to `RadixCache`.
**CN:** 声明 `LMCRadixCache` 类，并将其关联到 `RadixCache`。

### Lines 75-119: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        params: CacheInitParams,
        model_config: Optional["ModelConfig"] = None,
        tp_size: int = 1,
        rank: int = 0,
        tp_group: Optional[torch.distributed.ProcessGroup] = None,
    ):
        super().__init__(params)

        kvcache = self.token_to_kv_pool_allocator.get_kvcache()
        self.lmcache_connector = LMCacheLayerwiseConnector(
# ... omitted for brevity ...
        kvcache.register_layer_transfer_counter(self.layer_done_executor)

        self._in_flight_nodes: list[TreeNode] = []
        self._node_lock = threading.Lock()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `LMCRadixCache`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `LMCRadixCache`。

### Lines 121-125: reset implementation / reset 实现
```python
    def reset(self):  # type: ignore[override]
        super().reset()
        if hasattr(self, "_in_flight_nodes"):
            with self._node_lock:
                self._in_flight_nodes.clear()
```
**EN:** Resets the component back to its starting state. It belongs to `LMCRadixCache`.
**CN:** 将组件恢复到初始状态。 该方法属于 `LMCRadixCache`。

### Lines 127-213: match_prefix implementation / match_prefix 实现
```python
    def match_prefix(self, params: MatchPrefixParams) -> MatchResult:  # type: ignore[override]
        """Match cached prefix; if there's a tail miss, prefetch from LMCache.

        Reuses the base matching logic to obtain (value, last_node). If there
        remains a *page-aligned* uncached suffix and there is room (or after
        eviction), we allocate token slots and trigger an async LMCache load
        into those slots, then materialize a new child node for the retrieved
        chunk.
        """
        key = params.key
        if self.disable or not key:
            return super().match_prefix(params)
# ... omitted for brevity ...
                best_match_node=last_node,
            )

        return base_res
```
**EN:** Match cached prefix; if there's a tail miss, prefetch from LMCache. Matches input data against cached state and returns the best fit. It belongs to `LMCRadixCache`. It returns a computed result to its caller.
**CN:** 将输入与缓存状态匹配并返回最佳结果。 该方法属于 `LMCRadixCache`。它会向调用方返回计算结果。

### Lines 215-255: cache_finished_req implementation / cache_finished_req 实现
```python
    def cache_finished_req(self, req: Req, is_insert: bool = True) -> None:  # type: ignore[override]
        """On request completion, insert device KV into radix and store to LMCache."""

        super().cache_finished_req(req, is_insert=is_insert)
        if not is_insert:
            return

        from sglang.srt.server_args import get_global_server_args

        global_server_args = get_global_server_args()
        topk = global_server_args.speculative_eagle_topk
        enable_kv_committed_len = topk is None or topk == 1
# ... omitted for brevity ...
        with torch.cuda.stream(self.store_stream):
            self.lmcache_connector.store_kv(store_md)
        with self._node_lock:
            self._in_flight_nodes.append(new_last_node)
```
**EN:** On request completion, insert device KV into radix and store to LMCache. Implements the cache finished REQ routine for this scope. It belongs to `LMCRadixCache`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的缓存finished req例程。 该方法属于 `LMCRadixCache`。它会向调用方返回计算结果。

### Lines 257-268: evict implementation / evict 实现
```python
    def evict(self, params: EvictParams) -> EvictResult:
        """Before base eviction, wait for any outstanding stores and release locks."""
        if self.disable:
            return EvictResult()

        self.store_stream.synchronize()
        with self._node_lock:
            for node in self._in_flight_nodes:
                self.dec_lock_ref(node)
            self._in_flight_nodes.clear()

        return super().evict(params)
```
**EN:** Before base eviction, wait for any outstanding stores and release locks. Removes cache entries according to the active policy. It belongs to `LMCRadixCache`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。 该方法属于 `LMCRadixCache`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 270-277: pretty_print implementation / pretty_print 实现
```python
    def pretty_print(self):  # type: ignore[override]
        super().pretty_print()
        try:
            logger.debug(
                "evictable=%d protected=%d", self.evictable_size_, self.protected_size_
            )
        except Exception:  # pragma: no cover
            pass
```
**EN:** Implements the pretty print routine for this scope. It belongs to `LMCRadixCache`.
**CN:** 实现当前作用域中的pretty print例程。 该方法属于 `LMCRadixCache`。

## Key Concepts / 关键概念
- **`LayerTransferCounter`**: Defines the `LayerTransferCounter` type and its core responsibilities. / 定义 `LayerTransferCounter` 类型及其核心职责。
- **`LMCRadixCache`**: Defines the `LMCRadixCache` type and its core responsibilities. / 定义 `LMCRadixCache` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `threading`, `typing`, `torch`, `lmcache.integration.sglang.sglang_adapter`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.configs.model_config`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.server_args`
