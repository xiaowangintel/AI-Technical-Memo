# group_norm_silu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/group_norm_silu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import math". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import math”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import math

import torch
import torch.nn.functional as F
import triton  # type: ignore
import triton.language as tl  # type: ignore

from sglang.srt.utils.custom_op import register_custom_op

_SUPPORTED_DTYPES = {torch.float16, torch.bfloat16, torch.float32}
_LARGE_GROUP_THRESHOLD = 1 << 18
_BLOCK_SIZE = 4096
_BLOCKS_PER_PROGRAM = 2
_CHUNK_SIZE = _BLOCK_SIZE * _BLOCKS_PER_PROGRAM


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-66: Function `_group_norm_silu_contiguous_kernel`
```python
def _group_norm_silu_contiguous_kernel(
    input_ptr,
    weight_ptr,
    bias_ptr,
    output_ptr,
    channels,
    spatial_size,
    channels_per_group,
    group_size,
    eps,
    BLOCK_SIZE: tl.constexpr,
):
    group_id = tl.program_id(0).to(tl.int64)
    batch_id = tl.program_id(1).to(tl.int64)

    group_base = batch_id * channels * spatial_size + group_id * group_size
    offsets = tl.arange(0, BLOCK_SIZE)

    sum_val = tl.zeros((), dtype=tl.float32)
    sum_sq = tl.zeros((), dtype=tl.float32)
    for off in range(0, group_size, BLOCK_SIZE):
        idx = off + offsets
        mask = idx < group_size
        x = tl.load(input_ptr + group_base + idx, mask=mask, other=0.0).to(tl.float32)
        sum_val += tl.sum(x, axis=0)
        sum_sq += tl.sum(x * x, axis=0)

    inv_group = 1.0 / group_size
    mean = sum_val * inv_group
    var = sum_sq * inv_group - mean * mean
    rstd = tl.rsqrt(var + eps)

    weight_group_offset = group_id * channels_per_group
    for off in range(0, group_size, BLOCK_SIZE):
        idx = off + offsets
        mask = idx < group_size
        x = tl.load(input_ptr + group_base + idx, mask=mask, other=0.0).to(tl.float32)
        channel_offsets = weight_group_offset + idx // spatial_size
        weight = tl.load(weight_ptr + channel_offsets, mask=mask, other=1.0).to(
            tl.float32
        )
        bias = tl.load(bias_ptr + channel_offsets, mask=mask, other=0.0).to(tl.float32)
        y = (x - mean) * rstd
        y = y * weight + bias
        y = y * tl.sigmoid(y)
        tl.store(output_ptr + group_base + idx, y, mask=mask)


@triton.jit
```
**EN:** This block defines `_group_norm_silu_contiguous_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_silu_contiguous_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 67-104: Function `_group_norm_stats_kernel`
```python
def _group_norm_stats_kernel(
    input_ptr,
    partial_sum_ptr,
    partial_sq_ptr,
    channels,
    spatial_size,
    num_groups,
    channels_per_group,
    group_size,
    chunks_per_row,
    BLOCK_SIZE: tl.constexpr,
    BLOCKS_PER_PROGRAM: tl.constexpr,
):
    row = tl.program_id(0).to(tl.int64)
    chunk_id = tl.program_id(1).to(tl.int64)

    batch_id = row // num_groups
    group_id = row - batch_id * num_groups
    chunk_start = chunk_id * BLOCK_SIZE * BLOCKS_PER_PROGRAM
    group_base = batch_id * channels * spatial_size + group_id * group_size

    sum_val = tl.zeros((), dtype=tl.float32)
    sum_sq = tl.zeros((), dtype=tl.float32)
    offsets = tl.arange(0, BLOCK_SIZE)

    for block_id in range(BLOCKS_PER_PROGRAM):
        idx = chunk_start + block_id * BLOCK_SIZE + offsets
        mask = idx < group_size
        x = tl.load(input_ptr + group_base + idx, mask=mask, other=0.0).to(tl.float32)
        sum_val += tl.sum(x, axis=0)
        sum_sq += tl.sum(x * x, axis=0)

    partial_index = row * chunks_per_row + chunk_id
    tl.store(partial_sum_ptr + partial_index, sum_val)
    tl.store(partial_sq_ptr + partial_index, sum_sq)


