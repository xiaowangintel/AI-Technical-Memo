# mova.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/mova.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for mova, connecting stages, tensors, and runtime metadata. Key symbols include `MOVALatentPreparationStage`, `MOVATimestepPreparationStage`, `MOVADenoisingStage`. / 该模块负责 mova 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `MOVALatentPreparationStage`, `MOVATimestepPreparationStage`, `MOVADenoisingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-75: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
MOVA-specific pipeline stages.

Sequence Parallelism (SP) Support:
- Video latents are sharded along the sequence dimension (T*H*W) after patchify
- Audio latents are sharded along the sequence dimension (L) after patchify
- USPAttention handles all-to-all communication internally
- Latents are gathered before unpatchify to restore full sequence
"""

from __future__ import annotations

import functools
# ...
from sglang.srt.utils.common import get_compiler_backend

_is_npu = current_platform.is_npu()
logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 78-121: Class `MOVALatentPreparationStage` / 类 `MOVALatentPreparationStage`
```python
class MOVALatentPreparationStage(PipelineStage):
    """Prepare video/audio noise latents for MOVA."""

    def __init__(self, audio_vae, require_vae_embedding: bool = True) -> None:
        super().__init__()
        self.audio_vae = audio_vae
        self.require_vae_embedding = require_vae_embedding

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        batch_size = batch.batch_size
        num_frames = batch.num_frames
        if num_frames is None:
            raise ValueError("num_frames is required for MOVA")

# ...
            batch.y = batch.image_latent.to(device=device, dtype=dit_dtype)
        elif self.require_vae_embedding:
            raise ValueError("MOVA requires reference image latents for denoising")
        return batch
```
**EN:** This class models `MOVALatentPreparationStage` as a specialization of `PipelineStage`. Prepare video/audio noise latents for MOVA. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MOVALatentPreparationStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Prepare video/audio noise latents for MOVA. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 124-147: Class `MOVATimestepPreparationStage` / 类 `MOVATimestepPreparationStage`
```python
class MOVATimestepPreparationStage(PipelineStage):
    """Prepare paired timesteps for MOVA."""

    def __init__(self, scheduler) -> None:
        super().__init__()
        self.scheduler = scheduler

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        scheduler = self.scheduler
        scheduler.set_timesteps(
            batch.num_inference_steps,
            denoising_strength=1.0,
            shift=getattr(batch, "sigma_shift", scheduler.shift),
        )
# ...
        batch.paired_timesteps = paired
        batch.timesteps = paired
        batch.scheduler = scheduler
        return batch
```
**EN:** This class models `MOVATimestepPreparationStage` as a specialization of `PipelineStage`. Prepare paired timesteps for MOVA. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MOVATimestepPreparationStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Prepare paired timesteps for MOVA. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 150-936: Class `MOVADenoisingStage` / 类 `MOVADenoisingStage`
```python
class MOVADenoisingStage(PipelineStage):
    """Run MOVA dual-tower denoising loop."""

    def __init__(self, video_dit, video_dit_2, audio_dit, dual_tower_bridge, scheduler):
        super().__init__()
        self.video_dit = video_dit
        self.video_dit_2 = video_dit_2
        self.audio_dit = audio_dit
        self.dual_tower_bridge = dual_tower_bridge
        self.scheduler = scheduler
        self._cache_dit_enabled = False
        self._cached_num_steps = None
        self._torch_compiled = False

# ...
                visual_x, visual_context, visual_t_mod, visual_freqs
            )

        return visual_x, audio_x
```
**EN:** This class models `MOVADenoisingStage` as a specialization of `PipelineStage`. Run MOVA dual-tower denoising loop. Important methods include `__init__`, `component_uses`, `parallelism_type`, `_predict`.
**CN:** 该类实现 `MOVADenoisingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Run MOVA dual-tower denoising loop. 其中较重要的方法包括 `__init__`, `component_uses`, `parallelism_type`, `_predict`。

### Lines 939-1010: Class `MOVADecodingStage` / 类 `MOVADecodingStage`
```python
class MOVADecodingStage(PipelineStage):
    """Decode video and audio outputs for MOVA."""

    def __init__(self, video_vae, audio_vae) -> None:
        super().__init__()
        self.video_vae = video_vae
        self.audio_vae = audio_vae

    def component_uses(
        self, server_args: ServerArgs, stage_name: str | None = None
    ) -> list[ComponentUse]:
        stage_name = self._component_stage_name(stage_name)
        vae_dtype = PRECISION_TO_TYPE[server_args.pipeline_config.vae_precision]
        return [
# ...
            audio_sample_rate=getattr(self.audio_vae, "sample_rate", None),
            metrics=batch.metrics,
        )
        return output_batch
```
**EN:** This class models `MOVADecodingStage` as a specialization of `PipelineStage`. Decode video and audio outputs for MOVA. Important methods include `__init__`, `component_uses`, `parallelism_type`, `forward`.
**CN:** 该类实现 `MOVADecodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Decode video and audio outputs for MOVA. 其中较重要的方法包括 `__init__`, `component_uses`, `parallelism_type`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.models.dits.mova_video_dit`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`
- **External / 外部**: `__future__`, `torch`, `torch.nn`, `diffusers.utils.torch_utils`, `tqdm.auto`, `torch._inductor.config`
- **Stdlib / 标准库**: `functools`, `inspect`, `os`, `collections.abc`
