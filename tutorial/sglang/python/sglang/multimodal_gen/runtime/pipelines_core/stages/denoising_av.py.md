# denoising_av.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/denoising_av.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for denoising av, connecting stages, tensors, and runtime metadata. Key symbols include `LTX2AVDenoisingStage`, `LTX2RefinementStage`. / 该模块负责 denoising av 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LTX2AVDenoisingStage`, `LTX2RefinementStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup / 导入与模块初始化
```python
import torch
from diffusers.utils.torch_utils import randn_tensor

from sglang.multimodal_gen.configs.pipeline_configs.ltx_2 import is_ltx23_native_variant
from sglang.multimodal_gen.runtime.managers.memory_managers.component_manager import (
    ComponentUse,
)
from sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils import (
    clone_scheduler_runtime,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.base import (
    StageParallelismType,
)
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 24-103: Class `LTX2AVDenoisingStage` / 类 `LTX2AVDenoisingStage`
```python
class LTX2AVDenoisingStage(LTX2DenoisingStage):
    """
    Thin AV layer that adds audio trajectory gathering and final unpacking on top of
    the LTX-2 denoising semantics.
    """

    def __init__(self, transformer, scheduler, vae=None, audio_vae=None, **kwargs):
        super().__init__(
            transformer=transformer, scheduler=scheduler, vae=vae, **kwargs
        )
        self.audio_vae = audio_vae

    def component_uses(
        self, server_args: ServerArgs, stage_name: str | None = None
# ...
                )
            )
            batch.latents = latents
            batch.audio_latents = audio_latents
```
**EN:** This class models `LTX2AVDenoisingStage` as a specialization of `LTX2DenoisingStage`. Thin AV layer that adds audio trajectory gathering and final unpacking on top of Important methods include `__init__`, `component_uses`, `_post_denoising_loop`.
**CN:** 该类实现 `LTX2AVDenoisingStage`，并继承/扩展 `LTX2DenoisingStage`。 文档字符串指出：Thin AV layer that adds audio trajectory gathering and final unpacking on top of 其中较重要的方法包括 `__init__`, `component_uses`, `_post_denoising_loop`。

### Lines 106-398: Class `LTX2RefinementStage` / 类 `LTX2RefinementStage`
```python
class LTX2RefinementStage(LTX2AVDenoisingStage):
    """Stage-2 refinement wrapper that re-noises distilled LTX latents once."""

    def __init__(
        self,
        transformer,
        scheduler,
        distilled_sigmas,
        vae=None,
        audio_vae=None,
        pipeline=None,
        sampler_name: str = "euler",
    ):
        super().__init__(
# ...
            batch.do_classifier_free_guidance = original_do_cfg
            batch.ltx2_ti2v_clean_latent_background = original_clean_latent_background

        return batch
```
**EN:** This class models `LTX2RefinementStage` as a specialization of `LTX2AVDenoisingStage`. Stage-2 refinement wrapper that re-noises distilled LTX latents once. Important methods include `__init__`, `component_uses`, `parallelism_type`, `_randn_like_with_batch_generators`.
**CN:** 该类实现 `LTX2RefinementStage`，并继承/扩展 `LTX2AVDenoisingStage`。 文档字符串指出：Stage-2 refinement wrapper that re-noises distilled LTX latents once. 其中较重要的方法包括 `__init__`, `component_uses`, `parallelism_type`, `_randn_like_with_batch_generators`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.ltx_2`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.ltx_2_denoising`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `diffusers.utils.torch_utils`
