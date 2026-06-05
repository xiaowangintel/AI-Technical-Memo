# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ModelTaskType`, `STA_Mode`, and `TextConditioningOutput`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `ModelTaskType`、`STA_Mode` 和 `TextConditioningOutput` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-44: module setup and imports / 模块初始化与导入
```python
import json
import math
import os
from collections.abc import Callable
from dataclasses import asdict, dataclass, field, fields
from enum import Enum, auto
from typing import Any

import numpy as np
import PIL
import torch
from einops import rearrange

from sglang.multimodal_gen.configs.models import (
    DiTConfig,
    EncoderConfig,
    ModelConfig,
    VAEConfig,
)
from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.t5 import T5Config
from sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config import (
    ModelDeploymentConfig,
)
from sglang.multimodal_gen.configs.sample.sampling_params import DataType
from sglang.multimodal_gen.configs.utils import update_config_from_args
from sglang.multimodal_gen.runtime.distributed.cfg_policy import CFGPolicy
from sglang.multimodal_gen.runtime.distributed.communication_op import (
    sequence_model_parallel_all_gather,
)
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_sp_parallel_rank,
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.models.vision_utils import get_default_height_width
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import (
    FlexibleArgumentParser,
    StoreBoolean,
    shallow_asdict,
)
```
**EN:** This block establishes the module context and imports `json`, `math`, `os`, `collections.abc`, `dataclasses`, and `enum`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`math`、`os`、`collections.abc`、`dataclasses` 和 `enum`。这些依赖为后续实现提供所需符号。

### Lines 46-46: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 51-53: `ModelTaskType` class overview / `ModelTaskType` 类概览
```python
class ModelTaskType(Enum):
    # TODO: check if I2V/TI2V models can work w/wo text
```
**EN:** This block defines class `ModelTaskType`. It encapsulates model task type behavior. It inherits from `Enum`.
**CN:** 该代码块定义了类 `ModelTaskType`。 它用于封装 model task type 相关行为。 它继承自 `Enum`。

### Lines 54-61: supporting statements / 辅助语句
```python
    I2V = auto()  # Image to Video
    T2V = auto()  # Text to Video
    TI2V = auto()  # Text and Image to Video

    T2I = auto()  # Text to Image
    I2I = auto()  # Image to Image
    TI2I = auto()  # Image to Image or Text-Image to Image
    I2M = auto()  # Image to Mesh
```
**EN:** This block gathers supporting statements inside `ModelTaskType`. It updates names such as `I2V`, `T2V`, `TI2V`, `T2I`, `I2I`, and `TI2I`. The code collaborates with `auto`.
**CN:** 该代码块汇集了位于 `ModelTaskType` 内部的辅助语句。 它会更新 `I2V`、`T2V`、`TI2V`、`T2I`、`I2I` 和 `TI2I` 等名称。 代码会与 `auto` 协同工作。

### Lines 63-68: `is_image_gen` implementation / `is_image_gen` 实现
```python
    def is_image_gen(self) -> bool:
        return (
            self == ModelTaskType.T2I
            or self == ModelTaskType.I2I
            or self == ModelTaskType.TI2I
        )
```
**EN:** This block defines method `is_image_gen` on `ModelTaskType`. It handles is image gen logic.
**CN:** 该代码块定义了 `ModelTaskType` 的方法 `is_image_gen`。 它用于处理 is image gen 相关逻辑。

### Lines 70-75: `requires_image_input` implementation / `requires_image_input` 实现
```python
    def requires_image_input(self) -> bool:
        return (
            self == ModelTaskType.I2V
            or self == ModelTaskType.I2I
            or self == ModelTaskType.I2M
        )
```
**EN:** This block defines method `requires_image_input` on `ModelTaskType`. It handles requires image input logic.
**CN:** 该代码块定义了 `ModelTaskType` 的方法 `requires_image_input`。 它用于处理 requires image input 相关逻辑。

### Lines 77-84: `accepts_image_input` implementation / `accepts_image_input` 实现
```python
    def accepts_image_input(self) -> bool:
        return (
            self == ModelTaskType.I2V
            or self == ModelTaskType.I2I
            or self == ModelTaskType.TI2I
            or self == ModelTaskType.TI2V
            or self == ModelTaskType.I2M
        )
```
**EN:** This block defines method `accepts_image_input` on `ModelTaskType`. It handles accepts image input logic.
**CN:** 该代码块定义了 `ModelTaskType` 的方法 `accepts_image_input`。 它用于处理 accepts image input 相关逻辑。

### Lines 86-92: `data_type` implementation / `data_type` 实现
```python
    def data_type(self) -> DataType:
        if self == ModelTaskType.I2M:
            return DataType.MESH
        if self.is_image_gen():
            return DataType.IMAGE
        else:
            return DataType.VIDEO
```
**EN:** This block defines method `data_type` on `ModelTaskType`. It handles data type logic. Key calls include `self.is_image_gen`. The implementation branches on conditions.
**CN:** 该代码块定义了 `ModelTaskType` 的方法 `data_type`。 它用于处理 data type 相关逻辑。 关键调用包括 `self.is_image_gen`。 实现中包含条件分支。

### Lines 95-97: `STA_Mode` class overview / `STA_Mode` 类概览
```python
class STA_Mode(str, Enum):
    """STA (Sliding Tile Attention) modes."""
```
**EN:** This block defines class `STA_Mode`. STA (Sliding Tile Attention) modes. It inherits from `str`, and `Enum`.
**CN:** 该代码块定义了类 `STA_Mode`。 它用于封装 sta mode 相关行为。 它继承自 `str` 和 `Enum`。

### Lines 98-102: supporting statements / 辅助语句
```python
    STA_INFERENCE = "STA_inference"
    STA_SEARCHING = "STA_searching"
    STA_TUNING = "STA_tuning"
    STA_TUNING_CFG = "STA_tuning_cfg"
    NONE = None
```
**EN:** This block gathers supporting statements inside `STA_Mode`. It updates names such as `STA_INFERENCE`, `STA_SEARCHING`, `STA_TUNING`, `STA_TUNING_CFG`, and `NONE`.
**CN:** 该代码块汇集了位于 `STA_Mode` 内部的辅助语句。 它会更新 `STA_INFERENCE`、`STA_SEARCHING`、`STA_TUNING`、`STA_TUNING_CFG` 和 `NONE` 等名称。

### Lines 105-106: `postprocess_text` implementation / `postprocess_text` 实现
```python
def postprocess_text(output: BaseEncoderOutput, _text_inputs) -> torch.tensor:
    raise NotImplementedError
```
**EN:** This block defines function `postprocess_text`. It handles postprocess text logic. Parameters such as `output`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `postprocess_text`。 它用于处理 postprocess text 相关逻辑。 本段逻辑主要由 `output` 和 `_text_inputs` 等参数驱动。

### Lines 110-116: `TextConditioningOutput` class overview / `TextConditioningOutput` 类概览
```python
class TextConditioningOutput:
    """Text embeddings and masks aligned to postprocessed sequence length.

    `prompt_embeds_mask` and `prompt_seq_lens` describe real text tokens after
    model-specific trimming or packing, not the raw tokenizer output.
    """
```
**EN:** This block defines class `TextConditioningOutput`. Text embeddings and masks aligned to postprocessed sequence length. `prompt_embeds_mask` and `prompt_seq_lens` describe real text tokens after model-specific trimming or packing, not the raw tokenizer output.
**CN:** 该代码块定义了类 `TextConditioningOutput`。 它用于封装 text conditioning output 相关行为。

### Lines 117-119: supporting statements / 辅助语句
```python
    prompt_embeds: torch.Tensor
    prompt_embeds_mask: torch.Tensor | None = None
    prompt_seq_lens: list[int] | None = None
```
**EN:** This block gathers supporting statements inside `TextConditioningOutput`. It updates names such as `prompt_embeds`, `prompt_embeds_mask`, and `prompt_seq_lens`.
**CN:** 该代码块汇集了位于 `TextConditioningOutput` 内部的辅助语句。 它会更新 `prompt_embeds`、`prompt_embeds_mask` 和 `prompt_seq_lens` 等名称。

### Lines 122-144: `pad_text_embeddings_with_mask` implementation / `pad_text_embeddings_with_mask` 实现
```python
def pad_text_embeddings_with_mask(
    text_embeds: list[torch.Tensor],
) -> TextConditioningOutput:
    """Pad variable-length text embeddings and return the valid-token mask."""
    if not text_embeds:
        raise ValueError("text_embeds must contain at least one tensor")

    max_seq_len = max(e.size(0) for e in text_embeds)
    prompt_embeds = torch.stack(
        [
            torch.cat([e, e.new_zeros(max_seq_len - e.size(0), e.size(1))])
            for e in text_embeds
        ]
    )
    seq_lens = [int(e.size(0)) for e in text_embeds]
    seq_lens_tensor = torch.tensor(
        seq_lens,
        device=prompt_embeds.device,
        dtype=torch.long,
    )
    positions = torch.arange(max_seq_len, device=prompt_embeds.device).unsqueeze(0)
    prompt_embeds_mask = positions < seq_lens_tensor.unsqueeze(1)
    return TextConditioningOutput(prompt_embeds, prompt_embeds_mask, seq_lens)
```
**EN:** This block defines function `pad_text_embeddings_with_mask`. Pad variable-length text embeddings and return the valid-token mask. Key calls include `max`, `torch.stack`, `torch.tensor`, `torch.arange.unsqueeze`, and `TextConditioningOutput`. The implementation branches on conditions. Parameters such as `text_embeds` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pad_text_embeddings_with_mask`。 它用于处理 pad text embeddings with mask 相关逻辑。 关键调用包括 `max`、`torch.stack`、`torch.tensor`、`torch.arange.unsqueeze` 和 `TextConditioningOutput`。 实现中包含条件分支。 本段逻辑主要由 `text_embeds` 等参数驱动。

### Lines 147-179: `shard_rotary_emb_for_sp` implementation / `shard_rotary_emb_for_sp` 实现
```python
def shard_rotary_emb_for_sp(emb):
    """
    Shard rotary embeddings [S, D] along sequence for SP.
    If S is not divisible by SP degree, pad by repeating the last row.
    """
    # Sequence Parallelism: slice image RoPE to local shard if enabled
    try:
        from sglang.multimodal_gen.runtime.distributed.parallel_state import (
            get_sp_parallel_rank,
            get_sp_world_size,
        )

        sp_world_size = get_sp_world_size()
    except Exception:
        sp_world_size = 1
    seq_len = emb.shape[0]
    if seq_len % sp_world_size != 0:
        pad_len = sp_world_size - (seq_len % sp_world_size)
        pad = emb[-1:].repeat(pad_len, 1)
        emb = torch.cat([emb, pad], dim=0)
    if sp_world_size > 1:
        try:
            rank = get_sp_parallel_rank()
        except Exception:
            rank = 0
        seq_len = emb.shape[0]
        local_len = seq_len // sp_world_size
        start = rank * local_len
        end = start + local_len
        emb = emb[start:end]
        return emb
    else:
        return emb
