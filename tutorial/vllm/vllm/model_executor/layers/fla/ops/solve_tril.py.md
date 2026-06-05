# solve_tril.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/solve_tril.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-22 — imports and setup
```python
import os

import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices
from .op import make_tensor_descriptor
from .utils import input_guard, is_amd, is_tma_supported

FLA_TRIL_PRECISION = os.environ.get("FLA_TRIL_PRECISION", "ieee")
ALLOWED_TRIL_PRECISIONS = ["ieee", "tf32"] if is_amd else ["ieee", "tf32", "tf32x3"]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `os.environ.get`. It writes or updates `FLA_TRIL_PRECISION`, `ALLOWED_TRIL_PRECISIONS`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `os.environ.get`。 它会写入或更新 `FLA_TRIL_PRECISION`, `ALLOWED_TRIL_PRECISIONS`。

### Lines 28-100 — function `solve_tril_16x16_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [1, 2, 4, 8]
        for num_stages in [2, 3, 4, 5]
    ],
    key=["BT"],
)
@triton.jit(do_not_specialize=["T"])
def solve_tril_16x16_kernel(
    A,
    Ai,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    BT: tl.constexpr,
    USE_TMA: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    DOT_PRECISION: tl.constexpr,
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
    o_i = tl.arange(0, 16)
    m_A = o_i[:, None] > o_i[None, :]
    m_I = o_i[:, None] == o_i[None, :]

    A = A + (bos * H + i_h) * BT
    Ai = Ai + (bos * H + i_h) * 16

    offset = (i_t * 16) % BT
    if not USE_TMA:
        p_A = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * 16, offset), (16, 16), (1, 0)
        )
        # [16, 16]
        b_A = tl.load(p_A, boundary_check=(0, 1)).to(tl.float32)
    else:
        desc = make_tensor_descriptor(A, [T, BT], [H * BT, 1], [16, 16])
        desc_o = make_tensor_descriptor(Ai, [T, 16], [H * 16, 1], [16, 16])
        b_A = desc.load([i_t * 16, offset]).to(tl.float32)
    b_A = -tl.where(m_A, b_A, 0)

# ... omitted for brevity ...
    else:
        desc_o.store([i_t * 16, 0], b_A.to(desc_o.dtype, fp_downcast_rounding="rtne"))
```
**EN:** This function defines `solve_tril_16x16_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `solve_tril_16x16_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 103-225 — function `merge_16x16_to_32x32_inverse_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [1, 2, 4, 8]
        for num_stages in [2, 3, 4, 5]
    ],
    key=["H", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def merge_16x16_to_32x32_inverse_kernel(
    A,
    Ai,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    BT: tl.constexpr,
    USE_TMA: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    DOT_PRECISION: tl.constexpr,
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

    o_i = tl.arange(0, 16)
    m_A = o_i[:, None] > o_i[None, :]
    m_I = o_i[:, None] == o_i[None, :]
    A += (bos * H + i_h) * BT
    Ai += (bos * H + i_h) * BT

    if not USE_TMA:
        p_A_11 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT, 0), (16, 16), (1, 0)
        )
        p_A_22 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT + 16, 16), (16, 16), (1, 0)
        )
        b_Ai_11 = tl.load(p_A_11, boundary_check=(0, 1)).to(tl.float32)
        b_Ai_22 = tl.load(p_A_22, boundary_check=(0, 1)).to(tl.float32)
    else:
        desc = make_tensor_descriptor(A, [T, BT], [H * BT, 1], [16, 16])
        desc_o = make_tensor_descriptor(Ai, [T, BT], [H * BT, 1], [16, 16])
        b_Ai_11 = desc.load([i_t * BT + 0, 0]).to(tl.float32)
# ... omitted for brevity ...
            [i_t * BT + 16, 16], b_Ai_22.to(desc_o.dtype, fp_downcast_rounding="rtne")
        )
