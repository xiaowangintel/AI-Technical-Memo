# batch_invariant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/batch_invariant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: batch-invariant execution helpers / 批次无关的执行辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-15 — imports and setup
```python
import math
import os
from collections.abc import Callable
from typing import Any

import torch

import vllm.envs as envs
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.mem_utils import get_max_shared_memory_bytes
from vllm.utils.platform_utils import num_compute_units
from vllm.utils.torch_utils import is_torch_equal_or_newer
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 18-28 — function `_matmul_launch_metadata`
```python
def _matmul_launch_metadata(
    grid: Callable[..., Any], kernel: Any, args: dict[str, Any]
) -> dict[str, Any]:
    ret = {}
    m, n, k = args["M"], args["N"], args["K"]
    ret["name"] = f"{kernel.name} [M={m}, N={n}, K={k}]"

    bytes_per_elem = args["c_ptr"].element_size()
    ret[f"flops{bytes_per_elem * 8}"] = 2.0 * m * n * k
    ret["bytes"] = bytes_per_elem * (m * k + n * k + m * n)
    return ret
```
**EN:** This function defines `_matmul_launch_metadata`. It provides one of the file's main runtime building blocks. The main inputs are `grid`, `kernel`, `args`. Key calls include `args.element_size`. It writes or updates `ret`, `m`, `n`, `k`, `bytes_per_elem`.
**CN:** 该函数定义 `_matmul_launch_metadata`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `grid`, `kernel`, `args`。 关键调用包括 `args.element_size`。 它会写入或更新 `ret`, `m`, `n`, `k`, `bytes_per_elem`。

### Lines 31-38 — function `_compute_pid`
```python
@triton.jit
def _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M):
    group_id = tile_id // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (tile_id % group_size_m)
    pid_n = (tile_id % num_pid_in_group) // group_size_m
    return pid_m, pid_n
```
**EN:** This function defines `_compute_pid`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `tile_id`, `num_pid_in_group`, `num_pid_m`, `GROUP_SIZE_M`. Key calls include `min`. It writes or updates `group_id`, `first_pid_m`, `group_size_m`, `pid_m`, `pid_n`.
**CN:** 该函数定义 `_compute_pid`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `tile_id`, `num_pid_in_group`, `num_pid_m`, `GROUP_SIZE_M`。 关键调用包括 `min`。 它会写入或更新 `group_id`, `first_pid_m`, `group_size_m`, `pid_m`, `pid_n`。

### Lines 41-129 — function `matmul_kernel_persistent`
```python
@triton.jit(launch_metadata=_matmul_launch_metadata)
def matmul_kernel_persistent(
    a_ptr,
    b_ptr,
    c_ptr,  #
    bias_ptr,
    M,
    N,
    K,  #
    stride_am,
    stride_ak,
    stride_bk,
    stride_bn,
    stride_cm,
    stride_cn,
    BLOCK_SIZE_M: tl.constexpr,  #
    BLOCK_SIZE_N: tl.constexpr,  #
    BLOCK_SIZE_K: tl.constexpr,  #
    GROUP_SIZE_M: tl.constexpr,  #
    NUM_SMS: tl.constexpr,  #
    A_LARGE: tl.constexpr,
    B_LARGE: tl.constexpr,
    C_LARGE: tl.constexpr,
    HAS_BIAS: tl.constexpr,
):
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n

    tile_id_c = start_pid - NUM_SMS

    offs_k_for_mask = tl.arange(0, BLOCK_SIZE_K)
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    for tile_id in tl.range(start_pid, num_tiles, NUM_SMS, flatten=True):
        pid_m, pid_n = _compute_pid(tile_id, num_pid_in_group, num_pid_m, GROUP_SIZE_M)
        start_m = pid_m * BLOCK_SIZE_M
        start_n = pid_n * BLOCK_SIZE_N
        offs_am = start_m + tl.arange(0, BLOCK_SIZE_M)
        offs_bn = start_n + tl.arange(0, BLOCK_SIZE_N)
        if A_LARGE:
            offs_am = offs_am.to(tl.int64)
        if B_LARGE:
            offs_bn = offs_bn.to(tl.int64)
        offs_am = tl.where(offs_am < M, offs_am, 0)
        offs_bn = tl.where(offs_bn < N, offs_bn, 0)
        offs_am = tl.max_contiguous(tl.multiple_of(offs_am, BLOCK_SIZE_M), BLOCK_SIZE_M)
        offs_bn = tl.max_contiguous(tl.multiple_of(offs_bn, BLOCK_SIZE_N), BLOCK_SIZE_N)

        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            if A_LARGE or B_LARGE:
                offs_k = ki * BLOCK_SIZE_K + tl.arange(0, BLOCK_SIZE_K).to(tl.int64)
            else:
