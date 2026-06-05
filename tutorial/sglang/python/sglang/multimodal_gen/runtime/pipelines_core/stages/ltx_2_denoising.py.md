# ltx_2_denoising.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/ltx_2_denoising.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for ltx 2 denoising, connecting stages, tensors, and runtime metadata. Key symbols include `LTX2DenoisingContext`, `LTX2ModelInputs`, `LTX2GuidancePassSpec`. / 该模块负责 ltx 2 denoising 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LTX2DenoisingContext`, `LTX2ModelInputs`, `LTX2GuidancePassSpec`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-48: Imports and module setup / 导入与模块初始化
```python
import math
from contextlib import contextmanager
from dataclasses import dataclass, field

import torch
from diffusers.utils.torch_utils import randn_tensor

from sglang.multimodal_gen.configs.pipeline_configs.ltx_2 import (
    is_ltx23_native_variant,
)
from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
    get_sp_world_size,
)
# ...
)

LTX23_RES2S_STEP_NOISE_SEED = -1
LTX23_RES2S_SUBSTEP_NOISE_SEED = 9999
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 51-72: Class `LTX2DenoisingContext` / 类 `LTX2DenoisingContext`
```python
@dataclass(slots=True)
class LTX2DenoisingContext(DenoisingContext):
    """Loop-scoped denoising state for joint LTX-2 video and audio generation."""

    audio_latents: torch.Tensor | None = None
    audio_scheduler: object | None = None
    is_ltx23_variant: bool = False
    use_ltx23_legacy_one_stage: bool = False
    replicate_audio_for_sp: bool = False
    stage: str = "one_stage"
    latent_num_frames_for_model: int = 0
    latent_height: int = 0
    latent_width: int = 0
    denoise_mask: torch.Tensor | None = None
# ...
    use_native_hq_res2s_sde_noise: bool = False
    use_ltx23_hq_timestep_semantics: bool = False
    res2s_step_noise_generator: torch.Generator | None = None
    res2s_substep_noise_generator: torch.Generator | None = None
```
**EN:** This class models `LTX2DenoisingContext` as a specialization of `DenoisingContext`. Loop-scoped denoising state for joint LTX-2 video and audio generation.
**CN:** 该类实现 `LTX2DenoisingContext`，并继承/扩展 `DenoisingContext`。 文档字符串指出：Loop-scoped denoising state for joint LTX-2 video and audio generation.

### Lines 75-89: Class `LTX2ModelInputs` / 类 `LTX2ModelInputs`
```python
@dataclass(slots=True)
class LTX2ModelInputs:
    latent_model_input: torch.Tensor
    audio_latent_model_input: torch.Tensor
    audio_num_frames_latent: int
    video_coords: torch.Tensor | None
    audio_coords: torch.Tensor | None
    timestep_video: torch.Tensor
    timestep_audio: torch.Tensor
    prompt_timestep_video: torch.Tensor | None
    prompt_timestep_audio: torch.Tensor | None
    video_self_attention_mask: torch.Tensor | None
    audio_self_attention_mask: torch.Tensor | None
    a2v_cross_attention_mask: torch.Tensor | None
    v2a_cross_attention_mask: torch.Tensor | None
```
**EN:** This class models `LTX2ModelInputs`.
**CN:** 该类实现 `LTX2ModelInputs`。

### Lines 92-101: Class `LTX2GuidancePassSpec` / 类 `LTX2GuidancePassSpec`
```python
@dataclass(slots=True)
class LTX2GuidancePassSpec:
    name: str
    encoder_hidden_states: torch.Tensor
    audio_encoder_hidden_states: torch.Tensor
    encoder_attention_mask: torch.Tensor | None
    skip_video_self_attn_blocks: tuple[int, ...] = ()
    skip_audio_self_attn_blocks: tuple[int, ...] = ()
    disable_a2v_cross_attn: bool = False
    disable_v2a_cross_attn: bool = False
```
**EN:** This class models `LTX2GuidancePassSpec`.
**CN:** 该类实现 `LTX2GuidancePassSpec`。

### Lines 104-2622: Class `LTX2DenoisingStage` / 类 `LTX2DenoisingStage`
```python
class LTX2DenoisingStage(DenoisingStage):
    """
    LTX-2 specific denoising stage that handles joint video and audio generation.
    """

    _LTX2_BATCH_REPEATABLE_KWARG_KEYS = (
        "hidden_states",
        "audio_hidden_states",
        "timestep",
        "audio_timestep",
        "prompt_timestep",
        "audio_prompt_timestep",
        "video_coords",
        "audio_coords",
# ...
            lambda x: (not batch.do_classifier_free_guidance)
            or V.is_tensor(x)
            or V.list_not_empty(x)
        )
```
**EN:** This class models `LTX2DenoisingStage` as a specialization of `DenoisingStage`. LTX-2 specific denoising stage that handles joint video and audio generation. Important methods include `__init__`, `_randn_like_with_batch_generators`, `parallelism_type`, `_combine_cfg_parallel_av`.
**CN:** 该类实现 `LTX2DenoisingStage`，并继承/扩展 `DenoisingStage`。 文档字符串指出：LTX-2 specific denoising stage that handles joint video and audio generation. 其中较重要的方法包括 `__init__`, `_randn_like_with_batch_generators`, `parallelism_type`, `_combine_cfg_parallel_av`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.ltx_2`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.cfg_parallel_utils`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`
- **External / 外部**: `torch`, `diffusers.utils.torch_utils`
- **Stdlib / 标准库**: `math`, `contextlib`, `dataclasses`
