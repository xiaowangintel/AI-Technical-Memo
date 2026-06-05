# upsampling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/upsampling.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for upsampling, connecting stages, tensors, and runtime metadata. Key symbols include `LTX2HalveResolutionStage`, `LTX2LoRASwitchStage`, `LTX2UpsampleStage`. / 该模块负责 upsampling 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LTX2HalveResolutionStage`, `LTX2LoRASwitchStage`, `LTX2UpsampleStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup / 导入与模块初始化
```python
import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.memory_managers.component_manager import (
    ComponentUse,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.base import PipelineStage
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 15-38: Class `LTX2HalveResolutionStage` / 类 `LTX2HalveResolutionStage`
```python
class LTX2HalveResolutionStage(PipelineStage):
    """Halve batch height/width for two-stage Stage 1 (low-res generation)."""

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        original_h, original_w = batch.height, batch.width

        vae_scale_factor = getattr(server_args.pipeline_config, "vae_scale_factor", 32)
        required_alignment = max(64, int(vae_scale_factor) * 2)
        if original_h % required_alignment != 0 or original_w % required_alignment != 0:
            raise ValueError(
                "LTX-2 two-stage requires resolution divisible by "
                f"{required_alignment}, got ({original_h}x{original_w})."
            )

# ...
            batch.height,
            batch.width,
        )
        return batch
```
**EN:** This class models `LTX2HalveResolutionStage` as a specialization of `PipelineStage`. Halve batch height/width for two-stage Stage 1 (low-res generation). Important methods include `forward`.
**CN:** 该类实现 `LTX2HalveResolutionStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Halve batch height/width for two-stage Stage 1 (low-res generation). 其中较重要的方法包括 `forward`。

### Lines 41-55: Class `LTX2LoRASwitchStage` / 类 `LTX2LoRASwitchStage`
```python
class LTX2LoRASwitchStage(PipelineStage):
    """Switch LoRA configuration for the requested two-stage phase."""

    def __init__(self, pipeline, phase: str):
        super().__init__()
        self.pipeline = pipeline
        self.phase = phase

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        if self.pipeline.should_skip_ltx2_lora_switch_stage():
            batch.extra["ltx2_phase"] = self.phase
            return batch
        self.pipeline.switch_lora_phase(self.phase, batch=batch)
        batch.extra["ltx2_phase"] = self.phase
        return batch
```
**EN:** This class models `LTX2LoRASwitchStage` as a specialization of `PipelineStage`. Switch LoRA configuration for the requested two-stage phase. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2LoRASwitchStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Switch LoRA configuration for the requested two-stage phase. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 58-150: Class `LTX2UpsampleStage` / 类 `LTX2UpsampleStage`
```python
class LTX2UpsampleStage(PipelineStage):
    """Upsample Stage-1 video latents and prepare Stage-2 inputs."""

    def __init__(
        self,
        spatial_upsampler,
        vae,
        audio_vae=None,
        pipeline=None,
    ):
        super().__init__()
        self.spatial_upsampler = spatial_upsampler
        self.vae = vae
        self.audio_vae = audio_vae
# ...
            batch.width,
        )
        self._repack_audio_latents(batch, server_args)
        return batch
```
**EN:** This class models `LTX2UpsampleStage` as a specialization of `PipelineStage`. Upsample Stage-1 video latents and prepare Stage-2 inputs. Important methods include `__init__`, `component_uses`, `_upsample_video_latents`, `_restore_full_resolution`.
**CN:** 该类实现 `LTX2UpsampleStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Upsample Stage-1 video latents and prepare Stage-2 inputs. 其中较重要的方法包括 `__init__`, `component_uses`, `_upsample_video_latents`, `_restore_full_resolution`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Sampling parameter control / 采样参数控制
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`