# ... omitted for brevity ...
        c = accumulator.to(c_ptr.dtype.element_ty)
        tl.store(c_ptrs, c, mask=c_mask)
```
**EN:** This function defines `matmul_kernel_persistent`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `bias_ptr`, `M`, `N`. Key calls include `triton.jit`, `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.range`, `_compute_pid`. It writes or updates `start_pid`, `num_pid_m`, `num_pid_n`, `k_tiles`, `num_tiles`, `tile_id_c`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `matmul_kernel_persistent`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `bias_ptr`, `M`, `N`。 关键调用包括 `triton.jit`, `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.range`, `_compute_pid`。 它会写入或更新 `start_pid`, `num_pid_m`, `num_pid_n`, `k_tiles`, `num_tiles`, `tile_id_c`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 132-205 — function `matmul_persistent`
```python
def matmul_persistent(
    a: torch.Tensor, b: torch.Tensor, bias: torch.Tensor | None = None
):
    # Check constraints.
    assert a.shape[1] == b.shape[0], "Incompatible dimensions"
    assert a.dtype == b.dtype, "Incompatible dtypes"
    assert bias is None or bias.dim() == 1, (
        "Currently assuming bias is 1D, let Horace know if you run into this"
    )
    NUM_SMS = num_compute_units(a.device.index)
    M, K = a.shape
    K, N = b.shape
    dtype = a.dtype
    # Allocates output.
    c = torch.empty((M, N), device=a.device, dtype=dtype)

    # 1D launch kernel where each block gets its own program.
    def grid(META):
        return (
            min(
                NUM_SMS,
                triton.cdiv(M, META["BLOCK_SIZE_M"])
                * triton.cdiv(N, META["BLOCK_SIZE_N"]),
            ),
        )

    configs = {
        torch.bfloat16: {
            "BLOCK_SIZE_M": 128,
            "BLOCK_SIZE_N": 128,
            "BLOCK_SIZE_K": 64,
            "GROUP_SIZE_M": 8,
            "num_stages": 3,
            "num_warps": 8,
        },
        torch.float16: {
            "BLOCK_SIZE_M": 128,
            "BLOCK_SIZE_N": _fp16_block_size_n,
            "BLOCK_SIZE_K": 64,
            "GROUP_SIZE_M": 8,
            "num_stages": 3,
            "num_warps": 8,
        },
        torch.float32: {
# ... omitted for brevity ...
    )
    return c
```
**EN:** This function defines `matmul_persistent`. It provides one of the file's main runtime building blocks. The main inputs are `a`, `b`, `bias`. Key calls include `num_compute_units`, `torch.empty`, `matmul_kernel_persistent`, `a.stride`, `b.stride`, `c.stride`. It writes or updates `NUM_SMS`, `M`, `K`, `N`, `dtype`, `c`.
**CN:** 该函数定义 `matmul_persistent`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `a`, `b`, `bias`。 关键调用包括 `num_compute_units`, `torch.empty`, `matmul_kernel_persistent`, `a.stride`, `b.stride`, `c.stride`。 它会写入或更新 `NUM_SMS`, `M`, `K`, `N`, `dtype`, `c`。

### Lines 208-338 — function `bmm_kernel`
```python
@triton.jit
def bmm_kernel(
    a_ptr,  # (*, ) pointer to A, (B, M, K)
    b_ptr,  # (*, ) pointer to B, (B, K, N)
    c_ptr,  # (*, ) pointer to C, (B, M, N)
    B,  # int, batch size
    M,  # int, output rows
    N,  # int, output cols
    K,  # int, reduction dim
    stride_ab,
    stride_am,
    stride_ak,
    stride_bb,
    stride_bk,
    stride_bn,
    stride_cb,
    stride_cm,
    stride_cn,
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    A_LARGE: tl.constexpr,
    B_LARGE: tl.constexpr,
    C_LARGE: tl.constexpr,
):
    """Batched GEMM: (B, M, K) x (B, K, N) -> (B, M, N)

    Each program computes one (batch_idx, tile_m, tile_n) tile, accumulating
    along K in a fixed order to preserve batch invariance.
    """
    pid_b = tl.program_id(0)
    pid = tl.program_id(1)

    if pid_b >= B:
        return

    # number of tiles along M / N
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)

    pid_m = pid // num_pid_n
    pid_n = pid % num_pid_n

    if pid_m >= num_pid_m or pid_n >= num_pid_n:
        return

    # offs_m / offs_n: raw global row/col indices for this tile
    offs_m = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
    offs_n = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    # masks for valid logical rows/cols within (M, N)
    mask_m = offs_m < M  # [BLOCK_SIZE_M]
    mask_n = offs_n < N  # [BLOCK_SIZE_N]

    if A_LARGE or B_LARGE or C_LARGE:
        offs_m = offs_m.to(tl.int64)
        offs_n = offs_n.to(tl.int64)
