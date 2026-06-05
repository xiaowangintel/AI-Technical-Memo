# nemotron_h.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/nemotron_h.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for nemotron h so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 nemotron h 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

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
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/transformers_utils/configs/nemotron_h.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 18-18: Documents the scope
```python
"""NemotronH model configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 19-19: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-27: Imports dependencies
```python
from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging

from sglang.srt.configs.mamba_utils import (
    Mamba2CacheParams,
    Mamba2StateShape,
    mamba2_state_dtype,
)
```
**EN:** This block groups related imports for the module, including transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape, sglang.srt.configs.mamba_utils.mamba2_state_dtype. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.configuration_utils.PretrainedConfig, transformers.utils.logging, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape, sglang.srt.configs.mamba_utils.mamba2_state_dtype，为后续代码准备所需名称。

### Lines 28-28: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 29-29: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 30-30: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 31-37: Declares MAMBA, ATTENTION, MLP, MOE, DEFAULT_LAYERS_BLOCK_TYPE, and 2 more
```python
MAMBA = "M"
ATTENTION = "*"
MLP = "-"
MOE = "E"
DEFAULT_LAYERS_BLOCK_TYPE = ["mamba", "moe", "attention", "moe"]
DEFAULT_MTP_LAYERS_BLOCK_TYPE = ["attention", "moe"]
DEFAULT_MAMBA_CHUNK_SIZE = 256
```
**EN:** This block initializes a related set of values in the module, including MAMBA, ATTENTION, MLP, MOE, DEFAULT_LAYERS_BLOCK_TYPE, and 2 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 MAMBA, ATTENTION, MLP, MOE, DEFAULT_LAYERS_BLOCK_TYPE 等 2 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 38-39: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 40-40: Declares class NemotronHConfig
```python
class NemotronHConfig(PretrainedConfig):
```
**EN:** This block introduces class `NemotronHConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. This is the configuration class to store the configuration of a [`NemotronHModel`]. It is used to instantiate a NemotronH model according to the specified arguments, defining the model architecture.
**CN:** 该代码块声明类 `NemotronHConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：This is the configuration class to store the configuration of a [`NemotronHModel`]. It is used to instantiate a NemotronH model according to the specified arguments, defining the model architecture.

### Lines 41-120: Documents the scope (part 1)
```python
    r"""
    This is the configuration class to store the configuration of a
    [`NemotronHModel`]. It is used to instantiate a NemotronH model according
    to the specified arguments, defining the model architecture. Instantiating
    a configuration with the defaults will yield a similar configuration to
    that of the NemotronH-v0.1 model.
    Args:
        vocab_size (`int`, *optional*, defaults to 131072):
            Vocabulary size of the NemotronH model. Defines the number of
            different tokens that can be represented by the `inputs_ids`
            passed when calling [`NemotronHModel`]
        tie_word_embeddings (`bool`, *optional*, defaults to `False`):
            Whether the model's input and output word embeddings should be
            tied. Note that this is only relevant if the model has an output
            word embedding layer.
        hidden_size (`int`, *optional*, defaults to 4096):
            Dimension of the hidden representations.
        intermediate_size (`int`, *optional*, defaults to 21504):
            Dimension of the MLP representations.
        num_hidden_layers (`int`, *optional*):
            Deprecated. Kept only for backward compatibility. The effective
            layer count is derived from `layers_block_type`.
        hybrid_override_pattern (`str`, *optional*, defaults to
            `"M-M-M-M*-M-M-M-M-M*-M-M-M-M-M*-M-M-M-M-M*-M-M-M-M-M-"`):
            Deprecated compatibility field. Pattern string where each
            character represents Mamba2 (`M`), Attention (`*`), MLP (`-`),
            or MoE (`E`).
        layers_block_type (`list[str]`, *optional*):
            Canonical layer layout. Each entry is one of:
            `"mamba"`, `"attention"`, `"mlp"`, `"moe"`.
        num_attention_heads (`int`, *optional*, defaults to 32):
            Number of attention heads for each attention layer in the
            Transformer encoder.
        attention_head_dim (`int`, *optional*, defaults to 128):
            Dimension of each attention head.
        num_key_value_heads (`int`, *optional*, defaults to 8):
            This is the number of key_value heads that should be used to
            implement Grouped Query Attention. If
            `num_key_value_heads=num_attention_heads`, the model will use
            Multi Head Attention (MHA), if `num_key_value_heads=1` the model
            will use Multi Query Attention (MQA) otherwise GQA is used.
        mlp_hidden_act (`str`, *optional*, defaults to "relu2"):
            The non-linear activation function in the MLP layers.
        attention_bias (`bool`, *optional*, defaults to `False`):
            Whether to use bias in attention layers.
        mlp_bias (`bool`, *optional*, defaults to `False`):
            Whether to use bias in MLP layers.
        use_bias (`bool`, *optional*, defaults to `False`):
            Whether to use bias in the model.
        initializer_range (`float`, *optional*, defaults to 0.02):
            The standard deviation of the truncated_normal_initializer for
            initializing all weight matrices.
        layer_norm_epsilon (`float`, *optional*, defaults to 1e-5):
            The epsilon used by the layer normalization layers.
        residual_in_fp32 (`bool`, *optional*, defaults to `False`):
            Whether or not residuals should be in `float32`. If set to `False`
            residuals will keep the same `dtype` as the rest of the model.
        use_cache (`bool`, *optional*, defaults to `True`):
            Whether or not the model should return the last key/values
            attentions (not used by all models). Only relevant if
            `config.is_decoder=True`.
        num_logits_to_keep (`int` or `None`, *optional*, defaults to 1):
            Number of prompt logits to calculate during generation. If `None`,
            all logits will be calculated. If an integer value, only last
            `num_logits_to_keep` logits will be calculated.
        pad_token_id (`int`, *optional*, defaults to 0):
            The id of the padding token.
        bos_token_id (`int`, *optional*, defaults to 1):
            The id of the "beginning-of-sequence" token.
        eos_token_id (`int`, *optional*, defaults to 2):
            The id of the "end-of-sequence" token.
        sliding_window (`int`, *optional*, defaults to None):
            Sliding window attention window size.
        max_position_embeddings (`int`, *optional*, defaults to 4096):
            The maximum sequence length that this model might ever be used
            with.
        attention_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the attention probabilities.
        hidden_dropout (`float`, *optional*, defaults to 0.0):
            The dropout ratio for the hidden states.
```
**EN:** This string literal serves as documentation for the NemotronHConfig, explaining intent or usage without affecting execution. This subsection covers lines 41-120 of the same logical block.
**CN:** 该字符串字面量作为 NemotronHConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 41-120 行。

### Lines 121-157: Documents the scope (part 2)
```python
        use_mamba_kernels (`bool`, *optional*, defaults to `True`):
            Flag indicating whether or not to use the fast mamba kernels.
            These are available only if `mamba-ssm` and `causal-conv1d`
            are installed, and the mamba modules are running on a CUDA device.
        ssm_state_size (`int`, *optional*, defaults to 128):
            The dimension of the mamba state space latents.
        mamba_num_heads (`int`, *optional*, defaults to 128):
            Number of heads in Mamba layers.
        mamba_n_groups (`int`, *optional*, defaults to 8):
            Number of groups in Mamba layers.
        mamba_head_dim (`int`, *optional*, defaults to 64):
            Dimension of each Mamba head.
        mamba_d_conv (`int`, *optional*, defaults to 4):
            The size of the mamba convolution kernel.
        mamba_expand (`int`, *optional*, defaults to 2):
            Expanding factor used to determine the mamba intermediate size.
        mamba_hidden_act (`str`, *optional*, defaults to "silu"):
            The non-linear activation function in the Mamba layers.
        mamba_dt_min (`float`, *optional*, defaults to 0.001):
            Minimum value for the time step in Mamba.
        mamba_dt_max (`float`, *optional*, defaults to 0.1):
            Maximum value for the time step in Mamba.
        mamba_dt_limit (`tuple`, *optional*, defaults to (0.0, float("inf"))):
            Limits for the time step in Mamba.
        mamba_dt_init_floor (`float`, *optional*, defaults to 1e-4):
            Floor value for time step initialization in Mamba.
        mamba_conv_bias (`bool`, *optional*, defaults to `True`):
            Whether to use bias in the convolution layer of the mamba mixer
            block.
        mamba_proj_bias (`bool`, *optional*, defaults to `False`):
            Whether to use bias in the input and output projections of the
            mamba mixer block.
        mamba_chunk_size (`int`, *optional*, defaults to 256):
            Size of chunks for Mamba processing.
        rescale_prenorm_residual (`bool`, *optional*, defaults to `True`):
            Whether to rescale the pre-normalization residual connections.
    """
