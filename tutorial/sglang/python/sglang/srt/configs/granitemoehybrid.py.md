# granitemoehybrid.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/granitemoehybrid.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for granitemoehybrid so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 granitemoehybrid 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
```python
# coding=utf-8
# Copyright 2025 IBM and the HuggingFace Inc. team. All rights reserved.
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
"""GraniteMoeHybrid model configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 16-16: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-20: Imports dependencies
```python
from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging

from sglang.srt.configs.mamba_utils import Mamba2CacheParams, Mamba2StateShape
```
**EN:** This block groups related imports for the module, including transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape，为后续代码准备所需名称。

### Lines 21-21: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 22-22: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-25: Declares MAMBA, ATTENTION
```python
MAMBA = "mamba"
ATTENTION = "attention"
```
**EN:** This block initializes a related set of values in the module, including MAMBA, ATTENTION. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 MAMBA, ATTENTION。将这些赋值集中在一起有助于理解周边配置。

### Lines 26-27: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-28: Declares class GraniteMoeHybridConfig
```python
class GraniteMoeHybridConfig(PretrainedConfig):
```
**EN:** This block introduces class `GraniteMoeHybridConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`GraniteMoeHybridModel`]. It is used to instantiate a GraniteMoeHybrid model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `GraniteMoeHybridConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`GraniteMoeHybridModel`]. It is used to instantiate a GraniteMoeHybrid model according to the specified arguments, defining the model architecture.

### Lines 29-35: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a [`GraniteMoeHybridModel`]. It is used to instantiate a
    GraniteMoeHybrid model according to the specified arguments, defining the model architecture. The GraniteMoeHybrid is a
    hybrid architecture combining Mamba2 layers with attention layers, developed by IBM.

    Configuration objects inherit from [`PretrainedConfig`] and can be used to control the model outputs. Read the
    documentation from [`PretrainedConfig`] for more information.
```
**EN:** This string literal serves as documentation for the GraniteMoeHybridConfig, explaining intent or usage without affecting execution. This subsection covers lines 29-35 of the same logical block.
**CN:** 该字符串字面量作为 GraniteMoeHybridConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 29-35 行。

### Lines 36-36: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 37-116: Documents the scope (part 2)
```python
    Args:
        vocab_size (`int`, *optional*, defaults to 100352):
            Vocabulary size of the GraniteMoeHybrid model. Defines the number of different tokens that can be represented by the
            `inputs_ids` passed when calling [`GraniteMoeHybridModel`]
        tie_word_embeddings (`bool`, *optional*, defaults to `True`):
            Whether the model's input and output word embeddings should be tied. Note that this is only relevant if the
            model has a output word embedding layer.
        hidden_size (`int`, *optional*, defaults to 2048):
            Dimension of the hidden representations.
        intermediate_size (`int`, *optional*, defaults to 8192):
            Dimension of the MLP representations.
        num_hidden_layers (`int`, *optional*, defaults to 40):
            Number of hidden layers in the model.
        layer_types (`list[str]`, *optional*):
            List of layer types for each layer. Each element should be either "mamba" or "attention".
            If not provided, defaults to alternating pattern based on num_hidden_layers.
        num_attention_heads (`int`, *optional*, defaults to 32):
            Number of attention heads for each attention layer.
        num_key_value_heads (`int`, *optional*, defaults to 8):
            This is the number of key_value heads that should be used to implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use Multi Head Attention (MHA), if
            `num_key_value_heads=1` the model will use Multi Query Attention (MQA) otherwise GQA is used.
        hidden_act (`str` or `function`, *optional*, defaults to `"silu"`):
            The non-linear activation function (function or string) in the decoder.
        initializer_range (`float`, *optional*, defaults to 0.1):
            The standard deviation of the truncated_normal_initializer for initializing all weight matrices.
        rms_norm_eps (`float`, *optional*, defaults to 1e-05):
            The epsilon used by the rms normalization layers.
        normalization_function (`str`, *optional*, defaults to `"rmsnorm"`):
            The normalization function to use. Currently only "rmsnorm" is supported.
        use_cache (`bool`, *optional*, defaults to `True`):
            Whether or not the model should return the last key/values attentions (not used by all models). Only
            relevant if `config.is_decoder=True`.
        pad_token_id (`int`, *optional*, defaults to 100256):
            The id of the padding token.
        bos_token_id (`int`, *optional*, defaults to 100257):
            The id of the "beginning-of-sequence" token.
        eos_token_id (`int`, *optional*, defaults to 100257):
            The id of the "end-of-sequence" token.
        max_position_embeddings (`int`, *optional*, defaults to 131072):
            Max cached sequence length for the model
        attention_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the attention probabilities.
        attention_bias (`bool`, *optional*, defaults to `False`):
            Whether to use bias in attention layers.
        position_embedding_type (`str`, *optional*, defaults to `"nope"`):
            Type of position embedding. Can be "nope" (no position embedding) or "rope".
        rope_theta (`float`, *optional*, defaults to 10000.0):
            The theta value used for the RoPE embeddings.
        rope_scaling (`dict`, *optional*):
            The scaling configuration for the RoPE embeddings. If `None`, no scaling is applied.
        mamba_d_state (`int`, *optional*, defaults to 128):
            The dimension of the mamba state space latents
        mamba_d_conv (`int`, *optional*, defaults to 4):
            The size of the mamba convolution kernel
        mamba_expand (`int`, *optional*, defaults to 2):
            Expanding factor (relative to hidden_size) used to determine the mamba intermediate size
        mamba_d_head (`int`, *optional*, defaults to 64):
            Head embedding dimension size for Mamba
        mamba_n_heads (`int`, *optional*, defaults to 64):
            The number of mamba heads
        mamba_n_groups (`int`, *optional*, defaults to 1):
            The number of the mamba groups
        mamba_chunk_size (`int`, *optional*, defaults to 256):
            The chunks in which to break the sequence when doing prefill/training
        mamba_conv_bias (`bool`, *optional*, defaults to `True`):
            Flag indicating whether or not to use bias in the convolution layer of the mamba mixer block.
        mamba_proj_bias (`bool`, *optional*, defaults to `False`):
            Flag indicating whether or not to use bias in the input and output projections of the mamba mixer block
        embedding_multiplier (`float`, *optional*, defaults to 12.0):
            The multiplier for the embedding layer. This is used to scale the output of the embedding layer.
        logits_scaling (`float`, *optional*, defaults to 8.0):
            The scaling factor for the logits.
        attention_multiplier (`float`, *optional*, defaults to 0.015625):
            The multiplier for the attention layers.
        residual_multiplier (`float`, *optional*, defaults to 0.22):
            The multiplier for residual connections.
        num_local_experts (`int`, *optional*, defaults to 0):
            Number of local experts in MoE layers.
        num_experts_per_tok (`int`, *optional*, defaults to 0):
```
**EN:** This string literal serves as documentation for the GraniteMoeHybridConfig, explaining intent or usage without affecting execution. This subsection covers lines 37-116 of the same logical block.
**CN:** 该字符串字面量作为 GraniteMoeHybridConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 37-116 行。

