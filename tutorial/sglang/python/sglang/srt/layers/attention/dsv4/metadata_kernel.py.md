# metadata_kernel.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/metadata_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module wraps low-level metadata kernel compute routines used by the attention stack. It focuses on tensor layout, launch preparation, and accelerated execution details. / 该模块封装注意力栈使用的底层 metadata kernel 计算例程，重点处理张量布局、启动准备和加速执行细节。
## Line-by-Line Analysis / 逐行分析
### Lines 1-5: imports
```python
from typing import Optional, Tuple

import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 8-84: function _init_compressed_attn_metadata_kernel
```python
@triton.jit
def _init_compressed_attn_metadata_kernel(
    seq_lens_ptr,
    positions_ptr,
    raw_out_loc_ptr,
    page_table_ptr,
    c4_out_loc_ptr,
    c4_positions_ptr,
    c4_seq_lens_raw_ptr,
    c4_seq_lens_clamp1_ptr,
    c128_out_loc_ptr,
    c128_positions_ptr,
    c128_seq_lens_clamp1_ptr,
    c128_page_indices_ptr,
    bs,
    max_pages,
    page_size: tl.constexpr,
    c128_max_seq_len: tl.constexpr,
    c128_page_size: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
    COMPUTE_PAGE_INDICES: tl.constexpr,
):
    batch_id = tl.program_id(0)
    if batch_id >= bs:
        return

    seq_len = tl.load(seq_lens_ptr + batch_id)
    position = tl.load(positions_ptr + batch_id)
    raw_out_loc = tl.load(raw_out_loc_ptr + batch_id)

    c4_should_compress = (seq_len % 4) == 0
    c4_out_loc = tl.where(c4_should_compress, raw_out_loc // 4, 0)
# ... omitted 33 lines ...
            )

            c_page_indices_vals = page_table_vals * c128_page_size + offset_in_page

            valid_mask = offsets < c128_seq_lens_raw
            c_page_indices_vals = tl.where(valid_mask, c_page_indices_vals, -1)

            tl.store(
                c128_page_indices_ptr + page_indices_base + offsets,
                c_page_indices_vals,
                mask=mask,
            )
```
**EN:** Implements the init compressed attn metadata kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 init compressed attn metadata kernel 例程。

### Lines 87-173: function _init_compressed_attn_metadata_triton
```python
def _init_compressed_attn_metadata_triton(
    seq_lens: torch.Tensor,
    positions: torch.Tensor,
    raw_out_loc: torch.Tensor,
    page_table: Optional[torch.Tensor] = None,
    page_size: int = 0,
    compute_page_indices: bool = True,
) -> Tuple[
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    Optional[torch.Tensor],
]:
    bs = seq_lens.shape[0]
    device = seq_lens.device

    c4_out_loc = torch.empty(bs, dtype=torch.int32, device=device)
    c4_positions = torch.empty(bs, dtype=torch.int32, device=device)
    c4_seq_lens_raw = torch.empty(bs, dtype=torch.int32, device=device)
    c4_seq_lens_clamp1 = torch.empty(bs, dtype=torch.int32, device=device)

    c128_out_loc = torch.empty(bs, dtype=torch.int32, device=device)
    c128_positions = torch.empty(bs, dtype=torch.int32, device=device)
    c128_seq_lens_clamp1 = torch.empty(bs, dtype=torch.int32, device=device)

    if compute_page_indices:
        assert (
            page_table is not None
# ... omitted 43 lines ...
    )

    return (
        c4_out_loc,
        c4_positions,
        c4_seq_lens_raw,
        c4_seq_lens_clamp1,
        c128_out_loc,
        c128_positions,
        c128_seq_lens_clamp1,
        c128_page_indices,
    )
```
**EN:** Implements the init compressed attn metadata triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 init compressed attn metadata triton 例程。

### Lines 176-200: function init_compression_metadata
```python
def init_compression_metadata(
    seq_lens: torch.Tensor,
    positions: torch.Tensor,
    raw_out_loc: torch.Tensor,
    page_table: Optional[torch.Tensor] = None,
    page_size: int = 0,
    compute_page_indices: bool = True,
) -> Tuple[
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
    Optional[torch.Tensor],
]:
    return _init_compressed_attn_metadata_triton(
        seq_lens,
        positions,
        raw_out_loc,
        page_table,
        page_size,
        compute_page_indices,
    )
```
**EN:** Prepares init compression metadata so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init compression metadata，使后续内核能够使用正确的元数据、布局和缓存状态执行。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `triton`
- `triton.language`
