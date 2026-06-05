# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for base within the multimodal runtime. Key symbols include `BaseDiT`, `CachableDiT`. / 该模块实现多模态运行时中与 base 相关的模型构件。 关键符号包括 `BaseDiT`, `CachableDiT`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
from abc import ABC, abstractmethod
from typing import Any

import torch
from torch import nn

from sglang.multimodal_gen.configs.models import DiTConfig

# NOTE: TeaCacheContext and TeaCacheMixin have been moved to
# sglang.multimodal_gen.runtime.cache.teacache
# For backwards compatibility, re-export from the new location
from sglang.multimodal_gen.runtime.cache.teacache import TeaCacheContext  # noqa: F401
from sglang.multimodal_gen.runtime.cache.teacache import TeaCacheMixin
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-87: Class `BaseDiT` / 类 `BaseDiT`
```python
class BaseDiT(nn.Module, ABC):
    _fsdp_shard_conditions: list = []
    _compile_conditions: list = []
    param_names_mapping: dict
    reverse_param_names_mapping: dict
    hidden_size: int
    num_attention_heads: int
    num_channels_latents: int
    # always supports torch_sdpa
    _supported_attention_backends: set[AttentionBackendEnum] = (
        DiTConfig()._supported_attention_backends
    )

    def __init_subclass__(cls) -> None:
# ...
    @property
    def device(self) -> torch.device:
        """Get the device of the model."""
        return next(self.parameters()).device
```
**EN:** This class models `BaseDiT` as a specialization of `nn.Module`, `ABC`. Important methods include `__init_subclass__`, `__init__`, `forward`, `__post_init__`.
**CN:** 该类实现 `BaseDiT`，并继承/扩展 `nn.Module`, `ABC`。 其中较重要的方法包括 `__init_subclass__`, `__init__`, `forward`, `__post_init__`。

### Lines 90-127: Class `CachableDiT` / 类 `CachableDiT`
```python
class CachableDiT(TeaCacheMixin, BaseDiT):
    """
    An intermediate base class that adds TeaCache optimization functionality to DiT models.

    Inherits TeaCacheMixin for cache logic and BaseDiT for core DiT functionality.
    """

    # These are required class attributes that should be overridden by concrete implementations
    _fsdp_shard_conditions = []
    param_names_mapping = {}
    reverse_param_names_mapping = {}
    lora_param_names_mapping: dict = {}
    # Ensure these instance attributes are properly defined in subclasses
    hidden_size: int
# ...
            "awq_w4a16": [list of patterns for AWQ W4A16],
        }
        """
        return {}
```
**EN:** This class models `CachableDiT` as a specialization of `TeaCacheMixin`, `BaseDiT`. An intermediate base class that adds TeaCache optimization functionality to DiT models. Important methods include `__init__`, `get_nunchaku_quant_rules`.
**CN:** 该类实现 `CachableDiT`，并继承/扩展 `TeaCacheMixin`, `BaseDiT`。 文档字符串指出：An intermediate base class that adds TeaCache optimization functionality to DiT models. 其中较重要的方法包括 `__init__`, `get_nunchaku_quant_rules`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.runtime.cache.teacache`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `abc`, `typing`