@triton.jit
```
**EN:** This block defines `_group_norm_stats_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_stats_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 105-138: Function `_group_norm_finalize_stats_kernel`
```python
def _group_norm_finalize_stats_kernel(
    partial_sum_ptr,
    partial_sq_ptr,
    stats_ptr,
    chunks_per_row,
    group_size,
    eps,
    BLOCK_SIZE: tl.constexpr,
):
    row = tl.program_id(0).to(tl.int64)
    offsets = tl.arange(0, BLOCK_SIZE)

    sum_val = tl.zeros((), dtype=tl.float32)
    sum_sq = tl.zeros((), dtype=tl.float32)
    base = row * chunks_per_row
    for off in range(0, chunks_per_row, BLOCK_SIZE):
        idx = off + offsets
        mask = idx < chunks_per_row
        sum_val += tl.sum(
            tl.load(partial_sum_ptr + base + idx, mask=mask, other=0.0), axis=0
        )
        sum_sq += tl.sum(
            tl.load(partial_sq_ptr + base + idx, mask=mask, other=0.0), axis=0
        )

    inv_group = 1.0 / group_size
    mean = sum_val * inv_group
    var = sum_sq * inv_group - mean * mean
    rstd = tl.rsqrt(var + eps)
    tl.store(stats_ptr + row * 2, mean)
    tl.store(stats_ptr + row * 2 + 1, rstd)


@triton.jit
```
**EN:** This block defines `_group_norm_finalize_stats_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_finalize_stats_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 139-182: Function `_group_norm_apply_kernel`
```python
def _group_norm_apply_kernel(
    input_ptr,
    weight_ptr,
    bias_ptr,
    output_ptr,
    stats_ptr,
    channels,
    spatial_size,
    num_groups,
    channels_per_group,
    group_size,
    chunks_per_row,
    BLOCK_SIZE: tl.constexpr,
    BLOCKS_PER_PROGRAM: tl.constexpr,
):
    row = tl.program_id(0).to(tl.int64)
    chunk_id = tl.program_id(1).to(tl.int64)

    batch_id = row // num_groups
    group_id = row - batch_id * num_groups
    chunk_start = chunk_id * BLOCK_SIZE * BLOCKS_PER_PROGRAM
    group_base = batch_id * channels * spatial_size + group_id * group_size
    weight_group_offset = group_id * channels_per_group

    mean = tl.load(stats_ptr + row * 2)
    rstd = tl.load(stats_ptr + row * 2 + 1)
    offsets = tl.arange(0, BLOCK_SIZE)

    for block_id in range(BLOCKS_PER_PROGRAM):
        idx = chunk_start + block_id * BLOCK_SIZE + offsets
        mask = idx < group_size
        x = tl.load(input_ptr + group_base + idx, mask=mask, other=0.0).to(tl.float32)
        channel_offsets = weight_group_offset + idx // spatial_size
        weight = tl.load(weight_ptr + channel_offsets, mask=mask, other=1.0).to(
            tl.float32
        )
        bias = tl.load(bias_ptr + channel_offsets, mask=mask, other=0.0).to(tl.float32)
        y = (x - mean) * rstd
        y = y * weight + bias
        y = y * tl.sigmoid(y)
        tl.store(output_ptr + group_base + idx, y, mask=mask)


