# nsa_backend_mtp_precompute.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/nsa_backend_mtp_precompute.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the nsa backend mtp precompute attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 nsa backend mtp precompute 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-5: docstring
```python
"""Multi-step precompute utilities for Native Sparse Attention backend.

This module provides optimization utilities for multi-step speculative decoding
by precomputing shared metadata once and copying it to multiple backend instances.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 7-14: imports
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.layers.attention.nsa.utils import compute_nsa_seqlens
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 16-18: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardMode
    from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 21-50: class PrecomputedMetadata
```python
@dataclass
class PrecomputedMetadata:
    """Precomputed metadata shared across multiple backend instances.

    Used for multi-step speculative decoding where multiple backends
    need identical metadata. Precomputing once and copying N times
    is much faster than computing N times.

    """

    # Basic seqlens
    cache_seqlens: torch.Tensor  # int32, [bs]
    cu_seqlens_k: torch.Tensor  # int32, [bs+1]

    # Page table
    page_indices: torch.Tensor  # int32, [bs, max_len] or [expanded_bs, max_len]
    real_page_table: Optional[torch.Tensor]  # int32, transformed version

    # NSA seqlens
    seqlens_expanded: torch.Tensor  # int32, [expanded_size]
    nsa_cache_seqlens: torch.Tensor  # int32, [expanded_size]
    nsa_cu_seqlens_k: torch.Tensor  # int32, [expanded_size+1]
    seqlens_expanded_size: int

    # Dimensions
    max_len: int  # for decode/draft_extend
    max_seqlen_k: int  # for target_verify

    # FlashMLA (optional)
    flashmla_metadata: Optional[torch.Tensor] = None
```
**EN:** Dataclass-style container that stores structured runtime state for precomputed metadata.
**CN:** 该数据类风格的容器用于存储 precomputed metadata 的结构化运行时状态。

### Lines 53-58: function compute_cu_seqlens
```python
def compute_cu_seqlens(seqlens: torch.Tensor) -> torch.Tensor:
    """Compute cumulative sequence lengths with padding."""
    assert seqlens.dtype == torch.int32
    return torch.nn.functional.pad(
        torch.cumsum(seqlens, dim=0, dtype=torch.int32), (1, 0)
    )
```
**EN:** Implements the compute cu seqlens routine used by this attention module.
**CN:** 实现该注意力模块使用的 compute cu seqlens 例程。

### Lines 61-67: class NativeSparseAttnBackendMTPPrecomputeMixin
```python
class NativeSparseAttnBackendMTPPrecomputeMixin:
    """Mixin class providing metadata precomputation for multi-step speculative decoding.

    This mixin provides the _precompute_replay_metadata method and its helpers,
    which are used to optimize CUDA graph replay in multi-step scenarios.
    """
```
**EN:** Concrete attention backend that connects native sparse attn backend mtpprecompute mixin to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 native sparse attn backend mtpprecompute mixin 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 68-113: method NativeSparseAttnBackendMTPPrecomputeMixin._precompute_replay_metadata
```python
    def _precompute_replay_metadata(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        forward_mode: "ForwardMode",
        spec_info: Optional["SpecInput"],
    ) -> PrecomputedMetadata:
        """Precompute all shared metadata for multi-step backends.

        This function extracts and computes all operations that are
        identical across different backend instances in multi-step
        speculative decoding.

        Args:
            bs: Batch size
            req_pool_indices: Request pool indices [bs]
            seq_lens: Sequence lengths [bs]
            seq_lens_cpu: Sequence lengths on CPU [bs]
            forward_mode: Forward mode (decode/target_verify/draft_extend)
            spec_info: Speculative decoding info (for draft_extend mode)

        Returns:
            PrecomputedMetadata containing all shared intermediate results
        """
        # Slice inputs to batch size
        seq_lens = seq_lens[:bs]
        seq_lens_cpu = seq_lens_cpu[:bs]
        req_pool_indices = req_pool_indices[:bs]

        # Dispatch to mode-specific precomputation
        if forward_mode.is_decode_or_idle():
            return self._precompute_decode_mode(
                bs, req_pool_indices, seq_lens, seq_lens_cpu
            )
        elif forward_mode.is_target_verify():
            return self._precompute_target_verify_mode(
                bs, req_pool_indices, seq_lens, seq_lens_cpu
            )
        elif forward_mode.is_draft_extend():
            return self._precompute_draft_extend_mode(
                bs, req_pool_indices, seq_lens, seq_lens_cpu, spec_info
            )
        else:
            raise ValueError(f"Unsupported forward mode: {forward_mode}")
```
**EN:** Implements the precompute replay metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 precompute replay metadata 例程。

### Lines 115-168: method NativeSparseAttnBackendMTPPrecomputeMixin._precompute_decode_mode
```python
    def _precompute_decode_mode(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
    ) -> PrecomputedMetadata:
        """Precompute metadata for normal decode mode."""
        max_len = int(seq_lens_cpu.max().item())

        # Convert to int32 and compute cumsum
        cache_seqlens = seq_lens.to(torch.int32)
        cu_seqlens_k = compute_cu_seqlens(cache_seqlens)

        # Get page indices from cache
        page_indices = self.req_to_token[req_pool_indices, :max_len].contiguous()

        # Compute NSA seqlens
        nsa_cache_seqlens = compute_nsa_seqlens(
            cache_seqlens, nsa_index_topk=self.nsa_index_topk
        )
        seqlens_expanded = cache_seqlens
        seqlens_expanded_size = seqlens_expanded.shape[0]

        # Compute NSA cumsum
        nsa_cu_seqlens_k = compute_cu_seqlens(nsa_cache_seqlens)

        # Transform page table if needed
        if self.real_page_size > 1:
            real_page_table = self._transform_table_1_to_real(page_indices)
        else:
            real_page_table = None  # Will use page_indices directly