```
**EN:** This block defines function `shard_rotary_emb_for_sp`. Shard rotary embeddings [S, D] along sequence for SP. If S is not divisible by SP degree, pad by repeating the last row. Key calls include `get_sp_world_size`, `emb.repeat`, `torch.cat`, and `get_sp_parallel_rank`. The implementation branches on conditions, handles exceptional paths. Parameters such as `emb` drive the behavior in this section.
**CN:** 该代码块定义了函数 `shard_rotary_emb_for_sp`。 它用于处理 shard rotary emb for sp 相关逻辑。 关键调用包括 `get_sp_world_size`、`emb.repeat`、`torch.cat` 和 `get_sp_parallel_rank`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `emb` 等参数驱动。

### Lines 182-194: `maybe_unpad_latents` implementation / `maybe_unpad_latents` 实现
```python
def maybe_unpad_latents(latents, batch):
    # If SP padding was applied, remove extra tokens before reshaping
    raw_shape = batch.raw_latent_shape
    if len(raw_shape) == 3:
        # Sequence format [B, S, D]: use seq_len directly
        target_tokens = raw_shape[1]
    else:
        # Spatial format [B, C, H, W] or [B, C, T, H, W]: use width * height
        width, height = raw_shape[-1], raw_shape[-2]
        target_tokens = width * height
    if latents.shape[1] > target_tokens:
        latents = latents[:, :target_tokens, :]
    return latents
```
**EN:** This block defines function `maybe_unpad_latents`. It handles maybe unpad latents logic. Key calls include `len`. The implementation branches on conditions. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了函数 `maybe_unpad_latents`。 它用于处理 maybe unpad latents 相关逻辑。 关键调用包括 `len`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 198-200: `PipelineConfig` class overview / `PipelineConfig` 类概览
```python
class PipelineConfig:
    """The base configuration class for a generation pipeline."""
```
**EN:** This block defines class `PipelineConfig`. The base configuration class for a generation pipeline.
**CN:** 该代码块定义了类 `PipelineConfig`。 它用于封装 pipeline config 相关行为。

### Lines 201-242: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2I
    skip_input_image_preprocess: bool = False

    model_path: str = ""
    pipeline_config_path: str | None = None

    # precision and autocast
    enable_autocast: bool = True

    # generation parameters
    # controls the timestep embedding generation
    should_use_guidance: bool = True
    embedded_cfg_scale: float = 6.0
    cfg_policy: CFGPolicy = field(default_factory=CFGPolicy)
    generator_device: str | None = None
    flow_shift: float | None = None
    disable_autocast: bool = False

    # Model configuration
    dit_config: DiTConfig = field(default_factory=DiTConfig)
    dit_precision: str = "bf16"

    # VAE configuration
    vae_config: VAEConfig = field(default_factory=VAEConfig)
    vae_precision: str = "fp32"
    vae_tiling: bool = True
    vae_slicing: bool = False
    vae_sp: bool = True

    # Image encoder configuration
    image_encoder_config: EncoderConfig = field(default_factory=EncoderConfig)
    image_encoder_precision: str = "fp32"
    image_encoder_extra_args: dict = field(default_factory=lambda: {})

    # Text encoder configuration
    DEFAULT_TEXT_ENCODER_PRECISIONS = ("fp32",)
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (EncoderConfig(),)
    )
    # See PRECISION_TO_TYPE for detailed mapping
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("fp32",))
    text_encoder_extra_args: list[dict] = field(default_factory=lambda: [{}])
```
**EN:** This block gathers supporting statements inside `PipelineConfig`. It updates names such as `task_type`, `skip_input_image_preprocess`, `model_path`, `pipeline_config_path`, `enable_autocast`, and `should_use_guidance`. The code collaborates with `field`, and `EncoderConfig`.
**CN:** 该代码块汇集了位于 `PipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`skip_input_image_preprocess`、`model_path`、`pipeline_config_path`、`enable_autocast` 和 `should_use_guidance` 等名称。 代码会与 `field` 和 `EncoderConfig` 协同工作。

### Lines 244-245: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig()
```
**EN:** This block defines method `get_model_deployment_config` on `PipelineConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 247-248: `postprocess_image` implementation / `postprocess_image` 实现
```python
    def postprocess_image(self, image):
        return image.last_hidden_state
```
**EN:** This block defines method `postprocess_image` on `PipelineConfig`. It handles postprocess image logic. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `postprocess_image`。 它用于处理 postprocess image 相关逻辑。 本段逻辑主要由 `image` 等参数驱动。

### Lines 250-265: supporting statements / 辅助语句
```python
    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (None,)
    )

    # get prompt_embeds from encoder output
    postprocess_text_funcs: tuple[Callable[[BaseEncoderOutput], torch.tensor], ...] = (
        field(default_factory=lambda: (postprocess_text,))
    )

    # STA (Sliding Tile Attention) parameters
    mask_strategy_file_path: str | None = None
    STA_mode: STA_Mode = STA_Mode.STA_INFERENCE
    skip_time_steps: int = 15

    # DMD parameters
    dmd_denoising_steps: list[int] | None = field(default=None)
```
**EN:** This block gathers supporting statements inside `PipelineConfig`. It updates names such as `preprocess_text_funcs`, `postprocess_text_funcs`, `mask_strategy_file_path`, `STA_mode`, `skip_time_steps`, and `dmd_denoising_steps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `PipelineConfig` 内部的辅助语句。 它会更新 `preprocess_text_funcs`、`postprocess_text_funcs`、`mask_strategy_file_path`、`STA_mode`、`skip_time_steps` 和 `dmd_denoising_steps` 等名称。 代码会与 `field` 协同工作。

### Lines 267-269: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        # return the model-specific config for optimal deployment setting
        return ModelDeploymentConfig()
```
**EN:** This block defines method `get_model_deployment_config` on `PipelineConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 272-272: supporting statements / 辅助语句
```python
    boundary_ratio: float | None = None
```
**EN:** This block gathers supporting statements inside `PipelineConfig`. It updates names such as `boundary_ratio`.
**CN:** 该代码块汇集了位于 `PipelineConfig` 内部的辅助语句。 它会更新 `boundary_ratio` 等名称。

### Lines 280-283: `calculate_condition_image_size` implementation / `calculate_condition_image_size` 实现
```python
    def calculate_condition_image_size(self, image, width, height) -> tuple[int, int]:
        vae_scale_factor = self.vae_config.arch_config.spatial_compression_ratio
        height, width = get_default_height_width(image, vae_scale_factor, height, width)
        return width, height
```
**EN:** This block defines method `calculate_condition_image_size` on `PipelineConfig`. It handles calculate condition image size logic. Key calls include `get_default_height_width`. Parameters such as `image`, `width`, and `height` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `calculate_condition_image_size`。 它用于处理 calculate condition image size 相关逻辑。 关键调用包括 `get_default_height_width`。 本段逻辑主要由 `image`、`width` 和 `height` 等参数驱动。

### Lines 287-288: `prepare_sigmas` implementation / `prepare_sigmas` 实现
```python
    def prepare_sigmas(self, sigmas, num_inference_steps):
        return sigmas
```
**EN:** This block defines method `prepare_sigmas` on `PipelineConfig`. It prepares sigmas. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_sigmas`。 它用于准备sigmas。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 290-291: `get_classifier_free_guidance_scale` implementation / `get_classifier_free_guidance_scale` 实现
```python
    def get_classifier_free_guidance_scale(self, batch, guidance_scale: float) -> float:
        return guidance_scale
```
**EN:** This block defines method `get_classifier_free_guidance_scale` on `PipelineConfig`. It retrieves classifier free guidance scale. Parameters such as `batch`, and `guidance_scale` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_classifier_free_guidance_scale`。 它用于获取classifier free guidance scale。 本段逻辑主要由 `batch` 和 `guidance_scale` 等参数驱动。

### Lines 293-301: `postprocess_cfg_noise` implementation / `postprocess_cfg_noise` 实现
```python
    def postprocess_cfg_noise(
        self,
        batch,
        noise_pred: torch.Tensor,
        noise_pred_cond: torch.Tensor,
    ) -> torch.Tensor:
        # Model-specific CFG variants can override this hook
        # e.g. Qwen-Image's true-CFG norm matching.
        return noise_pred
```
**EN:** This block defines method `postprocess_cfg_noise` on `PipelineConfig`. It handles postprocess cfg noise logic. Parameters such as `batch`, `noise_pred`, and `noise_pred_cond` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `postprocess_cfg_noise`。 它用于处理 postprocess cfg noise 相关逻辑。 本段逻辑主要由 `batch`、`noise_pred` 和 `noise_pred_cond` 等参数驱动。

