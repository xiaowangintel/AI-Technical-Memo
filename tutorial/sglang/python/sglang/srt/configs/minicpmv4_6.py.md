# minicpmv4_6.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/minicpmv4_6.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for minicpmv4 6 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 minicpmv4 6 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7: Comments and module notes
```python
# Copyright 2026 The SGLang team.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 8-13: Documents the scope
```python
"""Sglang-side ``PretrainedConfig`` classes for MiniCPM-V 4.6.

Mirrors HF ref ``transformers/models/minicpmv4_6/configuration_minicpmv4_6.py``
so we can register the configs ourselves while transformers main has not
yet shipped native ``MiniCPMV4_6Config`` (lands 5.7+).
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 14-14: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-20: Imports dependencies
```python
from typing import Any, Dict, Optional, Union

from transformers import AutoConfig, PretrainedConfig
from transformers.models.auto import CONFIG_MAPPING

from sglang.srt.configs.qwen3_5 import Qwen3_5TextConfig
```
**EN:** This block groups related imports for the module, including typing.Any, typing.Dict, typing.Optional, typing.Union, transformers.AutoConfig, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Any, typing.Dict, typing.Optional, typing.Union, transformers.AutoConfig 等 3 项，为后续代码准备所需名称。

### Lines 21-22: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares class MiniCPMV4_6VisionConfig
```python
class MiniCPMV4_6VisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `MiniCPMV4_6VisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `MiniCPMV4_6VisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 24-25: Declares model_type, base_config_key
```python
    model_type = "minicpmv4_6_vision"
    base_config_key = "vision_config"
```
**EN:** This block initializes a related set of values in the MiniCPMV4_6VisionConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MiniCPMV4_6VisionConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 26-26: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMV4_6VisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMV4_6VisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 27-53: Defines function MiniCPMV4_6VisionConfig.__init__
```python
    def __init__(
        self,
        hidden_size: int = 1152,
        intermediate_size: int = 4304,
        num_hidden_layers: int = 27,
        num_attention_heads: int = 16,
        num_channels: int = 3,
        image_size: int = 980,
        patch_size: int = 14,
        hidden_act: str = "gelu_pytorch_tanh",
        layer_norm_eps: float = 1e-6,
        attention_dropout: float = 0.0,
        insert_layer_id: int = 6,
        **kwargs: Any,
    ) -> None:
        super().__init__(**kwargs)
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.num_channels = num_channels
        self.image_size = image_size
        self.patch_size = patch_size
        self.hidden_act = hidden_act
        self.layer_norm_eps = layer_norm_eps
        self.attention_dropout = attention_dropout
        self.insert_layer_id = insert_layer_id
```
**EN:** This block defines function `MiniCPMV4_6VisionConfig.__init__`. Parameters: self, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_channels, image_size, patch_size, hidden_act, layer_norm_eps, attention_dropout, insert_layer_id.
**CN:** 该代码块定义函数 `MiniCPMV4_6VisionConfig.__init__`。 参数包括 self、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_channels、image_size、patch_size、hidden_act、layer_norm_eps、attention_dropout、insert_layer_id。

### Lines 54-55: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 56-68: Defines function _resolve_text_config_class
```python
def _resolve_text_config_class(model_type: Optional[str]) -> type:
    """``model_type`` -> registered config class. sglang's ``Qwen3_5TextConfig``
    wins over the stock entry when both exist (it carries ``layers_block_type``
    etc. that the model code reads); ``AutoConfig.register`` doesn't replace
    existing entries so we have to short-circuit here. Note that
    ``CONFIG_MAPPING.get`` returns ``None`` even on hit — go through
    ``__getitem__`` to trigger the lazy class import.
    """
    if model_type == Qwen3_5TextConfig.model_type:
        return Qwen3_5TextConfig
    if model_type and model_type in CONFIG_MAPPING:
        return CONFIG_MAPPING[model_type]
    raise KeyError(f"Unknown text_config model_type: {model_type!r}")
```
**EN:** This block defines function `_resolve_text_config_class`. Parameters: model_type. ``model_type`` -> registered config class. sglang's ``Qwen3_5TextConfig`` wins over the stock entry when both exist (it carries ``layers_block_type`` etc.
**CN:** 该代码块定义函数 `_resolve_text_config_class`。 参数包括 model_type。 文档字符串摘要：``model_type`` -> registered config class. sglang's ``Qwen3_5TextConfig`` wins over the stock entry when both exist (it carries ``layers_block_type`` etc.

### Lines 69-70: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 71-91: Defines function _build_text_config
```python
def _build_text_config(
    text_config: Union[None, dict, PretrainedConfig],
) -> PretrainedConfig:
    """Coerce ``text_config`` into the right registered backbone class.

    ``AutoConfig.from_pretrained`` resolves the ``"text_config"`` entry of
    ``sub_configs`` and hands us a pre-built ``PretrainedConfig``; manual
    construction in tests / examples passes a dict or ``None``.
    """
    if text_config is None:
        return _resolve_text_config_class(Qwen3_5TextConfig.model_type)()
    if isinstance(text_config, PretrainedConfig):
        cls = _resolve_text_config_class(getattr(text_config, "model_type", None))
        if isinstance(text_config, cls):
            return text_config
        return cls(**text_config.to_dict())
    if isinstance(text_config, dict):
        cfg = dict(text_config)
        cls = _resolve_text_config_class(cfg.pop("model_type", None))
        return cls(**cfg)
    raise TypeError(f"Unsupported text_config type: {type(text_config)}")
```
**EN:** This block defines function `_build_text_config`. Parameters: text_config. Coerce ``text_config`` into the right registered backbone class. ``AutoConfig.from_pretrained`` resolves the ``"text_config"`` entry of ``sub_configs`` and hands us a pre-built ``PretrainedConfig``; manual construction in tests / examples passes a dict or ``No
**CN:** 该代码块定义函数 `_build_text_config`。 参数包括 text_config。 文档字符串摘要：Coerce ``text_config`` into the right registered backbone class. ``AutoConfig.from_pretrained`` resolves the ``"text_config"`` entry of ``sub_configs`` and hands us a pre-built ``PretrainedConfig``; manual construction in tests / examples passes a dict or ``No

### Lines 92-93: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 94-94: Declares class MiniCPMV4_6Config
```python
class MiniCPMV4_6Config(PretrainedConfig):
```
**EN:** This block introduces class `MiniCPMV4_6Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `MiniCPMV4_6Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 95-95: Declares model_type
```python
    model_type = "minicpmv4_6"
```
**EN:** This statement initializes model_type in the MiniCPMV4_6Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 MiniCPMV4_6Config 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 96-98: Comments and module notes
```python
    # No type annotation: transformers 5+ wraps PretrainedConfig subclasses
    # with @dataclass(kw_only=True), and an annotated mutable default would be
    # rejected as a dataclass field. Matches qwen3_5/qwen3_vl/qwen3_omni.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the MiniCPMV4_6Config.
**CN:** 该范围包含注释或说明，用于记录 MiniCPMV4_6Config 的假设、来源或实现备注。

### Lines 99-102: Declares sub_configs
```python
    sub_configs = {
        "vision_config": MiniCPMV4_6VisionConfig,
        "text_config": AutoConfig,
    }
```
**EN:** This statement initializes sub_configs in the MiniCPMV4_6Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 MiniCPMV4_6Config 中初始化 sub_configs。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 103-103: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMV4_6Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMV4_6Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 104-144: Defines function MiniCPMV4_6Config.__init__
```python
    def __init__(
        self,
        text_config: Optional[Union[Dict[str, Any], PretrainedConfig]] = None,
        vision_config: Optional[Union[Dict[str, Any], PretrainedConfig]] = None,
        insert_layer_id: int = 6,
        image_size: int = 448,
        drop_vision_last_layer: bool = False,
        image_token_id: Optional[int] = None,
        video_token_id: Optional[int] = None,
        tie_word_embeddings: bool = False,
        downsample_mode: str = "16x",
        merge_kernel_size=(2, 2),
        merger_times: int = 1,
        **kwargs: Any,
    ) -> None:
        super().__init__(tie_word_embeddings=tie_word_embeddings, **kwargs)

        if isinstance(vision_config, dict):
            vc = dict(vision_config)
            vc.pop("model_type", None)
            self.vision_config = MiniCPMV4_6VisionConfig(**vc)
        elif vision_config is None:
            self.vision_config = MiniCPMV4_6VisionConfig()
        else:
            self.vision_config = vision_config

        # Mirror the ref ``__post_init__``: keep ``insert_layer_id`` in sync on
        # both the top-level and the vision sub-config.
        self.vision_config.insert_layer_id = insert_layer_id
        self.patch_size = self.vision_config.patch_size

        self.text_config = _build_text_config(text_config)

        self.insert_layer_id = insert_layer_id
        self.image_size = image_size
        self.drop_vision_last_layer = drop_vision_last_layer
        self.image_token_id = image_token_id
        self.video_token_id = video_token_id
        self.downsample_mode = downsample_mode
        self.merge_kernel_size = tuple(merge_kernel_size)
        self.merger_times = merger_times
```
**EN:** This block defines function `MiniCPMV4_6Config.__init__`. Parameters: self, text_config, vision_config, insert_layer_id, image_size, drop_vision_last_layer, image_token_id, video_token_id, tie_word_embeddings, downsample_mode, merge_kernel_size, merger_times.
**CN:** 该代码块定义函数 `MiniCPMV4_6Config.__init__`。 参数包括 self、text_config、vision_config、insert_layer_id、image_size、drop_vision_last_layer、image_token_id、video_token_id、tie_word_embeddings、downsample_mode、merge_kernel_size、merger_times。

### Lines 145-149: Comments and module notes
```python

    # ``MiniCPMBaseModel.__init__`` reads ``self.config.hidden_size`` (written
    # against flat 2.6/4.0/4.5 configs) and ``LogitsProcessor.__init__`` reads
    # ``config.vocab_size`` — proxy both to ``text_config`` so we don't have to
    # fork the base class / logits processor.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the MiniCPMV4_6Config.
**CN:** 该范围包含注释或说明，用于记录 MiniCPMV4_6Config 的假设、来源或实现备注。

### Lines 150-152: Defines function MiniCPMV4_6Config.hidden_size
```python
    @property
    def hidden_size(self) -> int:
        return self.text_config.hidden_size
```
**EN:** This block defines function `MiniCPMV4_6Config.hidden_size`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `MiniCPMV4_6Config.hidden_size`。 参数包括 self。 装饰器包括 property。

### Lines 153-153: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMV4_6Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMV4_6Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 154-156: Defines function MiniCPMV4_6Config.vocab_size
```python
    @property
    def vocab_size(self) -> int:
        return self.text_config.vocab_size
```
**EN:** This block defines function `MiniCPMV4_6Config.vocab_size`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `MiniCPMV4_6Config.vocab_size`。 参数包括 self。 装饰器包括 property。

### Lines 157-158: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 159-159: Declares __all__
```python
__all__ = ["MiniCPMV4_6Config", "MiniCPMV4_6VisionConfig"]
```
**EN:** This statement initializes __all__ in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 __all__。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

## Key Concepts / 关键概念
- **Classes / 类**: `MiniCPMV4_6VisionConfig`, `MiniCPMV4_6Config`
- **Functions / 函数**: `_resolve_text_config_class`, `_build_text_config`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers`, `transformers.models.auto`
- **Local Modules / 本地模块**: `sglang.srt.configs.qwen3_5`
