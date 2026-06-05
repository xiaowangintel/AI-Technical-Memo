# fused_recurrent.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/fused_recurrent.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-15 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .op import exp
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 18-175 — function `fused_recurrent_gated_delta_rule_fwd_kernel`
```python
@triton.heuristics(
    {
        "USE_INITIAL_STATE": lambda args: args["h0"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
        "IS_CONTINUOUS_BATCHING": lambda args: args["ssm_state_indices"] is not None,
        "IS_SPEC_DECODING": lambda args: args["num_accepted_tokens"] is not None,
    }
)
@triton.jit(do_not_specialize=["N", "T"])
def fused_recurrent_gated_delta_rule_fwd_kernel(
    q,
    k,
    v,
    g,
    beta,
    o,
    h0,
    ht,
    cu_seqlens,
    ssm_state_indices,
    num_accepted_tokens,
    scale,
    N: tl.int64,  # num of sequences
    T: tl.int64,  # num of tokens
    B: tl.constexpr,
    H: tl.constexpr,
    HV: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    stride_init_state_token: tl.constexpr,
    stride_final_state_token: tl.constexpr,
    stride_indices_seq: tl.constexpr,
    stride_indices_tok: tl.constexpr,
    USE_INITIAL_STATE: tl.constexpr,  # whether to use initial state
    INPLACE_FINAL_STATE: tl.constexpr,  # whether to store final state inplace
    IS_BETA_HEADWISE: tl.constexpr,  # whether beta is headwise vector or scalar,
    USE_QK_L2NORM_IN_KERNEL: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    IS_CONTINUOUS_BATCHING: tl.constexpr,
    IS_SPEC_DECODING: tl.constexpr,
    IS_KDA: tl.constexpr,
):
    i_k, i_v, i_nh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_n, i_hv = i_nh // HV, i_nh % HV
    i_h = i_hv // (HV // H)
    if IS_VARLEN:
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int64),
            tl.load(cu_seqlens + i_n + 1).to(tl.int64),
        )
        all = T
        T = eos - bos
    else:
        bos, eos = i_n * T, i_n * T + T
# ... omitted for brevity ...
            p_gk += HV * K
        p_beta += HV * (V if IS_BETA_HEADWISE else 1)
```
**EN:** This function defines `fused_recurrent_gated_delta_rule_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `q`, `k`, `v`, `g`, `beta`, `o`. Key calls include `triton.heuristics`, `triton.jit`, `tl.zeros`, `range`, `tl.program_id`, `tl.arange`. It writes or updates `i_k`, `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_gated_delta_rule_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `o`。 关键调用包括 `triton.heuristics`, `triton.jit`, `tl.zeros`, `range`, `tl.program_id`, `tl.arange`。 它会写入或更新 `i_k`, `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 178-252 — function `fused_recurrent_gated_delta_rule_fwd`
```python
def fused_recurrent_gated_delta_rule_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    inplace_final_state: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    ssm_state_indices: torch.Tensor | None = None,
    num_accepted_tokens: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    B, T, H, K, V = *k.shape, v.shape[-1]
    HV = v.shape[2]
    N = B if cu_seqlens is None else len(cu_seqlens) - 1
    BK, BV = triton.next_power_of_2(K), min(triton.next_power_of_2(V), 32)
    NK, NV = triton.cdiv(K, BK), triton.cdiv(V, BV)
    assert NK == 1, "NK > 1 is not supported yet"
    num_stages = 3
    num_warps = 1

    o = q.new_empty(NK, *v.shape)
    if inplace_final_state:
        final_state = initial_state
    else:
        final_state = q.new_empty(T, HV, V, K, dtype=initial_state.dtype)

    stride_init_state_token = initial_state.stride(0)
    stride_final_state_token = final_state.stride(0)

    if ssm_state_indices is None:
        stride_indices_seq, stride_indices_tok = 1, 1
    elif ssm_state_indices.ndim == 1:
        stride_indices_seq, stride_indices_tok = ssm_state_indices.stride(0), 1
    else:
        stride_indices_seq, stride_indices_tok = ssm_state_indices.stride()

    grid = (NK, NV, N * HV)
    fused_recurrent_gated_delta_rule_fwd_kernel[grid](
        q=q,
        k=k,
        v=v,
# ... omitted for brevity ...
    o = o.squeeze(0)
    return o, final_state
```
**EN:** This function defines `fused_recurrent_gated_delta_rule_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `q.new_empty`, `initial_state.stride`, `final_state.stride`, `fused_recurrent_gated_delta_rule_fwd_kernel`, `o.squeeze`, `triton.next_power_of_2`. It writes or updates `B`, `T`, `H`, `K`, `V`, `HV`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_gated_delta_rule_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `q.new_empty`, `initial_state.stride`, `final_state.stride`, `fused_recurrent_gated_delta_rule_fwd_kernel`, `o.squeeze`, `triton.next_power_of_2`。 它会写入或更新 `B`, `T`, `H`, `K`, `V`, `HV`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 255-336 — function `fused_recurrent_gated_delta_rule_packed_decode_kernel`
```python
@triton.jit
def fused_recurrent_gated_delta_rule_packed_decode_kernel(
    mixed_qkv,
    a,
    b,
    A_log,
    dt_bias,
    o,
    h0,
    ht,
    ssm_state_indices,
    scale,
    stride_mixed_qkv_tok: tl.constexpr,
    stride_a_tok: tl.constexpr,
    stride_b_tok: tl.constexpr,
    stride_init_state_token: tl.constexpr,
    stride_final_state_token: tl.constexpr,
    stride_indices_seq: tl.constexpr,
    H: tl.constexpr,
    HV: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    SOFTPLUS_THRESHOLD: tl.constexpr,
    USE_QK_L2NORM_IN_KERNEL: tl.constexpr,
):
    i_v, i_nh = tl.program_id(0), tl.program_id(1)
    i_n, i_hv = i_nh // HV, i_nh % HV
    i_h = i_hv // (HV // H)

    o_k = tl.arange(0, BK)
    o_v = i_v * BV + tl.arange(0, BV)
    mask_k = o_k < K
    mask_v = o_v < V
    mask_h = mask_v[:, None] & mask_k[None, :]

    state_idx = tl.load(ssm_state_indices + i_n * stride_indices_seq).to(tl.int64)
    p_o = o + (i_n * HV + i_hv) * V + o_v

    # Skip if state index is invalid (NULL_BLOCK_ID=0)
    if state_idx <= 0:
        zero = tl.zeros([BV], dtype=tl.float32).to(p_o.dtype.element_ty)
        tl.store(p_o, zero, mask=mask_v)
        return

    p_h0 = h0 + state_idx * stride_init_state_token
    p_h0 = p_h0 + i_hv * V * K + o_v[:, None] * K + o_k[None, :]
    b_h = tl.load(p_h0, mask=mask_h, other=0).to(tl.float32)

    p_mixed = mixed_qkv + i_n * stride_mixed_qkv_tok
    q_off = i_h * K + o_k
    k_off = (H * K) + i_h * K + o_k
    v_off = (2 * H * K) + i_hv * V + o_v
    b_q = tl.load(p_mixed + q_off, mask=mask_k, other=0).to(tl.float32)
    b_k = tl.load(p_mixed + k_off, mask=mask_k, other=0).to(tl.float32)
# ... omitted for brevity ...
    p_ht = p_ht + i_hv * V * K + o_v[:, None] * K + o_k[None, :]
    tl.store(p_ht, b_h.to(p_ht.dtype.element_ty), mask=mask_h)
```
**EN:** This function defines `fused_recurrent_gated_delta_rule_packed_decode_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `mixed_qkv`, `a`, `b`, `A_log`, `dt_bias`, `o`. Key calls include `tl.arange`, `tl.load.to`, `tl.where`, `tl.sigmoid.to.to`, `exp`, `tl.sum`. It writes or updates `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`, `o_k`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_gated_delta_rule_packed_decode_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `mixed_qkv`, `a`, `b`, `A_log`, `dt_bias`, `o`。 关键调用包括 `tl.arange`, `tl.load.to`, `tl.where`, `tl.sigmoid.to.to`, `exp`, `tl.sum`。 它会写入或更新 `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`, `o_k`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 339-478 — function `fused_recurrent_gated_delta_rule_packed_decode`
```python
def fused_recurrent_gated_delta_rule_packed_decode(
    mixed_qkv: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    A_log: torch.Tensor,
    dt_bias: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    out: torch.Tensor,
    ssm_state_indices: torch.Tensor,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    if mixed_qkv.ndim != 2:
        raise ValueError(
            f"`mixed_qkv` must be a 2D tensor (got ndim={mixed_qkv.ndim})."
        )
    if mixed_qkv.stride(-1) != 1:
        raise ValueError("`mixed_qkv` must be contiguous in the last dim.")
    if a.ndim != 2 or b.ndim != 2:
        raise ValueError(
            f"`a` and `b` must be 2D tensors (got a.ndim={a.ndim}, b.ndim={b.ndim})."
        )
    if a.stride(-1) != 1 or b.stride(-1) != 1:
        raise ValueError("`a`/`b` must be contiguous in the last dim.")
    if A_log.ndim != 1 or dt_bias.ndim != 1:
        raise ValueError("`A_log`/`dt_bias` must be 1D tensors.")
    if A_log.stride(0) != 1 or dt_bias.stride(0) != 1:
        raise ValueError("`A_log`/`dt_bias` must be contiguous.")
    if ssm_state_indices.ndim != 1:
        raise ValueError(
            f"`ssm_state_indices` must be 1D for packed decode (got ndim={ssm_state_indices.ndim})."
        )
    if not out.is_contiguous():
        raise ValueError("`out` must be contiguous.")

    dev = mixed_qkv.device
    if (
        a.device != dev
        or b.device != dev
        or A_log.device != dev
        or dt_bias.device != dev
        or initial_state.device != dev
        or out.device != dev
        or ssm_state_indices.device != dev
# ... omitted for brevity ...
    )
    return out, initial_state
```
**EN:** This function defines `fused_recurrent_gated_delta_rule_packed_decode`. It provides one of the file's main runtime building blocks. The main inputs are `mixed_qkv`, `a`, `b`, `A_log`, `dt_bias`, `scale`. Key calls include `triton.next_power_of_2`, `min`, `mixed_qkv.stride`, `a.stride`, `b.stride`, `initial_state.stride`. It writes or updates `dev`, `B`, `HV`, `V`, `K`, `qkv_dim`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_gated_delta_rule_packed_decode`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `mixed_qkv`, `a`, `b`, `A_log`, `dt_bias`, `scale`。 关键调用包括 `triton.next_power_of_2`, `min`, `mixed_qkv.stride`, `a.stride`, `b.stride`, `initial_state.stride`。 它会写入或更新 `dev`, `B`, `HV`, `V`, `K`, `qkv_dim`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 481-513 — class `FusedRecurrentFunction`
```python
class FusedRecurrentFunction(torch.autograd.Function):
    @staticmethod
    def forward(
        ctx,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        scale: float,
        initial_state: torch.Tensor,
        inplace_final_state: bool = True,
        cu_seqlens: torch.Tensor | None = None,
        ssm_state_indices: torch.Tensor | None = None,
# ... omitted for brevity ...

        return o, final_state
```
**EN:** This class defines `FusedRecurrentFunction`. It inherits from `torch.autograd.Function`. It provides one of the file's main runtime building blocks. Important methods include `forward`. Key calls include `fused_recurrent_gated_delta_rule_fwd`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`, `beta.contiguous`. It writes or updates `o`, `final_state`.
**CN:** 该类定义了 `FusedRecurrentFunction`。 它继承自 `torch.autograd.Function`。 它是该文件中的一个主要运行时构件。 重要方法包括 `forward`。 关键调用包括 `fused_recurrent_gated_delta_rule_fwd`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`, `beta.contiguous`。 它会写入或更新 `o`, `final_state`。

### Lines 482-513 — method `FusedRecurrentFunction.forward`
```python
    @staticmethod
    def forward(
        ctx,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        scale: float,
        initial_state: torch.Tensor,
        inplace_final_state: bool = True,
        cu_seqlens: torch.Tensor | None = None,
        ssm_state_indices: torch.Tensor | None = None,
        num_accepted_tokens: torch.Tensor | None = None,
        use_qk_l2norm_in_kernel: bool = False,
    ):
        o, final_state = fused_recurrent_gated_delta_rule_fwd(
            q=q.contiguous(),
            k=k.contiguous(),
            v=v.contiguous(),
            g=g.contiguous(),
            beta=beta.contiguous(),
            scale=scale,
            initial_state=initial_state,
            inplace_final_state=inplace_final_state,
            cu_seqlens=cu_seqlens,
            ssm_state_indices=ssm_state_indices,
            num_accepted_tokens=num_accepted_tokens,
            use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
        )

        return o, final_state
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `ctx`, `q`, `k`, `v`, `g`, `beta`. Key calls include `fused_recurrent_gated_delta_rule_fwd`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`, `beta.contiguous`. It writes or updates `o`, `final_state`.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `ctx`, `q`, `k`, `v`, `g`, `beta`。 关键调用包括 `fused_recurrent_gated_delta_rule_fwd`, `q.contiguous`, `k.contiguous`, `v.contiguous`, `g.contiguous`, `beta.contiguous`。 它会写入或更新 `o`, `final_state`。

### Lines 516-619 — function `fused_recurrent_gated_delta_rule`
```python
def fused_recurrent_gated_delta_rule(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor = None,
    scale: float = None,
    initial_state: torch.Tensor = None,
    inplace_final_state: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    ssm_state_indices: torch.Tensor | None = None,
    num_accepted_tokens: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    r"""
    Args:
        q (torch.Tensor):
            queries of shape `[B, T, H, K]`.
        k (torch.Tensor):
            keys of shape `[B, T, H, K]`.
        v (torch.Tensor):
            values of shape `[B, T, HV, V]`.
            GVA is applied if `HV > H`.
        g (torch.Tensor):
            g (decays) of shape `[B, T, HV]`.
        beta (torch.Tensor):
            betas of shape `[B, T, HV]`.
        scale (Optional[int]):
            Scale factor for the RetNet attention scores.
            If not provided, it will default to `1 / sqrt(K)`. Default: `None`.
        initial_state (Optional[torch.Tensor]):
            Initial state of shape `[N, HV, V, K]` for `N` input sequences.
            For equal-length input sequences, `N` equals the batch size `B`.
            Default: `None`.
        inplace_final_state: bool:
            Whether to store the final state in-place to save memory.
            Default: `True`.
        cu_seqlens (torch.Tensor):
            Cumulative sequence lengths of shape `[N+1]` used for variable-length training,
            consistent with the FlashAttention API.
        ssm_state_indices (Optional[torch.Tensor]):
            Indices to map the input sequences to the initial/final states.
        num_accepted_tokens (Optional[torch.Tensor]):
            Number of accepted tokens for each sequence during decoding.
# ... omitted for brevity ...
    )
    return o, final_state
```
**EN:** This function defines `fused_recurrent_gated_delta_rule`. Args: q (torch.Tensor): queries of shape `[B, T, H, K]`. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `FusedRecurrentFunction.apply`, `ValueError`, `torch.ones_like`. It writes or updates `o`, `final_state`, `scale`, `beta`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_recurrent_gated_delta_rule`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `FusedRecurrentFunction.apply`, `ValueError`, `torch.ones_like`。 它会写入或更新 `o`, `final_state`, `scale`, `beta`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `fused_recurrent_gated_delta_rule_fwd_kernel`, `fused_recurrent_gated_delta_rule_fwd`, `fused_recurrent_gated_delta_rule_packed_decode_kernel`, `fused_recurrent_gated_delta_rule_packed_decode` / [CN] 核心符号：`fused_recurrent_gated_delta_rule_fwd_kernel`, `fused_recurrent_gated_delta_rule_fwd`, `fused_recurrent_gated_delta_rule_packed_decode_kernel`, `fused_recurrent_gated_delta_rule_packed_decode`

## Dependencies / 依赖关系
- **External**: `torch`, `op` / **外部依赖**: `torch`, `op`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
