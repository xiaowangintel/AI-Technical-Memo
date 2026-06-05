# wy_fast.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/wy_fast.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 12-16 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 19-116 — function `recompute_w_u_fwd_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["H", "K", "V", "BT", "BK", "BV", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def recompute_w_u_fwd_kernel(
    k,
    v,
    beta,
    w,
    u,
    A,
    g,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_t, i_bh = tl.program_id(0), tl.program_id(1)
    i_b, i_h = i_bh // H, i_bh % H
    if IS_VARLEN:
        i_n, i_t = (
            tl.load(chunk_indices + i_t * 2).to(tl.int32),
            tl.load(chunk_indices + i_t * 2 + 1).to(tl.int32),
        )
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
            tl.load(cu_seqlens + i_n + 1).to(tl.int32),
        )
        T = eos - bos
    else:
        bos, eos = i_b * T, i_b * T + T
    p_beta = tl.make_block_ptr(
        beta + bos * H + i_h, (T,), (H,), (i_t * BT,), (BT,), (0,)
    )
    p_g = tl.make_block_ptr(g + (bos * H + i_h), (T,), (H,), (i_t * BT,), (BT,), (0,))
    p_A = tl.make_block_ptr(
        A + (bos * H + i_h) * BT, (T, BT), (H * BT, 1), (i_t * BT, 0), (BT, BT), (1, 0)
    )
    b_beta = tl.load(p_beta, boundary_check=(0,))
    b_A = tl.load(p_A, boundary_check=(0, 1))
    b_g = tl.exp(tl.load(p_g, boundary_check=(0,)))

    for i_v in range(tl.cdiv(V, BV)):
# ... omitted for brevity ...
        b_w = tl.dot(b_A, b_kb)
        tl.store(p_w, b_w.to(p_w.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `recompute_w_u_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `k`, `v`, `beta`, `w`, `u`, `A`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.exp`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `p_beta`, `p_g`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `recompute_w_u_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `k`, `v`, `beta`, `w`, `u`, `A`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.exp`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `p_beta`, `p_g`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 119-158 — function `recompute_w_u_fwd`
```python
def recompute_w_u_fwd(
    k: torch.Tensor,
    v: torch.Tensor,
    beta: torch.Tensor,
    g_cumsum: torch.Tensor,
    A: torch.Tensor,
    cu_seqlens: torch.Tensor | None,
    chunk_indices: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    B, T, Hg, K, V = *k.shape, v.shape[-1]
    H = v.shape[-2]
    BT = A.shape[-1]

    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)
    BK = 64
    BV = 64
    u = torch.empty_like(v)
    w = k.new_empty(B, T, H, K)
    recompute_w_u_fwd_kernel[(NT, B * H)](
        k=k,
        v=v,
        beta=beta,
        w=w,
        u=u,
        A=A,
        g=g_cumsum,
        cu_seqlens=cu_seqlens,
        chunk_indices=chunk_indices,
        T=T,
        H=H,
        Hg=Hg,
        K=K,
        V=V,
        BT=BT,
        BK=BK,
        BV=BV,
    )
    return w, u
```
**EN:** This function defines `recompute_w_u_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `k`, `v`, `beta`, `g_cumsum`, `A`, `cu_seqlens`. Key calls include `torch.empty_like`, `k.new_empty`, `recompute_w_u_fwd_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`. It writes or updates `B`, `T`, `Hg`, `K`, `V`, `H`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `recompute_w_u_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `k`, `v`, `beta`, `g_cumsum`, `A`, `cu_seqlens`。 关键调用包括 `torch.empty_like`, `k.new_empty`, `recompute_w_u_fwd_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`。 它会写入或更新 `B`, `T`, `Hg`, `K`, `V`, `H`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `recompute_w_u_fwd_kernel`, `recompute_w_u_fwd` / [CN] 核心符号：`recompute_w_u_fwd_kernel`, `recompute_w_u_fwd`

## Dependencies / 依赖关系
- **External**: `torch`, `index` / **外部依赖**: `torch`, `index`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
