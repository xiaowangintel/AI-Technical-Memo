# ssd_bmm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/ssd_bmm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects ssd bmm helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 ssd bmm 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 11-15: imports
```python
import math

import torch
import triton
import triton.language as tl
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 18-128: function _bmm_chunk_fwd_kernel
```python
@triton.jit
def _bmm_chunk_fwd_kernel(
    # Pointers to matrices
    a_ptr,
    b_ptr,
    out_ptr,
    seq_idx_ptr,
    # Matrix dimensions
    seqlen,
    chunk_size,
    K,
    ngroups,
    stride_a_batch,
    stride_a_seqlen,
    stride_a_head,
    stride_ak,
    stride_b_batch,
    stride_b_seqlen,
    stride_b_head,
    stride_bk,
    stride_out_batch,
    stride_out_chunk,
    stride_out_head,
    stride_outm,
    stride_outn,
    stride_seq_idx_batch,
    stride_seq_idx_seqlen,
    # Meta-parameters
    IS_CAUSAL: tl.constexpr,
    dot_dtype: tl.constexpr,
    HAS_SEQ_IDX: tl.constexpr,
    BLOCK_SIZE_M: tl.constexpr = 16,
# ... omitted 67 lines ...
        acc = tl.where(seq_idx_m[:, None] == seq_idx_n[None, :], acc, 0.0)
    out = acc.to(out_ptr.dtype.element_ty)

    out_ptr += (
        pid_b * stride_out_batch + pid_c * stride_out_chunk + pid_h * stride_out_head
    )
    out_ptrs = out_ptr + (stride_outm * offs_m[:, None] + offs_n[None, :] * stride_outn)
    tl.store(
        out_ptrs,
        out,
        mask=(offs_m[:, None] < chunk_size) & (offs_n[None, :] < chunk_size),
    )
```
**EN:** Implements the bmm chunk fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 bmm chunk fwd kernel 例程。

### Lines 131-214: function _bmm_chunk_fwd
```python
def _bmm_chunk_fwd(a, b, chunk_size, seq_idx=None, causal=False, output_dtype=None):
    """
    Argument:
        a: (batch, seqlen, k) or (batch, seqlen, ngroups, k)
        b: (batch, seqlen, k) or (batch, seqlen, ngroups, k)
        seq_idx: (batch, seqlen) or None. out[i, j] for seq_idx[i] != seq_idx[j] will be zeroed out.
        causal: if True, then out[i, j] for i > j will be arbitrary, only out[i, j] for i <= j are
            guaranteed to be correct.
    Return:
        out: (batch, nchunks, chunk_size, chunk_size) or (batch, nchunks, ngroups, chunk_size, chunk_size)
    """
    # Check constraints.
    has_groups = a.dim() == 4
    if not has_groups:
        batch, seqlen, k = a.shape
    else:
        batch, seqlen, ngroups, k = a.shape
    assert b.shape == a.shape
    if seq_idx is not None:
        assert seq_idx.shape == (batch, seqlen)
    if a.stride(-1) != 1 and a.stride(1) != 1:
        a = a.contiguous()
    if b.stride(-1) != 1 and b.stride(1) != 1:
        b = b.contiguous()
    nchunks = math.ceil(seqlen / chunk_size)
    # Allocates output.
    out_dtype = a.dtype if output_dtype is None else output_dtype
    out = torch.empty(
        (
            (batch, nchunks, chunk_size, chunk_size)
            if not has_groups
            else (batch, nchunks, ngroups, chunk_size, chunk_size)
# ... omitted 40 lines ...
            out.stride(-2),
            out.stride(-1),
            *(
                (seq_idx.stride(0), seq_idx.stride(1))
                if seq_idx is not None
                else (0, 0)
            ),
            causal,
            dot_dtype,
            HAS_SEQ_IDX=seq_idx is not None,
        )
    return out
```
**EN:** Implements the bmm chunk fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 bmm chunk fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `math`
- `torch`
- `triton`
- `triton.language`
