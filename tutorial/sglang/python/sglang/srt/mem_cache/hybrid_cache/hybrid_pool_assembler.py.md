# hybrid_pool_assembler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hybrid_cache/hybrid_pool_assembler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hybrid pool assembler logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的混合池assembler相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Callable, Optional

from sglang.srt.mem_cache.hicache_storage import PoolName, SidecarPoolSpec
from sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller import (
    HybridCacheController,
)
from sglang.srt.mem_cache.memory_pool_host import (
    DeepSeekV4PagedHostPool,
    DeepSeekV4StateHostPool,
# ... omitted for brevity ...
    MLATokenToKVPoolHost,
    NSAIndexerPoolHost,
    PoolEntry,
)
```
**EN:** Imports `__future__`, `logging`, `typing`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller`, `sglang.srt.mem_cache.memory_pool_host` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `typing`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller`, `sglang.srt.mem_cache.memory_pool_host` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 22-29: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    import torch

    from sglang.srt.mem_cache.cache_init_params import CacheInitParams
    from sglang.srt.mem_cache.hi_mamba_radix_cache import HiMambaRadixCache
    from sglang.srt.mem_cache.hiradix_cache import HiRadixCache
    from sglang.srt.mem_cache.unified_radix_cache import UnifiedRadixCache
    from sglang.srt.server_args import ServerArgs
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 31-31: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 34-43: _make_layer_mapper implementation / _make_layer_mapper 实现
```python
def _make_layer_mapper(
    layer_mapping: dict[int, int],
    transfer_layer_num: int,
) -> Callable[[int], Optional[int]]:
    def mapper(layer_id: int) -> Optional[int]:
        if not 0 <= layer_id < transfer_layer_num:
            return None
        return layer_mapping.get(layer_id)

    return mapper
```
**EN:** Implements the make layer mapper routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的make layer mapper例程。它会向调用方返回计算结果。

### Lines 46-66: build_kv_host_pool implementation / build_kv_host_pool 实现
```python
def build_kv_host_pool(
    *,
    kv_pool: Any,
    page_size: int,
    server_args: ServerArgs,
    use_mla: bool,
    override_kv_cache_dim: Optional[int] = None,
):
    kv_host_pool_cls = MLATokenToKVPoolHost if use_mla else MHATokenToKVPoolHost
    kwargs = {}
    if override_kv_cache_dim is not None:
        kwargs["override_kv_cache_dim"] = override_kv_cache_dim
# ... omitted for brevity ...
        server_args.hicache_mem_layout,
        allocator_type=server_args.hicache_storage_backend,
        **kwargs,
    )
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 69-92: build_pool_entry implementation / build_pool_entry 实现
```python
def build_pool_entry(
    *,
    name: PoolName,
    host_pool: Any,
    device_pool: Any,
    layer_mapping: dict[int, int],
    transfer_layer_num: int,
    is_anchor: bool = False,
    host_evict_fn: Optional[Callable[[int], Any]] = None,
    device_evict_fn: Optional[Callable[[int], Any]] = None,
    device_alloc_fn: Optional[Callable[[int], Any]] = None,
    device_free_fn: Optional[Callable[[Any], Any]] = None,
# ... omitted for brevity ...
        device_evict_fn=device_evict_fn,
        device_alloc_fn=device_alloc_fn,
        device_free_fn=device_free_fn,
    )
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 95-154: build_kv_only_stack implementation / build_kv_only_stack 实现
```python
def build_kv_only_stack(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    kv_pool: Any,
    full_layer_mapping: dict[int, int],
    page_size: int,
    tp_group,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
    storage_backend: Optional[str],
# ... omitted for brevity ...
        transfer_layer_num=transfer_layer_num,
        enable_storage_metrics=enable_storage_metrics,
    )
    return host_pool_group, cache_controller
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 157-238: build_hybrid_swa_stack implementation / build_hybrid_swa_stack 实现
```python
def build_hybrid_swa_stack(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    full_kv_pool: Any,
    swa_kv_pool: Any,
    full_layer_mapping: dict[int, int],
    swa_layer_mapping: dict[int, int],
    page_size: int,
    tp_group,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
# ... omitted for brevity ...
        transfer_layer_num=transfer_layer_num,
        enable_storage_metrics=enable_storage_metrics,
    )
    return host_pool_group, cache_controller
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 241-263: _deepseek_v4_num_host_pages implementation / _deepseek_v4_num_host_pages 实现
```python
def _deepseek_v4_num_host_pages(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    kvcache: Any,
    page_size: int,
    swa_page_size: int,
) -> tuple[int, int]:
    allocator = params.token_to_kv_pool_allocator
    device_full_size = getattr(allocator, "size_full", kvcache.size)
    device_full_pages = (device_full_size + page_size - 1) // page_size

