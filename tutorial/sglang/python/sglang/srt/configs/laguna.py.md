# laguna.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/laguna.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for laguna so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 laguna 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Comments and module notes
```python
# coding=utf-8
# Copyright 2023-2026 SGLang Team
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 9-9: Documents the scope
```python
"""Laguna (poolside/Laguna-XS.2) model configuration."""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 10-10: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 11-16: Imports dependencies
```python
from __future__ import annotations

from typing import Any, Dict, List, Optional

from transformers.configuration_utils import PretrainedConfig
from transformers.utils import logging
```
**EN:** This block groups related imports for the module, including __future__.annotations, typing.Any, typing.Dict, typing.List, typing.Optional, and 2 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 __future__.annotations, typing.Any, typing.Dict, typing.List, typing.Optional 等 2 项，为后续代码准备所需名称。

### Lines 17-17: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 18-18: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 19-20: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 21-23: Defines function _first_not_none
```python
def _first_not_none(*candidates: Any) -> Any:
    """First non-None candidate. Unlike `a or b`, preserves falsy values."""
    return next((c for c in candidates if c is not None), None)
```
**EN:** This block defines function `_first_not_none`. It takes no explicit parameters. First non-None candidate. Unlike `a or b`, preserves falsy values.
**CN:** 该代码块定义函数 `_first_not_none`。 它没有显式参数。 文档字符串摘要：First non-None candidate. Unlike `a or b`, preserves falsy values.

### Lines 24-25: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-57: Defines function _to_sglang_rope_scaling
```python
def _to_sglang_rope_scaling(rope_params: Dict[str, Any]) -> Optional[Dict[str, Any]]:
    """HF per-layer rope dict → SGLang `get_rope` `rope_scaling`. None means plain RoPE."""
    if not rope_params:
        return None
    rope_type = rope_params.get("rope_type") or rope_params.get("type")
    if rope_type in (None, "default"):
        return None

    out: Dict[str, Any] = {"rope_type": rope_type}
    pass_through = (
        "factor",
        "original_max_position_embeddings",
        "beta_fast",
        "beta_slow",
        "extrapolation_factor",
        "truncate",
        "low_freq_factor",
        "high_freq_factor",
        "mscale",
        "mscale_all_dim",
        "short_factor",
        "long_factor",
        "short_mscale",
        "long_mscale",
    )
    for key in pass_through:
        if key in rope_params:
            out[key] = rope_params[key]
    if "attention_factor" in rope_params:
        # HF spells it attention_factor; SGLang's factory reads attn_factor.
        out["attn_factor"] = rope_params["attention_factor"]
    return out
```
**EN:** This block defines function `_to_sglang_rope_scaling`. Parameters: rope_params. HF per-layer rope dict → SGLang `get_rope` `rope_scaling`. None means plain RoPE.
**CN:** 该代码块定义函数 `_to_sglang_rope_scaling`。 参数包括 rope_params。 文档字符串摘要：HF per-layer rope dict → SGLang `get_rope` `rope_scaling`. None means plain RoPE.

