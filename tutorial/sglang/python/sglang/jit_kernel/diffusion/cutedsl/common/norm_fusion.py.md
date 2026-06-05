# norm_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/cutedsl/common/norm_fusion.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from typing import Optional, Tuple, Union". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from typing import Optional, Tuple, Union”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup
```python
from typing import Optional, Tuple, Union

import cutlass
import cutlass.cute as cute
import torch
from einops import rearrange

from sglang.jit_kernel.diffusion.cutedsl.common.reduce import (
    cta_reduce_sum,
    warp_reduce_sum,
)


@cute.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 15-31: Function `apply_norm_cta`
```python
def apply_norm_cta(
    norm_type: cutlass.Constexpr,
    num_warps: cutlass.Constexpr,
    tidx: cutlass.Int32,
    tXrX: cute.Tensor,
    tWrW: Optional[cute.Tensor],
    tBrB: Optional[cute.Tensor],
    D: Union[cutlass.Int32, cutlass.Constexpr],
    eps: Union[cutlass.Float32, cutlass.Constexpr],
) -> cute.Tensor:
    if cutlass.const_expr(norm_type == "rms"):
        return apply_rmsnorm_cta(num_warps, tidx, tXrX, tWrW, D, eps)
    else:
        return apply_layernorm_cta(num_warps, tidx, tXrX, tWrW, tBrB, D, eps)


@cute.jit
```
**EN:** This block defines `apply_norm_cta`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_norm_cta`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-60: Function `apply_rmsnorm_cta`
```python
def apply_rmsnorm_cta(
    num_warps: Union[cutlass.Int32, cutlass.Constexpr],
    tidx: cutlass.Int32,
    tXrX: cute.Tensor,
    tWrW: Optional[cute.Tensor],
    D: Union[cutlass.Int32, cutlass.Constexpr],
    eps: Union[cutlass.Float32, cutlass.Constexpr],
) -> cute.Tensor:
    """
    RMSNorm:
      y[i] = x[i] / sqrt(sum(x ^ 2) / D + eps) * w[i]
    """
    val = cute.Float32(0.0)
    for idx in range(cute.size(tXrX)):
        # Accumulate in FP32 to improve numerical precision.
        x_fp32 = tXrX[idx].to(cutlass.Float32)
        val += x_fp32 * x_fp32
    val = warp_reduce_sum(val)
    acc_sq = cta_reduce_sum(val, num_warps, tidx)
    factor = cute.rsqrt(acc_sq / D + eps)
    tNrN = cute.make_fragment_like(tXrX)
    if cutlass.const_expr(isinstance(tWrW, cute.Tensor)):
        tNrN.store((tXrX.load() * factor * tWrW.load()).to(tNrN.element_type))
    else:
        tNrN.store((tXrX.load() * factor).to(tNrN.element_type))
    return tNrN


@cute.jit
```
**EN:** This block defines `apply_rmsnorm_cta`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rmsnorm_cta`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 61-129: Function `apply_layernorm_cta`
```python
def apply_layernorm_cta(
    num_warps: Union[cutlass.Int32, cutlass.Constexpr],
    tidx: cutlass.Int32,
    tXrX: cute.Tensor,
    tWrW: Optional[cute.Tensor],
    tBrB: Optional[cute.Tensor],
    D: Union[cutlass.Int32, cutlass.Constexpr],
    eps: Union[cutlass.Float32, cutlass.Constexpr],
) -> cute.Tensor:
    """
    LayerNorm:
        mean = sum(x) / D
        var  = sum((x - mean) ^ 2) / D
        y[i] = (x[i] - mean) / sqrt(var + eps) * w[i] + b[i]
    """
    # Reduce mean
    val = cute.Float32(0.0)
    for idx in range(cute.size(tXrX)):
        # Accumulate in FP32 to improve numerical precision.
        val += tXrX[idx].to(cutlass.Float32)
    val = warp_reduce_sum(val)
    val = cta_reduce_sum(val, num_warps, tidx)
    mean = val / D
    # Reduce variance
    val = cute.Float32(0.0)
    for idx in range(cute.size(tXrX)):
        # Accumulate in FP32 to improve numerical precision.
        x_fp32 = tXrX[idx].to(cutlass.Float32)
        val += (x_fp32 - mean) * (x_fp32 - mean)
    val = warp_reduce_sum(val)
    val = cta_reduce_sum(val, num_warps, tidx)
    factor = cute.rsqrt(val / D + eps)
    # Normalize
    tNrN = cute.make_fragment_like(tXrX)
    if cutlass.const_expr(
        isinstance(tWrW, cute.Tensor) and isinstance(tBrB, cute.Tensor)
    ):
        tNrN.store(
            ((tXrX.load() - mean) * factor * tWrW.load() + tBrB.load()).to(
                tNrN.element_type
            )
        )
    else:
        tNrN.store(((tXrX.load() - mean) * factor).to(tNrN.element_type))
    return tNrN


