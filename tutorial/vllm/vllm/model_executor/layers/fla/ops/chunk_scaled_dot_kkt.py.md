# chunk_scaled_dot_kkt.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/chunk_scaled_dot_kkt.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-17 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices
from .op import exp
from .utils import FLA_CHUNK_SIZE
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 20-99 — function `chunk_scaled_dot_kkt_fwd_kernel`
```python
@triton.heuristics(
    {
        "USE_G": lambda args: args["g"] is not None,
        "IS_VARLEN": lambda args: args["cu_seqlens"] is not None,
    }
)
@triton.autotune(
    configs=[
        triton.Config({"BK": BK}, num_warps=num_warps, num_stages=num_stages)
        for BK in [32, 64, 128]
        for num_warps in [2, 4, 8]
        for num_stages in [2, 3, 4]
    ],
    key=["H", "K", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_scaled_dot_kkt_fwd_kernel(
    k,
    beta,
    g,
    A,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    Hg: tl.constexpr,
    K: tl.constexpr,
    BT: tl.constexpr,
    BK: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    USE_G: tl.constexpr,
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
    o_t = i_t * BT + tl.arange(0, BT)
    m_t = o_t < T

    p_beta = tl.make_block_ptr(
        beta + bos * H + i_h, (T,), (H,), (i_t * BT,), (BT,), (0,)
    )
    b_beta = tl.load(p_beta, boundary_check=(0,))

    b_A = tl.zeros([BT, BT], dtype=tl.float32)
    for i_k in range(tl.cdiv(K, BK)):
# ... omitted for brevity ...
    )
    tl.store(p_A, b_A.to(p_A.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `chunk_scaled_dot_kkt_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `k`, `beta`, `g`, `A`, `cu_seqlens`, `chunk_indices`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.zeros`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `o_t`, `m_t`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `chunk_scaled_dot_kkt_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `k`, `beta`, `g`, `A`, `cu_seqlens`, `chunk_indices`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.make_block_ptr`, `tl.load`, `tl.zeros`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `o_t`, `m_t`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 102-158 — function `chunk_scaled_dot_kkt_fwd`
```python
def chunk_scaled_dot_kkt_fwd(
    k: torch.Tensor,
    g: torch.Tensor | None = None,
    beta: torch.Tensor | None = None,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    chunk_size: int = FLA_CHUNK_SIZE,
    output_dtype: torch.dtype = torch.float32,
) -> torch.Tensor:
    r"""
    Compute beta * K * K^T.

    Args:
        k (torch.Tensor):
            The key tensor of shape `[B, T, H, K]`.
        beta (torch.Tensor):
            The beta tensor of shape `[B, T, H]`.
        g (torch.Tensor):
            The cumulative sum of the gate tensor of shape `[B, T, H]`. Default: `None`.
        cu_seqlens (torch.Tensor):
            The cumulative sequence lengths of the input tensor.
            Default: None
        chunk_indices (torch.Tensor):
            Pre-computed chunk indices. If None and cu_seqlens is provided,
            computed internally. Default: None
        chunk_size (int):
            The chunk size. Default: 64.
        output_dtype (torch.dtype):
            The dtype of the output tensor. Default: `torch.float32`

    Returns:
        beta * K * K^T of shape `[B, T, H, BT]` where `BT` is the chunk size.
    """
    # This kernel is slightly different from fla to support Q/K with different head numbers.
    # In fla, Q/K always have the same head number, so Hg is always equal to H.
    B, T, Hg, K = k.shape
    H = beta.shape[-1]
    BT = chunk_size
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)

    A = torch.empty(B, T, H, BT, device=k.device, dtype=output_dtype)
    chunk_scaled_dot_kkt_fwd_kernel[(NT, B * H)](
# ... omitted for brevity ...
    )
    return A
```
**EN:** This function defines `chunk_scaled_dot_kkt_fwd`. Compute beta * K * K^T. The main inputs are `k`, `g`, `beta`, `cu_seqlens`, `chunk_indices`, `chunk_size`. Key calls include `torch.empty`, `chunk_scaled_dot_kkt_fwd_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`. It writes or updates `B`, `T`, `Hg`, `K`, `H`, `BT`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_scaled_dot_kkt_fwd`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `k`, `g`, `beta`, `cu_seqlens`, `chunk_indices`, `chunk_size`。 关键调用包括 `torch.empty`, `chunk_scaled_dot_kkt_fwd_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`。 它会写入或更新 `B`, `T`, `Hg`, `K`, `H`, `BT`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `chunk_scaled_dot_kkt_fwd_kernel`, `chunk_scaled_dot_kkt_fwd` / [CN] 核心符号：`chunk_scaled_dot_kkt_fwd_kernel`, `chunk_scaled_dot_kkt_fwd`

## Dependencies / 依赖关系
- **External**: `torch`, `index`, `op`, `utils` / **外部依赖**: `torch`, `index`, `op`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