# ... omitted for brevity ...
    ratio = server_args.hicache_ratio
    full_host_pages = max(int(device_full_pages * ratio), device_full_pages + 1)
    swa_host_pages = max(int(device_swa_pages * ratio), device_swa_pages + 1)
    return full_host_pages, swa_host_pages
```
**EN:** Implements the deepseek V 4 NUM host pages routine for this scope. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的DeepSeek v 4 num主机侧pages例程。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 266-484: build_deepseek_v4_hicache_stack implementation / build_deepseek_v4_hicache_stack 实现
```python
def build_deepseek_v4_hicache_stack(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    kvcache: Any,
    page_size: int,
    tp_group,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
    storage_backend: Optional[str],
    host_swa_evict_fn: Optional[Callable[[int], Any]] = None,
# ... omitted for brevity ...
        transfer_layer_num=transfer_layer_num,
        enable_storage_metrics=enable_storage_metrics,
    )
    return host_pool_group, cache_controller
```
**EN:** Builds a derived structure from the available configuration or inputs. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 487-564: build_hybrid_mamba_stack implementation / build_hybrid_mamba_stack 实现
```python
def build_hybrid_mamba_stack(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    kv_pool: Any,
    mamba_pool: Any,
    full_layer_mapping: dict[int, int],
    mamba_layer_mapping: dict[int, int],
    page_size: int,
    tp_group,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
# ... omitted for brevity ...
        transfer_layer_num=transfer_layer_num,
        enable_storage_metrics=enable_storage_metrics,
    )
    return host_pool_group, cache_controller
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 567-636: build_anchor_sidecar_stack implementation / build_anchor_sidecar_stack 实现
```python
def build_anchor_sidecar_stack(
    *,
    params: CacheInitParams,
    server_args: ServerArgs,
    kv_pool: Any,
    sidecar_pool_name: PoolName,
    full_layer_mapping: dict[int, int],
    page_size: int,
    tp_group,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
# ... omitted for brevity ...
        transfer_layer_num=transfer_layer_num,
        enable_storage_metrics=enable_storage_metrics,
    )
    return host_pool_group, cache_controller
```
**EN:** Builds a derived structure from the available configuration or inputs. It returns a computed result to its caller.
**CN:** 根据现有配置或输入构建派生结构。它会向调用方返回计算结果。

