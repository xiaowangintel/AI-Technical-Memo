# chunk_intra.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk_intra.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects chunk intra helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 chunk intra 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 4-19: imports
```python
import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.chunk_intra_token_parallel import (
    chunk_kda_fwd_intra_token_parallel,
)
from sglang.srt.layers.attention.fla.index import (
    prepare_chunk_indices,
)
from sglang.srt.layers.attention.fla.op import exp, exp2, gather
from sglang.srt.layers.attention.fla.utils import (
    autotune_cache_kwargs,
    is_gather_supported,
    is_tf32_supported,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 21-24: conditional branch
```python
if is_tf32_supported:
    SOLVE_TRIL_DOT_PRECISION = tl.constexpr("tf32")
else:
    SOLVE_TRIL_DOT_PRECISION = tl.constexpr("ieee")
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 32-779: function chunk_kda_fwd_kernel_inter_solve_fused
```python
@triton.heuristics(
    {
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BK": BK, "BV": 64}, num_warps=num_warps)
        for BK in [32, 64]
        for num_warps in [1, 2, 4]
    ],
    key=["H", "K", "BC", "V", "FUSE_RECOMPUTE", "FUSE_DIAGONAL"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_fwd_kernel_inter_solve_fused(
    q,
    k,
    g,
    beta,
    Aqk,
    Akkd,
    Akk,
    scale,
    v_in,
    w_out,
    u_out,
    kg_out,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
# ... omitted 704 lines ...
        )

        tl.store(p_Akk00, b_Ai00.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk10, b_Ai10.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk11, b_Ai11.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk20, b_Ai20.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk21, b_Ai21.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk22, b_Ai22.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk30, b_Ai30.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk31, b_Ai31.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk32, b_Ai32.to(Akk.dtype.element_ty), boundary_check=(0, 1))
        tl.store(p_Akk33, b_Ai33.to(Akk.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk kda fwd kernel inter solve fused routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd kernel inter solve fused 例程。

### Lines 782-905: function chunk_kda_fwd_kernel_intra_sub_chunk
```python
@triton.heuristics(
    {
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [1, 2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["BT", "BC"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_fwd_kernel_intra_sub_chunk(
    q,
    k,
    g,
    beta,
    Aqk,
    Akk,
    scale,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    IS_VARLEN: tl.constexpr,
# ... omitted 80 lines ...
    ################################################################################
    # forward substitution
    ################################################################################

    b_Ai = -b_Akk
    for i in range(2, min(BC, T - i_ti)):
        b_a = -tl.load(Akk + (i_ti + i) * H * BC + o_i)
        b_a = tl.where(o_i < i, b_a, 0.0)
        b_a += tl.sum(b_a[:, None] * b_Ai, 0)
        b_Ai = tl.where((o_i == i)[:, None], b_a, b_Ai)
    b_Ai += m_I
    tl.store(p_Akk, b_Ai.to(Akk.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk kda fwd kernel intra sub chunk routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd kernel intra sub chunk 例程。

### Lines 908-1053: function chunk_kda_fwd_intra
```python
def chunk_kda_fwd_intra(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    gk: torch.Tensor | None = None,
    beta: torch.Tensor | None = None,
    scale: float | None = None,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_size: int = 64,
    chunk_indices: torch.LongTensor | None = None,
    safe_gate: bool = False,
    disable_recompute: bool = False,
    fuse_recompute: bool = False,
    fuse_diagonal: bool = False,
):
    B, T, H, K = k.shape
    V = v.shape[-1]
    BT = chunk_size
    BC = 16
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)
    NC = triton.cdiv(BT, BC)

    if fuse_diagonal:
        Aqk = torch.zeros(B, T, H, BT, device=k.device, dtype=k.dtype)
    else:
        Aqk = torch.empty(B, T, H, BT, device=k.device, dtype=k.dtype)
    Akkd = torch.empty(B, T, H, BC, device=k.device, dtype=torch.float32)

    # Step 1: compute diagonal blocks into Akkd (fp32)
    # When fuse_diagonal=True, diagonal blocks are computed inside inter_solve
# ... omitted 102 lines ...

    w, u, qg, kg = kda_recompute_w_u_fwd(
        k=k,
        v=v,
        beta=beta,
        A=Akk,
        q=q if disable_recompute else None,
        gk=gk,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )
    return w, u, qg, kg, Aqk, Akk
```
**EN:** Implements the chunk kda fwd intra routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd intra 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.fla.chunk_intra_token_parallel.chunk_kda_fwd_intra_token_parallel`
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.op.exp`
- `sglang.srt.layers.attention.fla.op.exp2`
- `sglang.srt.layers.attention.fla.op.gather`
- `sglang.srt.layers.attention.fla.utils.autotune_cache_kwargs`
- `sglang.srt.layers.attention.fla.utils.is_gather_supported`
- `sglang.srt.layers.attention.fla.utils.is_tf32_supported`
- `sglang.srt.layers.attention.fla.kda.recompute_w_u_fwd`
