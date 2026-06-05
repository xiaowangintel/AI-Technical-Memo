# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for init, connecting stages, tensors, and runtime metadata. Key symbols include `PipelineWithLoRA`, `build_pipeline`. / 该模块负责 init 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `PipelineWithLoRA`, `build_pipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Diffusion pipelines for sglang.multimodal_gen.

This package contains diffusion pipelines for generating videos and images.
"""

from typing import cast

from sglang.multimodal_gen.registry import get_model_info
from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
# ...
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 28-31: Class `PipelineWithLoRA` / 类 `PipelineWithLoRA`
```python
class PipelineWithLoRA(LoRAPipeline, ComposedPipelineBase):
    """Type for a pipeline that has both ComposedPipelineBase and LoRAPipeline functionality."""

    pass
```
**EN:** This class models `PipelineWithLoRA` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Type for a pipeline that has both ComposedPipelineBase and LoRAPipeline functionality.
**CN:** 该类实现 `PipelineWithLoRA`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 文档字符串指出：Type for a pipeline that has both ComposedPipelineBase and LoRAPipeline functionality.

### Lines 34-83: Function `build_pipeline` / 函数 `build_pipeline`
```python
def build_pipeline(
    server_args: ServerArgs,
) -> PipelineWithLoRA:
    """
    Only works with valid hf diffusers configs. (model_index.json)
    We want to build a pipeline based on the inference args mode_path:
    1. download the model from the hub if it's not already downloaded
    2. verify the model config and directory
    3. based on the config, determine the pipeline class
    """
    model_path = server_args.model_path

    # Check if pipeline class is explicitly specified
    if server_args.pipeline_class_name:
# ...

    logger.info("Pipeline instantiated")

    return cast(PipelineWithLoRA, pipeline)
```
**EN:** This function drives `build_pipeline` with inputs such as `server_args`. Only works with valid hf diffusers configs.
**CN:** 这个函数负责 `build_pipeline`，主要处理 `server_args` 等输入。 文档字符串说明：Only works with valid hf diffusers configs.

### Lines 84-91: Registration and exports / 注册与导出
```python


__all__ = [
    "build_pipeline",
    "ComposedPipelineBase",
    "Req",
    "LoRAPipeline",
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- LoRA adaptation support / LoRA 适配支持
- Symbol `PipelineWithLoRA` anchors the module API / 符号 `PipelineWithLoRA` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.registry`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.lora_pipeline`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **Stdlib / 标准库**: `typing`
