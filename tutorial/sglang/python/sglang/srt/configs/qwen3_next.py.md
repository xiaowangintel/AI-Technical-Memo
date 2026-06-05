# qwen3_next.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/qwen3_next.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for qwen3 next so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 qwen3 next 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
```python
# coding=utf-8
# Copyright 2024 The Qwen team, Alibaba Group and the HuggingFace Inc. team. All rights reserved.
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

### Lines 15-15: Documents the scope
```python
"""Qwen3Hybrid model configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 16-16: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-28: Imports dependencies
```python
import enum

from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging

from sglang.srt.configs.mamba_utils import (
    Mamba2CacheParams,
    Mamba2StateShape,
    mamba2_state_dtype,
)
from sglang.srt.configs.update_config import adjust_tp_num_heads_if_necessary
from sglang.srt.utils import is_cpu
```
**EN:** This block groups related imports for the module, including enum, transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 enum, transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape 等 3 项，为后续代码准备所需名称。

### Lines 29-29: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 30-31: Declares logger, _is_cpu
```python
logger = logging.get_logger(__name__)
_is_cpu = is_cpu()
```
**EN:** This block initializes a related set of values in the module, including logger, _is_cpu. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 logger, _is_cpu。将这些赋值集中在一起有助于理解周边配置。

### Lines 32-33: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 34-34: Declares class HybridLayerType
```python
class HybridLayerType(enum.Enum):
```
**EN:** This block introduces class `HybridLayerType` as a reusable abstraction inside the module. It inherits from enum.Enum.
**CN:** 该代码块声明类 `HybridLayerType`，作为模块中的可复用抽象。 它继承自 enum.Enum。

### Lines 35-36: Declares full_attention, linear_attention
```python
    full_attention = "attention"
    linear_attention = "linear_attention"
```
**EN:** This block initializes a related set of values in the HybridLayerType, including full_attention, linear_attention. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 HybridLayerType 中初始化一组相关值，包括 full_attention, linear_attention。将这些赋值集中在一起有助于理解周边配置。

### Lines 37-38: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 39-39: Declares class Qwen3NextConfig
```python
class Qwen3NextConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3NextConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`Qwen3NextModel`]. It is used to instantiate a Qwen3-Next model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `Qwen3NextConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`Qwen3NextModel`]. It is used to instantiate a Qwen3-Next model according to the specified arguments, defining the model architecture.

### Lines 40-48: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a [`Qwen3NextModel`]. It is used to instantiate a
    Qwen3-Next model according to the specified arguments, defining the model architecture.
    Instantiating a configuration with the defaults will yield a similar configuration to that of
    Qwen3-Next-80B-A3B-Instruct [Qwen/Qwen3-Next-80B-A3B-Instruct](https://huggingface.co/Qwen/Qwen3-Next-80B-A3B-Instruct).

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.

```
**EN:** This string literal serves as documentation for the Qwen3NextConfig, explaining intent or usage without affecting execution. This subsection covers lines 40-48 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3NextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 40-48 行。

### Lines 49-49: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 50-129: Documents the scope (part 2)
```python
    Args:
        vocab_size (`int`, *optional*, defaults to 151936):
            Vocabulary size of the model. Defines the number of different tokens that can be represented by the
            `inputs_ids`.
        hidden_size (`int`, *optional*, defaults to 2048):
            Dimension of the hidden representations.
        intermediate_size (`int`, *optional*, defaults to 5632):
            Dimension of the MLP representations.
        num_hidden_layers (`int`, *optional*, defaults to 48):
            Number of hidden layers in the Transformer encoder.
        num_attention_heads (`int`, *optional*, defaults to 16):
            Number of attention heads for each attention layer in the Transformer encoder.
        num_key_value_heads (`int`, *optional*, defaults to 2):
            This is the number of key_value heads that should be used to implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use Multi Head Attention (MHA), if
            `num_key_value_heads=1` the model will use Multi Query Attention (MQA) otherwise GQA is used. When
            converting a multi-head checkpoint to a GQA checkpoint, each group key and value head should be constructed
            by meanpooling all the original heads within that group. For more details checkout [this
            paper](https://arxiv.org/pdf/2305.13245.pdf). If it is not specified, will default to `32`.
        hidden_act (`str`, *optional*, defaults to `"silu"`):
            The non-linear activation function in the decoder.
        max_position_embeddings (`int`, *optional*, defaults to 32768):
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
        rope_theta (`float`, *optional*, defaults to 10000.0):
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
                `short_factor` (`List[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to short contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `long_factor` (`List[float]`, *optional*):
                    Only used with 'longrope'. The scaling factor to be applied to long contexts (<
                    `original_max_position_embeddings`). Must be a list of numbers with the same length as the hidden
                    size divided by the number of attention heads divided by 2
                `low_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to low frequency components of the RoPE
                `high_freq_factor` (`float`, *optional*):
                    Only used with 'llama3'. Scaling factor applied to high frequency components of the RoPE
        partial_rotary_factor (`float`, *optional*, defaults to 0.25):
            Percentage of the query and keys which will have rotary embedding.
        attention_bias (`bool`, *optional*, defaults to `False`):
            Whether to use a bias in the query, key, value and output projection layers during self-attention.
        attention_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the attention probabilities.
        head_dim (`int`, *optional*, defaults to 256):
            Projection weights dimension in multi-head attention.
        linear_conv_kernel_dim (`int`, *optional*, defaults to 4):
```
**EN:** This string literal serves as documentation for the Qwen3NextConfig, explaining intent or usage without affecting execution. This subsection covers lines 50-129 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3NextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 50-129 行。

### Lines 130-170: Documents the scope (part 3)
```python
            Kernel size of the convolution used in linear attention layers.
        linear_key_head_dim (`int`, *optional*, defaults to 128):
            Dimension of each key head in linear attention.
        linear_value_head_dim (`int`, *optional*, defaults to 128):
            Dimension of each value head in linear attention.
        linear_num_key_heads (`int`, *optional*, defaults to 16):
            Number of key heads used in linear attention layers.
        linear_num_value_heads (`int`, *optional*, defaults to 32):
            Number of value heads used in linear attention layers.
        decoder_sparse_step (`int`, *optional*, defaults to 1):
            The frequency of the MoE layer.
        moe_intermediate_size (`int`, *optional*, defaults to 512):
            Intermediate size of the routed expert.
        shared_expert_intermediate_size (`int`, *optional*, defaults to 512):
            Intermediate size of the shared expert.
        num_experts_per_tok (`int`, *optional*, defaults to 10):
            Number of selected experts.
        num_experts (`int`, *optional*, defaults to 512):
            Number of routed experts.
        norm_topk_prob (`bool`, *optional*, defaults to `True`):
            Whether to normalize the topk probabilities.
        output_router_logits (`bool`, *optional*, defaults to `False`):
            Whether or not the router logits should be returned by the model. Enabling this will also
            allow the model to output the auxiliary loss, including load balancing loss and router z-loss.
        router_aux_loss_coef (`float`, *optional*, defaults to 0.001):
            The aux loss factor for the total loss.
        mlp_only_layers (`list[int]`, *optional*, defaults to `[]`):
            Indicate which layers use Qwen3NextMLP rather than Qwen3NextSparseMoeBlock
            The list contains layer index, from 0 to num_layers-1 if we have num_layers layers
            If `mlp_only_layers` is empty, `decoder_sparse_step` is used to determine the sparsity.
        layer_types (`list[str]`, *optional*, defaults to None):
            Types of each layer (attention or linear).

    ```python
    >>> from transformers import Qwen3NextModel, Qwen3NextConfig

    >>> # Initializing a Qwen3Next style configuration
    >>> configuration =  Qwen3NextConfig()

    >>> # Initializing a model from the Qwen3-Next-80B-A3B style configuration
    >>> model = Qwen3NextModel(configuration)
```
**EN:** This string literal serves as documentation for the Qwen3NextConfig, explaining intent or usage without affecting execution. This subsection covers lines 130-170 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3NextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 130-170 行。

### Lines 171-171: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 172-175: Documents the scope (part 4)
```python
    >>> # Accessing the model configuration
    >>> configuration = model.config
    ```
    """
