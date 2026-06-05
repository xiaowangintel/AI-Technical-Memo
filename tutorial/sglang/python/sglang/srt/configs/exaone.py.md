# exaone.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/exaone.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for exaone so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 exaone 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Comments and module notes
```python
# coding=utf-8
# Copyright 2024 The LG AI Research EXAONE Lab. All rights reserved.
# Copyright 2024 The LG CNS AI Engineering Team.
# Copyright 2023-2024 SGLang Team.
#
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
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 17-17: Documents the scope
```python
"""EXAONE model configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 18-18: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 19-22: Imports dependencies
```python
from typing import Any, Dict

from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging
```
**EN:** This block groups related imports for the module, including typing.Any, typing.Dict, transformers.configuration_utils.PretrainedConfig, transformers.utils.logging. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Any, typing.Dict, transformers.configuration_utils.PretrainedConfig, transformers.utils.logging，为后续代码准备所需名称。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-24: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-26: Declares EXAONE_PRETRAINED_CONFIG_ARCHIVE_MAP
```python
EXAONE_PRETRAINED_CONFIG_ARCHIVE_MAP: Dict[str, Any] = {}
```
**EN:** This statement initializes EXAONE_PRETRAINED_CONFIG_ARCHIVE_MAP in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 EXAONE_PRETRAINED_CONFIG_ARCHIVE_MAP。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 27-29: Comments and module notes
```python


# ruff: noqa: E501
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 30-30: Declares class ExaoneConfig
```python
class ExaoneConfig(PretrainedConfig):
```
**EN:** This block introduces class `ExaoneConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a :class:`~transformers.ExaoneModel`. It is used to instantiate a EXAONE model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `ExaoneConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a :class:`~transformers.ExaoneModel`. It is used to instantiate a EXAONE model according to the specified arguments, defining the model architecture.

### Lines 31-38: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a :class:`~transformers.ExaoneModel`. It is used to
    instantiate a EXAONE model according to the specified arguments, defining the model architecture. Instantiating a
    configuration with the defaults will yield a similar configuration to that of the Exaone

    Configuration objects inherit from :class:`~transformers.PretrainedConfig` and can be used to control the model
    outputs. Read the documentation from :class:`~transformers.PretrainedConfig` for more information.

```
**EN:** This string literal serves as documentation for the ExaoneConfig, explaining intent or usage without affecting execution. This subsection covers lines 31-38 of the same logical block.
**CN:** 该字符串字面量作为 ExaoneConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 31-38 行。

### Lines 39-39: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 40-119: Documents the scope (part 2)
```python
    Args:
        vocab_size (:obj:`int`, `optional`, defaults to 102400):
            Vocabulary size of the EXAONE model. Defines the number of different tokens that can be represented by the
            :obj:`inputs_ids` passed when calling :class:`~transformers.ExaoneModel`. Vocabulary size of the model.
            Defines the different tokens that can be represented by the `inputs_ids` passed to the forward method of
            :class:`~transformers.EXAONEModel`.
        max_position_embeddings (:obj:`int`, `optional`, defaults to 2048):
            The maximum sequence length that this model might ever be used with. Typically set this to something large
            just in case (e.g., 512 or 1024 or 2048).
        hidden_size (:obj:`int`, `optional`, defaults to 2048):
            Dimensionality of the encoder layers and the pooler layer.
        num_layers (:obj:`int`, `optional`, defaults to 32):
            Number of hidden layers in the Transformer encoder.
        num_attention_heads (:obj:`int`, `optional`, defaults to 32):
            Number of attention heads for each attention layer in the Transformer decoder.
        num_key_value_heads (:obj:`int`, `optional`):
            This is the number of key_value heads that should be used to implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use Multi Head Attention (MHA), if
            `num_key_value_heads=1 the model will use Multi Query Attention (MQA) otherwise GQA is used. When
            converting a multi-head checkpoint to a GQA checkpoint, each group key and value head should be constructed
            by meanpooling all the original heads within that group. For more details checkout [this
            paper](https://arxiv.org/pdf/2305.13245.pdf). If it is not specified, will default to
            `num_attention_heads`.
        intermediate_size (:obj:`int`, `optional`, defaults to `hidden_size * 4`):
            Dimensionality of the "intermediate" (i.e., feed-forward) layer in the Transformer encoder.
        activation_function (:obj:`str` or :obj:`function`, `optional`, defaults to :obj:`"silu"`):
            The non-linear activation function (function or string) in the decoder.
        rope_theta (:obj:`float`, `optional`, defaults to 10000.0):
            The base period of the RoPE embeddings.
        rope_scaling (:obj:`Dict`, `optional`):
            Dictionary containing the scaling configuration for the RoPE embeddings. NOTE: if you apply new rope type
            and you expect the model to work on longer `max_position_embeddings`, we recommend you to update this value
            accordingly.
            Expected contents:
                `rope_type` (:obj:`str`):
                    The sub-variant of RoPE to use. Can be one of ['default', 'linear', 'dynamic', 'yarn', 'longrope',
                    'llama3'], with 'default' being the original RoPE implementation.
                `factor` (:obj:`float`, `optional`):
                    Used with all rope types except 'default'. The scaling factor to apply to the RoPE embeddings. In
                    most scaling types, a `factor` of x will enable the model to handle sequences of length x *
                    original maximum pre-trained length.
                `original_max_position_embeddings` (:obj:`int`, `optional`):
                    Used with 'dynamic', 'longrope' and 'llama3'. The original max position embeddings used during
                    pretraining.
                `attention_factor` (:obj:`float`, `optional`):
                    Used with 'yarn' and 'longrope'. The scaling factor to be applied on the attention
                    computation. If unspecified, it defaults to value recommended by the implementation, using the
                    `factor` field to infer the suggested value.
                `beta_fast` (:obj:`float`, `optional`):
                    Only used with 'yarn'. Parameter to set the boundary for extrapolation (only) in the linear
                    ramp function. If unspecified, it defaults to 32.
                `beta_slow` (:obj:`float`, `optional`):
                    Only used with 'yarn'. Parameter to set the boundary for interpolation (only) in the linear
                    ramp function. If unspecified, it defaults to 1.
                `short_factor` (:obj:`List[float]`, `optional`):
                    Only used with 'longrope'. The scaling factor to be applied to short contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `long_factor` (:obj:`List[float]`, `optional`):
                    Only used with 'longrope'. The scaling factor to be applied to long contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `low_freq_factor` (:obj:`float`, `optional`):
                    Only used with 'llama3'. Scaling factor applied to low frequency components of the RoPE
                `high_freq_factor` (:obj:`float`, `optional`):
                    Only used with 'llama3'. Scaling factor applied to high frequency components of the RoPE
        embed_dropout (:obj:`float`, `optional`, defaults to 0.0):
            The dropout probabilitiy for all fully connected layers in the embeddings, encoder, and pooler.
        attention_dropout (:obj:`float`, `optional`, defaults to 0.0):
            The dropout ratio for the attention probabilities.
        layer_norm_epsilon (:obj:`float`, `optional`, defaults to 1e-5):
            The epsilon used by the layer normalization layers.
        initializer_range (:obj:`float`, `optional`, defaults to 0.02):
            The standard deviation of the truncated_normal_initializer for initializing all weight matrices.
        use_cache (:obj:`bool`, `optional`, defaults to :obj:`True`):
            Whether or not the model should return the last key/values attentions (not used by all models). Only
            relevant if ``configs.is_decoder=True``.
        bos_token_id (:obj:`int`, `optional`, defaults to 0):
            Beginning of stream token id.
        eos_token_id (:obj:`int`, `optional`, defaults to 2):
```
**EN:** This string literal serves as documentation for the ExaoneConfig, explaining intent or usage without affecting execution. This subsection covers lines 40-119 of the same logical block.
**CN:** 该字符串字面量作为 ExaoneConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 40-119 行。

### Lines 120-134: Documents the scope (part 3)
```python
            End of stream token id.
        tie_word_embeddings (:obj:`bool`, `optional`, defaults to :obj:`True`):
            Whether to tie weight embeddings
        gradient_checkpointing (:obj:`bool`, `optional`, defaults to :obj:`False`):
            If True, use gradient checkpointing to save memory at the expense of slower backward pass.

        Example::

            >>> from transformers import EXAONEModel, ExaoneConfig

            >>> # Initializing a EXAONE configuration
            >>> configuration = ExaoneConfig()

            >>> # Initializing a model from configuration
            >>> model = EXAONEModel(configuration)
```
**EN:** This string literal serves as documentation for the ExaoneConfig, explaining intent or usage without affecting execution. This subsection covers lines 120-134 of the same logical block.
**CN:** 该字符串字面量作为 ExaoneConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 120-134 行。

### Lines 135-135: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 136-138: Documents the scope (part 4)
```python
            >>> # Accessing the model configuration
            >>> configuration = model.configs
    """
