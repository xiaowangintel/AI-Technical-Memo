# deepseek_v4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/deepseek_v4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for deepseek v4 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 deepseek v4 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Imports dependencies
```python
import logging
import os
from dataclasses import dataclass, field
from typing import Dict, List, Optional

from transformers import PretrainedConfig

from sglang.srt.layers.quantization.base_config import QuantizationConfig
```
**EN:** This block groups related imports for the module, including logging, os, dataclasses.dataclass, dataclasses.field, typing.Dict, and 4 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, os, dataclasses.dataclass, dataclasses.field, typing.Dict 等 4 项，为后续代码准备所需名称。

### Lines 9-9: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 10-10: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 11-12: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-44: Defines function try_detect_fp4_experts
```python
def try_detect_fp4_experts(model_path: str) -> Optional[bool]:
    """True = mxfp4-packed (U8/I8/F4), False = converted FP8 (F8_E4M3),
    None when the header isn't readable (HF slug not cached yet, etc.).
    Caller falls back to user default. Pure read; never mutates env.
    """
    from sglang.srt.model_loader.weight_utils import (
        probe_routed_expert_weight_dtype,
    )
    from sglang.srt.utils import find_local_repo_dir

    if os.path.isdir(model_path):
        local_path = model_path
    else:
        local_path = find_local_repo_dir(model_path)
        if not local_path or not os.path.isdir(local_path):
            return None

    try:
        dtype = probe_routed_expert_weight_dtype(local_path)
    except Exception as e:
        logger.warning("Failed to probe routed-expert dtype for %s: %s", model_path, e)
        return None
    if dtype is None:
        return None
    if dtype in ("U8", "I8", "F4"):
        return True
    if dtype == "F8_E4M3":
        return False
    logger.warning(
        "Unexpected routed-expert safetensors dtype=%s for DeepSeek V4", dtype
    )
    return None
```
**EN:** This block defines function `try_detect_fp4_experts`. Parameters: model_path. True = mxfp4-packed (U8/I8/F4), False = converted FP8 (F8_E4M3), None when the header isn't readable (HF slug not cached yet, etc.). Caller falls back to user default.
**CN:** 该代码块定义函数 `try_detect_fp4_experts`。 参数包括 model_path。 文档字符串摘要：True = mxfp4-packed (U8/I8/F4), False = converted FP8 (F8_E4M3), None when the header isn't readable (HF slug not cached yet, etc.). Caller falls back to user default.

### Lines 45-46: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-48: Declares class DeepSeekV4Config
```python
@dataclass(kw_only=True)
class DeepSeekV4Config(PretrainedConfig):
```
**EN:** This block introduces class `DeepSeekV4Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepSeekV4Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 49-71: Declares architectures, attention_bias, attention_dropout, bos_token_id, eos_token_id, and 18 more
```python
    architectures: List[str]
    attention_bias: bool = False
    attention_dropout: float = 0.0
    bos_token_id: int = 0
    eos_token_id: int = 1
    ep_size: int = 1
    first_k_dense_replace: int = 0
    hidden_act: str = "silu"
    hidden_size: int = 4096
    index_head_dim: int = 128
    index_n_heads: int = 64
    index_topk: int = 512
    initializer_range: float = 0.02
    intermediate_size: int = 2048
    kv_lora_rank: int = 512
    max_position_embeddings: int = 65536
    model_type: str = "deepseek_v4"
    moe_intermediate_size: int = 2048
    moe_layer_freq: int = 1
    n_group: int = 8
    n_routed_experts: int = 256
    n_shared_experts: int = 1
    norm_topk_prob: bool = True
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including architectures, attention_bias, attention_dropout, bos_token_id, eos_token_id, and 18 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 architectures, attention_bias, attention_dropout, bos_token_id, eos_token_id 等 18 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 72-72: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 73-76: Declares num_attention_heads, num_experts_per_tok, num_hidden_layers, num_key_value_heads
```python
    num_attention_heads: int = 64
    num_experts_per_tok: int = 6
    num_hidden_layers: int = 43
    num_key_value_heads: int = 1
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including num_attention_heads, num_experts_per_tok, num_hidden_layers, num_key_value_heads. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 num_attention_heads, num_experts_per_tok, num_hidden_layers, num_key_value_heads。将这些赋值集中在一起有助于理解周边配置。

### Lines 77-77: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 78-80: Declares q_lora_rank, qk_nope_head_dim, qk_rope_head_dim
```python
    q_lora_rank: int = 1024
    qk_nope_head_dim: int = 448
    qk_rope_head_dim: int = 64
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including q_lora_rank, qk_nope_head_dim, qk_rope_head_dim. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 q_lora_rank, qk_nope_head_dim, qk_rope_head_dim。将这些赋值集中在一起有助于理解周边配置。

