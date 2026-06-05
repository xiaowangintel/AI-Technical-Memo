# kda.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/kda.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects kda helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 kda 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 9-30: imports
```python
from typing import Optional

import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.chunk_delta_h import chunk_gated_delta_rule_fwd_h
from sglang.srt.layers.attention.fla.chunk_intra import chunk_kda_fwd_intra
from sglang.srt.layers.attention.fla.cumsum import chunk_local_cumsum
from sglang.srt.layers.attention.fla.fused_norm_gate import layer_norm_gated_fwd
from sglang.srt.layers.attention.fla.fused_recurrent import (
    fused_recurrent_gated_delta_rule_fwd_kernel,
)
from sglang.srt.layers.attention.fla.index import (
    prepare_chunk_indices,
)
from sglang.srt.layers.attention.fla.l2norm import l2norm_fwd
from sglang.srt.layers.attention.fla.op import exp, log
from sglang.srt.layers.attention.fla.utils import (
    check_shared_mem,
    is_intel,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 32-35: conditional branch
```python
if is_intel:
    from sglang.srt.hardware_backend.xpu.kernels.fla.chunk_delta_h import (
        chunk_gated_delta_rule_fwd_h,
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 38-38: module constants
```python
BS_LIST = [32, 64] if check_shared_mem() else [16, 32]
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 41-43: function cdiv
```python
def cdiv(a: int, b: int) -> int:
    """Ceiling division."""
    return -(a // -b)
```
**EN:** Implements the cdiv routine used by this attention module.
**CN:** 实现该注意力模块使用的 cdiv 例程。

### Lines 46-50: function next_power_of_2
```python
def next_power_of_2(n: int) -> int:
    """The next power of 2 (inclusive)"""
    if n < 1:
        return 1
    return 1 << (n - 1).bit_length()
```
**EN:** Implements the next power of 2 routine used by this attention module.
**CN:** 实现该注意力模块使用的 next power of 2 例程。

### Lines 53-129: function fused_recurrent_kda_fwd
```python
def fused_recurrent_kda_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    inplace_final_state: bool = True,
    cu_seqlens: torch.LongTensor | None = None,
    # ssm_state_indices: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    B, T, H, K, V = *k.shape, v.shape[-1]
    HV = v.shape[2]
    N = B if cu_seqlens is None else len(cu_seqlens) - 1
    BK, BV = next_power_of_2(K), min(next_power_of_2(V), 8)
    NK, NV = cdiv(K, BK), cdiv(V, BV)
    assert NK == 1, "NK > 1 is not supported yet"
    num_stages = 3
    num_warps = 1

    o = q.new_empty(NK, *v.shape)
    if inplace_final_state:
        final_state = initial_state
    else:
        final_state = q.new_empty(N, HV, V, K, dtype=initial_state.dtype)

    stride_init_state_token = initial_state.stride(0)
    stride_final_state_token = final_state.stride(0)

    # if ssm_state_indices is None:
# ... omitted 33 lines ...
        STORE_FINAL_STATE=final_state is not None,
        IS_BETA_HEADWISE=beta.ndim == v.ndim,
        USE_QK_L2NORM_IN_KERNEL=use_qk_l2norm_in_kernel,
        IS_VARLEN=cu_seqlens is not None,
        # INPLACE_FINAL_STATE=inplace_final_state,
        IS_KDA=True,
        num_warps=num_warps,
        num_stages=num_stages,
    )

    o = o.squeeze(0)
    return o, final_state
```
**EN:** Implements the fused recurrent kda fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused recurrent kda fwd 例程。

### Lines 132-167: function fused_recurrent_kda
```python
def fused_recurrent_kda(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor = None,
    scale: float = None,
    initial_state: torch.Tensor = None,
    inplace_final_state: bool = True,
    use_qk_l2norm_in_kernel: bool = True,
    cu_seqlens: torch.LongTensor | None = None,
    # ssm_state_indices: torch.LongTensor | None = None,
    **kwargs,
) -> tuple[torch.Tensor, torch.Tensor]:
    if cu_seqlens is not None and q.shape[0] != 1:
        raise ValueError(
            f"The batch size is expected to be 1 rather than {q.shape[0]} when using `cu_seqlens`."
            f"Please flatten variable-length inputs before processing."
        )
    if scale is None:
        scale = k.shape[-1] ** -0.5

    o, final_state = fused_recurrent_kda_fwd(
        q=q.contiguous(),
        k=k.contiguous(),
        v=v.contiguous(),
        g=g.contiguous(),
        beta=beta.contiguous(),
        scale=scale,
        initial_state=initial_state,
        inplace_final_state=inplace_final_state,
        cu_seqlens=cu_seqlens,
        # ssm_state_indices=ssm_state_indices,
        use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
    )
    return o, final_state
```
**EN:** Implements the fused recurrent kda routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused recurrent kda 例程。

### Lines 170-205: function rms_norm_gated
```python
def rms_norm_gated(
    x: torch.Tensor,
    g: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    activation: str = "swish",
    residual: torch.Tensor | None = None,
    prenorm: bool = False,
    residual_in_fp32: bool = False,
    eps: float = 1e-6,
):
    x_shape_og = x.shape
    # reshape input data into 2D tensor
    x = x.contiguous().reshape(-1, x.shape[-1])
    g = g.contiguous().reshape(-1, g.shape[-1])
    if residual is not None:
        assert residual.shape == x_shape_og
        residual = residual.contiguous().reshape(-1, residual.shape[-1])
    residual_dtype = (
        residual.dtype
        if residual is not None
        else (torch.float if residual_in_fp32 else None)
    )
    y, _, _, residual_out = layer_norm_gated_fwd(
        x=x,
        g=g,
        weight=weight,
        bias=bias,
        activation=activation,
        eps=eps,
        residual=residual,
        residual_dtype=residual_dtype,
        is_rms_norm=True,
    )
    y = y.reshape(x_shape_og)
    return y if not prenorm else (y, residual_out.reshape(x_shape_og))
```
**EN:** Implements the rms norm gated routine used by this attention module.
**CN:** 实现该注意力模块使用的 rms norm gated 例程。

### Lines 208-315: function chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter
```python
@triton.autotune(
    configs=[
        triton.Config({"BK": BK}, num_warps=num_warps, num_stages=num_stages)
        for BK in [32, 64]
        for num_warps in [1, 2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["BC", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_inter(
    q,
    k,
    g,
    beta,
    A,
    Aqk,
    scale,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BC: tl.constexpr,
    BK: tl.constexpr,
    NC: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_t, i_c, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H
    i_i, i_j = i_c // NC, i_c % NC
# ... omitted 64 lines ...
        b_Aqk += tl.dot(b_qg, b_ktg)

    b_A *= b_b[:, None]

    p_A = tl.make_block_ptr(
        A, (T, BT), (H * BT, 1), (i_t * BT + i_i * BC, i_j * BC), (BC, BC), (1, 0)
    )
    tl.store(p_A, b_A.to(A.dtype.element_ty), boundary_check=(0, 1))
    p_Aqk = tl.make_block_ptr(
        Aqk, (T, BT), (H * BT, 1), (i_t * BT + i_i * BC, i_j * BC), (BC, BC), (1, 0)
    )
    tl.store(p_Aqk, b_Aqk.to(Aqk.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk kda scaled dot kkt fwd kernel intra sub inter routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda scaled dot kkt fwd kernel intra sub inter 例程。

### Lines 318-410: function chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra
```python
@triton.autotune(
    configs=[triton.Config({}, num_warps=num_warps) for num_warps in [1, 2, 4, 8]],
    key=["BK", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_kda_scaled_dot_kkt_fwd_kernel_intra_sub_intra(
    q,
    k,
    g,
    beta,
    A,
    Aqk,
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
):
    i_t, i_i, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H
    if IS_VARLEN:
        i_n, i_t = (
            tl.load(chunk_indices + i_t * 2).to(tl.int32),
            tl.load(chunk_indices + i_t * 2 + 1).to(tl.int32),
        )
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
# ... omitted 49 lines ...
    for j in range(0, min(BC, T - i_t * BT - i_i * BC)):
        b_kt = tl.load(p_kt, mask=m_k, other=0).to(tl.float32)
        b_gk = tl.load(p_gk, mask=m_k, other=0).to(tl.float32)
        b_ktg = b_kt[None, :] * exp(b_g - b_gk[None, :])
        b_A = tl.sum(b_k * b_ktg, 1)
        b_A = tl.where(o_i > j, b_A, 0.0)
        b_Aqk = tl.sum(b_q * b_ktg, 1)
        b_Aqk = tl.where(o_i >= j, b_Aqk * scale, 0.0)
        tl.store(A + o_A + j, b_A, mask=m_A)
        tl.store(Aqk + o_A + j, b_Aqk, mask=m_A)
        p_kt += H * K
        p_gk += H * K
```
**EN:** Implements the chunk kda scaled dot kkt fwd kernel intra sub intra routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda scaled dot kkt fwd kernel intra sub intra 例程。

### Lines 413-496: function chunk_kda_scaled_dot_kkt_fwd
```python
def chunk_kda_scaled_dot_kkt_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    gk: torch.Tensor | None = None,
    beta: torch.Tensor | None = None,
    scale: float | None = None,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_size: int = 64,
    output_dtype: torch.dtype = torch.float32,
) -> tuple[torch.Tensor, torch.Tensor]:
    r"""
    Compute beta * K * K^T.

    Args:
        k (torch.Tensor):
            The key tensor of shape `[B, T, H, K]`.
        beta (torch.Tensor):
            The beta tensor of shape `[B, T, H]`.
        gk (torch.Tensor):
            The cumulative sum of the gate tensor of shape `[B, T, H, K]` applied to the key tensor. Default: `None`.
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
    B, T, H, K = k.shape
# ... omitted 40 lines ...
        scale=scale,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        T=T,
        H=H,
        K=K,
        BT=BT,
        BC=BC,
        BK=BK,
        IS_VARLEN=cu_seqlens is not None,
    )
    return A, Aqk
```
**EN:** Implements the chunk kda scaled dot kkt fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda scaled dot kkt fwd 例程。

### Lines 499-650: function recompute_w_u_fwd_kernel
```python
@triton.autotune(
    configs=[
        triton.Config({"BK": BK, "BV": BV}, num_warps=num_warps, num_stages=num_stages)
        for BK in [64, 128]
        for BV in [64, 128]
        for num_warps in [2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["H", "K", "V", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def recompute_w_u_fwd_kernel(
    q,
    k,
    qg,
    kg,
    v,
    beta,
    w,
    u,
    A,
    gk,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    STORE_QG: tl.constexpr,
# ... omitted 108 lines ...
            p_kg = tl.make_block_ptr(
                kg + (bos * H + i_h) * K,
                (T, K),
                (H * K, 1),
                (i_t * BT, i_k * BK),
                (BT, BK),
                (1, 0),
            )
            tl.store(p_kg, b_kg.to(p_kg.dtype.element_ty), boundary_check=(0, 1))

        b_w = tl.dot(b_A, b_kb.to(b_k.dtype))
        tl.store(p_w, b_w.to(p_w.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the recompute w u fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 recompute w u fwd kernel 例程。

### Lines 653-696: function recompute_w_u_fwd
```python
def recompute_w_u_fwd(
    k: torch.Tensor,
    v: torch.Tensor,
    beta: torch.Tensor,
    A: torch.Tensor,
    q: torch.Tensor | None = None,
    gk: torch.Tensor | None = None,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_indices: torch.LongTensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    B, T, H, K, V = *k.shape, v.shape[-1]
    BT = A.shape[-1]

    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)

    w = torch.empty_like(k)
    u = torch.empty_like(v)
    kg = torch.empty_like(k) if gk is not None else None
    recompute_w_u_fwd_kernel[(NT, B * H)](
        q=q,
        k=k,
        qg=None,
        kg=kg,
        v=v,
        beta=beta,
        w=w,
        u=u,
        A=A,
        gk=gk,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        T=T,
        H=H,
        K=K,
        V=V,
        BT=BT,
        STORE_QG=False,
        STORE_KG=kg is not None,
        IS_VARLEN=cu_seqlens is not None,
        DOT_PRECISION="tf32",
    )
    return w, u, None, kg
```
**EN:** Implements the recompute w u fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 recompute w u fwd 例程。

### Lines 699-815: function chunk_gla_fwd_kernel_o
```python
@triton.autotune(
    configs=[
        triton.Config({"BK": BK, "BV": BV}, num_warps=num_warps, num_stages=num_stages)
        for BK in [64]
        for BV in [64]
        for num_warps in [2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_gla_fwd_kernel_o(
    q,
    v,
    g,
    h,
    o,
    A,
    cu_seqlens,
    chunk_indices,
    scale,
    T,
    H: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_v, i_t, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H
# ... omitted 73 lines ...
        (1, 0),
    )
    p_A = tl.make_block_ptr(
        A + (bos * H + i_h) * BT, (T, BT), (H * BT, 1), (i_t * BT, 0), (BT, BT), (1, 0)
    )
    # [BT, BV]
    b_v = tl.load(p_v, boundary_check=(0, 1))
    # [BT, BT]
    b_A = tl.load(p_A, boundary_check=(0, 1))
    b_A = tl.where(m_s, b_A, 0.0).to(b_v.dtype)
    b_o += tl.dot(b_A, b_v)
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the chunk gla fwd kernel o routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gla fwd kernel o 例程。

### Lines 818-857: function chunk_gla_fwd_o_gk
```python
def chunk_gla_fwd_o_gk(
    q: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    A: torch.Tensor,
    h: torch.Tensor,
    o: torch.Tensor,
    scale: float,
    cu_seqlens: torch.LongTensor | None = None,
    chunk_size: int = 64,
    chunk_indices: torch.LongTensor | None = None,
):
    B, T, H, K, V = *q.shape, v.shape[-1]
    BT = chunk_size

    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)

    def grid(meta):
        return (cdiv(V, meta["BV"]), NT, B * H)

    chunk_gla_fwd_kernel_o[grid](
        q=q,
        v=v,
        g=g,
        h=h,
        o=o,
        A=A,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        scale=scale,
        T=T,
        H=H,
        K=K,
        V=V,
        BT=BT,
        IS_VARLEN=cu_seqlens is not None,
    )
    return o
```
**EN:** Implements the chunk gla fwd o gk routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk gla fwd o gk 例程。

### Lines 860-863: function softplus_fwd
```python
@triton.jit
def softplus_fwd(x):
    """Standard softplus: log(1 + exp(x)), with linear approx for large x."""
    return tl.where(x < 20.0, log(1.0 + exp(x)), x)
```
**EN:** Implements the softplus fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 softplus fwd 例程。

### Lines 866-961: function kda_gate_chunk_cumsum_vector_kernel
```python
@triton.heuristics(
    {
        "HAS_BIAS": lambda args: args["dt_bias"] is not None,
        "HAS_SCALE": lambda args: args["scale"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
        "USE_LOWER_BOUND": lambda args: args["lower_bound"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BS": BS}, num_warps=num_warps)
        for BS in BS_LIST
        for num_warps in [2, 4, 8]
    ],
    key=["H", "S", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def kda_gate_chunk_cumsum_vector_kernel(
    s,
    A_log,
    dt_bias,
    o,
    scale,
    cu_seqlens,
    chunk_indices,
    lower_bound,
    T,
    H: tl.constexpr,
    S: tl.constexpr,
    BT: tl.constexpr,
    BS: tl.constexpr,
    HAS_BIAS: tl.constexpr,
# ... omitted 52 lines ...
        # Standard gate: -exp(A_log) * softplus(g + bias)
        b_gate = -exp(b_A) * softplus_fwd(b_s)
    else:
        # Safe gate: lower_bound * sigmoid(exp(A_log) * (g + bias))
        b_gate = lower_bound * tl.sigmoid(exp(b_A) * b_s)

    # Chunk-local cumulative sum
    b_o = tl.cumsum(b_gate, axis=0)

    if HAS_SCALE:
        b_o *= scale
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** Implements the kda gate chunk cumsum vector kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 kda gate chunk cumsum vector kernel 例程。

### Lines 964-1029: function kda_gate_chunk_cumsum
```python
def kda_gate_chunk_cumsum(
    g: torch.Tensor,
    A_log: torch.Tensor,
    chunk_size: int,
    scale: float = None,
    dt_bias: Optional[torch.Tensor] = None,
    cu_seqlens: Optional[torch.Tensor] = None,
    output_dtype: Optional[torch.dtype] = torch.float,
    chunk_indices: Optional[torch.LongTensor] = None,
    lower_bound: Optional[float] = None,
) -> torch.Tensor:
    """
    Fused KDA gate activation + chunk-local cumulative sum.

    Combines two memory-bound kernels into one:
      1. Gate activation: g = -exp(A_log) * softplus(raw_g + dt_bias)
      2. Chunk-local cumsum along the time axis

    Args:
        g: Raw gate tensor of shape [B, T, H, K] (before activation).
        A_log: Per-head log-scale parameter, [H] elements (any shape, numel=H).
        chunk_size: Chunk size for cumsum (must be power of 2).
        scale: Optional scale factor applied to output.
        dt_bias: Optional per-head bias, flat [H*K] elements.
        cu_seqlens: Cumulative sequence lengths for variable-length input.
        output_dtype: Output dtype (default float32).
        chunk_indices: Pre-computed chunk indices for varlen mode.
        lower_bound: If set, use safe gate: lower_bound * sigmoid(exp(A_log) * g).

    Returns:
        Cumulative-summed gated tensor of shape [B, T, H, K].
    """
# ... omitted 22 lines ...
        dt_bias=dt_bias,
        o=g,
        scale=scale,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        lower_bound=lower_bound,
        T=T,
        H=H,
        S=S,
        BT=BT,
    )
    return g
```
**EN:** Implements the kda gate chunk cumsum routine used by this attention module.
**CN:** 实现该注意力模块使用的 kda gate chunk cumsum 例程。

### Lines 1032-1133: function chunk_kda_fwd
```python
def chunk_kda_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    initial_state_indices: torch.Tensor,
    cu_seqlens: Optional[torch.LongTensor] = None,
    A_log: Optional[torch.Tensor] = None,
    dt_bias: Optional[torch.Tensor] = None,
    lower_bound: Optional[float] = None,
):
    chunk_size = 64
    # Pre-compute chunk indices once and thread through all downstream kernels.
    # Without this, each of the 4 callees would recompute independently.
    chunk_indices = (
        prepare_chunk_indices(cu_seqlens, chunk_size)
        if cu_seqlens is not None
        else None
    )

    if A_log is not None:
        # Fused: gate activation + chunk-local cumsum in one kernel.
        # g is raw gate (before activation); A_log, dt_bias drive the activation.
        g = kda_gate_chunk_cumsum(
            g,
            A_log=A_log,
            chunk_size=chunk_size,
            dt_bias=dt_bias,
            cu_seqlens=cu_seqlens,
# ... omitted 58 lines ...
        g=g,
        A=Aqk,
        h=h,
        o=v,
        scale=scale,
        chunk_size=chunk_size,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )
    del Aqk, v_new, h

    return o
```
**EN:** Implements the chunk kda fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda fwd 例程。

### Lines 1136-1173: function chunk_kda
```python
def chunk_kda(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float = None,
    initial_state: torch.Tensor = None,
    initial_state_indices: torch.Tensor = None,
    use_qk_l2norm_in_kernel: bool = False,
    cu_seqlens: Optional[torch.LongTensor] = None,
    A_log: Optional[torch.Tensor] = None,
    dt_bias: Optional[torch.Tensor] = None,
    lower_bound: Optional[float] = None,
    **kwargs,
):
    if scale is None:
        scale = k.shape[-1] ** -0.5

    if use_qk_l2norm_in_kernel:
        q = l2norm_fwd(q.contiguous())
        k = l2norm_fwd(k.contiguous())

    o = chunk_kda_fwd(
        q=q,
        k=k,
        v=v.contiguous(),
        g=g.contiguous(),
        beta=beta.contiguous(),
        scale=scale,
        initial_state=initial_state,
        initial_state_indices=initial_state_indices,
        cu_seqlens=cu_seqlens,
        A_log=A_log,
        dt_bias=dt_bias,
        lower_bound=lower_bound,
    )
    return o
```
**EN:** Implements the chunk kda routine used by this attention module.
**CN:** 实现该注意力模块使用的 chunk kda 例程。

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
- `sglang.srt.layers.attention.fla.chunk_delta_h.chunk_gated_delta_rule_fwd_h`
- `sglang.srt.layers.attention.fla.chunk_intra.chunk_kda_fwd_intra`
- `sglang.srt.layers.attention.fla.cumsum.chunk_local_cumsum`
- `sglang.srt.layers.attention.fla.fused_norm_gate.layer_norm_gated_fwd`
- `sglang.srt.layers.attention.fla.fused_recurrent.fused_recurrent_gated_delta_rule_fwd_kernel`
- `sglang.srt.layers.attention.fla.index.prepare_chunk_indices`
- `sglang.srt.layers.attention.fla.l2norm.l2norm_fwd`
- `sglang.srt.layers.attention.fla.op.exp`
- `sglang.srt.layers.attention.fla.op.log`
- `sglang.srt.layers.attention.fla.utils.check_shared_mem`
- `sglang.srt.layers.attention.fla.utils.is_intel`
- `sglang.srt.hardware_backend.xpu.kernels.fla.chunk_delta_h.chunk_gated_delta_rule_fwd_h`
