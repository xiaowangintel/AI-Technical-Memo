# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Common utilities. / 该模块实现与工具相关的核心逻辑，并服务于 SGLang 的内存缓存子系统。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: License header and introductory comments / 许可证头与说明注释
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Common utilities."""

```
**EN:** Preserves licensing information and introductory commentary for the module.
**CN:** 保留模块的许可证信息以及开场说明文字。

### Lines 16-24: Imports and setup / 导入与初始化
```python
import hashlib
from typing import Any, List, Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.jit_kernel.utils import is_arch_support_pdl
from sglang.srt.environ import envs
```
**EN:** Imports `hashlib`, `typing`, `torch`, `triton`, `triton.language`, `sglang.jit_kernel.utils` and other helpers used by the surrounding scope.
**CN:** 导入 `hashlib`, `typing`, `torch`, `triton`, `triton.language`, `sglang.jit_kernel.utils` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 27-91: set_mla_kv_buffer_kernel implementation / set_mla_kv_buffer_kernel 实现
```python
@triton.jit
def set_mla_kv_buffer_kernel(
    kv_buffer_ptr,
    cache_k_nope_ptr,
    cache_k_rope_ptr,
    loc_ptr,
    buffer_stride: tl.constexpr,
    nope_stride: tl.constexpr,
    rope_stride: tl.constexpr,
    nope_dim: tl.constexpr,
    rope_dim: tl.constexpr,
    BLOCK: tl.constexpr,
# ... omitted for brevity ...
    tl.store(dst_ptr, src, mask=mask)

    if USE_GDC:
        tl.extra.cuda.gdc_launch_dependents()
```
**EN:** Updates internal state with the provided value.
**CN:** 使用给定值更新内部状态。

### Lines 92-97: Comment block / 注释块
```python


# Above this loc count the TMA bulk-store path overtakes the single-CTA-per-loc
# Triton kernel. Below it, Triton with BLOCK = next_pow2(total_dim) (one CTA
# does the whole row in one tile, no boundary fan-out) is the winning fallback.
# Tuned on GB300 with DSv4 row widths.
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 98-98: Shared state definitions / 共享状态定义
```python
_TMA_BULK_STORE_MIN_LOCS = 768
```
**EN:** Defines module-level variables such as `_TMA_BULK_STORE_MIN_LOCS`.
**CN:** 定义模块级变量，例如 `_TMA_BULK_STORE_MIN_LOCS`。

### Lines 101-171: set_mla_kv_buffer_triton implementation / set_mla_kv_buffer_triton 实现
```python
def set_mla_kv_buffer_triton(
    kv_buffer: torch.Tensor,
    loc: torch.Tensor,
    cache_k_nope: torch.Tensor,
    cache_k_rope: torch.Tensor,
):
    """Dispatch MLA paged-KV scatter writes to the fastest available path.

    Two paths, chosen on ``n_loc``:

    - ``n_loc >= 768`` (and SM90+ with TMA-compatible row widths): JIT CUDA
      kernel where each warp loads one (nope, rope) row into shared memory and
# ... omitted for brevity ...
        rope_dim,
        BLOCK=BLOCK,
        **pdl_kwargs,
    )
```
**EN:** Dispatch MLA paged-KV scatter writes to the fastest available path. Updates internal state with the provided value. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。它会向调用方返回计算结果。

### Lines 174-234: set_mla_kv_buffer_fp8_quant_kernel implementation / set_mla_kv_buffer_fp8_quant_kernel 实现
```python
@triton.jit
def set_mla_kv_buffer_fp8_quant_kernel(
    kv_buffer_fp8_ptr,
    cache_k_nope_ptr,
    cache_k_rope_ptr,
    loc_ptr,
    buffer_stride: tl.constexpr,
    nope_stride: tl.constexpr,
    rope_stride: tl.constexpr,
    nope_dim: tl.constexpr,
    rope_dim: tl.constexpr,
    BLOCK: tl.constexpr,
# ... omitted for brevity ...
    tl.store(dst_ptr, src, mask=mask)

    if USE_GDC:
        tl.extra.cuda.gdc_launch_dependents()
```
**EN:** Fuse BF16/FP16->FP8 cast with paged KV write. Updates internal state with the provided value.
**CN:** 使用给定值更新内部状态。

### Lines 237-268: set_mla_kv_buffer_triton_fp8_quant implementation / set_mla_kv_buffer_triton_fp8_quant 实现
```python
def set_mla_kv_buffer_triton_fp8_quant(
    kv_buffer: torch.Tensor,
    loc: torch.Tensor,
    cache_k_nope: torch.Tensor,
    cache_k_rope: torch.Tensor,
    fp8_dtype: torch.dtype,
):
    """Fuse BF16/FP16 MLA K quantization with paged KV write."""
    kv_buffer_fp8 = kv_buffer.view(fp8_dtype)

    nope_dim = cache_k_nope.shape[-1]
    rope_dim = cache_k_rope.shape[-1]
# ... omitted for brevity ...
        rope_dim,
        BLOCK=BLOCK,
        **pdl_kwargs,
    )