### Lines 117-124: Documents the scope (part 3)
```python
            Number of experts to use per token in MoE layers.
        shared_intermediate_size (`int`, *optional*, defaults to 8192):
            Intermediate size for shared experts.
        output_router_logits (`bool`, *optional*, defaults to `False`):
            Whether to output router logits.
        router_aux_loss_coef (`float`, *optional*, defaults to 0.01):
            Auxiliary loss coefficient for the router.
    """
```
**EN:** This string literal serves as documentation for the GraniteMoeHybridConfig, explaining intent or usage without affecting execution. This subsection covers lines 117-124 of the same logical block.
**CN:** 该字符串字面量作为 GraniteMoeHybridConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 117-124 行。

### Lines 125-125: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 126-127: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "granitemoehybrid"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the GraniteMoeHybridConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 GraniteMoeHybridConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 128-128: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 129-202: Defines function GraniteMoeHybridConfig.__init__ (part 1)
```python
    def __init__(
        self,
        vocab_size=100352,
        tie_word_embeddings=True,
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=40,
        layer_types=None,
        num_attention_heads=32,
        num_key_value_heads=8,
        hidden_act="silu",
        initializer_range=0.1,
        rms_norm_eps=1e-5,
        normalization_function="rmsnorm",
        use_cache=True,
        pad_token_id=100256,
        bos_token_id=100257,
        eos_token_id=100257,
        max_position_embeddings=131072,
        attention_dropout=0.0,
        attention_bias=False,
        position_embedding_type="nope",
        rope_theta=10000.0,
        rope_scaling=None,
        mamba_d_state=128,
        mamba_d_conv=4,
        mamba_expand=2,
        mamba_d_head=64,
        mamba_n_heads=64,
        mamba_n_groups=1,
        mamba_chunk_size=256,
        mamba_conv_bias=True,
        mamba_proj_bias=False,
        embedding_multiplier=12.0,
        logits_scaling=8.0,
        attention_multiplier=0.015625,
        residual_multiplier=0.22,
        num_local_experts=0,
        num_experts_per_tok=0,
        shared_intermediate_size=8192,
        output_router_logits=False,
        router_aux_loss_coef=0.01,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers

        # Set layer types - if not provided, create default pattern
        if layer_types is None:
            # Default pattern: mamba layers with attention every 6th layer (roughly)
            self.layer_types = []
            for i in range(num_hidden_layers):
                if (i + 1) % 6 == 0:
                    self.layer_types.append(ATTENTION)
                else:
                    self.layer_types.append(MAMBA)
        else:
            self.layer_types = layer_types

        # Validate layer_types
        if len(self.layer_types) != self.num_hidden_layers:
            raise ValueError(
                f"layer_types must have length equal to num_hidden_layers ({num_hidden_layers}), "
                f"but got {len(self.layer_types)}"
            )

        for layer_type in self.layer_types:
            if layer_type not in [MAMBA, ATTENTION]:
                raise ValueError(
                    f"Each element in layer_types must be either '{MAMBA}' or '{ATTENTION}', "
                    f"but got '{layer_type}'"
                )
```
**EN:** This block defines function `GraniteMoeHybridConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, layer_types, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, normalization_function, use_cache, pad_token_id, bos_token_id, eos_token_id, max_position_embeddings, attention_dropout, attention_bias, position_embedding_type, rope_theta, rope_scaling, mamba_d_state, mamba_d_conv, mamba_expand, mamba_d_head, mamba_n_heads, mamba_n_groups, mamba_chunk_size, mamba_conv_bias, mamba_proj_bias, embedding_multiplier, logits_scaling, attention_multiplier, residual_multiplier, num_local_experts, num_experts_per_tok, shared_intermediate_size, output_router_logits, router_aux_loss_coef. This subsection covers lines 129-202 of the same logical block.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、layer_types、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、normalization_function、use_cache、pad_token_id、bos_token_id、eos_token_id、max_position_embeddings、attention_dropout、attention_bias、position_embedding_type、rope_theta、rope_scaling、mamba_d_state、mamba_d_conv、mamba_expand、mamba_d_head、mamba_n_heads、mamba_n_groups、mamba_chunk_size、mamba_conv_bias、mamba_proj_bias、embedding_multiplier、logits_scaling、attention_multiplier、residual_multiplier、num_local_experts、num_experts_per_tok、shared_intermediate_size、output_router_logits、router_aux_loss_coef。 本小节覆盖同一逻辑块中的第 129-202 行。

### Lines 203-203: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 204-258: Defines function GraniteMoeHybridConfig.__init__ (part 2)
```python
        self.num_attention_heads = num_attention_heads
        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.normalization_function = normalization_function

        self.use_cache = use_cache
        self.max_position_embeddings = max_position_embeddings
        self.attention_dropout = attention_dropout
        self.attention_bias = attention_bias

        self.position_embedding_type = position_embedding_type
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling

        # Mamba configuration
        self.mamba_d_state = mamba_d_state
        self.mamba_d_conv = mamba_d_conv
        self.mamba_expand = mamba_expand
        self.mamba_d_head = mamba_d_head
        self.mamba_n_heads = mamba_n_heads
        self.mamba_n_groups = mamba_n_groups
        self.mamba_chunk_size = mamba_chunk_size
        self.mamba_conv_bias = mamba_conv_bias
        self.mamba_proj_bias = mamba_proj_bias

        # Calculate mamba intermediate size
        self.mamba_intermediate_size = mamba_expand * hidden_size

        # Validate mamba configuration
        if self.mamba_intermediate_size % mamba_n_heads != 0:
            raise ValueError(
                f"mamba_intermediate_size ({self.mamba_intermediate_size}) must be divisible by "
                f"mamba_n_heads ({mamba_n_heads})"
            )

        if mamba_d_head * mamba_n_heads != self.mamba_intermediate_size:
            raise ValueError(
                f"mamba_d_head ({mamba_d_head}) * mamba_n_heads ({mamba_n_heads}) must equal "
                f"mamba_intermediate_size ({self.mamba_intermediate_size})"
            )

        # Scaling factors
        self.embedding_multiplier = embedding_multiplier
        self.logits_scaling = logits_scaling
        self.attention_multiplier = attention_multiplier
        self.residual_multiplier = residual_multiplier

        # MoE configuration
        self.num_local_experts = num_local_experts
        self.num_experts_per_tok = num_experts_per_tok
        self.shared_intermediate_size = shared_intermediate_size
        self.output_router_logits = output_router_logits
        self.router_aux_loss_coef = router_aux_loss_coef
