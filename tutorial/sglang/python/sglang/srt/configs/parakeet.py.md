# parakeet.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/parakeet.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for parakeet so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 parakeet 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Comments and module notes
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2026 SGLang Team
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
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/transformers_utils/configs/parakeet.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 18-20: Imports dependencies
```python
from dataclasses import dataclass

from transformers import ParakeetEncoderConfig, PretrainedConfig
```
**EN:** This block groups related imports for the module, including dataclasses.dataclass, transformers.ParakeetEncoderConfig, transformers.PretrainedConfig. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 dataclasses.dataclass, transformers.ParakeetEncoderConfig, transformers.PretrainedConfig，为后续代码准备所需名称。

### Lines 21-22: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares class ParakeetConfig
```python
class ParakeetConfig(ParakeetEncoderConfig):
```
**EN:** This block introduces class `ParakeetConfig` as a reusable abstraction inside the module. It inherits from ParakeetEncoderConfig.
**CN:** 该代码块声明类 `ParakeetConfig`，作为模块中的可复用抽象。 它继承自 ParakeetEncoderConfig。

### Lines 24-38: Defines function ParakeetConfig.__init__
```python
    def __init__(
        self,
        llm_hidden_size: int,
        projection_hidden_size: int,
        projection_bias: bool,
        sampling_rate: int,
        projection_eps: float = 1e-5,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.llm_hidden_size = llm_hidden_size
        self.projection_hidden_size = projection_hidden_size
        self.projection_bias = projection_bias
        self.sampling_rate = sampling_rate
        self.projection_eps = projection_eps
```
**EN:** This block defines function `ParakeetConfig.__init__`. Parameters: self, llm_hidden_size, projection_hidden_size, projection_bias, sampling_rate, projection_eps.
**CN:** 该代码块定义函数 `ParakeetConfig.__init__`。 参数包括 self、llm_hidden_size、projection_hidden_size、projection_bias、sampling_rate、projection_eps。

### Lines 39-39: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ParakeetConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ParakeetConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 40-51: Defines function ParakeetConfig.from_hf_config
```python
    @staticmethod
    def from_hf_config(
        config: PretrainedConfig, *, llm_hidden_size: int, max_model_len: int
    ) -> "ParakeetConfig":
        assert isinstance(config, PretrainedConfig)
        return ParakeetConfig(
            **config.to_dict(),
            scale_input=False,
            attention_bias=False,
            llm_hidden_size=llm_hidden_size,
            max_position_embeddings=max_model_len + 1,
        )
```
**EN:** This block defines function `ParakeetConfig.from_hf_config`. Parameters: config. Decorators: staticmethod.
**CN:** 该代码块定义函数 `ParakeetConfig.from_hf_config`。 参数包括 config。 装饰器包括 staticmethod。

### Lines 52-53: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 54-55: Declares class ExtractorConfig
```python
@dataclass(kw_only=True, frozen=True)
class ExtractorConfig:
```
**EN:** This block introduces class `ExtractorConfig` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `ExtractorConfig`，作为模块中的可复用抽象。

### Lines 56-63: Declares feature_size, sampling_rate, subsampling_factor, subsampling_conv_kernel_size, subsampling_conv_stride, and 3 more
```python
    feature_size: int
    sampling_rate: int
    subsampling_factor: int
    subsampling_conv_kernel_size: int
    subsampling_conv_stride: int
    hop_length: int = 160
    clip_duration_s: int = 30
    clip_min_duration_s: float = 0.1
```
**EN:** This block initializes a related set of values in the ExtractorConfig, including feature_size, sampling_rate, subsampling_factor, subsampling_conv_kernel_size, subsampling_conv_stride, and 3 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 ExtractorConfig 中初始化一组相关值，包括 feature_size, sampling_rate, subsampling_factor, subsampling_conv_kernel_size, subsampling_conv_stride 等 3 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 64-64: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ExtractorConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ExtractorConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 65-76: Defines function ExtractorConfig.from_hf_config
```python
    @staticmethod
    def from_hf_config(config: PretrainedConfig) -> "ExtractorConfig":
        assert isinstance(config, PretrainedConfig)
        hop_length = int(getattr(config, "hop_length", ExtractorConfig.hop_length))
        return ExtractorConfig(
            feature_size=config.num_mel_bins,
            sampling_rate=config.sampling_rate,
            hop_length=hop_length,
            subsampling_factor=config.subsampling_factor,
            subsampling_conv_kernel_size=config.subsampling_conv_kernel_size,
            subsampling_conv_stride=config.subsampling_conv_stride,
        )
```
**EN:** This block defines function `ExtractorConfig.from_hf_config`. Parameters: config. Decorators: staticmethod.
**CN:** 该代码块定义函数 `ExtractorConfig.from_hf_config`。 参数包括 config。 装饰器包括 staticmethod。

## Key Concepts / 关键概念
- **Classes / 类**: `ParakeetConfig`, `ExtractorConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`
- **Third-Party / 第三方**: `transformers`