```
**EN:** This string literal serves as documentation for the NemotronHConfig, explaining intent or usage without affecting execution. This subsection covers lines 121-157 of the same logical block.
**CN:** 该字符串字面量作为 NemotronHConfig 的文档说明，用于解释意图或用法，但不会影响执行。 本小节覆盖同一逻辑块中的第 121-157 行。

### Lines 158-158: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 159-160: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "nemotron_h"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the NemotronHConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 NemotronHConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 161-161: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 162-188: Defines function NemotronHConfig._validate_layers_block_type
```python
    @staticmethod
    def _validate_layers_block_type(
        layers_block_type, expected_length=None, param_name="layers_block_type"
    ):
        """
        Validate layers_block_type list.
        Args:
            layers_block_type: List of layer types to validate.
            expected_length: If provided, validate the list has this length.
            param_name: Parameter name for error messages.
        Raises:
            ValueError: If validation fails.
        """
        if not isinstance(layers_block_type, list):
            raise ValueError(
                f"{param_name} must be a list of strings. Got type: {type(layers_block_type)}"
            )
        if expected_length is not None and len(layers_block_type) != expected_length:
            raise ValueError(
                f"{param_name} must have length {expected_length}. Got length {len(layers_block_type)}."
            )
        valid_types = {"mamba", "attention", "mlp", "moe"}
        if not all(block_type in valid_types for block_type in layers_block_type):
            invalid = set(layers_block_type) - valid_types
            raise ValueError(
                f"{param_name} contains invalid types: {invalid}. Must be one of: {valid_types}"
            )
