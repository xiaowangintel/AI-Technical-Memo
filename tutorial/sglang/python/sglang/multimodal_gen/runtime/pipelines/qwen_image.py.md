# qwen_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/qwen_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for qwen image, connecting stages, tensors, and runtime metadata. Key symbols include `calculate_shift`, `prepare_mu`, `QwenImagePipeline`. / 该模块负责 qwen image 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `calculate_shift`, `prepare_mu`, `QwenImagePipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
from diffusers.image_processor import VaeImageProcessor

from sglang.multimodal_gen.runtime.pipelines_core import LoRAPipeline
from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.qwen_image_layered import (
    QwenImageLayeredBeforeDenoisingStage,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

# TODO(will): move PRECISION_TO_TYPE to better place

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 22-32: Function `calculate_shift` / 函数 `calculate_shift`
```python
def calculate_shift(
    image_seq_len,
    base_seq_len: int = 256,
    max_seq_len: int = 4096,
    base_shift: float = 0.5,
    max_shift: float = 1.15,
):
    m = (max_shift - base_shift) / (max_seq_len - base_seq_len)
    b = base_shift - m * base_seq_len
    mu = image_seq_len * m + b
    return mu
```
**EN:** This function drives `calculate_shift` with inputs such as `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_shift`，主要处理 `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 35-50: Function `prepare_mu` / 函数 `prepare_mu`
```python
def prepare_mu(batch: Req, server_args: ServerArgs):
    height = batch.height
    width = batch.width
    vae_scale_factor = server_args.pipeline_config.vae_config.vae_scale_factor
    image_seq_len = (int(height) // vae_scale_factor // 2) * (
        int(width) // vae_scale_factor // 2
    )
    mu = calculate_shift(
        image_seq_len,
        # hard code, since scheduler_config is not in PipelineConfig now
        256,
        8192,
        0.5,
        0.9,
    )
    return "mu", mu
```
**EN:** This function drives `prepare_mu` with inputs such as `batch`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `prepare_mu`，主要处理 `batch`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 53-65: Class `QwenImagePipeline` / 类 `QwenImagePipeline`
```python
class QwenImagePipeline(LoRAPipeline, ComposedPipelineBase):
    pipeline_name = "QwenImagePipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
        "transformer",
        "scheduler",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
        self.add_standard_t2i_stages(prepare_extra_timestep_kwargs=[prepare_mu])
```
**EN:** This class models `QwenImagePipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `QwenImagePipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 68-92: Class `QwenImageEditPipeline` / 类 `QwenImageEditPipeline`
```python
class QwenImageEditPipeline(LoRAPipeline, ComposedPipelineBase):
    pipeline_name = "QwenImageEditPipeline"

    _required_config_modules = [
        "processor",
        "scheduler",
        "text_encoder",
        "tokenizer",
        "transformer",
        "vae",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
        vae_image_processor = VaeImageProcessor(
# ...
            image_processor_key="processor",
            prompt_text_encoder_key="text_encoder",
            prepare_extra_timestep_kwargs=[prepare_mu],
        )
```
**EN:** This class models `QwenImageEditPipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `QwenImageEditPipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 95-96: Class `QwenImageEditPlusPipeline` / 类 `QwenImageEditPlusPipeline`
```python
class QwenImageEditPlusPipeline(QwenImageEditPipeline):
    pipeline_name = "QwenImageEditPlusPipeline"
```
**EN:** This class models `QwenImageEditPlusPipeline` as a specialization of `QwenImageEditPipeline`.
**CN:** 该类实现 `QwenImageEditPlusPipeline`，并继承/扩展 `QwenImageEditPipeline`。

### Lines 99-102: Function `prepare_mu_layered` / 函数 `prepare_mu_layered`
```python
def prepare_mu_layered(batch: Req, server_args: ServerArgs):
    base_seqlen = 256 * 256 / 16 / 16
    mu = (batch.image_latent.shape[1] / base_seqlen) ** 0.5
    return "mu", mu
```
**EN:** This function drives `prepare_mu_layered` with inputs such as `batch`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `prepare_mu_layered`，主要处理 `batch`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 105-132: Class `QwenImageLayeredPipeline` / 类 `QwenImageLayeredPipeline`
```python
class QwenImageLayeredPipeline(QwenImageEditPipeline):
    pipeline_name = "QwenImageLayeredPipeline"

    _required_config_modules = [
        "vae",
        "tokenizer",
        "processor",
        "transformer",
        "scheduler",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
        self.add_stage(
            QwenImageLayeredBeforeDenoisingStage(
# ...
            prepare_extra_kwargs=[prepare_mu_layered]
        )
        self.add_standard_denoising_stage()
        self.add_standard_decoding_stage()
```
**EN:** This class models `QwenImageLayeredPipeline` as a specialization of `QwenImageEditPipeline`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `QwenImageLayeredPipeline`，并继承/扩展 `QwenImageEditPipeline`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 133-140: Top-level configuration / 顶层配置
```python


EntryClass = [
    QwenImagePipeline,
    QwenImageEditPipeline,
    QwenImageEditPlusPipeline,
    QwenImageLayeredPipeline,
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- LoRA adaptation support / LoRA 适配支持

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.model_specific_stages.qwen_image_layered`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `diffusers.image_processor`
