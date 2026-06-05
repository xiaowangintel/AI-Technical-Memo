# cumsum.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/cumsum.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 11-18 — imports and setup
```python
import torch

from vllm.triton_utils import tl, triton

from .index import prepare_chunk_indices
from .utils import check_shared_mem, input_guard

BS_LIST = [32, 64] if check_shared_mem() else [16, 32]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `check_shared_mem`. It writes or updates `BS_LIST`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `check_shared_mem`。 它会写入或更新 `BS_LIST`。

### Lines 21-71 — function `chunk_local_cumsum_scalar_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[triton.Config({}, num_warps=num_warps) for num_warps in [1, 2, 4, 8]],
    key=["B", "H", "BT", "IS_VARLEN", "REVERSE"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_local_cumsum_scalar_kernel(
    s,
    o,
    cu_seqlens,
    chunk_indices,
    T,
    B: tl.constexpr,
    H: tl.constexpr,
    BT: tl.constexpr,
    REVERSE: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    HEAD_FIRST: tl.constexpr,
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

    if HEAD_FIRST:
        p_s = tl.make_block_ptr(
            s + bos * H + i_h * T, (T,), (1,), (i_t * BT,), (BT,), (0,)
        )
        p_o = tl.make_block_ptr(
            o + bos * H + i_h * T, (T,), (1,), (i_t * BT,), (BT,), (0,)
        )
    else:
        p_s = tl.make_block_ptr(s + bos * H + i_h, (T,), (H,), (i_t * BT,), (BT,), (0,))
        p_o = tl.make_block_ptr(o + bos * H + i_h, (T,), (H,), (i_t * BT,), (BT,), (0,))
    # [BT]
    b_s = tl.load(p_s, boundary_check=(0,)).to(tl.float32)
    b_o = tl.cumsum(b_s, axis=0)
    if REVERSE:
        b_z = tl.sum(b_s, axis=0)
        b_o = -b_o + b_z[None] + b_s
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0,))
```
**EN:** This function defines `chunk_local_cumsum_scalar_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `s`, `o`, `cu_seqlens`, `chunk_indices`, `T`, `B`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.load.to`, `tl.cumsum`, `tl.store`. It writes or updates `i_t`, `i_bh`, `i_b`, `i_h`, `b_s`, `b_o`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_local_cumsum_scalar_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `s`, `o`, `cu_seqlens`, `chunk_indices`, `T`, `B`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.load.to`, `tl.cumsum`, `tl.store`。 它会写入或更新 `i_t`, `i_bh`, `i_b`, `i_h`, `b_s`, `b_o`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 74-157 — function `chunk_local_cumsum_vector_kernel`
```python
@triton.heuristics({"IS_VARLEN": lambda args: args["cu_seqlens"] is not None})
@triton.autotune(
    configs=[
        triton.Config({"BS": BS}, num_warps=num_warps)
        for BS in BS_LIST
        for num_warps in [2, 4, 8]
    ],
    key=["B", "H", "S", "BT", "IS_VARLEN", "REVERSE"],
)
@triton.jit(do_not_specialize=["T"])
def chunk_local_cumsum_vector_kernel(
    s,
    o,
    cu_seqlens,
    chunk_indices,
    T,
    B: tl.constexpr,
    H: tl.constexpr,
    S: tl.constexpr,
    BT: tl.constexpr,
    BS: tl.constexpr,
    REVERSE: tl.constexpr,
    IS_VARLEN: tl.constexpr,
    HEAD_FIRST: tl.constexpr,
):
    i_s, i_t, i_bh = tl.program_id(0), tl.program_id(1), tl.program_id(2)
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

    o_i = tl.arange(0, BT)
    if REVERSE:
        m_s = tl.where(o_i[:, None] <= o_i[None, :], 1.0, 0.0)
    else:
        m_s = tl.where(o_i[:, None] >= o_i[None, :], 1.0, 0.0)

    if HEAD_FIRST:
        p_s = tl.make_block_ptr(
            s + (bos * H + i_h * T) * S,
            (T, S),
            (S, 1),
            (i_t * BT, i_s * BS),
            (BT, BS),
            (1, 0),
        )
        p_o = tl.make_block_ptr(
# ... omitted for brevity ...
    b_o = tl.dot(m_s, b_s, allow_tf32=False)
    tl.store(p_o, b_o.to(p_o.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `chunk_local_cumsum_vector_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `s`, `o`, `cu_seqlens`, `chunk_indices`, `T`, `B`. Key calls include `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.load.to`, `tl.dot`. It writes or updates `i_s`, `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_local_cumsum_vector_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `s`, `o`, `cu_seqlens`, `chunk_indices`, `T`, `B`。 关键调用包括 `triton.heuristics`, `triton.autotune`, `triton.jit`, `tl.arange`, `tl.load.to`, `tl.dot`。 它会写入或更新 `i_s`, `i_t`, `i_bh`, `i_b`, `i_h`, `o_i`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 160-194 — function `chunk_local_cumsum_scalar`
```python
def chunk_local_cumsum_scalar(
    g: torch.Tensor,
    chunk_size: int,
    reverse: bool = False,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    head_first: bool = False,
    output_dtype: torch.dtype | None = torch.float,
) -> torch.Tensor:
    if head_first:
        B, H, T = g.shape
    else:
        B, T, H = g.shape
    assert chunk_size == 2 ** (chunk_size.bit_length() - 1), (
        "chunk_size must be a power of 2"
    )
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, chunk_size)
    BT = chunk_size
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)
    g_org, g = g, torch.empty_like(g, dtype=output_dtype or g.dtype)
    grid = (NT, B * H)
    chunk_local_cumsum_scalar_kernel[grid](
        g_org,
        g,
        cu_seqlens,
        chunk_indices,
        T=T,
        B=B,
        H=H,
        BT=BT,
        HEAD_FIRST=head_first,
        REVERSE=reverse,
    )
    return g
