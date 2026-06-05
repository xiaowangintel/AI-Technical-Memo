# qwen3_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/qwen3_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for qwen3 vl so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 qwen3 vl 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Imports dependencies
```python
from transformers import PretrainedConfig
```
**EN:** This block imports dependencies for the module, including transformers.PretrainedConfig. These imports supply standard utilities, third-party packages, or SGLang runtime components used later in the file.
**CN:** 该代码块为 模块 导入依赖，包括 transformers.PretrainedConfig。这些导入为后续实现提供标准库工具、第三方包或 SGLang 运行时组件。

### Lines 2-3: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 4-4: Declares class Qwen3VLVisionConfig
```python
class Qwen3VLVisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLVisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3VLVisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 5-6: Declares model_type, base_config_key
```python
    model_type = "qwen3_vl"
    base_config_key = "vision_config"
```
**EN:** This block initializes a related set of values in the Qwen3VLVisionConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLVisionConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 7-7: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLVisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLVisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 8-39: Defines function Qwen3VLVisionConfig.__init__
```python
    def __init__(
        self,
        depth=27,
        hidden_size=1152,
        hidden_act="gelu_pytorch_tanh",
        intermediate_size=4304,
        num_heads=16,
        in_channels=3,
        patch_size=16,
        spatial_merge_size=2,
        temporal_patch_size=2,
        out_hidden_size=3584,
        num_position_embeddings=2304,
        deepstack_visual_indexes=[8, 16, 24],
        initializer_range=0.02,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.depth = depth
        self.hidden_size = hidden_size
        self.hidden_act = hidden_act
        self.intermediate_size = intermediate_size
        self.num_heads = num_heads
        self.in_channels = in_channels
        self.patch_size = patch_size
        self.spatial_merge_size = spatial_merge_size
        self.temporal_patch_size = temporal_patch_size
        self.out_hidden_size = out_hidden_size
        self.num_position_embeddings = num_position_embeddings
        self.initializer_range = initializer_range
        self.deepstack_visual_indexes = deepstack_visual_indexes
```
**EN:** This block defines function `Qwen3VLVisionConfig.__init__`. Parameters: self, depth, hidden_size, hidden_act, intermediate_size, num_heads, in_channels, patch_size, spatial_merge_size, temporal_patch_size, out_hidden_size, num_position_embeddings, deepstack_visual_indexes, initializer_range.
**CN:** 该代码块定义函数 `Qwen3VLVisionConfig.__init__`。 参数包括 self、depth、hidden_size、hidden_act、intermediate_size、num_heads、in_channels、patch_size、spatial_merge_size、temporal_patch_size、out_hidden_size、num_position_embeddings、deepstack_visual_indexes、initializer_range。

### Lines 40-41: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 42-42: Declares class Qwen3VLTextConfig
```python
class Qwen3VLTextConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLTextConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3VLTextModel`]. It is used to instantiate a Qwen3-VL model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3VLTextConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3VLTextModel`]. It is used to instantiate a Qwen3-VL model according to the specified arguments, defining the model architecture.