### Lines 304-312: `preprocess_condition_image` implementation / `preprocess_condition_image` 实现
```python
    def preprocess_condition_image(
        self, image, target_width, target_height, _vae_image_processor
    ):
        """
        preprocess the condition image, returns (image, final_image_width, final_image_height)
        """
        return image.resize(
            (target_width, target_height), PIL.Image.Resampling.LANCZOS
        ), (target_width, target_height)
```
**EN:** This block defines method `preprocess_condition_image` on `PipelineConfig`. preprocess the condition image, returns (image, final_image_width, final_image_height) Key calls include `image.resize`. Parameters such as `image`, `target_width`, `target_height`, and `_vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `preprocess_condition_image`。 它用于处理 preprocess condition image 相关逻辑。 关键调用包括 `image.resize`。 本段逻辑主要由 `image`、`target_width`、`target_height` 和 `_vae_image_processor` 等参数驱动。

### Lines 314-315: `prepare_calculated_size` implementation / `prepare_calculated_size` 实现
```python
    def prepare_calculated_size(self, image):
        return self.calculate_condition_image_size(image, image.width, image.height)
```
**EN:** This block defines method `prepare_calculated_size` on `PipelineConfig`. It prepares calculated size. Key calls include `self.calculate_condition_image_size`. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_calculated_size`。 它用于准备calculated size。 关键调用包括 `self.calculate_condition_image_size`。 本段逻辑主要由 `image` 等参数驱动。

### Lines 317-318: `prepare_image_processor_kwargs` implementation / `prepare_image_processor_kwargs` 实现
```python
    def prepare_image_processor_kwargs(self, batch, neg=False):
        return {}
```
**EN:** This block defines method `prepare_image_processor_kwargs` on `PipelineConfig`. It prepares image processor kwargs. Parameters such as `batch`, and `neg` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_image_processor_kwargs`。 它用于准备image processor kwargs。 本段逻辑主要由 `batch` 和 `neg` 等参数驱动。

### Lines 320-348: `postprocess_image_latent` implementation / `postprocess_image_latent` 实现
```python
    def postprocess_image_latent(self, latent_condition, batch):
        vae_arch_config = self.vae_config.arch_config
        spatial_compression_ratio = vae_arch_config.spatial_compression_ratio
        temporal_compression_ratio = vae_arch_config.temporal_compression_ratio
        num_frames = batch.num_frames
        latent_height = batch.height // spatial_compression_ratio
        latent_width = batch.width // spatial_compression_ratio
        mask_lat_size = torch.ones(1, 1, num_frames, latent_height, latent_width)
        mask_lat_size[:, :, 1:] = 0
        first_frame_mask = mask_lat_size[:, :, 0:1]
        first_frame_mask = torch.repeat_interleave(
            first_frame_mask,
            repeats=temporal_compression_ratio,
            dim=2,
        )
        mask_lat_size = torch.concat(
            [first_frame_mask, mask_lat_size[:, :, 1:, :]], dim=2
        )
        mask_lat_size = mask_lat_size.view(
            1,
            -1,
            temporal_compression_ratio,
            latent_height,
            latent_width,
        )
        mask_lat_size = mask_lat_size.transpose(1, 2)
        mask_lat_size = mask_lat_size.to(latent_condition.device)
        image_latents = torch.concat([mask_lat_size, latent_condition], dim=1)
        return image_latents
```
**EN:** This block defines method `postprocess_image_latent` on `PipelineConfig`. It handles postprocess image latent logic. Key calls include `torch.ones`, `torch.repeat_interleave`, `torch.concat`, `mask_lat_size.view`, and `mask_lat_size.transpose`. Parameters such as `latent_condition`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `postprocess_image_latent`。 它用于处理 postprocess image latent 相关逻辑。 关键调用包括 `torch.ones`、`torch.repeat_interleave`、`torch.concat`、`mask_lat_size.view` 和 `mask_lat_size.transpose`。 本段逻辑主要由 `latent_condition` 和 `batch` 等参数驱动。

### Lines 350-351: `slice_noise_pred` implementation / `slice_noise_pred` 实现
```python
    def slice_noise_pred(self, noise, latents):
        return noise
```
**EN:** This block defines method `slice_noise_pred` on `PipelineConfig`. It handles slice noise pred logic. Parameters such as `noise`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `slice_noise_pred`。 它用于处理 slice noise pred 相关逻辑。 本段逻辑主要由 `noise` 和 `latents` 等参数驱动。

### Lines 353-354: `adjust_num_frames` implementation / `adjust_num_frames` 实现
```python
    def adjust_num_frames(self, num_frames):
        return num_frames
```
**EN:** This block defines method `adjust_num_frames` on `PipelineConfig`. It handles adjust num frames logic. Parameters such as `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `adjust_num_frames`。 它用于处理 adjust num frames 相关逻辑。 本段逻辑主要由 `num_frames` 等参数驱动。

### Lines 357-358: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompt: list[str], tokenizer, tok_kwargs) -> dict:
        return tokenizer(prompt, **tok_kwargs)
