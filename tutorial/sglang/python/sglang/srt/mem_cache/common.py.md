# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the common logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的通用相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import numpy as np
import torch
import triton
import triton.language as tl

from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache, EvictParams
from sglang.srt.mem_cache.memory_pool import HybridReqToTokenPool, ReqToTokenPool
from sglang.srt.mem_cache.swa_memory_pool import SWATokenToKVPoolAllocator
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import is_hip, support_triton
from sglang.srt.utils.common import ceil_align
```
**EN:** Imports `__future__`, `logging`, `typing`, `numpy`, `torch`, `triton` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `typing`, `numpy`, `torch`, `triton` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 18-18: Shared state definitions / 共享状态定义
```python
_is_hip = is_hip()
```
**EN:** Defines module-level variables such as `_is_hip`.
**CN:** 定义模块级变量，例如 `_is_hip`。

### Lines 20-21: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 22-23: Comment block / 注释块
```python

# Needs 2 + 1 slots for mamba request with prefix cache. 2 for ping pong cache, 1 for running mamba state.
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 24-27: Shared state definitions / 共享状态定义
```python
MAMBA_STATE_PER_REQ_PREFIX_CACHE = 3
MAMBA_STATE_PER_REQ_NO_CACHE = 1

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `MAMBA_STATE_PER_REQ_PREFIX_CACHE`, `MAMBA_STATE_PER_REQ_NO_CACHE`, `logger`.
**CN:** 定义模块级变量，例如 `MAMBA_STATE_PER_REQ_PREFIX_CACHE`, `MAMBA_STATE_PER_REQ_NO_CACHE`, `logger`。

### Lines 30-35: kv_to_page_indices implementation / kv_to_page_indices 实现
```python
def kv_to_page_indices(kv_indices: np.ndarray, page_size: int):
    # The page is guaranteed to be full except the last page.
    if page_size == 1:
        return kv_indices

    return kv_indices[::page_size] // page_size
```
**EN:** Implements the KV TO page indices routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的KV to页indices例程。它会向调用方返回计算结果。

### Lines 38-39: kv_to_page_num implementation / kv_to_page_num 实现
```python
def kv_to_page_num(num_kv_indices: int, page_size: int):
    return (num_kv_indices + page_size - 1) // page_size
```
**EN:** Implements the KV TO page NUM routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的KV to页num例程。它会向调用方返回计算结果。

### Lines 42-43: page_align_floor implementation / page_align_floor 实现
```python
def page_align_floor(length: int, page_size: int) -> int:
    return (length // page_size) * page_size
```
**EN:** Implements the page align floor routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的页align floor例程。它会向调用方返回计算结果。

### Lines 46-50: maybe_cache_unfinished_req implementation / maybe_cache_unfinished_req 实现
```python
def maybe_cache_unfinished_req(req: Req, tree_cache: BasePrefixCache, **kwargs):
    if getattr(req, "skip_radix_cache_insert", False):
        return

    tree_cache.cache_unfinished_req(req, **kwargs)
```
**EN:** Implements the maybe cache unfinished REQ routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe缓存unfinished req例程。它会向调用方返回计算结果。

### Lines 53-101: write_req_to_token_pool_triton implementation / write_req_to_token_pool_triton 实现
```python
@triton.jit
def write_req_to_token_pool_triton(
    req_to_token_ptr,  # [max_batch, max_context_len]
    req_pool_indices,
    prefix_tensors,
    pre_lens,
    seq_lens,
    extend_lens,
    out_cache_loc,
    req_to_token_ptr_stride: tl.constexpr,
):
    BLOCK_SIZE: tl.constexpr = 512
# ... omitted for brevity ...
            + pre_len,
            value,
            mask=mask,
        )
```
**EN:** Implements the write REQ TO token pool triton routine for this scope. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的write req to Token池triton例程。实现过程中会遍历输入或受管条目。

### Lines 104-150: write_cache_indices implementation / write_cache_indices 实现
```python
def write_cache_indices(
    out_cache_loc: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    req_pool_indices_cpu: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
    prefix_lens_cpu: torch.Tensor,
    seq_lens_tensor: torch.Tensor,
    seq_lens_cpu: torch.Tensor,
    extend_lens_tensor: torch.Tensor,
    extend_lens_cpu: torch.Tensor,
    prefix_tensors: list[torch.Tensor],
    req_to_token_pool: ReqToTokenPool,
# ... omitted for brevity ...
                (req_idx, slice(prefix_len, seq_len)),
                out_cache_loc[pt : pt + extend_len],
            )
            pt += extend_len
