# text_encoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/text_encoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for text encoding, connecting stages, tensors, and runtime metadata. Key symbols include `TextEncodingFingerprint`, `stack_tensors`, `TextEncodingStage`. / 该模块负责 text encoding 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `TextEncodingFingerprint`, `stack_tensors`, `TextEncodingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Prompt encoding stages for diffusion pipelines.

This module contains implementations of prompt encoding stages for diffusion pipelines.
"""

import inspect
from dataclasses import dataclass
from typing import Any

import torch
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 37-43: Class `TextEncodingFingerprint` / 类 `TextEncodingFingerprint`
```python
@dataclass(frozen=True)
class TextEncodingFingerprint:
    prompt: Any
    negative_prompt: Any
    do_classifier_free_guidance: bool
    prompt_template: Any
    max_sequence_length: int | None
```
**EN:** This class models `TextEncodingFingerprint`.
**CN:** 该类实现 `TextEncodingFingerprint`。

### Lines 46-53: Function `stack_tensors` / 函数 `stack_tensors`
```python
def stack_tensors(name: str, tensors: list[torch.Tensor]) -> torch.Tensor:
    base_shape = list(tensors[0].shape)
    for tensor in tensors[1:]:
        if list(tensor.shape) != base_shape:
            raise ValueError(
                f"Cannot stack {name} with differing shapes: {[list(t.shape) for t in tensors]}"
            )
    return torch.stack(tensors, dim=0)
```
**EN:** This function drives `stack_tensors` with inputs such as `name`, `tensors`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `stack_tensors`，主要处理 `name`, `tensors` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 56-662: Class `TextEncodingStage` / 类 `TextEncodingStage`
```python
class TextEncodingStage(PipelineStage):
    """
    Stage for encoding text prompts into embeddings for diffusion models.

    This stage handles the encoding of text prompts into the embedding space
    expected by the diffusion model.
    """

    deduplicated_output_fields = (
        "prompt_embeds",
        "negative_prompt_embeds",
        "prompt_attention_mask",
        "negative_attention_mask",
        "prompt_embeds_mask",
# ...
        if batch.debug:
            logger.debug(f"{batch.prompt_embeds=}")
            logger.debug(f"{batch.negative_prompt_embeds=}")
        return result
```
**EN:** This class models `TextEncodingStage` as a specialization of `PipelineStage`. Stage for encoding text prompts into embeddings for diffusion models. Important methods include `__init__`, `component_uses`, `get_or_compute_negative_text_embedding`, `_build_negative_text_cache_key`.
**CN:** 该类实现 `TextEncodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for encoding text prompts into embeddings for diffusion models. 其中较重要的方法包括 `__init__`, `component_uses`, `get_or_compute_negative_text_embedding`, `_build_negative_text_cache_key`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `inspect`, `dataclasses`, `typing`