```
**EN:** This block defines function `NemotronHConfig._validate_layers_block_type`. Parameters: layers_block_type, expected_length, param_name. Decorators: staticmethod. Validate layers_block_type list. Args: layers_block_type: List of layer types to validate.
**CN:** 该代码块定义函数 `NemotronHConfig._validate_layers_block_type`。 参数包括 layers_block_type、expected_length、param_name。 装饰器包括 staticmethod。 文档字符串摘要：Validate layers_block_type list. Args: layers_block_type: List of layer types to validate.

### Lines 189-189: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 190-205: Defines function NemotronHConfig._resolve_layers_block_type
```python
    @staticmethod
    def _resolve_layers_block_type(
        layers_block_type, hybrid_override_pattern, kwargs
    ) -> list[str]:
        """Resolve canonical layers_block_type from new and legacy config fields."""
        # Prefer explicit kwargs override first (legacy HF path), otherwise use
        # the function argument value from config fields.
        pattern = kwargs.pop("hybrid_override_pattern", hybrid_override_pattern)
        if layers_block_type is None:
            if pattern is not None:
                layers_block_type = NemotronHConfig._pattern_to_list(pattern)
            else:
                # Last-resort fallback to preserve compatibility when neither
                # canonical nor legacy pattern fields are provided.
                layers_block_type = DEFAULT_LAYERS_BLOCK_TYPE
        return layers_block_type
```
**EN:** This block defines function `NemotronHConfig._resolve_layers_block_type`. Parameters: layers_block_type, hybrid_override_pattern, kwargs. Decorators: staticmethod. Resolve canonical layers_block_type from new and legacy config fields.
**CN:** 该代码块定义函数 `NemotronHConfig._resolve_layers_block_type`。 参数包括 layers_block_type、hybrid_override_pattern、kwargs。 装饰器包括 staticmethod。 文档字符串摘要：Resolve canonical layers_block_type from new and legacy config fields.

### Lines 206-206: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 207-217: Defines function NemotronHConfig._resolve_mtp_layers_block_type
```python
    @staticmethod
    def _resolve_mtp_layers_block_type(mtp_layers_block_type, kwargs) -> list[str]:
        """Resolve canonical mtp_layers_block_type from new and legacy config fields."""
        if "mtp_hybrid_override_pattern" in kwargs:
            pattern = kwargs.pop("mtp_hybrid_override_pattern")
            if mtp_layers_block_type is None or mtp_layers_block_type == [
                "attention",
                "moe",
            ]:
                mtp_layers_block_type = NemotronHConfig._pattern_to_list(pattern)
        return mtp_layers_block_type
```
**EN:** This block defines function `NemotronHConfig._resolve_mtp_layers_block_type`. Parameters: mtp_layers_block_type, kwargs. Decorators: staticmethod. Resolve canonical mtp_layers_block_type from new and legacy config fields.
**CN:** 该代码块定义函数 `NemotronHConfig._resolve_mtp_layers_block_type`。 参数包括 mtp_layers_block_type、kwargs。 装饰器包括 staticmethod。 文档字符串摘要：Resolve canonical mtp_layers_block_type from new and legacy config fields.

### Lines 218-218: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 219-239: Defines function NemotronHConfig._resolve_mamba_chunk_size
```python
    @staticmethod
    def _resolve_mamba_chunk_size(mamba_chunk_size, kwargs) -> int:
        """Resolve canonical mamba_chunk_size from new and legacy config fields."""
        chunk_size = kwargs.pop("chunk_size", None)
        if (
            mamba_chunk_size is not None
            and chunk_size is not None
            and mamba_chunk_size != chunk_size
        ):
            logger.warning(
                "Both chunk_size=%s and mamba_chunk_size=%s were provided. "
                "Using mamba_chunk_size.",
                chunk_size,
                mamba_chunk_size,
            )

        if mamba_chunk_size is None:
            mamba_chunk_size = chunk_size
        if mamba_chunk_size is None:
            mamba_chunk_size = DEFAULT_MAMBA_CHUNK_SIZE
        return mamba_chunk_size
