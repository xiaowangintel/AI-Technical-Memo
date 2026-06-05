# lfm2_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/lfm2_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for lfm2 moe so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 lfm2 moe 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Comments and module notes
```python
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
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 13-18: Documents the scope
```python
"""LFM2-MoE (Liquid Foundation Model 2 - Mixture of Experts) configuration

Note: HF transformers has Lfm2MoeConfig in v5.0.0rc2 (unreleased).
Once released, we could inherit from it like Lfm2Config does with HFLfm2Config.
For now, we define a standalone config to support the model immediately.
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 19-19: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-25: Imports dependencies
```python
from typing import List, Optional

from transformers import CONFIG_MAPPING
from transformers.configuration_utils import PretrainedConfig

from sglang.srt.configs.mamba_utils import Mamba2CacheParams, Mamba2StateShape
```
**EN:** This block groups related imports for the module, including typing.List, typing.Optional, transformers.CONFIG_MAPPING, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams, and 1 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Optional, transformers.CONFIG_MAPPING, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams 等 1 项，为后续代码准备所需名称。

### Lines 26-27: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-28: Declares class Lfm2MoeConfig
```python
class Lfm2MoeConfig(PretrainedConfig):
```
**EN:** This block introduces class `Lfm2MoeConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. Configuration for LFM2-MoE models (e.g., LiquidAI/LFM2-8B-A1B). LFM2-MoE is a hybrid architecture with: - Attention layers and ShortConv layers (like dense LFM2) - MoE (Mixture of Experts) FFN layers with sigmoid routing Key MoE specifics: - First `num_dense_l
**CN:** 该代码块声明类 `Lfm2MoeConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：Configuration for LFM2-MoE models (e.g., LiquidAI/LFM2-8B-A1B). LFM2-MoE is a hybrid architecture with: - Attention layers and ShortConv layers (like dense LFM2) - MoE (Mixture of Experts) FFN layers with sigmoid routing Key MoE specifics: - First `num_dense_l

### Lines 29-40: Documents the scope
```python
    """
    Configuration for LFM2-MoE models (e.g., LiquidAI/LFM2-8B-A1B).

    LFM2-MoE is a hybrid architecture with:
    - Attention layers and ShortConv layers (like dense LFM2)
    - MoE (Mixture of Experts) FFN layers with sigmoid routing

    Key MoE specifics:
    - First `num_dense_layers` use dense MLP, rest use MoE
    - Sigmoid routing (not softmax) with expert_bias for load balancing
    - expert_bias is fp32 for numerical stability
    """
```
**EN:** This string literal serves as documentation for the Lfm2MoeConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Lfm2MoeConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 41-41: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 42-43: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "lfm2_moe"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Lfm2MoeConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Lfm2MoeConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 44-44: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 45-122: Defines function Lfm2MoeConfig.__init__
```python
    def __init__(
        self,
        vocab_size: int = 65536,
        hidden_size: int = 2048,
        intermediate_size: int = 7168,
        moe_intermediate_size: int = 1792,
        num_hidden_layers: int = 32,
        num_attention_heads: int = 32,
        num_key_value_heads: int = 8,
        max_position_embeddings: int = 128000,
        initializer_range: float = 0.02,
        norm_eps: float = 1e-5,
        use_cache: bool = True,
        pad_token_id: int = 0,
        bos_token_id: int = 1,
        eos_token_id: int = 2,
        tie_word_embeddings: bool = True,
        rope_parameters: Optional[dict] = None,
        conv_bias: bool = False,
        conv_L_cache: int = 3,
        # MoE-specific parameters
        num_dense_layers: int = 2,
        num_experts: int = 32,
        num_experts_per_tok: int = 4,
        use_expert_bias: bool = True,
        routed_scaling_factor: float = 1.0,
        norm_topk_prob: bool = True,
        # Layer types
        layer_types: Optional[List[str]] = None,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.moe_intermediate_size = moe_intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.num_key_value_heads = num_key_value_heads
        self.max_position_embeddings = max_position_embeddings
        self.initializer_range = initializer_range
        self.norm_eps = norm_eps
        self.use_cache = use_cache

        # Conv parameters
        self.conv_bias = conv_bias
        self.conv_L_cache = conv_L_cache

        # MoE parameters
        self.num_dense_layers = num_dense_layers
        self.num_experts = num_experts
        self.num_experts_per_tok = num_experts_per_tok
        self.use_expert_bias = use_expert_bias
        self.routed_scaling_factor = routed_scaling_factor
        self.norm_topk_prob = norm_topk_prob

        # Layer types (attention vs conv)
        self.layer_types = layer_types

        # RoPE parameters
        self.rope_parameters = rope_parameters

        # Validate layer_types length matches num_hidden_layers
        if layer_types is not None and len(layer_types) != num_hidden_layers:
            raise ValueError(
                f"layer_types length ({len(layer_types)}) must match "
                f"num_hidden_layers ({num_hidden_layers})"
            )

        # Handle tie_embedding alias from original config
        tie_word_embeddings = kwargs.pop("tie_embedding", tie_word_embeddings)

        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `Lfm2MoeConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, max_position_embeddings, initializer_range, norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, tie_word_embeddings, rope_parameters, conv_bias, conv_L_cache, num_dense_layers, num_experts, num_experts_per_tok, use_expert_bias, routed_scaling_factor, norm_topk_prob, layer_types.
**CN:** 该代码块定义函数 `Lfm2MoeConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、max_position_embeddings、initializer_range、norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、tie_word_embeddings、rope_parameters、conv_bias、conv_L_cache、num_dense_layers、num_experts、num_experts_per_tok、use_expert_bias、routed_scaling_factor、norm_topk_prob、layer_types。

### Lines 123-123: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 124-129: Defines function Lfm2MoeConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self) -> List[int]:
        """Return indices of attention layers for KV cache."""
        if self.layer_types is None:
            return []
        return [i for i, lt in enumerate(self.layer_types) if lt == "full_attention"]
```
**EN:** This block defines function `Lfm2MoeConfig.full_attention_layer_ids`. Parameters: self. Decorators: property. Return indices of attention layers for KV cache.
**CN:** 该代码块定义函数 `Lfm2MoeConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return indices of attention layers for KV cache.

### Lines 130-130: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 131-138: Defines function Lfm2MoeConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self) -> List[int]:
        """Return indices of conv layers for conv state cache."""
        if self.layer_types is None:
            return []
        return [
            i for i, lt in enumerate(self.layer_types) if lt in ("conv", "short_conv")
        ]
```
**EN:** This block defines function `Lfm2MoeConfig.linear_layer_ids`. Parameters: self. Decorators: property. Return indices of conv layers for conv state cache.
**CN:** 该代码块定义函数 `Lfm2MoeConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return indices of conv layers for conv state cache.

### Lines 139-139: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 140-143: Defines function Lfm2MoeConfig.mamba_chunk_size
```python
    @property
    def mamba_chunk_size(self) -> int:
        """Return chunk size for Mamba2 backend. LFM2 doesn't use chunking."""
        return 1
