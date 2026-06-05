# helios_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/helios_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for helios pipeline, connecting stages, tensors, and runtime metadata. Key symbols include `HeliosPipeline`, `HeliosPyramidPipeline`. / 该模块负责 helios pipeline 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `HeliosPipeline`, `HeliosPyramidPipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Helios video diffusion pipeline implementation.

This module contains an implementation of the Helios video diffusion pipeline
using the modular pipeline architecture. Phase 1: T2V only.
"""

from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.pipelines_core.lora_pipeline import LoRAPipeline
from sglang.multimodal_gen.runtime.pipelines_core.stages import (
    InputValidationStage,
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 28-80: Class `HeliosPipeline` / 类 `HeliosPipeline`
```python
class HeliosPipeline(LoRAPipeline, ComposedPipelineBase):
    """
    Helios video diffusion pipeline with LoRA support.

    Implements the Helios T2V pipeline with chunked denoising,
    multi-term memory history, and CFG Zero Star guidance.
    """

    pipeline_name = "HeliosPipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
# ...
        self.add_stage(
            HeliosDecodingStage(vae=self.get_module("vae"), pipeline=self),
            "helios_decoding_stage",
        )
```
**EN:** This class models `HeliosPipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Helios video diffusion pipeline with LoRA support. Important methods include `initialize_pipeline`, `create_pipeline_stages`.
**CN:** 该类实现 `HeliosPipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 文档字符串指出：Helios video diffusion pipeline with LoRA support. 其中较重要的方法包括 `initialize_pipeline`, `create_pipeline_stages`。

### Lines 83-86: Class `HeliosPyramidPipeline` / 类 `HeliosPyramidPipeline`
```python
class HeliosPyramidPipeline(HeliosPipeline):
    """Helios pyramid SR pipeline (used by Helios-Mid and Helios-Distilled)."""

    pipeline_name = "HeliosPyramidPipeline"
```
**EN:** This class models `HeliosPyramidPipeline` as a specialization of `HeliosPipeline`. Helios pyramid SR pipeline (used by Helios-Mid and Helios-Distilled).
**CN:** 该类实现 `HeliosPyramidPipeline`，并继承/扩展 `HeliosPipeline`。 文档字符串指出：Helios pyramid SR pipeline (used by Helios-Mid and Helios-Distilled).

### Lines 87-89: Top-level configuration / 顶层配置
```python


EntryClass = [HeliosPipeline, HeliosPyramidPipeline]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Video generation flow / 视频生成流程
- LoRA adaptation support / LoRA 适配支持
- Symbol `HeliosPipeline` anchors the module API / 符号 `HeliosPipeline` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.lora_pipeline`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.helios_decoding`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.helios_denoising`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
