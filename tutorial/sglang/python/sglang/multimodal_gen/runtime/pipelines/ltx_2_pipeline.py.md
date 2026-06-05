# ltx_2_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines/ltx_2_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for ltx 2 pipeline, connecting stages, tensors, and runtime metadata. Key symbols include `_resolve_ltx2_two_stage_component_paths`, `calculate_ltx2_shift`, `prepare_ltx2_mu`. / 该模块负责 ltx 2 pipeline 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `_resolve_ltx2_two_stage_component_paths`, `calculate_ltx2_shift`, `prepare_ltx2_mu`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-58: Imports and module setup / 导入与模块初始化
```python
import math
import os

import numpy as np
import torch
from diffusers import FlowMatchEulerDiscreteScheduler

from sglang.multimodal_gen.configs.pipeline_configs.ltx_2 import (
    LTX2PipelineConfig,
    is_ltx23_native_variant,
    sync_ltx23_runtime_vae_markers,
)
from sglang.multimodal_gen.configs.sample.ltx_2 import LTX23HQSamplingParams
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
# ...
logger = init_logger(__name__)

BASE_SHIFT_ANCHOR = 1024
MAX_SHIFT_ANCHOR = 4096
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 61-97: Function `_resolve_ltx2_two_stage_component_paths` / 函数 `_resolve_ltx2_two_stage_component_paths`
```python
def _resolve_ltx2_two_stage_component_paths(
    model_path: str, component_paths: dict[str, str]
) -> dict[str, str]:
    resolved = dict(component_paths)
    auto_resolved = []

    if "spatial_upsampler" not in resolved:
        spatial_candidates = [
            os.path.join(model_path, "ltx-2.3-spatial-upscaler-x2-1.0.safetensors"),
            os.path.join(model_path, "ltx-2.3-spatial-upscaler-x2-1.1.safetensors"),
            os.path.join(model_path, "latent_upsampler"),
            os.path.join(model_path, "ltx-2-spatial-upscaler-x2-1.0.safetensors"),
        ]
        for candidate in spatial_candidates:
# ...
            "Auto-resolved LTX2 two-stage components: %s", ", ".join(auto_resolved)
        )

    return resolved
```
**EN:** This function drives `_resolve_ltx2_two_stage_component_paths` with inputs such as `model_path`, `component_paths`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_ltx2_two_stage_component_paths`，主要处理 `model_path`, `component_paths` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 100-109: Function `calculate_ltx2_shift` / 函数 `calculate_ltx2_shift`
```python
def calculate_ltx2_shift(
    image_seq_len: int,
    base_seq_len: int = BASE_SHIFT_ANCHOR,
    max_seq_len: int = MAX_SHIFT_ANCHOR,
    base_shift: float = 0.95,
    max_shift: float = 2.05,
) -> float:
    mm = (max_shift - base_shift) / (max_seq_len - base_seq_len)
    b = base_shift - mm * base_seq_len
    return image_seq_len * mm + b
```
**EN:** This function drives `calculate_ltx2_shift` with inputs such as `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_ltx2_shift`，主要处理 `image_seq_len`, `base_seq_len`, `max_seq_len`, `base_shift` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 112-123: Function `prepare_ltx2_mu` / 函数 `prepare_ltx2_mu`
```python
def prepare_ltx2_mu(batch: Req, server_args: ServerArgs):
    if is_ltx23_native_variant(server_args.pipeline_config.vae_config.arch_config):
        return "mu", None
    latent_num_frames = (int(batch.num_frames) - 1) // int(
        server_args.pipeline_config.vae_temporal_compression
    ) + 1
    latent_height = int(batch.height) // int(
        server_args.pipeline_config.vae_scale_factor
    )
    latent_width = int(batch.width) // int(server_args.pipeline_config.vae_scale_factor)
    video_sequence_length = latent_num_frames * latent_height * latent_width
    return "mu", calculate_ltx2_shift(video_sequence_length)
```
**EN:** This function drives `prepare_ltx2_mu` with inputs such as `batch`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `prepare_ltx2_mu`，主要处理 `batch`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 126-160: Function `build_official_ltx2_sigmas` / 函数 `build_official_ltx2_sigmas`
```python
def build_official_ltx2_sigmas(
    steps: int,
    *,
    max_shift: float = 2.05,
    base_shift: float = 0.95,
    stretch: bool = True,
    terminal: float = 0.1,
    default_number_of_tokens: int = MAX_SHIFT_ANCHOR,
    number_of_tokens: int | None = None,
) -> list[float]:
    sigmas = torch.linspace(1.0, 0.0, steps + 1, dtype=torch.float32)

    mm = (max_shift - base_shift) / (MAX_SHIFT_ANCHOR - BASE_SHIFT_ANCHOR)
    b = base_shift - mm * BASE_SHIFT_ANCHOR
# ...
            scale_factor = one_minus_z[-1] / (1.0 - terminal)
            shifted[non_zero_mask] = 1.0 - (one_minus_z / scale_factor)

    return shifted[:-1].tolist()
```
**EN:** This function drives `build_official_ltx2_sigmas` with inputs such as `steps`, `max_shift`, `base_shift`, `stretch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_official_ltx2_sigmas`，主要处理 `steps`, `max_shift`, `base_shift`, `stretch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 163-206: Class `LTX2SigmaPreparationStage` / 类 `LTX2SigmaPreparationStage`
```python
class LTX2SigmaPreparationStage(PipelineStage):
    """Prepare native LTX-2 sigma schedule before timestep setup."""

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        batch.extra["ltx2_phase"] = "stage1"
        if is_ltx23_native_variant(server_args.pipeline_config.vae_config.arch_config):
            # Gate on pipeline class to mirror the three official entry points:
            # - HQ (`ti2vid_two_stages_hq.py:164`) calls
            #   `LTX2Scheduler.execute(latent=empty_latent, ...)` where
            #   `empty_latent` is built from the **half-resolution** stage-1
            #   shape → resolution-aware sigma shift.
            # - Non-HQ two-stage (`ti2vid_two_stages.py:145`) and
            #   one-stage (`ti2vid_one_stage.py:138`) call
            #   `LTX2Scheduler.execute(steps=...)` with no `latent` →
