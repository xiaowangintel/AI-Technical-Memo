# realesrgan_upscaler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/postprocess/realesrgan_upscaler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for realesrgan upscaler in the multimodal generation stack. Key symbols include `SRVGGNetCompact`, `ResidualDenseBlock`, `RRDB`. / 该模块包含多模态生成体系中与 realesrgan upscaler 相关的运行时支持代码。 关键符号包括 `SRVGGNetCompact`, `ResidualDenseBlock`, `RRDB`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Real-ESRGAN upscaling for SGLang diffusion pipelines.

Real-ESRGAN model code is vendored and adapted from:
  - https://github.com/xinntao/Real-ESRGAN  (BSD-3-Clause License)
  Copyright (c) 2021 xinntao

The ImageUpscaler wrapper and integration code are original work.
"""

import math
import os
from typing import Optional
# ...
_REALESRGAN_TILE_PAD = 32

# Module-level cache: model_path -> UpscalerModel instance
_MODEL_CACHE: dict[str, "UpscalerModel"] = {}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 43-98: Class `SRVGGNetCompact` / 类 `SRVGGNetCompact`
```python
class SRVGGNetCompact(nn.Module):
    """Compact VGG-style network for super resolution.

    Corresponds to ``realesr-animevideov3`` and ``realesr-general-x4v3``.
    Reference: xinntao/Real-ESRGAN (BSD-3-Clause).
    """

    def __init__(
        self,
        num_in_ch: int = 3,
        num_out_ch: int = 3,
        num_feat: int = 64,
        num_conv: int = 16,
        upscale: int = 4,
# ...
        out = self.upsampler(out)
        # residual addition with nearest upsampled input
        base = F.interpolate(x, scale_factor=self.upscale, mode="nearest")
        return out + base
```
**EN:** This class models `SRVGGNetCompact` as a specialization of `nn.Module`. Compact VGG-style network for super resolution. Important methods include `__init__`, `_make_act`, `forward`.
**CN:** 该类实现 `SRVGGNetCompact`，并继承/扩展 `nn.Module`。 文档字符串指出：Compact VGG-style network for super resolution. 其中较重要的方法包括 `__init__`, `_make_act`, `forward`。

### Lines 101-119: Class `ResidualDenseBlock` / 类 `ResidualDenseBlock`
```python
class ResidualDenseBlock(nn.Module):
    """Residual Dense Block used in RRDB (RealESRGAN_x4plus)."""

    def __init__(self, num_feat: int = 64, num_grow_ch: int = 32):
        super().__init__()
        self.conv1 = nn.Conv2d(num_feat, num_grow_ch, 3, 1, 1)
        self.conv2 = nn.Conv2d(num_feat + num_grow_ch, num_grow_ch, 3, 1, 1)
        self.conv3 = nn.Conv2d(num_feat + 2 * num_grow_ch, num_grow_ch, 3, 1, 1)
        self.conv4 = nn.Conv2d(num_feat + 3 * num_grow_ch, num_grow_ch, 3, 1, 1)
        self.conv5 = nn.Conv2d(num_feat + 4 * num_grow_ch, num_feat, 3, 1, 1)
        self.lrelu = nn.LeakyReLU(negative_slope=0.2, inplace=True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x1 = self.lrelu(self.conv1(x))
        x2 = self.lrelu(self.conv2(torch.cat((x, x1), 1)))
        x3 = self.lrelu(self.conv3(torch.cat((x, x1, x2), 1)))
        x4 = self.lrelu(self.conv4(torch.cat((x, x1, x2, x3), 1)))
        x5 = self.conv5(torch.cat((x, x1, x2, x3, x4), 1))
        return x5 * 0.2 + x
```
**EN:** This class models `ResidualDenseBlock` as a specialization of `nn.Module`. Residual Dense Block used in RRDB (RealESRGAN_x4plus). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ResidualDenseBlock`，并继承/扩展 `nn.Module`。 文档字符串指出：Residual Dense Block used in RRDB (RealESRGAN_x4plus). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 122-135: Class `RRDB` / 类 `RRDB`
```python
class RRDB(nn.Module):
    """Residual in Residual Dense Block."""

    def __init__(self, num_feat: int, num_grow_ch: int = 32):
        super().__init__()
        self.rdb1 = ResidualDenseBlock(num_feat, num_grow_ch)
        self.rdb2 = ResidualDenseBlock(num_feat, num_grow_ch)
        self.rdb3 = ResidualDenseBlock(num_feat, num_grow_ch)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        out = self.rdb1(x)
        out = self.rdb2(out)
        out = self.rdb3(out)
        return out * 0.2 + x
```
**EN:** This class models `RRDB` as a specialization of `nn.Module`. Residual in Residual Dense Block. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `RRDB`，并继承/扩展 `nn.Module`。 文档字符串指出：Residual in Residual Dense Block. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 138-185: Class `RRDBNet` / 类 `RRDBNet`
```python
class RRDBNet(nn.Module):
    """RRDB network for RealESRGAN_x4plus (heavier, higher quality for photos)."""

    def __init__(
        self,
        num_in_ch: int = 3,
        num_out_ch: int = 3,
        scale: int = 4,
        num_feat: int = 64,
        num_block: int = 23,
        num_grow_ch: int = 32,
    ):
        super().__init__()
        self.scale = scale
# ...
        feat = self.lrelu(
            self.conv_up2(F.interpolate(feat, scale_factor=2, mode="nearest"))
        )
        return self.conv_last(self.lrelu(self.conv_hr(feat)))
```
**EN:** This class models `RRDBNet` as a specialization of `nn.Module`. RRDB network for RealESRGAN_x4plus (heavier, higher quality for photos). Important methods include `__init__`, `forward`.
**CN:** 该类实现 `RRDBNet`，并继承/扩展 `nn.Module`。 文档字符串指出：RRDB network for RealESRGAN_x4plus (heavier, higher quality for photos). 其中较重要的方法包括 `__init__`, `forward`。

### Lines 193-250: Function `_build_net_from_state_dict` / 函数 `_build_net_from_state_dict`
```python
def _build_net_from_state_dict(state_dict: dict) -> nn.Module:
    """Detect architecture from checkpoint keys and return an unloaded network."""
    if "conv_first.weight" in state_dict:
        # RRDBNet (e.g., RealESRGAN_x4plus)
        num_feat = state_dict["conv_first.weight"].shape[0]
        num_block = sum(
            1
            for k in state_dict
            if k.startswith("body.") and k.endswith(".rdb1.conv1.weight")
        )
        num_grow_ch = state_dict["body.0.rdb1.conv1.weight"].shape[0]
        logger.info(
            "Detected RRDBNet: num_feat=%d, num_block=%d, num_grow_ch=%d",
            num_feat,
# ...
            num_conv=num_conv,
            upscale=upscale,
            act_type="prelu",
        )
```
**EN:** This function drives `_build_net_from_state_dict` with inputs such as `state_dict`. Detect architecture from checkpoint keys and return an unloaded network.
**CN:** 这个函数负责 `_build_net_from_state_dict`，主要处理 `state_dict` 等输入。 文档字符串说明：Detect architecture from checkpoint keys and return an unloaded network.

### Lines 258-375: Class `UpscalerModel` / 类 `UpscalerModel`
```python
class UpscalerModel:
    """Wraps a Real-ESRGAN network, provides load() and upscale() API."""

    def __init__(self, net: nn.Module, scale: int):
        self.net = net
        self.scale = scale  # the model's native upscaling factor (e.g. 4)

    @property
    def device(self) -> torch.device:
        return next(self.net.parameters()).device

    @property
    def dtype(self) -> torch.dtype:
        return next(self.net.parameters()).dtype
# ...
            )

        out_np = out.squeeze(0).permute(1, 2, 0).clamp(0.0, 1.0).cpu().numpy()
        return (out_np * 255.0).astype(np.uint8)
```
**EN:** This class models `UpscalerModel`. Wraps a Real-ESRGAN network, provides load() and upscale() API. Important methods include `__init__`, `device`, `dtype`, `_should_use_tiled_upscale`.
**CN:** 该类实现 `UpscalerModel`。 文档字符串指出：Wraps a Real-ESRGAN network, provides load() and upscale() API. 其中较重要的方法包括 `__init__`, `device`, `dtype`, `_should_use_tiled_upscale`。

### Lines 383-473: Class `ImageUpscaler` / 类 `ImageUpscaler`
```python
class ImageUpscaler:
    """
    Lazy-loaded Real-ESRGAN upscaler.

    Weights are downloaded and cached on first call to `.upscale()`.
    Supports both SRVGGNetCompact (lightweight, default) and RRDBNet (heavier).
    """

    def __init__(
        self,
        model_path: Optional[str] = None,
        scale: int = 4,
        half_precision: bool = False,
    ):
# ...
            return frames
        model = self._ensure_model_loaded()
        outscale = self._scale if self._scale != model.scale else None
        return [model.upscale(frame, outscale=outscale) for frame in frames]
```
**EN:** This class models `ImageUpscaler`. Lazy-loaded Real-ESRGAN upscaler. Important methods include `__init__`, `_ensure_model_loaded`, `upscale`.
**CN:** 该类实现 `ImageUpscaler`。 文档字符串指出：Lazy-loaded Real-ESRGAN upscaler. 其中较重要的方法包括 `__init__`, `_ensure_model_loaded`, `upscale`。

### Lines 481-527: Function `_resolve_model_path` / 函数 `_resolve_model_path`
```python
def _resolve_model_path(model_path: str) -> str:
    """Return a local .pth file path.

    Accepts:
    - An existing local file path (pass-through).
    - A HuggingFace ``repo_id`` → downloads the default weight file
      (``RealESRGAN_x4.pth``).
    - A HuggingFace ``repo_id:filename`` → downloads *filename* from *repo_id*,
      allowing users to specify custom weight files hosted on HF.
    """
    if os.path.isfile(model_path):
        return model_path

    # Parse optional "repo_id:filename" syntax; fall back to default filename.
# ...
            f"'repo_id:filename' format (e.g. 'my-org/my-esrgan:weights.pth'). "
            f"Original error: {e}"
        ) from e
    return local_path
```
**EN:** This function drives `_resolve_model_path` with inputs such as `model_path`. Return a local .pth file path.
**CN:** 这个函数负责 `_resolve_model_path`，主要处理 `model_path` 等输入。 文档字符串说明：Return a local .pth file path.

### Lines 535-566: Function `upscale_frames` / 函数 `upscale_frames`
```python
def upscale_frames(
    frames: list[np.ndarray],
    model_path: Optional[str] = None,
    scale: int = 4,
    half_precision: bool = False,
) -> list[np.ndarray]:
    """
    Convenience wrapper around ImageUpscaler.

    The model always runs at its native resolution (e.g. 4× for
    ``RealESRGAN_x4.pth``).  If *scale* differs from the native factor,
    a cheap bicubic resize is applied after the network output – the same
    approach used by the official Real-ESRGAN ``--outscale`` flag.

# ...
    upscaler = ImageUpscaler(
        model_path=model_path, scale=scale, half_precision=half_precision
    )
    return upscaler.upscale(frames)
```
**EN:** This function drives `upscale_frames` with inputs such as `frames`, `model_path`, `scale`, `half_precision`. Convenience wrapper around ImageUpscaler.
**CN:** 这个函数负责 `upscale_frames`，主要处理 `frames`, `model_path`, `scale`, `half_precision` 等输入。 文档字符串说明：Convenience wrapper around ImageUpscaler.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `numpy`, `torch`, `torch.nn`, `torch.nn.functional`, `huggingface_hub`
- **Stdlib / 标准库**: `math`, `os`, `typing`