```
**EN:** This block defines function `NemotronHConfig._resolve_mamba_chunk_size`. Parameters: mamba_chunk_size, kwargs. Decorators: staticmethod. Resolve canonical mamba_chunk_size from new and legacy config fields.
**CN:** 该代码块定义函数 `NemotronHConfig._resolve_mamba_chunk_size`。 参数包括 mamba_chunk_size、kwargs。 装饰器包括 staticmethod。 文档字符串摘要：Resolve canonical mamba_chunk_size from new and legacy config fields.

### Lines 240-240: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 241-317: Defines function NemotronHConfig.__init__ (part 1)
```python
    def __init__(
        self,
        vocab_size=131072,
        tie_word_embeddings=False,
        hidden_size=4096,
        intermediate_size=21504,
        num_hidden_layers=None,  # Deprecated, only for backward compatibility
        hybrid_override_pattern="M-M-M-M*-M-M-M-M-M*-M-M-M-M-M*-M-M-M-M-M*-M-M-M-M-M-",
        layers_block_type=None,
        num_attention_heads=32,
        head_dim=128,
        num_key_value_heads=8,  # nemo: num_query_groups
        mlp_hidden_act="relu2",
        attention_bias=False,
        mlp_bias=False,
        use_bias=False,
        initializer_range=0.02,  # nemo: init_method_std
        layer_norm_epsilon=1e-5,  # nemo: layernorm_epsilon
        residual_in_fp32=False,  #  Megatron Core default value
        use_cache=True,
        num_logits_to_keep=1,
        pad_token_id=0,
        bos_token_id=1,
        eos_token_id=2,
        sliding_window=None,
        max_position_embeddings=4096,
        attention_dropout=0.0,
        hidden_dropout=0.0,  # * ADDED
        use_mamba_kernels=True,
        ssm_state_size=128,  # mamba_state_size
        mamba_num_heads=128,
        mamba_n_groups=8,  # nemo: mamba_ssm_ngroups = num_heads
        mamba_head_dim=64,
        mamba_d_conv=4,
        mamba_expand=2,
        mamba_hidden_act="silu",
        mamba_dt_min=0.001,
        mamba_dt_max=0.1,
        mamba_dt_limit=(0.0, float("inf")),
        mamba_dt_init_floor=1e-4,
        mamba_conv_bias=True,
        mamba_proj_bias=False,
        mamba_chunk_size=None,
        rescale_prenorm_residual=True,
        n_routed_experts=8,
        n_shared_experts=1,
        moe_intermediate_size=7688,
        moe_shared_expert_intermediate_size=7688,
        moe_latent_size=None,
        num_experts_per_tok=2,
        routed_scaling_factor=1.0,
        n_group=1,
        topk_group=1,
        norm_topk_prob=True,
        num_nextn_predict_layers=0,
        mtp_layers_block_type=DEFAULT_MTP_LAYERS_BLOCK_TYPE,
        **kwargs,
    ):
        mamba_chunk_size = self._resolve_mamba_chunk_size(mamba_chunk_size, kwargs)

        # Compatibility parsing: normalize legacy pattern fields into canonical list fields.
        layers_block_type = self._resolve_layers_block_type(
            layers_block_type, hybrid_override_pattern, kwargs
        )
        mtp_layers_block_type = self._resolve_mtp_layers_block_type(
            mtp_layers_block_type, kwargs
        )

        # num_hidden_layers is deprecated and ignored as a source of truth.
        if (
            num_hidden_layers is not None
            and len(layers_block_type) != num_hidden_layers
        ):
            logger.warning(
                f"num_hidden_layers ({num_hidden_layers}) is deprecated and doesn't match "
                f"layers_block_type length ({len(layers_block_type)}). Using layers_block_type length."
            )
