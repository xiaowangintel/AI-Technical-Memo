# encoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/encoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for encoding, connecting stages, tensors, and runtime metadata. Key symbols include `EncodingStage`. / 该模块负责 encoding 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `EncodingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Encoding stage for diffusion pipelines.
"""

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.memory_managers.component_manager import (
    ComponentUse,
)
from sglang.multimodal_gen.runtime.models.vaes.common import ParallelTiledVAE
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-119: Class `EncodingStage` / 类 `EncodingStage`
```python
class EncodingStage(PipelineStage):
    """
    Stage for encoding pixel space representations into latent space.

    This stage handles the encoding of pixel-space video/images into latent
    representations for further processing in the diffusion pipeline.
    """

    def __init__(self, vae: ParallelTiledVAE) -> None:
        super().__init__()
        self.vae: ParallelTiledVAE = vae

    def component_uses(
        self, server_args: ServerArgs, stage_name: str | None = None
# ...
        # Update batch with encoded latents
        batch.latents = latents

        return batch
```
**EN:** This class models `EncodingStage` as a specialization of `PipelineStage`. Stage for encoding pixel space representations into latent space. Important methods include `__init__`, `component_uses`, `verify_input`, `verify_output`.
**CN:** 该类实现 `EncodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for encoding pixel space representations into latent space. 其中较重要的方法包括 `__init__`, `component_uses`, `verify_input`, `verify_output`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.models.vaes.common`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `torch`