# ... omitted for brevity ...
    c = accumulator.to(c_ptr.dtype.element_ty)
    tl.store(c_ptrs, c, mask=c_mask)
```
**EN:** This function defines `bmm_kernel`. Batched GEMM: (B, M, K) x (B, K, N) -> (B, M, N) Each program computes one (batch_idx, tile_m, tile_n) tile, accumulating along K in a fixed order to preserve batch invariance. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `B`, `M`, `N`. Key calls include `tl.program_id`, `tl.cdiv`, `tl.where`, `tl.max_contiguous`, `tl.zeros`, `tl.arange`. It writes or updates `pid_b`, `pid`, `num_pid_m`, `num_pid_n`, `pid_m`, `pid_n`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `bmm_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `B`, `M`, `N`。 关键调用包括 `tl.program_id`, `tl.cdiv`, `tl.where`, `tl.max_contiguous`, `tl.zeros`, `tl.arange`。 它会写入或更新 `pid_b`, `pid`, `num_pid_m`, `num_pid_n`, `pid_m`, `pid_n`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 341-401 — function `_log_softmax_kernel`
```python
@triton.jit
def _log_softmax_kernel(
    input_ptr,
    output_ptr,
    input_row_stride,
    output_row_stride,
    n_cols,
    BLOCK_SIZE: tl.constexpr,
):
    """
    Compute log_softmax along the last dimension of a 2D tensor.
    Each block handles one row of the input tensor.
    """
    # Get the row index for this block
    row_idx = tl.program_id(0).to(tl.int64)

    # Compute base pointers for input and output rows
    row_start_ptr = input_ptr + row_idx * input_row_stride
    output_row_start_ptr = output_ptr + row_idx * output_row_stride

    # Step 1: Find maximum value in the row for numerical stability
    max_val = -float("inf")
    for col_offset in range(0, n_cols, BLOCK_SIZE):
        col_idx = col_offset + tl.arange(0, BLOCK_SIZE)
        mask = col_idx < n_cols

        # Load values
        vals = tl.load(row_start_ptr + col_idx, mask=mask, other=-float("inf"))

        # Update maximum
        max_val = tl.max(tl.maximum(vals, max_val))

    # Step 2: Compute sum of exp(x - max_val)
    sum_exp = 0.0
    for col_offset in range(0, n_cols, BLOCK_SIZE):
        col_idx = col_offset + tl.arange(0, BLOCK_SIZE)
        mask = col_idx < n_cols

        # Load values
        vals = tl.load(row_start_ptr + col_idx, mask=mask, other=0.0)

        # Compute exp(x - max_val) and accumulate
        exp_vals = tl.exp(vals - max_val)
        sum_exp += tl.sum(tl.where(mask, exp_vals, 0.0))

    # Compute log(sum_exp)
    log_sum_exp = tl.log(sum_exp)

    # Step 3: Compute final log_softmax values: x - max_val - log_sum_exp
    for col_offset in range(0, n_cols, BLOCK_SIZE):
        col_idx = col_offset + tl.arange(0, BLOCK_SIZE)
        mask = col_idx < n_cols

        # Load values
        vals = tl.load(row_start_ptr + col_idx, mask=mask)

# ... omitted for brevity ...
        # Store results
        tl.store(output_row_start_ptr + col_idx, output, mask=mask)
```
**EN:** This function defines `_log_softmax_kernel`. Compute log_softmax along the last dimension of a 2D tensor. The main inputs are `input_ptr`, `output_ptr`, `input_row_stride`, `output_row_stride`, `n_cols`, `BLOCK_SIZE`. Key calls include `tl.program_id.to`, `range`, `tl.log`, `float`, `tl.load`, `tl.max`. It writes or updates `row_idx`, `row_start_ptr`, `output_row_start_ptr`, `max_val`, `sum_exp`, `log_sum_exp`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_log_softmax_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `input_ptr`, `output_ptr`, `input_row_stride`, `output_row_stride`, `n_cols`, `BLOCK_SIZE`。 关键调用包括 `tl.program_id.to`, `range`, `tl.log`, `float`, `tl.load`, `tl.max`。 它会写入或更新 `row_idx`, `row_start_ptr`, `output_row_start_ptr`, `max_val`, `sum_exp`, `log_sum_exp`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 404-445 — function `log_softmax`
```python
def log_softmax(input: torch.Tensor, dim: int = -1) -> torch.Tensor:
    """
    Compute log_softmax using Triton kernel.

    Args:
        input: Input tensor
        dim: Dimension along which to compute log_softmax
             (only -1 or last dim supported)

    Returns:
        Tensor with log_softmax applied along the specified dimension
    """
    if dim != -1 and dim != input.ndim - 1:
        raise ValueError(
            "This implementation only supports log_softmax along the last dimension"
        )

    # Flatten all dimensions except the last one
    original_shape = input.shape
    input_2d = input.reshape(-1, input.shape[-1])
    input_2d = input_2d.contiguous()

    n_rows, n_cols = input_2d.shape

    # Allocate output tensor
    output = torch.empty_like(input_2d)

    # Choose block size based on the number of columns
    BLOCK_SIZE = 1024

    # Launch kernel with one block per row
    grid = (n_rows,)
    _log_softmax_kernel[grid](
        input_2d,
        output,
        input_2d.stride(0),
        output.stride(0),
        n_cols,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    # Reshape output back to original shape
    return output.reshape(original_shape)
```
**EN:** This function defines `log_softmax`. Compute log_softmax using Triton kernel. The main inputs are `input`, `dim`. Key calls include `input.reshape`, `input_2d.contiguous`, `torch.empty_like`, `_log_softmax_kernel`, `output.reshape`, `ValueError`. It writes or updates `original_shape`, `input_2d`, `n_rows`, `n_cols`, `output`, `BLOCK_SIZE`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `log_softmax`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `input`, `dim`。 关键调用包括 `input.reshape`, `input_2d.contiguous`, `torch.empty_like`, `_log_softmax_kernel`, `output.reshape`, `ValueError`。 它会写入或更新 `original_shape`, `input_2d`, `n_rows`, `n_cols`, `output`, `BLOCK_SIZE`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 448-495 — function `mean_kernel`
```python
@triton.jit
def mean_kernel(
    input_ptr,
    output_ptr,
    input_stride0,
    input_stride1,
    input_stride2,
    output_stride0,
    output_stride1,
    M,  # size before reduction dim
    N,  # size of reduction dim
    K,  # size after reduction dim
    BLOCK_SIZE: tl.constexpr,
):
    """
    Kernel for computing mean along a single dimension.
    Input is viewed as (M, N, K) where N is the dimension being reduced.
    """
    # Program ID gives us which output element we're computing
    pid = tl.program_id(0)

    # Compute output indices
    m_idx = pid // K
    k_idx = pid % K

    # Bounds check
    if m_idx >= M or k_idx >= K:
        return

    # Accumulate sum across reduction dimension
    acc = 0.0
    for n_start in range(0, N, BLOCK_SIZE):
        n_offsets = n_start + tl.arange(0, BLOCK_SIZE)
        mask = n_offsets < N

        # Calculate input indices
        input_idx = (
            m_idx * input_stride0 + n_offsets * input_stride1 + k_idx * input_stride2
        )

        # Load and accumulate
        vals = tl.load(input_ptr + input_idx, mask=mask, other=0.0)
        acc += tl.sum(vals)

    # Compute mean and store
    mean_val = acc / N
    output_idx = m_idx * output_stride0 + k_idx * output_stride1
    tl.store(output_ptr + output_idx, mean_val)
```
**EN:** This function defines `mean_kernel`. Kernel for computing mean along a single dimension. The main inputs are `input_ptr`, `output_ptr`, `input_stride0`, `input_stride1`, `input_stride2`, `output_stride0`. Key calls include `tl.program_id`, `range`, `tl.store`, `tl.load`, `tl.sum`, `tl.arange`. It writes or updates `pid`, `m_idx`, `k_idx`, `acc`, `mean_val`, `output_idx`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `mean_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `input_ptr`, `output_ptr`, `input_stride0`, `input_stride1`, `input_stride2`, `output_stride0`。 关键调用包括 `tl.program_id`, `range`, `tl.store`, `tl.load`, `tl.sum`, `tl.arange`。 它会写入或更新 `pid`, `m_idx`, `k_idx`, `acc`, `mean_val`, `output_idx`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 498-585 — function `mean_dim`
```python
def mean_dim(
    input: torch.Tensor,
    dim: int,
    keepdim: bool = False,
    dtype: torch.dtype | None = None,
) -> torch.Tensor:
    """
    Triton implementation of torch.mean with single dimension reduction.

    Args:
        input: Input tensor
        dim: Single dimension along which to compute mean
        keepdim: Whether to keep the reduced dimension
        dtype: Output dtype. If None, uses input dtype
               (or float32 for integer inputs)

    Returns:
        Tensor with mean values along specified dimension
    """
    # Validate inputs
    assert -input.ndim <= dim < input.ndim, (
        f"Invalid dimension {dim} for tensor with {input.ndim} dimensions"
    )

    # Handle negative dim
    if dim < 0:
        dim = dim + input.ndim

    # Handle dtype
    if dtype is None:
        if input.dtype in [torch.int8, torch.int16, torch.int32, torch.int64]:
            dtype = torch.float32
        else:
            dtype = input.dtype

    # Convert input to appropriate dtype if needed
    if input.dtype != dtype:
        input = input.to(dtype)

    # Get input shape and strides
    shape = list(input.shape)

    # Calculate dimensions for kernel
    M = 1
