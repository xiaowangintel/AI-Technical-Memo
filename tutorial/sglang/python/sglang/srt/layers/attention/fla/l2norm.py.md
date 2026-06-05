# l2norm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/l2norm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main l2norm classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 l2norm 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 5-12: imports
```python
from typing import Optional

import torch
import torch.nn as nn
import triton
import triton.language as tl

from sglang.srt.layers.attention.fla.utils import input_guard
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 14-14: module constants
```python
BT_LIST = [8, 16, 32, 64, 128]
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 23-43: function l2norm_fwd_kernel1
```python
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
**EN:** Implements the l2norm fwd kernel1 routine used by this attention module.
**CN:** 实现该注意力模块使用的 l2norm fwd kernel1 例程。

### Lines 54-71: function l2norm_fwd_kernel
```python
@triton.jit
def l2norm_fwd_kernel(
    x,
    y,
    eps,
    NB: tl.constexpr,
    T: tl.constexpr,
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
**EN:** Implements the l2norm fwd kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 l2norm fwd kernel 例程。

### Lines 74-122: function l2norm_fwd
```python
def l2norm_fwd(
    x: torch.Tensor, eps: float = 1e-6, output_dtype: Optional[torch.dtype] = None
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
            D=D,
# ... omitted 5 lines ...
    else:
        l2norm_fwd_kernel1[(T,)](
            x,
            y,
            eps=eps,
            D=D,
            BD=BD,
            num_warps=8,
            num_stages=3,
        )

    return y.view(x_shape_og)
```
**EN:** Implements the l2norm fwd routine used by this attention module.
**CN:** 实现该注意力模块使用的 l2norm fwd 例程。

### Lines 125-126: class L2NormFunction
```python
class L2NormFunction(torch.autograd.Function):
```
**EN:** Defines the l2 norm function type and the state it exposes to the rest of the attention stack.
**CN:** 定义 l2 norm function 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 127-130: method L2NormFunction.forward
```python
    @staticmethod
    @input_guard
    def forward(ctx, x, eps=1e-6, output_dtype=None):
        return l2norm_fwd(x, eps, output_dtype)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 133-136: function l2norm
```python
def l2norm(
    x: torch.Tensor, eps: float = 1e-6, output_dtype: Optional[torch.dtype] = None
) -> torch.Tensor:
    return L2NormFunction.apply(x, eps, output_dtype)
```
**EN:** Implements the l2norm routine used by this attention module.
**CN:** 实现该注意力模块使用的 l2norm 例程。

### Lines 139-139: module constants
```python
l2_norm = l2norm
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 142-143: class L2Norm
```python
class L2Norm(nn.Module):
```
**EN:** Defines the l2 norm type and the state it exposes to the rest of the attention stack.
**CN:** 定义 l2 norm 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 144-147: method L2Norm.__init__
```python
    def __init__(self, eps: float = 1e-6, output_dtype: Optional[torch.dtype] = None):
        super().__init__()
        self.eps = eps
        self.output_dtype = output_dtype
```
**EN:** Initializes the L2Norm instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 L2Norm 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 149-150: method L2Norm.forward
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return l2norm(x, self.eps, self.output_dtype)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `torch.nn`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.fla.utils.input_guard`