```
**EN:** This block defines method `tokenize_prompt` on `PipelineConfig`. It handles tokenize prompt logic. Key calls include `tokenizer`. Parameters such as `prompt`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tokenizer`。 本段逻辑主要由 `prompt`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 360-365: `is_flux_v1` implementation / `is_flux_v1` 实现
```python
    def is_flux_v1(self) -> bool:
        """True if this pipeline is FLUX v1 (dual CLIP + T5 text encoders).

        Used by text encoding (e.g. fixed CLIP context). Other pipelines return False.
        """
        return False
```
**EN:** This block defines method `is_flux_v1` on `PipelineConfig`. True if this pipeline is FLUX v1 (dual CLIP + T5 text encoders). Used by text encoding (e.g.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `is_flux_v1`。 它用于处理 is flux v1 相关逻辑。

### Lines 367-380: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        height = batch.height // self.vae_config.arch_config.spatial_compression_ratio
        width = batch.width // self.vae_config.arch_config.spatial_compression_ratio

        # Calculate latent shape
        shape = (
            batch_size,
            self.dit_config.num_channels_latents,
            num_frames,
            height,
            width,
        )

        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `PipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 382-383: `get_latent_dtype` implementation / `get_latent_dtype` 实现
```python
    def get_latent_dtype(self, prompt_dtype: torch.dtype) -> torch.dtype:
        return prompt_dtype
```
**EN:** This block defines method `get_latent_dtype` on `PipelineConfig`. It retrieves latent dtype. Parameters such as `prompt_dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_latent_dtype`。 它用于获取latent dtype。 本段逻辑主要由 `prompt_dtype` 等参数驱动。

### Lines 385-386: `allow_set_num_frames` implementation / `allow_set_num_frames` 实现
```python
    def allow_set_num_frames(self):
        return False
```
**EN:** This block defines method `allow_set_num_frames` on `PipelineConfig`. It handles allow set num frames logic.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `allow_set_num_frames`。 它用于处理 allow set num frames 相关逻辑。

### Lines 388-393: `supports_dynamic_batching` implementation / `supports_dynamic_batching` 实现
```python
    def supports_dynamic_batching(self):
        """Return whether this pipeline can opt in to dynamic batching.

        The scheduler still checks each request before merging it into a batch.
        """
        return self.task_type in (ModelTaskType.T2I, ModelTaskType.T2V)
```
**EN:** This block defines method `supports_dynamic_batching` on `PipelineConfig`. Return whether this pipeline can opt in to dynamic batching. The scheduler still checks each request before merging it into a batch.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `supports_dynamic_batching`。 它用于处理 supports dynamic batching 相关逻辑。

### Lines 395-421: `estimate_request_cost` implementation / `estimate_request_cost` 实现
```python
    def estimate_request_cost(self, batch) -> float:
        """Return the relative cost used for batching admission caps.

        This is compared with `max_cost` from the batching config; it is not a
        memory estimate. The default cost is latent tokens times frames times
        outputs; pipelines can override it for model-specific admission.
        """
        latent_tokens = float(batch.n_tokens or 0)
        if latent_tokens <= 0:
            width = int(batch.width or 0)
            height = int(batch.height or 0)
            if width > 0 and height > 0:
                vae_scale = getattr(
                    self.vae_config.arch_config, "vae_scale_factor", None
                )
                if vae_scale is None and hasattr(
                    self.vae_config, "get_vae_scale_factor"
                ):
                    vae_scale = self.vae_config.get_vae_scale_factor()
                vae_scale = max(1, int(vae_scale or 1))
                latent_tokens = math.ceil(width / vae_scale) * math.ceil(
                    height / vae_scale
                )

        num_frames = max(1, int(batch.num_frames or 1))
        num_outputs = max(1, int(batch.num_outputs_per_prompt or 1))
        return latent_tokens * num_frames * num_outputs
```
**EN:** This block defines method `estimate_request_cost` on `PipelineConfig`. Return the relative cost used for batching admission caps. This is compared with `max_cost` from the batching config; it is not a memory estimate. Key calls include `float`, `max`, `int`, `getattr`, and `hasattr`. The implementation branches on conditions. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `estimate_request_cost`。 它用于处理 estimate request cost 相关逻辑。 关键调用包括 `float`、`max`、`int`、`getattr` 和 `hasattr`。 实现中包含条件分支。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 423-432: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        vae_arch_config = self.vae_config.arch_config
        scaling_factor = getattr(vae_arch_config, "scaling_factor", None)
        if scaling_factor is None:
            scaling_factor = getattr(vae, "scaling_factor", None)

        shift_factor = getattr(vae_arch_config, "shift_factor", None)
        if shift_factor is None:
            shift_factor = getattr(vae, "shift_factor", None)
        return scaling_factor, shift_factor
```
**EN:** This block defines method `get_decode_scale_and_shift` on `PipelineConfig`. It retrieves decode scale and shift. Key calls include `getattr`. The implementation branches on conditions. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 435-436: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        return latents
```
**EN:** This block defines method `maybe_pack_latents` on `PipelineConfig`. It handles maybe pack latents logic. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 438-439: `maybe_prepare_latent_ids` implementation / `maybe_prepare_latent_ids` 实现
```python
    def maybe_prepare_latent_ids(self, latents):
        return None
```
**EN:** This block defines method `maybe_prepare_latent_ids` on `PipelineConfig`. It handles maybe prepare latent ids logic. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `maybe_prepare_latent_ids`。 它用于处理 maybe prepare latent ids 相关逻辑。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 442-443: `postprocess_vae_encode` implementation / `postprocess_vae_encode` 实现
```python
    def postprocess_vae_encode(self, image_latents, vae):
        return image_latents
```
**EN:** This block defines method `postprocess_vae_encode` on `PipelineConfig`. It handles postprocess vae encode logic. Parameters such as `image_latents`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `postprocess_vae_encode`。 它用于处理 postprocess vae encode 相关逻辑。 本段逻辑主要由 `image_latents` 和 `vae` 等参数驱动。

### Lines 446-447: `normalize_vae_encode` implementation / `normalize_vae_encode` 实现
```python
    def normalize_vae_encode(self, image_latents, vae):
        return None
```
**EN:** This block defines method `normalize_vae_encode` on `PipelineConfig`. It handles normalize vae encode logic. Parameters such as `image_latents`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `normalize_vae_encode`。 它用于处理 normalize vae encode 相关逻辑。 本段逻辑主要由 `image_latents` 和 `vae` 等参数驱动。

### Lines 450-451: `preprocess_decoding` implementation / `preprocess_decoding` 实现
```python
    def preprocess_decoding(self, latents, server_args=None, vae=None):
        return latents
```
**EN:** This block defines method `preprocess_decoding` on `PipelineConfig`. It handles preprocess decoding logic. Parameters such as `latents`, `server_args`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `preprocess_decoding`。 它用于处理 preprocess decoding 相关逻辑。 本段逻辑主要由 `latents`、`server_args` 和 `vae` 等参数驱动。

### Lines 453-456: `_gather_sp_tensor` implementation / `_gather_sp_tensor` 实现
```python
    @staticmethod
    def _gather_sp_tensor(tensor: torch.Tensor, *, dim: int) -> torch.Tensor:
        """All-gather an SP-sharded tensor along the specified logical dimension."""
        return sequence_model_parallel_all_gather(tensor.contiguous(), dim=dim)
```
**EN:** This block defines method `_gather_sp_tensor` on `PipelineConfig`. All-gather an SP-sharded tensor along the specified logical dimension. Key calls include `sequence_model_parallel_all_gather`, and `tensor.contiguous`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `_gather_sp_tensor`。 它用于处理 gather sp tensor 相关逻辑。 关键调用包括 `sequence_model_parallel_all_gather` 和 `tensor.contiguous`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 458-470: `_trim_sp_gather_padding` implementation / `_trim_sp_gather_padding` 实现
```python
    @staticmethod
    def _trim_sp_gather_padding(
        tensor: torch.Tensor, *, orig_len: int | None, dim: int
    ) -> torch.Tensor:
        """Trim padding introduced before SP sharding back to the original length."""
        if orig_len is None:
            return tensor
        orig_len = int(orig_len)
        if orig_len <= 0 or tensor.shape[dim] <= orig_len:
            return tensor
        slices = [slice(None)] * tensor.ndim
        slices[dim] = slice(orig_len)
        return tensor[tuple(slices)]
```
**EN:** This block defines method `_trim_sp_gather_padding` on `PipelineConfig`. Trim padding introduced before SP sharding back to the original length. Key calls include `int`, `slice`, and `tuple`. The implementation branches on conditions. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `_trim_sp_gather_padding`。 它用于处理 trim sp gather padding 相关逻辑。 关键调用包括 `int`、`slice` 和 `tuple`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 472-474: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents, batch=None):
        # For video latents [B, C, T_local, H, W], gather along time dim=2
        return self._gather_sp_tensor(latents, dim=2)
```
**EN:** This block defines method `gather_latents_for_sp` on `PipelineConfig`. It handles gather latents for sp logic. Key calls include `self._gather_sp_tensor`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 关键调用包括 `self._gather_sp_tensor`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 476-478: `can_shard_audio_latents_for_sp` implementation / `can_shard_audio_latents_for_sp` 实现
```python
    def can_shard_audio_latents_for_sp(self, audio_latents) -> bool:
        """Return whether this pipeline uses packed audio latents that can be SP-sharded."""
        return False
```
**EN:** This block defines method `can_shard_audio_latents_for_sp` on `PipelineConfig`. Return whether this pipeline uses packed audio latents that can be SP-sharded. Parameters such as `audio_latents` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `can_shard_audio_latents_for_sp`。 它用于处理 can shard audio latents for sp 相关逻辑。 本段逻辑主要由 `audio_latents` 等参数驱动。

### Lines 480-482: `shard_audio_latents_for_sp` implementation / `shard_audio_latents_for_sp` 实现
```python
    def shard_audio_latents_for_sp(self, batch, audio_latents):
        """Shard packed audio latents for SP. Pipelines without packed audio latents should return the input unchanged."""
        return audio_latents, False
```
**EN:** This block defines method `shard_audio_latents_for_sp` on `PipelineConfig`. Shard packed audio latents for SP. Pipelines without packed audio latents should return the input unchanged. Parameters such as `batch`, and `audio_latents` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `shard_audio_latents_for_sp`。 它用于处理 shard audio latents for sp 相关逻辑。 本段逻辑主要由 `batch` 和 `audio_latents` 等参数驱动。

### Lines 484-486: `gather_audio_latents_for_sp` implementation / `gather_audio_latents_for_sp` 实现
```python
    def gather_audio_latents_for_sp(self, audio_latents, batch):
        """Gather SP-sharded audio latents back to full sequence length."""
        return audio_latents
```
**EN:** This block defines method `gather_audio_latents_for_sp` on `PipelineConfig`. Gather SP-sharded audio latents back to full sequence length. Parameters such as `audio_latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `gather_audio_latents_for_sp`。 它用于处理 gather audio latents for sp 相关逻辑。 本段逻辑主要由 `audio_latents` 和 `batch` 等参数驱动。

### Lines 488-499: `prepare_video_rope_coords_for_sp` implementation / `prepare_video_rope_coords_for_sp` 实现
```python
    def prepare_video_rope_coords_for_sp(
        self,
        model,
        batch,
        latent_model_input,
        *,
        num_frames,
        height,
        width,
    ):
        """Prepare model-side video RoPE coordinates for the local SP shard when the pipeline requires them."""
        return None
```
**EN:** This block defines method `prepare_video_rope_coords_for_sp` on `PipelineConfig`. Prepare model-side video RoPE coordinates for the local SP shard when the pipeline requires them. Parameters such as `model`, `batch`, and `latent_model_input` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_video_rope_coords_for_sp`。 它用于准备video rope coords for sp。 本段逻辑主要由 `model`、`batch` 和 `latent_model_input` 等参数驱动。

### Lines 501-510: `prepare_audio_rope_coords_for_sp` implementation / `prepare_audio_rope_coords_for_sp` 实现
```python
    def prepare_audio_rope_coords_for_sp(
        self,
        model,
        batch,
        audio_latent_model_input,
        *,
        num_frames,
    ):
        """Prepare model-side audio RoPE coordinates for the local SP shard when the pipeline requires them."""
        return None
```
**EN:** This block defines method `prepare_audio_rope_coords_for_sp` on `PipelineConfig`. Prepare model-side audio RoPE coordinates for the local SP shard when the pipeline requires them. Parameters such as `model`, `batch`, and `audio_latent_model_input` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_audio_rope_coords_for_sp`。 它用于准备audio rope coords for sp。 本段逻辑主要由 `model`、`batch` 和 `audio_latent_model_input` 等参数驱动。

### Lines 512-519: `gather_noise_pred_for_sp` implementation / `gather_noise_pred_for_sp` 实现
```python
    def gather_noise_pred_for_sp(self, batch, noise_pred):
        noise_pred = self.gather_latents_for_sp(noise_pred)
        raw_latent_shape = getattr(batch, "raw_latent_shape", None)
        if raw_latent_shape is not None and noise_pred.dim() == 3:
            noise_pred = self._trim_sp_gather_padding(
                noise_pred, orig_len=raw_latent_shape[1], dim=1
            )
        return noise_pred
```
**EN:** This block defines method `gather_noise_pred_for_sp` on `PipelineConfig`. It handles gather noise pred for sp logic. Key calls include `self.gather_latents_for_sp`, `getattr`, `self._trim_sp_gather_padding`, and `noise_pred.dim`. The implementation branches on conditions. Parameters such as `batch`, and `noise_pred` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `gather_noise_pred_for_sp`。 它用于处理 gather noise pred for sp 相关逻辑。 关键调用包括 `self.gather_latents_for_sp`、`getattr`、`self._trim_sp_gather_padding` 和 `noise_pred.dim`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `noise_pred` 等参数驱动。

### Lines 521-522: `preprocess_vae_image` implementation / `preprocess_vae_image` 实现
```python
    def preprocess_vae_image(self, batch, vae_image_processor):
        pass
```
**EN:** This block defines method `preprocess_vae_image` on `PipelineConfig`. It handles preprocess vae image logic. Parameters such as `batch`, and `vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `preprocess_vae_image`。 它用于处理 preprocess vae image 相关逻辑。 本段逻辑主要由 `batch` 和 `vae_image_processor` 等参数驱动。

### Lines 524-551: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        # general logic for video models
        sp_world_size, rank_in_sp_group = get_sp_world_size(), get_sp_parallel_rank()
        if batch.enable_sequence_shard and sp_world_size > 1:
            return latents, False
        if latents.dim() != 5:
            return latents, False
        time_dim = latents.shape[2]

        # Pad to next multiple of SP degree if needed
        if time_dim > 0 and time_dim % sp_world_size != 0:
            logger.debug(
                "Padding latents to next multiple of SP degree, performance is sub-optimal"
            )
            pad_len = sp_world_size - (time_dim % sp_world_size)
            pad = torch.zeros(
                (*latents.shape[:2], pad_len, *latents.shape[3:]),
                dtype=latents.dtype,
                device=latents.device,
            )
            latents = torch.cat([latents, pad], dim=2)

        assert latents.shape[2] % sp_world_size == 0
        sharded_tensor = rearrange(
            latents, "b c (n t) h w -> b c n t h w", n=sp_world_size
        ).contiguous()
        sharded_tensor = sharded_tensor[:, :, rank_in_sp_group, :, :, :]
        return sharded_tensor, True
```
**EN:** This block defines method `shard_latents_for_sp` on `PipelineConfig`. It handles shard latents for sp logic. Key calls include `rearrange.contiguous`, `get_sp_world_size`, `get_sp_parallel_rank`, `latents.dim`, and `logger.debug`. The implementation branches on conditions. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 关键调用包括 `rearrange.contiguous`、`get_sp_world_size`、`get_sp_parallel_rank`、`latents.dim` 和 `logger.debug`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 553-560: `get_text_encoder_attention_mask` implementation / `get_text_encoder_attention_mask` 实现
```python
    def get_text_encoder_attention_mask(
        self, text_inputs: dict, encoder_index: int
    ) -> "torch.Tensor | None":
        """Return the attention mask for the given text encoder.

        Override to suppress (return None) or modify the mask per model.
        """
        return text_inputs.get("attention_mask")
```
**EN:** This block defines method `get_text_encoder_attention_mask` on `PipelineConfig`. Return the attention mask for the given text encoder. Override to suppress (return None) or modify the mask per model. Key calls include `text_inputs.get`. Parameters such as `text_inputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_text_encoder_attention_mask`。 它用于获取text encoder attention mask。 关键调用包括 `text_inputs.get`。 本段逻辑主要由 `text_inputs` 和 `encoder_index` 等参数驱动。

### Lines 562-607: `build_text_conditioning_mask` implementation / `build_text_conditioning_mask` 实现
```python
    def build_text_conditioning_mask(
        self,
        text_inputs: dict,
        text_encoder_attention_mask: "torch.Tensor | None",
        prompt_embeds: "torch.Tensor",
        encoder_index: int,
    ) -> "torch.Tensor":
        """Return a mask aligned with post-processed prompt embeddings.

        True values mark valid text tokens. Dynamic batching must carry
        post-processed semantic text lengths explicitly; if a model-specific
        postprocessor changes the sequence length, it must return
        TextConditioningOutput with an embedding-aligned mask.
        """
        if prompt_embeds.ndim < 2:
            raise ValueError(
                "prompt_embeds must have shape [batch, seq, ...] to build text conditioning mask"
            )

        if prompt_embeds.ndim == 2:
            batch_size, embed_seq_len = 1, prompt_embeds.shape[0]
        else:
            batch_size, embed_seq_len = prompt_embeds.shape[:2]
        device = prompt_embeds.device
        if text_encoder_attention_mask is None:
            return torch.ones(
                (batch_size, embed_seq_len), dtype=torch.bool, device=device
            )

        raw_mask = text_encoder_attention_mask.to(device=device).bool()
        if raw_mask.ndim != 2 or raw_mask.shape[0] != batch_size:
            raise ValueError(
                "text attention mask must have shape [batch, seq] matching prompt_embeds batch"
            )

        if raw_mask.shape[1] == embed_seq_len:
            return raw_mask

        if prompt_embeds.ndim == 2 and raw_mask.shape[0] == 1:
            return torch.ones((1, embed_seq_len), dtype=torch.bool, device=device)

        raise ValueError(
            "text attention mask length does not match postprocessed prompt embeddings. "
            "Postprocess functions that trim, pack, or otherwise change text sequence "
            "length must return TextConditioningOutput with an embedding-aligned mask."
        )
```
**EN:** This block defines method `build_text_conditioning_mask` on `PipelineConfig`. Return a mask aligned with post-processed prompt embeddings. True values mark valid text tokens. Key calls include `text_encoder_attention_mask.to.bool`, `ValueError`, `torch.ones`, and `text_encoder_attention_mask.to`. The implementation branches on conditions. Parameters such as `text_inputs`, `text_encoder_attention_mask`, `prompt_embeds`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `build_text_conditioning_mask`。 它用于构建text conditioning mask。 关键调用包括 `text_encoder_attention_mask.to.bool`、`ValueError`、`torch.ones` 和 `text_encoder_attention_mask.to`。 实现中包含条件分支。 本段逻辑主要由 `text_inputs`、`text_encoder_attention_mask`、`prompt_embeds` 和 `encoder_index` 等参数驱动。

### Lines 609-613: `seq_lens_from_text_conditioning_mask` implementation / `seq_lens_from_text_conditioning_mask` 实现
```python
    @staticmethod
    def seq_lens_from_text_conditioning_mask(mask: "torch.Tensor") -> list[int]:
        if mask.ndim != 2:
            raise ValueError("text conditioning mask must have shape [batch, seq]")
        return torch.count_nonzero(mask, dim=1).tolist()
```
**EN:** This block defines method `seq_lens_from_text_conditioning_mask` on `PipelineConfig`. It handles seq lens from text conditioning mask logic. Key calls include `torch.count_nonzero.tolist`, `ValueError`, and `torch.count_nonzero`. The implementation branches on conditions. Parameters such as `mask` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `seq_lens_from_text_conditioning_mask`。 它用于处理 seq lens from text conditioning mask 相关逻辑。 关键调用包括 `torch.count_nonzero.tolist`、`ValueError` 和 `torch.count_nonzero`。 实现中包含条件分支。 本段逻辑主要由 `mask` 等参数驱动。

### Lines 615-646: `require_text_seq_lens` implementation / `require_text_seq_lens` 实现
```python
    def require_text_seq_lens(
        self,
        batch,
        encoder_index: int,
        *,
        negative: bool = False,
        expected_batch_size: int | None = None,
    ) -> list[int]:
        """Return postprocessed text lengths captured during text encoding.

        Dynamic batches use these lengths for model masks, RoPE, and cache
        sizing after text embeddings have been padded.
        """
        seq_lens_by_encoder = (
            batch.negative_prompt_seq_lens if negative else batch.prompt_seq_lens
        )
        kind = "negative" if negative else "positive"
        if seq_lens_by_encoder is None or encoder_index >= len(seq_lens_by_encoder):
            raise ValueError(
                f"Missing {kind} prompt_seq_lens for text encoder {encoder_index}; "
                "dynamic text conditioning requires explicit sequence lengths."
            )

        seq_lens = [int(x) for x in seq_lens_by_encoder[encoder_index]]
        if expected_batch_size is not None and len(seq_lens) != int(
            expected_batch_size
        ):
            raise ValueError(
                f"{kind} prompt_seq_lens for text encoder {encoder_index} has "
                f"{len(seq_lens)} entries, expected {expected_batch_size}."
            )
        return seq_lens
```
**EN:** This block defines method `require_text_seq_lens` on `PipelineConfig`. Return postprocessed text lengths captured during text encoding. Dynamic batches use these lengths for model masks, RoPE, and cache sizing after text embeddings have been padded. Key calls include `ValueError`, `int`, and `len`. The implementation branches on conditions. Parameters such as `batch`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `require_text_seq_lens`。 它用于处理 require text seq lens 相关逻辑。 关键调用包括 `ValueError`、`int` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `encoder_index` 等参数驱动。

### Lines 648-655: `get_text_encoder_pooler_output` implementation / `get_text_encoder_pooler_output` 实现
```python
    def get_text_encoder_pooler_output(
        self, outputs: "BaseEncoderOutput", encoder_index: int
    ) -> "torch.Tensor | None":
        """Return the pooler output for the given text encoder, or None to skip.

        Override for models that need pooled embeddings (e.g. FLUX v1, SD3).
        """
        return None
```
**EN:** This block defines method `get_text_encoder_pooler_output` on `PipelineConfig`. Return the pooler output for the given text encoder, or None to skip. Override for models that need pooled embeddings (e.g. Parameters such as `outputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_text_encoder_pooler_output`。 它用于获取text encoder pooler output。 本段逻辑主要由 `outputs` 和 `encoder_index` 等参数驱动。

### Lines 657-664: `select_vae_weight_files` implementation / `select_vae_weight_files` 实现
```python
    def select_vae_weight_files(
        self,
        safetensors_list: list[str],
        component_model_path: str,
        component_name: str,
        vae_precision: str,
    ) -> list[str]:
        return safetensors_list
```
**EN:** This block defines method `select_vae_weight_files` on `PipelineConfig`. It selects vae weight files. Parameters such as `safetensors_list`, `component_model_path`, `component_name`, and `vae_precision` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `select_vae_weight_files`。 它用于选择vae weight files。 本段逻辑主要由 `safetensors_list`、`component_model_path`、`component_name` 和 `vae_precision` 等参数驱动。

### Lines 666-667: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        return batch.prompt_embeds
```
**EN:** This block defines method `get_pos_prompt_embeds` on `PipelineConfig`. It retrieves pos prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 669-670: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return batch.negative_prompt_embeds
```
**EN:** This block defines method `get_neg_prompt_embeds` on `PipelineConfig`. It retrieves neg prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 672-674: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        latents = maybe_unpad_latents(latents, batch)
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `PipelineConfig`. It post-processes denoising loop. Key calls include `maybe_unpad_latents`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `maybe_unpad_latents`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 676-677: `post_decoding` implementation / `post_decoding` 实现
```python
    def post_decoding(self, frames, server_args):
        return frames
```
**EN:** This block defines method `post_decoding` on `PipelineConfig`. It post-processes decoding. Parameters such as `frames`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `post_decoding`。 它用于后处理decoding。 本段逻辑主要由 `frames` 和 `server_args` 等参数驱动。

### Lines 679-680: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {}
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `PipelineConfig`. It prepares pos cond kwargs. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 682-683: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {}
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `PipelineConfig`. It prepares neg cond kwargs. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 685-686: `_unpad_and_unpack_latents` implementation / `_unpad_and_unpack_latents` 实现
```python
    def _unpad_and_unpack_latents(self, latents, audio_latents, batch, vae, audio_vae):
        raise NotImplementedError("not yet implemented")
```
**EN:** This block defines method `_unpad_and_unpack_latents` on `PipelineConfig`. It handles unpad and unpack latents logic. Key calls include `NotImplementedError`. Parameters such as `latents`, `audio_latents`, `batch`, `vae`, and `audio_vae` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `_unpad_and_unpack_latents`。 它用于处理 unpad and unpack latents 相关逻辑。 关键调用包括 `NotImplementedError`。 本段逻辑主要由 `latents`、`audio_latents`、`batch`、`vae` 和 `audio_vae` 等参数驱动。

### Lines 688-689: `gather_denoising_env_static_for_sp` implementation / `gather_denoising_env_static_for_sp` 实现
```python
    def gather_denoising_env_static_for_sp(self, batch, cond_kwargs: dict | None):
        return cond_kwargs
```
**EN:** This block defines method `gather_denoising_env_static_for_sp` on `PipelineConfig`. It handles gather denoising env static for sp logic. Parameters such as `batch`, and `cond_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `gather_denoising_env_static_for_sp`。 它用于处理 gather denoising env static for sp 相关逻辑。 本段逻辑主要由 `batch` 和 `cond_kwargs` 等参数驱动。

### Lines 691-821: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(
        parser: FlexibleArgumentParser, prefix: str = ""
    ) -> FlexibleArgumentParser:
        prefix_with_dot = f"{prefix}." if (prefix.strip() != "") else ""

        # model_path will be conflicting with the model_path in ServerArgs,
        # so we add it separately if prefix is not empty
        if prefix_with_dot != "":
            parser.add_argument(
                f"--{prefix_with_dot}model-path",
                type=str,
                dest=f"{prefix_with_dot.replace('-', '_')}model_path",
                default=PipelineConfig.model_path,
                help="Path to the pretrained model",
            )

        parser.add_argument(
            f"--{prefix_with_dot}pipeline-config-path",
            type=str,
            dest=f"{prefix_with_dot.replace('-', '_')}pipeline_config_path",
            default=PipelineConfig.pipeline_config_path,
            help="Path to the pipeline config",
        )
        parser.add_argument(
            f"--{prefix_with_dot}embedded-cfg-scale",
            type=float,
            dest=f"{prefix_with_dot.replace('-', '_')}embedded_cfg_scale",
            default=PipelineConfig.embedded_cfg_scale,
            help="Embedded CFG scale",
        )
        parser.add_argument(
            f"--{prefix_with_dot}flow-shift",
            type=float,
            dest=f"{prefix_with_dot.replace('-', '_')}flow_shift",
            default=PipelineConfig.flow_shift,
            help="Flow shift parameter",
        )
        parser.add_argument(
            f"--{prefix_with_dot}resolution",
            type=int,
            dest=f"{prefix_with_dot.replace('-', '_')}resolution",
            default=None,
            help="Override the selected pipeline config's resolution setting. Only applies to pipelines that define a resolution field.",
        )

        # DiT configuration
        parser.add_argument(
            f"--{prefix_with_dot}dit-precision",
            type=str,
            dest=f"{prefix_with_dot.replace('-', '_')}dit_precision",
            default=PipelineConfig.dit_precision,
            choices=["fp32", "fp16", "bf16"],
            help="Precision for the DiT model",
        )

        # VAE configuration
        parser.add_argument(
            f"--{prefix_with_dot}vae-precision",
            type=str,
            dest=f"{prefix_with_dot.replace('-', '_')}vae_precision",
            default=PipelineConfig.vae_precision,
            choices=["fp32", "fp16", "bf16"],
            help="Precision for VAE",
        )
        parser.add_argument(
            f"--{prefix_with_dot}vae-tiling",
            action=StoreBoolean,
            dest=f"{prefix_with_dot.replace('-', '_')}vae_tiling",
            default=PipelineConfig.vae_tiling,
            help="Enable VAE tiling",
        )
        parser.add_argument(
            f"--{prefix_with_dot}vae-slicing",
            action=StoreBoolean,
            dest=f"{prefix_with_dot.replace('-', '_')}vae_slicing",
            default=PipelineConfig.vae_slicing,
            help="Enable VAE slicing",
        )
        parser.add_argument(
            f"--{prefix_with_dot}vae-sp",
            action=StoreBoolean,
            dest=f"{prefix_with_dot.replace('-', '_')}vae_sp",
            help="Enable VAE spatial parallelism",
        )

        # Text encoder configuration
        parser.add_argument(
            f"--{prefix_with_dot}text-encoder-precisions",
            nargs="+",
            type=str,
            dest=f"{prefix_with_dot.replace('-', '_')}text_encoder_precisions",
            default=PipelineConfig.DEFAULT_TEXT_ENCODER_PRECISIONS,
            choices=["fp32", "fp16", "bf16"],
            help="Precision for each text encoder",
        )

        # Image encoder configuration
        parser.add_argument(
            f"--{prefix_with_dot}image-encoder-precision",
            type=str,
            dest=f"{prefix_with_dot.replace('-', '_')}image_encoder_precision",
            default=PipelineConfig.image_encoder_precision,
            choices=["fp32", "fp16", "bf16"],
            help="Precision for image encoder",
        )

        # DMD parameters
        parser.add_argument(
            f"--{prefix_with_dot}dmd-denoising-steps",
            type=parse_int_list,
            default=PipelineConfig.dmd_denoising_steps,
            help="Comma-separated list of denoising steps (e.g., '1000,757,522')",
        )

        # Add VAE configuration arguments
        from sglang.multimodal_gen.configs.models.vaes.base import VAEConfig

        VAEConfig.add_cli_args(parser, prefix=f"{prefix_with_dot}vae-config")

        # Add DiT configuration arguments
        from sglang.multimodal_gen.configs.models.dits.base import DiTConfig

        DiTConfig.add_cli_args(parser, prefix=f"{prefix_with_dot}dit-config")

        # Add T5 configuration arguments
        from sglang.multimodal_gen.configs.models.encoders.t5 import T5Config

        T5Config.add_cli_args(parser, prefix=f"{prefix_with_dot}t5-config")

        return parser
