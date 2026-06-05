# torch_fallback.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/torch_fallback.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Pytorch native based fallbacks for Triton diffusion kernels.". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Pytorch native based fallbacks for Triton diffusion kernels.”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
```python
"""Pytorch native based fallbacks for Triton diffusion kernels.

Triton is not available on some platforms, so these pure-PyTorch
implementations replace the Triton kernels

"""

from typing import Optional

import torch
from torch import Tensor
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 14-46: Function `fuse_scale_shift_kernel_native`
```python
def fuse_scale_shift_kernel_native(
    x: torch.Tensor,
    scale: torch.Tensor,
    shift: torch.Tensor,
    scale_constant: float = 1.0,
    block_l: int = 128,
    block_c: int = 128,
):
    """Native fallback for fuse_scale_shift_kernel with scale_constant support."""
    B, L, C = x.shape

    def _expand(t: torch.Tensor) -> torch.Tensor:
        if t.dim() == 4:
            # [B, F, 1, C] -> [B, L, C]
            num_frames = t.shape[1]
            frame_seqlen = L // num_frames
            return (
                t.squeeze(2)
                .unsqueeze(2)
                .expand(-1, -1, frame_seqlen, -1)
                .reshape(B, L, C)
            )
        elif t.dim() == 2:
            # [B, C] -> [B, 1, C]
            return t.unsqueeze(1)
        return t

    scale = _expand(scale)
    shift = _expand(shift)

    return x * (scale_constant + scale) + shift
```
**EN:** This block defines `fuse_scale_shift_kernel_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_scale_shift_kernel_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 47-62: Function `apply_rotary_embedding_native`
```python
def apply_rotary_embedding_native(
    x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, interleaved: bool = False
) -> torch.Tensor:
    """Native fallback for rotary embedding (shared with NPU implementation)."""
    if interleaved and cos.shape[-1] == x.shape[-1]:
        cos = cos[..., ::2]
        sin = sin[..., ::2]
    cos = cos.unsqueeze(-2).to(x.dtype)
    sin = sin.unsqueeze(-2).to(x.dtype)
    x1 = x[..., ::2]
    x2 = x[..., 1::2]
    o1 = x1 * cos - x2 * sin
    o2 = x2 * cos + x1 * sin
    return torch.stack((o1, o2), dim=-1).flatten(-2)
```
**EN:** This block defines `apply_rotary_embedding_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rotary_embedding_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 63-91: Function `norm_infer_native`
```python
def norm_infer_native(
    x: Tensor,
    weight: Optional[Tensor],
    bias: Optional[Tensor],
    eps: float,
    is_rms_norm: bool = False,
    out: Optional[Tensor] = None,
) -> Tensor:
    """Native fallback for norm_infer (layer norm / rms norm inference)."""
    orig_dtype = x.dtype
    x = x.contiguous().float()
    if is_rms_norm:
        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x_hat = x * torch.rsqrt(variance + eps)
    else:
        mean = x.mean(dim=-1, keepdim=True)
        variance = (x - mean).pow(2).mean(dim=-1, keepdim=True)
        x_hat = (x - mean) * torch.rsqrt(variance + eps)
    if weight is not None:
        x_hat = x_hat * weight.float()
    if bias is not None:
        x_hat = x_hat + bias.float()
    result = x_hat.to(orig_dtype)
    if out is not None:
        out.copy_(result)
        return out
    return result
```
**EN:** This block defines `norm_infer_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `norm_infer_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 92-103: Function `triton_one_pass_rms_norm_native`
```python
def triton_one_pass_rms_norm_native(
    x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6
) -> torch.Tensor:
    """Native fallback for triton_one_pass_rms_norm."""
    shape = x.shape
    orig_dtype = x.dtype
    x = x.contiguous().float()
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x_hat = x * torch.rsqrt(variance + eps)
    return (x_hat * w.float()).to(orig_dtype).view(shape)
```
**EN:** This block defines `triton_one_pass_rms_norm_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `triton_one_pass_rms_norm_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 104-146: Function `rms_norm_fn_native`
```python
def rms_norm_fn_native(
    x,
    weight,
    bias,
    residual=None,
    x1=None,
    weight1=None,
    bias1=None,
    eps=1e-6,
    dropout_p=0.0,
    rowscale=None,
    prenorm=False,
    residual_in_fp32=False,
    zero_centered_weight=False,
    return_dropout_mask=False,
    out_dtype=None,
    out=None,
    residual_out=None,
):
    """Native fallback for rms_norm_fn (inference only, no dropout/x1 support)."""
    x_shape_og = x.shape
    orig_dtype = x.dtype
    x = x.reshape(-1, x.shape[-1]).float()
    if residual is not None:
        residual = residual.reshape(-1, residual.shape[-1]).float()
        x = x + residual
        residual_out_val = x.to(torch.float32 if residual_in_fp32 else orig_dtype)
    else:
        residual_out_val = None
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x_hat = x * torch.rsqrt(variance + eps)
    if weight is not None:
        w = weight.float()
        if zero_centered_weight:
            w = w + 1.0
        x_hat = x_hat * w
    if bias is not None:
        x_hat = x_hat + bias.float()
    final_dtype = out_dtype if out_dtype is not None else orig_dtype
    y = x_hat.to(final_dtype).reshape(x_shape_og)
    if residual is not None and residual_out_val is not None:
        return y, residual_out_val.reshape(x_shape_og)
    return y
```
**EN:** This block defines `rms_norm_fn_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `rms_norm_fn_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Optional`
- `torch`
- `torch -> Tensor`