# ... omitted for brevity ...

    return output
```
**EN:** This function defines `mean_dim`. Triton implementation of torch.mean with single dimension reduction. The main inputs are `input`, `dim`, `keepdim`, `dtype`. Key calls include `list`, `range`, `input.reshape`, `torch.empty`, `mean_kernel`, `input.to`. It writes or updates `shape`, `M`, `N`, `K`, `input_3d`, `output`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `mean_dim`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `input`, `dim`, `keepdim`, `dtype`。 关键调用包括 `list`, `range`, `input.reshape`, `torch.empty`, `mean_kernel`, `input.to`。 它会写入或更新 `shape`, `M`, `N`, `K`, `input_3d`, `output`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 588-589 — function `mm_batch_invariant`
```python
def mm_batch_invariant(a, b):
    return matmul_persistent(a, b)
```
**EN:** This function defines `mm_batch_invariant`. It provides one of the file's main runtime building blocks. The main inputs are `a`, `b`. Key calls include `matmul_persistent`.
**CN:** 该函数定义 `mm_batch_invariant`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `a`, `b`。 关键调用包括 `matmul_persistent`。

### Lines 592-639 — function `matmul_batch_invariant`
```python
def matmul_batch_invariant(a, b, *, out=None):
    # torch.matmul can handle various dimensions
    # For 2D x 2D, it's the same as mm
    if a.ndim == 2 and b.ndim == 2:
        result = matmul_persistent(a, b)
        if out is not None:
            out.copy_(result)
            return out
        return result
    elif b.ndim == 2:
        # Handle ND x 2D: Common for linear layers
        # (..., batch, seq, hidden) @ (hidden, out) -> (..., batch, seq, out)
        batch_dims = a.shape[:-1]
        hidden = a.shape[-1]
        out_dim = b.shape[-1]
        a_2d = a.reshape(-1, hidden)
        result_2d = matmul_persistent(a_2d, b)
        result = result_2d.reshape(batch_dims + (out_dim,))
        if out is not None:
            out.copy_(result)
            return out
        return result
    elif a.ndim >= 2 and b.ndim >= 3:
        # Generic handler for 2D x ND and ND x ND (except 1D)
        # Broadcast dims to ensure both matrices have the same shape
        # If 2D x ND, then unsqueeze to add a dim to a
        if a.ndim == 2:
            a = a.unsqueeze(0)
        broadcast_shape = torch.broadcast_shapes(a.shape[:-2], b.shape[:-2])
        a = a.expand(broadcast_shape + a.shape[-2:])
        b = b.expand(broadcast_shape + b.shape[-2:])
        batch_dim = math.prod(broadcast_shape)
        # Reuse broadcast shape to get all dims except mm dims
        a_3d = a.reshape(batch_dim, a.shape[-2], a.shape[-1])
        b_3d = b.reshape(batch_dim, b.shape[-2], b.shape[-1])
        # Do batched matmul
        result_3d = bmm_batch_invariant(a_3d, b_3d)
        # Reshape back to [broadcast_shape, seq_a, seq_b]
        result = result_3d.reshape(broadcast_shape + (a.shape[-2], b.shape[-1]))
        if out is not None:
            out.copy_(result)
            return out
        return result
    else:
        raise ValueError(
            f"matmul_batch_invariant requires both inputs be at least 2D "
            f"got shapes {a.shape} and {b.shape}"
        )
