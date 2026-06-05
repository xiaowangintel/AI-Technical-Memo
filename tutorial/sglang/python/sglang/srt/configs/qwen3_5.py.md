# qwen3_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/qwen3_5.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for qwen3 5 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 qwen3 5 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Imports dependencies
```python
from transformers import PretrainedConfig

from sglang.srt.configs.qwen3_next import Qwen3NextConfig
from sglang.srt.configs.qwen3_vl import Qwen3VLVisionConfig
```
**EN:** This block groups related imports for the module, including transformers.PretrainedConfig, sglang.srt.configs.qwen3_next.Qwen3NextConfig, sglang.srt.configs.qwen3_vl.Qwen3VLVisionConfig. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.PretrainedConfig, sglang.srt.configs.qwen3_next.Qwen3NextConfig, sglang.srt.configs.qwen3_vl.Qwen3VLVisionConfig，为后续代码准备所需名称。

### Lines 5-6: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 7-7: Declares class Qwen3_5VisionConfig
```python
class Qwen3_5VisionConfig(Qwen3VLVisionConfig):
```
**EN:** This block introduces class `Qwen3_5VisionConfig` as a reusable abstraction inside the module. It inherits from Qwen3VLVisionConfig.
**CN:** 该代码块声明类 `Qwen3_5VisionConfig`，作为模块中的可复用抽象。 它继承自 Qwen3VLVisionConfig。

### Lines 8-9: Declares model_type, base_config_key
```python
    model_type = "qwen3_5"
    base_config_key = "vision_config"
```
**EN:** This block initializes a related set of values in the Qwen3_5VisionConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3_5VisionConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 10-10: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5VisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5VisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 11-12: Defines function Qwen3_5VisionConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```
**EN:** This block defines function `Qwen3_5VisionConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `Qwen3_5VisionConfig.__init__`。 参数包括 self。

### Lines 13-14: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-15: Declares class Qwen3_5TextConfig
```python
class Qwen3_5TextConfig(Qwen3NextConfig):
```
**EN:** This block introduces class `Qwen3_5TextConfig` as a reusable abstraction inside the module. It inherits from Qwen3NextConfig.
**CN:** 该代码块声明类 `Qwen3_5TextConfig`，作为模块中的可复用抽象。 它继承自 Qwen3NextConfig。

### Lines 16-17: Declares model_type, base_config_key
```python
    model_type = "qwen3_5_text"
    base_config_key = "text_config"
```
**EN:** This block initializes a related set of values in the Qwen3_5TextConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3_5TextConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 18-18: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5TextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5TextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 19-34: Defines function Qwen3_5TextConfig.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        # HF Qwen3.5 checkpoints may provide RoPE settings under rope_parameters.
        # Normalize it before parent init so downstream code sees the expected values.
        rope_parameters = kwargs.pop("rope_parameters", None)
        if kwargs.get("rope_scaling") is None and rope_parameters is not None:
            kwargs["rope_scaling"] = rope_parameters

        super().__init__(**kwargs)
        if self.rope_scaling is None:
            self.rope_scaling = rope_parameters or {}

        # Keep both names for compatibility with model code paths that read either.
        self.rope_parameters = rope_parameters or self.rope_scaling
```
**EN:** This block defines function `Qwen3_5TextConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `Qwen3_5TextConfig.__init__`。 参数包括 self。