### Lines 639-904: attach_hybrid_pool_to_unified_cache implementation / attach_hybrid_pool_to_unified_cache 实现
```python
def attach_hybrid_pool_to_unified_cache(
    cache: UnifiedRadixCache,
    params: CacheInitParams,
    server_args: ServerArgs,
    *,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
) -> None:
    """Attach HostPoolGroup + HybridCacheController to UnifiedRadixCache."""
    from sglang.srt.mem_cache.base_prefix_cache import EvictParams
    from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
# ... omitted for brevity ...
        )
    except Exception:
        logger.exception("attach_hybrid_pool_to_unified_cache failed")
        raise
```
**EN:** Attach HostPoolGroup + HybridCacheController to UnifiedRadixCache. Implements the attach hybrid pool TO unified cache routine for this scope. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的attach混合池to统一缓存例程。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 907-963: attach_hybrid_nsa_pool_to_hiradix_cache implementation / attach_hybrid_nsa_pool_to_hiradix_cache 实现
```python
def attach_hybrid_nsa_pool_to_hiradix_cache(
    radix_cache: HiRadixCache,
    params: CacheInitParams,
    server_args: ServerArgs,
    *,
    extra_config: dict,
    prefetch_threshold: int,
    enable_storage_metrics: bool,
    load_cache_event,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
) -> None:
# ... omitted for brevity ...
        )
    except Exception:
        logger.exception("attach_hybrid_nsa_pool_to_hiradix_cache failed")
        raise
```
**EN:** Attach HostPoolGroup (KV + indexer) + HybridCacheController for HiRadixCache. Implements the attach hybrid NSA pool TO hiradix cache routine for this scope. It validates error cases explicitly.
**CN:** 实现当前作用域中的attach混合nsa池to hiradix缓存例程。它会显式处理错误场景。

### Lines 966-1026: attach_hybrid_pool_to_mamba_cache implementation / attach_hybrid_pool_to_mamba_cache 实现
```python
def attach_hybrid_pool_to_mamba_cache(
    mamba_cache: HiMambaRadixCache,
    params: CacheInitParams,
    server_args: ServerArgs,
    *,
    extra_config: dict,
    prefetch_threshold: int,
    load_cache_event,
    enable_storage_metrics: bool = False,
    attn_cp_group: Optional[torch.distributed.ProcessGroup] = None,
    attn_tp_group: Optional[torch.distributed.ProcessGroup] = None,
) -> None:
# ... omitted for brevity ...
        )
    except Exception:
        logger.exception("attach_hybrid_pool_to_mamba_cache failed")
        raise
```
**EN:** Attach HostPoolGroup (KV + Mamba) + HybridCacheController for HiMambaRadixCache. Implements the attach hybrid pool TO mamba cache routine for this scope. It validates error cases explicitly.
**CN:** 实现当前作用域中的attach混合池to Mamba缓存例程。它会显式处理错误场景。

## Key Concepts / 关键概念
- **`_make_layer_mapper`**: Provides the `_make_layer_mapper` entry point for module-level behavior. / 提供模块级行为的 `_make_layer_mapper` 入口。
- **`build_kv_host_pool`**: Provides the `build_kv_host_pool` entry point for module-level behavior. / 提供模块级行为的 `build_kv_host_pool` 入口。
- **`build_pool_entry`**: Provides the `build_pool_entry` entry point for module-level behavior. / 提供模块级行为的 `build_pool_entry` 入口。
- **`build_kv_only_stack`**: Provides the `build_kv_only_stack` entry point for module-level behavior. / 提供模块级行为的 `build_kv_only_stack` 入口。
- **`build_hybrid_swa_stack`**: Provides the `build_hybrid_swa_stack` entry point for module-level behavior. / 提供模块级行为的 `build_hybrid_swa_stack` 入口。
- **`_deepseek_v4_num_host_pages`**: Provides the `_deepseek_v4_num_host_pages` entry point for module-level behavior. / 提供模块级行为的 `_deepseek_v4_num_host_pages` 入口。
- **`build_deepseek_v4_hicache_stack`**: Provides the `build_deepseek_v4_hicache_stack` entry point for module-level behavior. / 提供模块级行为的 `build_deepseek_v4_hicache_stack` 入口。
- **`build_hybrid_mamba_stack`**: Provides the `build_hybrid_mamba_stack` entry point for module-level behavior. / 提供模块级行为的 `build_hybrid_mamba_stack` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.hybrid_cache.hybrid_cache_controller`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.hi_mamba_radix_cache`, `sglang.srt.mem_cache.hiradix_cache`, `sglang.srt.mem_cache.unified_radix_cache`, `sglang.srt.server_args`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.swa_memory_pool`
