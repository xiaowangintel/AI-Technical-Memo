# fused_sigmoid_gating.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/fused_sigmoid_gating.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 10-12 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 15-178 — function `fused_sigmoid_gating_delta_rule_update_kernel`
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
def fused_sigmoid_gating_delta_rule_update_kernel(
    A_log,
    a,
    b,
    dt_bias,
    beta,
    threshold,
    q,
    k,
    v,
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
# ... omitted for brevity ...
        p_b += HV
        p_a += HV
```
**EN:** This function defines `fused_sigmoid_gating_delta_rule_update_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold`. Key calls include `triton.heuristics`, `triton.jit`, `tl.zeros`, `range`, `tl.program_id`, `tl.arange`. It writes or updates `i_k`, `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `fused_sigmoid_gating_delta_rule_update_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold`。 关键调用包括 `triton.heuristics`, `triton.jit`, `tl.zeros`, `range`, `tl.program_id`, `tl.arange`。 它会写入或更新 `i_k`, `i_v`, `i_nh`, `i_n`, `i_hv`, `i_h`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 181-279 — function `fused_sigmoid_gating_delta_rule_update`
```python
def fused_sigmoid_gating_delta_rule_update(
    A_log: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    dt_bias: torch.Tensor,
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    beta: float = 1.0,
    threshold: float = 20.0,
    scale: float = None,
    initial_state: torch.Tensor = None,
    inplace_final_state: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    ssm_state_indices: torch.Tensor | None = None,
    num_accepted_tokens: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = False,
    is_kda: bool = False,
):
    """
    Fused triton implementation of sigmoid gating delta rule update.
    This function uses a single fused kernel that combines both sigmoid gating
    computation and the recurrent delta rule update for better performance.
    """
    B, T, H, K, V = *k.shape, v.shape[-1]
    HV = v.shape[2]
    N = B if cu_seqlens is None else len(cu_seqlens) - 1
    BK, BV = triton.next_power_of_2(K), min(triton.next_power_of_2(V), 32)
    NK, NV = triton.cdiv(K, BK), triton.cdiv(V, BV)
    assert NK == 1, "NK > 1 is not supported yet"
    num_stages = 3
    num_warps = 4

    if cu_seqlens is not None and q.shape[0] != 1:
        raise ValueError(
            f"The batch size is expected to be 1 rather than {q.shape[0]}"
            f" when using `cu_seqlens`. Please flatten variable-length"
            f" inputs before processing."
        )
    if scale is None:
        scale = k.shape[-1] ** -0.5
    else:
        assert scale > 0, "scale must be positive"

# ... omitted for brevity ...
    o = o.squeeze(0)
    return o, final_state
```
**EN:** This function defines `fused_sigmoid_gating_delta_rule_update`. Fused triton implementation of sigmoid gating delta rule update. The main inputs are `A_log`, `a`, `b`, `dt_bias`, `q`, `k`. Key calls include `q.new_empty`, `initial_state.stride`, `final_state.stride`, `fused_sigmoid_gating_delta_rule_update_kernel`, `o.squeeze`, `triton.next_power_of_2`. It writes or updates `B`, `T`, `H`, `K`, `V`, `HV`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_sigmoid_gating_delta_rule_update`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `A_log`, `a`, `b`, `dt_bias`, `q`, `k`。 关键调用包括 `q.new_empty`, `initial_state.stride`, `final_state.stride`, `fused_sigmoid_gating_delta_rule_update_kernel`, `o.squeeze`, `triton.next_power_of_2`。 它会写入或更新 `B`, `T`, `H`, `K`, `V`, `HV`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `fused_sigmoid_gating_delta_rule_update_kernel`, `fused_sigmoid_gating_delta_rule_update` / [CN] 核心符号：`fused_sigmoid_gating_delta_rule_update_kernel`, `fused_sigmoid_gating_delta_rule_update`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
