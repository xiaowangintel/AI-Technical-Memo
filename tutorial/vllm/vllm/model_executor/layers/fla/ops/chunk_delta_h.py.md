# chunk_delta_h.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/chunk_delta_h.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-19 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices, prepare_chunk_offsets
from .op import exp
from .utils import FLA_CHUNK_SIZE, use_cuda_graph

NUM_WARPS = [2, 4, 8, 16]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `NUM_WARPS`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `NUM_WARPS`。

### Lines 22-298 — function `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`
```python
@triton.heuristics(
    {
        "USE_G": lambda args: args["g"] is not None,
        "USE_GK": lambda args: args["gk"] is not None,
        "USE_INITIAL_STATE": lambda args: args["h0"] is not None,
        "STORE_FINAL_STATE": lambda args: args["ht"] is not None,
        "SAVE_NEW_VALUE": lambda args: args["v_new"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BV": BV}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [2, 4]
        for num_stages in [2, 3, 4]
        for BV in [32, 64]
    ],
    key=["H", "K", "V", "BT"],
    use_cuda_graph=use_cuda_graph,
)
@triton.jit(do_not_specialize=["T"])
def chunk_gated_delta_rule_fwd_kernel_h_blockdim64(
    k,
    v,
    w,
    v_new,
    g,
    gk,
    h,
    h0,
    ht,
    cu_seqlens,
    chunk_offsets,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BV: tl.constexpr,
    USE_G: tl.constexpr,
    USE_GK: tl.constexpr,
    USE_INITIAL_STATE: tl.constexpr,
    STORE_FINAL_STATE: tl.constexpr,
    SAVE_NEW_VALUE: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_v, i_nh = tl.program_id(0), tl.program_id(1)
    i_n, i_h = i_nh // H, i_nh % H
    if IS_VARLEN:
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
            tl.load(cu_seqlens + i_n + 1).to(tl.int32),
        )
        T = eos - bos
        NT = tl.cdiv(T, BT)
# ... omitted for brevity ...
            )
            tl.store(p_ht, b_h4.to(p_ht.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `k`, `v`, `w`, `v_new`, `g`, `gk`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.zeros`, `to`, `range`. It writes or updates `i_v`, `i_nh`, `i_n`, `i_h`, `b_h1`, `h`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `k`, `v`, `w`, `v_new`, `g`, `gk`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.zeros`, `to`, `range`。 它会写入或更新 `i_v`, `i_nh`, `i_n`, `i_h`, `b_h1`, `h`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 301-361 — function `chunk_gated_delta_rule_fwd_h`
```python
def chunk_gated_delta_rule_fwd_h(
    k: torch.Tensor,
    w: torch.Tensor,
    u: torch.Tensor,
    g: torch.Tensor | None = None,
    gk: torch.Tensor | None = None,
    initial_state: torch.Tensor | None = None,
    output_final_state: bool = False,
    chunk_size: int = FLA_CHUNK_SIZE,
    save_new_value: bool = True,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    chunk_offsets: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    # This kernel is slightly different from fla to support Q/K with different head numbers.
    # In fla, Q/K always have the same head number, so Hg is always equal to H.
    B, T, Hg, K, V = *k.shape, u.shape[-1]
    H = u.shape[-2]
    BT = chunk_size

    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, chunk_size)
    # N: the actual number of sequences in the batch with either equal or variable lengths
    if cu_seqlens is None:
        N, NT, chunk_offsets = B, triton.cdiv(T, BT), None
    else:
        N, NT = len(cu_seqlens) - 1, len(chunk_indices)
        if chunk_offsets is None:
            chunk_offsets = prepare_chunk_offsets(cu_seqlens, BT)
    assert K <= 256, "current kernel does not support head dimension larger than 256."

    h = k.new_empty(B, NT, H, V, K)
    final_state = (
        k.new_empty(N, H, V, K, dtype=torch.float32) if output_final_state else None
    )

    v_new = torch.empty_like(u) if save_new_value else None

    def grid(meta):
        return (triton.cdiv(V, meta["BV"]), N * H)

    chunk_gated_delta_rule_fwd_kernel_h_blockdim64[grid](
        k=k,
        v=u,
# ... omitted for brevity ...
    )
    return h, v_new, final_state
```
**EN:** This function defines `chunk_gated_delta_rule_fwd_h`. It provides one of the file's main runtime building blocks. The main inputs are `k`, `w`, `u`, `g`, `gk`, `initial_state`. Key calls include `k.new_empty`, `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`, `prepare_chunk_indices`, `torch.empty_like`, `triton.cdiv`, `len`. It writes or updates `B`, `T`, `Hg`, `K`, `V`, `H`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_gated_delta_rule_fwd_h`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `k`, `w`, `u`, `g`, `gk`, `initial_state`。 关键调用包括 `k.new_empty`, `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`, `prepare_chunk_indices`, `torch.empty_like`, `triton.cdiv`, `len`。 它会写入或更新 `B`, `T`, `Hg`, `K`, `V`, `H`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `chunk_gated_delta_rule_fwd_kernel_h_blockdim64`, `chunk_gated_delta_rule_fwd_h` / [CN] 核心符号：`chunk_gated_delta_rule_fwd_kernel_h_blockdim64`, `chunk_gated_delta_rule_fwd_h`

## Dependencies / 依赖关系
- **External**: `torch`, `index`, `op`, `utils` / **外部依赖**: `torch`, `index`, `op`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
