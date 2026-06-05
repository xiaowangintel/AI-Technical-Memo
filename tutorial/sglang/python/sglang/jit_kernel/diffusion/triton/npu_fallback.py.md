# npu_fallback.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/triton/npu_fallback.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import torch". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import torch”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
```python
import torch
import torch_npu

NPU_ROTARY_MUL_MAX_NUM_HEADS = 1000
NPU_ROTARY_MUL_MAX_HEAD_SIZE = 896


# TODO: remove this when triton ascend bug is fixed
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 9-19: Function `fuse_scale_shift_native`
```python
def fuse_scale_shift_native(
    x: torch.Tensor,
    scale: torch.Tensor,
    shift: torch.Tensor,
    block_l: int = 128,
    block_c: int = 128,
):
    return x * (1 + scale) + shift


# TODO: remove this when triton ascend bug is fixed
```
**EN:** This block defines `fuse_scale_shift_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fuse_scale_shift_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 20-50: Function `apply_rotary_embedding_native`
```python
def apply_rotary_embedding_native(
    x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, interleaved: bool = False
) -> torch.Tensor:
    if interleaved and cos.shape[-1] == x.shape[-1]:
        cos = cos[..., ::2]
        sin = sin[..., ::2]
    cos = cos.unsqueeze(-2).to(x.dtype)
    sin = sin.unsqueeze(-2).to(x.dtype)

    if (
        cos.dim() == 3
        and x.dim() == 3
        and x.shape[1] < NPU_ROTARY_MUL_MAX_NUM_HEADS
        and x.shape[2] < NPU_ROTARY_MUL_MAX_HEAD_SIZE
        and not interleaved
    ):
        if cos.size(-1) * 2 == x.size(-1):
            cos = torch.cat([cos, cos], dim=-1)
            sin = torch.cat([sin, sin], dim=-1)
        cos = cos.unsqueeze(0)
        sin = sin.unsqueeze(0)
        x = x.unsqueeze(0)
        x_embed = torch_npu.npu_rotary_mul(x, cos, sin)
        x_embed = x_embed.squeeze(0)
        return x_embed

    x1 = x[..., ::2]
    x2 = x[..., 1::2]
    o1 = x1 * cos - x2 * sin
    o2 = x2 * cos + x1 * sin
    return torch.stack((o1, o2), dim=-1).flatten(-2)
```
**EN:** This block defines `apply_rotary_embedding_native`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `apply_rotary_embedding_native`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `torch`
- `torch_npu`