```
**EN:** Implements the write cache indices routine for this scope. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的write缓存indices例程。实现过程中会遍历输入或受管条目。

### Lines 153-180: get_last_loc implementation / get_last_loc 实现
```python
def get_last_loc(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    attn_backend = get_global_server_args().attention_backend
    uses_triton_dispatch = attn_backend not in ("ascend", "torch_native")

    if _is_hip and uses_triton_dispatch:
        # HIP-only: the legacy get_last_loc_triton kernel emits a
        # mixed-width int32->int64 store that Triton mis-compiles on HIP,
        # producing out-of-range last_loc values under EAGLE +
# ... omitted for brevity ...
    else:
        impl = get_last_loc_torch

    return impl(req_to_token, req_pool_indices_tensor, prefix_lens_tensor)
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 183-192: get_last_loc_torch implementation / get_last_loc_torch 实现
```python
def get_last_loc_torch(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    return torch.where(
        prefix_lens_tensor > 0,
        req_to_token[req_pool_indices_tensor, prefix_lens_tensor - 1],
        torch.full_like(prefix_lens_tensor, -1),
    )
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 195-224: _get_last_loc_safe_kernel implementation / _get_last_loc_safe_kernel 实现
```python
@triton.jit
def _get_last_loc_safe_kernel(
    req_to_token,
    req_pool_indices_tensor,
    prefix_lens_tensor,
    result_i32,
    num_tokens,
    req_to_token_stride,
    BLOCK_SIZE: tl.constexpr,
    PREFIX_DTYPE_IS_I64: tl.constexpr,
):
    pid = tl.program_id(0)
# ... omitted for brevity ...
    token_mask = mask & (prefix_lens > 0)
    tokens = tl.load(req_to_token + token_index, mask=token_mask, other=-1)
    # Result stays int32 (req_to_token dtype); caller promotes after return.
    tl.store(result_i32 + offset, tokens, mask=mask)
```
**EN:** Implements the get last LOC safe kernel routine for this scope.
**CN:** 实现当前作用域中的get last loc safe kernel例程。

### Lines 227-253: get_last_loc_triton_safe implementation / get_last_loc_triton_safe 实现
```python
def get_last_loc_triton_safe(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    """Fused `last_loc` Triton kernel whose in-kernel result buffer is int32
    (the dtype of req_to_token). The consumer-dtype promotion happens in
    torch after the kernel returns, so Triton never issues a mixed-width
    store — avoiding the HIP int32->int64 store bug hit by the legacy kernel.
    """
    num_tokens = prefix_lens_tensor.shape[0]
    BLOCK_SIZE = 256
# ... omitted for brevity ...
        BLOCK_SIZE=BLOCK_SIZE,
        PREFIX_DTYPE_IS_I64=(prefix_lens_tensor.dtype == torch.int64),
    )
    return result_i32.to(prefix_lens_tensor.dtype)
```
**EN:** Fused `last_loc` Triton kernel whose in-kernel result buffer is int32 (the dtype of req_to_token). Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 256-277: get_last_loc_kernel implementation / get_last_loc_kernel 实现
```python
@triton.jit
def get_last_loc_kernel(
    req_to_token,
    req_pool_indices_tensor,
    prefix_lens_tensor,
    result,
    num_tokens,
    req_to_token_stride,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)
    offset = tl.arange(0, BLOCK_SIZE) + pid * BLOCK_SIZE
# ... omitted for brevity ...
    token_index = req_pool_indices * req_to_token_stride + (prefix_lens - 1)
    tokens = tl.load(req_to_token + token_index, mask=token_mask, other=-1)

    tl.store(result + offset, tokens, mask=mask)
```
**EN:** Retrieves the requested data or state from the current object.
**CN:** 获取当前对象中的目标数据或状态。

### Lines 280-299: get_last_loc_triton implementation / get_last_loc_triton 实现
```python
def get_last_loc_triton(
    req_to_token: torch.Tensor,
    req_pool_indices_tensor: torch.Tensor,
    prefix_lens_tensor: torch.Tensor,
) -> torch.Tensor:
    BLOCK_SIZE = 256
    num_tokens = prefix_lens_tensor.shape[0]
    result = torch.empty_like(prefix_lens_tensor)
    grid = (triton.cdiv(num_tokens, BLOCK_SIZE),)

    get_last_loc_kernel[grid](
        req_to_token,
# ... omitted for brevity ...
        req_to_token.stride(0),
        BLOCK_SIZE,
    )
    return result
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 302-327: alloc_token_slots implementation / alloc_token_slots 实现
```python
def alloc_token_slots(
    tree_cache: BasePrefixCache,
    num_tokens: int,
    backup_state: bool = False,
):
    allocator = tree_cache.token_to_kv_pool_allocator
    evict_from_tree_cache(tree_cache, num_tokens)

    state = None
    if backup_state:
        state = allocator.backup_state()

# ... omitted for brevity ...
            tree_cache.pretty_print()
        raise RuntimeError(error_msg)

    return (out_cache_loc, state) if backup_state else out_cache_loc
```
**EN:** Allocates cache or memory resources for the requested workload. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 330-353: evict_from_tree_cache implementation / evict_from_tree_cache 实现
```python
def evict_from_tree_cache(tree_cache: BasePrefixCache | None, num_tokens: int):
    if tree_cache is None:
        return

    if tree_cache.is_chunk_cache():
        return

    allocator = tree_cache.token_to_kv_pool_allocator

    if isinstance(allocator, SWATokenToKVPoolAllocator):
        # Hybrid allocator
        full_available_size = allocator.full_available_size()
# ... omitted for brevity ...
    else:
        # Standard allocator
        if allocator.available_size() < num_tokens:
            tree_cache.evict(EvictParams(num_tokens=num_tokens))
```
**EN:** Removes cache entries according to the active policy. It returns a computed result to its caller.
**CN:** 按照当前策略移除缓存条目。它会向调用方返回计算结果。

### Lines 356-395: alloc_paged_token_slots_extend implementation / alloc_paged_token_slots_extend 实现
```python
def alloc_paged_token_slots_extend(
    tree_cache: BasePrefixCache,
    prefix_lens: torch.Tensor,
    prefix_lens_cpu: torch.Tensor,
    seq_lens: torch.Tensor,
    seq_lens_cpu: torch.Tensor,
    last_loc: torch.Tensor,
    extend_num_tokens: int,
    backup_state: bool = False,
):
    # Over estimate the number of tokens: assume each request needs a new page.
    allocator = tree_cache.token_to_kv_pool_allocator
# ... omitted for brevity ...
            tree_cache.pretty_print()
        raise RuntimeError(error_msg)

    return (out_cache_loc, state) if backup_state else out_cache_loc
```
**EN:** Allocates cache or memory resources for the requested workload. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 398-426: alloc_req_slots implementation / alloc_req_slots 实现
```python
def alloc_req_slots(
    req_to_token_pool: ReqToTokenPool,
    reqs: list[Req],
    tree_cache: BasePrefixCache | None,
) -> list[int]:
    """Allocate request slots from the pool."""
    num_reqs = len(reqs)
    if isinstance(req_to_token_pool, HybridReqToTokenPool):
        mamba_available_size = req_to_token_pool.mamba_pool.available_size()
        factor = (
            MAMBA_STATE_PER_REQ_PREFIX_CACHE
            if tree_cache.supports_mamba()
# ... omitted for brevity ...
            f"{req_to_token_pool.available_size()=}, "
            f"{num_reqs=}, "
        )
    return req_pool_indices
```
**EN:** Allocate request slots from the pool. Allocates cache or memory resources for the requested workload. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 429-492: alloc_for_extend implementation / alloc_for_extend 实现
```python
def alloc_for_extend(
    batch: ScheduleBatch,
) -> tuple[torch.Tensor, torch.Tensor, list[int]]:
    """
    Allocate KV cache for extend batch and write to req_to_token_pool.

    Returns:
        out_cache_loc: allocated cache locations
        req_pool_indices_device: request pool indices at a device tensor
        req_pool_indices: request pool indices as list
    """
    # free out-of-window swa tokens
# ... omitted for brevity ...
        batch.req_to_token_pool,
    )

    return out_cache_loc, req_pool_indices_device, req_pool_indices
```
**EN:** Allocate KV cache for extend batch and write to req_to_token_pool. Allocates cache or memory resources for the requested workload. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会向调用方返回计算结果。

### Lines 495-521: alloc_paged_token_slots_decode implementation / alloc_paged_token_slots_decode 实现
```python
def alloc_paged_token_slots_decode(
    tree_cache: BasePrefixCache,
    seq_lens: torch.Tensor,
    seq_lens_cpu: torch.Tensor,
    last_loc: torch.Tensor,
    token_per_req: int = 1,
) -> torch.Tensor:
    """Allocate paged KV cache for decode batch."""
    allocator = tree_cache.token_to_kv_pool_allocator
    # Over estimate the number of tokens: assume each request needs a new page.
    num_tokens = len(seq_lens) * allocator.page_size
    evict_from_tree_cache(tree_cache, num_tokens)
# ... omitted for brevity ...
            tree_cache.pretty_print()
        raise RuntimeError(error_msg)

    return out_cache_loc
```
**EN:** Allocate paged KV cache for decode batch. Allocates cache or memory resources for the requested workload. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 524-563: alloc_for_decode implementation / alloc_for_decode 实现
```python
def alloc_for_decode(batch: ScheduleBatch, token_per_req: int) -> torch.Tensor:
    """
    Allocate KV cache for decode batch and write to req_to_token_pool.

    Returns:
        out_cache_loc: allocated cache locations
    """

    batch.maybe_evict_swa()

    bs = batch.seq_lens.shape[0]

# ... omitted for brevity ...
        (batch.req_pool_indices, locs), out_cache_loc.to(torch.int32)
    )

    return out_cache_loc
```
**EN:** Allocate KV cache for decode batch and write to req_to_token_pool. Allocates cache or memory resources for the requested workload. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。它会向调用方返回计算结果。

### Lines 566-619: release_kv_cache implementation / release_kv_cache 实现
```python
def release_kv_cache(req: Req, tree_cache: BasePrefixCache, is_insert: bool = True):
    # MambaRadixCache may alloc mamba state before alloc KV cache
    if req.req_pool_idx is None:
        assert (
            tree_cache.supports_mamba()
        ), "Only MambaRadixCache allow freeing before alloc"
        # TODO (csy, hanming): clean up this early allocation logic
        if req.mamba_pool_idx is not None:
            tree_cache.req_to_token_pool.mamba_pool.free(
                req.mamba_pool_idx.unsqueeze(-1)
            )
            req.mamba_pool_idx = None
# ... omitted for brevity ...
            req.mamba_pool_idx is not None
        ), "mamba state is freed while the tree cache does not manage mamba states"
        tree_cache.req_to_token_pool.free_mamba_cache(req)
    tree_cache.req_to_token_pool.free(req)
```
**EN:** Implements the release KV cache routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的release KV缓存例程。它会向调用方返回计算结果。

### Lines 622-623: available_and_evictable_str implementation / available_and_evictable_str 实现
```python
def available_and_evictable_str(tree_cache: BasePrefixCache) -> str:
    return tree_cache.available_and_evictable_str()
```
**EN:** Implements the available AND evictable STR routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available and evictable str例程。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`kv_to_page_indices`**: Provides the `kv_to_page_indices` entry point for module-level behavior. / 提供模块级行为的 `kv_to_page_indices` 入口。
- **`kv_to_page_num`**: Provides the `kv_to_page_num` entry point for module-level behavior. / 提供模块级行为的 `kv_to_page_num` 入口。
- **`page_align_floor`**: Provides the `page_align_floor` entry point for module-level behavior. / 提供模块级行为的 `page_align_floor` 入口。
- **`maybe_cache_unfinished_req`**: Provides the `maybe_cache_unfinished_req` entry point for module-level behavior. / 提供模块级行为的 `maybe_cache_unfinished_req` 入口。
- **`write_req_to_token_pool_triton`**: Provides the `write_req_to_token_pool_triton` entry point for module-level behavior. / 提供模块级行为的 `write_req_to_token_pool_triton` 入口。
- **`write_cache_indices`**: Provides the `write_cache_indices` entry point for module-level behavior. / 提供模块级行为的 `write_cache_indices` 入口。
- **`get_last_loc`**: Provides the `get_last_loc` entry point for module-level behavior. / 提供模块级行为的 `get_last_loc` 入口。
- **`get_last_loc_torch`**: Provides the `get_last_loc_torch` entry point for module-level behavior. / 提供模块级行为的 `get_last_loc_torch` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `typing`, `numpy`, `torch`, `triton`, `triton.language`
- **Internal / 内部**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.srt.managers.schedule_batch`
