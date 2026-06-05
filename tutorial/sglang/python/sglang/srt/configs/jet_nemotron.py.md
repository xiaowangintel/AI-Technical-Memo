# jet_nemotron.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/jet_nemotron.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for jet nemotron so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 jet nemotron 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10: Imports dependencies
```python
from dataclasses import dataclass
from typing import Any

from transformers.configuration_utils import PretrainedConfig

from sglang.srt.configs.mamba_utils import (
    Mamba2CacheParams,
    Mamba2StateShape,
    mamba2_state_dtype,
)
```
**EN:** This block groups related imports for the module, including dataclasses.dataclass, typing.Any, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape, and 1 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 dataclasses.dataclass, typing.Any, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.mamba_utils.Mamba2CacheParams, sglang.srt.configs.mamba_utils.Mamba2StateShape 等 1 项，为后续代码准备所需名称。

### Lines 11-12: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-14: Declares class JetBlockConfig
```python
@dataclass
class JetBlockConfig:
```
**EN:** This block introduces class `JetBlockConfig` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `JetBlockConfig`，作为模块中的可复用抽象。

### Lines 15-22: Declares mode, expand_v, num_heads, head_dim, norm_eps, and 3 more
```python
    mode: str
    expand_v: float
    num_heads: int
    head_dim: int
    norm_eps: str
    conv_size: int
    dconv_generator_reduction: int
    dconv_implementation: str
```
**EN:** This block initializes a related set of values in the JetBlockConfig, including mode, expand_v, num_heads, head_dim, norm_eps, and 3 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 JetBlockConfig 中初始化一组相关值，包括 mode, expand_v, num_heads, head_dim, norm_eps 等 3 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 23-24: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-25: Declares class JetNemotronConfig
```python
class JetNemotronConfig(PretrainedConfig):
```
**EN:** This block introduces class `JetNemotronConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `JetNemotronConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 26-26: Declares model_type
```python
    model_type: str = "jet_nemotron"
```
**EN:** This statement initializes model_type in the JetNemotronConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 JetNemotronConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 27-27: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetNemotronConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetNemotronConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-39: Declares efficient_attention_config, hidden_act, hidden_size, initializer_range, intermediate_size, and 7 more
```python
    efficient_attention_config: dict[str, dict[str, Any]] = None
    hidden_act: str = None
    hidden_size: int = None
    initializer_range: float = None
    intermediate_size: int = None
    layer_types: list[str] = None
    max_position_embeddings: int = None
    num_attention_heads: int = None
    num_key_value_heads: int = None
    rms_norm_eps: float = None
    rope_scaling: None = None
    rope_theta: float = None
```
**EN:** This block initializes a related set of values in the JetNemotronConfig, including efficient_attention_config, hidden_act, hidden_size, initializer_range, intermediate_size, and 7 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 JetNemotronConfig 中初始化一组相关值，包括 efficient_attention_config, hidden_act, hidden_size, initializer_range, intermediate_size 等 7 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 40-40: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetNemotronConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetNemotronConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-47: Defines function JetNemotronConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self) -> list[int]:
        return [
            idx
            for idx, layer_type in enumerate(self.layer_types)
            if layer_type in ("attn", "swa")
        ]
```
**EN:** This block defines function `JetNemotronConfig.full_attention_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetNemotronConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 48-48: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetNemotronConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetNemotronConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 49-55: Defines function JetNemotronConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self) -> list[int]:
        return [
            idx
            for idx, layer_type in enumerate(self.layer_types)
            if layer_type == "jet"
        ]
```
**EN:** This block defines function `JetNemotronConfig.linear_layer_ids`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetNemotronConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。

### Lines 56-56: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JetNemotronConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JetNemotronConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 57-80: Defines function JetNemotronConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Mamba2CacheParams:
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        jet_block_config = JetBlockConfig(**self.efficient_attention_config["jet"])

        num_heads = jet_block_config.num_heads
        head_k_dim = jet_block_config.head_dim
        head_v_dim = int(head_k_dim * jet_block_config.expand_v)
        total_v_dim = num_heads * head_v_dim

        shape = Mamba2StateShape.create(
            tp_world_size=get_attention_tp_size(),
            intermediate_size=total_v_dim,
            n_groups=num_heads,
            num_heads=num_heads,
            head_dim=head_v_dim,
            state_size=head_k_dim,
            conv_kernel=jet_block_config.conv_size,
        )

        return Mamba2CacheParams(
            shape=shape, layers=self.linear_layer_ids, dtype=mamba2_state_dtype(self)
        )
```
**EN:** This block defines function `JetNemotronConfig.mamba2_cache_params`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `JetNemotronConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。

## Key Concepts / 关键概念
- **Classes / 类**: `JetBlockConfig`, `JetNemotronConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
