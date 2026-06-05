# fused_gdn_prefill_post_conv.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/fused_gdn_prefill_post_conv.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused post-conv1d preparation for GDN prefill. / 快速线性注意力自定义内核与工具算子；卷积式张量变换与辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 12-16 — imports and setup
```python
from __future__ import annotations

import torch

from vllm.triton_utils import tl, triton
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 19-149 — function `_fused_post_conv_kernel`
```python
@triton.jit
def _fused_post_conv_kernel(
    # ---- inputs ----
    mixed_qkv_ptr,  # [L, qkv_dim] conv'd output (contiguous)
    a_ptr,  # [L, HV]
    b_ptr,  # [L, HV]
    # ---- params ----
    A_log_ptr,  # [HV]
    dt_bias_ptr,  # [HV]
    # ---- outputs ----
    q_ptr,  # [L, H, K] contiguous
    k_ptr,  # [L, H, K] contiguous
    v_ptr,  # [L, HV, V] contiguous
    g_ptr,  # [L, HV] float32
    beta_ptr,  # [L, HV] float32
    # ---- strides ----
    stride_x_tok,  # qkv_dim
    stride_a_tok,  # HV
    stride_b_tok,  # HV
    stride_q_tok,  # H * K
    stride_k_tok,  # H * K
    stride_v_tok,  # HV * V
    # ---- dims ----
    L,
    H: tl.constexpr,
    HV: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    APPLY_L2NORM: tl.constexpr,
    L2NORM_EPS: tl.constexpr,
    OUTPUT_G_EXP: tl.constexpr,
    SOFTPLUS_THRESHOLD: tl.constexpr,
    BLOCK_T: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
):
    """Single fused kernel for post-conv1d preparation.

    Grid: (ceil(L, BLOCK_T), H + HV)
      - program_id(1) in [0, H):    Q/K head processing + l2norm
      - program_id(1) in [H, H+HV): V head processing + gating
    """
    i_tb = tl.program_id(0)
    i_head = tl.program_id(1)

    HK: tl.constexpr = H * K

    offs_t = i_tb * BLOCK_T + tl.arange(0, BLOCK_T)  # [BLOCK_T]
    mask_t = offs_t < L

    if i_head < H:
        # ============ Q/K head processing ============
        i_h = i_head
        offs_k = tl.arange(0, BK)  # [BK]
        mask_k = offs_k < K
        mask_2d = mask_t[:, None] & mask_k[None, :]  # [BLOCK_T, BK]
# ... omitted for brevity ...
        tl.store(g_ptr + gb_offsets, g_vals, mask=mask_t)
        tl.store(beta_ptr + gb_offsets, beta_vals, mask=mask_t)
```
**EN:** This function defines `_fused_post_conv_kernel`. Single fused kernel for post-conv1d preparation. The main inputs are `mixed_qkv_ptr`, `a_ptr`, `b_ptr`, `A_log_ptr`, `dt_bias_ptr`, `q_ptr`. Key calls include `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, `tl.load`, `tl.where`. It writes or updates `i_tb`, `i_head`, `HK`, `offs_t`, `mask_t`, `i_h`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_fused_post_conv_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `mixed_qkv_ptr`, `a_ptr`, `b_ptr`, `A_log_ptr`, `dt_bias_ptr`, `q_ptr`。 关键调用包括 `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, `tl.load`, `tl.where`。 它会写入或更新 `i_tb`, `i_head`, `HK`, `offs_t`, `mask_t`, `i_h`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 152-248 — function `fused_post_conv_prep`
```python
def fused_post_conv_prep(
    conv_output: torch.Tensor,  # [L, qkv_dim] conv'd mixed_qkv
    a: torch.Tensor,  # [L, HV]
    b: torch.Tensor,  # [L, HV]
    A_log: torch.Tensor,  # [HV]
    dt_bias: torch.Tensor,  # [HV]
    num_k_heads: int,
    head_k_dim: int,
    head_v_dim: int,
    apply_l2norm: bool = True,
    output_g_exp: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Fused post-conv1d prep: split + l2norm + gating in one kernel.

    Args:
        conv_output: [L, qkv_dim] contiguous conv'd mixed_qkv
        a: [L, HV] gating input
        b: [L, HV] gating input
        A_log: [HV] log decay parameter
        dt_bias: [HV] dt bias parameter
        num_k_heads: number of K heads (H)
        head_k_dim: dimension per K head (K)
        head_v_dim: dimension per V head (V)
        apply_l2norm: whether to L2-normalize q and k
        output_g_exp: if True, output exp(g) instead of g (for FlashInfer)

    Returns:
        q: [L, H, K] contiguous, optionally l2-normalized
        k: [L, H, K] contiguous, optionally l2-normalized
        v: [L, HV, V] contiguous
        g: [L, HV] float32
        beta: [L, HV] float32
    """
    L = conv_output.shape[0]
    qkv_dim = conv_output.shape[1]
    H = num_k_heads
    K = head_k_dim
    V = head_v_dim
    HV = A_log.shape[0]
    dtype = conv_output.dtype
    device = conv_output.device

    assert qkv_dim == 2 * H * K + HV * V, (
        f"qkv_dim={qkv_dim} != 2*H*K + HV*V = {2 * H * K + HV * V}"
# ... omitted for brevity ...

    return q, k, v, g, beta
```
**EN:** This function defines `fused_post_conv_prep`. Fused post-conv1d prep: split + l2norm + gating in one kernel. The main inputs are `conv_output`, `a`, `b`, `A_log`, `dt_bias`, `num_k_heads`. Key calls include `torch.empty`, `triton.next_power_of_2`, `_fused_post_conv_kernel`, `triton.cdiv`, `conv_output.stride`, `a.stride`. It writes or updates `L`, `qkv_dim`, `H`, `K`, `V`, `HV`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_post_conv_prep`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `conv_output`, `a`, `b`, `A_log`, `dt_bias`, `num_k_heads`。 关键调用包括 `torch.empty`, `triton.next_power_of_2`, `_fused_post_conv_kernel`, `triton.cdiv`, `conv_output.stride`, `a.stride`。 它会写入或更新 `L`, `qkv_dim`, `H`, `K`, `V`, `HV`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] Convolution-style tensor transforms and helpers / [CN] 卷积式张量变换与辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `_fused_post_conv_kernel`, `fused_post_conv_prep` / [CN] 核心符号：`_fused_post_conv_kernel`, `fused_post_conv_prep`

## Dependencies / 依赖关系
- **External**: `__future__`, `torch` / **外部依赖**: `__future__`, `torch`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
