# trtllm_fp8_kv_kernel.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/triton_ops/trtllm_fp8_kv_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module wraps low-level trtllm fp8 kv kernel compute routines used by the attention stack. It focuses on tensor layout, launch preparation, and accelerated execution details. / 该模块封装注意力栈使用的底层 trtllm fp8 kv kernel 计算例程，重点处理张量布局、启动准备和加速执行细节。
## Line-by-Line Analysis / 逐行分析
### Lines 1-13: docstring
```python
"""
Fused FP8 quantization + paged KV cache write kernel for TRTLLM MHA backend.

This kernel fuses the following operations:
1. FP8 quantization of K and V tensors (from BF16/FP16 to FP8)
2. Per-token or per-page scale computation
3. Writing quantized K/V to paged KV cache layout

Performance benefits:
- Eliminates intermediate FP8 tensors in memory
- Reduces kernel launch overhead
- Better memory bandwidth utilization
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 15-20: imports
```python
import logging
from typing import Optional

import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 22-22: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 25-84: function _process_kv_tensor
```python
@triton.jit
def _process_kv_tensor(
    token_id,
    head_block_id,
    page_id,
    page_offset,
    input_ptr,
    cache_ptr,
    inv_scale,
    use_provided_scale: tl.constexpr,
    num_kv_heads: tl.constexpr,
    head_dim: tl.constexpr,
    input_stride_token: tl.constexpr,
    input_stride_head: tl.constexpr,
    input_stride_dim: tl.constexpr,
    cache_stride_page: tl.constexpr,
    cache_stride_offset: tl.constexpr,
    cache_stride_head: tl.constexpr,
    cache_stride_dim: tl.constexpr,
    BLOCK_HEAD: tl.constexpr,
    BLOCK_DIM: tl.constexpr,
):
    """Process a block of heads for a single K or V tensor."""
    head_idx = head_block_id * BLOCK_HEAD
    num_heads_in_block = min(BLOCK_HEAD, num_kv_heads - head_idx)

    for dim_idx in range(0, head_dim, BLOCK_DIM):
        num_dims_in_block = min(BLOCK_DIM, head_dim - dim_idx)

        head_offsets = head_idx + tl.arange(0, BLOCK_HEAD)
        dim_offsets = dim_idx + tl.arange(0, BLOCK_DIM)

# ... omitted 16 lines ...
        else:
            block_fp8 = block.to(tl.float8e4nv)

        # Write to cache at [page_id, page_offset, head, dim]
        cache_offsets = (
            page_id * cache_stride_page
            + page_offset * cache_stride_offset
            + head_offsets[:, None] * cache_stride_head
            + dim_offsets[None, :] * cache_stride_dim
        )

        tl.store(cache_ptr + cache_offsets, block_fp8, mask=mask)
```
**EN:** Implements the process kv tensor routine used by this attention module.
**CN:** 实现该注意力模块使用的 process kv tensor 例程。

### Lines 87-201: function _fused_fp8_set_kv_buffer_kernel
```python
@triton.jit
def _fused_fp8_set_kv_buffer_kernel(
    # Input tensors (post-RoPE K and V in FP16/BF16)
    k_ptr,  # [num_tokens, num_kv_heads, head_dim]
    v_ptr,  # [num_tokens, num_kv_heads, head_dim]
    # Output KV cache buffers (FP8 paged layout)
    k_cache_ptr,  # [total_slots, num_kv_heads, head_dim]
    v_cache_ptr,  # [total_slots, num_kv_heads, head_dim]
    # Cache location indices
    cache_loc_ptr,  # [num_tokens] -> token to cache location mapping
    # Pointers to scalar inverse scales (computed on GPU in wrapper)
    inv_k_scale_ptr,  # pointer to 0-D tensor on GPU
    inv_v_scale_ptr,  # pointer to 0-D tensor on GPU
    use_provided_scale: tl.constexpr,  # whether to use provided scale
    # Tensor dimensions
    num_kv_heads: tl.constexpr,
    head_dim: tl.constexpr,
    page_size: tl.constexpr,
    # Strides for K input [num_tokens, num_kv_heads, head_dim]
    k_stride_token: tl.constexpr,
    k_stride_head: tl.constexpr,
    k_stride_dim: tl.constexpr,
    # Strides for K cache [total_slots, num_kv_heads, head_dim] (logically paged)
    k_cache_stride_page: tl.constexpr,
    k_cache_stride_offset: tl.constexpr,
    k_cache_stride_head: tl.constexpr,
    k_cache_stride_dim: tl.constexpr,
    # Strides for V input [num_tokens, num_kv_heads, head_dim]
    v_stride_token: tl.constexpr,
    v_stride_head: tl.constexpr,
    v_stride_dim: tl.constexpr,
    # Strides for V cache [total_slots, num_kv_heads, head_dim] (logically paged)
# ... omitted 71 lines ...
            num_kv_heads,
            head_dim,
            v_stride_token,
            v_stride_head,
            v_stride_dim,
            v_cache_stride_page,
            v_cache_stride_offset,
            v_cache_stride_head,
            v_cache_stride_dim,
            BLOCK_HEAD,
            BLOCK_DIM,
        )
```
**EN:** Implements the fused fp8 set kv buffer kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused fp8 set kv buffer kernel 例程。

