# legacy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/transformers/legacy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Legacy support code for the Transformers-based backend inside vLLM. The module docstring summarizes it as: "Transformers modeling backend mixin for legacy models." / 为 vLLM 内部基于 Transformers 的后端实现 Legacy 相关支持代码。 模块文档字符串还将其概括为：“Transformers modeling backend mixin for legacy models。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-24)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Copyright 2024 The vLLM team.
#
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
"""Transformers modeling backend mixin for legacy models."""

from typing import TYPE_CHECKING

import torch

from vllm.sequence import IntermediateTensors
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch supply framework primitives, while internal modules like vllm.sequence connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch 这样的外部依赖提供基础框架能力，而 vllm.sequence 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### TYPE_CHECKING branch (lines 25-26)
```python
if TYPE_CHECKING:
    from vllm.config import VllmConfig
```
**EN:** This conditional block keeps optional imports, type-only definitions, or backend-specific branches isolated from the hot runtime path.
**CN:** 这个条件分支把可选导入、仅类型定义或特定后端分支与主要运行路径隔离开。

### Class `LegacyMixin` (lines 29-77)
```python
class LegacyMixin:
    def __init__(self, *, vllm_config: "VllmConfig", prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        # Skip unsupported/unwanted output embeddings layers
        self.skip_prefixes.extend(
            [
                "model.lm_head.",
                "model.predictions.",
                "model.qa_outputs.",
                "model.embeddings_project.",
                "model.discriminator_predictions.",
            ]
        )

        # Some encoder models have the position_ids buffer in the checkpoint.
        # vLLM will always pass position_ids as an argument, so we skip loading
        # the buffer if it exists
        self.skip_substrs.append("position_ids")

        # Some encoder models have the bias of the final classifier layer
        # in the checkpoint. vLLM does not use this bias, so we skip loading
        # it if it exists
        self.skip_substrs.append("score.bias")

        # roberta-like models an extra padding in positions.
        # FIXME(Isotr0py): This is quite hacky for roberta edge case,
        # we should find a better way to handle this.
        self.is_roberta = "roberta" in self.text_config.model_type
        self.padding_idx = self.text_config.pad_token_id

    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if self.is_roberta:
            # RoBERTa positions start at padding_idx + 1.
            # Non-in-place add to avoid mutating the persistent GPU buffer --
            # in-place += would accumulate on CUDA graph padding slots.
            positions = positions + self.padding_idx + 1
        return super().forward(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
```
**EN:** Defines `LegacyMixin`, a supporting module used by the surrounding model implementation. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LegacyMixin`，它是一个被周边模型实现复用的支撑模块。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.sequence
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