```
**EN:** This block defines function `NemotronHConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, hybrid_override_pattern, layers_block_type, num_attention_heads, head_dim, num_key_value_heads, mlp_hidden_act, attention_bias, mlp_bias, use_bias, initializer_range, layer_norm_epsilon, residual_in_fp32, use_cache, num_logits_to_keep, pad_token_id, bos_token_id, eos_token_id, sliding_window, max_position_embeddings, attention_dropout, hidden_dropout, use_mamba_kernels, ssm_state_size, mamba_num_heads, mamba_n_groups, mamba_head_dim, mamba_d_conv, mamba_expand, mamba_hidden_act, mamba_dt_min, mamba_dt_max, mamba_dt_limit, mamba_dt_init_floor, mamba_conv_bias, mamba_proj_bias, mamba_chunk_size, rescale_prenorm_residual, n_routed_experts, n_shared_experts, moe_intermediate_size, moe_shared_expert_intermediate_size, moe_latent_size, num_experts_per_tok, routed_scaling_factor, n_group, topk_group, norm_topk_prob, num_nextn_predict_layers, mtp_layers_block_type. This subsection covers lines 241-317 of the same logical block.
**CN:** 该代码块定义函数 `NemotronHConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、hybrid_override_pattern、layers_block_type、num_attention_heads、head_dim、num_key_value_heads、mlp_hidden_act、attention_bias、mlp_bias、use_bias、initializer_range、layer_norm_epsilon、residual_in_fp32、use_cache、num_logits_to_keep、pad_token_id、bos_token_id、eos_token_id、sliding_window、max_position_embeddings、attention_dropout、hidden_dropout、use_mamba_kernels、ssm_state_size、mamba_num_heads、mamba_n_groups、mamba_head_dim、mamba_d_conv、mamba_expand、mamba_hidden_act、mamba_dt_min、mamba_dt_max、mamba_dt_limit、mamba_dt_init_floor、mamba_conv_bias、mamba_proj_bias、mamba_chunk_size、rescale_prenorm_residual、n_routed_experts、n_shared_experts、moe_intermediate_size、moe_shared_expert_intermediate_size、moe_latent_size、num_experts_per_tok、routed_scaling_factor、n_group、topk_group、norm_topk_prob、num_nextn_predict_layers、mtp_layers_block_type。 本小节覆盖同一逻辑块中的第 241-317 行。

### Lines 318-318: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 319-393: Defines function NemotronHConfig.__init__ (part 2)
```python
        # Core model attributes.
        self.vocab_size = vocab_size
        self.tie_word_embeddings = tie_word_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_attention_heads = num_attention_heads
        self.head_dim = head_dim
        self.sliding_window = sliding_window
        self.max_position_embeddings = max_position_embeddings
        self.attention_dropout = attention_dropout
        self.hidden_dropout = hidden_dropout

        self._validate_layers_block_type(
            layers_block_type, expected_length=None, param_name="layers_block_type"
        )
        self.layers_block_type = layers_block_type

        # for backward compatibility
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads

        self.num_key_value_heads = num_key_value_heads
        self.mlp_hidden_act = mlp_hidden_act
        self.attention_bias = attention_bias
        self.mlp_bias = mlp_bias
        self.use_bias = use_bias
        self.initializer_range = initializer_range
        self.layer_norm_epsilon = layer_norm_epsilon
        self.residual_in_fp32 = residual_in_fp32

        self.use_cache = use_cache
        self.num_logits_to_keep = num_logits_to_keep

        # Mamba attributes.
        self.use_mamba_kernels = use_mamba_kernels
        self.mamba_n_groups = mamba_n_groups
        self.mamba_head_dim = mamba_head_dim
        self.ssm_state_size = ssm_state_size
        self.mamba_num_heads = mamba_num_heads
        self.conv_kernel = mamba_d_conv
        self.expand = mamba_expand
        self.mamba_hidden_act = mamba_hidden_act
        self.time_step_min = mamba_dt_min
        self.time_step_max = mamba_dt_max
        self.time_step_limit = mamba_dt_limit
        self.time_step_floor = mamba_dt_init_floor
        self.use_conv_bias = mamba_conv_bias
        self.mamba_proj_bias = mamba_proj_bias
        self.mamba_chunk_size = mamba_chunk_size
        self.rescale_prenorm_residual = rescale_prenorm_residual
        # MoE attributes.
        self.n_routed_experts = n_routed_experts
        self.n_shared_experts = n_shared_experts
        self.moe_intermediate_size = moe_intermediate_size
        self.moe_shared_expert_intermediate_size = moe_shared_expert_intermediate_size
        self.moe_latent_size = moe_latent_size
        self.num_experts_per_tok = num_experts_per_tok
        self.routed_scaling_factor = routed_scaling_factor
        self.n_group = n_group
        self.topk_group = topk_group
        self.norm_topk_prob = norm_topk_prob
        # MTP attributes.
        self.num_nextn_predict_layers = num_nextn_predict_layers

        if self.num_nextn_predict_layers > 0:
            if mtp_layers_block_type is None:
                raise ValueError(
                    "mtp_layers_block_type is required when num_nextn_predict_layers > 0. "
                    "Please provide an explicit list of layer types for MTP layers. "
                    "Example: mtp_layers_block_type=['attention', 'moe']"
                )
            self._validate_layers_block_type(
                mtp_layers_block_type, None, "mtp_layers_block_type"
            )
        self.mtp_layers_block_type = mtp_layers_block_type