################################################################################
# BSFD Indexing
################################################################################
# In diffusion norm-fusion kernels, we compute `norm(x) + y`, where
# `x` has shape [B, S, D] and `y` may come in various broadcastable forms:
#   [1], [D], [1, D], [1, 1, D], [B, D], [B, 1, D], [B, S, D], or [B, F, 1, D].
#
# For a given (batch_id, seq_id), the index mapping for `y` falls into 3 cases:
#   1) Scalar broadcast [1]:
#        (batch_id, seq_id, *) -> (0)
#   2) Frame-based BSFD broadcast [B, F, 1, D]:
#        frame_id = seq_id // len_frame
#        (batch_id, seq_id, *) -> (batch_id, frame_id, *)
#   3) All other cases:
#        `y` is broadcast to [B, S, D] (via view/expand, no materialization),
#        and indexed as (batch_id, seq_id, *).
#
# This helper normalizes `y` into a BSFD-compatible view so that kernel
# indexing logic remains simple and uniform.
################################################################################
```
**EN:** This block defines `apply_layernorm_cta`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_layernorm_cta`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 130-159: Function `broadcast_tensor_for_bsfd`
```python
def broadcast_tensor_for_bsfd(
    tensor: Union[Optional[torch.Tensor], int],
    B: int,
    S: int,
    D: int,
) -> Union[Optional[torch.Tensor], int]:
    """
    Broadcast to (B, S, D) without memory copy for following shapes:
    - [D], [1, D], [1, 1, D], [B, D], [B, 1, D], [B, S, D].
    """

    # Return directly for non-tensor value
    if not isinstance(tensor, torch.Tensor):
        return tensor

    if tensor.ndim == 1:
        # Scalar [1] is preserved as-is and handled specially in CuTe kernel.
        if tensor.numel() == 1:
            return tensor
        return rearrange(tensor, "d -> 1 1 d").expand(B, S, D)
    if tensor.ndim == 2:
        return rearrange(tensor, "b d -> b 1 d").expand(B, S, D)
    if tensor.ndim == 3:
        return tensor.expand(B, S, D)
    if tensor.ndim == 4:
        return tensor
    raise ValueError(f"BSFD broadcast: unsupported tensor ndim: {tensor.ndim}.")


@cute.jit
```
**EN:** This block defines `broadcast_tensor_for_bsfd`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `broadcast_tensor_for_bsfd`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 160-201: Function `tensor_slice_for_bsfd`
```python
def tensor_slice_for_bsfd(
    mV: cute.Tensor,
    thr_copy: cute.ThrCopy,
    batch_id: cutlass.Int32,
    seq_id: cutlass.Int32,
    S: Union[cutlass.Int32, cutlass.Constexpr],
    D: Union[cutlass.Int32, cutlass.Constexpr],
) -> Tuple[cute.Tensor, cute.Tensor]:
    """
    Slice a BSFD-compatible tensor into a per-thread gmem tile and rmem fragment.

    Given a logical (batch_id, seq_id), this helper selects the corresponding
    D-length slice from `mV` and prepares it for vectorized copy.
    """
    gV: cute.Tensor
    if cutlass.const_expr(cute.is_static(mV.layout) and cute.size(mV.layout) == 1):
        # build a ((1,1),(1,)) layout so it could broadcast-align with the
        # regular rmem fragment shape ((4,1),(k,)).
        layout = cute.make_layout(shape=((1, 1), (1,)))
        tVgV = cute.make_tensor(mV.iterator, layout)
        tVrV = cute.make_rmem_tensor(layout, mV.element_type)
        return tVgV, tVrV

    # Use `local_tile` instead of direct indexing to preserve gmem base pointer
    # alignment required for vectorized loads.
    if cutlass.const_expr(len(mV.shape) == 1):
        gV = mV
    elif cutlass.const_expr(len(mV.shape) == 3):
        gV = cute.local_tile(mV, tiler=(1, 1, D), coord=(batch_id, seq_id, 0))
        gV = gV[0, 0, None]
    elif cutlass.const_expr(len(mV.shape) == 4):
        # Compute frame length at runtime (instead of compile time) to avoid
        # specializing kernels on the frame dimension.
        frame_len = S // mV.shape[1]
        frame_id = seq_id // frame_len
        gV = cute.local_tile(mV, tiler=(1, 1, 1, D), coord=(batch_id, frame_id, 0, 0))
        gV = gV[0, 0, 0, None]
    else:
        raise NotImplementedError(f"BSFD slice: unsupported shape {mV.shape}.")
    tVgV = thr_copy.partition_S(gV)
    tVrV = cute.make_fragment_like(tVgV, tVgV.element_type)
    return tVgV, tVrV
```
**EN:** This block defines `tensor_slice_for_bsfd`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `tensor_slice_for_bsfd`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Optional`
- `cutlass`
- `cutlass.cute as cute`
- `torch`
- `einops -> rearrange`
- `sglang.jit_kernel.diffusion.cutedsl.common.reduce -> (`