```
**EN:** This function defines `merge_16x16_to_32x32_inverse_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `merge_16x16_to_32x32_inverse_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 228-503 — function `merge_16x16_to_64x64_inverse_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps, num_stages=num_stages)
        for num_warps in [2, 4, 8]
        for num_stages in [2, 3, 4, 5]
    ],
    key=["H", "BT", "IS_VARLEN"],
)
@triton.jit(do_not_specialize=["T"])
def merge_16x16_to_64x64_inverse_kernel(
    A,
    Ai,
    cu_seqlens,
    chunk_indices,
    T,
    H: tl.constexpr,
    BT: tl.constexpr,
    USE_TMA: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    DOT_PRECISION: tl.constexpr,
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

    o_i = tl.arange(0, 16)
    m_A = o_i[:, None] > o_i[None, :]
    m_I = o_i[:, None] == o_i[None, :]
    A += (bos * H + i_h) * BT
    Ai += (bos * H + i_h) * BT

    if not USE_TMA:
        p_A_11 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT, 0), (16, 16), (1, 0)
        )
        p_A_22 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT + 16, 16), (16, 16), (1, 0)
        )
        p_A_33 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT + 32, 32), (16, 16), (1, 0)
        )
        p_A_44 = tl.make_block_ptr(
            A, (T, BT), (H * BT, 1), (i_t * BT + 48, 48), (16, 16), (1, 0)
        )
# ... omitted for brevity ...
            [i_t * BT + 48, 32], b_Ai_43.to(desc_o.dtype, fp_downcast_rounding="rtne")
        )
```
**EN:** This function defines `merge_16x16_to_64x64_inverse_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `merge_16x16_to_64x64_inverse_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `Ai`, `cu_seqlens`, `chunk_indices`, `T`, `H`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `range`, `tl.program_id`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`, `m_A`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 506-558 — function `solve_tril`
```python
@input_guard
def solve_tril(
    A: torch.Tensor,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    output_dtype: torch.dtype = torch.float,
) -> torch.Tensor:
    """
    Compute the inverse of the matrix I + A
    A should be strictly lower triangular, i.e., A.triu() == 0.

    Args:
        A (torch.Tensor):
            [B, T, H, BT], where BT should only be 16, 32, or 64.
        cu_seqlens (torch.Tensor):
            The cumulative sequence lengths of the input tensor. Default: `None`.
        chunk_indices (torch.Tensor):
            Pre-computed chunk indices. Default: `None`.
        output_dtype (torch.dtype):
            The dtype of the output tensor. Default: `torch.float`.
            If `None`, the output dtype will be the same as the input dtype.

    Returns:
        (I + A)^-1 with the same shape as A
    """
    assert A.shape[-1] in [16, 32, 64]
    output_dtype = A.dtype if output_dtype is None else output_dtype

    B, T, H, BT = A.shape
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, BT)
    NT = len(chunk_indices) if cu_seqlens is not None else triton.cdiv(T, BT)

    Ai = torch.zeros_like(A, dtype=output_dtype)
    if BT == 16:
        merge_fn = solve_tril_16x16_kernel
    elif BT == 32:
        merge_fn = merge_16x16_to_32x32_inverse_kernel
    elif BT == 64:
        merge_fn = merge_16x16_to_64x64_inverse_kernel

    merge_fn[NT, B * H](
        A=A,
        Ai=Ai,
# ... omitted for brevity ...
    )
    return Ai
```
**EN:** This function defines `solve_tril`. Compute the inverse of the matrix I + A A should be strictly lower triangular, i.e., A.triu() == 0. The main inputs are `A`, `cu_seqlens`, `chunk_indices`, `output_dtype`. Key calls include `torch.zeros_like`, `merge_fn`, `prepare_chunk_indices`, `len`, `triton.cdiv`. It writes or updates `output_dtype`, `B`, `T`, `H`, `BT`, `NT`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `solve_tril`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `A`, `cu_seqlens`, `chunk_indices`, `output_dtype`。 关键调用包括 `torch.zeros_like`, `merge_fn`, `prepare_chunk_indices`, `len`, `triton.cdiv`。 它会写入或更新 `output_dtype`, `B`, `T`, `H`, `BT`, `NT`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `solve_tril_16x16_kernel`, `merge_16x16_to_32x32_inverse_kernel`, `merge_16x16_to_64x64_inverse_kernel`, `solve_tril` / [CN] 核心符号：`solve_tril_16x16_kernel`, `merge_16x16_to_32x32_inverse_kernel`, `merge_16x16_to_64x64_inverse_kernel`, `solve_tril`

## Dependencies / 依赖关系
- **External**: `os`, `torch`, `index`, `op`, `utils` / **外部依赖**: `os`, `torch`, `index`, `op`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