```
**EN:** This block defines function `Lfm2MoeConfig.mamba_chunk_size`. Parameters: self. Decorators: property. Return chunk size for Mamba2 backend. LFM2 doesn't use chunking.
**CN:** 该代码块定义函数 `Lfm2MoeConfig.mamba_chunk_size`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return chunk size for Mamba2 backend. LFM2 doesn't use chunking.

### Lines 144-144: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2MoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2MoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 145-183: Defines function Lfm2MoeConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Optional[Mamba2CacheParams]:
        """
        Get cache params for HybridReqToTokenPool initialization.

        LFM2-MoE uses ShortConv layers with a small fixed-size cache.
        """
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        conv_layer_ids = self.linear_layer_ids
        if not conv_layer_ids:
            return None

        hidden_size = self.hidden_size
        # conv_L_cache in config is kernel_size (e.g., 3)
        conv_kernel = int(self.conv_L_cache)
        # actual cache size is kernel_size - 1 (e.g., 2 for kernel=3)

        try:
            tp_size = get_attention_tp_size()
        except (AssertionError, RuntimeError):
            tp_size = 1

        shape = Mamba2StateShape.create(
            tp_world_size=tp_size,
            intermediate_size=hidden_size,
            n_groups=1,
            num_heads=tp_size,  # Ensures divide works; temporal state is empty anyway
            head_dim=hidden_size,
            state_size=0,
            conv_kernel=conv_kernel,
        )

        # Uses default mamba2_state_dtype() which reads SGLANG_MAMBA_CONV_DTYPE env var
        # (defaults to bfloat16). Set SGLANG_MAMBA_CONV_DTYPE=float16 for fp16 inference.
        return Mamba2CacheParams(
            shape=shape,
            layers=conv_layer_ids,
        )
```
**EN:** This block defines function `Lfm2MoeConfig.mamba2_cache_params`. Parameters: self. Decorators: property. Get cache params for HybridReqToTokenPool initialization. LFM2-MoE uses ShortConv layers with a small fixed-size cache.
**CN:** 该代码块定义函数 `Lfm2MoeConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Get cache params for HybridReqToTokenPool initialization. LFM2-MoE uses ShortConv layers with a small fixed-size cache.

### Lines 184-187: Comments and module notes
```python


# Register with transformers CONFIG_MAPPING so AutoConfig.from_pretrained()
# can instantiate our config class when loading models with model_type="lfm2_moe"
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 188-192: Handles exceptional control flow
```python
try:
    CONFIG_MAPPING.register("lfm2_moe", Lfm2MoeConfig)
except Exception:
    # Already registered or registration failed - use direct assignment
    CONFIG_MAPPING._extra_content["lfm2_moe"] = Lfm2MoeConfig
```
**EN:** This block protects a section of the module with exception handling and optional cleanup logic.
**CN:** 该代码块为 模块 中的一段逻辑提供异常处理与可选清理流程。

## Key Concepts / 关键概念
- **Classes / 类**: `Lfm2MoeConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers`, `transformers.configuration_utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