```
**EN:** This block defines method `add_cli_args` on `PipelineConfig`. It handles add cli args logic. Key calls include `parser.add_argument`, `VAEConfig.add_cli_args`, `DiTConfig.add_cli_args`, `T5Config.add_cli_args`, and `prefix.strip`. The implementation branches on conditions. Parameters such as `parser`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument`、`VAEConfig.add_cli_args`、`DiTConfig.add_cli_args`、`T5Config.add_cli_args` 和 `prefix.strip`。 实现中包含条件分支。 本段逻辑主要由 `parser` 和 `prefix` 等参数驱动。

### Lines 823-839: `update_config_from_dict` implementation / `update_config_from_dict` 实现
```python
    def update_config_from_dict(self, args: dict[str, Any], prefix: str = "") -> None:
        prefix_with_dot = f"{prefix}." if (prefix.strip() != "") else ""
        update_config_from_args(self, args, prefix, pop_args=True)
        update_config_from_args(
            self.vae_config, args, f"{prefix_with_dot}vae_config", pop_args=True
        )
        update_config_from_args(
            self.dit_config, args, f"{prefix_with_dot}dit_config", pop_args=True
        )
        for text_encoder_config in self.text_encoder_configs:
            if isinstance(text_encoder_config, T5Config):
                update_config_from_args(
                    text_encoder_config,
                    args,
                    f"{prefix_with_dot}t5_config",
                    pop_args=True,
                )
```
**EN:** This block defines method `update_config_from_dict` on `PipelineConfig`. It updates config from dict. Key calls include `update_config_from_args`, `isinstance`, and `prefix.strip`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `args`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `update_config_from_dict`。 它用于更新config from dict。 关键调用包括 `update_config_from_args`、`isinstance` 和 `prefix.strip`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `args` 和 `prefix` 等参数驱动。