```
**EN:** This block defines function `GraniteMoeHybridConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, layer_types, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, normalization_function, use_cache, pad_token_id, bos_token_id, eos_token_id, max_position_embeddings, attention_dropout, attention_bias, position_embedding_type, rope_theta, rope_scaling, mamba_d_state, mamba_d_conv, mamba_expand, mamba_d_head, mamba_n_heads, mamba_n_groups, mamba_chunk_size, mamba_conv_bias, mamba_proj_bias, embedding_multiplier, logits_scaling, attention_multiplier, residual_multiplier, num_local_experts, num_experts_per_tok, shared_intermediate_size, output_router_logits, router_aux_loss_coef. This subsection covers lines 204-258 of the same logical block.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、layer_types、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、normalization_function、use_cache、pad_token_id、bos_token_id、eos_token_id、max_position_embeddings、attention_dropout、attention_bias、position_embedding_type、rope_theta、rope_scaling、mamba_d_state、mamba_d_conv、mamba_expand、mamba_d_head、mamba_n_heads、mamba_n_groups、mamba_chunk_size、mamba_conv_bias、mamba_proj_bias、embedding_multiplier、logits_scaling、attention_multiplier、residual_multiplier、num_local_experts、num_experts_per_tok、shared_intermediate_size、output_router_logits、router_aux_loss_coef。 本小节覆盖同一逻辑块中的第 204-258 行。

