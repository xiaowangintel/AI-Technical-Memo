# kv_cache_builder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/kv_cache_builder.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the KV cache builder logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的KV缓存构建器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
```
**EN:** Imports `__future__`, `logging` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 5-5: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 7-8: Imports and setup / 导入与初始化
```python
from dataclasses import dataclass
from typing import Optional
```
**EN:** Imports `dataclasses`, `typing` and other helpers used by the surrounding scope.
**CN:** 导入 `dataclasses`, `typing` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 11-12: KVCacheBuildResult declaration / KVCacheBuildResult 声明
```python
@dataclass(frozen=True, slots=True, kw_only=True)
class KVCacheBuildResult:
```
**EN:** Declares the `KVCacheBuildResult` class.
**CN:** 声明 `KVCacheBuildResult` 类。

### Lines 13-21: Shared state definitions / 共享状态定义
```python
    is_hybrid_swa: bool
    is_hybrid_ssm: bool
    sliding_window_size: Optional[int]
    full_tokens_per_layer: Optional[int]
    swa_tokens_per_layer: Optional[int]
    req_to_token_pool: object
    token_to_kv_pool_allocator: object
    disable_radix_cache: bool
    tree_cache: object
```
**EN:** Defines class-level variables such as `is_hybrid_swa`, `is_hybrid_ssm`, `sliding_window_size`, `full_tokens_per_layer`, `swa_tokens_per_layer`.
**CN:** 定义类级变量，例如 `is_hybrid_swa`, `is_hybrid_ssm`, `sliding_window_size`, `full_tokens_per_layer`, `swa_tokens_per_layer`。

### Lines 24-32: Imports and setup / 导入与初始化
```python
from typing import TYPE_CHECKING

from sglang.srt.configs.model_config import ModelImpl
from sglang.srt.environ import envs
from sglang.srt.managers.mm_utils import init_mm_embedding_cache
from sglang.srt.mem_cache.cache_init_params import CacheInitParams
from sglang.srt.mem_cache.radix_cache import RadixCache
from sglang.srt.model_loader.utils import get_resolved_model_impl
from sglang.srt.session.streaming_session import StreamingSession
```
**EN:** Imports `typing`, `sglang.srt.configs.model_config`, `sglang.srt.environ`, `sglang.srt.managers.mm_utils`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.radix_cache` and other helpers used by the surrounding scope.
**CN:** 导入 `typing`, `sglang.srt.configs.model_config`, `sglang.srt.environ`, `sglang.srt.managers.mm_utils`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.radix_cache` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 34-44: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:

    from torch.distributed import ProcessGroup

    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.distributed.parallel_state import GroupCoordinator
    from sglang.srt.distributed.parallel_state_wrapper import ParallelState
    from sglang.srt.managers.tp_worker import BaseTpWorker
    from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
    from sglang.srt.server_args import ServerArgs
    from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 47-69: get_draft_kv_pool implementation / get_draft_kv_pool 实现
```python
def get_draft_kv_pool(
    *,
    draft_worker: "BaseTpWorker",
    spec_algorithm: SpeculativeAlgorithm,
    server_args: ServerArgs,
    enable_overlap: bool,
):
    """Return (draft_token_to_kv_pool, draft_model_config) for the current
    draft worker, or (None, None) when no draft KV pool is available."""
    if draft_worker is None or spec_algorithm.is_ngram():
        return None, None

# ... omitted for brevity ...
    return (
        draft_worker.model_runner.token_to_kv_pool,
        draft_worker.model_config,
    )
```
**EN:** Return (draft_token_to_kv_pool, draft_model_config) for the current draft worker, or (None, None) when no draft KV pool is available. Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 72-129: maybe_register_hicache_draft implementation / maybe_register_hicache_draft 实现
```python
def maybe_register_hicache_draft(
    *,
    tree_cache: "BasePrefixCache",
    draft_worker: "BaseTpWorker",
    spec_algorithm: SpeculativeAlgorithm,
    server_args: ServerArgs,
    enable_hierarchical_cache: bool,
    enable_overlap: bool,
    page_size: int,
) -> None:
    """Register draft KV pool with HiCacheController for piggyback L2/L3 ops."""
    if not enable_hierarchical_cache:
# ... omitted for brevity ...
        )
        return

    tree_cache.cache_controller.set_draft_kv_pool(pool, draft_host_pool)
```
**EN:** Register draft KV pool with HiCacheController for piggyback L2/L3 ops. Implements the maybe register hicache draft routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe register hicache draft例程。它会向调用方返回计算结果。

### Lines 132-318: build_kv_cache implementation / build_kv_cache 实现
```python
def build_kv_cache(
    *,
    server_args: "ServerArgs",
    model_config: "ModelConfig",
    tp_worker: "BaseTpWorker",
    page_size: int,
    spec_algorithm: "SpeculativeAlgorithm",
    attn_tp_cpu_group: "ProcessGroup",
    tp_cpu_group: "ProcessGroup",
    attn_cp_cpu_group: "ProcessGroup",
    enable_metrics: bool,
    enable_kv_cache_events: bool,
# ... omitted for brevity ...
        token_to_kv_pool_allocator=token_to_kv_pool_allocator,
        disable_radix_cache=disable_radix_cache,
        tree_cache=tree_cache,
    )
```
**EN:** Builds a derived structure from the available configuration or inputs. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会显式处理错误场景。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`KVCacheBuildResult`**: Defines the `KVCacheBuildResult` type and its core responsibilities. / 定义 `KVCacheBuildResult` 类型及其核心职责。
- **`get_draft_kv_pool`**: Provides the `get_draft_kv_pool` entry point for module-level behavior. / 提供模块级行为的 `get_draft_kv_pool` 入口。
- **`maybe_register_hicache_draft`**: Provides the `maybe_register_hicache_draft` entry point for module-level behavior. / 提供模块级行为的 `maybe_register_hicache_draft` 入口。
- **`build_kv_cache`**: Provides the `build_kv_cache` entry point for module-level behavior. / 提供模块级行为的 `build_kv_cache` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `dataclasses`, `typing`, `torch.distributed`
- **Internal / 内部**: `sglang.srt.configs.model_config`, `sglang.srt.environ`, `sglang.srt.managers.mm_utils`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.model_loader.utils`, `sglang.srt.session.streaming_session`, `sglang.srt.distributed.parallel_state`, `sglang.srt.distributed.parallel_state_wrapper`, `sglang.srt.managers.tp_worker`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.server_args`
