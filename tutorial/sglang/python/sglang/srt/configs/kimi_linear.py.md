# kimi_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/kimi_linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for kimi linear so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 kimi linear 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Comments and module notes
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from: https://github.com/vllm-project/vllm/blob/0384aa7150c4c9778efca041ffd1beb3ad2bd694/vllm/transformers_utils/configs/kimi_linear.py
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 4-6: Imports dependencies
```python
from transformers.configuration_utils import PretrainedConfig

from sglang.srt.configs.mamba_utils import KimiLinearCacheParams, KimiLinearStateShape
```
**EN:** This block groups related imports for the module, including transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.KimiLinearCacheParams, sglang.srt.configs.mamba_utils.KimiLinearStateShape. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.KimiLinearCacheParams, sglang.srt.configs.mamba_utils.KimiLinearStateShape，为后续代码准备所需名称。

### Lines 7-8: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 9-9: Declares class KimiLinearConfig
```python
class KimiLinearConfig(PretrainedConfig):
```
**EN:** This block introduces class `KimiLinearConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `KimiLinearConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 10-11: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "kimi_linear"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the KimiLinearConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 KimiLinearConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 12-12: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-75: Defines function KimiLinearConfig.__init__ (part 1)
```python
    def __init__(
        self,
        model_type="kimi_linear",
        vocab_size=163840,
        hidden_size=4096,
        head_dim=None,
        intermediate_size=11008,
        num_hidden_layers=32,
        num_attention_heads=32,
        num_key_value_heads=None,
        hidden_act="silu",
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        pad_token_id=0,
        bos_token_id=1,
        eos_token_id=2,
        rope_theta=10000.0,
        rope_scaling=None,
        tie_word_embeddings=False,
        moe_intermediate_size: int | None = None,
        moe_renormalize: bool = True,
        moe_router_activation_func: str = "sigmoid",
        num_experts: int | None = None,
        num_experts_per_token: int | None = None,
        num_shared_experts: int = 0,
        routed_scaling_factor: float = 1.0,
        first_k_dense_replace: int = 0,
        moe_layer_freq: int = 1,
        use_grouped_topk: bool = True,
        num_expert_group: int = 1,
        topk_group: int = 1,
        q_lora_rank: int | None = None,
        kv_lora_rank: int | None = None,
        qk_nope_head_dim: int | None = None,
        qk_rope_head_dim: int | None = None,
        v_head_dim: int | None = None,
        mla_use_nope: bool | None = False,
        num_nextn_predict_layers: int = 0,
        linear_attn_config: dict | None = None,
        **kwargs,
    ):
        self.model_type = model_type
        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.head_dim = (
            head_dim if head_dim is not None else hidden_size // num_attention_heads
        )
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
        self.rope_scaling = rope_scaling
```
**EN:** This block defines function `KimiLinearConfig.__init__`. Parameters: self, model_type, vocab_size, hidden_size, head_dim, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, rope_theta, rope_scaling, tie_word_embeddings, moe_intermediate_size, moe_renormalize, moe_router_activation_func, num_experts, num_experts_per_token, num_shared_experts, routed_scaling_factor, first_k_dense_replace, moe_layer_freq, use_grouped_topk, num_expert_group, topk_group, q_lora_rank, kv_lora_rank, qk_nope_head_dim, qk_rope_head_dim, v_head_dim, mla_use_nope, num_nextn_predict_layers, linear_attn_config. This subsection covers lines 13-75 of the same logical block.
**CN:** 该代码块定义函数 `KimiLinearConfig.__init__`。 参数包括 self、model_type、vocab_size、hidden_size、head_dim、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、rope_theta、rope_scaling、tie_word_embeddings、moe_intermediate_size、moe_renormalize、moe_router_activation_func、num_experts、num_experts_per_token、num_shared_experts、routed_scaling_factor、first_k_dense_replace、moe_layer_freq、use_grouped_topk、num_expert_group、topk_group、q_lora_rank、kv_lora_rank、qk_nope_head_dim、qk_rope_head_dim、v_head_dim、mla_use_nope、num_nextn_predict_layers、linear_attn_config。 本小节覆盖同一逻辑块中的第 13-75 行。

### Lines 76-76: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 77-102: Defines function KimiLinearConfig.__init__ (part 2)
```python
        self.q_lora_rank = q_lora_rank
        self.kv_lora_rank = kv_lora_rank
        self.qk_nope_head_dim = qk_nope_head_dim
        self.qk_rope_head_dim = qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.mla_use_nope = mla_use_nope
        # moe config
        self.n_routed_experts = self.num_experts = num_experts
        self.num_experts_per_token = num_experts_per_token
        self.moe_renormalize = moe_renormalize
        self.num_shared_experts = num_shared_experts
        self.routed_scaling_factor = routed_scaling_factor
        self.moe_router_activation_func = moe_router_activation_func
        assert self.moe_router_activation_func in ("softmax", "sigmoid")
        self.moe_intermediate_size = moe_intermediate_size
        self.first_k_dense_replace = first_k_dense_replace
        self.moe_layer_freq = moe_layer_freq
        self.use_grouped_topk = use_grouped_topk
        self.num_expert_group = num_expert_group
        self.topk_group = topk_group
        self.num_nextn_predict_layers = num_nextn_predict_layers

        if linear_attn_config is not None:
            assert linear_attn_config["kda_layers"] is not None
            assert linear_attn_config["full_attn_layers"] is not None
        self.linear_attn_config = linear_attn_config
```
**EN:** This block defines function `KimiLinearConfig.__init__`. Parameters: self, model_type, vocab_size, hidden_size, head_dim, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, rope_theta, rope_scaling, tie_word_embeddings, moe_intermediate_size, moe_renormalize, moe_router_activation_func, num_experts, num_experts_per_token, num_shared_experts, routed_scaling_factor, first_k_dense_replace, moe_layer_freq, use_grouped_topk, num_expert_group, topk_group, q_lora_rank, kv_lora_rank, qk_nope_head_dim, qk_rope_head_dim, v_head_dim, mla_use_nope, num_nextn_predict_layers, linear_attn_config. This subsection covers lines 77-102 of the same logical block.
**CN:** 该代码块定义函数 `KimiLinearConfig.__init__`。 参数包括 self、model_type、vocab_size、hidden_size、head_dim、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、rope_theta、rope_scaling、tie_word_embeddings、moe_intermediate_size、moe_renormalize、moe_router_activation_func、num_experts、num_experts_per_token、num_shared_experts、routed_scaling_factor、first_k_dense_replace、moe_layer_freq、use_grouped_topk、num_expert_group、topk_group、q_lora_rank、kv_lora_rank、qk_nope_head_dim、qk_rope_head_dim、v_head_dim、mla_use_nope、num_nextn_predict_layers、linear_attn_config。 本小节覆盖同一逻辑块中的第 77-102 行。