@triton.jit
```
**EN:** This block defines `_group_norm_apply_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_apply_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 183-224: Function `_group_norm_apply_scalar_affine_kernel`
```python
def _group_norm_apply_scalar_affine_kernel(
    input_ptr,
    weight_ptr,
    bias_ptr,
    output_ptr,
    stats_ptr,
    channels,
    spatial_size,
    num_groups,
    channels_per_group,
    group_size,
    chunks_per_row,
    BLOCK_SIZE: tl.constexpr,
    BLOCKS_PER_PROGRAM: tl.constexpr,
):
    row = tl.program_id(0).to(tl.int64)
    chunk_id = tl.program_id(1).to(tl.int64)

    batch_id = row // num_groups
    group_id = row - batch_id * num_groups
    chunk_start = chunk_id * BLOCK_SIZE * BLOCKS_PER_PROGRAM
    group_base = batch_id * channels * spatial_size + group_id * group_size

    channel_id = chunk_start // spatial_size
    affine_offset = group_id * channels_per_group + channel_id
    weight = tl.load(weight_ptr + affine_offset).to(tl.float32)
    bias = tl.load(bias_ptr + affine_offset).to(tl.float32)

    mean = tl.load(stats_ptr + row * 2)
    rstd = tl.load(stats_ptr + row * 2 + 1)
    offsets = tl.arange(0, BLOCK_SIZE)

    for block_id in range(BLOCKS_PER_PROGRAM):
        idx = chunk_start + block_id * BLOCK_SIZE + offsets
        mask = idx < group_size
        x = tl.load(input_ptr + group_base + idx, mask=mask, other=0.0).to(tl.float32)
        y = (x - mean) * rstd
        y = y * weight + bias
        y = y * tl.sigmoid(y)
        tl.store(output_ptr + group_base + idx, y, mask=mask)
```
**EN:** This block defines `_group_norm_apply_scalar_affine_kernel`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_apply_scalar_affine_kernel`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 225-234: Function `_group_norm_silu_native`
```python
def _group_norm_silu_native(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float,
) -> torch.Tensor:
    return F.silu(F.group_norm(x, num_groups, weight=weight, bias=bias, eps=eps))
```
**EN:** This block defines `_group_norm_silu_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_group_norm_silu_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 235-257: Function `_can_use_triton_group_norm_silu`
```python
def _can_use_triton_group_norm_silu(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
) -> bool:
    return (
        x.is_cuda
        and not torch.is_grad_enabled()
        and not x.requires_grad
        and x.dtype in _SUPPORTED_DTYPES
        and x.ndim in (2, 3, 4, 5)
        and x.shape[1] % num_groups == 0
        and weight.is_cuda
        and bias.is_cuda
        and weight.dtype == x.dtype
        and bias.dtype == x.dtype
        and weight.ndim == 1
        and bias.ndim == 1
        and weight.shape == bias.shape == (x.shape[1],)
    )
```
**EN:** This block defines `_can_use_triton_group_norm_silu`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_can_use_triton_group_norm_silu`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 258-288: Function `_launch_one_pass`
```python
def _launch_one_pass(
    x_contiguous: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float,
) -> torch.Tensor:
    batch_size, channels = x_contiguous.shape[:2]
    spatial_size = math.prod(x_contiguous.shape[2:]) if x_contiguous.ndim > 2 else 1
    channels_per_group = channels // num_groups
    group_size = channels_per_group * spatial_size

    x_flat = x_contiguous.reshape(batch_size, channels, spatial_size, 1)
    y_flat = torch.empty_like(x_flat)
    block_size = min(4096, triton.next_power_of_2(max(1, min(group_size, 4096))))

    _group_norm_silu_contiguous_kernel[(num_groups, batch_size)](
        x_flat,
        weight,
        bias,
        y_flat,
        channels,
        spatial_size,
        channels_per_group,
        group_size,
        eps,
        BLOCK_SIZE=block_size,
    )
    return y_flat.reshape_as(x_contiguous)