# ...
                1.0 / int(batch.num_inference_steps),
                int(batch.num_inference_steps),
            ).tolist()
        return batch
```
**EN:** This class models `LTX2SigmaPreparationStage` as a specialization of `PipelineStage`. Prepare native LTX-2 sigma schedule before timestep setup. Important methods include `forward`.
**CN:** 该类实现 `LTX2SigmaPreparationStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Prepare native LTX-2 sigma schedule before timestep setup. 其中较重要的方法包括 `forward`。

### Lines 209-219: Function `_add_ltx2_front_stages` / 函数 `_add_ltx2_front_stages`
```python
def _add_ltx2_front_stages(pipeline: ComposedPipelineBase):
    pipeline.add_stages(
        [
            InputValidationStage(),
            TextEncodingStage(
                text_encoders=[pipeline.get_module("text_encoder")],
                tokenizers=[pipeline.get_module("tokenizer")],
            ),
            LTX2TextConnectorStage(connectors=pipeline.get_module("connectors")),
        ]
    )
```
**EN:** This function drives `_add_ltx2_front_stages` with inputs such as `pipeline`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_add_ltx2_front_stages`，主要处理 `pipeline` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 222-250: Function `_add_ltx2_stage1_generation_stages` / 函数 `_add_ltx2_stage1_generation_stages`
```python
def _add_ltx2_stage1_generation_stages(
    pipeline: ComposedPipelineBase,
    *,
    denoising_sampler_name: str = "euler",
):
    pipeline.add_stage(LTX2SigmaPreparationStage())
    pipeline.add_standard_timestep_preparation_stage(
        prepare_extra_kwargs=[prepare_ltx2_mu]
    )
    pipeline.add_stages(
        [
            LTX2AVLatentPreparationStage(
                scheduler=pipeline.get_module("scheduler"),
                transformer=pipeline.get_module("transformer"),
# ...
                pipeline=pipeline,
            ),
        ]
    )