### Lines 35-36: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-37: Declares class Qwen3_5Config
```python
class Qwen3_5Config(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3_5Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3_5Model`]. It is used to instantiate a Qwen3.5 model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3_5Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3_5Model`]. It is used to instantiate a Qwen3.5 model according to the specified arguments, defining the model architecture.

### Lines 38-75: Documents the scope
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3_5Model`]. It is used to instantiate a
    Qwen3.5 model according to the specified arguments, defining the model architecture. Instantiating a configuration
    with the defaults will yield a similar configuration to that of
    Qwen3.5.

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.


    Args:
        text_config (`Union[PreTrainedConfig, dict]`, *optional*, defaults to `Qwen3_5TextConfig`):
            The config object or dictionary of the text backbone.
        vision_config (`Union[PreTrainedConfig, dict]`,  *optional*, defaults to `Qwen3_5VisionConfig`):
            The config object or dictionary of the vision backbone.
        image_token_id (`int`, *optional*, defaults to 151655):
            The image token index to encode the image prompt.
        video_token_id (`int`, *optional*, defaults to 151656):
            The video token index to encode the image prompt.
        vision_start_token_id (`int`, *optional*, defaults to 151652):
            The start token index to encode the image prompt.
        vision_end_token_id (`int`, *optional*, defaults to 151653):
            The end token index to encode the image prompt.
        tie_word_embeddings (`bool`, *optional*, defaults to `False`):
            Whether to tie the word embeddings.

    ```python
    >>> from transformers import Qwen3_5ForConditionalGeneration, Qwen3_5Config

    >>> # Initializing a Qwen3.5 style configuration
    >>> configuration = Qwen3_5Config()

    >>> # Initializing a model from the Qwen3.5 style configuration
    >>> model = Qwen3_5ForConditionalGeneration(configuration)

    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```"""
```
**EN:** This string literal serves as documentation for the Qwen3_5Config, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Qwen3_5Config 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 76-76: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 77-82: Declares model_type, sub_configs, keys_to_ignore_at_inference
```python
    model_type = "qwen3_5"
    sub_configs = {
        "vision_config": Qwen3_5VisionConfig,
        "text_config": Qwen3_5TextConfig,
    }
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3_5Config, including model_type, sub_configs, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3_5Config 中初始化一组相关值，包括 model_type, sub_configs, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 83-83: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 84-109: Defines function Qwen3_5Config.__init__
```python
    def __init__(
        self,
        text_config=None,
        vision_config=None,
        image_token_id=151655,
        video_token_id=151656,
        vision_start_token_id=151652,
        vision_end_token_id=151653,
        tie_word_embeddings=False,
        **kwargs,
    ):
        if isinstance(vision_config, dict):
            self.vision_config = self.sub_configs["vision_config"](**vision_config)
        elif vision_config is None:
            self.vision_config = self.sub_configs["vision_config"]()

        if isinstance(text_config, dict):
            self.text_config = self.sub_configs["text_config"](**text_config)
        elif text_config is None:
            self.text_config = self.sub_configs["text_config"]()

        self.image_token_id = image_token_id
        self.video_token_id = video_token_id
        self.vision_start_token_id = vision_start_token_id
        self.vision_end_token_id = vision_end_token_id
        super().__init__(**kwargs, tie_word_embeddings=tie_word_embeddings)
```
**EN:** This block defines function `Qwen3_5Config.__init__`. Parameters: self, text_config, vision_config, image_token_id, video_token_id, vision_start_token_id, vision_end_token_id, tie_word_embeddings.
**CN:** 该代码块定义函数 `Qwen3_5Config.__init__`。 参数包括 self、text_config、vision_config、image_token_id、video_token_id、vision_start_token_id、vision_end_token_id、tie_word_embeddings。

### Lines 110-111: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 112-112: Declares class Qwen3_5MoeVisionConfig
```python
class Qwen3_5MoeVisionConfig(Qwen3_5VisionConfig):
```
**EN:** This block introduces class `Qwen3_5MoeVisionConfig` as a reusable abstraction inside the module. It inherits from Qwen3_5VisionConfig.
**CN:** 该代码块声明类 `Qwen3_5MoeVisionConfig`，作为模块中的可复用抽象。 它继承自 Qwen3_5VisionConfig。

### Lines 113-113: Declares model_type
```python
    model_type = "qwen3_5_moe"
```
**EN:** This statement initializes model_type in the Qwen3_5MoeVisionConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen3_5MoeVisionConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 114-114: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5MoeVisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5MoeVisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 115-116: Defines function Qwen3_5MoeVisionConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```
**EN:** This block defines function `Qwen3_5MoeVisionConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `Qwen3_5MoeVisionConfig.__init__`。 参数包括 self。

### Lines 117-118: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 119-119: Declares class Qwen3_5MoeTextConfig
```python
class Qwen3_5MoeTextConfig(Qwen3_5TextConfig):
```
**EN:** This block introduces class `Qwen3_5MoeTextConfig` as a reusable abstraction inside the module. It inherits from Qwen3_5TextConfig.
**CN:** 该代码块声明类 `Qwen3_5MoeTextConfig`，作为模块中的可复用抽象。 它继承自 Qwen3_5TextConfig。

### Lines 120-120: Declares model_type
```python
    model_type = "qwen3_5_moe_text"
```
**EN:** This statement initializes model_type in the Qwen3_5MoeTextConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen3_5MoeTextConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 121-121: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5MoeTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5MoeTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 122-123: Defines function Qwen3_5MoeTextConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```
**EN:** This block defines function `Qwen3_5MoeTextConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `Qwen3_5MoeTextConfig.__init__`。 参数包括 self。

### Lines 124-129: Comments and module notes
```python


# All Moe variant classes need explicit __init__ because the kw_only=True
# dataclass decorator in transformers v5.5.3+ auto-generates __init__ for
# subclasses, bypassing parent __init__ methods that set up attributes
# (e.g. norm_topk_prob, rope_scaling) and convert sub-config dicts to objects.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 130-130: Declares class Qwen3_5MoeConfig
```python
class Qwen3_5MoeConfig(Qwen3_5Config):
```
**EN:** This block introduces class `Qwen3_5MoeConfig` as a reusable abstraction inside the module. It inherits from Qwen3_5Config.
**CN:** 该代码块声明类 `Qwen3_5MoeConfig`，作为模块中的可复用抽象。 它继承自 Qwen3_5Config。

### Lines 131-135: Declares model_type, sub_configs
```python
    model_type = "qwen3_5_moe"
    sub_configs = {
        "vision_config": Qwen3_5MoeVisionConfig,
        "text_config": Qwen3_5MoeTextConfig,
    }
```
**EN:** This block initializes a related set of values in the Qwen3_5MoeConfig, including model_type, sub_configs. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3_5MoeConfig 中初始化一组相关值，包括 model_type, sub_configs。将这些赋值集中在一起有助于理解周边配置。

### Lines 136-136: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3_5MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3_5MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 137-138: Defines function Qwen3_5MoeConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```
**EN:** This block defines function `Qwen3_5MoeConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `Qwen3_5MoeConfig.__init__`。 参数包括 self。

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen3_5VisionConfig`, `Qwen3_5TextConfig`, `Qwen3_5Config`, `Qwen3_5MoeVisionConfig`, `Qwen3_5MoeTextConfig`, `Qwen3_5MoeConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers`
- **Local Modules / 本地模块**: `sglang.srt.configs.qwen3_next`, `sglang.srt.configs.qwen3_vl`