### Lines 58-59: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 60-60: Declares class LagunaConfig
```python
class LagunaConfig(PretrainedConfig):
```
**EN:** This block introduces class `LagunaConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `LagunaConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 61-62: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "laguna"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the LagunaConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 LagunaConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 63-63: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LagunaConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LagunaConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 64-131: Defines function LagunaConfig.__init__ (part 1)
```python
    def __init__(
        self,
        vocab_size: int = 100352,
        hidden_size: int = 2048,
        intermediate_size: int = 8192,
        num_hidden_layers: int = 40,
        num_attention_heads: int = 48,
        num_key_value_heads: int = 8,
        head_dim: int = 128,
        hidden_act: str = "silu",
        max_position_embeddings: int = 131072,
        initializer_range: float = 0.02,
        rms_norm_eps: float = 1e-6,
        use_cache: bool = True,
        tie_word_embeddings: bool = False,
        attention_bias: bool = False,
        attention_dropout: float = 0.0,
        sliding_window: int = 512,
        layer_types: Optional[List[str]] = None,
        mlp_layer_types: Optional[List[str]] = None,
        num_attention_heads_per_layer: Optional[List[int]] = None,
        num_experts: int = 256,
        num_experts_per_tok: int = 8,
        moe_intermediate_size: int = 512,
        shared_expert_intermediate_size: int = 512,
        moe_routed_scaling_factor: float = 1.0,
        moe_router_logit_softcapping: float = 0.0,
        moe_apply_router_weight_on_input: bool = False,
        # Per-layer-type rope dict; nested under "full_attention" / "sliding_attention".
        rope_parameters: Optional[Dict[str, Any]] = None,
        partial_rotary_factor: Optional[float] = None,
        rope_theta: Optional[float] = None,
        rope_scaling: Optional[Dict[str, Any]] = None,
        bos_token_id: Optional[int] = 2,
        eos_token_id: Optional[Any] = None,
        pad_token_id: Optional[int] = 9,
        **kwargs,
    ):
        super().__init__(
            tie_word_embeddings=tie_word_embeddings,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            pad_token_id=pad_token_id,
            **kwargs,
        )

        self.vocab_size = vocab_size
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_key_value_heads = num_key_value_heads
        self.head_dim = head_dim
        self.hidden_act = hidden_act
        self.max_position_embeddings = max_position_embeddings
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        self.sliding_window = sliding_window

        self.num_experts = num_experts
        self.num_experts_per_tok = num_experts_per_tok
        self.moe_intermediate_size = moe_intermediate_size
        self.shared_expert_intermediate_size = shared_expert_intermediate_size
        self.moe_routed_scaling_factor = moe_routed_scaling_factor
        self.moe_router_logit_softcapping = moe_router_logit_softcapping
        self.moe_apply_router_weight_on_input = moe_apply_router_weight_on_input
```
**EN:** This block defines function `LagunaConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, attention_bias, attention_dropout, sliding_window, layer_types, mlp_layer_types, num_attention_heads_per_layer, num_experts, num_experts_per_tok, moe_intermediate_size, shared_expert_intermediate_size, moe_routed_scaling_factor, moe_router_logit_softcapping, moe_apply_router_weight_on_input, rope_parameters, partial_rotary_factor, rope_theta, rope_scaling, bos_token_id, eos_token_id, pad_token_id. This subsection covers lines 64-131 of the same logical block.
**CN:** 该代码块定义函数 `LagunaConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、attention_bias、attention_dropout、sliding_window、layer_types、mlp_layer_types、num_attention_heads_per_layer、num_experts、num_experts_per_tok、moe_intermediate_size、shared_expert_intermediate_size、moe_routed_scaling_factor、moe_router_logit_softcapping、moe_apply_router_weight_on_input、rope_parameters、partial_rotary_factor、rope_theta、rope_scaling、bos_token_id、eos_token_id、pad_token_id。 本小节覆盖同一逻辑块中的第 64-131 行。