### Lines 841-957: `from_kwargs` implementation / `from_kwargs` 实现
```python
    @classmethod
    def from_kwargs(
        cls, kwargs: dict[str, Any], config_cli_prefix: str = ""
    ) -> "PipelineConfig":
        """
        Load PipelineConfig from kwargs Dictionary, as part of the ServerArg initialization process
        kwargs: dictionary of kwargs
        config_cli_prefix: prefix of CLI arguments for this PipelineConfig instance
        """
        from sglang.multimodal_gen.registry import get_model_info

        prefix_with_dot = (
            f"{config_cli_prefix}." if (config_cli_prefix.strip() != "") else ""
        )
        model_path: str | None = kwargs.get(
            prefix_with_dot + "model_path", None
        ) or kwargs.get("model_path")
        pipeline_config_or_path: str | PipelineConfig | dict[str, Any] | None = (
            kwargs.get(prefix_with_dot + "pipeline_config", None)
            or kwargs.get("pipeline_config")
        )
        if model_path is None:
            raise ValueError("model_path is required in kwargs")

        # Check if model_path is a safetensors file and pipeline_class_name is specified
        pipeline_class_name = kwargs.get(
            prefix_with_dot + "pipeline_class_name"
        ) or kwargs.get("pipeline_class_name")
        is_safetensors_file = os.path.isfile(model_path) and model_path.endswith(
            ".safetensors"
        )

        # 1. Get the pipeline config class from the registry
        from sglang.multimodal_gen.configs.pipeline_configs.flux import (
            Flux2PipelineConfig,
        )
        from sglang.multimodal_gen.registry import get_pipeline_config_classes

        # If model_path is a safetensors file and pipeline_class_name is specified,
        # try to get PipelineConfig from the registry first
        if is_safetensors_file and pipeline_class_name:
            config_classes = get_pipeline_config_classes(pipeline_class_name)
            if config_classes is not None:
                pipeline_config_cls, _ = config_classes
                logger.info(
                    f"Detected safetensors file with {pipeline_class_name}, "
                    f"using {pipeline_config_cls.__name__} directly without model_index.json"
                )
            else:
                model_info = get_model_info(
                    model_path,
                    backend=kwargs.get("backend"),
                    model_id=kwargs.get("model_id"),
                )
                if model_info is None:
                    from sglang.multimodal_gen.registry import (
                        _PIPELINE_CONFIG_REGISTRY,
                        _discover_and_register_pipelines,
                    )

                    _discover_and_register_pipelines()
                    available_pipelines = list(_PIPELINE_CONFIG_REGISTRY.keys())
                    raise ValueError(
                        f"Could not get model info for '{model_path}'. "
                        f"If using a safetensors file, please specify a valid pipeline_class_name. "
                        f"Available pipelines with config classes: {available_pipelines}"
                    )
                pipeline_config_cls = model_info.pipeline_config_cls
        else:
            model_info = get_model_info(
                model_path,
                backend=kwargs.get("backend"),
                model_id=kwargs.get("model_id"),
            )
            if model_info is None:
                raise ValueError(
                    f"Could not get model info for '{model_path}'. "
                    f"If using a safetensors file, please specify pipeline_class_name"
                )
            # 1.5. Adjust pipeline config for fine-tuned VAE if needed
            pipeline_config_cls = model_info.pipeline_config_cls
        vae_path = kwargs.get(prefix_with_dot + "vae_path") or kwargs.get("vae_path")
        if vae_path is None:
            component_paths = kwargs.get(
                prefix_with_dot + "component_paths"
            ) or kwargs.get("component_paths")
            if isinstance(component_paths, dict):
                vae_path = component_paths.get("vae")

        # Check if this is a Flux2 model with fal/FLUX.2-Tiny-AutoEncoder
        if (
            isinstance(pipeline_config_cls, type)
            and issubclass(pipeline_config_cls, Flux2PipelineConfig)
            and vae_path is not None
            and "FLUX.2-Tiny-AutoEncoder" in vae_path
        ):
            from sglang.multimodal_gen.configs.pipeline_configs.flux_finetuned import (
                Flux2FinetunedPipelineConfig,
            )

            pipeline_config_cls = Flux2FinetunedPipelineConfig

        pipeline_config = pipeline_config_cls()

        # 2. Load PipelineConfig from a json file or a PipelineConfig object if provided
        if isinstance(pipeline_config_or_path, str):
            pipeline_config.load_from_json(pipeline_config_or_path)
            kwargs[prefix_with_dot + "pipeline_config_path"] = pipeline_config_or_path
        elif isinstance(pipeline_config_or_path, PipelineConfig):
            pipeline_config = pipeline_config_or_path
        elif isinstance(pipeline_config_or_path, dict):
            pipeline_config.update_pipeline_config(pipeline_config_or_path)

        # 3. Update PipelineConfig from CLI arguments if provided
        kwargs[prefix_with_dot + "model_path"] = model_path
        pipeline_config.update_config_from_dict(kwargs, config_cli_prefix)
        return pipeline_config
```
**EN:** This block defines method `from_kwargs` on `PipelineConfig`. Load PipelineConfig from kwargs Dictionary, as part of the ServerArg initialization process kwargs: dictionary of kwargs config_cli_prefix: prefix of CLI arguments for this PipelineConfig instance Key calls include `pipeline_config_cls`, `isinstance`, `pipeline_config.update_config_from_dict`, `kwargs.get`, and `ValueError`. The implementation branches on conditions. Parameters such as `kwargs`, and `config_cli_prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `from_kwargs`。 它用于从…构造kwargs。 关键调用包括 `pipeline_config_cls`、`isinstance`、`pipeline_config.update_config_from_dict`、`kwargs.get` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `kwargs` 和 `config_cli_prefix` 等参数驱动。

### Lines 959-978: `check_pipeline_config` implementation / `check_pipeline_config` 实现
```python
    def check_pipeline_config(self) -> None:
        if self.vae_sp and not self.vae_tiling:
            raise ValueError(
                "Currently enabling vae_sp requires enabling vae_tiling, please set --vae-tiling to True."
            )

        if len(self.text_encoder_configs) != len(self.text_encoder_precisions):
            raise ValueError(
                f"Length of text encoder configs ({len(self.text_encoder_configs)}) must be equal to length of text encoder precisions ({len(self.text_encoder_precisions)})"
            )

        if len(self.text_encoder_configs) != len(self.preprocess_text_funcs):
            raise ValueError(
                f"Length of text encoder configs ({len(self.text_encoder_configs)}) must be equal to length of text preprocessing functions ({len(self.preprocess_text_funcs)})"
            )

        if len(self.preprocess_text_funcs) != len(self.postprocess_text_funcs):
            raise ValueError(
                f"Length of text postprocess functions ({len(self.postprocess_text_funcs)}) must be equal to length of text preprocessing functions ({len(self.preprocess_text_funcs)})"
            )
