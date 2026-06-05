# sana.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/sana.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for sana, connecting stages, tensors, and runtime metadata. Key symbols include `SanaPipeline`. / 该模块负责 sana 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `SanaPipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
#
# SANA text-to-image pipeline.
#
# Stage order matches Flux (InputValidation -> TextEncoding -> TimestepPrep ->
# LatentPrep -> Denoising -> Decoding) rather than the add_standard_t2i_stages
# helper (which puts LatentPrep before TimestepPrep). Both orderings are
# functionally equivalent since these stages are independent.
#
# SANA uses a single text encoder (Gemma2), so only one text_encoder + tokenizer
# pair is registered — unlike Flux which has text_encoder + text_encoder_2.
# The pipeline_name must match the _class_name in HF model_index.json.

from sglang.multimodal_gen.runtime.pipelines_core import LoRAPipeline
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 28-53: Class `SanaPipeline` / 类 `SanaPipeline`
```python
class SanaPipeline(LoRAPipeline, ComposedPipelineBase):
    pipeline_name = "SanaPipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
        "transformer",
        "scheduler",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
        self.add_stage(InputValidationStage())

# ...
        self.add_standard_timestep_preparation_stage()
        self.add_standard_latent_preparation_stage()
        self.add_standard_denoising_stage()
        self.add_standard_decoding_stage()
```
**EN:** This class models `SanaPipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `SanaPipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 54-56: Top-level configuration / 顶层配置
```python


EntryClass = SanaPipeline
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- LoRA adaptation support / LoRA 适配支持
- Symbol `SanaPipeline` anchors the module API / 符号 `SanaPipeline` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