```
**EN:** Fuse BF16/FP16 MLA K quantization with paged KV write. Updates internal state with the provided value.
**CN:** 使用给定值更新内部状态。

### Lines 271-308: set_mla_kv_scale_buffer_kernel implementation / set_mla_kv_scale_buffer_kernel 实现
```python
@triton.jit
def set_mla_kv_scale_buffer_kernel(
    kv_buffer_ptr,
    cache_k_nope_ptr,
    cache_k_rope_ptr,
    loc_ptr,
    buffer_stride: tl.constexpr,
    nope_stride: tl.constexpr,
    rope_stride: tl.constexpr,
    nope_dim: tl.constexpr,
    rope_dim: tl.constexpr,
    BLOCK: tl.constexpr,
# ... omitted for brevity ...

    # Combine nope + rope
    src = src_nope + src_rope
    tl.store(dst_ptr, src, mask=mask)
```
**EN:** Updates internal state with the provided value.
**CN:** 使用给定值更新内部状态。

### Lines 311-335: set_mla_kv_scale_buffer_triton implementation / set_mla_kv_scale_buffer_triton 实现
```python
def set_mla_kv_scale_buffer_triton(
    kv_buffer: torch.Tensor,
    loc: torch.Tensor,
    cache_k_nope: torch.Tensor,
    cache_k_rope: torch.Tensor,
):
    nope_dim = cache_k_nope.shape[-1]
    rope_dim = cache_k_rope.shape[-1]
    total_dim = nope_dim + rope_dim
    BLOCK = 128  # Keep origin, works for smaller total_dim as well.
    n_loc = loc.numel()
    grid = (n_loc, triton.cdiv(total_dim, BLOCK))
# ... omitted for brevity ...
        nope_dim,
        rope_dim,
        BLOCK=BLOCK,
    )
```
**EN:** Updates internal state with the provided value.
**CN:** 使用给定值更新内部状态。

### Lines 338-369: get_mla_kv_buffer_kernel implementation / get_mla_kv_buffer_kernel 实现
```python
@triton.jit
def get_mla_kv_buffer_kernel(
    kv_buffer_ptr,
    cache_k_nope_ptr,
    cache_k_rope_ptr,
    loc_ptr,
    buffer_stride: tl.constexpr,
    nope_stride: tl.constexpr,
    rope_stride: tl.constexpr,
    nope_dim: tl.constexpr,
    rope_dim: tl.constexpr,
):
# ... omitted for brevity ...
    tl.store(
        cache_k_rope_ptr + pid_loc * rope_stride + rope_offs,
        rope_src,
    )
```
**EN:** Retrieves the requested data or state from the current object.
**CN:** 获取当前对象中的目标数据或状态。

### Lines 372-394: get_mla_kv_buffer_triton implementation / get_mla_kv_buffer_triton 实现
```python
def get_mla_kv_buffer_triton(
    kv_buffer: torch.Tensor,
    loc: torch.Tensor,
    cache_k_nope: torch.Tensor,
    cache_k_rope: torch.Tensor,
):
    # The source data type will be implicitly converted to the target data type.
    nope_dim = cache_k_nope.shape[-1]  # 512
    rope_dim = cache_k_rope.shape[-1]  # 64
    n_loc = loc.numel()
    grid = (n_loc,)

# ... omitted for brevity ...
        cache_k_rope.stride(0),
        nope_dim,
        rope_dim,
    )