### Lines 132-132: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 133-192: Defines function LagunaConfig.__init__ (part 2)
```python
        # Synthesise per-layer schedules when the caller omits them so the model
        # file can index by layer_id without per-call guards.
        self.layer_types = (
            list(layer_types)
            if layer_types
            else [
                "full_attention" if i % 4 == 0 else "sliding_attention"
                for i in range(num_hidden_layers)
            ]
        )
        self.mlp_layer_types = (
            list(mlp_layer_types)
            if mlp_layer_types
            else (["dense"] + ["sparse"] * (num_hidden_layers - 1))
        )
        self.num_attention_heads_per_layer = (
            list(num_attention_heads_per_layer)
            if (num_attention_heads_per_layer)
            else [num_attention_heads] * num_hidden_layers
        )

        # SGLang's hybrid-SWA core reads `swa_*` KV/head_dim from hf_text_config.
        # Per-layer Q-head count is read directly from num_attention_heads_per_layer.
        # Pure-SWA models would have no full_attention layer, but the synthesized
        # default above always plants one at index 0; let .index() raise if a
        # caller passes an all-sliding layer_types — silent fallback would wire
        # the SWA head count into a "full" attribute and corrupt downstream sizes.
        full_idx = self.layer_types.index("full_attention")
        self.num_attention_heads = self.num_attention_heads_per_layer[full_idx]
        self.swa_num_key_value_heads = num_key_value_heads
        self.swa_head_dim = head_dim
        self.swa_v_head_dim = head_dim

        # Released checkpoint nests rope_parameters under layer-type keys.
        rp = rope_parameters if isinstance(rope_parameters, dict) else {}
        full_rp = rp.get("full_attention") or {}
        swa_rp = rp.get("sliding_attention") or {}

        # transformers v5 aliases `rope_scaling` ↔ `rope_parameters` on
        # PretrainedConfig — writing one clobbers the other. Keep the nested
        # form on those two slots (so HF's reference modeling code can index
        # rope_parameters[layer_type] when invoked via trust_remote_code) and
        # publish our SGLang-shaped flat rope dicts under different names.
        self.rope_parameters = rope_parameters

        self.rope_theta = _first_not_none(
            full_rp.get("rope_theta"), rope_theta, 10000.0
        )
        self.partial_rotary_factor = _first_not_none(
            full_rp.get("partial_rotary_factor"), partial_rotary_factor, 1.0
        )
        self.full_rope_scaling = _first_not_none(
            _to_sglang_rope_scaling(full_rp), rope_scaling
        )

        self.swa_rope_theta = _first_not_none(swa_rp.get("rope_theta"), self.rope_theta)
        self.swa_partial_rotary_factor = _first_not_none(
            swa_rp.get("partial_rotary_factor"), self.partial_rotary_factor
        )
        self.swa_rope_scaling = _to_sglang_rope_scaling(swa_rp)
```
**EN:** This block defines function `LagunaConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, attention_bias, attention_dropout, sliding_window, layer_types, mlp_layer_types, num_attention_heads_per_layer, num_experts, num_experts_per_tok, moe_intermediate_size, shared_expert_intermediate_size, moe_routed_scaling_factor, moe_router_logit_softcapping, moe_apply_router_weight_on_input, rope_parameters, partial_rotary_factor, rope_theta, rope_scaling, bos_token_id, eos_token_id, pad_token_id. This subsection covers lines 133-192 of the same logical block.
**CN:** 该代码块定义函数 `LagunaConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、attention_bias、attention_dropout、sliding_window、layer_types、mlp_layer_types、num_attention_heads_per_layer、num_experts、num_experts_per_tok、moe_intermediate_size、shared_expert_intermediate_size、moe_routed_scaling_factor、moe_router_logit_softcapping、moe_apply_router_weight_on_input、rope_parameters、partial_rotary_factor、rope_theta、rope_scaling、bos_token_id、eos_token_id、pad_token_id。 本小节覆盖同一逻辑块中的第 133-192 行。

### Lines 193-193: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 194-209: Defines function LagunaConfig.__init__ (part 3)
```python
        # DeepSeek-style aliases consumed by cross-cutting infra outside this
        # model file: `lora/mem_pool.py` and `lora/utils.py` read
        # `n_routed_experts` / `n_shared_experts` / `first_k_dense_replace`,
        # `elastic_ep/expert_backup_*` reads `n_routed_experts`. The
        # hardcoded `n_shared_experts=1` and `norm_topk_prob=True` reflect
        # Laguna's fixed architecture (one shared expert, sigmoid-renormalized
        # top-k routing).
        self.n_routed_experts = num_experts
        self.n_shared_experts = 1
        self.routed_scaling_factor = moe_routed_scaling_factor
        self.norm_topk_prob = True
        self.first_k_dense_replace = (
            self.mlp_layer_types.index("sparse")
            if "sparse" in self.mlp_layer_types
            else num_hidden_layers
        )
```
**EN:** This block defines function `LagunaConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, attention_bias, attention_dropout, sliding_window, layer_types, mlp_layer_types, num_attention_heads_per_layer, num_experts, num_experts_per_tok, moe_intermediate_size, shared_expert_intermediate_size, moe_routed_scaling_factor, moe_router_logit_softcapping, moe_apply_router_weight_on_input, rope_parameters, partial_rotary_factor, rope_theta, rope_scaling, bos_token_id, eos_token_id, pad_token_id. This subsection covers lines 194-209 of the same logical block.
**CN:** 该代码块定义函数 `LagunaConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、attention_bias、attention_dropout、sliding_window、layer_types、mlp_layer_types、num_attention_heads_per_layer、num_experts、num_experts_per_tok、moe_intermediate_size、shared_expert_intermediate_size、moe_routed_scaling_factor、moe_router_logit_softcapping、moe_apply_router_weight_on_input、rope_parameters、partial_rotary_factor、rope_theta、rope_scaling、bos_token_id、eos_token_id、pad_token_id。 本小节覆盖同一逻辑块中的第 194-209 行。

## Key Concepts / 关键概念
- **Classes / 类**: `LagunaConfig`
- **Functions / 函数**: `_first_not_none`, `_to_sglang_rope_scaling`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `__future__`, `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`, `transformers.utils`
