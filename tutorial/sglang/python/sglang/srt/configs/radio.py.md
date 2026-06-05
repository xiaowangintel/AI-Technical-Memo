# radio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/radio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for radio so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 radio 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Comments and module notes
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/transformers_utils/configs/radio.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 18-18: Documents the scope
```python
"""Radio vision model configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 19-19: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-21: Imports dependencies
```python
from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging
```
**EN:** This block groups related imports for the module, including transformers.configuration_utils.PretrainedConfig, transformers.utils.logging. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.configuration_utils.PretrainedConfig, transformers.utils.logging，为后续代码准备所需名称。

### Lines 22-22: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 24-24: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-30: Declares VIT_TIMM_DIM_BY_NAME
```python
VIT_TIMM_DIM_BY_NAME: dict[str, tuple[int, int, int, int]] = {
    "vit_small_patch16_224": (384, 12, 6, 1536),
    "vit_base_patch16_224": (768, 12, 12, 3072),
    "vit_large_patch16_224": (1024, 24, 16, 4096),
    "vit_huge_patch16_224": (1280, 32, 16, 5120),
}
```
**EN:** This statement initializes VIT_TIMM_DIM_BY_NAME in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 VIT_TIMM_DIM_BY_NAME。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 31-31: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-33: Declares OPENAI_CLIP_MEAN, OPENAI_CLIP_STD
```python
OPENAI_CLIP_MEAN = (0.48145466, 0.4578275, 0.40821073)
OPENAI_CLIP_STD = (0.26862954, 0.26130258, 0.27577711)
```
**EN:** This block initializes a related set of values in the module, including OPENAI_CLIP_MEAN, OPENAI_CLIP_STD. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 OPENAI_CLIP_MEAN, OPENAI_CLIP_STD。将这些赋值集中在一起有助于理解周边配置。

### Lines 34-35: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 36-36: Declares class RadioConfig
```python
class RadioConfig(PretrainedConfig):
```
**EN:** This block introduces class `RadioConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a Radio vision model. It is used to instantiate a Radio model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `RadioConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a Radio vision model. It is used to instantiate a Radio model according to the specified arguments, defining the model architecture.

### Lines 37-60: Documents the scope
```python
    r"""
    This is the configuration class to store the configuration of a Radio
    vision model. It is used to instantiate a Radio model according to the
    specified arguments, defining the model architecture.

    Args:
        model_name: Name of the vision transformer model
            (e.g., "vit_base_patch16_224"). Used to determine architecture
            dimensions from `VIT_TIMM_DIM_BY_NAME`.
        image_size: The size (resolution) of each image.
        patch_size: The size (resolution) of each patch.
        qkv_bias: Whether to add a bias to the queries, keys and values.
        qk_normalization: Whether to apply normalization to queries and keys.
        norm_type: The normalization type to use.
        layer_norm_eps: The epsilon used by the layer normalization layers.
        initializer_factor: A factor for initializing all weight matrices.
        hidden_act: The non-linear activation function in the encoder.
        max_img_size: Maximum image size for position embeddings.
        norm_mean: Mean values for image normalization (RGB channels).
            Defaults to (0.48145466, 0.4578275, 0.40821073)).
        norm_std: Standard deviation values for image normalization
            (RGB channels). Defaults to (0.26862954, 0.26130258, 0.27577711)).
        reg_tokens: Number of register tokens to use.
    """
```
**EN:** This string literal serves as documentation for the RadioConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 RadioConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 61-61: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the RadioConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 RadioConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 62-62: Declares model_type
```python
    model_type = "radio"
```
**EN:** This statement initializes model_type in the RadioConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 RadioConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 63-63: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the RadioConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 RadioConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 64-120: Defines function RadioConfig.__init__
```python
    def __init__(
        self,
        model_name: str,
        image_size: int = 224,
        patch_size: int = 16,
        qkv_bias: bool = True,
        qk_normalization: bool = False,
        norm_type: str = "layer_norm",
        layer_norm_eps: float = 1e-6,
        initializer_factor: float = 1.0,
        hidden_act: str = "gelu",
        max_img_size: int = 2048,
        norm_mean: tuple[float, float, float] | list = OPENAI_CLIP_MEAN,
        norm_std: tuple[float, float, float] | list = OPENAI_CLIP_STD,
        reg_tokens: int | None = None,
        min_num_patches: int = 0,
        max_num_patches: int = 0,
        video_temporal_patch_size: int = 1,
        separate_video_embedder: bool = True,
        video_target_num_patches: int = 0,
        video_maintain_aspect_ratio: bool = True,
        drop_path_rate: float = 0.0,
        dropout: float = 0.0,
        **kwargs,
    ):
        self.model_name = model_name
        (
            self.hidden_size,
            self.num_hidden_layers,
            self.num_attention_heads,
            self.intermediate_size,
        ) = VIT_TIMM_DIM_BY_NAME[model_name]
        self.image_size = image_size
        self.patch_size = patch_size
        self.qkv_bias = qkv_bias
        self.qk_normalization = qk_normalization
        self.norm_type = norm_type
        self.layer_norm_eps = layer_norm_eps
        self.initializer_factor = initializer_factor
        self.hidden_act = hidden_act
        self.max_img_size = max_img_size
        self.norm_mean = (
            list(norm_mean) if isinstance(norm_mean, (tuple, list)) else norm_mean
        )
        self.norm_std = (
            list(norm_std) if isinstance(norm_std, (tuple, list)) else norm_std
        )
        self.reg_tokens = reg_tokens
        self.min_num_patches = min_num_patches
        self.max_num_patches = max_num_patches
        self.video_temporal_patch_size = video_temporal_patch_size
        self.separate_video_embedder = separate_video_embedder
        self.video_target_num_patches = video_target_num_patches
        self.video_maintain_aspect_ratio = video_maintain_aspect_ratio
        self.drop_path_rate = drop_path_rate
        self.dropout = dropout
        super().__init__(**kwargs)
```
**EN:** This block defines function `RadioConfig.__init__`. Parameters: self, model_name, image_size, patch_size, qkv_bias, qk_normalization, norm_type, layer_norm_eps, initializer_factor, hidden_act, max_img_size, norm_mean, norm_std, reg_tokens, min_num_patches, max_num_patches, video_temporal_patch_size, separate_video_embedder, video_target_num_patches, video_maintain_aspect_ratio, drop_path_rate, dropout.
**CN:** 该代码块定义函数 `RadioConfig.__init__`。 参数包括 self、model_name、image_size、patch_size、qkv_bias、qk_normalization、norm_type、layer_norm_eps、initializer_factor、hidden_act、max_img_size、norm_mean、norm_std、reg_tokens、min_num_patches、max_num_patches、video_temporal_patch_size、separate_video_embedder、video_target_num_patches、video_maintain_aspect_ratio、drop_path_rate、dropout。

## Key Concepts / 关键概念
- **Classes / 类**: `RadioConfig`
- **Constants / 常量**: `VIT_TIMM_DIM_BY_NAME`, `OPENAI_CLIP_MEAN`, `OPENAI_CLIP_STD`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
