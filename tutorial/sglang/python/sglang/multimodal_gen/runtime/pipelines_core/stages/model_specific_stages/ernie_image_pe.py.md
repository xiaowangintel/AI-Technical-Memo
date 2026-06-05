# ernie_image_pe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/ernie_image_pe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for ernie image pe, connecting stages, tensors, and runtime metadata. Key symbols include `PromptEnhancementStage`. / 该模块负责 ernie image pe 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `PromptEnhancementStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Prompt enhancement stage for ErnieImage pipeline.
"""

import json

import torch

from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.base import PipelineStage
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 18-98: Class `PromptEnhancementStage` / 类 `PromptEnhancementStage`
```python
class PromptEnhancementStage(PipelineStage):

    def __init__(self, pe_model, pe_tokenizer):
        super().__init__()
        self.pe_model = pe_model
        self.pe_tokenizer = pe_tokenizer

    @torch.no_grad()
    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        # Skip if use_pe is disabled or tokenizer unavailable
        use_pe = getattr(batch, "use_pe", True)
        if not use_pe or self.pe_model is None:
            return batch

# ...
            sampling_params=sampling_params,
        )

        return output["text"].strip()
```
**EN:** This class models `PromptEnhancementStage` as a specialization of `PipelineStage`. Important methods include `__init__`, `forward`, `_enhance_single_prompt`.
**CN:** 该类实现 `PromptEnhancementStage`，并继承/扩展 `PipelineStage`。 其中较重要的方法包括 `__init__`, `forward`, `_enhance_single_prompt`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Sampling parameter control / 采样参数控制
- Image generation flow / 图像生成流程
- Symbol `PromptEnhancementStage` anchors the module API / 符号 `PromptEnhancementStage` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `json`