# ... omitted 10 lines ...
            cache_seqlens=cache_seqlens,
            cu_seqlens_k=cu_seqlens_k,
            page_indices=page_indices,
            real_page_table=real_page_table,
            seqlens_expanded=seqlens_expanded,
            nsa_cache_seqlens=nsa_cache_seqlens,
            nsa_cu_seqlens_k=nsa_cu_seqlens_k,
            seqlens_expanded_size=seqlens_expanded_size,
            max_len=max_len,
            max_seqlen_k=max_len,
            flashmla_metadata=flashmla_metadata,
        )
```
**EN:** Implements the precompute decode mode routine used by this attention module.
**CN:** 实现该注意力模块使用的 precompute decode mode 例程。

### Lines 170-247: method NativeSparseAttnBackendMTPPrecomputeMixin._precompute_target_verify_mode
```python
    def _precompute_target_verify_mode(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
    ) -> PrecomputedMetadata:
        """Precompute metadata for target verify mode."""
        max_seqlen_k = int(
            seq_lens_cpu.max().item() + self.speculative_num_draft_tokens
        )

        # Cache seqlens with draft tokens
        cache_seqlens = (seq_lens + self.speculative_num_draft_tokens).to(torch.int32)
        cu_seqlens_k = compute_cu_seqlens(cache_seqlens)

        # Page indices (repeated for each draft token)
        page_indices = self.req_to_token[req_pool_indices, :max_seqlen_k]
        page_indices = torch.repeat_interleave(
            page_indices, repeats=self.speculative_num_draft_tokens, dim=0
        ).contiguous()

        # Generate expanded seqlens
        extend_seq_lens_cpu = [self.speculative_num_draft_tokens] * bs
        seqlens_int32_cpu = [
            self.speculative_num_draft_tokens + kv_len
            for kv_len in seq_lens_cpu.tolist()
        ]
        seqlens_expanded = torch.cat(
            [
                torch.arange(
                    kv_len - qo_len + 1,
# ... omitted 34 lines ...
            cache_seqlens=cache_seqlens,
            cu_seqlens_k=cu_seqlens_k,
            page_indices=page_indices,
            real_page_table=real_page_table,
            seqlens_expanded=seqlens_expanded,
            nsa_cache_seqlens=nsa_cache_seqlens,
            nsa_cu_seqlens_k=nsa_cu_seqlens_k,
            seqlens_expanded_size=seqlens_expanded_size,
            max_len=-1,  # Not used in this mode
            max_seqlen_k=max_seqlen_k,
            flashmla_metadata=flashmla_metadata,
        )
```
**EN:** Implements the precompute target verify mode routine used by this attention module.
**CN:** 实现该注意力模块使用的 precompute target verify mode 例程。

### Lines 249-325: method NativeSparseAttnBackendMTPPrecomputeMixin._precompute_draft_extend_mode
```python
    def _precompute_draft_extend_mode(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        spec_info: "SpecInput",
    ) -> PrecomputedMetadata:
        """Precompute metadata for draft extend mode."""
        max_seqlen_k = int(seq_lens_cpu.max().item())

        # Cache seqlens
        cache_seqlens = seq_lens.to(torch.int32)
        cu_seqlens_k = compute_cu_seqlens(cache_seqlens)

        # Extend seqlens from spec_info: num_accept_tokens already includes
        # the bonus token (drafts + 1).
        extend_seq_lens = spec_info.num_accept_tokens[:bs]
        extend_seq_lens_cpu = extend_seq_lens.tolist()

        # Page indices (repeated per accept length)
        page_indices = self.req_to_token[req_pool_indices, :max_seqlen_k]
        page_indices = torch.repeat_interleave(
            page_indices, repeats=extend_seq_lens, dim=0
        ).contiguous()

        # Generate expanded seqlens
        seqlens_expanded = torch.cat(
            [
                torch.arange(
                    kv_len - qo_len + 1,
                    kv_len + 1,
# ... omitted 33 lines ...
            cache_seqlens=cache_seqlens,
            cu_seqlens_k=cu_seqlens_k,
            page_indices=page_indices,
            real_page_table=real_page_table,
            seqlens_expanded=seqlens_expanded,
            nsa_cache_seqlens=nsa_cache_seqlens,
            nsa_cu_seqlens_k=nsa_cu_seqlens_k,
            seqlens_expanded_size=seqlens_expanded_size,
            max_len=max_seqlen_k,
            max_seqlen_k=max_seqlen_k,
            flashmla_metadata=flashmla_metadata,
        )
```
**EN:** Implements the precompute draft extend mode routine used by this attention module.
**CN:** 实现该注意力模块使用的 precompute draft extend mode 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `__future__.annotations`
- `dataclasses.dataclass`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `torch`
- `sglang.srt.layers.attention.nsa.utils.compute_nsa_seqlens`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.speculative.spec_info.SpecInput`
