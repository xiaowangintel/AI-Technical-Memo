# flux_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/flux_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for flux 2, connecting stages, tensors, and runtime metadata. Key symbols include `compute_empirical_mu`, `Flux2Pipeline`. / 该模块负责 flux 2 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `compute_empirical_mu`, `Flux2Pipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo
# SPDX-License-Identifier: Apache-2.0

from diffusers.pipelines.flux2.image_processor import Flux2ImageProcessor

from sglang.multimodal_gen.runtime.pipelines_core import LoRAPipeline, Req
from sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base import (
    ComposedPipelineBase,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 16-33: Function `compute_empirical_mu` / 函数 `compute_empirical_mu`
```python
def compute_empirical_mu(batch: Req, server_args: ServerArgs):
    num_steps = batch.num_inference_steps
    image_seq_len = batch.raw_latent_shape[1]
    a1, b1 = 8.73809524e-05, 1.89833333
    a2, b2 = 0.00016927, 0.45666666

    if image_seq_len > 4300:
        mu = a2 * image_seq_len + b2
        return "mu", float(mu)

    m_200 = a2 * image_seq_len + b2
    m_10 = a1 * image_seq_len + b1

    a = (m_200 - m_10) / 190.0
    b = m_200 - 200.0 * a
    mu = a * num_steps + b

    return "mu", float(mu)
```
**EN:** This function drives `compute_empirical_mu` with inputs such as `batch`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `compute_empirical_mu`，主要处理 `batch`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 36-59: Class `Flux2Pipeline` / 类 `Flux2Pipeline`
```python
class Flux2Pipeline(LoRAPipeline, ComposedPipelineBase):
    pipeline_name = "Flux2Pipeline"

    _required_config_modules = [
        "text_encoder",
        "tokenizer",
        "vae",
        "transformer",
        "scheduler",
    ]

    def create_pipeline_stages(self, server_args: ServerArgs):
        vae_image_processor = Flux2ImageProcessor(
            vae_scale_factor=server_args.pipeline_config.vae_config.arch_config.vae_scale_factor
# ...
            prompt_encoding="text",
            image_vae_stage_kwargs={"vae_image_processor": vae_image_processor},
            prepare_extra_timestep_kwargs=[compute_empirical_mu],
        )
```
**EN:** This class models `Flux2Pipeline` as a specialization of `LoRAPipeline`, `ComposedPipelineBase`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `Flux2Pipeline`，并继承/扩展 `LoRAPipeline`, `ComposedPipelineBase`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 60-62: Top-level configuration / 顶层配置
```python


EntryClass = Flux2Pipeline
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
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `diffusers.pipelines.flux2.image_processor`