```
**EN:** This block defines `_launch_one_pass`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_launch_one_pass`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 289-380: Function `_launch_chunked`
```python
def _launch_chunked(
    x_contiguous: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float,
) -> torch.Tensor:
    batch_size, channels = x_contiguous.shape[:2]
    spatial_size = math.prod(x_contiguous.shape[2:]) if x_contiguous.ndim > 2 else 1
    channels_per_group = channels // num_groups
    group_size = channels_per_group * spatial_size
    rows = batch_size * num_groups
    chunks_per_row = triton.cdiv(group_size, _CHUNK_SIZE)

    x_flat = x_contiguous.reshape(-1)
    y = torch.empty_like(x_contiguous)
    y_flat = y.reshape(-1)
    partial_sum = torch.empty(
        (rows, chunks_per_row), device=x_contiguous.device, dtype=torch.float32
    )
    partial_sq = torch.empty_like(partial_sum)
    stats = torch.empty((rows, 2), device=x_contiguous.device, dtype=torch.float32)

    _group_norm_stats_kernel[(rows, chunks_per_row)](
        x_flat,
        partial_sum,
        partial_sq,
        channels,
        spatial_size,
        num_groups,
        channels_per_group,
        group_size,
        chunks_per_row,
        BLOCK_SIZE=_BLOCK_SIZE,
        BLOCKS_PER_PROGRAM=_BLOCKS_PER_PROGRAM,
        num_warps=8,
        num_stages=3,
    )

    reduce_block = min(1024, triton.next_power_of_2(max(1, chunks_per_row)))
    _group_norm_finalize_stats_kernel[(rows,)](
        partial_sum,
        partial_sq,
        stats,
        chunks_per_row,
        group_size,
        eps,
        BLOCK_SIZE=reduce_block,
        num_warps=4,
        num_stages=2,
    )

    if spatial_size % _CHUNK_SIZE == 0 and chunks_per_row >= 64:
        _group_norm_apply_scalar_affine_kernel[(rows, chunks_per_row)](
            x_flat,
            weight,
            bias,
            y_flat,
            stats,
            channels,
            spatial_size,
            num_groups,
            channels_per_group,
            group_size,
            chunks_per_row,
            BLOCK_SIZE=_BLOCK_SIZE,
            BLOCKS_PER_PROGRAM=_BLOCKS_PER_PROGRAM,
            num_warps=4,
            num_stages=3,
        )
    else:
        _group_norm_apply_kernel[(rows, chunks_per_row)](
            x_flat,
            weight,
            bias,
            y_flat,
            stats,
            channels,
            spatial_size,
            num_groups,
# ...
```
**EN:** This block defines `_launch_chunked`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_launch_chunked`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 381-401: Function `_triton_group_norm_silu_cuda`
```python
def _triton_group_norm_silu_cuda(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float = 1e-5,
) -> torch.Tensor:
    if not _can_use_triton_group_norm_silu(x, weight, bias, num_groups):
        return _group_norm_silu_native(x, weight, bias, num_groups, eps)

    x_contiguous = x.contiguous()
    spatial_size = math.prod(x_contiguous.shape[2:]) if x_contiguous.ndim > 2 else 1
    channels_per_group = x_contiguous.shape[1] // num_groups
    group_size = channels_per_group * spatial_size

    with torch.cuda.device(x.device):
        if group_size >= _LARGE_GROUP_THRESHOLD:
            return _launch_chunked(x_contiguous, weight, bias, num_groups, eps)
        return _launch_one_pass(x_contiguous, weight, bias, num_groups, eps)
```
**EN:** This block defines `_triton_group_norm_silu_cuda`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_triton_group_norm_silu_cuda`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 402-412: Function `triton_group_norm_silu`
```python
def triton_group_norm_silu(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float = 1e-5,
) -> torch.Tensor:
    return _triton_group_norm_silu_cuda(x, weight, bias, num_groups, eps)


__all__ = ["triton_group_norm_silu"]
```
**EN:** This block defines `triton_group_norm_silu`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `triton_group_norm_silu`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `math`
- `torch`
- `torch.nn.functional as F`
- `triton`
- `triton.language as tl`
- `sglang.srt.utils.custom_op -> register_custom_op`
