# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/csrc/render/hunyuan3d_rasterizer/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `_load_custom_rasterizer`, `rasterize`, and `interpolate`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Custom CUDA rasterizer for Hunyuan3D texture generation. This module provides JIT-compiled CUDA rasterization for fast mesh rendering. / 该文件属于多模态生成模块。它围绕 `_load_custom_rasterizer`、`rasterize` 和 `interpolate` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-15: module setup and imports / 模块初始化与导入
```python
"""
Custom CUDA rasterizer for Hunyuan3D texture generation.

This module provides JIT-compiled CUDA rasterization for fast mesh rendering.
Adapted from Hunyuan3D-2: https://github.com/Tencent/Hunyuan3D-2
"""

from __future__ import annotations

import os
from typing import List, Tuple

import torch
from sglang.multimodal_gen.csrc.render import load_extension_with_recovery
```
**EN:** This block establishes the module context and imports `__future__`, `os`, `typing`, `torch`, and `sglang.multimodal_gen.csrc.render`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`os`、`typing`、`torch` 和 `sglang.multimodal_gen.csrc.render`。这些依赖为后续实现提供所需符号。

### Lines 17-18: supporting statements / 辅助语句
```python
_abs_path = os.path.dirname(os.path.abspath(__file__))
_custom_rasterizer_kernel = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_abs_path`, and `_custom_rasterizer_kernel`. The code collaborates with `os.path.dirname`, and `os.path.abspath`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_abs_path` 和 `_custom_rasterizer_kernel` 等名称。 代码会与 `os.path.dirname` 和 `os.path.abspath` 协同工作。

### Lines 21-41: `_load_custom_rasterizer` implementation / `_load_custom_rasterizer` 实现
```python
def _load_custom_rasterizer(
    is_cuda: bool = True,
):
    """JIT compile and load the custom rasterizer kernel."""
    global _custom_rasterizer_kernel

    if _custom_rasterizer_kernel is not None:
        return _custom_rasterizer_kernel
    
    cuda_enabled_flag = ["-DCUDA_ENABLED"] if is_cuda else []
    
    _custom_rasterizer_kernel = load_extension_with_recovery(
        name="custom_rasterizer_kernel",
        sources=[
            f"{_abs_path}/rasterizer.cpp",
        ] + ([f"{_abs_path}/rasterizer_gpu.cu"] if is_cuda else []),
        extra_cflags=["-O3"] + cuda_enabled_flag,
        extra_cuda_cflags=["-O3", "--use_fast_math"] + cuda_enabled_flag,
        verbose=False,
    )
    return _custom_rasterizer_kernel
```
**EN:** This block defines function `_load_custom_rasterizer`. JIT compile and load the custom rasterizer kernel. Key calls include `load_extension_with_recovery`. The implementation branches on conditions. Parameters such as `is_cuda` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_load_custom_rasterizer`。 它用于加载custom rasterizer。 关键调用包括 `load_extension_with_recovery`。 实现中包含条件分支。 本段逻辑主要由 `is_cuda` 等参数驱动。

### Lines 44-69: `rasterize` implementation / `rasterize` 实现
```python
def rasterize(
    pos: torch.Tensor,
    tri: torch.Tensor,
    resolution: Tuple[int, int],
    clamp_depth: torch.Tensor = None,
    use_depth_prior: int = 0,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Rasterize mesh to get face indices and barycentric coordinates."""
    device = "cpu" if pos.device.type == "npu" else pos.device.type
    kernel = _load_custom_rasterizer(device == "cuda")

    if clamp_depth is None:
        clamp_depth = torch.zeros(0, device=pos.device)

    # pos should be [N, 4], remove batch dim if present
    if pos.dim() == 3:
        pos = pos[0]

    findices, barycentric = kernel.rasterize_image(
        pos.to(device), tri.to(device), clamp_depth.to(device), resolution[1], resolution[0], 1e-6, use_depth_prior
    )

    findices = findices.to(pos.device)
    barycentric = barycentric.to(pos.device)

    return findices, barycentric
```
**EN:** This block defines function `rasterize`. Rasterize mesh to get face indices and barycentric coordinates. Key calls include `_load_custom_rasterizer`, `kernel.rasterize_image`, `findices.to`, `barycentric.to`, and `torch.zeros`. The implementation branches on conditions. Parameters such as `pos`, `tri`, `resolution`, `clamp_depth`, and `use_depth_prior` drive the behavior in this section.
**CN:** 该代码块定义了函数 `rasterize`。 它用于处理 rasterize 相关逻辑。 关键调用包括 `_load_custom_rasterizer`、`kernel.rasterize_image`、`findices.to`、`barycentric.to` 和 `torch.zeros`。 实现中包含条件分支。 本段逻辑主要由 `pos`、`tri`、`resolution`、`clamp_depth` 和 `use_depth_prior` 等参数驱动。

### Lines 72-84: `interpolate` implementation / `interpolate` 实现
```python
def interpolate(
    col: torch.Tensor,
    findices: torch.Tensor,
    barycentric: torch.Tensor,
    tri: torch.Tensor,
) -> torch.Tensor:
    """Interpolate vertex attributes using barycentric coordinates."""
    # Handle zero indices (background)
    f = findices - 1 + (findices == 0)
    vcol = col[0, tri.long()[f.long()]]
    result = barycentric.view(*barycentric.shape, 1) * vcol
    result = torch.sum(result, axis=-2)
    return result.view(1, *result.shape)
```
**EN:** This block defines function `interpolate`. Interpolate vertex attributes using barycentric coordinates. Key calls include `torch.sum`, `result.view`, `barycentric.view`, `tri.long`, and `f.long`. Parameters such as `col`, `findices`, `barycentric`, and `tri` drive the behavior in this section.
**CN:** 该代码块定义了函数 `interpolate`。 它用于处理 interpolate 相关逻辑。 关键调用包括 `torch.sum`、`result.view`、`barycentric.view`、`tri.long` 和 `f.long`。 本段逻辑主要由 `col`、`findices`、`barycentric` 和 `tri` 等参数驱动。

### Lines 87-87: supporting statements / 辅助语句
```python
__all__ = ["rasterize", "interpolate"]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__` 等名称。

## Key Concepts / 关键概念
- `_load_custom_rasterizer`: JIT compile and load the custom rasterizer kernel. / 顶层函数，用于加载custom rasterizer。
- `rasterize`: Rasterize mesh to get face indices and barycentric coordinates. / 顶层函数，用于处理 rasterize 相关逻辑。
- `interpolate`: Interpolate vertex attributes using barycentric coordinates. / 顶层函数，用于处理 interpolate 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.csrc.render`

- **Total lines / 总行数**: 87
