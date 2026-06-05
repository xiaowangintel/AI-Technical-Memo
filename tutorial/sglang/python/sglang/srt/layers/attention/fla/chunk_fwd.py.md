# chunk_fwd.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/chunk_fwd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects chunk fwd helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 chunk fwd 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 4-14: imports
```python
import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.index import prepare_chunk_indices
from sglang.srt.layers.attention.fla.op import safe_exp
from sglang.srt.layers.attention.fla.utils import (
    autotune_cache_kwargs,
    is_tf32_supported,
)
from sglang.srt.layers.attention.fla.wy_fast import recompute_w_u_fwd
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 18-21: conditional branch
```python
if is_tf32_supported:
    _MERGE_DOT_PRECISION = tl.constexpr("tf32")
else:
    _MERGE_DOT_PRECISION = tl.constexpr("ieee")
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 24-336: function chunk_gated_delta_rule_fwd_kkt_solve_kernel
```python
@triton.heuristics(
    {
        "USE_G": lambda args: args["g"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BK": BK}, num_warps=num_warps)
        for BK in [32, 64]
        for num_warps in [1, 2, 4]
    ],
    key=["H", "Hg", "K", "BC"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T"])
def chunk_gated_delta_rule_fwd_kkt_solve_kernel(
    k,
    g,
    beta,
    A,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    USE_G: tl.constexpr,
    IS_VARLEN: tl.constexpr,
# ... omitted 269 lines ...
    )

    tl.store(p_A00, b_Ai00.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A10, b_Ai10.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A11, b_Ai11.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A20, b_Ai20.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A21, b_Ai21.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A22, b_Ai22.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A30, b_Ai30.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A31, b_Ai31.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A32, b_Ai32.to(A.dtype.element_ty), boundary_check=(0, 1))
    tl.store(p_A33, b_Ai33.to(A.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk gated delta rule fwd kkt solve kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gated delta rule fwd kkt solve kernel 例程。

### Lines 339-416: function chunk_gated_delta_rule_fwd_intra
```python
def chunk_gated_delta_rule_fwd_intra(
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor | None = None,
    beta: torch.Tensor | None = None,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_size: int = 64,
    chunk_indices: torch.LongTensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    r"""
    GDN intra-chunk forward: fused kkt + solve_tril + recompute_w_u.

    Equivalent to:
        A = chunk_scaled_dot_kkt_fwd(k, g, beta, ...)       # kernel 1
        A = solve_tril(A, ...)                                # kernel 2
        w, u = recompute_w_u_fwd(k, v, beta, A, g, ...)      # kernel 3

    Fuses kernels 1+2 into a single kernel, reducing from 3 to 2 kernel launches
    and eliminating the HBM round-trip for the intermediate A matrix.

    Args:
        k (torch.Tensor):
            The key tensor of shape `[B, T, H, K]`.
        v (torch.Tensor):
            The value tensor of shape `[B, T, H, V]`.
        g (torch.Tensor):
            The cumulative sum of the gate tensor of shape `[B, T, H]`. Default: `None`.
        beta (torch.Tensor):
            The beta tensor of shape `[B, T, H]`.
        cu_seqlens (torch.LongTensor):
            The cumulative sequence lengths. Default: `None`.
        chunk_size (int):
# ... omitted 34 lines ...

    # Step 2: recompute_w_u
    w, u = recompute_w_u_fwd(
        k=k,
        v=v,
        beta=beta,
        A=A,
        g_cumsum=g,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )
    return w, u, A
```
**EN:** Implements the chunk gated delta rule fwd intra routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gated delta rule fwd intra 例程。

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
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.op.safe_exp`
- `sglang.srt.layers.attention.fla.utils.autotune_cache_kwargs`
- `sglang.srt.layers.attention.fla.utils.is_tf32_supported`
- `sglang.srt.layers.attention.fla.wy_fast.recompute_w_u_fwd`
