# mps_fallback.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/mps_fallback.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"MPS (Apple Silicon) fallbacks for Triton diffusion kernels.". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""MPS (Apple Silicon) fallbacks for Triton diffusion kernels.”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-125: Imports and module setup
```python
"""MPS (Apple Silicon) fallbacks for Triton diffusion kernels.

Triton is not available on macOS / Metal, so these pure-PyTorch (and
optionally MLX-accelerated) implementations replace the Triton kernels
at import time when ``current_platform.is_mps()`` is True.

MLX acceleration (opt-in via ``SGLANG_USE_MLX=1``):
    Norm ops use ``mx.fast.rms_norm`` / ``mx.fast.layer_norm`` — single fused
    Metal kernels that are 1.4x–2.9x faster than the multi-step PyTorch MPS
    decomposition for medium-to-large tensors.
"""

from typing import Optional

import torch
from torch import Tensor

from sglang.srt.utils.tensor_bridge import mlx_to_torch, torch_to_mlx, use_mlx

from .torch_fallback import (
    apply_rotary_embedding_native,
    fuse_scale_shift_kernel_native,
    norm_infer_native,
    rms_norm_fn_native,
    triton_one_pass_rms_norm_native,
)

_use_mlx = use_mlx()

if _use_mlx:
    import mlx.core as mx

# use the common torch native version form torch_fallback
fuse_scale_shift_kernel_native = fuse_scale_shift_kernel_native
apply_rotary_embedding_native = apply_rotary_embedding_native
norm_infer_native = norm_infer_native
triton_one_pass_rms_norm_native = triton_one_pass_rms_norm_native
rms_norm_fn_native = rms_norm_fn_native

# MLX-accelerated norm ops (1.4x–2.9x faster than torch native on MPS)
# Uses mx.fast.rms_norm / mx.fast.layer_norm — single fused Metal kernels
# instead of 7+ separate PyTorch MPS kernel launches.

if _use_mlx:

    def norm_infer_native(  # noqa: F811
        x: Tensor,
        weight: Optional[Tensor],
        bias: Optional[Tensor],
        eps: float,
        is_rms_norm: bool = False,
        out: Optional[Tensor] = None,
    ) -> Tensor:
        """MLX-accelerated norm_infer (layer norm / rms norm inference)."""
        device = x.device
        orig_dtype = x.dtype
        x_mx = torch_to_mlx(x)
        if is_rms_norm:
            w_mx = (
                torch_to_mlx(weight) if weight is not None else mx.ones(x_mx.shape[-1])
            )
            result_mx = mx.fast.rms_norm(x_mx, w_mx, eps)
        else:
            w_mx = torch_to_mlx(weight) if weight is not None else None
            b_mx = torch_to_mlx(bias) if bias is not None else None
            result_mx = mx.fast.layer_norm(x_mx, w_mx, b_mx, eps)
        result = mlx_to_torch(result_mx, device).to(orig_dtype)
        if out is not None:
            out.copy_(result)
            return out
        return result

    def triton_one_pass_rms_norm_native(  # noqa: F811
        x: torch.Tensor, w: torch.Tensor, eps: float = 1e-6
    ) -> torch.Tensor:
        """MLX-accelerated triton_one_pass_rms_norm."""
        device = x.device
        orig_dtype = x.dtype
        x_mx = torch_to_mlx(x)
        w_mx = torch_to_mlx(w)
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `typing -> Optional`
- `torch`
- `torch -> Tensor`
- `sglang.srt.utils.tensor_bridge -> mlx_to_torch`
- `.torch_fallback -> (`
- `mlx.core as mx`