```
**EN:** This block defines function `NemotronHConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, hybrid_override_pattern, layers_block_type, num_attention_heads, head_dim, num_key_value_heads, mlp_hidden_act, attention_bias, mlp_bias, use_bias, initializer_range, layer_norm_epsilon, residual_in_fp32, use_cache, num_logits_to_keep, pad_token_id, bos_token_id, eos_token_id, sliding_window, max_position_embeddings, attention_dropout, hidden_dropout, use_mamba_kernels, ssm_state_size, mamba_num_heads, mamba_n_groups, mamba_head_dim, mamba_d_conv, mamba_expand, mamba_hidden_act, mamba_dt_min, mamba_dt_max, mamba_dt_limit, mamba_dt_init_floor, mamba_conv_bias, mamba_proj_bias, mamba_chunk_size, rescale_prenorm_residual, n_routed_experts, n_shared_experts, moe_intermediate_size, moe_shared_expert_intermediate_size, moe_latent_size, num_experts_per_tok, routed_scaling_factor, n_group, topk_group, norm_topk_prob, num_nextn_predict_layers, mtp_layers_block_type. This subsection covers lines 319-393 of the same logical block.
**CN:** 该代码块定义函数 `NemotronHConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、hybrid_override_pattern、layers_block_type、num_attention_heads、head_dim、num_key_value_heads、mlp_hidden_act、attention_bias、mlp_bias、use_bias、initializer_range、layer_norm_epsilon、residual_in_fp32、use_cache、num_logits_to_keep、pad_token_id、bos_token_id、eos_token_id、sliding_window、max_position_embeddings、attention_dropout、hidden_dropout、use_mamba_kernels、ssm_state_size、mamba_num_heads、mamba_n_groups、mamba_head_dim、mamba_d_conv、mamba_expand、mamba_hidden_act、mamba_dt_min、mamba_dt_max、mamba_dt_limit、mamba_dt_init_floor、mamba_conv_bias、mamba_proj_bias、mamba_chunk_size、rescale_prenorm_residual、n_routed_experts、n_shared_experts、moe_intermediate_size、moe_shared_expert_intermediate_size、moe_latent_size、num_experts_per_tok、routed_scaling_factor、n_group、topk_group、norm_topk_prob、num_nextn_predict_layers、mtp_layers_block_type。 本小节覆盖同一逻辑块中的第 319-393 行。

### Lines 394-394: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 395-401: Defines function NemotronHConfig.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `NemotronHConfig.__init__`. Parameters: self, vocab_size, tie_word_embeddings, hidden_size, intermediate_size, num_hidden_layers, hybrid_override_pattern, layers_block_type, num_attention_heads, head_dim, num_key_value_heads, mlp_hidden_act, attention_bias, mlp_bias, use_bias, initializer_range, layer_norm_epsilon, residual_in_fp32, use_cache, num_logits_to_keep, pad_token_id, bos_token_id, eos_token_id, sliding_window, max_position_embeddings, attention_dropout, hidden_dropout, use_mamba_kernels, ssm_state_size, mamba_num_heads, mamba_n_groups, mamba_head_dim, mamba_d_conv, mamba_expand, mamba_hidden_act, mamba_dt_min, mamba_dt_max, mamba_dt_limit, mamba_dt_init_floor, mamba_conv_bias, mamba_proj_bias, mamba_chunk_size, rescale_prenorm_residual, n_routed_experts, n_shared_experts, moe_intermediate_size, moe_shared_expert_intermediate_size, moe_latent_size, num_experts_per_tok, routed_scaling_factor, n_group, topk_group, norm_topk_prob, num_nextn_predict_layers, mtp_layers_block_type. This subsection covers lines 395-401 of the same logical block.
**CN:** 该代码块定义函数 `NemotronHConfig.__init__`。 参数包括 self、vocab_size、tie_word_embeddings、hidden_size、intermediate_size、num_hidden_layers、hybrid_override_pattern、layers_block_type、num_attention_heads、head_dim、num_key_value_heads、mlp_hidden_act、attention_bias、mlp_bias、use_bias、initializer_range、layer_norm_epsilon、residual_in_fp32、use_cache、num_logits_to_keep、pad_token_id、bos_token_id、eos_token_id、sliding_window、max_position_embeddings、attention_dropout、hidden_dropout、use_mamba_kernels、ssm_state_size、mamba_num_heads、mamba_n_groups、mamba_head_dim、mamba_d_conv、mamba_expand、mamba_hidden_act、mamba_dt_min、mamba_dt_max、mamba_dt_limit、mamba_dt_init_floor、mamba_conv_bias、mamba_proj_bias、mamba_chunk_size、rescale_prenorm_residual、n_routed_experts、n_shared_experts、moe_intermediate_size、moe_shared_expert_intermediate_size、moe_latent_size、num_experts_per_tok、routed_scaling_factor、n_group、topk_group、norm_topk_prob、num_nextn_predict_layers、mtp_layers_block_type。 本小节覆盖同一逻辑块中的第 395-401 行。

### Lines 402-402: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 403-409: Defines function NemotronHConfig.mamba_layer_ids
```python
    @property
    def mamba_layer_ids(self):
        return [
            i
            for i in range(self.num_hidden_layers)
            if self.hybrid_override_pattern[i] == MAMBA
        ]