```
**EN:** This string literal serves as documentation for the Qwen3NextConfig, explaining intent or usage without affecting execution. This subsection covers lines 172-175 of the same logical block.
**CN:** 该字符串字面量作为 Qwen3NextConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 172-175 行。

### Lines 176-176: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 177-178: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "qwen3_next"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3NextConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3NextConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 179-179: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 180-252: Defines function Qwen3NextConfig.__init__
```python
    def __init__(
        self,
        vocab_size=151936,
        hidden_size=2048,
        intermediate_size=5632,
        num_hidden_layers=48,
        num_attention_heads=16,
        num_key_value_heads=2,
        hidden_act="silu",
        max_position_embeddings=32768,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=10000.0,
        rope_scaling=None,
        partial_rotary_factor=0.25,
        attention_bias=False,
        attention_dropout=0.0,
        head_dim=256,
        linear_conv_kernel_dim=4,
        linear_key_head_dim=128,
        linear_value_head_dim=128,
        linear_num_key_heads=16,
        linear_num_value_heads=32,
        decoder_sparse_step=1,
        moe_intermediate_size=512,
        shared_expert_intermediate_size=512,
        num_experts_per_tok=10,
        num_experts=512,
        norm_topk_prob=True,
        output_router_logits=False,
        router_aux_loss_coef=0.001,
        mlp_only_layers=[],
        layer_types=None,
        **kwargs,
    ):
        super().__init__(tie_word_embeddings=tie_word_embeddings, **kwargs)
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.partial_rotary_factor = partial_rotary_factor
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        self.head_dim = head_dim

        # linear attention (gdn now part)
        self.linear_conv_kernel_dim = linear_conv_kernel_dim
        self.linear_key_head_dim = linear_key_head_dim
        self.linear_value_head_dim = linear_value_head_dim
        self.linear_num_key_heads = linear_num_key_heads
        self.linear_num_value_heads = linear_num_value_heads

        # MoE arguments
        self.decoder_sparse_step = decoder_sparse_step
        self.moe_intermediate_size = moe_intermediate_size
        self.shared_expert_intermediate_size = shared_expert_intermediate_size
        self.num_experts_per_tok = num_experts_per_tok
        self.num_experts = num_experts
        self.norm_topk_prob = norm_topk_prob
        self.output_router_logits = output_router_logits
        self.router_aux_loss_coef = router_aux_loss_coef
        self.mlp_only_layers = mlp_only_layers
