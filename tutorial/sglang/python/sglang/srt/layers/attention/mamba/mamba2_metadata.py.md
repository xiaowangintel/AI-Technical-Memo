# mamba2_metadata.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/mamba2_metadata.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the mamba2 metadata structures that carry execution metadata through the attention pipeline. / 该模块定义 mamba2 metadata 结构，用于在注意力流水线中传递执行元数据。
## Line-by-Line Analysis / 逐行分析
### Lines 19-25: imports
```python
import math
from dataclasses import dataclass
from typing import Optional

import torch

from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 28-49: class ForwardMetadata
```python
@dataclass(kw_only=True)
class ForwardMetadata:
    query_start_loc: torch.Tensor
    mamba_cache_indices: torch.Tensor
    mamba_cache_indices_gdn: Optional[torch.Tensor] = None
    # For topk > 1 eagle
    retrieve_next_token: Optional[torch.Tensor] = None
    retrieve_next_sibling: Optional[torch.Tensor] = None
    retrieve_parent_token: Optional[torch.Tensor] = None
    # For prefill radix cache
    track_conv_indices: Optional[torch.Tensor] = None
    track_ssm_h_src: Optional[torch.Tensor] = None
    track_ssm_h_dst: Optional[torch.Tensor] = None
    track_ssm_final_src: Optional[torch.Tensor] = None
    track_ssm_final_dst: Optional[torch.Tensor] = None

    is_target_verify: bool = False
    draft_token_num: int = 1

    has_mamba_track_mask: bool = False
    mamba_track_mask_indices: Optional[torch.Tensor] = None
    conv_states_mask_indices: Optional[torch.Tensor] = None
```
**EN:** Dataclass-style container that stores structured runtime state for forward metadata.
**CN:** 该数据类风格的容器用于存储 forward metadata 的结构化运行时状态。

### Lines 52-59: class Mamba2Metadata
```python
@dataclass(kw_only=True)
class Mamba2Metadata(ForwardMetadata):
    """stable metadata across all mamba2 layers in the forward pass"""

    num_prefills: int
    num_prefill_tokens: int
    num_decodes: int
```
**EN:** Dataclass-style container that stores structured runtime state for mamba2 metadata.
**CN:** 该数据类风格的容器用于存储 mamba2 metadata 的结构化运行时状态。

### Lines 60-70: class MixedMetadata
```python
    @dataclass(kw_only=True, frozen=True)
    class MixedMetadata:
        has_initial_states: torch.Tensor
        prep_initial_states: bool

        chunk_size: int
        seq_idx: torch.Tensor
        chunk_indices: torch.Tensor
        chunk_offsets: torch.Tensor

        extend_seq_lens_cpu: list[int]
```
**EN:** Dataclass-style container that stores structured runtime state for mixed metadata.
**CN:** 该数据类风格的容器用于存储 mixed metadata 的结构化运行时状态。

### Lines 72-72: Mamba2Metadata fields
```python
    mixed_metadata: MixedMetadata | None = None
