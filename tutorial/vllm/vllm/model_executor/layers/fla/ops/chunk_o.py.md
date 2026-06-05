# chunk_o.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/chunk_o.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 13-22 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices
from .op import exp
from .utils import FLA_CHUNK_SIZE, check_shared_mem, is_nvidia_hopper

BKV_LIST = [64, 128] if check_shared_mem() else [32, 64]
NUM_WARPS = [2, 4] if is_nvidia_hopper else [2, 4, 8]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `check_shared_mem`. It writes or updates `BKV_LIST`, `NUM_WARPS`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `check_shared_mem`。 它会写入或更新 `BKV_LIST`, `NUM_WARPS`。

### Lines 25-138 — function `chunk_fwd_kernel_o`
```python
@triton.heuristics(
    {
        "USE_G": lambda args: args["g"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BK": BK, "BV": BV}, num_warps=num_warps, num_stages=num_stages)
        for BK in BKV_LIST
        for BV in BKV_LIST
        for num_warps in NUM_WARPS
        for num_stages in [2, 3, 4]
    ],
    key=["H", "K", "V", "BT"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_fwd_kernel_o(
    q,
    k,
    v,
    h,
    g,
    o,
    cu_seqlens,
    chunk_indices,
    scale,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    V: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    BV: tl.constexpr,
    USE_G: tl.constexpr,
    IS_VARLEN: tl.constexpr,
):
    i_v, i_t, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    i_b, i_h = i_bh // H, i_bh % H

    if IS_VARLEN:
        i_tg = i_t
        i_n, i_t = (
            tl.load(chunk_indices + i_t * 2).to(tl.int32),
            tl.load(chunk_indices + i_t * 2 + 1).to(tl.int32),
        )
        bos, eos = (
            tl.load(cu_seqlens + i_n).to(tl.int32),
            tl.load(cu_seqlens + i_n + 1).to(tl.int32),
        )
        T = eos - bos
        NT = tl.cdiv(T, BT)
    else:
        NT = tl.cdiv(T, BT)
        i_tg = i_b * NT + i_t
# ... omitted for brevity ...
    b_o = b_o * scale + tl.dot(b_A.to(b_v.dtype), b_v) * scale
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `chunk_fwd_kernel_o`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `q`, `k`, `v`, `h`, `g`, `o`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.zeros`, `range`, `tl.where`. It writes or updates `i_v`, `i_t`, `i_bh`, `i_b`, `i_h`, `q`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `chunk_fwd_kernel_o`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `q`, `k`, `v`, `h`, `g`, `o`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.zeros`, `range`, `tl.where`。 它会写入或更新 `i_v`, `i_t`, `i_bh`, `i_b`, `i_h`, `q`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 141-190 — function `chunk_fwd_o`
```python
def chunk_fwd_o(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    h: torch.Tensor,
    g: torch.Tensor | None = None,  # cumsum of log decay
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    chunk_size: int = FLA_CHUNK_SIZE,
    core_attn_out: torch.Tensor | None = None,
) -> torch.Tensor:
    B, T, Hg, K, V = *q.shape, v.shape[-1]
    H = v.shape[-2]
    BT = chunk_size
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)
    if scale is None:
        scale = k.shape[-1] ** -0.5

    if core_attn_out is not None:
        assert core_attn_out.numel() >= v.numel(), (
            f"core_attn_out too small: {core_attn_out.numel()} < {v.numel()}"
        )
        o = core_attn_out[: v.numel()].view(*v.shape)
    else:
        o = torch.empty_like(v)

    def grid(meta):
        return (triton.cdiv(V, meta["BV"]), NT, B * H)

    chunk_fwd_kernel_o[grid](
        q,
        k,
        v,
        h,
        g,
        o,
        cu_seqlens,
        chunk_indices,
        scale,
        T=T,
        H=H,
# ... omitted for brevity ...
    )
    return o
```
**EN:** This function defines `chunk_fwd_o`. It provides one of the file's main runtime building blocks. The main inputs are `q`, `k`, `v`, `h`, `g`, `scale`. Key calls include `chunk_fwd_kernel_o`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `core_attn_out.view`, `torch.empty_like`. It writes or updates `B`, `T`, `Hg`, `K`, `V`, `H`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_fwd_o`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `q`, `k`, `v`, `h`, `g`, `scale`。 关键调用包括 `chunk_fwd_kernel_o`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `core_attn_out.view`, `torch.empty_like`。 它会写入或更新 `B`, `T`, `Hg`, `K`, `V`, `H`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `chunk_fwd_kernel_o`, `chunk_fwd_o` / [CN] 核心符号：`chunk_fwd_kernel_o`, `chunk_fwd_o`

## Dependencies / 依赖关系
- **External**: `torch`, `index`, `op`, `utils` / **外部依赖**: `torch`, `index`, `op`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