```
**EN:** This block defines function `NemotronHConfig.mamba_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `NemotronHConfig.mamba_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 410-410: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 411-417: Defines function NemotronHConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self):
        return [
            i
            for i in range(self.num_hidden_layers)
            if self.hybrid_override_pattern[i] == ATTENTION
        ]
```
**EN:** This block defines function `NemotronHConfig.full_attention_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `NemotronHConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 418-418: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 419-435: Defines function NemotronHConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Mamba2CacheParams:
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        shape = Mamba2StateShape.create(
            tp_world_size=get_attention_tp_size(),
            intermediate_size=self.mamba_num_heads * self.mamba_head_dim,
            n_groups=self.n_groups,
            num_heads=self.mamba_num_heads,
            head_dim=self.mamba_head_dim,
            state_size=self.ssm_state_size,
            conv_kernel=self.conv_kernel,
        )

        return Mamba2CacheParams(
            shape=shape, layers=self.mamba_layer_ids, dtype=mamba2_state_dtype(self)
        )
```
**EN:** This block defines function `NemotronHConfig.mamba2_cache_params`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `NemotronHConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。

### Lines 436-436: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 437-443: Defines function NemotronHConfig.num_hidden_layers
```python
    @property
    def num_hidden_layers(self) -> int:
        """
        Number of hidden layers derived from the length of layers_block_type.
        This property replaces the deprecated num_hidden_layers parameter.
        """
        return len(self.layers_block_type)
```
**EN:** This block defines function `NemotronHConfig.num_hidden_layers`. Parameters: self. Decorators: property. Number of hidden layers derived from the length of layers_block_type. This property replaces the deprecated num_hidden_layers parameter.
**CN:** 该代码块定义函数 `NemotronHConfig.num_hidden_layers`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Number of hidden layers derived from the length of layers_block_type. This property replaces the deprecated num_hidden_layers parameter.

### Lines 444-444: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 445-451: Defines function NemotronHConfig.num_hidden_layers
```python
    @num_hidden_layers.setter
    def num_hidden_layers(self, value):
        """
        Setter for backward compatibility when loading configs.
        The value is ignored since num_hidden_layers is computed from layers_block_type.
        """
        pass
```
**EN:** This block defines function `NemotronHConfig.num_hidden_layers`. Parameters: self, value. Decorators: num_hidden_layers.setter. Setter for backward compatibility when loading configs. The value is ignored since num_hidden_layers is computed from layers_block_type.
**CN:** 该代码块定义函数 `NemotronHConfig.num_hidden_layers`。 参数包括 self、value。 装饰器包括 num_hidden_layers.setter。 文档字符串摘要：Setter for backward compatibility when loading configs. The value is ignored since num_hidden_layers is computed from layers_block_type.

### Lines 452-452: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 453-459: Defines function NemotronHConfig.hybrid_override_pattern
```python
    @property
    def hybrid_override_pattern(self) -> str:
        """
        Backward compatibility property.
        Returns the pattern string representation of layers_block_type.
        """
        return self._list_to_pattern(self.layers_block_type)
```
**EN:** This block defines function `NemotronHConfig.hybrid_override_pattern`. Parameters: self. Decorators: property. Backward compatibility property. Returns the pattern string representation of layers_block_type.
**CN:** 该代码块定义函数 `NemotronHConfig.hybrid_override_pattern`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Backward compatibility property. Returns the pattern string representation of layers_block_type.