```
**EN:** This function defines `chunk_local_cumsum_scalar`. It provides one of the file's main runtime building blocks. The main inputs are `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`. Key calls include `chunk_local_cumsum_scalar_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `torch.empty_like`, `chunk_size.bit_length`. It writes or updates `BT`, `NT`, `g_org`, `g`, `grid`, `B`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_local_cumsum_scalar`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`。 关键调用包括 `chunk_local_cumsum_scalar_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `torch.empty_like`, `chunk_size.bit_length`。 它会写入或更新 `BT`, `NT`, `g_org`, `g`, `grid`, `B`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 197-239 — function `chunk_local_cumsum_vector`
```python
def chunk_local_cumsum_vector(
    g: torch.Tensor,
    chunk_size: int,
    reverse: bool = False,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    head_first: bool = False,
    output_dtype: torch.dtype | None = torch.float,
) -> torch.Tensor:
    if head_first:
        B, H, T, S = g.shape
    else:
        B, T, H, S = g.shape
    assert chunk_size == 2 ** (chunk_size.bit_length() - 1), (
        "chunk_size must be a power of 2"
    )
    if chunk_indices is None and cu_seqlens is not None:
        chunk_indices = prepare_chunk_indices(cu_seqlens, chunk_size)
    BT = chunk_size
    NT = triton.cdiv(T, BT) if cu_seqlens is None else len(chunk_indices)

    g_org, g = g, torch.empty_like(g, dtype=output_dtype or g.dtype)

    def grid(meta):
        return (triton.cdiv(meta["S"], meta["BS"]), NT, B * H)

    # keep cumulative normalizer in fp32
    # this kernel is equivalent to
    # g = g.view(B, H, NT, BT, -1).cumsum(-2).view(B, H, T, -1)
    chunk_local_cumsum_vector_kernel[grid](
        g_org,
        g,
        cu_seqlens,
        chunk_indices,
        T=T,
        B=B,
        H=H,
        S=S,
        BT=BT,
        HEAD_FIRST=head_first,
        REVERSE=reverse,
    )
    return g
```
**EN:** This function defines `chunk_local_cumsum_vector`. It provides one of the file's main runtime building blocks. The main inputs are `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`. Key calls include `chunk_local_cumsum_vector_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `torch.empty_like`, `chunk_size.bit_length`. It writes or updates `BT`, `NT`, `g_org`, `g`, `B`, `H`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_local_cumsum_vector`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`。 关键调用包括 `chunk_local_cumsum_vector_kernel`, `prepare_chunk_indices`, `triton.cdiv`, `len`, `torch.empty_like`, `chunk_size.bit_length`。 它会写入或更新 `BT`, `NT`, `g_org`, `g`, `B`, `H`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 242-282 — function `chunk_local_cumsum`
```python
@input_guard
def chunk_local_cumsum(
    g: torch.Tensor,
    chunk_size: int,
    reverse: bool = False,
    cu_seqlens: torch.Tensor | None = None,
    chunk_indices: torch.Tensor | None = None,
    head_first: bool = False,
    output_dtype: torch.dtype | None = torch.float,
    **kwargs,
) -> torch.Tensor:
    if cu_seqlens is not None:
        assert g.shape[0] == 1, (
            "Only batch size 1 is supported when cu_seqlens are provided"
        )
    if len(g.shape) == 3:
        return chunk_local_cumsum_scalar(
            g,
            chunk_size,
            reverse,
            cu_seqlens,
            chunk_indices,
            head_first,
            output_dtype,
        )
    elif len(g.shape) == 4:
        return chunk_local_cumsum_vector(
            g,
            chunk_size,
            reverse,
            cu_seqlens,
            chunk_indices,
            head_first,
            output_dtype,
        )
    else:
        raise ValueError(
            f"Unsupported input shape {g.shape}. "
            f"which should be (B, T, H, D) if `head_first=False` "
            f"or (B, H, T, D) otherwise"
        )
```
**EN:** This function defines `chunk_local_cumsum`. It provides one of the file's main runtime building blocks. The main inputs are `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`. Key calls include `len`, `chunk_local_cumsum_scalar`, `chunk_local_cumsum_vector`, `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `chunk_local_cumsum`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `g`, `chunk_size`, `reverse`, `cu_seqlens`, `chunk_indices`, `head_first`。 关键调用包括 `len`, `chunk_local_cumsum_scalar`, `chunk_local_cumsum_vector`, `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `chunk_local_cumsum_scalar_kernel`, `chunk_local_cumsum_vector_kernel`, `chunk_local_cumsum_scalar`, `chunk_local_cumsum_vector` / [CN] 核心符号：`chunk_local_cumsum_scalar_kernel`, `chunk_local_cumsum_vector_kernel`, `chunk_local_cumsum_scalar`, `chunk_local_cumsum_vector`

## Dependencies / 依赖关系
- **External**: `torch`, `index`, `utils` / **外部依赖**: `torch`, `index`, `utils`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