```
**EN:** This string literal serves as documentation for the ExaoneConfig, explaining intent or usage without affecting execution. This subsection covers lines 136-138 of the same logical block.
**CN:** 该字符串字面量作为 ExaoneConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 136-138 行。

### Lines 139-139: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ExaoneConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ExaoneConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 140-142: Declares model_type, keys_to_ignore_at_inference, attribute_map
```python
    model_type = "exaone"
    keys_to_ignore_at_inference = ["past_key_values"]
    attribute_map = {"num_hidden_layers": "num_layers"}
```
**EN:** This block initializes a related set of values in the ExaoneConfig, including model_type, keys_to_ignore_at_inference, attribute_map. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 ExaoneConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference, attribute_map。将这些赋值集中在一起有助于理解周边配置。

### Lines 143-143: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ExaoneConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ExaoneConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 144-196: Defines function ExaoneConfig.__init__
```python
    def __init__(
        self,
        vocab_size=102400,
        max_position_embeddings=2048,
        hidden_size=2048,
        num_layers=32,
        num_attention_heads=32,
        num_key_value_heads=None,
        intermediate_size=None,
        activation_function="silu",
        rope_theta=10000.0,
        rope_scaling=None,
        embed_dropout=0.0,
        attention_dropout=0.0,
        layer_norm_epsilon=1e-5,
        initializer_range=0.02,
        use_cache=True,
        bos_token_id=0,
        eos_token_id=2,
        tie_word_embeddings=True,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.num_attention_heads = num_attention_heads
        self.num_hidden_layers = num_layers
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads
        self.num_key_value_heads = num_key_value_heads
        if intermediate_size:
            self.intermediate_size = intermediate_size
        else:
            self.intermediate_size = hidden_size * 4
        self.activation_function = activation_function
        self.embed_dropout = embed_dropout
        self.attention_dropout = attention_dropout
        self.layer_norm_epsilon = layer_norm_epsilon
        self.initializer_range = initializer_range
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling

        self.bos_token_id = bos_token_id
        self.eos_token_id = eos_token_id

        super().__init__(
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `ExaoneConfig.__init__`. Parameters: self, vocab_size, max_position_embeddings, hidden_size, num_layers, num_attention_heads, num_key_value_heads, intermediate_size, activation_function, rope_theta, rope_scaling, embed_dropout, attention_dropout, layer_norm_epsilon, initializer_range, use_cache, bos_token_id, eos_token_id, tie_word_embeddings.
**CN:** 该代码块定义函数 `ExaoneConfig.__init__`。 参数包括 self、vocab_size、max_position_embeddings、hidden_size、num_layers、num_attention_heads、num_key_value_heads、intermediate_size、activation_function、rope_theta、rope_scaling、embed_dropout、attention_dropout、layer_norm_epsilon、initializer_range、use_cache、bos_token_id、eos_token_id、tie_word_embeddings。

## Key Concepts / 关键概念
- **Classes / 类**: `ExaoneConfig`
- **Constants / 常量**: `EXAONE_PRETRAINED_CONFIG_ARCHIVE_MAP`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