```
**EN:** Retrieves the requested data or state from the current object.
**CN:** 获取当前对象中的目标数据或状态。

### Lines 397-423: maybe_init_custom_mem_pool implementation / maybe_init_custom_mem_pool 实现
```python
def maybe_init_custom_mem_pool(
    device: str,
) -> Tuple[bool, Optional[Any], Optional[str]]:
    """
    Initialize custom memory pool based on environment variable.

    This function can be modified to support more features that require a custom memory pool.

    Args:
        device: The device to allocate memory on

    Returns:
# ... omitted for brevity ...

        return init_mooncake_custom_mem_pool(device)
    else:
        return False, None, None
```
**EN:** Initialize custom memory pool based on environment variable. Implements the maybe init custom MEM pool routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的maybe初始化custom mem池例程。它会向调用方返回计算结果。

### Lines 426-441: get_hash_str implementation / get_hash_str 实现
```python
def get_hash_str(token_ids: List[int], prior_hash: Optional[str] = None) -> str:
    hasher = hashlib.sha256()

    if prior_hash:
        hasher.update(bytes.fromhex(prior_hash))

    for t in token_ids:
        if isinstance(t, tuple):
            # EAGLE bigram mode: hash both elements to uniquely identify the bigram
            for elem in t:
                hasher.update(elem.to_bytes(4, byteorder="little", signed=False))
        else:
            # Regular mode: single integer token
            hasher.update(t.to_bytes(4, byteorder="little", signed=False))

    return hasher.hexdigest()
```
**EN:** Retrieves the requested data or state from the current object. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 444-452: hash_str_to_int64 implementation / hash_str_to_int64 实现
```python
def hash_str_to_int64(hash_str: str) -> int:
    """Convert SHA256 hex string to signed 64-bit integer for events.

    Takes first 16 hex characters (64 bits) and converts to signed int64 range.
    """
    uint64_val = int(hash_str[:16], 16)
    if uint64_val >= 2**63:
        return uint64_val - 2**64
    return uint64_val
```
**EN:** Convert SHA256 hex string to signed 64-bit integer for events. Implements the hash STR TO INT 64 routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的hash str to int 64例程。它会向调用方返回计算结果。

### Lines 455-472: compute_node_hash_values implementation / compute_node_hash_values 实现
```python
def compute_node_hash_values(node: Any, page_size: int) -> List[str]:
    """Compute SHA256-based hash values for position-aware KV block IDs."""
    hash_values = []

    parent_hash = None
    if node.parent is not None and node.parent.hash_value is not None:
        if len(node.parent.key) > 0 and len(node.parent.hash_value) > 0:
            parent_hash = node.parent.hash_value[-1]

    logical_len = len(node.key)
    for start in range(0, logical_len, page_size):
        end = min(start + page_size, logical_len)
        if end <= start:
            continue
        hash_val = node.key.hash_page(start, end, parent_hash)
        hash_values.append(hash_val)
        parent_hash = hash_val
    return hash_values
```
**EN:** Compute SHA256-based hash values for position-aware KV block IDs. Implements the compute node hash values routine for this scope. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的compute node hash values例程。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 475-499: split_node_hash_value implementation / split_node_hash_value 实现
```python
def split_node_hash_value(
    child_hash_value: Optional[List[str]], split_len: int, page_size: int
) -> tuple[Optional[List[str]], Optional[List[str]]]:
    """Split hash_value between parent and child nodes during node splitting.

    Args:
        child_hash_value: The hash_value list from the child node being split
        split_len: The length at which to split (in tokens)
        page_size: The page size for calculating number of pages

    Returns:
        Tuple of (new_node_hash_value, updated_child_hash_value)
# ... omitted for brevity ...
    new_node_hash = child_hash_value[:split_pages]
    child_hash = child_hash_value[split_pages:]

    return new_node_hash, child_hash
```
**EN:** Split hash_value between parent and child nodes during node splitting. Implements the split node hash value routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的split node hash value例程。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`set_mla_kv_buffer_kernel`**: Provides the `set_mla_kv_buffer_kernel` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_buffer_kernel` 入口。
- **`set_mla_kv_buffer_triton`**: Provides the `set_mla_kv_buffer_triton` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_buffer_triton` 入口。
- **`set_mla_kv_buffer_fp8_quant_kernel`**: Provides the `set_mla_kv_buffer_fp8_quant_kernel` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_buffer_fp8_quant_kernel` 入口。
- **`set_mla_kv_buffer_triton_fp8_quant`**: Provides the `set_mla_kv_buffer_triton_fp8_quant` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_buffer_triton_fp8_quant` 入口。
- **`set_mla_kv_scale_buffer_kernel`**: Provides the `set_mla_kv_scale_buffer_kernel` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_scale_buffer_kernel` 入口。
- **`set_mla_kv_scale_buffer_triton`**: Provides the `set_mla_kv_scale_buffer_triton` entry point for module-level behavior. / 提供模块级行为的 `set_mla_kv_scale_buffer_triton` 入口。
- **`get_mla_kv_buffer_kernel`**: Provides the `get_mla_kv_buffer_kernel` entry point for module-level behavior. / 提供模块级行为的 `get_mla_kv_buffer_kernel` 入口。
- **`get_mla_kv_buffer_triton`**: Provides the `get_mla_kv_buffer_triton` entry point for module-level behavior. / 提供模块级行为的 `get_mla_kv_buffer_triton` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `hashlib`, `typing`, `torch`, `triton`, `triton.language`
- **Internal / 内部**: `sglang.jit_kernel.utils`, `sglang.srt.environ`, `sglang.jit_kernel.set_mla_kv_buffer`, `sglang.srt.disaggregation.mooncake.utils`