```
**EN:** This function drives `_add_ltx2_stage1_generation_stages` with inputs such as `pipeline`, `denoising_sampler_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_add_ltx2_stage1_generation_stages`，主要处理 `pipeline`, `denoising_sampler_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 253-261: Function `_add_ltx2_decoding_stage` / 函数 `_add_ltx2_decoding_stage`
```python
def _add_ltx2_decoding_stage(pipeline: ComposedPipelineBase):
    pipeline.add_stage(
        LTX2AVDecodingStage(
            vae=pipeline.get_module("vae"),
            audio_vae=pipeline.get_module("audio_vae"),
            vocoder=pipeline.get_module("vocoder"),
            pipeline=pipeline,
        )
    )
```
**EN:** This function drives `_add_ltx2_decoding_stage` with inputs such as `pipeline`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_add_ltx2_decoding_stage`，主要处理 `pipeline` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 264-299: Class `LTX2FlowMatchScheduler` / 类 `LTX2FlowMatchScheduler`
```python
class LTX2FlowMatchScheduler(FlowMatchEulerDiscreteScheduler):
    """Override ``_time_shift_exponential`` to use torch f32 instead of numpy f64."""

    def set_timesteps(
        self,
        num_inference_steps=None,
        device=None,
        sigmas=None,
        mu=None,
        timesteps=None,
    ):
        if sigmas is not None and timesteps is None and mu is None:
            sigmas = torch.tensor(sigmas, dtype=torch.float32, device=device)
            timesteps = sigmas * self.config.num_train_timesteps
# ...
            t_torch = torch.from_numpy(t).to(torch.float32)
            result = math.exp(mu) / (math.exp(mu) + (1 / t_torch - 1) ** sigma)
            return result.numpy()
        return math.exp(mu) / (math.exp(mu) + (1 / t - 1) ** sigma)
```
**EN:** This class models `LTX2FlowMatchScheduler` as a specialization of `FlowMatchEulerDiscreteScheduler`. Override ``_time_shift_exponential`` to use torch f32 instead of numpy f64. Important methods include `set_timesteps`, `_time_shift_exponential`.
**CN:** 该类实现 `LTX2FlowMatchScheduler`，并继承/扩展 `FlowMatchEulerDiscreteScheduler`。 文档字符串指出：Override ``_time_shift_exponential`` to use torch f32 instead of numpy f64. 其中较重要的方法包括 `set_timesteps`, `_time_shift_exponential`。

### Lines 302-320: Class `_BaseLTX2Pipeline` / 类 `_BaseLTX2Pipeline`
```python
class _BaseLTX2Pipeline(LoRAPipeline):
    _required_config_modules = [
        "transformer",
        "text_encoder",
        "tokenizer",
        "scheduler",
        "vae",
        "audio_vae",
        "vocoder",
        "connectors",
    ]

    def initialize_pipeline(self, server_args: ServerArgs):
        orig = self.get_module("scheduler")
        self.modules["scheduler"] = LTX2FlowMatchScheduler.from_config(orig.config)
        sync_ltx23_runtime_vae_markers(
            server_args.pipeline_config.vae_config.arch_config,
            getattr(self.get_module("vae"), "config", None),
        )
```
**EN:** This class models `_BaseLTX2Pipeline` as a specialization of `LoRAPipeline`. Important methods include `initialize_pipeline`.
**CN:** 该类实现 `_BaseLTX2Pipeline`，并继承/扩展 `LoRAPipeline`。 其中较重要的方法包括 `initialize_pipeline`。

### Lines 323-330: Class `LTX2Pipeline` / 类 `LTX2Pipeline`
```python
class LTX2Pipeline(_BaseLTX2Pipeline):
    # Must match model_index.json `_class_name`.
    pipeline_name = "LTX2Pipeline"

    def create_pipeline_stages(self, server_args: ServerArgs):
        _add_ltx2_front_stages(self)
        _add_ltx2_stage1_generation_stages(self)
        _add_ltx2_decoding_stage(self)
```
**EN:** This class models `LTX2Pipeline` as a specialization of `_BaseLTX2Pipeline`. Important methods include `create_pipeline_stages`.
**CN:** 该类实现 `LTX2Pipeline`，并继承/扩展 `_BaseLTX2Pipeline`。 其中较重要的方法包括 `create_pipeline_stages`。

### Lines 333-411: Class `LTX2TwoStageResidencyStrategy` / 类 `LTX2TwoStageResidencyStrategy`
```python
class LTX2TwoStageResidencyStrategy(ComponentResidencyStrategy):
    name = "ltx2_original"

    def __init__(self, manager: "LTX2TwoStageResidencyController") -> None:
        self.manager = manager

    @property
    def pipeline(self) -> "LTX2TwoStagePipeline":
        return self.manager.pipeline

    @property
    def server_args(self) -> ServerArgs:
        return self.manager.server_args

# ...

    @staticmethod
    def _module_is_on_gpu(module: torch.nn.Module | None) -> bool:
        return SnapshotModuleResidency.is_on_gpu(module)