```
**EN:** Defines class-level fields, defaults, or reusable constants consumed by later methods.
**CN:** 定义后续方法会使用的类级字段、默认值或可复用常量。

### Lines 73-73: docstring
```python
    """`mixed_metadata` is used for extend/mixed requests"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 75-157: method Mamba2Metadata._query_start_loc_to_chunk_indices_offsets
```python
    @staticmethod
    def _query_start_loc_to_chunk_indices_offsets(
        query_start_loc: torch.Tensor, chunk_size: int, total_seqlens: int
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Args:
            query_start_loc (torch.Tensor): 1D tensor of cumulative sequence
                lengths, shape (num_seqs + 1,).
                The first element should be 0. Each entry represents the starting
                index of a sequence in the flattened token array.
            chunk_size (int): The size of each physical mamba chunk
                (number of tokens per chunk).
            total_seqlens (int): The total number of tokens in the batch.

        Returns:
            Tuple[torch.Tensor, torch.Tensor]: A tuple containing:
                - chunk_indices (torch.Tensor): 1D tensor of indices
                    indicating the physical chunk for each logical chunk.
                - chunk_offsets (torch.Tensor): 1D tensor of offsets
                    indicating the starting index of each logical chunk within
                    its physical chunk.

        This function computes the chunk indices and offsets for the given
        query_start_loc and chunk_size. Both are tensors of integers with length N,
        where N is the number of logical (pseudo) chunks.
        A logical chunk is a sequence of tokens that are all part of the same
        sequence and are all in the same physical mamba chunk.
        In other words, a logical chunk changes every time we cross a sequence
        boundary or a physical mamba chunk boundary.
        Logical chunks are needed to handle batched requests with initial states
        (see _state_passing_fwd and _chunk_scan_fwd).
        The chunk_indices tensor contains the index of the physical chunk for each
# ... omitted 39 lines ...
            p += s % chunk_size > 0

            # get the dimensions
            # - the + 1 for _e is to shift the boundary by one chunk
            # - this shifting is not needed if chunk_size divides e
            _s, _e = s // chunk_size + p, e // chunk_size + p + (e % chunk_size > 0)

            # adjust indices and offsets
            chunk_indices[_s:_e] -= p
            chunk_offsets[_s] = s % chunk_size

        return chunk_indices, chunk_offsets
```
**EN:** Implements the query start loc to chunk indices offsets routine used by this attention module.
**CN:** 实现该注意力模块使用的 query start loc to chunk indices offsets 例程。

### Lines 159-179: method Mamba2Metadata.prepare_decode
```python
    @staticmethod
    def prepare_decode(
        forward_metadata: ForwardMetadata,
        seq_lens: torch.Tensor,
        *,
        is_target_verify: bool,
        draft_token_num: int,
    ) -> "Mamba2Metadata":
        """This path is run during CUDA graph capture, i.e. decode only, so `num_prefills` is 0"""
        return Mamba2Metadata(
            query_start_loc=forward_metadata.query_start_loc,
            mamba_cache_indices=forward_metadata.mamba_cache_indices,
            retrieve_next_token=forward_metadata.retrieve_next_token,
            retrieve_next_sibling=forward_metadata.retrieve_next_sibling,
            retrieve_parent_token=forward_metadata.retrieve_parent_token,
            num_decodes=len(seq_lens),
            num_prefills=0,
            num_prefill_tokens=0,
            is_target_verify=is_target_verify,
            draft_token_num=draft_token_num,
        )
```
**EN:** Prepares prepare decode so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare decode，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 181-256: method Mamba2Metadata.prepare_mixed
```python
    @classmethod
    def prepare_mixed(
        cls,
        forward_metadata: ForwardMetadata,
        chunk_size: int,
        forward_batch: ForwardBatch,
    ) -> "Mamba2Metadata":
        """This path cannot run with CUDA graph, as it contains extend requests."""
        if forward_batch.extend_num_tokens is None:
            draft_token_num = (
                forward_batch.spec_info.draft_token_num
                if forward_batch.spec_info is not None
                else 1
            )
            return cls.prepare_decode(
                forward_metadata,
                forward_batch.seq_lens,
                is_target_verify=forward_batch.forward_mode.is_target_verify(),
                draft_token_num=draft_token_num,
            )
        num_prefills = len(forward_batch.extend_seq_lens)
        num_prefill_tokens = forward_batch.extend_num_tokens
        num_decodes = len(forward_batch.seq_lens) - num_prefills
        context_lens_tensor = forward_batch.extend_prefix_lens
        assert context_lens_tensor is not None
        # precompute flag to avoid device syncs later
        has_initial_states = context_lens_tensor > 0
        prep_initial_states = torch.any(has_initial_states[:num_prefills]).item()

        query_start_loc = forward_metadata.query_start_loc[: num_prefills + 1]
        seq_idx = torch.repeat_interleave(
            torch.arange(
# ... omitted 32 lines ...
            is_target_verify=forward_batch.forward_mode.is_target_verify(),
            draft_token_num=draft_token_num,
            mixed_metadata=cls.MixedMetadata(
                has_initial_states=has_initial_states,
                prep_initial_states=prep_initial_states,
                chunk_size=chunk_size,
                seq_idx=seq_idx,
                chunk_indices=chunk_indices,
                chunk_offsets=chunk_offsets,
                extend_seq_lens_cpu=forward_batch.extend_seq_lens_cpu,
            ),
        )
```
**EN:** Prepares prepare mixed so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 prepare mixed，使后续内核能够使用正确的元数据、布局和缓存状态执行。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Compressed or sparse attention helpers / **CN:** 压缩或稀疏注意力辅助逻辑

## Dependencies / 依赖关系
- `math`
- `dataclasses.dataclass`
- `typing.Optional`
- `torch`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
