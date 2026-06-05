# index.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects index helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 index 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 5-8: imports
```python
import torch
import triton

from sglang.srt.layers.attention.fla.utils import tensor_cache
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 11-13: function prepare_lens
```python
@tensor_cache
def prepare_lens(cu_seqlens: torch.LongTensor) -> torch.LongTensor:
    return cu_seqlens[1:] - cu_seqlens[:-1]
```
**EN:** Prepares prepare lens so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare lens，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 16-26: function prepare_chunk_indices
```python
@tensor_cache
def prepare_chunk_indices(
    cu_seqlens: torch.LongTensor, chunk_size: int
) -> torch.LongTensor:
    indices = torch.cat(
        [
            torch.arange(n)
            for n in triton.cdiv(prepare_lens(cu_seqlens), chunk_size).tolist()
        ]
    )
    return torch.stack([indices.eq(0).cumsum(0) - 1, indices], 1).to(cu_seqlens)
```
**EN:** Prepares prepare chunk indices so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare chunk indices，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 29-35: function prepare_chunk_offsets
```python
@tensor_cache
def prepare_chunk_offsets(
    cu_seqlens: torch.LongTensor, chunk_size: int
) -> torch.LongTensor:
    return torch.cat(
        [cu_seqlens.new_tensor([0]), triton.cdiv(prepare_lens(cu_seqlens), chunk_size)]
    ).cumsum(-1)
```
**EN:** Prepares prepare chunk offsets so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare chunk offsets，使后续内核能够使用正确的元数据、布局和缓存状态执行。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成

## Dependencies / 依赖关系
- `torch`
- `triton`
- `sglang.srt.layers.attention.fla.utils.tensor_cache`