```
**EN:** This class models `LTX2TwoStageResidencyStrategy` as a specialization of `ComponentResidencyStrategy`. Important methods include `__init__`, `pipeline`, `server_args`, `_phase`.
**CN:** 该类实现 `LTX2TwoStageResidencyStrategy`，并继承/扩展 `ComponentResidencyStrategy`。 其中较重要的方法包括 `__init__`, `pipeline`, `server_args`, `_phase`。

### Lines 414-415: Class `LTX2OriginalResidencyStrategy` / 类 `LTX2OriginalResidencyStrategy`
```python
class LTX2OriginalResidencyStrategy(LTX2TwoStageResidencyStrategy):
    pass
```
**EN:** This class models `LTX2OriginalResidencyStrategy` as a specialization of `LTX2TwoStageResidencyStrategy`.
**CN:** 该类实现 `LTX2OriginalResidencyStrategy`，并继承/扩展 `LTX2TwoStageResidencyStrategy`。

### Lines 418-435: Class `LTX2ResidentResidencyStrategy` / 类 `LTX2ResidentResidencyStrategy`
```python
class LTX2ResidentResidencyStrategy(LTX2TwoStageResidencyStrategy):
    """A residency strategy for ltx two-stage pipeline with pre-merged lora, that keep both dits always resident"""

    name = "ltx2_resident"

    def initialize(self) -> None:
        self._ensure_on_gpu("transformer")
        self._ensure_on_gpu("transformer_2")
        logger.info(
            "Using resident LTX-2.3 two-stage transformers mode (both DiTs stay on GPU)"
        )
        self.manager._active_phase = "stage1"
        self.manager._sync_refinement_stage_transformer("stage1")

    def enter_phase(self, phase: str) -> bool:
        self.manager._sync_refinement_stage_transformer(phase)
        self.manager._active_phase = phase
        return True
```
**EN:** This class models `LTX2ResidentResidencyStrategy` as a specialization of `LTX2TwoStageResidencyStrategy`. A residency strategy for ltx two-stage pipeline with pre-merged lora, that keep both dits always resident Important methods include `initialize`, `enter_phase`.
**CN:** 该类实现 `LTX2ResidentResidencyStrategy`，并继承/扩展 `LTX2TwoStageResidencyStrategy`。 文档字符串指出：A residency strategy for ltx two-stage pipeline with pre-merged lora, that keep both dits always resident 其中较重要的方法包括 `initialize`, `enter_phase`。

### Lines 438-694: Class `LTX2SnapshotResidencyStrategy` / 类 `LTX2SnapshotResidencyStrategy`
```python
class LTX2SnapshotResidencyStrategy(LTX2TwoStageResidencyStrategy):
    """
    Snapshot mode keeps CPU snapshots and prefetches the target DiT with async H2D. (only with pre-merged lora enabled)

    The DiT_1 will always be kept a replica in CPU.
    - default snapshot behavior: allow stage1/stage2 overlap by prefetching
      stage2 while stage1 is still running.
    - snapshot low-VRAM behavior (`_snapshot_low_vram_mode=True`): evict
      stage1 before stage2 prefetch and disable early overlap prefetch to
      reduce peak VRAM, at the cost of higher phase-switch latency.
    - default toggle: low-VRAM auto-enables on H100-like (<130 GiB) CUDA
      GPUs, and stays disabled by default on higher-memory GPUs. It can be
      overridden with `SGLANG_LTX2_SNAPSHOT_LOW_VRAM_MODE`.
    """
# ...
            logger.info(
                "Pinned stage1 transformer on GPU for LTX-2.3 two-stage startup"
            )
        self.manager._active_phase = "stage1"