```
**EN:** This block defines function `Qwen3NextConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, rope_scaling, partial_rotary_factor, attention_bias, attention_dropout, head_dim, linear_conv_kernel_dim, linear_key_head_dim, linear_value_head_dim, linear_num_key_heads, linear_num_value_heads, decoder_sparse_step, moe_intermediate_size, shared_expert_intermediate_size, num_experts_per_tok, num_experts, norm_topk_prob, output_router_logits, router_aux_loss_coef, mlp_only_layers, layer_types.
**CN:** 该代码块定义函数 `Qwen3NextConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、rope_scaling、partial_rotary_factor、attention_bias、attention_dropout、head_dim、linear_conv_kernel_dim、linear_key_head_dim、linear_value_head_dim、linear_num_key_heads、linear_num_value_heads、decoder_sparse_step、moe_intermediate_size、shared_expert_intermediate_size、num_experts_per_tok、num_experts、norm_topk_prob、output_router_logits、router_aux_loss_coef、mlp_only_layers、layer_types。

### Lines 253-253: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 254-264: Defines function Qwen3NextConfig.layers_block_type
```python
    @property
    def layers_block_type(self):
        layer_type_list = []

        for l in range(self.num_hidden_layers):
            if (l + 1) % self.full_attention_interval == 0:
                layer_type_list.append(HybridLayerType.full_attention.value)
            else:
                layer_type_list.append(HybridLayerType.linear_attention.value)

        return layer_type_list
```
**EN:** This block defines function `Qwen3NextConfig.layers_block_type`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `Qwen3NextConfig.layers_block_type`。 参数包括 self。 装饰器包括 property。

### Lines 265-265: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 266-272: Defines function Qwen3NextConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self):
        return [
            i
            for i, type_value in enumerate(self.layers_block_type)
            if type_value == HybridLayerType.linear_attention.value
        ]
```
**EN:** This block defines function `Qwen3NextConfig.linear_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `Qwen3NextConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 273-273: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 274-280: Defines function Qwen3NextConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self):
        return [
            i
            for i, type_value in enumerate(self.layers_block_type)
            if type_value == HybridLayerType.full_attention.value
        ]
```
**EN:** This block defines function `Qwen3NextConfig.full_attention_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `Qwen3NextConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 281-281: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3NextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3NextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 282-302: Defines function Qwen3NextConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Mamba2CacheParams:
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        if _is_cpu:
            world_size = get_attention_tp_size()
            adjust_tp_num_heads_if_necessary(self, world_size, False)

        shape = Mamba2StateShape.create(
            tp_world_size=get_attention_tp_size(),
            intermediate_size=self.linear_value_head_dim * self.linear_num_value_heads,
            n_groups=self.linear_num_key_heads,
            num_heads=self.linear_num_value_heads,
            head_dim=self.linear_value_head_dim,
            state_size=self.linear_key_head_dim,
            conv_kernel=self.linear_conv_kernel_dim,
        )

        return Mamba2CacheParams(
            shape=shape, layers=self.linear_layer_ids, dtype=mamba2_state_dtype(self)
        )
```
**EN:** This block defines function `Qwen3NextConfig.mamba2_cache_params`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `Qwen3NextConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。

## Key Concepts / 关键概念
- **Classes / 类**: `HybridLayerType`, `Qwen3NextConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `enum`
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.configs.update_config`, `sglang.srt.layers.dp_attention`, `sglang.srt.utils`
