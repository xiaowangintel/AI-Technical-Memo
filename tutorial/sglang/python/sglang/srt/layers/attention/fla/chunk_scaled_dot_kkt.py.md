# chunk_scaled_dot_kkt.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk_scaled_dot_kkt.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects chunk scaled dot kkt helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 chunk scaled dot kkt 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 5-12: imports
```python
from typing import Optional

import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.index import prepare_chunk_indices
from sglang.srt.layers.attention.fla.op import safe_exp
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 24-86: function chunk_scaled_dot_kkt_fwd_kernel
```python
@triton.jit(do_not_specialize=["T"])
def chunk_scaled_dot_kkt_fwd_kernel(
    k,
    beta,
    g_cumsum,
    A,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    USE_G: tl.constexpr,
):
    i_t, i_bh = tl.program_id(0), tl.program_id(1)
    i_b, i_h = i_bh // H, i_bh % H
    if IS_VARLEN:
        i_n, i_t = tl.load(chunk_indices + i_t * 2).to(tl.int32), tl.load(
            chunk_indices + i_t * 2 + 1
        ).to(tl.int32)
        bos, eos = tl.load(cu_seqlens + i_n).to(tl.int32), tl.load(
            cu_seqlens + i_n + 1
        ).to(tl.int32)
        T = eos - bos
    else:
        bos, eos = i_b * T, i_b * T + T
    o_t = tl.arange(0, BT)

    p_beta = tl.make_block_ptr(
# ... omitted 19 lines ...
            g_cumsum + bos * H + i_h, (T,), (H,), (i_t * BT,), (BT,), (0,)
        )
        b_g = tl.load(p_g, boundary_check=(0,))
        b_g_diff = b_g[:, None] - b_g[None, :]
        b_A = b_A * safe_exp(b_g_diff)

    b_A *= b_beta[:, None]
    b_A = tl.where(o_t[:, None] > o_t[None, :], b_A, 0)
    p_A = tl.make_block_ptr(
        A + (bos * H + i_h) * BT, (T, BT), (BT * H, 1), (i_t * BT, 0), (BT, BT), (1, 0)
    )
    tl.store(p_A, b_A.to(p_A.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk scaled dot kkt fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk scaled dot kkt fwd kernel 例程。

### Lines 89-147: function chunk_scaled_dot_kkt_fwd
```python
def chunk_scaled_dot_kkt_fwd(
    k: torch.Tensor,
    beta: torch.Tensor,
    g_cumsum: Optional[torch.Tensor] = None,
    cu_seqlens: Optional[torch.LongTensor] = None,
    chunk_size: int = 64,
    output_dtype: torch.dtype = torch.float32,
) -> torch.Tensor:
    r"""
    Compute beta * K * K^T.

    Args:
        k (torch.Tensor):
            The key tensor of shape `[B, T, H, K]`.
        beta (torch.Tensor):
            The beta tensor of shape `[B, T, H]`.
        g_cumsum (torch.Tensor):
            The cumulative sum of the gate tensor of shape `[B, T, H]`.
            Default: None
        cu_seqlens (torch.LongTensor):
            The cumulative sequence lengths of the input tensor.
            Default: None
        chunk_size (int):
            The chunk size. Default: 64.
        output_dtype (torch.dtype):
            The dtype of the output tensor. Default: `torch.float32`

    Returns:
        beta * K * K^T of shape `[B, T, H, BT]` where `BT` is the chunk size.
    """

    B, T, Hg, K = k.shape
# ... omitted 15 lines ...
        T=T,
        H=H,
        Hg=Hg,
        K=K,
        BT=BT,
        BK=64,
        IS_VARLEN=cu_seqlens is not None,
        USE_G=g_cumsum is not None,
        num_warps=8,
        num_stages=3,
    )
    return A
```
**EN:** Implements the chunk scaled dot kkt fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk scaled dot kkt fwd 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.op.safe_exp`