### Lines 259-259: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 260-266: Defines function GraniteMoeHybridConfig.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `GraniteMoeHybridConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, layer_types, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, normalization_function, use_cache, pad_token_id, bos_token_id, eos_token_id, max_position_embeddings, attention_dropout, attention_bias, position_embedding_type, rope_theta, rope_scaling, mamba_d_state, mamba_d_conv, mamba_expand, mamba_d_head, mamba_n_heads, mamba_n_groups, mamba_chunk_size, mamba_conv_bias, mamba_proj_bias, embedding_multiplier, logits_scaling, attention_multiplier, residual_multiplier, num_local_experts, num_experts_per_tok, shared_intermediate_size, output_router_logits, router_aux_loss_coef. This subsection covers lines 260-266 of the same logical block.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、layer_types、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、normalization_function、use_cache、pad_token_id、bos_token_id、eos_token_id、max_position_embeddings、attention_dropout、attention_bias、position_embedding_type、rope_theta、rope_scaling、mamba_d_state、mamba_d_conv、mamba_expand、mamba_d_head、mamba_n_heads、mamba_n_groups、mamba_chunk_size、mamba_conv_bias、mamba_proj_bias、embedding_multiplier、logits_scaling、attention_multiplier、residual_multiplier、num_local_experts、num_experts_per_tok、shared_intermediate_size、output_router_logits、router_aux_loss_coef。 本小节覆盖同一逻辑块中的第 260-266 行。

### Lines 267-267: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 268-273: Defines function GraniteMoeHybridConfig.mamba_layer_ids
```python
    @property
    def mamba_layer_ids(self):
        """Returns the indices of layers that are Mamba layers."""
        return [
            i for i in range(self.num_hidden_layers) if self.layer_types[i] == MAMBA
        ]
```
**EN:** This block defines function `GraniteMoeHybridConfig.mamba_layer_ids`. Parameters: self. Decorators: property. Returns the indices of layers that are Mamba layers.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.mamba_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Returns the indices of layers that are Mamba layers.

### Lines 274-274: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 275-280: Defines function GraniteMoeHybridConfig.attention_layer_ids
```python
    @property
    def attention_layer_ids(self):
        """Returns the indices of layers that are attention layers."""
        return [
            i for i in range(self.num_hidden_layers) if self.layer_types[i] == ATTENTION
        ]
```
**EN:** This block defines function `GraniteMoeHybridConfig.attention_layer_ids`. Parameters: self. Decorators: property. Returns the indices of layers that are attention layers.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.attention_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Returns the indices of layers that are attention layers.

### Lines 281-281: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 282-285: Defines function GraniteMoeHybridConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self):
        """Alias for attention_layer_ids for compatibility."""
        return self.attention_layer_ids
```
**EN:** This block defines function `GraniteMoeHybridConfig.full_attention_layer_ids`. Parameters: self. Decorators: property. Alias for attention_layer_ids for compatibility.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Alias for attention_layer_ids for compatibility.

### Lines 286-286: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GraniteMoeHybridConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GraniteMoeHybridConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 287-301: Defines function GraniteMoeHybridConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self):
        """Returns the Mamba2 cache parameters for this configuration."""
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        shape = Mamba2StateShape.create(
            tp_world_size=get_attention_tp_size(),
            intermediate_size=self.mamba_intermediate_size,
            n_groups=self.mamba_n_groups,
            num_heads=self.mamba_n_heads,
            head_dim=self.mamba_d_head,
            state_size=self.mamba_d_state,
            conv_kernel=self.mamba_d_conv,
        )
        return Mamba2CacheParams(shape=shape, layers=self.mamba_layer_ids)
```
**EN:** This block defines function `GraniteMoeHybridConfig.mamba2_cache_params`. Parameters: self. Decorators: property. Returns the Mamba2 cache parameters for this configuration.
**CN:** 该代码块定义函数 `GraniteMoeHybridConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Returns the Mamba2 cache parameters for this configuration.

## Key Concepts / 关键概念
- **Classes / 类**: `GraniteMoeHybridConfig`
- **Constants / 常量**: `MAMBA`, `ATTENTION`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
