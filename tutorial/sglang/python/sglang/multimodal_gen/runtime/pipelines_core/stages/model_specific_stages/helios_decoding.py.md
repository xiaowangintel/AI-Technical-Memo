# helios_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/helios_decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for helios decoding, connecting stages, tensors, and runtime metadata. Key symbols include `HeliosDecodingStage`. / 该模块负责 helios decoding 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `HeliosDecodingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Helios-specific decoding stage.

Decodes latent chunks one at a time (matching diffusers HeliosPipeline behavior)
to avoid temporal artifacts at chunk boundaries caused by Wan VAE's causal convolutions.
"""

import torch

from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch, Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.decoding import (
    DecodingStage,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 22-77: Class `HeliosDecodingStage` / 类 `HeliosDecodingStage`
```python
class HeliosDecodingStage(DecodingStage):
    """
    Helios-specific decoding stage that decodes latent chunks independently.

    The Wan VAE uses causal 3D convolutions with feature caching. When decoding
    the full latent sequence at once, the causal conv processes all frames with
    continuous context, producing a different number of output frames per latent
    frame compared to chunk-by-chunk decoding. This causes temporal misalignment
    and visible seams at chunk boundaries.

    This stage decodes each chunk's latents separately (matching diffusers'
    HeliosPipeline behavior) and concatenates the results in pixel space.
    """

# ...
            metrics=batch.metrics,
        )

        return output_batch
```
**EN:** This class models `HeliosDecodingStage` as a specialization of `DecodingStage`. Helios-specific decoding stage that decodes latent chunks independently. Important methods include `forward`.
**CN:** 该类实现 `HeliosDecodingStage`，并继承/扩展 `DecodingStage`。 文档字符串指出：Helios-specific decoding stage that decodes latent chunks independently. 其中较重要的方法包括 `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Video generation flow / 视频生成流程
- Symbol `HeliosDecodingStage` anchors the module API / 符号 `HeliosDecodingStage` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.decoding`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`
- **External / 外部**: `torch`
