# chunk_intra_token_parallel.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk_intra_token_parallel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects chunk intra token parallel helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 chunk intra token parallel 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 5-10: imports
```python
import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.op import exp2
from sglang.srt.layers.attention.fla.utils import autotune_cache_kwargs
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 13-137: function chunk_kda_fwd_kernel_intra_token_parallel
```python
@triton.heuristics(
    {
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BH": BH}, num_warps=num_warps)
        for BH in [1, 2, 4, 8]
        for num_warps in [1, 2, 4, 8]
    ],
    key=["K", "H"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T", "N"])
def chunk_kda_fwd_kernel_intra_token_parallel(
    q,
    k,
    g,
    beta,
    Aqk,
    Akk,
    scale,
    cu_seqlens,
    N,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    BH: tl.constexpr,
# ... omitted 81 lines ...
        b_Akk = tl.sum(b_k * b_kgj, axis=1) * tl.where(j < i_t, 1.0, 0.0)

        tl.store(
            Aqk + i_t * H * BT + (i_hg * BH + o_h) * BT + j % BT,
            b_Aqk.to(Aqk.dtype.element_ty),
            mask=m_h,
        )
        tl.store(
            Akk + i_t * H * BC + (i_hg * BH + o_h) * BC + j - i_ts,
            b_Akk.to(Akk.dtype.element_ty),
            mask=m_h,
        )
```
**EN:** Implements the chunk kda fwd kernel intra token parallel routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd kernel intra token parallel 例程。

### Lines 140-197: function chunk_kda_fwd_intra_token_parallel
```python
def chunk_kda_fwd_intra_token_parallel(
    q: torch.Tensor,
    k: torch.Tensor,
    gk: torch.Tensor,
    beta: torch.Tensor,
    Aqk: torch.Tensor,
    Akk: torch.Tensor,
    scale: float,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_size: int = 64,
    sub_chunk_size: int = 16,
) -> None:
    """
    Token-parallel implementation: each token gets its own thread block.
    Supports both fixed-length and variable-length sequences.
    Reduces wasted computation on padding.

    Writes directly to Aqk and Akk tensors (in-place).

    Args:
        q: [B, T, H, K]
        k: [B, T, H, K]
        gk: [B, T, H, K] cumsum of gates
        beta: [B, T, H]
        Aqk: [B, T, H, BT] output tensor to write to
        Akk: [B, T, H, BC] output tensor for diagonal blocks (fp32)
        scale: attention scale
        chunk_size: BT (default 64)
        sub_chunk_size: BC (default 16)
    """
    B, T, H, K = q.shape
    N = len(cu_seqlens) - 1 if cu_seqlens is not None else B
# ... omitted 14 lines ...
        Akk=Akk,
        scale=scale,
        cu_seqlens=cu_seqlens,
        N=N,
        T=T,
        H=H,
        K=K,
        BT=BT,
        BC=BC,
        BK=BK,
    )
    return Aqk, Akk
```
**EN:** Implements the chunk kda fwd intra token parallel routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd intra token parallel 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.fla.op.exp2`
- `sglang.srt.layers.attention.fla.utils.autotune_cache_kwargs`
