# aiter_unified_attention.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/triton_ops/aiter_unified_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages aiter unified attention logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 aiter unified attention 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-2: imports
```python
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 5-45: function scatter_ragged_to_page_table_kernel
```python
@triton.jit
def scatter_ragged_to_page_table_kernel(
    kv_flat_ptr,
    kv_indptr_ptr,
    dest_ptr,
    dest_stride,
    sw_page_table_ptr,
    swa_slot_mapping_ptr,
    PAGE_SIZE: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    HAS_SWA: tl.constexpr,
):
    """Scatter ragged token-level kv_indices into a 2D block-level page table."""
    pid = tl.program_id(0)
    block_id = tl.program_id(1)

    start = tl.load(kv_indptr_ptr + pid).to(tl.int64)
    kv_len = tl.load(kv_indptr_ptr + pid + 1).to(tl.int64) - start
    num_blocks = (kv_len + PAGE_SIZE - 1) // PAGE_SIZE

    offsets = block_id * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    if block_id * BLOCK_SIZE >= num_blocks:
        return
    mask = offsets < num_blocks
    token_idx = offsets.to(tl.int64) * PAGE_SIZE
    vals = tl.load(kv_flat_ptr + start + token_idx, mask=mask, other=0)
    block_vals = vals // PAGE_SIZE
    tl.store(
        dest_ptr + pid.to(tl.int64) * dest_stride + offsets,
        block_vals,
        mask=mask,
    )

    if HAS_SWA:
        sw_vals = tl.load(swa_slot_mapping_ptr + vals)
        block_vals = sw_vals // PAGE_SIZE
        tl.store(
            sw_page_table_ptr + pid.to(tl.int64) * dest_stride + offsets,
            block_vals,
            mask=mask,
        )
```
**EN:** Implements the scatter ragged to page table kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 scatter ragged to page table kernel 例程。

### Lines 48-97: function scatter_req_to_token_to_page_table_kernel
```python
@triton.jit
def scatter_req_to_token_to_page_table_kernel(
    req_to_token_ptr,
    req_pool_indices_ptr,
    seq_lens_ptr,
    page_table_ptr,
    req_to_token_stride,
    page_table_stride,
    sw_page_table_ptr,
    swa_slot_mapping_ptr,
    DRAFT_NUM: tl.constexpr,
    PAGE_SIZE: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    HAS_SWA: tl.constexpr,
):
    """Build the 2D block-level page_table for target_verify from req_to_token."""
    pid = tl.program_id(0)
    block_id = tl.program_id(1)

    seq_len = tl.load(seq_lens_ptr + pid).to(tl.int64)
    kv_len = seq_len + DRAFT_NUM
    num_blocks = (kv_len + PAGE_SIZE - 1) // PAGE_SIZE

    offsets = block_id * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    if block_id * BLOCK_SIZE >= num_blocks:
        return
    mask = offsets < num_blocks

    rp = tl.load(req_pool_indices_ptr + pid).to(tl.int64)
    token_idx = offsets.to(tl.int64) * PAGE_SIZE
    vals = tl.load(
        req_to_token_ptr + rp * req_to_token_stride + token_idx,
# ... omitted 6 lines ...
        block_vals,
        mask=mask,
    )

    if HAS_SWA:
        sw_vals = tl.load(swa_slot_mapping_ptr + vals)
        block_vals = sw_vals // PAGE_SIZE
        tl.store(
            sw_page_table_ptr + pid.to(tl.int64) * page_table_stride + offsets,
            block_vals,
            mask=mask,
        )
```
**EN:** Implements the scatter req to token to page table kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 scatter req to token to page table kernel 例程。

## Key Concepts / 关键概念
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `triton`
- `triton.language`
