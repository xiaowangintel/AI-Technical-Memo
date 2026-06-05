# latent_upsampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/upsampler/latent_upsampler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for latent upsampler within the multimodal runtime. Key symbols include `BlurDownsample`, `PixelShuffleND`, `ResBlock`. / 该模块实现多模态运行时中与 latent upsampler 相关的模型构件。 关键符号包括 `BlurDownsample`, `PixelShuffleND`, `ResBlock`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup / 导入与模块初始化
```python
# Ported from https://github.com/Lightricks/LTX-2
# SPDX-License-Identifier: Apache-2.0

import math
from typing import Optional, Tuple

import torch
import torch.nn.functional as F
from einops import rearrange

from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 16-56: Class `BlurDownsample` / 类 `BlurDownsample`
```python
class BlurDownsample(torch.nn.Module):
    """Anti-aliased spatial downsampling by integer stride using a fixed separable binomial kernel."""

    def __init__(self, dims: int, stride: int, kernel_size: int = 5) -> None:
        super().__init__()
        assert dims in (2, 3)
        assert isinstance(stride, int) and stride >= 1
        assert kernel_size >= 3 and kernel_size % 2 == 1
        self.dims = dims
        self.stride = stride
        self.kernel_size = kernel_size

        k = torch.tensor([math.comb(kernel_size - 1, i) for i in range(kernel_size)])
        k2d = k[:, None] @ k[None, :]
# ...
            padding=self.kernel_size // 2,
            groups=c,
        )
        return x2d
```
**EN:** This class models `BlurDownsample` as a specialization of `torch.nn.Module`. Anti-aliased spatial downsampling by integer stride using a fixed separable binomial kernel. Important methods include `__init__`, `forward`, `_apply_2d`.
**CN:** 该类实现 `BlurDownsample`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：Anti-aliased spatial downsampling by integer stride using a fixed separable binomial kernel. 其中较重要的方法包括 `__init__`, `forward`, `_apply_2d`。

### Lines 59-91: Class `PixelShuffleND` / 类 `PixelShuffleND`
```python
class PixelShuffleND(torch.nn.Module):
    """N-dimensional pixel shuffle for upsampling tensors."""

    def __init__(self, dims: int, upscale_factors: Tuple[int, int, int] = (2, 2, 2)):
        super().__init__()
        assert dims in [1, 2, 3]
        self.dims = dims
        self.upscale_factors = upscale_factors

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.dims == 3:
            return rearrange(
                x,
                "b (c p1 p2 p3) d h w -> b c (d p1) (h p2) (w p3)",
# ...
                p1=self.upscale_factors[0],
            )
        else:
            raise ValueError(f"Unsupported dims: {self.dims}")
```
**EN:** This class models `PixelShuffleND` as a specialization of `torch.nn.Module`. N-dimensional pixel shuffle for upsampling tensors. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `PixelShuffleND`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：N-dimensional pixel shuffle for upsampling tensors. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 94-118: Class `ResBlock` / 类 `ResBlock`
```python
class ResBlock(torch.nn.Module):
    """Residual block with two conv layers, group norm, and SiLU activation."""

    def __init__(
        self, channels: int, mid_channels: Optional[int] = None, dims: int = 3
    ):
        super().__init__()
        if mid_channels is None:
            mid_channels = channels
        conv = torch.nn.Conv2d if dims == 2 else torch.nn.Conv3d
        self.conv1 = conv(channels, mid_channels, kernel_size=3, padding=1)
        self.norm1 = torch.nn.GroupNorm(32, mid_channels)
        self.conv2 = conv(mid_channels, channels, kernel_size=3, padding=1)
        self.norm2 = torch.nn.GroupNorm(32, channels)
# ...
        x = self.conv2(x)
        x = self.norm2(x)
        x = self.activation(x + residual)
        return x
```
**EN:** This class models `ResBlock` as a specialization of `torch.nn.Module`. Residual block with two conv layers, group norm, and SiLU activation. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ResBlock`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：Residual block with two conv layers, group norm, and SiLU activation. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 121-127: Function `_rational_for_scale` / 函数 `_rational_for_scale`
```python
def _rational_for_scale(scale: float) -> Tuple[int, int]:
    mapping = {0.75: (3, 4), 1.5: (3, 2), 2.0: (2, 1), 4.0: (4, 1)}
    if float(scale) not in mapping:
        raise ValueError(
            f"Unsupported scale {scale}. Choose from {list(mapping.keys())}"
        )
    return mapping[float(scale)]
```
**EN:** This function drives `_rational_for_scale` with inputs such as `scale`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_rational_for_scale`，主要处理 `scale` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 130-150: Class `SpatialRationalResampler` / 类 `SpatialRationalResampler`
```python
class SpatialRationalResampler(torch.nn.Module):
    """Fully-learned rational spatial scaling via PixelShuffle + anti-aliased downsample."""

    def __init__(self, mid_channels: int, scale: float):
        super().__init__()
        self.scale = float(scale)
        self.num, self.den = _rational_for_scale(self.scale)
        self.conv = torch.nn.Conv2d(
            mid_channels, (self.num**2) * mid_channels, kernel_size=3, padding=1
        )
        self.pixel_shuffle = PixelShuffleND(2, upscale_factors=(self.num, self.num))
        self.blur_down = BlurDownsample(dims=2, stride=self.den)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
# ...
        x = self.pixel_shuffle(x)
        x = self.blur_down(x)
        x = rearrange(x, "(b f) c h w -> b c f h w", b=b, f=f)
        return x
```
**EN:** This class models `SpatialRationalResampler` as a specialization of `torch.nn.Module`. Fully-learned rational spatial scaling via PixelShuffle + anti-aliased downsample. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SpatialRationalResampler`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：Fully-learned rational spatial scaling via PixelShuffle + anti-aliased downsample. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 153-275: Class `LatentUpsampler` / 类 `LatentUpsampler`
```python
class LatentUpsampler(torch.nn.Module, LayerwiseOffloadableModuleMixin):
    """
    Upsample VAE latents spatially and/or temporally.

    Args:
        in_channels: Number of channels in the input latent.
        mid_channels: Number of channels in the middle layers.
        num_blocks_per_stage: Number of ResBlocks per stage (pre/post upsampling).
        dims: Dimensionality of convolutions (2 or 3).
        spatial_upsample: Whether to spatially upsample.
        temporal_upsample: Whether to temporally upsample.
        spatial_scale: Scale factor for spatial upsampling.
        rational_resampler: Whether to use rational resampler for spatial upsampling.
    """
# ...
                x = block(x)
            x = self.final_conv(x)

        return x
```
**EN:** This class models `LatentUpsampler` as a specialization of `torch.nn.Module`, `LayerwiseOffloadableModuleMixin`. Upsample VAE latents spatially and/or temporally. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LatentUpsampler`，并继承/扩展 `torch.nn.Module`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Upsample VAE latents spatially and/or temporally. 其中较重要的方法包括 `__init__`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Sampling parameter control / 采样参数控制
- Symbol `BlurDownsample` anchors the module API / 符号 `BlurDownsample` 构成该模块的核心 API
- Symbol `PixelShuffleND` anchors the module API / 符号 `PixelShuffleND` 构成该模块的核心 API
- Symbol `ResBlock` anchors the module API / 符号 `ResBlock` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `torch`, `torch.nn.functional`, `einops`
- **Stdlib / 标准库**: `math`, `typing`