```
**EN:** This function defines `matmul_batch_invariant`. It provides one of the file's main runtime building blocks. The main inputs are `a`, `b`, `out`. Key calls include `matmul_persistent`, `out.copy_`, `a.reshape`, `result_2d.reshape`, `torch.broadcast_shapes`, `a.expand`. It writes or updates `result`, `batch_dims`, `hidden`, `out_dim`, `a_2d`, `result_2d`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `matmul_batch_invariant`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `a`, `b`, `out`。 关键调用包括 `matmul_persistent`, `out.copy_`, `a.reshape`, `result_2d.reshape`, `torch.broadcast_shapes`, `a.expand`。 它会写入或更新 `result`, `batch_dims`, `hidden`, `out_dim`, `a_2d`, `result_2d`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 642-727 — function `bmm_batch_invariant`
```python
def bmm_batch_invariant(a, b, *, out=None):
    # Batched matrix multiply: (B, M, K) x (B, K, N) -> (B, M, N)
    if not (a.ndim == 3 and b.ndim == 3):
        raise ValueError(
            f"bmm_batch_invariant expects 3D tensors, "
            f"got shapes {a.shape} and {b.shape}"
        )

    if a.shape[0] != b.shape[0]:
        raise ValueError(
            f"Batch dimensions of tensors must match, "
            f"but got {a.shape[0]} and {b.shape[0]}."
        )
    if a.shape[2] != b.shape[1]:
        raise ValueError(
            f"Incompatible inner dimensions for matmul: got {a.shape} and {b.shape}."
        )
    if a.dtype != b.dtype:
        raise ValueError(f"Incompatible dtypes: got {a.dtype} and {b.dtype}.")

    B, M, K = a.shape
    _, _, N = b.shape
    dtype = a.dtype

    if out is None:
        c = torch.empty((B, M, N), device=a.device, dtype=dtype)
    else:
        assert out.shape == (B, M, N), "out tensor has incorrect shape"
        assert out.dtype == dtype and out.device == a.device, "out tensor mismatch"
        c = out

    configs = {
        torch.bfloat16: {
            "BLOCK_SIZE_M": 128,
            "BLOCK_SIZE_N": 128,
            "BLOCK_SIZE_K": 64,
            "num_stages": 3,
            "num_warps": 8,
        },
        torch.float16: {
            "BLOCK_SIZE_M": 128,
            "BLOCK_SIZE_N": _fp16_block_size_n,
            "BLOCK_SIZE_K": 64,
            "num_stages": 3,
# ... omitted for brevity ...

    return c
```
**EN:** This function defines `bmm_batch_invariant`. It provides one of the file's main runtime building blocks. The main inputs are `a`, `b`, `out`. Key calls include `bmm_kernel`, `ValueError`, `torch.empty`, `a.stride`, `b.stride`, `c.stride`. It writes or updates `B`, `M`, `K`, `_`, `N`, `dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `bmm_batch_invariant`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `a`, `b`, `out`。 关键调用包括 `bmm_kernel`, `ValueError`, `torch.empty`, `a.stride`, `b.stride`, `c.stride`。 它会写入或更新 `B`, `M`, `K`, `_`, `N`, `dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 730-731 — function `addmm_batch_invariant`
```python
def addmm_batch_invariant(bias, a, b):
    return matmul_persistent(a, b, bias=bias)
```
**EN:** This function defines `addmm_batch_invariant`. It provides one of the file's main runtime building blocks. The main inputs are `bias`, `a`, `b`. Key calls include `matmul_persistent`.
**CN:** 该函数定义 `addmm_batch_invariant`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `bias`, `a`, `b`。 关键调用包括 `matmul_persistent`。

## Key Concepts / 关键概念
- [EN] Batch-invariant execution helpers / [CN] 批次无关的执行辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `_matmul_launch_metadata`, `_compute_pid`, `matmul_kernel_persistent`, `matmul_persistent` / [CN] 核心符号：`_matmul_launch_metadata`, `_compute_pid`, `matmul_kernel_persistent`, `matmul_persistent`

## Dependencies / 依赖关系
- **External**: `math`, `os`, `collections.abc`, `typing`, `torch` / **外部依赖**: `math`, `os`, `collections.abc`, `typing`, `torch`
- **Internal**: `vllm.envs`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.mem_utils`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils` / **内部依赖**: `vllm.envs`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.mem_utils`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels / **运行时特征**: platform-aware dispatch, Triton kernels