```
**EN:** This class models `LTX2SnapshotResidencyStrategy` as a specialization of `LTX2TwoStageResidencyStrategy`. Snapshot mode keeps CPU snapshots and prefetches the target DiT with async H2D. Important methods include `__init__`, `_module_name_for_phase`, `_resolve_snapshot_low_vram_mode`, `initialize`.
**CN:** 该类实现 `LTX2SnapshotResidencyStrategy`，并继承/扩展 `LTX2TwoStageResidencyStrategy`。 文档字符串指出：Snapshot mode keeps CPU snapshots and prefetches the target DiT with async H2D. 其中较重要的方法包括 `__init__`, `_module_name_for_phase`, `_resolve_snapshot_low_vram_mode`, `initialize`。

### Lines 697-777: Class `LTX2TwoStageResidencyController` / 类 `LTX2TwoStageResidencyController`
```python
class LTX2TwoStageResidencyController:
    """
    LTX-2.3 two-stage residency controller.
    It builds the selected LTX2 ComponentResidencyStrategy and keeps the
    thin stage adapter methods that are specific to two-stage LoRA flow.

    Modes:
    - resident: keep both DiTs on GPU; phase switch is pointer rebinding only.
    - snapshot: keep CPU snapshots and prefetch the target DiT.
    - original: official two-stage semantics without premerged stage-2.
    """

    VALID_MODES = ("original", "snapshot", "resident")

# ...
        target_name = "transformer_2" if phase == "stage2" else "transformer"
        target_transformer = self.pipeline.get_module(target_name)
        if target_transformer is not None:
            refinement_stage.transformer = target_transformer
```
**EN:** This class models `LTX2TwoStageResidencyController`. LTX-2.3 two-stage residency controller. Important methods include `__init__`, `_resolve_mode`, `_build_strategy`, `strategy`.
**CN:** 该类实现 `LTX2TwoStageResidencyController`。 文档字符串指出：LTX-2.3 two-stage residency controller. 其中较重要的方法包括 `__init__`, `_resolve_mode`, `_build_strategy`, `strategy`。

### Lines 780-1032: Class `LTX2TwoStagePipeline` / 类 `LTX2TwoStagePipeline`
```python
class LTX2TwoStagePipeline(_BaseLTX2Pipeline):
    pipeline_name = "LTX2TwoStagePipeline"
    STAGE_2_DISTILLED_SIGMA_VALUES = [0.909375, 0.725, 0.421875, 0.0]
    STAGE_1_DISTILLED_LORA_STRENGTH = 0.0
    STAGE_2_DISTILLED_LORA_STRENGTH = 1.0
    STAGE_1_DENOISING_SAMPLER_NAME = "euler"
    STAGE_2_DENOISING_SAMPLER_NAME = "euler"

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._ltx2_residency = LTX2TwoStageResidencyController(self, self.server_args)
        self._use_premerged_stage2_transformer = (
            self._ltx2_residency.should_use_premerged
        )
# ...
                ),
            ]
        )
        _add_ltx2_decoding_stage(self)
```
**EN:** This class models `LTX2TwoStagePipeline` as a specialization of `_BaseLTX2Pipeline`. Important methods include `__init__`, `_should_merge_stage2_distilled_lora`, `initialize_pipeline`, `_initialize_premerged_stage2_transformer`.
**CN:** 该类实现 `LTX2TwoStagePipeline`，并继承/扩展 `_BaseLTX2Pipeline`。 其中较重要的方法包括 `__init__`, `_should_merge_stage2_distilled_lora`, `initialize_pipeline`, `_initialize_premerged_stage2_transformer`。

### Lines 1035-1042: Class `LTX2TwoStageHQPipeline` / 类 `LTX2TwoStageHQPipeline`
```python
class LTX2TwoStageHQPipeline(LTX2TwoStagePipeline):
    pipeline_name = "LTX2TwoStageHQPipeline"
    pipeline_config_cls = LTX2PipelineConfig
    sampling_params_cls = LTX23HQSamplingParams
    STAGE_1_DISTILLED_LORA_STRENGTH = 0.25
    STAGE_2_DISTILLED_LORA_STRENGTH = 0.5
    STAGE_1_DENOISING_SAMPLER_NAME = "res2s"
    STAGE_2_DENOISING_SAMPLER_NAME = "res2s"
```
**EN:** This class models `LTX2TwoStageHQPipeline` as a specialization of `LTX2TwoStagePipeline`.
**CN:** 该类实现 `LTX2TwoStageHQPipeline`，并继承/扩展 `LTX2TwoStagePipeline`。

### Lines 1043-1045: Top-level configuration / 顶层配置
```python


EntryClass = [LTX2Pipeline, LTX2TwoStagePipeline, LTX2TwoStageHQPipeline]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Artifact storage management / 产物存储管理

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.ltx_2`, `sglang.multimodal_gen.configs.sample.ltx_2`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_resident_strategies`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`
- **External / 外部**: `numpy`, `torch`, `diffusers`
- **Stdlib / 标准库**: `math`, `os`
