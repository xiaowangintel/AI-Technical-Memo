# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vaes/common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for common within the multimodal runtime. Key symbols include `ParallelTiledVAE`, `DiagonalGaussianDistribution`. / 该模块实现多模态运行时中与 common 相关的模型构件。 关键符号包括 `ParallelTiledVAE`, `DiagonalGaussianDistribution`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

from abc import ABC, abstractmethod
from math import isqrt, prod
from typing import Optional, cast

import numpy as np
import torch
import torch.distributed as dist
from diffusers.models.autoencoders.vae import DiagonalGaussianDistribution
from diffusers.utils.torch_utils import randn_tensor
from torch import nn
# ...
)
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 26-686: Class `ParallelTiledVAE` / 类 `ParallelTiledVAE`
```python
class ParallelTiledVAE(ABC, nn.Module, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "encoder.down_blocks",
        "decoder.up_blocks",
        "encoder.down",
        "decoder.up",
    ]
    tile_sample_min_height: int
    tile_sample_min_width: int
    tile_sample_min_num_frames: int
    tile_sample_stride_height: int
    tile_sample_stride_width: int
    tile_sample_stride_num_frames: int
# ...
        Disable tiled VAE decoding. If `enable_tiling` was previously enabled, this method will go back to computing
        decoding in one step.
        """
        self.use_tiling = False
```
**EN:** This class models `ParallelTiledVAE` as a specialization of `ABC`, `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `device`, `temporal_compression_ratio`, `spatial_compression_ratio`.
**CN:** 该类实现 `ParallelTiledVAE`，并继承/扩展 `ABC`, `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `device`, `temporal_compression_ratio`, `spatial_compression_ratio`。

### Lines 690-750: Class `DiagonalGaussianDistribution` / 类 `DiagonalGaussianDistribution`
```python
class DiagonalGaussianDistribution:

    def __init__(self, parameters: torch.Tensor, deterministic: bool = False):
        self.parameters = parameters
        self.mean, self.logvar = torch.chunk(parameters, 2, dim=1)
        self.logvar = torch.clamp(self.logvar, -30.0, 20.0)
        self.deterministic = deterministic
        self.std = torch.exp(0.5 * self.logvar)
        self.var = torch.exp(self.logvar)
        if self.deterministic:
            self.var = self.std = torch.zeros_like(
                self.mean, device=self.parameters.device, dtype=self.parameters.dtype
            )

# ...
        )

    def mode(self) -> torch.Tensor:
        return self.mean
```
**EN:** This class models `DiagonalGaussianDistribution`. Important methods include `__init__`, `sample`, `kl`, `nll`.
**CN:** 该类实现 `DiagonalGaussianDistribution`。 其中较重要的方法包括 `__init__`, `sample`, `kl`, `nll`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Distributed execution / 分布式执行
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `ParallelTiledVAE` anchors the module API / 符号 `ParallelTiledVAE` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `numpy`, `torch`, `torch.distributed`, `diffusers.models.autoencoders.vae`, `diffusers.utils.torch_utils`
- **Stdlib / 标准库**: `abc`, `math`, `typing`