### Lines 43-50: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3VLTextModel`]. It is used to instantiate a
    Qwen3-VL model according to the specified arguments, defining the model architecture. Instantiating a configuration
    with the defaults will yield a similar configuration to that of
    Qwen3-VL-4B-Instruct [Qwen/Qwen3-VL-4B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct).

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.
```
**EN:** This string literal serves as documentation for the Qwen3VLTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 43-50 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 43-50 行。

### Lines 51-51: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 52-128: Documents the scope (part 2)
```python
    Args:
        vocab_size (`int`, *optional*, defaults to 151936):
            Vocabulary size of the Qwen3VL model. Defines the number of different tokens that can be represented by the
            `inputs_ids` passed when calling [`Qwen3VLModel`]
        hidden_size (`int`, *optional*, defaults to 4096):
            Dimension of the hidden representations.
        intermediate_size (`int`, *optional*, defaults to 22016):
            Dimension of the MLP representations.
        num_hidden_layers (`int`, *optional*, defaults to 32):
            Number of hidden layers in the Transformer encoder.
        num_attention_heads (`int`, *optional*, defaults to 32):
            Number of attention heads for each attention layer in the Transformer encoder.
        num_key_value_heads (`int`, *optional*, defaults to 32):
            This is the number of key_value heads that should be used to implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use Multi Head Attention (MHA), if
            `num_key_value_heads=1` the model will use Multi Query Attention (MQA) otherwise GQA is used. When
            converting a multi-head checkpoint to a GQA checkpoint, each group key and value head should be constructed
            by meanpooling all the original heads within that group. For more details, check out [this
            paper](https://huggingface.co/papers/2305.13245). If it is not specified, will default to `32`.
        head_dim (`int`, *optional*, defaults to 128):
            The dimension of the head. If not specified, will default to `hidden_size // num_attention_heads`.
        hidden_act (`str` or `function`, *optional*, defaults to `"silu"`):
            The non-linear activation function (function or string) in the decoder.
        max_position_embeddings (`int`, *optional*, defaults to 128000):
            The maximum sequence length that this model might ever be used with.
        initializer_range (`float`, *optional*, defaults to 0.02):
            The standard deviation of the truncated_normal_initializer for initializing all weight matrices.
        rms_norm_eps (`float`, *optional*, defaults to 1e-06):
            The epsilon used by the rms normalization layers.
        use_cache (`bool`, *optional*, defaults to `True`):
            Whether or not the model should return the last key/values attentions (not used by all models). Only
            relevant if `config.is_decoder=True`.
        tie_word_embeddings (`bool`, *optional*, defaults to `False`):
            Whether the model's input and output word embeddings should be tied.
        rope_theta (`float`, *optional*, defaults to 5000000.0):
            The base period of the RoPE embeddings.
        rope_scaling (`Dict`, *optional*):
            Dictionary containing the scaling configuration for the RoPE embeddings. NOTE: if you apply new rope type
            and you expect the model to work on longer `max_position_embeddings`, we recommend you to update this value
            accordingly.
            Expected contents:
                `rope_type` (`str`):
                    The sub-variant of RoPE to use. Can be one of ['default', 'linear', 'dynamic', 'yarn', 'longrope',
                    'llama3'], with 'default' being the original RoPE implementation.
                `factor` (`float`, *optional*):
                    Used with all rope types except 'default'. The scaling factor to apply to the RoPE embeddings. In
                    most scaling types, a `factor` of x will enable the model to handle sequences of length x *
                    original maximum pre-trained length.
                `original_max_position_embeddings` (`int`, *optional*):
                    Used with 'dynamic', 'longrope' and 'llama3'. The original max position embeddings used during
                    pretraining.
                `attention_factor` (`float`, *optional*):
                    Used with 'yarn' and 'longrope'. The scaling factor to be applied on the attention
                    computation. If unspecified, it defaults to value recommended by the implementation, using the
                    `factor` field to infer the suggested value.
                `beta_fast` (`float`, *optional*):
                    Only used with 'yarn'. Parameter to set the boundary for extrapolation (only) in the linear
                    ramp function. If unspecified, it defaults to 32.
                `beta_slow` (`float`, *optional*):
                    Only used with 'yarn'. Parameter to set the boundary for interpolation (only) in the linear
                    ramp function. If unspecified, it defaults to 1.
                `short_factor` (`list[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to short contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `long_factor` (`list[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to long contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `low_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to low frequency components of the RoPE
                `high_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to high frequency components of the RoPE
        attention_bias (`bool`, defaults to `False`, *optional*, defaults to `False`):
            Whether to use a bias in the query, key, value and output projection layers during self-attention.
        attention_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the attention probabilities.
```
**EN:** This string literal serves as documentation for the Qwen3VLTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 52-128 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 52-128 行。

### Lines 129-129: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 130-137: Documents the scope (part 3)
```python
    ```python
    >>> from transformers import Qwen3VLTextModel, Qwen3VLTextConfig

    >>> # Initializing a Qwen3VL style configuration
    >>> configuration = Qwen3VLTextConfig()

    >>> # Initializing a model from the Qwen3-VL-7B style configuration
    >>> model = Qwen3VLTextModel(configuration)
```
**EN:** This string literal serves as documentation for the Qwen3VLTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 130-137 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 130-137 行。

### Lines 138-138: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 139-141: Documents the scope (part 4)
```python
    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```"""
```
**EN:** This string literal serves as documentation for the Qwen3VLTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 139-141 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 139-141 行。

### Lines 142-142: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 143-144: Declares model_type, base_config_key
```python
    model_type = "qwen3_vl_text"
    base_config_key = "text_config"
```
**EN:** This block initializes a related set of values in the Qwen3VLTextConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLTextConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 145-145: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 146-189: Defines function Qwen3VLTextConfig.__init__
```python
    def __init__(
        self,
        vocab_size=151936,
        hidden_size=4096,
        intermediate_size=22016,
        num_hidden_layers=32,
        num_attention_heads=32,
        num_key_value_heads=32,
        head_dim=128,
        hidden_act="silu",
        max_position_embeddings=128000,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=5000000.0,
        rope_scaling=None,
        attention_bias=False,
        attention_dropout=0.0,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads

        # for backward compatibility
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads

        self.num_key_value_heads = num_key_value_heads
        self.head_dim = head_dim
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout

        super().__init__(tie_word_embeddings=tie_word_embeddings, **kwargs)
```
**EN:** This block defines function `Qwen3VLTextConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout.
**CN:** 该代码块定义函数 `Qwen3VLTextConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout。

### Lines 190-191: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 192-192: Declares class Qwen3VLConfig
```python
class Qwen3VLConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3VLModel`]. It is used to instantiate a Qwen3-VL model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3VLConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3VLModel`]. It is used to instantiate a Qwen3-VL model according to the specified arguments, defining the model architecture.

### Lines 193-230: Documents the scope
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3VLModel`]. It is used to instantiate a
    Qwen3-VL model according to the specified arguments, defining the model architecture. Instantiating a configuration
    with the defaults will yield a similar configuration to that of
    Qwen3-VL-4B-Instruct [Qwen/Qwen3-VL-4B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct).

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.


    Args:
        text_config (`Union[PreTrainedConfig, dict]`, *optional*, defaults to `Qwen3VLTextConfig`):
            The config object or dictionary of the text backbone.
        vision_config (`Union[PreTrainedConfig, dict]`,  *optional*, defaults to `Qwen3VLVisionConfig`):
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
    >>> from transformers import Qwen3VLForConditionalGeneration, Qwen3VLConfig

    >>> # Initializing a Qwen3-VL style configuration
    >>> configuration = Qwen3VLConfig()

    >>> # Initializing a model from the Qwen3-VL-4B style configuration
    >>> model = Qwen3VLForConditionalGeneration(configuration)

    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```"""
```
**EN:** This string literal serves as documentation for the Qwen3VLConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Qwen3VLConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 231-231: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 232-237: Declares model_type, sub_configs, keys_to_ignore_at_inference
```python
    model_type = "qwen3_vl"
    sub_configs = {
        "vision_config": Qwen3VLVisionConfig,
        "text_config": Qwen3VLTextConfig,
    }
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3VLConfig, including model_type, sub_configs, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLConfig 中初始化一组相关值，包括 model_type, sub_configs, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 238-238: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 239-264: Defines function Qwen3VLConfig.__init__
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
**EN:** This block defines function `Qwen3VLConfig.__init__`. Parameters: self, text_config, vision_config, image_token_id, video_token_id, vision_start_token_id, vision_end_token_id, tie_word_embeddings.
**CN:** 该代码块定义函数 `Qwen3VLConfig.__init__`。 参数包括 self、text_config、vision_config、image_token_id、video_token_id、vision_start_token_id、vision_end_token_id、tie_word_embeddings。

### Lines 265-266: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 267-267: Declares class Qwen3VLMoeTextConfig
```python
class Qwen3VLMoeTextConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLMoeTextConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3VLMoeTextModel`]. It is used to instantiate a Qwen3-VL-MOE model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3VLMoeTextConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3VLMoeTextModel`]. It is used to instantiate a Qwen3-VL-MOE model according to the specified arguments, defining the model architecture.

### Lines 268-275: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3VLMoeTextModel`]. It is used to instantiate a
    Qwen3-VL-MOE model according to the specified arguments, defining the model architecture. Instantiating a configuration
    with the defaults will yield a similar configuration to that of
    Qwen3-VL-30B-A3B-Instruct [Qwen/Qwen3-VL-30B-A3B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-30B-A3B-Instruct).

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.
```
**EN:** This string literal serves as documentation for the Qwen3VLMoeTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 268-275 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLMoeTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 268-275 行。

### Lines 276-276: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 277-356: Documents the scope (part 2)
```python
    Args:
        vocab_size (`int`, *optional*, defaults to 151936):
            Vocabulary size of the Qwen2MoE model. Defines the number of different tokens that can be represented by the
            `inputs_ids` passed when calling [`Qwen2MoeModel`]
        hidden_size (`int`, *optional*, defaults to 2048):
            Dimension of the hidden representations.
        intermediate_size (`int`, *optional*, defaults to 5632):
            Dimension of the MLP representations.
        num_hidden_layers (`int`, *optional*, defaults to 24):
            Number of hidden layers in the Transformer encoder.
        num_attention_heads (`int`, *optional*, defaults to 16):
            Number of attention heads for each attention layer in the Transformer encoder.
        num_key_value_heads (`int`, *optional*, defaults to 16):
            This is the number of key_value heads that should be used to implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use Multi Head Attention (MHA), if
            `num_key_value_heads=1` the model will use Multi Query Attention (MQA) otherwise GQA is used. When
            converting a multi-head checkpoint to a GQA checkpoint, each group key and value head should be constructed
            by meanpooling all the original heads within that group. For more details checkout [this
            paper](https://arxiv.org/pdf/2305.13245.pdf). If it is not specified, will default to `32`.
        hidden_act (`str` or `function`, *optional*, defaults to `"silu"`):
            The non-linear activation function (function or string) in the decoder.
        max_position_embeddings (`int`, *optional*, defaults to 128000):
            The maximum sequence length that this model might ever be used with.
        initializer_range (`float`, *optional*, defaults to 0.02):
            The standard deviation of the truncated_normal_initializer for initializing all weight matrices.
        rms_norm_eps (`float`, *optional*, defaults to 1e-06):
            The epsilon used by the rms normalization layers.
        use_cache (`bool`, *optional*, defaults to `True`):
            Whether or not the model should return the last key/values attentions (not used by all models). Only
            relevant if `config.is_decoder=True`.
        tie_word_embeddings (`bool`, *optional*, defaults to `False`):
            Whether the model's input and output word embeddings should be tied.
        rope_theta (`float`, *optional*, defaults to 5000000.0):
            The base period of the RoPE embeddings.
        attention_bias (`bool`, defaults to `False`, *optional*, defaults to `False`):
            Whether to use a bias in the query, key, value and output projection layers during self-attention.
        attention_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the attention probabilities.
        decoder_sparse_step (`int`, *optional*, defaults to 1):
            The frequency of the MoE layer.
        moe_intermediate_size (`int`, *optional*, defaults to 1408):
            Intermediate size of the routed expert.
        num_experts_per_tok (`int`, *optional*, defaults to 4):
            Number of selected experts.
        num_experts (`int`, *optional*, defaults to 60):
            Number of routed experts.
        norm_topk_prob (`bool`, *optional*, defaults to `True`):
            Whether to normalize the topk probabilities.
        mlp_only_layers (`List[int]`, *optional*, defaults to `[]`):
            Indicate which layers use Qwen3VLMoeMLP rather than Qwen3VLMoeSparseMoeBlock
            The list contains layer index, from 0 to num_layers-1 if we have num_layers layers
            If `mlp_only_layers` is empty, `decoder_sparse_step` is used to determine the sparsity.
        rope_scaling (`Dict`, *optional*):
            Dictionary containing the scaling configuration for the RoPE embeddings. NOTE: if you apply new rope type
            and you expect the model to work on longer `max_position_embeddings`, we recommend you to update this value
            accordingly.
            Expected contents:
                `rope_type` (`str`):
                    The sub-variant of RoPE to use. Can be one of ['default', 'linear', 'dynamic', 'yarn', 'longrope',
                    'llama3'], with 'default' being the original RoPE implementation.
                `factor` (`float`, *optional*):
                    Used with all rope types except 'default'. The scaling factor to apply to the RoPE embeddings. In
                    most scaling types, a `factor` of x will enable the model to handle sequences of length x *
                    original maximum pre-trained length.
                `original_max_position_embeddings` (`int`, *optional*):
                    Used with 'dynamic', 'longrope' and 'llama3'. The original max position embeddings used during
                    pretraining.
                `attention_factor` (`float`, *optional*):
                    Used with 'yarn' and 'longrope'. The scaling factor to be applied on the attention
                    computation. If unspecified, it defaults to value recommended by the implementation, using the
                    `factor` field to infer the suggested value.
                `beta_fast` (`float`, *optional*):
                    Only used with 'yarn'. Parameter to set the boundary for extrapolation (only) in the linear
                    ramp function. If unspecified, it defaults to 32.
                `beta_slow` (`float`, *optional*):
                    Only used with 'yarn'. Parameter to set the boundary for interpolation (only) in the linear
                    ramp function. If unspecified, it defaults to 1.
                `short_factor` (`List[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to short contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
```
**EN:** This string literal serves as documentation for the Qwen3VLMoeTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 277-356 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLMoeTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 277-356 行。

### Lines 357-376: Documents the scope (part 3)
```python
                    size divided by the number of attention heads divided by 2
                `long_factor` (`List[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to long contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `low_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to low frequency components of the RoPE
                `high_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to high frequency components of the RoPE
        head_dim (`int`, *optional*):
            The dimension of the head. If not specified, will default to `hidden_size // num_attention_heads`.

    ```python
    >>> from transformers import Qwen3VLMoeForConditionalGeneration, Qwen3VLMoeConfig

    >>> # Initializing a Qwen3VLMoe style configuration
    >>> configuration = Qwen3VLMoeConfig()

    >>> # Initializing a model from the Qwen3-VL-30B-A3B style configuration
    >>> model = Qwen3VLMoeForConditionalGeneration(configuration)
```
**EN:** This string literal serves as documentation for the Qwen3VLMoeTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 357-376 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLMoeTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 357-376 行。

### Lines 377-377: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 378-380: Documents the scope (part 4)
```python
    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```"""
```
**EN:** This string literal serves as documentation for the Qwen3VLMoeTextConfig, explaining intent or usage without affecting execution. This subsection covers lines 378-380 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3VLMoeTextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 378-380 行。

### Lines 381-381: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLMoeTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLMoeTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 382-384: Declares model_type, base_config_key, keys_to_ignore_at_inference
```python
    model_type = "qwen3_vl_moe_text"
    base_config_key = "text_config"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3VLMoeTextConfig, including model_type, base_config_key, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLMoeTextConfig 中初始化一组相关值，包括 model_type, base_config_key, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 385-385: Comments and module notes
```python
    # Default tensor parallel plan for base model `Qwen3VLMoe`
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the Qwen3VLMoeTextConfig.
**CN:** 该范围包含注释或说明，用于记录 Qwen3VLMoeTextConfig 的假设、来源或实现备注。

### Lines 386-399: Declares base_model_tp_plan, base_model_pp_plan
```python
    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
        "layers.*.self_attn.o_proj": "rowwise",
        "layers.*.mlp.gate_proj": "colwise",
        "layers.*.mlp.up_proj": "colwise",
        "layers.*.mlp.down_proj": "rowwise",
    }
    base_model_pp_plan = {
        "embed_tokens": (["input_ids"], ["inputs_embeds"]),
        "layers": (["hidden_states", "attention_mask"], ["hidden_states"]),
        "norm": (["hidden_states"], ["hidden_states"]),
    }
```
**EN:** This block initializes a related set of values in the Qwen3VLMoeTextConfig, including base_model_tp_plan, base_model_pp_plan. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLMoeTextConfig 中初始化一组相关值，包括 base_model_tp_plan, base_model_pp_plan。将这些赋值集中在一起有助于理解周边配置。

### Lines 400-400: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLMoeTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLMoeTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 401-458: Defines function Qwen3VLMoeTextConfig.__init__
```python
    def __init__(
        self,
        vocab_size=151936,
        hidden_size=2048,
        intermediate_size=5632,
        num_hidden_layers=24,
        num_attention_heads=16,
        num_key_value_heads=16,
        hidden_act="silu",
        max_position_embeddings=128000,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=5000000.0,
        attention_bias=False,
        attention_dropout=0.0,
        decoder_sparse_step=1,
        moe_intermediate_size=1408,
        num_experts_per_tok=4,
        num_experts=60,
        norm_topk_prob=True,
        mlp_only_layers=None,
        rope_scaling=None,
        head_dim=None,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads

        # for backward compatibility
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads

        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        self.rope_scaling = rope_scaling
        self.head_dim = head_dim or hidden_size // num_attention_heads

        # MoE arguments
        self.decoder_sparse_step = decoder_sparse_step
        self.moe_intermediate_size = moe_intermediate_size
        self.num_experts_per_tok = num_experts_per_tok
        self.num_experts = num_experts
        self.norm_topk_prob = norm_topk_prob
        self.mlp_only_layers = [] if mlp_only_layers is None else mlp_only_layers

        super().__init__(tie_word_embeddings=tie_word_embeddings, **kwargs)
```
**EN:** This block defines function `Qwen3VLMoeTextConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, attention_bias, attention_dropout, decoder_sparse_step, moe_intermediate_size, num_experts_per_tok, num_experts, norm_topk_prob, mlp_only_layers, rope_scaling, head_dim.
**CN:** 该代码块定义函数 `Qwen3VLMoeTextConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、attention_bias、attention_dropout、decoder_sparse_step、moe_intermediate_size、num_experts_per_tok、num_experts、norm_topk_prob、mlp_only_layers、rope_scaling、head_dim。

### Lines 459-460: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 461-461: Declares class Qwen3VLMoeVisionConfig
```python
class Qwen3VLMoeVisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLMoeVisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3VLMoeVisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 462-463: Declares model_type, base_config_key
```python
    model_type = "qwen3_vl_moe"
    base_config_key = "vision_config"
```
**EN:** This block initializes a related set of values in the Qwen3VLMoeVisionConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLMoeVisionConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 464-464: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLMoeVisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLMoeVisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 465-496: Defines function Qwen3VLMoeVisionConfig.__init__
```python
    def __init__(
        self,
        depth=27,
        hidden_size=1152,
        hidden_act="gelu_pytorch_tanh",
        intermediate_size=4304,
        num_heads=16,
        in_channels=3,
        patch_size=16,
        spatial_merge_size=2,
        temporal_patch_size=2,
        out_hidden_size=3584,
        num_position_embeddings=2304,
        deepstack_visual_indexes=[8, 16, 24],
        initializer_range=0.02,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.depth = depth
        self.hidden_size = hidden_size
        self.hidden_act = hidden_act
        self.intermediate_size = intermediate_size
        self.num_heads = num_heads
        self.in_channels = in_channels
        self.patch_size = patch_size
        self.spatial_merge_size = spatial_merge_size
        self.temporal_patch_size = temporal_patch_size
        self.out_hidden_size = out_hidden_size
        self.num_position_embeddings = num_position_embeddings
        self.initializer_range = initializer_range
        self.deepstack_visual_indexes = deepstack_visual_indexes
```
**EN:** This block defines function `Qwen3VLMoeVisionConfig.__init__`. Parameters: self, depth, hidden_size, hidden_act, intermediate_size, num_heads, in_channels, patch_size, spatial_merge_size, temporal_patch_size, out_hidden_size, num_position_embeddings, deepstack_visual_indexes, initializer_range.
**CN:** 该代码块定义函数 `Qwen3VLMoeVisionConfig.__init__`。 参数包括 self、depth、hidden_size、hidden_act、intermediate_size、num_heads、in_channels、patch_size、spatial_merge_size、temporal_patch_size、out_hidden_size、num_position_embeddings、deepstack_visual_indexes、initializer_range。

### Lines 497-498: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 499-499: Declares class Qwen3VLMoeConfig
```python
class Qwen3VLMoeConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3VLMoeConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3VLMoeModel`]. It is used to instantiate a Qwen3-VL-MOE model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3VLMoeConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3VLMoeModel`]. It is used to instantiate a Qwen3-VL-MOE model according to the specified arguments, defining the model architecture.

### Lines 500-537: Documents the scope
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3VLMoeModel`]. It is used to instantiate a
    Qwen3-VL-MOE model according to the specified arguments, defining the model architecture. Instantiating a configuration
    with the defaults will yield a similar configuration to that of
    Qwen3-VL-30B-A3B-Instruct [Qwen/Qwen3-VL-30B-A3B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-30B-A3B-Instruct).

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.


    Args:
        text_config (`Union[PreTrainedConfig, dict]`, *optional*, defaults to `Qwen3VLMoeTextConfig`):
            The config object or dictionary of the text backbone.
        vision_config (`Union[PreTrainedConfig, dict]`,  *optional*, defaults to `Qwen3VLMoeVisionConfig`):
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
    >>> from transformers import Qwen3VLMoeForConditionalGeneration, Qwen3VLMoeConfig

    >>> # Initializing a Qwen3-VL-MOE style configuration
    >>> configuration = Qwen3VLMoeConfig()

    >>> # Initializing a model from the Qwen3-VL-30B-A3B style configuration
    >>> model = Qwen3VLMoeForConditionalGeneration(configuration)

    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```"""
```
**EN:** This string literal serves as documentation for the Qwen3VLMoeConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Qwen3VLMoeConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 538-538: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLMoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLMoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 539-544: Declares model_type, sub_configs, keys_to_ignore_at_inference
```python
    model_type = "qwen3_vl_moe"
    sub_configs = {
        "vision_config": Qwen3VLMoeVisionConfig,
        "text_config": Qwen3VLMoeTextConfig,
    }
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3VLMoeConfig, including model_type, sub_configs, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3VLMoeConfig 中初始化一组相关值，包括 model_type, sub_configs, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 545-545: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3VLMoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3VLMoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 546-571: Defines function Qwen3VLMoeConfig.__init__
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
**EN:** This block defines function `Qwen3VLMoeConfig.__init__`. Parameters: self, text_config, vision_config, image_token_id, video_token_id, vision_start_token_id, vision_end_token_id, tie_word_embeddings.
**CN:** 该代码块定义函数 `Qwen3VLMoeConfig.__init__`。 参数包括 self、text_config、vision_config、image_token_id、video_token_id、vision_start_token_id、vision_end_token_id、tie_word_embeddings。

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen3VLVisionConfig`, `Qwen3VLTextConfig`, `Qwen3VLConfig`, `Qwen3VLMoeTextConfig`, `Qwen3VLMoeVisionConfig`, `Qwen3VLMoeConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers`
