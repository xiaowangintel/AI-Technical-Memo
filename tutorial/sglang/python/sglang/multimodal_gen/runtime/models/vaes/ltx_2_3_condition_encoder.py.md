# ltx_2_3_condition_encoder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/ltx_2_3_condition_encoder.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for ltx 2 3 condition encoder within the multimodal runtime. Key symbols include `_patchify_video`, `LTX23VideoPixelNorm`, `LTX23PerChannelStatistics`. / 该模块实现多模态运行时中与 ltx 2 3 condition encoder 相关的模型构件。 关键符号包括 `_patchify_video`, `LTX23VideoPixelNorm`, `LTX23PerChannelStatistics`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup / 导入与模块初始化
```python
from typing import Any

import torch
import torch.nn as nn

from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
from sglang.multimodal_gen.runtime.models.vaes.ltx_2_vae import (
    LTX2VideoCausalConv3d,
    LTX2VideoResnetBlock3d,
    LTXVideoDownsampler3d,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 16-30: Function `_patchify_video` / 函数 `_patchify_video`
```python
def _patchify_video(sample: torch.Tensor, patch_size: int) -> torch.Tensor:
    if patch_size == 1:
        return sample
    batch_size, channels, num_frames, height, width = sample.shape
    sample = sample.reshape(
        batch_size,
        channels,
        num_frames,
        1,
        height // patch_size,
        patch_size,
        width // patch_size,
        patch_size,
    )
    return sample.permute(0, 1, 3, 7, 5, 2, 4, 6).flatten(1, 4)
```
**EN:** This function drives `_patchify_video` with inputs such as `sample`, `patch_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_patchify_video`，主要处理 `sample`, `patch_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 33-42: Class `LTX23VideoPixelNorm` / 类 `LTX23VideoPixelNorm`
```python
class LTX23VideoPixelNorm(nn.Module):
    def __init__(self, dim: int = 1, eps: float = 1e-8) -> None:
        super().__init__()
        self.dim = dim
        self.eps = eps

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        mean_sq = torch.mean(x**2, dim=self.dim, keepdim=True)
        rms = torch.sqrt(mean_sq + self.eps)
        return x / rms
```
**EN:** This class models `LTX23VideoPixelNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX23VideoPixelNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 45-54: Class `LTX23PerChannelStatistics` / 类 `LTX23PerChannelStatistics`
```python
class LTX23PerChannelStatistics(nn.Module):
    def __init__(self, latent_channels: int) -> None:
        super().__init__()
        self.register_buffer("std-of-means", torch.empty(latent_channels))
        self.register_buffer("mean-of-means", torch.empty(latent_channels))

    def normalize(self, x: torch.Tensor) -> torch.Tensor:
        mean = self.get_buffer("mean-of-means").view(1, -1, 1, 1, 1).to(x)
        std = self.get_buffer("std-of-means").view(1, -1, 1, 1, 1).to(x)
        return (x - mean) / std
```
**EN:** This class models `LTX23PerChannelStatistics` as a specialization of `nn.Module`. Important methods include `__init__`, `normalize`.
**CN:** 该类实现 `LTX23PerChannelStatistics`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `normalize`。

### Lines 57-76: Class `LTX23VideoResBlockStack` / 类 `LTX23VideoResBlockStack`
```python
class LTX23VideoResBlockStack(nn.Module):
    def __init__(
        self, channels: int, num_layers: int, spatial_padding_mode: str
    ) -> None:
        super().__init__()
        self.res_blocks = nn.ModuleList(
            [
                LTX2VideoResnetBlock3d(
                    in_channels=channels,
                    out_channels=channels,
                    spatial_padding_mode=spatial_padding_mode,
                )
                for _ in range(num_layers)
            ]
# ...
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        for res_block in self.res_blocks:
            hidden_states = res_block(hidden_states, causal=True)
        return hidden_states
```
**EN:** This class models `LTX23VideoResBlockStack` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX23VideoResBlockStack`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 79-113: Function `_make_ltx23_encoder_block` / 函数 `_make_ltx23_encoder_block`
```python
def _make_ltx23_encoder_block(
    block_name: str,
    block_config: dict[str, Any],
    in_channels: int,
    spatial_padding_mode: str,
) -> tuple[nn.Module, int]:
    if block_name == "res_x":
        return (
            LTX23VideoResBlockStack(
                channels=in_channels,
                num_layers=int(block_config["num_layers"]),
                spatial_padding_mode=spatial_padding_mode,
            ),
            in_channels,
# ...
            spatial_padding_mode=spatial_padding_mode,
        ),
        out_channels,
    )
```
**EN:** This function drives `_make_ltx23_encoder_block` with inputs such as `block_name`, `block_config`, `in_channels`, `spatial_padding_mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_ltx23_encoder_block`，主要处理 `block_name`, `block_config`, `in_channels`, `spatial_padding_mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 116-210: Class `LTX23VideoConditionEncoder` / 类 `LTX23VideoConditionEncoder`
```python
class LTX23VideoConditionEncoder(nn.Module, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = ["down_blocks"]

    def __init__(self, config: dict[str, Any]) -> None:
        super().__init__()

        vae_config = config.get("vae", config)
        latent_channels = int(vae_config["latent_channels"])
        patch_size = int(vae_config.get("patch_size", 4))
        spatial_padding_mode = str(vae_config.get("spatial_padding_mode", "zeros"))
        encoder_blocks = list(vae_config["encoder_blocks"])
        latent_log_var = str(vae_config.get("latent_log_var", "uniform"))

# ...
            hidden_states = torch.cat([means, logvar], dim=1)

        means, _ = torch.chunk(hidden_states, 2, dim=1)
        return self.per_channel_statistics.normalize(means)
```
**EN:** This class models `LTX23VideoConditionEncoder` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX23VideoConditionEncoder`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Video generation flow / 视频生成流程
- Symbol `_patchify_video` anchors the module API / 符号 `_patchify_video` 构成该模块的核心 API
- Symbol `LTX23VideoPixelNorm` anchors the module API / 符号 `LTX23VideoPixelNorm` 构成该模块的核心 API
- Symbol `LTX23PerChannelStatistics` anchors the module API / 符号 `LTX23PerChannelStatistics` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.models.vaes.ltx_2_vae`
- **External / 外部**: `torch`, `torch.nn`
- **Stdlib / 标准库**: `typing`
