# jet_vlm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/jet_vlm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for jet vlm so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 jet vlm 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7: Imports dependencies
```python
from typing import Any

from transformers.configuration_utils import PretrainedConfig
from transformers.models.siglip import SiglipVisionConfig

from sglang.srt.configs.jet_nemotron import JetNemotronConfig
from sglang.srt.configs.mamba_utils import Mamba2CacheParams
```
**EN:** This block groups related imports for the module, including typing.Any, transformers.configuration_utils.PretrainedConfig, transformers.models.siglip.SiglipVisionConfig, sglang.srt.configs.jet_nemotron.JetNemotronConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Any, transformers.configuration_utils.PretrainedConfig, transformers.models.siglip.SiglipVisionConfig, sglang.srt.configs.jet_nemotron.JetNemotronConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams，为后续代码准备所需名称。

### Lines 8-9: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 10-10: Declares class JetVLMConfig
```python
class JetVLMConfig(PretrainedConfig):
```
**EN:** This block introduces class `JetVLMConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `JetVLMConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 11-16: Declares model_type, sub_configs, _auto_class
```python
    model_type = "jet_vlm"
    sub_configs = {
        "text_config": JetNemotronConfig,
        "vision_config": SiglipVisionConfig,
    }
    _auto_class = "AutoConfig"
```
**EN:** This block initializes a related set of values in the JetVLMConfig, including model_type, sub_configs, _auto_class. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 JetVLMConfig 中初始化一组相关值，包括 model_type, sub_configs, _auto_class。将这些赋值集中在一起有助于理解周边配置。

### Lines 17-17: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetVLMConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetVLMConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 18-41: Defines function JetVLMConfig.__init__
```python
    def __init__(
        self,
        *,
        text_config: dict[str, Any] | None = None,
        vision_config: dict[str, Any] | None = None,
        image_token_id: int | None = None,
        video_token_id: int | None = None,
        **kwargs,
    ):
        self.text_config = (
            JetNemotronConfig(**text_config)
            if text_config is not None
            else JetNemotronConfig()
        )
        self.vision_config = (
            SiglipVisionConfig(**vision_config)
            if vision_config is not None
            else SiglipVisionConfig()
        )

        self.image_token_id = image_token_id if image_token_id is not None else -1
        self.video_token_id = video_token_id if video_token_id is not None else -1

        super().__init__(**kwargs)
```
**EN:** This block defines function `JetVLMConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `JetVLMConfig.__init__`。 参数包括 self。

### Lines 42-42: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetVLMConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetVLMConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-45: Defines function JetVLMConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self) -> list[int]:
        return self.text_config.full_attention_layer_ids
```
**EN:** This block defines function `JetVLMConfig.full_attention_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetVLMConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 46-46: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetVLMConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetVLMConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-49: Defines function JetVLMConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self) -> list[int]:
        return self.text_config.linear_layer_ids
```
**EN:** This block defines function `JetVLMConfig.linear_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetVLMConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 50-50: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetVLMConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetVLMConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 51-53: Defines function JetVLMConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Mamba2CacheParams:
        return self.text_config.mamba2_cache_params
```
**EN:** This block defines function `JetVLMConfig.mamba2_cache_params`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetVLMConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。

## Key Concepts / 关键概念
- **Classes / 类**: `JetVLMConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.models.siglip`
- **Local Modules / 本地模块**: `sglang.srt.configs.jet_nemotron`, `sglang.srt.configs.mamba_utils`
