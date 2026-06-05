# step3p5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/step3p5.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for step3p5 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 step3p5 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Imports dependencies
```python
from typing import Any, Optional

from transformers.configuration_utils import PretrainedConfig
```
**EN:** This block groups related imports for the module, including typing.Any, typing.Optional, transformers.configuration_utils.PretrainedConfig. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Any, typing.Optional, transformers.configuration_utils.PretrainedConfig，为后续代码准备所需名称。

### Lines 4-5: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 6-6: Declares class Step3p5Config
```python
class Step3p5Config(PretrainedConfig):
```
**EN:** This block introduces class `Step3p5Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Step3p5Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 7-8: Declares model_type, architectures
```python
    model_type = "step3p5"
    architectures = ["Step3p5ForCausalLM"]
```
**EN:** This block initializes a related set of values in the Step3p5Config, including model_type, architectures. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Step3p5Config 中初始化一组相关值，包括 model_type, architectures。将这些赋值集中在一起有助于理解周边配置。

### Lines 9-9: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3p5Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3p5Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 10-89: Defines function Step3p5Config.__init__ (part 1)
```python
    def __init__(
        self,
        hidden_size: int = 4096,
        intermediate_size: int = 11264,
        num_attention_heads: int = 64,
        num_attention_groups: int = 8,
        num_hidden_layers: int = 45,
        max_seq_len: int = 128000,
        vocab_size: int = 128815,
        rms_norm_eps: float = 1e-5,
        moe_intermediate_size: int = 1280,
        moe_num_experts: int = 288,
        moe_top_k: int = 8,
        rope_theta: float = 10000,
        rope_scaling: Optional[dict[str, Any]] = None,
        max_position_embeddings: int = 128000,
        share_expert_dims: int = 1280,
        head_dim: int = 128,
        norm_expert_weight: bool = True,
        layer_types: list[str] = None,
        sliding_window: Optional[int] = None,
        moe_layers_enum: tuple[int] = (
            3,
            4,
            5,
            6,
            7,
            8,
            9,
            10,
            11,
            12,
            13,
            14,
            15,
            16,
            17,
            18,
            19,
            20,
            21,
            22,
            23,
            24,
            25,
            26,
            27,
            28,
            29,
            30,
            31,
            32,
            33,
            34,
            35,
            36,
            37,
            38,
            39,
            40,
            41,
            42,
            43,
            44,
        ),
        **kwargs,
    ) -> None:
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_attention_heads = num_attention_heads
        self.num_attention_groups = num_attention_groups
        self.num_hidden_layers = num_hidden_layers
        self.max_seq_len = max_seq_len
        self.vocab_size = vocab_size
        self.rms_norm_eps = rms_norm_eps
        self.moe_intermediate_size = moe_intermediate_size
        self.moe_num_experts = moe_num_experts
        self.moe_top_k = moe_top_k
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
```
**EN:** This block defines function `Step3p5Config.__init__`. Parameters: self, hidden_size, intermediate_size, num_attention_heads, num_attention_groups, num_hidden_layers, max_seq_len, vocab_size, rms_norm_eps, moe_intermediate_size, moe_num_experts, moe_top_k, rope_theta, rope_scaling, max_position_embeddings, share_expert_dims, head_dim, norm_expert_weight, layer_types, sliding_window, moe_layers_enum. This subsection covers lines 10-89 of the same logical block.
**CN:** 该代码块定义函数 `Step3p5Config.__init__`。 参数包括 self、hidden_size、intermediate_size、num_attention_heads、num_attention_groups、num_hidden_layers、max_seq_len、vocab_size、rms_norm_eps、moe_intermediate_size、moe_num_experts、moe_top_k、rope_theta、rope_scaling、max_position_embeddings、share_expert_dims、head_dim、norm_expert_weight、layer_types、sliding_window、moe_layers_enum。 本小节覆盖同一逻辑块中的第 10-89 行。

### Lines 90-106: Defines function Step3p5Config.__init__ (part 2)
```python
        self.max_position_embeddings = max_position_embeddings
        self.share_expert_dim = share_expert_dims
        self.head_dim = head_dim
        self.norm_expert_weight = norm_expert_weight
        self.moe_layers_enum = moe_layers_enum
        self.layer_types = layer_types
        self.sliding_window = sliding_window
        # The upstream Step-3.5-Flash config has layer_types with 48 entries
        # but num_hidden_layers=45. The extra 3 are for MTP/nextn predict
        # layers (indices 45-47) used by Step3p5DecoderLayer during EAGLE
        # speculative decoding. Temporarily align num_hidden_layers to pass
        # the transformers v5.5.3+ validator, then restore the real value.
        real_num_hidden_layers = self.num_hidden_layers
        if layer_types is not None and len(layer_types) != self.num_hidden_layers:
            self.num_hidden_layers = len(layer_types)
        super().__init__(**kwargs)
        self.num_hidden_layers = real_num_hidden_layers
```
**EN:** This block defines function `Step3p5Config.__init__`. Parameters: self, hidden_size, intermediate_size, num_attention_heads, num_attention_groups, num_hidden_layers, max_seq_len, vocab_size, rms_norm_eps, moe_intermediate_size, moe_num_experts, moe_top_k, rope_theta, rope_scaling, max_position_embeddings, share_expert_dims, head_dim, norm_expert_weight, layer_types, sliding_window, moe_layers_enum. This subsection covers lines 90-106 of the same logical block.
**CN:** 该代码块定义函数 `Step3p5Config.__init__`。 参数包括 self、hidden_size、intermediate_size、num_attention_heads、num_attention_groups、num_hidden_layers、max_seq_len、vocab_size、rms_norm_eps、moe_intermediate_size、moe_num_experts、moe_top_k、rope_theta、rope_scaling、max_position_embeddings、share_expert_dims、head_dim、norm_expert_weight、layer_types、sliding_window、moe_layers_enum。 本小节覆盖同一逻辑块中的第 90-106 行。

## Key Concepts / 关键概念
- **Classes / 类**: `Step3p5Config`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`
