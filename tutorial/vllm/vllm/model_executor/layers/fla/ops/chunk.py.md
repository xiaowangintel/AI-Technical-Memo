# chunk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/chunk.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-20 — imports and setup
```python
import torch

from .chunk_delta_h import chunk_gated_delta_rule_fwd_h
from .chunk_o import chunk_fwd_o
from .chunk_scaled_dot_kkt import chunk_scaled_dot_kkt_fwd
from .cumsum import chunk_local_cumsum
from .l2norm import l2norm_fwd
from .solve_tril import solve_tril
from .utils import FLA_CHUNK_SIZE, SUPPRESS_LEVEL, input_guard
from .wy_fast import recompute_w_u_fwd
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 23-86 — function `chunk_gated_delta_rule_fwd`
```python
def chunk_gated_delta_rule_fwd(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float,
    initial_state: torch.Tensor,
    output_final_state: bool,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    chunk_offsets: torch.Tensor | None = None,
    core_attn_out: torch.Tensor | None = None,
):
    g = chunk_local_cumsum(
        g, chunk_size=FLA_CHUNK_SIZE, cu_seqlens=cu_seqlens, chunk_indices=chunk_indices
    )
    # obtain WY representation. u is actually the new v.
    A = chunk_scaled_dot_kkt_fwd(
        k=k,
        beta=beta,
        g=g,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        output_dtype=torch.float32,
    )
    A = solve_tril(
        A=A, cu_seqlens=cu_seqlens, chunk_indices=chunk_indices, output_dtype=k.dtype
    )
    w, u = recompute_w_u_fwd(
        k=k,
        v=v,
        beta=beta,
        A=A,
        g_cumsum=g,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )
    h, v_new, final_state = chunk_gated_delta_rule_fwd_h(
        k=k,
        w=w,
        u=u,
        g=g,
        initial_state=initial_state,
# ... omitted for brevity ...
    elif SUPPRESS_LEVEL >= 3:
        return g, o, A, final_state, w, h, v_new
```
**EN:** This function defines `chunk_gated_delta_rule_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `chunk_local_cumsum`, `chunk_scaled_dot_kkt_fwd`, `solve_tril`, `recompute_w_u_fwd`, `chunk_gated_delta_rule_fwd_h`, `chunk_fwd_o`. It writes or updates `g`, `A`, `w`, `u`, `h`, `v_new`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_gated_delta_rule_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `chunk_local_cumsum`, `chunk_scaled_dot_kkt_fwd`, `solve_tril`, `recompute_w_u_fwd`, `chunk_gated_delta_rule_fwd_h`, `chunk_fwd_o`。 它会写入或更新 `g`, `A`, `w`, `u`, `h`, `v_new`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 89-134 — class `ChunkGatedDeltaRuleFunction`
```python
class ChunkGatedDeltaRuleFunction(torch.autograd.Function):
    @staticmethod
    @input_guard
    @torch.amp.custom_fwd(device_type="cuda")
    def forward(
        ctx,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        scale: float,
        initial_state: torch.Tensor,
        output_final_state: bool,
# ... omitted for brevity ...
            assert q.dtype == o.dtype, "Incompatible dtype for inplace computation"
        return o.to(q.dtype), final_state
```
**EN:** This class defines `ChunkGatedDeltaRuleFunction`. It inherits from `torch.autograd.Function`. It provides one of the file's main runtime building blocks. Important methods include `forward`. Key calls include `torch.amp.custom_fwd`, `chunk_gated_delta_rule_fwd`, `l2norm_fwd`, `o.to`, `torch.is_grad_enabled`. It writes or updates `g`, `o`, `A`, `final_state`, `w`, `h`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `ChunkGatedDeltaRuleFunction`。 它继承自 `torch.autograd.Function`。 它是该文件中的一个主要运行时构件。 重要方法包括 `forward`。 关键调用包括 `torch.amp.custom_fwd`, `chunk_gated_delta_rule_fwd`, `l2norm_fwd`, `o.to`, `torch.is_grad_enabled`。 它会写入或更新 `g`, `o`, `A`, `final_state`, `w`, `h`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 90-134 — method `ChunkGatedDeltaRuleFunction.forward`
```python
    @staticmethod
    @input_guard
    @torch.amp.custom_fwd(device_type="cuda")
    def forward(
        ctx,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        scale: float,
        initial_state: torch.Tensor,
        output_final_state: bool,
        cu_seqlens: torch.Tensor | None = None,
        chunk_indices: torch.Tensor | None = None,
        chunk_offsets: torch.Tensor | None = None,
        use_qk_l2norm_in_kernel: bool = False,
        core_attn_out: torch.Tensor | None = None,
    ):
        if use_qk_l2norm_in_kernel:
            q = l2norm_fwd(q)
            k = l2norm_fwd(k)

        g, o, A, final_state, w, h, v_new = chunk_gated_delta_rule_fwd(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            scale=scale,
            initial_state=initial_state,
            output_final_state=output_final_state,
            cu_seqlens=cu_seqlens,
            chunk_indices=chunk_indices,
            chunk_offsets=chunk_offsets,
            core_attn_out=core_attn_out,
        )
        ctx.scale = scale
        ctx.use_qk_l2norm_in_kernel = use_qk_l2norm_in_kernel
        if core_attn_out is not None:
            assert not torch.is_grad_enabled(), (
                "core_attn_out buffer reuse is only supported for inference"
            )
            assert q.dtype == o.dtype, "Incompatible dtype for inplace computation"
        return o.to(q.dtype), final_state
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `ctx`, `q`, `k`, `v`, `g`, `beta`. Key calls include `torch.amp.custom_fwd`, `chunk_gated_delta_rule_fwd`, `l2norm_fwd`, `o.to`, `torch.is_grad_enabled`. It writes or updates `g`, `o`, `A`, `final_state`, `w`, `h`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `ctx`, `q`, `k`, `v`, `g`, `beta`。 关键调用包括 `torch.amp.custom_fwd`, `chunk_gated_delta_rule_fwd`, `l2norm_fwd`, `o.to`, `torch.is_grad_enabled`。 它会写入或更新 `g`, `o`, `A`, `final_state`, `w`, `h`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 137-245 — function `chunk_gated_delta_rule`
```python
@torch.compiler.disable
def chunk_gated_delta_rule(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    scale: float = None,
    initial_state: torch.Tensor = None,
    output_final_state: bool = False,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    chunk_offsets: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
    core_attn_out: torch.Tensor | None = None,
):
    r"""
    Args:
        q (torch.Tensor):
            Queries of shape `[B, T, H, K]`.
        k (torch.Tensor):
            Keys of shape `[B, T, H, K]`.
        v (torch.Tensor):
            Values of shape `[B, T, H, V]`.
        g (torch.Tensor):
            (forget) Gating tensor (in log space!) of shape `[B, T, H]`.
        beta (torch.Tensor):
            Betas of shape `[B, T, H]`.
        scale (Optional[int]):
            Scale factor for the RetNet attention scores.
            If not provided, it will default to `1 / sqrt(K)`. Default: `None`.
        initial_state (Optional[torch.Tensor]):
            Initial state of shape `[N, H, V, K]` for `N` input sequences.
            For equal-length input sequences, `N` equals the batch size `B`.
            Default: `None`.
        output_final_state (Optional[bool]):
            Whether to output the final state of shape `[N, H, V, K]`. Default: `False`.
        cu_seqlens (torch.Tensor):
            Cumulative sequence lengths of shape `[N+1]` used for variable-length training,
            consistent with the FlashAttention API.
    Returns:
        o (torch.Tensor):
            Outputs of shape `[B, T, H, V]`.
        final_state (torch.Tensor):
# ... omitted for brevity ...
    )
    return o, final_state
```
**EN:** This function defines `chunk_gated_delta_rule`. Args: q (torch.Tensor): Queries of shape `[B, T, H, K]`. The main inputs are `q`, `k`, `v`, `g`, `beta`, `scale`. Key calls include `ChunkGatedDeltaRuleFunction.apply`, `len`, `ValueError`. It writes or updates `o`, `final_state`, `scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_gated_delta_rule`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `k`, `v`, `g`, `beta`, `scale`。 关键调用包括 `ChunkGatedDeltaRuleFunction.apply`, `len`, `ValueError`。 它会写入或更新 `o`, `final_state`, `scale`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `chunk_gated_delta_rule_fwd`, `ChunkGatedDeltaRuleFunction`, `chunk_gated_delta_rule` / [CN] 核心符号：`chunk_gated_delta_rule_fwd`, `ChunkGatedDeltaRuleFunction`, `chunk_gated_delta_rule`

## Dependencies / 依赖关系
- **External**: `torch`, `chunk_delta_h`, `chunk_o`, `chunk_scaled_dot_kkt`, `cumsum`, `l2norm`, `solve_tril`, `utils`, `wy_fast` / **外部依赖**: `torch`, `chunk_delta_h`, `chunk_o`, `chunk_scaled_dot_kkt`, `cumsum`, `l2norm`, `solve_tril`, `utils`, `wy_fast`
- **Internal**: none / **内部依赖**: 无
