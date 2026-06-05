# index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/index.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 10-14 — imports and setup
```python
import torch

from vllm.triton_utils import triton

from .utils import tensor_cache
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 17-19 — function `prepare_lens`
```python
@tensor_cache
def prepare_lens(cu_seqlens: torch.Tensor) -> torch.Tensor:
    return cu_seqlens[1:] - cu_seqlens[:-1]
```
**EN:** This function defines `prepare_lens`. It provides one of the file's main runtime building blocks. The main inputs are `cu_seqlens`.
**CN:** 该函数定义 `prepare_lens`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `cu_seqlens`。

### Lines 22-30 — function `prepare_chunk_indices`
```python
@tensor_cache
def prepare_chunk_indices(cu_seqlens: torch.Tensor, chunk_size: int) -> torch.Tensor:
    indices = torch.cat(
        [
            torch.arange(n)
            for n in triton.cdiv(prepare_lens(cu_seqlens), chunk_size).tolist()
        ]
    )
    return torch.stack([indices.eq(0).cumsum(0) - 1, indices], 1).to(cu_seqlens)
```
**EN:** This function defines `prepare_chunk_indices`. It provides one of the file's main runtime building blocks. The main inputs are `cu_seqlens`, `chunk_size`. Key calls include `torch.cat`, `torch.stack.to`, `torch.arange`, `torch.stack`, `triton.cdiv.tolist`, `triton.cdiv`. It writes or updates `indices`.
**CN:** 该函数定义 `prepare_chunk_indices`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `cu_seqlens`, `chunk_size`。 关键调用包括 `torch.cat`, `torch.stack.to`, `torch.arange`, `torch.stack`, `triton.cdiv.tolist`, `triton.cdiv`。 它会写入或更新 `indices`。

### Lines 33-37 — function `prepare_chunk_offsets`
```python
@tensor_cache
def prepare_chunk_offsets(cu_seqlens: torch.Tensor, chunk_size: int) -> torch.Tensor:
    return torch.cat(
        [cu_seqlens.new_tensor([0]), triton.cdiv(prepare_lens(cu_seqlens), chunk_size)]
    ).cumsum(-1)
```
**EN:** This function defines `prepare_chunk_offsets`. It provides one of the file's main runtime building blocks. The main inputs are `cu_seqlens`, `chunk_size`. Key calls include `torch.cat.cumsum`, `torch.cat`, `cu_seqlens.new_tensor`, `triton.cdiv`, `prepare_lens`.
**CN:** 该函数定义 `prepare_chunk_offsets`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `cu_seqlens`, `chunk_size`。 关键调用包括 `torch.cat.cumsum`, `torch.cat`, `cu_seqlens.new_tensor`, `triton.cdiv`, `prepare_lens`。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `prepare_lens`, `prepare_chunk_indices`, `prepare_chunk_offsets` / [CN] 核心符号：`prepare_lens`, `prepare_chunk_indices`, `prepare_chunk_offsets`

## Dependencies / 依赖关系
- **External**: `torch`, `utils` / **外部依赖**: `torch`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
