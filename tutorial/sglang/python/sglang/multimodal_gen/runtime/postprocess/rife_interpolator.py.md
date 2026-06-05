# rife_interpolator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/postprocess/rife_interpolator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for rife interpolator in the multimodal generation stack. Key symbols include `warp`, `_conv`, `ResConv`. / 该模块包含多模态生成体系中与 rife interpolator 相关的运行时支持代码。 关键符号包括 `warp`, `_conv`, `ResConv`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
RIFE 4.22.lite frame interpolation for SGLang diffusion pipelines.

RIFE model code is vendored and adapted from:
  - https://github.com/hzwer/ECCV2022-RIFE  (MIT License)
  - https://github.com/hzwer/Practical-RIFE  (MIT License)
  Copyright (c) 2021 Zhewei Huang

The FrameInterpolator wrapper and integration code are original work.
"""

import os
from typing import Optional
# ...
_DEFAULT_RIFE_HF_REPO = "elfgum/RIFE-4.22.lite"

# Module-level cache: model_path -> Model instance
_MODEL_CACHE: dict[str, "Model"] = {}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 39-69: Function `warp` / 函数 `warp`
```python
def warp(tenInput: torch.Tensor, tenFlow: torch.Tensor) -> torch.Tensor:
    """Warp tenInput by tenFlow using grid_sample."""
    # Build base grid for the current size
    tenHorizontal = (
        torch.linspace(-1.0, 1.0, tenFlow.shape[3], device=tenFlow.device)
        .view(1, 1, 1, tenFlow.shape[3])
        .expand(tenFlow.shape[0], -1, tenFlow.shape[2], -1)
    )
    tenVertical = (
        torch.linspace(-1.0, 1.0, tenFlow.shape[2], device=tenFlow.device)
        .view(1, 1, tenFlow.shape[2], 1)
        .expand(tenFlow.shape[0], -1, -1, tenFlow.shape[3])
    )
    tenGrid = torch.cat([tenHorizontal, tenVertical], dim=1)
# ...
        mode="bilinear",
        padding_mode="border",
        align_corners=True,
    )
```
**EN:** This function drives `warp` with inputs such as `tenInput`, `tenFlow`. Warp tenInput by tenFlow using grid_sample.
**CN:** 这个函数负责 `warp`，主要处理 `tenInput`, `tenFlow` 等输入。 文档字符串说明：Warp tenInput by tenFlow using grid_sample.

### Lines 72-85: Function `_conv` / 函数 `_conv`
```python
def _conv(in_planes, out_planes, kernel_size=3, stride=1, padding=1, dilation=1):
    """Conv2d + LeakyReLU helper (matches RIFE 4.22 conv())."""
    return nn.Sequential(
        nn.Conv2d(
            in_planes,
            out_planes,
            kernel_size=kernel_size,
            stride=stride,
            padding=padding,
            dilation=dilation,
            bias=True,
        ),
        nn.LeakyReLU(0.2, True),
    )
```
**EN:** This function drives `_conv` with inputs such as `in_planes`, `out_planes`, `kernel_size`, `stride`. Conv2d + LeakyReLU helper (matches RIFE 4.22 conv()).
**CN:** 这个函数负责 `_conv`，主要处理 `in_planes`, `out_planes`, `kernel_size`, `stride` 等输入。 文档字符串说明：Conv2d + LeakyReLU helper (matches RIFE 4.22 conv()).

### Lines 88-98: Class `ResConv` / 类 `ResConv`
```python
class ResConv(nn.Module):
    """Residual convolution block with learnable beta scaling (RIFE 4.22)."""

    def __init__(self, c: int, dilation: int = 1):
        super().__init__()
        self.conv = nn.Conv2d(c, c, 3, 1, dilation, dilation=dilation, groups=1)
        self.beta = nn.Parameter(torch.ones((1, c, 1, 1)), requires_grad=True)
        self.relu = nn.LeakyReLU(0.2, True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.relu(self.conv(x) * self.beta + x)
```
**EN:** This class models `ResConv` as a specialization of `nn.Module`. Residual convolution block with learnable beta scaling (RIFE 4.22). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ResConv`，并继承/扩展 `nn.Module`。 文档字符串指出：Residual convolution block with learnable beta scaling (RIFE 4.22). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 101-155: Class `IFBlock` / 类 `IFBlock`
```python
class IFBlock(nn.Module):
    """Single-scale optical flow + mask + feature block (RIFE 4.22)."""

    def __init__(self, in_planes: int, c: int = 64):
        super().__init__()
        self.conv0 = nn.Sequential(
            _conv(in_planes, c // 2, 3, 2, 1),
            _conv(c // 2, c, 3, 2, 1),
        )
        self.convblock = nn.Sequential(
            ResConv(c),
            ResConv(c),
            ResConv(c),
            ResConv(c),
# ...
        flow = tmp[:, :4] * scale
        mask = tmp[:, 4:5]
        feat = tmp[:, 5:]
        return flow, mask, feat
```
**EN:** This class models `IFBlock` as a specialization of `nn.Module`. Single-scale optical flow + mask + feature block (RIFE 4.22). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `IFBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Single-scale optical flow + mask + feature block (RIFE 4.22). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 158-177: Class `Head` / 类 `Head`
```python
class Head(nn.Module):
    """Feature encoder producing 4-channel features at full resolution (RIFE 4.22)."""

    def __init__(self):
        super().__init__()
        self.cnn0 = nn.Conv2d(3, 16, 3, 2, 1)
        self.cnn1 = nn.Conv2d(16, 16, 3, 1, 1)
        self.cnn2 = nn.Conv2d(16, 16, 3, 1, 1)
        self.cnn3 = nn.ConvTranspose2d(16, 4, 4, 2, 1)
        self.relu = nn.LeakyReLU(0.2, True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x0 = self.cnn0(x)
        x = self.relu(x0)
# ...
        x2 = self.cnn2(x)
        x = self.relu(x2)
        x3 = self.cnn3(x)
        return x3
```
**EN:** This class models `Head` as a specialization of `nn.Module`. Feature encoder producing 4-channel features at full resolution (RIFE 4.22). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Head`，并继承/扩展 `nn.Module`。 文档字符串指出：Feature encoder producing 4-channel features at full resolution (RIFE 4.22). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 180-259: Class `IFNet` / 类 `IFNet`
```python
class IFNet(nn.Module):
    """4-scale IFNet optical flow network (RIFE 4.22 backbone)."""

    def __init__(self):
        super().__init__()
        self.block0 = IFBlock(7 + 8, c=192)
        self.block1 = IFBlock(8 + 4 + 8 + 8, c=128)
        self.block2 = IFBlock(8 + 4 + 8 + 8, c=64)
        self.block3 = IFBlock(8 + 4 + 8 + 8, c=32)
        self.encode = Head()

    def forward(
        self,
        x: torch.Tensor,
# ...
        mask = torch.sigmoid(mask)
        merged[3] = warped_img0 * mask + warped_img1 * (1 - mask)

        return flow_list, mask_list[3], merged
```
**EN:** This class models `IFNet` as a specialization of `nn.Module`. 4-scale IFNet optical flow network (RIFE 4.22 backbone). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `IFNet`，并继承/扩展 `nn.Module`。 文档字符串指出：4-scale IFNet optical flow network (RIFE 4.22 backbone). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 262-333: Class `Model` / 类 `Model`
```python
class Model:
    """Wraps IFNet, provides load_model() and inference() API."""

    def __init__(self):
        self.flownet = IFNet()
        self.device_type: str = "cpu"

    def eval(self) -> "Model":
        self.flownet.eval()
        return self

    def device(self) -> torch.device:
        return next(self.flownet.parameters()).device

# ...
            )

        # Crop back to original resolution
        return merged[3][:, :, :h, :w]
```
**EN:** This class models `Model`. Wraps IFNet, provides load_model() and inference() API. Important methods include `__init__`, `eval`, `device`, `load_model`.
**CN:** 该类实现 `Model`。 文档字符串指出：Wraps IFNet, provides load_model() and inference() API. 其中较重要的方法包括 `__init__`, `eval`, `device`, `load_model`。

### Lines 341-455: Class `FrameInterpolator` / 类 `FrameInterpolator`
```python
class FrameInterpolator:
    """
    Lazy-loaded RIFE 4.22.lite frame interpolator.

    Weights are loaded on first call to `.interpolate()` and cached globally
    per model_path to avoid reloading across requests.
    """

    def __init__(self, model_path: Optional[str] = None):
        self._model_path = model_path
        self._resolved_path: Optional[str] = None

    def _ensure_model_loaded(self) -> Model:
        """Load RIFE model weights.
