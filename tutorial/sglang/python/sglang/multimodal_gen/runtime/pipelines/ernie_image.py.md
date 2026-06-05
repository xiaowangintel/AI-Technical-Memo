# ernie_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/ernie_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for ernie image, connecting stages, tensors, and runtime metadata. Key symbols include `ErnieImagePipeline`. / 该模块负责 ernie image 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `ErnieImagePipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""ErnieImage text-to-image pipeline."""

import json
import os

from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.pipelines_core.lora_pipeline import LoRAPipeline
from sglang.multimodal_gen.runtime.pipelines_core.stages.input_validation import (
    InputValidationStage,
)
from sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.ernie_image_pe import (
# ...
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 29-229: Class `ErnieImagePipeline` / 类 `ErnieImagePipeline`
```python
class ErnieImagePipeline(LoRAPipeline, ComposedPipelineBase):

    pipeline_name = "ErnieImagePipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
        "transformer",
        "scheduler",
    ]

    def _has_pe_in_model_index(self, server_args) -> bool:
        try:
# ...
        self.add_standard_timestep_preparation_stage()
        self.add_standard_latent_preparation_stage()
        self.add_standard_denoising_stage()
        self.add_standard_decoding_stage()
```
**EN:** This class models `ErnieImagePipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `_has_pe_in_model_index`, `_read_tokenizer_model_max_length`, `_resolve_pe_tokenizer_path`, `_read_pe_model_max_length`.
**CN:** 该类实现 `ErnieImagePipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `_has_pe_in_model_index`, `_read_tokenizer_model_max_length`, `_resolve_pe_tokenizer_path`, `_read_pe_model_max_length`。

### Lines 230-232: Top-level configuration / 顶层配置
```python


EntryClass = ErnieImagePipeline
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略
- Image generation flow / 图像生成流程
- LoRA adaptation support / LoRA 适配支持

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.lora_pipeline`, `sglang.multimodal_gen.runtime.pipelines_core.stages.input_validation`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.ernie_image_pe`, `sglang.multimodal_gen.runtime.pipelines_core.stages.text_encoding`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `huggingface_hub`, `transformers`
- **Stdlib / 标准库**: `json`, `os`, `tempfile`
