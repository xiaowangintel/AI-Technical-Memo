# l2norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/l2norm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 10-18 — imports and setup
```python
import os

import torch

from vllm.triton_utils import tl, triton

BT_LIST = [8, 16, 32, 64, 128]

USE_DEFAULT_FLA_NORM = int(os.getenv("USE_DEFAULT_FLA_NORM", "0"))
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `int`, `os.getenv`. It writes or updates `BT_LIST`, `USE_DEFAULT_FLA_NORM`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `int`, `os.getenv`。 它会写入或更新 `BT_LIST`, `USE_DEFAULT_FLA_NORM`。

### Lines 21-47 — function `l2norm_fwd_kernel1`
```python
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=num_warps) for num_warps in [1, 2, 4, 8, 16, 32]
    ],
    key=["D"],
)
@triton.jit
def l2norm_fwd_kernel1(
    x,
    y,
    D,
    BD: tl.constexpr,
    eps,
):
    i_t = tl.program_id(0)
    x += i_t * D
    y += i_t * D
    # Compute mean and variance
    cols = tl.arange(0, BD)
    mask = cols < D
    b_x = tl.load(x + cols, mask=mask, other=0.0).to(tl.float32)
    b_var = tl.sum(b_x * b_x, axis=0)
    b_rstd = 1 / tl.sqrt(b_var + eps)
    # tl.store(Rstd + i_t, rstd)
    # Normalize and apply linear transformation
    b_y = b_x * b_rstd
    tl.store(y + cols, b_y, mask=mask)
```
**EN:** This function defines `l2norm_fwd_kernel1`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `x`, `y`, `D`, `BD`, `eps`. Key calls include `triton.autotune`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.sum`, `tl.store`. It writes or updates `i_t`, `x`, `y`, `cols`, `mask`, `b_x`.
**CN:** 该函数定义 `l2norm_fwd_kernel1`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `x`, `y`, `D`, `BD`, `eps`。 关键调用包括 `triton.autotune`, `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.sum`, `tl.store`。 它会写入或更新 `i_t`, `x`, `y`, `cols`, `mask`, `b_x`。

### Lines 50-75 — function `l2norm_fwd_kernel`
```python
@triton.autotune(
    configs=[
        triton.Config({"BT": BT}, num_warps=num_warps)
        for num_warps in [1, 2, 4, 8, 16]
        for BT in BT_LIST
    ],
    key=["D"],
)
@triton.jit(do_not_specialize=["NB"])
def l2norm_fwd_kernel(
    x,
    y,
    eps,
    NB,
    T,
    D: tl.constexpr,
    BT: tl.constexpr,
    BD: tl.constexpr,
):
    i_t = tl.program_id(0)
    p_x = tl.make_block_ptr(x, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0))
    b_x = tl.load(p_x, boundary_check=(0, 1)).to(tl.float32)
    b_var = tl.sum(b_x * b_x, axis=1)
    b_y = b_x / tl.sqrt(b_var + eps)[:, None]
    p_y = tl.make_block_ptr(y, (T, D), (D, 1), (i_t * BT, 0), (BT, BD), (1, 0))
    tl.store(p_y, b_y.to(p_y.dtype.element_ty), boundary_check=(0, 1))
```
**EN:** This function defines `l2norm_fwd_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `x`, `y`, `eps`, `NB`, `T`, `D`. Key calls include `triton.autotune`, `triton.jit`, `tl.program_id`, `tl.make_block_ptr`, `tl.load.to`, `tl.sum`. It writes or updates `i_t`, `p_x`, `b_x`, `b_var`, `b_y`, `p_y`.
**CN:** 该函数定义 `l2norm_fwd_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `x`, `y`, `eps`, `NB`, `T`, `D`。 关键调用包括 `triton.autotune`, `triton.jit`, `tl.program_id`, `tl.make_block_ptr`, `tl.load.to`, `tl.sum`。 它会写入或更新 `i_t`, `p_x`, `b_x`, `b_var`, `b_y`, `p_y`。