# ...

        result.append(frames[-1])
        multiplier = 2**exp
        return result, multiplier
```
**EN:** This class models `FrameInterpolator`. Lazy-loaded RIFE 4.22.lite frame interpolator. Important methods include `__init__`, `_ensure_model_loaded`, `_frame_to_tensor`, `_tensor_to_frame`.
**CN:** 该类实现 `FrameInterpolator`。 文档字符串指出：Lazy-loaded RIFE 4.22.lite frame interpolator. 其中较重要的方法包括 `__init__`, `_ensure_model_loaded`, `_frame_to_tensor`, `_tensor_to_frame`。

### Lines 463-483: Function `interpolate_video_frames` / 函数 `interpolate_video_frames`
```python
def interpolate_video_frames(
    frames: list[np.ndarray],
    exp: int = 1,
    scale: float = 1.0,
    model_path: Optional[str] = None,
) -> tuple[list[np.ndarray], int]:
    """
    Convenience wrapper around FrameInterpolator.

    Args:
        frames:     List of uint8 HWC numpy frames.
        exp:        Interpolation exponent (1=2×, 2=4×).
        scale:      RIFE inference scale (default 1.0; use 0.5 for high-res).
        model_path: Local directory or HuggingFace repo ID containing
# ...
        (interpolated_frames, multiplier)
    """
    interpolator = FrameInterpolator(model_path=model_path)
    return interpolator.interpolate(frames, exp=exp, scale=scale)
```
**EN:** This function drives `interpolate_video_frames` with inputs such as `frames`, `exp`, `scale`, `model_path`. Convenience wrapper around FrameInterpolator.
**CN:** 这个函数负责 `interpolate_video_frames`，主要处理 `frames`, `exp`, `scale`, `model_path` 等输入。 文档字符串说明：Convenience wrapper around FrameInterpolator.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`
- **External / 外部**: `numpy`, `torch`, `torch.nn`, `torch.nn.functional`
- **Stdlib / 标准库**: `os`, `typing`
