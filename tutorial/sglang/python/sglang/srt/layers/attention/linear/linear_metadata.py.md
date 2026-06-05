# linear_metadata.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/linear_metadata.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the linear metadata structures that carry execution metadata through the attention pipeline. / 该模块定义 linear metadata 结构，用于在注意力流水线中传递执行元数据。
## Line-by-Line Analysis / 逐行分析
### Lines 1-6: imports
```python
from dataclasses import dataclass

import torch

from sglang.srt.layers.attention.mamba.mamba2_metadata import ForwardMetadata
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 9-17: class BailingLinearMetadata
```python
@dataclass(kw_only=True)
class BailingLinearMetadata(ForwardMetadata):
    num_prefills: int
    num_prefill_tokens: int
    num_decodes: int
    batch_size: int
    has_initial_states: torch.Tensor
    q_lengths: torch.Tensor
```
**EN:** Dataclass-style container that stores structured runtime state for bailing linear metadata.
**CN:** 该数据类风格的容器用于存储 bailing linear metadata 的结构化运行时状态。

### Lines 18-35: method BailingLinearMetadata.prepare_decode
```python
    @staticmethod
    def prepare_decode(
        query_start_loc: torch.Tensor,
        mamba_cache_indices: torch.Tensor,
        bs: int,
        seq_lens: torch.Tensor,
    ) -> "BailingLinearMetadata":
        """This path is run during CUDA graph capture, i.e. decode only, so `num_prefills` is 0"""
        return BailingLinearMetadata(
            batch_size=bs,
            query_start_loc=query_start_loc,
            mamba_cache_indices=mamba_cache_indices,
            num_decodes=seq_lens.shape[0],
            num_prefills=0,
            num_prefill_tokens=0,
            has_initial_states=torch.ones_like(seq_lens),
            q_lengths=query_start_loc.diff(),
        )
```
**EN:** Prepares prepare decode so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare decode，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 37-70: method BailingLinearMetadata.prepare_mixed
```python
    @classmethod
    def prepare_mixed(
        cls,
        query_start_loc: torch.Tensor,
        mamba_cache_indices: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> "BailingLinearMetadata":
        """This path cannot run with CUDA graph, as it contains extend requests."""
        if forward_batch.extend_num_tokens is None:
            return cls.prepare_decode(
                query_start_loc=query_start_loc,
                mamba_cache_indices=mamba_cache_indices,
                bs=forward_batch.batch_size,
                seq_lens=forward_batch.seq_lens,
            )
        num_prefills = len(forward_batch.extend_seq_lens)
        num_prefill_tokens = forward_batch.extend_num_tokens
        num_decodes = len(forward_batch.seq_lens) - num_prefills
        context_lens_tensor = forward_batch.extend_prefix_lens
        assert context_lens_tensor is not None
        has_initial_states = context_lens_tensor > 0

        query_start_loc = query_start_loc[: num_prefills + 1]

        return BailingLinearMetadata(
            batch_size=forward_batch.batch_size,
            query_start_loc=query_start_loc,
            mamba_cache_indices=mamba_cache_indices,
            num_prefills=num_prefills,
            num_prefill_tokens=num_prefill_tokens,
            num_decodes=num_decodes,
            has_initial_states=has_initial_states,
            q_lengths=query_start_loc.diff(),
        )
```
**EN:** Prepares prepare mixed so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare mixed，使后续内核能够使用正确的元数据、布局和缓存状态执行。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `dataclasses.dataclass`
- `torch`
- `sglang.srt.layers.attention.mamba.mamba2_metadata.ForwardMetadata`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