```
**EN:** This block defines method `check_pipeline_config` on `PipelineConfig`. It checks pipeline config. Key calls include `ValueError`, and `len`. The implementation branches on conditions.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `check_pipeline_config`。 它用于检查pipeline config。 关键调用包括 `ValueError` 和 `len`。 实现中包含条件分支。

### Lines 980-1007: `dump_to_json` implementation / `dump_to_json` 实现
```python
    def dump_to_json(self, file_path: str):
        output_dict = shallow_asdict(self)
        del_keys = []
        for key, value in output_dict.items():
            if isinstance(value, ModelConfig):
                model_dict = asdict(value)
                # Model Arch Config should be hidden away from the users
                model_dict.pop("arch_config")
                output_dict[key] = model_dict
            elif isinstance(value, tuple) and all(
                isinstance(v, ModelConfig) for v in value
            ):
                model_dicts = []
                for v in value:
                    model_dict = asdict(v)
                    # Model Arch Config should be hidden away from the users
                    model_dict.pop("arch_config")
                    model_dicts.append(model_dict)
                output_dict[key] = model_dicts
            elif isinstance(value, tuple) and all(callable(f) for f in value):
                # Skip dumping functions
                del_keys.append(key)

        for key in del_keys:
            output_dict.pop(key, None)

        with open(file_path, "w") as f:
            json.dump(output_dict, f, indent=2)
```
**EN:** This block defines method `dump_to_json` on `PipelineConfig`. It handles dump to json logic. Key calls include `shallow_asdict`, `output_dict.items`, `isinstance`, `output_dict.pop`, and `open`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `dump_to_json`。 它用于处理 dump to json 相关逻辑。 关键调用包括 `shallow_asdict`、`output_dict.items`、`isinstance`、`output_dict.pop` 和 `open`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 1009-1012: `load_from_json` implementation / `load_from_json` 实现
```python
    def load_from_json(self, file_path: str):
        with open(file_path) as f:
            input_pipeline_dict = json.load(f)
        self.update_pipeline_config(input_pipeline_dict)
```
**EN:** This block defines method `load_from_json` on `PipelineConfig`. It loads from json. Key calls include `self.update_pipeline_config`, `open`, and `json.load`. The implementation uses context-managed resources. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `load_from_json`。 它用于加载from json。 关键调用包括 `self.update_pipeline_config`、`open` 和 `json.load`。 实现中使用上下文管理资源。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 1014-1038: `update_pipeline_config` implementation / `update_pipeline_config` 实现
```python
    def update_pipeline_config(self, source_pipeline_dict: dict[str, Any]) -> None:
        for f in fields(self):
            key = f.name
            if key in source_pipeline_dict:
                current_value = getattr(self, key)
                new_value = source_pipeline_dict[key]

                # If it's a nested ModelConfig, update it recursively
                if isinstance(current_value, ModelConfig):
                    current_value.update_model_config(new_value)
                elif isinstance(current_value, tuple) and all(
                    isinstance(v, ModelConfig) for v in current_value
                ):
                    assert len(current_value) == len(
                        new_value
                    ), "Users shouldn't delete or add text encoder config objects in your json"
                    for target_config, source_config in zip(
                        current_value, new_value, strict=True
                    ):
                        target_config.update_model_config(source_config)
                else:
                    setattr(self, key, new_value)

        if hasattr(self, "__post_init__"):
            self.__post_init__()
```
**EN:** This block defines method `update_pipeline_config` on `PipelineConfig`. It updates pipeline config. Key calls include `fields`, `hasattr`, `self.__post_init__`, `getattr`, and `isinstance`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `source_pipeline_dict` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineConfig` 的方法 `update_pipeline_config`。 它用于更新pipeline config。 关键调用包括 `fields`、`hasattr`、`self.__post_init__`、`getattr` 和 `isinstance`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `source_pipeline_dict` 等参数驱动。

### Lines 1042-1044: `ImagePipelineConfig` class overview / `ImagePipelineConfig` 类概览
```python
class ImagePipelineConfig(PipelineConfig):
    """Base config for image generation pipelines with token-like latents [B, S, D]."""
```
**EN:** This block defines class `ImagePipelineConfig`. Base config for image generation pipelines with token-like latents [B, S, D]. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `ImagePipelineConfig`。 它用于封装 image pipeline config 相关行为。 它继承自 `PipelineConfig`。

