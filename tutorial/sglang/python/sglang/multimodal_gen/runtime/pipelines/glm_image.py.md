# glm_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/glm_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for glm image, connecting stages, tensors, and runtime metadata. Key symbols include `GlmImagePipeline`. / 该模块负责 glm image 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `GlmImagePipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup / 导入与模块初始化
```python
from sglang.multimodal_gen.runtime.pipelines_core import LoRAPipeline
from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.pipelines_core.stages import DenoisingStage
from sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.glm_image import (
    GlmImageBeforeDenoisingStage,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 15-49: Class `GlmImagePipeline` / 类 `GlmImagePipeline`
```python
class GlmImagePipeline(LoRAPipeline, ComposedPipelineBase):
    pipeline_name = "GlmImagePipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
        "vision_language_encoder",
        "processor",
        "transformer",
        "scheduler",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
# ...
            ),
        )

        self.add_standard_decoding_stage()
```
**EN:** This class models `GlmImagePipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `GlmImagePipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 50-52: Top-level configuration / 顶层配置
```python


EntryClass = [GlmImagePipeline]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- LoRA adaptation support / LoRA 适配支持
- Symbol `GlmImagePipeline` anchors the module API / 符号 `GlmImagePipeline` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.glm_image`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