### Lines 81-81: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 82-82: Declares quantization_config
```python
    quantization_config: QuantizationConfig = field(default_factory=QuantizationConfig)
```
**EN:** This statement initializes quantization_config in the DeepSeekV4Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 DeepSeekV4Config 中初始化 quantization_config。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 83-83: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 84-84: Declares rms_norm_eps
```python
    rms_norm_eps: float = 1e-6
```
**EN:** This statement initializes rms_norm_eps in the DeepSeekV4Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 DeepSeekV4Config 中初始化 rms_norm_eps。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 85-85: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 86-87: Declares rope_scaling, rope_theta
```python
    rope_scaling: Dict[str, float] = field(default_factory=dict)
    rope_theta: int = 10000
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including rope_scaling, rope_theta. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 rope_scaling, rope_theta。将这些赋值集中在一起有助于理解周边配置。

### Lines 88-88: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 89-90: Declares routed_scaling_factor, scoring_func
```python
    routed_scaling_factor: float = 1.5
    scoring_func: str = "sqrtsoftplus"
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including routed_scaling_factor, scoring_func. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 routed_scaling_factor, scoring_func。将这些赋值集中在一起有助于理解周边配置。

### Lines 91-91: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 92-92: Declares tie_word_embeddings
```python
    tie_word_embeddings: bool = False
```
**EN:** This statement initializes tie_word_embeddings in the DeepSeekV4Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 DeepSeekV4Config 中初始化 tie_word_embeddings。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 93-93: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 94-95: Declares topk_group, topk_method
```python
    topk_group: int = 8
    topk_method: str = "noaux_tc"
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including topk_group, topk_method. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 topk_group, topk_method。将这些赋值集中在一起有助于理解周边配置。

### Lines 96-96: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 97-102: Declares use_cache, v_head_dim, vocab_size, o_lora_rank, o_groups, and 1 more
```python
    use_cache: bool = True
    v_head_dim: int = 512
    vocab_size: int = 129280
    o_lora_rank: int = 1024
    o_groups: int = 8
    window_size: int = 128
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including use_cache, v_head_dim, vocab_size, o_lora_rank, o_groups, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 use_cache, v_head_dim, vocab_size, o_lora_rank, o_groups 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 103-103: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 104-105: Declares compress_rope_theta, compress_ratios
```python
    compress_rope_theta: int = 40000
    compress_ratios: List[int] = field(default_factory=list)
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including compress_rope_theta, compress_ratios. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 compress_rope_theta, compress_ratios。将这些赋值集中在一起有助于理解周边配置。

### Lines 106-106: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepSeekV4Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepSeekV4Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 107-110: Declares n_hash_layers, hc_mult, hc_sinkhorn_iters, hc_eps
```python
    n_hash_layers: int = 3
    hc_mult: int = 4
    hc_sinkhorn_iters: int = 20
    hc_eps: float = 1e-6
```
**EN:** This block initializes a related set of values in the DeepSeekV4Config, including n_hash_layers, hc_mult, hc_sinkhorn_iters, hc_eps. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepSeekV4Config 中初始化一组相关值，包括 n_hash_layers, hc_mult, hc_sinkhorn_iters, hc_eps。将这些赋值集中在一起有助于理解周边配置。

## Key Concepts / 关键概念
- **Classes / 类**: `DeepSeekV4Config`
- **Functions / 函数**: `try_detect_fp4_experts`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `logging`, `os`, `typing`
- **Third-Party / 第三方**: `transformers`
- **Local Modules / 本地模块**: `sglang.srt.layers.quantization.base_config`, `sglang.srt.model_loader.weight_utils`, `sglang.srt.utils`
