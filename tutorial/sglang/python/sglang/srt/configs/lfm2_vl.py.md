# lfm2_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/lfm2_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for lfm2 vl so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 lfm2 vl 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Comments and module notes
```python
# Copyright 2026 Liquid AI. All rights reserved.
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

### Lines 13-13: Documents the scope
```python
"""LFM2-VL (Liquid Foundation Model 2 Vision-Language) configuration"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 14-14: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-21: Imports dependencies
```python
from typing import List, Optional

from transformers import CONFIG_MAPPING
from transformers import Lfm2VlConfig as HFLfm2VlConfig
from transformers.utils import logging

from sglang.srt.configs.mamba_utils import Mamba2CacheParams, Mamba2StateShape
```
**EN:** This block groups related imports for the module, including typing.List, typing.Optional, transformers.CONFIG_MAPPING, transformers.Lfm2VlConfig, transformers.utils.logging, and 2 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Optional, transformers.CONFIG_MAPPING, transformers.Lfm2VlConfig, transformers.utils.logging 等 2 项，为后续代码准备所需名称。

### Lines 22-22: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares logger
```python
logger = logging.get_logger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 24-25: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-26: Declares class Lfm2VlConfig
```python
class Lfm2VlConfig(HFLfm2VlConfig):
```
**EN:** This block introduces class `Lfm2VlConfig` as a reusable abstraction inside the module. It inherits from HFLfm2VlConfig. SGLang configuration for LFM2-VL models. Extends HuggingFace's Lfm2VlConfig with hybrid model properties needed by SGLang.
**CN:** 该代码块声明类 `Lfm2VlConfig`，作为模块中的可复用抽象。 它继承自 HFLfm2VlConfig。 文档字符串摘要：SGLang configuration for LFM2-VL models. Extends HuggingFace's Lfm2VlConfig with hybrid model properties needed by SGLang.

### Lines 27-35: Documents the scope
```python
    """
    SGLang configuration for LFM2-VL models.

    Extends HuggingFace's Lfm2VlConfig with hybrid model properties needed by SGLang.
    LFM2-VL combines:
    - SigLip2 vision encoder with NaFlex variable-resolution support
    - LFM2 language model with hybrid attention + short convolution
    - Multimodal projector with pixel unshuffle downsampling
    """
```
**EN:** This string literal serves as documentation for the Lfm2VlConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Lfm2VlConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-44: Defines function Lfm2VlConfig.full_attention_layer_ids
```python
    @property
    def full_attention_layer_ids(self) -> List[int]:
        """Return indices of attention layers for KV cache (from text_config)."""
        return [
            i
            for i, lt in enumerate(self.text_config.layer_types)
            if lt == "full_attention"
        ]
```
**EN:** This block defines function `Lfm2VlConfig.full_attention_layer_ids`. Parameters: self. Decorators: property. Return indices of attention layers for KV cache (from text_config).
**CN:** 该代码块定义函数 `Lfm2VlConfig.full_attention_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return indices of attention layers for KV cache (from text_config).

### Lines 45-45: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 46-53: Defines function Lfm2VlConfig.linear_layer_ids
```python
    @property
    def linear_layer_ids(self) -> List[int]:
        """Return indices of conv layers for conv state cache (from text_config)."""
        return [
            i
            for i, lt in enumerate(self.text_config.layer_types)
            if lt in ("conv", "short_conv")
        ]
```
**EN:** This block defines function `Lfm2VlConfig.linear_layer_ids`. Parameters: self. Decorators: property. Return indices of conv layers for conv state cache (from text_config).
**CN:** 该代码块定义函数 `Lfm2VlConfig.linear_layer_ids`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return indices of conv layers for conv state cache (from text_config).

### Lines 54-54: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 55-58: Defines function Lfm2VlConfig.mamba_chunk_size
```python
    @property
    def mamba_chunk_size(self) -> int:
        """Return chunk size for Mamba2 backend. LFM2 doesn't use chunking, return 1."""
        return 1
```
**EN:** This block defines function `Lfm2VlConfig.mamba_chunk_size`. Parameters: self. Decorators: property. Return chunk size for Mamba2 backend. LFM2 doesn't use chunking, return 1.
**CN:** 该代码块定义函数 `Lfm2VlConfig.mamba_chunk_size`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Return chunk size for Mamba2 backend. LFM2 doesn't use chunking, return 1.

### Lines 59-59: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 60-103: Defines function Lfm2VlConfig.mamba2_cache_params
```python
    @property
    def mamba2_cache_params(self) -> Optional[Mamba2CacheParams]:
        """
        Get cache params for HybridReqToTokenPool initialization.

        LFM2 uses ShortConv layers with a small fixed-size cache (kernel_size - 1).
        Unlike full Mamba2 models, LFM2 only uses the conv state, not SSM temporal state.
        """
        from sglang.srt.layers.dp_attention import get_attention_tp_size

        conv_layer_ids = self.linear_layer_ids
        if not conv_layer_ids:
            return None

        hidden_size = self.text_config.hidden_size
        # conv_L_cache in config is kernel_size (e.g., 3)
        conv_kernel = int(self.text_config.conv_L_cache)

        # get_attention_tp_size() requires initialization, default to 1 if not available
        try:
            tp_size = get_attention_tp_size()
        except (AssertionError, RuntimeError):
            tp_size = 1

        # For ShortConv layers, we use a simplified Mamba2StateShape
        # LFM2 doesn't use SSM state (state_size=0), only conv state
        # We pass num_heads=tp_size so divide(tp_size, tp_size)=1 always works.
        # Since state_size=0, the temporal state shape has zero elements anyway.
        shape = Mamba2StateShape.create(
            tp_world_size=tp_size,
            intermediate_size=hidden_size,
            n_groups=1,  # ShortConv doesn't use grouping
            num_heads=tp_size,  # Ensures divide works; temporal state is empty anyway
            head_dim=hidden_size,  # Conv operates on full hidden dim
            state_size=0,  # No SSM temporal state for ShortConv
            conv_kernel=conv_kernel,
        )

        # Uses default mamba2_state_dtype() which reads SGLANG_MAMBA_CONV_DTYPE env var
        # (defaults to bfloat16). Set SGLANG_MAMBA_CONV_DTYPE=float16 for fp16 inference.
        return Mamba2CacheParams(
            shape=shape,
            layers=conv_layer_ids,
        )
```
**EN:** This block defines function `Lfm2VlConfig.mamba2_cache_params`. Parameters: self. Decorators: property. Get cache params for HybridReqToTokenPool initialization. LFM2 uses ShortConv layers with a small fixed-size cache (kernel_size - 1).
**CN:** 该代码块定义函数 `Lfm2VlConfig.mamba2_cache_params`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Get cache params for HybridReqToTokenPool initialization. LFM2 uses ShortConv layers with a small fixed-size cache (kernel_size - 1).

### Lines 104-108: Comments and module notes
```python


# Override HuggingFace's Lfm2VlConfig with our extended version
# Cannot use .register() because lfm2_vl may already be registered by transformers
# Directly modify the internal _extra_content dict instead
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 109-109: Declares module variables
```python
CONFIG_MAPPING._extra_content["lfm2_vl"] = Lfm2VlConfig
```
**EN:** This statement initializes module variables in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 模块变量。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

## Key Concepts / 关键概念
- **Classes / 类**: `Lfm2VlConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers`, `transformers.utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.layers.dp_attention`
