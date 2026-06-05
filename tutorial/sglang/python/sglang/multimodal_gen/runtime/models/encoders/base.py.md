# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for base within the multimodal runtime. Key symbols include `TextEncoder`, `ImageEncoder`. / 该模块实现多模态运行时中与 base 相关的模型构件。 关键符号包括 `TextEncoder`, `ImageEncoder`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
from abc import ABC, abstractmethod
from dataclasses import field

import torch
from torch import nn

from sglang.multimodal_gen.configs.models.encoders import (
    BaseEncoderOutput,
    ImageEncoderConfig,
    TextEncoderConfig,
)
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-59: Class `TextEncoder` / 类 `TextEncoder`
```python
class TextEncoder(nn.Module, ABC, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "layers",
        "encoder.block",
        "text_model.encoder.layers",
        "model.language_model.layers",
    ]
    _fsdp_shard_conditions: list = field(default_factory=lambda: [])
    _stacked_params_mapping: list[tuple[str, str, str]] = field(default_factory=list)
    _supported_attention_backends: set[AttentionBackendEnum] = (
        TextEncoderConfig()._supported_attention_backends
    )

# ...

    @property
    def supported_attention_backends(self) -> set[AttentionBackendEnum]:
        return self._supported_attention_backends
```
**EN:** This class models `TextEncoder` as a specialization of `nn.Module`, `ABC`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`, `supported_attention_backends`.
**CN:** 该类实现 `TextEncoder`，并继承/扩展 `nn.Module`, `ABC`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`, `supported_attention_backends`。

### Lines 62-87: Class `ImageEncoder` / 类 `ImageEncoder`
```python
class ImageEncoder(nn.Module, ABC, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "layers",
        "vision_model.encoder.layers",
        "model.visual.blocks",
    ]
    _supported_attention_backends: set[AttentionBackendEnum] = (
        ImageEncoderConfig()._supported_attention_backends
    )

    def __init__(self, config: ImageEncoderConfig) -> None:
        super().__init__()
        self.config = config
# ...

    @property
    def supported_attention_backends(self) -> set[AttentionBackendEnum]:
        return self._supported_attention_backends
```
**EN:** This class models `ImageEncoder` as a specialization of `nn.Module`, `ABC`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`, `supported_attention_backends`.
**CN:** 该类实现 `ImageEncoder`，并继承/扩展 `nn.Module`, `ABC`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`, `supported_attention_backends`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `TextEncoder` anchors the module API / 符号 `TextEncoder` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `abc`, `dataclasses`