### Lines 460-460: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 461-466: Defines function NemotronHConfig.hybrid_override_pattern
```python
    @hybrid_override_pattern.setter
    def hybrid_override_pattern(self, value):
        """
        Setter for backward compatibility when loading configs.
        """
        self.layers_block_type = self._pattern_to_list(value)
```
**EN:** This block defines function `NemotronHConfig.hybrid_override_pattern`. Parameters: self, value. Decorators: hybrid_override_pattern.setter. Setter for backward compatibility when loading configs.
**CN:** 该代码块定义函数 `NemotronHConfig.hybrid_override_pattern`。 参数包括 self、value。 装饰器包括 hybrid_override_pattern.setter。 文档字符串摘要：Setter for backward compatibility when loading configs.

### Lines 467-467: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 468-474: Defines function NemotronHConfig.mtp_hybrid_override_pattern
```python
    @property
    def mtp_hybrid_override_pattern(self) -> str:
        """
        Backward compatibility property.
        Returns the pattern string representation of mtp_layers_block_type.
        """
        return self._list_to_pattern(self.mtp_layers_block_type)
```
**EN:** This block defines function `NemotronHConfig.mtp_hybrid_override_pattern`. Parameters: self. Decorators: property. Backward compatibility property. Returns the pattern string representation of mtp_layers_block_type.
**CN:** 该代码块定义函数 `NemotronHConfig.mtp_hybrid_override_pattern`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Backward compatibility property. Returns the pattern string representation of mtp_layers_block_type.

### Lines 475-475: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 476-479: Defines function NemotronHConfig.mtp_hybrid_override_pattern
```python
    @mtp_hybrid_override_pattern.setter
    def mtp_hybrid_override_pattern(self, value):
        """Setter for backward compatibility when loading configs."""
        self.mtp_layers_block_type = self._pattern_to_list(value)
```
**EN:** This block defines function `NemotronHConfig.mtp_hybrid_override_pattern`. Parameters: self, value. Decorators: mtp_hybrid_override_pattern.setter. Setter for backward compatibility when loading configs.
**CN:** 该代码块定义函数 `NemotronHConfig.mtp_hybrid_override_pattern`。 参数包括 self、value。 装饰器包括 mtp_hybrid_override_pattern.setter。 文档字符串摘要：Setter for backward compatibility when loading configs.

### Lines 480-480: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 481-490: Defines function NemotronHConfig._list_to_pattern
```python
    @staticmethod
    def _list_to_pattern(layers_list: list[str]) -> str:
        """Convert list of layer types back to pattern string (for backward compatibility)."""
        reverse_mapping = {
            "mamba": MAMBA,
            "moe": MOE,
            "attention": ATTENTION,
            "mlp": MLP,
        }
        return "".join(reverse_mapping[layer_type] for layer_type in layers_list)
```
**EN:** This block defines function `NemotronHConfig._list_to_pattern`. Parameters: layers_list. Decorators: staticmethod. Convert list of layer types back to pattern string (for backward compatibility).
**CN:** 该代码块定义函数 `NemotronHConfig._list_to_pattern`。 参数包括 layers_list。 装饰器包括 staticmethod。 文档字符串摘要：Convert list of layer types back to pattern string (for backward compatibility).

### Lines 491-491: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronHConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronHConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 492-506: Defines function NemotronHConfig._pattern_to_list
```python
    @staticmethod
    def _pattern_to_list(pattern: str) -> list[str]:
        """Convert pattern string to list of layer types (for backward compatibility)."""
        if any(char not in {MAMBA, MOE, ATTENTION, MLP} for char in pattern):
            raise ValueError(
                "Pattern must only contain characters 'M', '*', '-' or 'E'. "
                f"Got: {pattern}"
            )
        pattern_mapping = {
            MAMBA: "mamba",
            MOE: "moe",
            ATTENTION: "attention",
            MLP: "mlp",
        }
        return [pattern_mapping[char] for char in pattern]
```
**EN:** This block defines function `NemotronHConfig._pattern_to_list`. Parameters: pattern. Decorators: staticmethod. Convert pattern string to list of layer types (for backward compatibility).
**CN:** 该代码块定义函数 `NemotronHConfig._pattern_to_list`。 参数包括 pattern。 装饰器包括 staticmethod。 文档字符串摘要：Convert pattern string to list of layer types (for backward compatibility).

## Key Concepts / 关键概念
- **Classes / 类**: `NemotronHConfig`
- **Constants / 常量**: `MAMBA`, `ATTENTION`, `MLP`, `MOE`, `DEFAULT_LAYERS_BLOCK_TYPE`, `DEFAULT_MTP_LAYERS_BLOCK_TYPE`, `DEFAULT_MAMBA_CHUNK_SIZE`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
