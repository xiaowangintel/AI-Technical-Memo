# chunk_fwd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/xpu/kernels/fla/chunk_fwd.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements XPU backend support for kernel-level operators inside the SGLang runtime. / 为 SGLang 运行时提供面向 XPU 后端的内核级算子支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.index import prepare_chunk_indices
from sglang.srt.layers.attention.fla.op import safe_exp
from sglang.srt.layers.attention.fla.utils import (
    autotune_cache_kwargs,
)
from sglang.srt.layers.attention.fla.wy_fast import recompute_w_u_fwd

_MERGE_DOT_PRECISION = tl.constexpr("ieee")
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `torch`, `triton`, `triton.language`, `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`, `sglang.srt.layers.attention.fla.utils`. It also defines symbols such as `_MERGE_DOT_PRECISION` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `torch`, `triton`, `triton.language`, `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`, `sglang.srt.layers.attention.fla.utils`。 同时定义了 `_MERGE_DOT_PRECISION` 等符号，供后续逻辑使用。

### Lines 15-234: Function `chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg` / 函数 `chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg`
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
        for BK in [16, 32, 64]
        for num_warps in [2, 4, 8, 16, 32]
    ],
    key=["H", "Hg", "K", "BC", "BK", "USE_G", "IS_VARLEN"],
    **autotune_cache_kwargs,
)
@triton.jit(do_not_specialize=["T"])
def chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg(
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
# ... omitted for brevity ...
                input_precision=_MERGE_DOT_PRECISION,
            )
            p_Ai_ij = tl.make_block_ptr(
                A, (T, BT), (H * BT, 1), (i_tci, j * BC), (BC, BC), (1, 0)
            )
            tl.store(p_Ai_ij, b_Ai_ij.to(A.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function implements `chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg`. It primarily calls `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.static_range`, `tl.program_id` to complete its work. State updates are written into `i_tc0`, `k`, `A`, `o_i`, `m_d`, `m_I`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg`。 它主要通过调用 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.static_range`, `tl.program_id` 来完成任务。 状态更新主要写入 `i_tc0`, `k`, `A`, `o_i`, `m_d`, `m_I`。 实现中使用了条件分支、迭代逻辑。

### Lines 237-315: Function `chunk_gated_delta_rule_fwd_intra` / 函数 `chunk_gated_delta_rule_fwd_intra`
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
# ... omitted for brevity ...
        A=A,
        g_cumsum=g,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
    )
    return w, u, A
```
**EN:** This function implements `chunk_gated_delta_rule_fwd_intra`. It primarily calls `torch.zeros`, `kernel`, `recompute_w_u_fwd`, `prepare_chunk_indices`, `triton.cdiv`, `len` to complete its work. State updates are written into `H`, `BT`, `BC`, `NT`, `A`, `kernel`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `chunk_gated_delta_rule_fwd_intra`。 它主要通过调用 `torch.zeros`, `kernel`, `recompute_w_u_fwd`, `prepare_chunk_indices`, `triton.cdiv`, `len` 来完成任务。 状态更新主要写入 `H`, `BT`, `BC`, `NT`, `A`, `kernel`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `chunk_gated_delta_rule_fwd_kkt_solve_kernel_low_reg`, `chunk_gated_delta_rule_fwd_intra`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.attention.fla.index`, `sglang.srt.layers.attention.fla.op`, `sglang.srt.layers.attention.fla.utils`, `sglang.srt.layers.attention.fla.wy_fast`
- **External / 外部依赖**: `torch`, `triton`, `triton.language`
- **Standard library / 标准库**: None / 无