### Lines 103-103: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 104-110: Defines function KimiLinearConfig.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `KimiLinearConfig.__init__`. Parameters: self, model_type, vocab_size, hidden_size, head_dim, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, rope_theta, rope_scaling, tie_word_embeddings, moe_intermediate_size, moe_renormalize, moe_router_activation_func, num_experts, num_experts_per_token, num_shared_experts, routed_scaling_factor, first_k_dense_replace, moe_layer_freq, use_grouped_topk, num_expert_group, topk_group, q_lora_rank, kv_lora_rank, qk_nope_head_dim, qk_rope_head_dim, v_head_dim, mla_use_nope, num_nextn_predict_layers, linear_attn_config. This subsection covers lines 104-110 of the same logical block.
**CN:** 该代码块定义函数 `KimiLinearConfig.__init__`。 参数包括 self、model_type、vocab_size、hidden_size、head_dim、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、rope_theta、rope_scaling、tie_word_embeddings、moe_intermediate_size、moe_renormalize、moe_router_activation_func、num_experts、num_experts_per_token、num_shared_experts、routed_scaling_factor、first_k_dense_replace、moe_layer_freq、use_grouped_topk、num_expert_group、topk_group、q_lora_rank、kv_lora_rank、qk_nope_head_dim、qk_rope_head_dim、v_head_dim、mla_use_nope、num_nextn_predict_layers、linear_attn_config。 本小节覆盖同一逻辑块中的第 104-110 行。

### Lines 111-111: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 112-121: Defines function KimiLinearConfig.is_mla
```python
    @property
    def is_mla(self):
        return (
            self.q_lora_rank is not None
            or self.kv_lora_rank is not None
            or self.qk_nope_head_dim is not None
            or self.qk_rope_head_dim is not None
            or self.v_head_dim is not None
            or self.mla_use_nope is True
        )
```
**EN:** This block defines function `KimiLinearConfig.is_mla`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.is_mla`。 参数包括 self。 装饰器包括 property。

### Lines 122-122: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 123-125: Defines function KimiLinearConfig.is_moe
```python
    @property
    def is_moe(self):
        return self.num_experts is not None
```
**EN:** This block defines function `KimiLinearConfig.is_moe`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.is_moe`。 参数包括 self。 装饰器包括 property。

### Lines 126-126: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 127-136: Defines function KimiLinearConfig.is_linear_attn
```python
    @property
    def is_linear_attn(self) -> bool:
        return not (
            self.linear_attn_config is None
            or (
                isinstance(self.linear_attn_config, dict)
                and self.linear_attn_config["kda_layers"] is not None
                and len(self.linear_attn_config["kda_layers"]) == 0
            )
        )
```
**EN:** This block defines function `KimiLinearConfig.is_linear_attn`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.is_linear_attn`。 参数包括 self。 装饰器包括 property。

### Lines 137-137: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 138-142: Defines function KimiLinearConfig.is_kda_layer
```python
    def is_kda_layer(self, layer_idx: int):
        return (
            self.linear_attn_config is not None
            and (layer_idx + 1) in self.linear_attn_config["kda_layers"]
        )
```
**EN:** This block defines function `KimiLinearConfig.is_kda_layer`. Parameters: self, layer_idx.
**CN:** 该代码块定义函数 `KimiLinearConfig.is_kda_layer`。 参数包括 self、layer_idx。

### Lines 143-143: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 144-146: Defines function KimiLinearConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self):
        return [i for i in range(self.num_hidden_layers) if self.is_kda_layer(i)]
```
**EN:** This block defines function `KimiLinearConfig.linear_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 147-147: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 148-150: Defines function KimiLinearConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self):
        return [i for i in range(self.num_hidden_layers) if not self.is_kda_layer(i)]
```
**EN:** This block defines function `KimiLinearConfig.full_attention_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 151-151: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiLinearConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiLinearConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 152-163: Defines function KimiLinearConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> KimiLinearCacheParams:
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        shape = KimiLinearStateShape.create(
            tp_world_size=get_attention_tp_size(),
            num_heads=self.linear_attn_config["num_heads"],
            head_dim=self.linear_attn_config["head_dim"],
            conv_kernel_size=self.linear_attn_config["short_conv_kernel_size"],
        )

        return KimiLinearCacheParams(shape=shape, layers=self.linear_layer_ids)
```
**EN:** This block defines function `KimiLinearConfig.mamba2_cache_params`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `KimiLinearConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。

## Key Concepts / 关键概念
- **Classes / 类**: `KimiLinearConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers.configuration_utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
