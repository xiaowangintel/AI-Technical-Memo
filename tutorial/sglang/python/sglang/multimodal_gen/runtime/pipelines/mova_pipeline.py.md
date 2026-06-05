# mova_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/mova_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for mova pipeline, connecting stages, tensors, and runtime metadata. Key symbols include `MOVAPipeline`, `MOVAPipelineAlias`. / 该模块负责 mova pipeline 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `MOVAPipeline`, `MOVAPipelineAlias`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
MOVA pipeline integration (native SGLang pipeline).
"""

from __future__ import annotations

from sglang.multimodal_gen.configs.pipeline_configs.mova import MOVAPipelineConfig
from sglang.multimodal_gen.configs.sample.mova import MOVASamplingParams
from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.pipelines_core.stages import (
    ImageVAEEncodingStage,
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 29-98: Class `MOVAPipeline` / 类 `MOVAPipeline`
```python
class MOVAPipeline(ComposedPipelineBase):
    """MOVA pipeline with SGLang stage orchestration."""

    pipeline_name = "MOVA"
    is_video_pipeline = True
    _required_config_modules = [
        "video_vae",
        "audio_vae",
        "text_encoder",
        "tokenizer",
        "scheduler",
        "video_dit",
        "video_dit_2",
        "audio_dit",
# ...
                audio_vae=self.get_module("audio_vae"),
            ),
            "mova_decoding_stage",
        )
```
**EN:** This class models `MOVAPipeline` as a specialization of `ComposedPipelineBase`. MOVA pipeline with SGLang stage orchestration. Important methods include `initialize_pipeline`, `create_pipeline_stages`.
**CN:** 该类实现 `MOVAPipeline`，并继承/扩展 `ComposedPipelineBase`。 文档字符串指出：MOVA pipeline with SGLang stage orchestration. 其中较重要的方法包括 `initialize_pipeline`, `create_pipeline_stages`。

### Lines 101-102: Class `MOVAPipelineAlias` / 类 `MOVAPipelineAlias`
```python
class MOVAPipelineAlias(MOVAPipeline):
    pipeline_name = "MOVAPipeline"
```
**EN:** This class models `MOVAPipelineAlias` as a specialization of `MOVAPipeline`.
**CN:** 该类实现 `MOVAPipelineAlias`，并继承/扩展 `MOVAPipeline`。

### Lines 103-105: Top-level configuration / 顶层配置
```python


EntryClass = [MOVAPipeline, MOVAPipelineAlias]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Distributed execution / 分布式执行
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.mova`, `sglang.multimodal_gen.configs.sample.mova`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.mova`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `__future__`