### Lines 204-417: function fused_fp8_set_kv_buffer
```python
def fused_fp8_set_kv_buffer(
    k: torch.Tensor,  # [num_tokens, num_kv_heads, head_dim] or [num_tokens, num_kv_heads * head_dim]
    v: torch.Tensor,  # [num_tokens, num_kv_heads, head_dim] or [num_tokens, num_kv_heads * head_dim]
    k_cache: torch.Tensor,  # [total_slots, num_kv_heads, head_dim] or [num_pages, page_size, num_kv_heads, head_dim]
    v_cache: torch.Tensor,  # [total_slots, num_kv_heads, head_dim] or [num_pages, page_size, num_kv_heads, head_dim]
    cache_loc: torch.Tensor,  # [num_tokens], dtype=int32
    k_scale: Optional[
        float
    ] = None,  # Scalar scale (matching original set_kv_buffer signature)
    v_scale: Optional[float] = None,
    page_size: int = 16,
    use_triton: bool = True,  # Whether to use Triton kernel (set to False to force naive fallback)
) -> None:
    """
    Python wrapper for the fused FP8 quantization + paged KV cache write kernel.

    This function replicates the exact behavior of the original set_kv_buffer but with
    a fused kernel that combines FP8 quantization and cache write.

    Args:
        k: Key tensor after RoPE, can be 2D or 3D
        v: Value tensor, can be 2D or 3D
        k_cache: Paged K cache buffer in FP8
        v_cache: Paged V cache buffer in FP8
        cache_loc: Cache location for each token, shape [num_tokens]
        k_scale: Optional scalar scale for K (matching original set_kv_buffer)
        v_scale: Optional scalar scale for V (matching original set_kv_buffer)
        page_size: Number of tokens per page
        use_triton: Whether to use optimized Triton kernel
    """
    num_tokens = k.shape[0]

# ... omitted 170 lines ...
            v_cache_stride_page,
            v_cache_stride_offset,
            v_cache_stride_head,
            v_cache_stride_dim,
            BLOCK_HEAD=BLOCK_HEAD,
            BLOCK_DIM=BLOCK_DIM,
        )
    else:
        # Fallback to naive implementation
        _naive_fp8_set_kv_buffer(
            k_2d, v_2d, k_cache, v_cache, cache_loc, k_scale, v_scale, page_size
        )
```
**EN:** Implements the fused fp8 set kv buffer routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused fp8 set kv buffer 例程。

### Lines 420-504: function _naive_fp8_set_kv_buffer
```python
def _naive_fp8_set_kv_buffer(
    k: torch.Tensor,
    v: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    cache_loc: torch.Tensor,
    k_scale: Optional[float],
    v_scale: Optional[float],
    page_size: int,
) -> None:
    """
    Naive fallback implementation that mimics the original set_kv_buffer logic.

    This directly replicates the behavior of MHATokenToKVPool.set_kv_buffer:
    1. Apply scale (if k.dtype != cache.dtype and scale is provided)
    2. Convert to FP8
    3. Write to cache at cache_loc

    Args:
        k: [num_tokens, num_kv_heads * head_dim], already reshaped to 2D
        v: [num_tokens, num_kv_heads * head_dim], already reshaped to 2D
        k_cache: [total_slots, num_kv_heads, head_dim] or [num_pages, page_size, num_kv_heads, head_dim]
        v_cache: Same shape as k_cache
        cache_loc: [num_tokens]
        k_scale: Optional scale for K
        v_scale: Optional scale for V
        page_size: Tokens per page
    """
    num_tokens = k.shape[0]

    # Infer dimensions from cache
    if k_cache.ndim == 3:
# ... omitted 41 lines ...
    # Write to cache using advanced indexing (same as original)
    if k_cache.ndim == 3:
        # 3D cache: [total_slots, H, D]
        k_cache[cache_loc] = k
        v_cache[cache_loc] = v
    else:
        # 4D cache: [num_pages, page_size, H, D]
        # Decompose loc into page_id and page_offset (vectorized)
        page_ids = cache_loc // page_size
        page_offsets = cache_loc % page_size
        k_cache[page_ids, page_offsets] = k
        v_cache[page_ids, page_offsets] = v
```
**EN:** Implements the naive fp8 set kv buffer routine used by this attention module.
**CN:** 实现该注意力模块使用的 naive fp8 set kv buffer 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `logging`
- `typing.Optional`
- `torch`
- `triton`
- `triton.language`
