# comfyui_qwen_image_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/comfyui_qwen_image_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for comfyui qwen image pipeline, connecting stages, tensors, and runtime metadata. Key symbols include `ComfyUIQwenImagePipelineBase`, `ComfyUIQwenImagePipeline`, `ComfyUIQwenImageEditPipeline`. / 该模块负责 comfyui qwen image pipeline 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `ComfyUIQwenImagePipelineBase`, `ComfyUIQwenImagePipeline`, `ComfyUIQwenImageEditPipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0

import os
from itertools import chain
from typing import Any

import torch
from torch.distributed import init_device_mesh
from torch.distributed.fsdp import MixedPrecisionPolicy

from sglang.multimodal_gen.configs.models.dits.qwenimage import QwenImageDitConfig
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.fsdp_load import (
    load_model_from_full_model_state_dict,
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE, set_mixed_precision_policy

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 43-313: Class `ComfyUIQwenImagePipelineBase` / 类 `ComfyUIQwenImagePipelineBase`
```python
class ComfyUIQwenImagePipelineBase(LoRAPipeline, ComposedPipelineBase):
    """
    Base pipeline for ComfyUI QwenImage integration with only denoising stage.

    This pipeline requires pre-processed inputs:
    - prompt_embeds: Pre-encoded text embeddings (list of tensors)
    - latents: Pre-processed image latents in sequence format [B, S, D]

    Usage:
        generator = DiffGenerator.from_pretrained(
            model_path="path/to/model",
            pipeline_class_name="ComfyUIQwenImagePipeline",
            device="cuda",
        )
# ...

        logger.info(
            f"{self.__class__.__name__} stages created: {list(self._stage_name_mapping.keys())}"
        )
```
**EN:** This class models `ComfyUIQwenImagePipelineBase` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Base pipeline for ComfyUI QwenImage integration with only denoising stage. Important methods include `initialize_pipeline`, `load_modules`, `_load_transformer_from_safetensors`, `_prepare_dit_config_and_mapping`.
**CN:** 该类实现 `ComfyUIQwenImagePipelineBase`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 文档字符串指出：Base pipeline for ComfyUI QwenImage integration with only denoising stage. 其中较重要的方法包括 `initialize_pipeline`, `load_modules`, `_load_transformer_from_safetensors`, `_prepare_dit_config_and_mapping`。

### Lines 316-328: Class `ComfyUIQwenImagePipeline` / 类 `ComfyUIQwenImagePipeline`
```python
class ComfyUIQwenImagePipeline(ComfyUIQwenImagePipelineBase):
    """ComfyUI QwenImage pipeline for text-to-image generation."""

    pipeline_name = "ComfyUIQwenImagePipeline"
    zero_cond_t = False

    from sglang.multimodal_gen.configs.pipeline_configs.qwen_image import (
        QwenImagePipelineConfig,
    )
    from sglang.multimodal_gen.configs.sample.qwenimage import QwenImageSamplingParams

    pipeline_config_cls = QwenImagePipelineConfig
    sampling_params_cls = QwenImageSamplingParams
```
**EN:** This class models `ComfyUIQwenImagePipeline` as a specialization of `ComfyUIQwenImagePipelineBase`. ComfyUI QwenImage pipeline for text-to-image generation.
**CN:** 该类实现 `ComfyUIQwenImagePipeline`，并继承/扩展 `ComfyUIQwenImagePipelineBase`。 文档字符串指出：ComfyUI QwenImage pipeline for text-to-image generation.

### Lines 331-345: Class `ComfyUIQwenImageEditPipeline` / 类 `ComfyUIQwenImageEditPipeline`
```python
class ComfyUIQwenImageEditPipeline(ComfyUIQwenImagePipelineBase):
    """ComfyUI QwenImage pipeline for image-to-image editing."""

    pipeline_name = "ComfyUIQwenImageEditPipeline"
    zero_cond_t = True

    from sglang.multimodal_gen.configs.pipeline_configs.qwen_image import (
        QwenImageEditPlusPipelineConfig,
    )
    from sglang.multimodal_gen.configs.sample.qwenimage import (
        QwenImageEditPlusSamplingParams,
    )

    pipeline_config_cls = QwenImageEditPlusPipelineConfig
    sampling_params_cls = QwenImageEditPlusSamplingParams
```
**EN:** This class models `ComfyUIQwenImageEditPipeline` as a specialization of `ComfyUIQwenImagePipelineBase`. ComfyUI QwenImage pipeline for image-to-image editing.
**CN:** 该类实现 `ComfyUIQwenImageEditPipeline`，并继承/扩展 `ComfyUIQwenImagePipelineBase`。 文档字符串指出：ComfyUI QwenImage pipeline for image-to-image editing.

### Lines 346-348: Top-level configuration / 顶层配置
```python


EntryClass = [ComfyUIQwenImagePipeline, ComfyUIQwenImageEditPipeline]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.qwenimage`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.fsdp_load`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.loader.weight_utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.models.schedulers.scheduling_comfyui_passthrough`, `sglang.multimodal_gen.runtime.pipelines_core`
- **External / 外部**: `torch`, `torch.distributed`, `torch.distributed.fsdp`
- **Stdlib / 标准库**: `os`, `itertools`, `typing`