### Lines 78-92 — function `l2norm_fwd_kernel2`
```python
@triton.jit
def l2norm_fwd_kernel2(
    X, Y, eps, M, N: tl.constexpr, BD: tl.constexpr, MBLOCK: tl.constexpr
):
    xoffset = tl.program_id(0) * MBLOCK
    row_idx = xoffset + tl.arange(0, MBLOCK)[:, None]
    xmask = row_idx < M
    rindex = tl.arange(0, BD)[None, :]
    cmask = rindex < N
    mask = xmask & cmask
    xs = tl.load(X + (rindex + N * row_idx), mask, other=0.0).to(tl.float32)
    square = tl.broadcast_to(xs * xs, [MBLOCK, BD])
    square_sum = tl.sum(tl.where(xmask, square, 0), 1)[:, None]
    rsqrt = tl.rsqrt(square_sum + eps)
    tl.store(Y + (rindex + N * row_idx), xs * rsqrt, mask)
```
**EN:** This function defines `l2norm_fwd_kernel2`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `X`, `Y`, `eps`, `M`, `N`, `BD`. Key calls include `tl.load.to`, `tl.broadcast_to`, `tl.rsqrt`, `tl.store`, `tl.program_id`, `tl.arange`. It writes or updates `xoffset`, `row_idx`, `xmask`, `rindex`, `cmask`, `mask`.
**CN:** 该函数定义 `l2norm_fwd_kernel2`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `X`, `Y`, `eps`, `M`, `N`, `BD`。 关键调用包括 `tl.load.to`, `tl.broadcast_to`, `tl.rsqrt`, `tl.store`, `tl.program_id`, `tl.arange`。 它会写入或更新 `xoffset`, `row_idx`, `xmask`, `rindex`, `cmask`, `mask`。

### Lines 95-151 — function `l2norm_fwd`
```python
def l2norm_fwd(
    x: torch.Tensor, eps: float = 1e-6, output_dtype: torch.dtype | None = None
):
    x_shape_og = x.shape
    x = x.view(-1, x.shape[-1])
    # allocate output
    if output_dtype is None:
        y = torch.empty_like(x)
    else:
        y = torch.empty_like(x, dtype=output_dtype)
    assert y.stride(-1) == 1
    T, D = x.shape[0], x.shape[-1]
    # rstd = torch.empty((T,), dtype=torch.float32, device=x.device)
    # Less than 64KB per feature: enqueue fused kernel
    MAX_FUSED_SIZE = 65536 // x.element_size()
    BD = min(MAX_FUSED_SIZE, triton.next_power_of_2(D))
    if D > BD:
        raise RuntimeError("This layer doesn't support feature dim >= 64KB.")

    if not USE_DEFAULT_FLA_NORM:
        MBLOCK = 32
        # M, N = x.shape
        l2norm_fwd_kernel2[(triton.cdiv(T, MBLOCK),)](
            x,
            y,
            eps,
            T,
            D,
            BD,
            MBLOCK,
        )
    else:
        if D <= 512:
            NB = triton.cdiv(T, 2048)

            def grid(meta):
                return (triton.cdiv(T, meta["BT"]),)

            l2norm_fwd_kernel[grid](
                x,
                y,
                eps,
                NB=NB,
                T=T,
# ... omitted for brevity ...

    return y.view(x_shape_og)
```
**EN:** This function defines `l2norm_fwd`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `eps`, `output_dtype`. Key calls include `x.view`, `min`, `y.view`, `torch.empty_like`, `y.stride`, `x.element_size`. It writes or updates `x_shape_og`, `x`, `T`, `D`, `MAX_FUSED_SIZE`, `BD`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `l2norm_fwd`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `eps`, `output_dtype`。 关键调用包括 `x.view`, `min`, `y.view`, `torch.empty_like`, `y.stride`, `x.element_size`。 它会写入或更新 `x_shape_og`, `x`, `T`, `D`, `MAX_FUSED_SIZE`, `BD`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `l2norm_fwd_kernel1`, `l2norm_fwd_kernel`, `l2norm_fwd_kernel2`, `l2norm_fwd` / [CN] 核心符号：`l2norm_fwd_kernel1`, `l2norm_fwd_kernel`, `l2norm_fwd_kernel2`, `l2norm_fwd`

## Dependencies / 依赖关系
- **External**: `os`, `torch` / **外部依赖**: `os`, `torch`
- **Internal**: `vllm.triton_utils` / **内部依赖**: `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