### Lines 1045-1051: `_prepare_sigmas` implementation / `_prepare_sigmas` 实现
```python
    def _prepare_sigmas(self, sigmas, num_inference_steps):
        sigmas = (
            np.linspace(1.0, 1 / num_inference_steps, num_inference_steps)
            if sigmas is None
            else sigmas
        )
        return sigmas
```
**EN:** This block defines method `_prepare_sigmas` on `ImagePipelineConfig`. It prepares sigmas. Key calls include `np.linspace`. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `ImagePipelineConfig` 的方法 `_prepare_sigmas`。 它用于准备sigmas。 关键调用包括 `np.linspace`。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 1053-1075: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        # latents: [B, H * W, C]
        sp_world_size, rank_in_sp_group = get_sp_world_size(), get_sp_parallel_rank()
        if batch.enable_sequence_shard:
            return latents, False
        seq_len = latents.shape[1]

        # TODO: reuse code in PipelineConfig::shard_latents_for_sp
        # Pad to next multiple of SP degree if needed
        if seq_len % sp_world_size != 0:
            pad_len = sp_world_size - (seq_len % sp_world_size)
            pad = torch.zeros(
                (*latents.shape[:1], pad_len, *latents.shape[2:]),
                dtype=latents.dtype,
                device=latents.device,
            )
            latents = torch.cat([latents, pad], dim=1)

        sharded_tensor = rearrange(
            latents, "b (n s) d -> b n s d", n=sp_world_size
        ).contiguous()
        sharded_tensor = sharded_tensor[:, rank_in_sp_group, :, :]
        return sharded_tensor, True
```
**EN:** This block defines method `shard_latents_for_sp` on `ImagePipelineConfig`. It handles shard latents for sp logic. Key calls include `rearrange.contiguous`, `get_sp_world_size`, `get_sp_parallel_rank`, `torch.zeros`, and `torch.cat`. The implementation branches on conditions. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `ImagePipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 关键调用包括 `rearrange.contiguous`、`get_sp_world_size`、`get_sp_parallel_rank`、`torch.zeros` 和 `torch.cat`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 1077-1079: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents, batch=None):
        # For image latents [B, S_local, D], gather along sequence dim=1
        return self._gather_sp_tensor(latents, dim=1)
```
**EN:** This block defines method `gather_latents_for_sp` on `ImagePipelineConfig`. It handles gather latents for sp logic. Key calls include `self._gather_sp_tensor`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ImagePipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 关键调用包括 `self._gather_sp_tensor`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 1081-1093: `_unpad_and_unpack_latents` implementation / `_unpad_and_unpack_latents` 实现
```python
    def _unpad_and_unpack_latents(self, latents, batch):
        vae_scale_factor = self.vae_config.arch_config.vae_scale_factor
        channels = self.dit_config.arch_config.in_channels
        batch_size = latents.shape[0]

        height = 2 * (int(batch.height) // (vae_scale_factor * 2))
        width = 2 * (int(batch.width) // (vae_scale_factor * 2))

        latents = maybe_unpad_latents(latents, batch)

        latents = latents.view(batch_size, height // 2, width // 2, channels // 4, 2, 2)
        latents = latents.permute(0, 3, 1, 4, 2, 5)
        return latents, batch_size, channels, height, width
```
**EN:** This block defines method `_unpad_and_unpack_latents` on `ImagePipelineConfig`. It handles unpad and unpack latents logic. Key calls include `maybe_unpad_latents`, `latents.view`, `latents.permute`, and `int`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ImagePipelineConfig` 的方法 `_unpad_and_unpack_latents`。 它用于处理 unpad and unpack latents 相关逻辑。 关键调用包括 `maybe_unpad_latents`、`latents.view`、`latents.permute` 和 `int`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 1097-1103: `SpatialImagePipelineConfig` class overview / `SpatialImagePipelineConfig` 类概览
```python
class SpatialImagePipelineConfig(ImagePipelineConfig):
    """Base config for spatial image pipelines (e.g. GLM-Image) with 4D latents (B, C, H', W').

    Overrides shard_latents_for_sp / gather_latents_for_sp to shard along the height dimension
    so that each SP rank gets (B, C, H'_local, W') instead of using the token-style (B, S, C) path.
    """
```
**EN:** This block defines class `SpatialImagePipelineConfig`. Base config for spatial image pipelines (e.g. GLM-Image) with 4D latents (B, C, H', W'). It inherits from `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `SpatialImagePipelineConfig`。 它用于封装 spatial image pipeline config 相关行为。 它继承自 `ImagePipelineConfig`。

### Lines 1104-1128: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        # 4D latents (B, C, H', W') -> shard along H' (dim=2); otherwise fall back to base (B, S, C)
        sp_world_size = get_sp_world_size()
        if sp_world_size <= 1:
            return latents, False
        if latents.dim() != 4:
            return super().shard_latents_for_sp(batch, latents)

        # (B, C, H', W')
        _, _, h_lat, w_lat = latents.shape
        if h_lat % sp_world_size != 0:
            pad_len = sp_world_size - (h_lat % sp_world_size)
            pad = torch.zeros(
                (latents.shape[0], latents.shape[1], pad_len, latents.shape[3]),
                dtype=latents.dtype,
                device=latents.device,
            )
            latents = torch.cat([latents, pad], dim=2)
            h_lat = latents.shape[2]
        rank_in_sp_group = get_sp_parallel_rank()
        chunk_size = h_lat // sp_world_size
        h0 = rank_in_sp_group * chunk_size
        h1 = h0 + chunk_size
        sharded = latents[:, :, h0:h1, :].contiguous()
        return sharded, True
```
**EN:** This block defines method `shard_latents_for_sp` on `SpatialImagePipelineConfig`. It handles shard latents for sp logic. Key calls include `get_sp_world_size`, `get_sp_parallel_rank`, `latents.contiguous`, `latents.dim`, and `super.shard_latents_for_sp`. The implementation branches on conditions. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `SpatialImagePipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 关键调用包括 `get_sp_world_size`、`get_sp_parallel_rank`、`latents.contiguous`、`latents.dim` 和 `super.shard_latents_for_sp`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 1130-1136: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents, batch=None):
        if get_sp_world_size() <= 1:
            return latents
        if latents.dim() != 4:
            return super().gather_latents_for_sp(latents, batch=batch)
        # Gather along dim=2 (H') to match shard_latents_for_sp
        return self._gather_sp_tensor(latents, dim=2)
```
**EN:** This block defines method `gather_latents_for_sp` on `SpatialImagePipelineConfig`. It handles gather latents for sp logic. Key calls include `self._gather_sp_tensor`, `get_sp_world_size`, `latents.dim`, `super.gather_latents_for_sp`, and `super`. The implementation branches on conditions. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `SpatialImagePipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 关键调用包括 `self._gather_sp_tensor`、`get_sp_world_size`、`latents.dim`、`super.gather_latents_for_sp` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 1140-1144: `SlidingTileAttnConfig` class overview / `SlidingTileAttnConfig` 类概览
```python
class SlidingTileAttnConfig(PipelineConfig):
    """Configuration for sliding tile attention."""

    # Override any BaseConfig defaults as needed
    # Add sliding tile specific parameters
```
**EN:** This block defines class `SlidingTileAttnConfig`. Configuration for sliding tile attention. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `SlidingTileAttnConfig`。 它用于封装 sliding tile attn config 相关行为。 它继承自 `PipelineConfig`。

### Lines 1145-1154: supporting statements / 辅助语句
```python
    window_size: int = 16
    stride: int = 8

    # You can provide custom defaults for inherited fields
    height: int = 576
    width: int = 1024

    # Additional configuration specific to sliding tile attention
    pad_to_square: bool = False
    use_overlap_optimization: bool = True
```
**EN:** This block gathers supporting statements inside `SlidingTileAttnConfig`. It updates names such as `window_size`, `stride`, `height`, `width`, `pad_to_square`, and `use_overlap_optimization`.
**CN:** 该代码块汇集了位于 `SlidingTileAttnConfig` 内部的辅助语句。 它会更新 `window_size`、`stride`、`height`、`width`、`pad_to_square` 和 `use_overlap_optimization` 等名称。

### Lines 1157-1161: `parse_int_list` implementation / `parse_int_list` 实现
```python
def parse_int_list(value: str) -> list[int]:
    """Parse a comma-separated string of integers into a list."""
    if not value:
        return []
    return [int(x.strip()) for x in value.split(",")]
```
**EN:** This block defines function `parse_int_list`. Parse a comma-separated string of integers into a list. Key calls include `int`, `x.strip`, and `value.split`. The implementation branches on conditions. Parameters such as `value` drive the behavior in this section.
**CN:** 该代码块定义了函数 `parse_int_list`。 它用于解析int list。 关键调用包括 `int`、`x.strip` 和 `value.split`。 实现中包含条件分支。 本段逻辑主要由 `value` 等参数驱动。

## Key Concepts / 关键概念
- `ModelTaskType`: Primary class that encapsulates model task type behavior. / 核心类，用于封装 model task type 相关行为。
- `STA_Mode`: STA (Sliding Tile Attention) modes. / 核心类，用于封装 sta mode 相关行为。
- `postprocess_text`: Top-level function that handles postprocess text logic. / 顶层函数，用于处理 postprocess text 相关逻辑。
- `TextConditioningOutput`: Text embeddings and masks aligned to postprocessed sequence length. / 核心类，用于封装 text conditioning output 相关行为。
- `pad_text_embeddings_with_mask`: Pad variable-length text embeddings and return the valid-token mask. / 顶层函数，用于处理 pad text embeddings with mask 相关逻辑。
- `shard_rotary_emb_for_sp`: Shard rotary embeddings [S, D] along sequence for SP. / 顶层函数，用于处理 shard rotary emb for sp 相关逻辑。
- `maybe_unpad_latents`: Top-level function that handles maybe unpad latents logic. / 顶层函数，用于处理 maybe unpad latents 相关逻辑。
- `PipelineConfig`: The base configuration class for a generation pipeline. / 核心类，用于封装 pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `math`, `os`, `collections.abc`, `dataclasses`, `enum`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `PIL`, `torch`, `einops`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.t5`, `sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.configs.utils`, `sglang.multimodal_gen.runtime.distributed.cfg_policy`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.models.vision_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 1161
