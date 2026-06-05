# afmoe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/afmoe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for afmoe so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 afmoe 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Imports dependencies
```python
from typing import List, Optional

from transformers import PretrainedConfig
```
**EN:** This block groups related imports for the module, including typing.List, typing.Optional, transformers.PretrainedConfig. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Optional, transformers.PretrainedConfig，为后续代码准备所需名称。

### Lines 4-5: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 6-6: Declares class AfmoeConfig
```python
class AfmoeConfig(PretrainedConfig):
```
**EN:** This block introduces class `AfmoeConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `AfmoeConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 7-7: Declares model_type
```python
    model_type = "afmoe"
```
**EN:** This statement initializes model_type in the AfmoeConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 AfmoeConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 8-8: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the AfmoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 AfmoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 9-86: Defines function AfmoeConfig.__init__ (part 1)
```python
    def __init__(
        self,
        vocab_size: int = 32000,
        hidden_size: int = 4096,
        intermediate_size: int = 11008,
        moe_intermediate_size: int = 256,
        num_hidden_layers: int = 32,
        num_attention_heads: int = 32,
        num_key_value_heads: Optional[int] = None,
        head_dim: Optional[int] = None,
        hidden_act: str = "silu",
        max_position_embeddings: int = 131072,
        initializer_range: float = 0.02,
        rms_norm_eps: float = 1e-5,
        use_cache: bool = True,
        pad_token_id: Optional[int] = None,
        bos_token_id: int = 1,
        eos_token_id: int = 2,
        tie_word_embeddings: bool = False,
        rope_theta: float = 10000.0,
        rope_scaling: Optional[dict] = None,
        attention_bias: bool = False,
        attention_dropout: float = 0.0,
        # MoE parameters
        num_experts: Optional[int] = None,
        num_experts_per_tok: Optional[int] = None,
        num_shared_experts: int = 0,
        num_dense_layers: int = 0,
        # Routing parameters
        score_func: str = "sigmoid",
        route_norm: bool = True,
        route_scale: float = 1.0,
        n_group: int = 1,
        topk_group: int = 1,
        # Attention parameters
        sliding_window: Optional[int] = None,
        layer_types: Optional[List[str]] = None,
        global_attn_every_n_layers: int = 4,
        # muP scaling
        mup_enabled: bool = False,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.moe_intermediate_size = moe_intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads

        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads

        self.num_key_value_heads = num_key_value_heads
        self.head_dim = (
            head_dim if head_dim is not None else hidden_size // num_attention_heads
        )
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout

        # MoE parameters
        self.num_experts = num_experts
        self.num_experts_per_tok = num_experts_per_tok
        self.num_shared_experts = num_shared_experts
        self.num_dense_layers = num_dense_layers

        # Routing parameters
        self.score_func = score_func
        self.route_norm = route_norm
        self.route_scale = route_scale
        self.n_group = n_group
        self.topk_group = topk_group
```
**EN:** This block defines function `AfmoeConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, num_experts, num_experts_per_tok, num_shared_experts, num_dense_layers, score_func, route_norm, route_scale, n_group, topk_group, sliding_window, layer_types, global_attn_every_n_layers, mup_enabled. This subsection covers lines 9-86 of the same logical block.
**CN:** 该代码块定义函数 `AfmoeConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、num_experts、num_experts_per_tok、num_shared_experts、num_dense_layers、score_func、route_norm、route_scale、n_group、topk_group、sliding_window、layer_types、global_attn_every_n_layers、mup_enabled。 本小节覆盖同一逻辑块中的第 9-86 行。

### Lines 87-87: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 88-94: Defines function AfmoeConfig.__init__ (part 2)
```python
        # Attention parameters
        self.sliding_window = sliding_window
        self.layer_types = layer_types
        self.global_attn_every_n_layers = global_attn_every_n_layers

        # muP scaling
        self.mup_enabled = mup_enabled
```
**EN:** This block defines function `AfmoeConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, num_experts, num_experts_per_tok, num_shared_experts, num_dense_layers, score_func, route_norm, route_scale, n_group, topk_group, sliding_window, layer_types, global_attn_every_n_layers, mup_enabled. This subsection covers lines 88-94 of the same logical block.
**CN:** 该代码块定义函数 `AfmoeConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、num_experts、num_experts_per_tok、num_shared_experts、num_dense_layers、score_func、route_norm、route_scale、n_group、topk_group、sliding_window、layer_types、global_attn_every_n_layers、mup_enabled。 本小节覆盖同一逻辑块中的第 88-94 行。

### Lines 95-95: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 96-102: Defines function AfmoeConfig.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `AfmoeConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, num_experts, num_experts_per_tok, num_shared_experts, num_dense_layers, score_func, route_norm, route_scale, n_group, topk_group, sliding_window, layer_types, global_attn_every_n_layers, mup_enabled. This subsection covers lines 96-102 of the same logical block.
**CN:** 该代码块定义函数 `AfmoeConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、num_experts、num_experts_per_tok、num_shared_experts、num_dense_layers、score_func、route_norm、route_scale、n_group、topk_group、sliding_window、layer_types、global_attn_every_n_layers、mup_enabled。 本小节覆盖同一逻辑块中的第 96-102 行。

## Key Concepts / 关键概念
- **Classes / 类**: `AfmoeConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers`
