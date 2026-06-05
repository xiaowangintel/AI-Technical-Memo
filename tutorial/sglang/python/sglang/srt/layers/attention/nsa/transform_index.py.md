# transform_index.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/transform_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages transform index logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 transform index 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-5: imports
```python
from typing import List, Optional

import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 8-9: function transform_index_page_table_prefill
```python
def transform_index_page_table_prefill(**kwargs):
    return transform_index_page_table_prefill_ref(**kwargs)
```
**EN:** Implements the transform index page table prefill routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table prefill 例程。

### Lines 12-13: function transform_index_page_table_decode
```python
def transform_index_page_table_decode(**kwargs):
    return transform_index_page_table_decode_ref(**kwargs)
```
**EN:** Implements the transform index page table decode routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table decode 例程。

### Lines 16-35: function transform_index_page_table_decode_kernel
```python
@triton.jit
def transform_index_page_table_decode_kernel(
    page_table_ptr: torch.Tensor,
    topk_indices_ptr: torch.Tensor,
    result_ptr: torch.Tensor,
    page_size: tl.constexpr,
    max_seqlen_k: tl.constexpr,
):
    TOPK: tl.constexpr = 2048
    req_id = tl.program_id(0)
    page_table_ptr = page_table_ptr + req_id * max_seqlen_k
    topk_indices_ptr = topk_indices_ptr + req_id * TOPK
    result_ptr = result_ptr + req_id * TOPK

    offset = tl.arange(0, TOPK)  # topk should be 2048
    loaded_topk_indices = tl.load(topk_indices_ptr + offset)
    mask = loaded_topk_indices >= 0
    loaded_kv_indices = tl.load(page_table_ptr + loaded_topk_indices, mask=mask)
    tl.store(result_ptr + offset, loaded_kv_indices, mask=mask)
    tl.store(result_ptr + offset, -1, mask=~mask)
```
**EN:** Implements the transform index page table decode kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table decode kernel 例程。

### Lines 38-69: function transform_index_page_table_decode_fast
```python
def transform_index_page_table_decode_fast(
    page_table: torch.Tensor,
    topk_indices: torch.Tensor,
    result: Optional[torch.Tensor] = None,
    page_size: int = 1,
) -> torch.Tensor:
    """
    Transform the page table according to topk indices for sparse topk attention.
    Args:
        page_table: [qo_len, max_seqlen_k], the original page table
        topk_indices: [qo_len, topk], the topk indices for each query position
    Returns:
        transformed_page_table: [qo_len, topk], the transformed page table
        For out-of-bound indices in topk_indices, this should be filled with -1.
    """
    assert page_size == 1
    assert page_table.shape[0] == topk_indices.shape[0]
    assert topk_indices.shape[1] == 2048
    qo_len = topk_indices.shape[0]
    max_seqlen_k = page_table.shape[1]
    if result is None:
        result = torch.empty_like(topk_indices, dtype=torch.int32)
    # Launch triton kernel
    grid = (qo_len,)
    transform_index_page_table_decode_kernel[grid](
        page_table,
        topk_indices,
        result,
        page_size,
        max_seqlen_k=max_seqlen_k,
    )
    return result
```
**EN:** Implements the transform index page table decode fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table decode fast 例程。

### Lines 72-91: function transform_index_page_table_prefill_fast
```python
def transform_index_page_table_prefill_fast(
    page_table: torch.Tensor,
    topk_indices: torch.Tensor,
    extend_lens_cpu: List[int],
    page_size: int = 1,
) -> torch.Tensor:
    # TODO(baizhou): can be implemented with another triton kernel
    assert page_size == 1
    result = torch.empty_like(topk_indices, dtype=torch.int32)
    assert len(extend_lens_cpu) == page_table.shape[0]
    offset = 0
    for i, l in enumerate(extend_lens_cpu):
        transform_index_page_table_decode_fast(
            page_table[i].unsqueeze(0).expand(l, -1),
            topk_indices[offset : offset + l],
            result=result[offset : offset + l],
        )
        offset += l
    assert offset == topk_indices.shape[0]
    return result
```
**EN:** Implements the transform index page table prefill fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table prefill fast 例程。

### Lines 94-112: function transform_index_page_table_decode_ref
```python
def transform_index_page_table_decode_ref(
    page_table: torch.Tensor,
    topk_indices: torch.Tensor,
    result: Optional[torch.Tensor] = None,
    page_size: int = 1,
) -> torch.Tensor:
    assert page_size == 1
    assert page_table.shape[0] == topk_indices.shape[0]
    if result is None:
        result = torch.empty_like(topk_indices, dtype=torch.int32)
    assert result.shape == topk_indices.shape
    torch.gather(
        page_table.to(result.dtype),
        dim=1,
        index=topk_indices.clamp(min=0),
        out=result,
    )
    result[topk_indices < 0] = -1
    return result
```
**EN:** Implements the transform index page table decode ref routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table decode ref 例程。

### Lines 115-133: function transform_index_page_table_prefill_ref
```python
def transform_index_page_table_prefill_ref(
    page_table: torch.Tensor,
    topk_indices: torch.Tensor,
    extend_lens_cpu: List[int],
    page_size: int = 1,
) -> torch.Tensor:
    assert page_size == 1
    result = torch.empty_like(topk_indices, dtype=torch.int32)
    assert len(extend_lens_cpu) == page_table.shape[0]
    offset = 0
    for i, l in enumerate(extend_lens_cpu):
        transform_index_page_table_decode_ref(
            page_table[i].unsqueeze(0).expand(l, -1),
            topk_indices[offset : offset + l],
            result=result[offset : offset + l],
        )
        offset += l
    assert offset == topk_indices.shape[0]
    return result
```
**EN:** Implements the transform index page table prefill ref routine used by this attention module.
**CN:** 实现该注意力模块使用的 transform index page table prefill ref 例程。

### Lines 136-144: conditional branch
```python
if __name__ == "__main__":
    bs, topk, max_seqlen = 10, 2048, 3000
    page_table = torch.randint(0, 100, (bs, max_seqlen), device="cuda")
    topk_indices = torch.full((bs, topk), -1, device="cuda")
    topk_indices[:, :1600] = torch.arange(1600).unsqueeze(0).repeat(bs, 1)
    ref_result = transform_index_page_table_decode_ref(page_table, topk_indices)
    result = transform_index_page_table_decode_fast(page_table, topk_indices)
    assert torch.all(result == ref_result)
    print("Passed")
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Position encoding handling / **CN:** 位置编码处理
- **EN:** Compressed or sparse attention helpers / **CN:** 压缩或稀疏注意力辅助逻辑

## Dependencies / 依赖关系
- `typing.List`
- `typing.Optional`
- `torch`
- `triton`
- `triton.language`
