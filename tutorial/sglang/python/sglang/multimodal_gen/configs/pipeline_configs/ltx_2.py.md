# ltx_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/ltx_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `LTX2PipelineConfig`, `LTX2I2VPipelineConfig`, and `pack_text_embeds`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `LTX2PipelineConfig`、`LTX2I2VPipelineConfig` 和 `pack_text_embeds` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module setup and imports / 模块初始化与导入
```python
import dataclasses
from dataclasses import field
from typing import Callable, Optional

import torch

from sglang.multimodal_gen.configs.models.dits.ltx_2 import LTX2Config
from sglang.multimodal_gen.configs.models.encoders import (
    BaseEncoderOutput,
    EncoderConfig,
)
from sglang.multimodal_gen.configs.models.encoders.gemma_3 import Gemma3Config
from sglang.multimodal_gen.configs.models.vaes.ltx_audio import LTXAudioVAEConfig
from sglang.multimodal_gen.configs.models.vaes.ltx_video import LTXVideoVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
from sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config import (
    ModelDeploymentConfig,
)
from sglang.multimodal_gen.runtime.distributed import (
    get_sp_parallel_rank,
    get_sp_world_size,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models.dits.ltx_2`, `sglang.multimodal_gen.configs.models.encoders`, and `sglang.multimodal_gen.configs.models.encoders.gemma_3`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models.dits.ltx_2`、`sglang.multimodal_gen.configs.models.encoders` 和 `sglang.multimodal_gen.configs.models.encoders.gemma_3`。这些依赖为后续实现提供所需符号。

### Lines 28-95: `pack_text_embeds` implementation / `pack_text_embeds` 实现
```python
def pack_text_embeds(
    text_hidden_states: torch.Tensor,
    sequence_lengths: torch.Tensor,
    padding_side: str = "left",
    scale_factor: int = 8,
    eps: float = 1e-6,
) -> torch.Tensor:
    """
    Packs and normalizes text encoder hidden states, respecting padding. Normalization is performed per-batch and
    per-layer in a masked fashion (only over non-padded positions).

    Args:
        text_hidden_states (`torch.Tensor` of shape `(batch_size, seq_len, hidden_dim, num_layers)`):
            Per-layer hidden_states from a text encoder (e.g. `Gemma3ForConditionalGeneration`).
        sequence_lengths (`torch.Tensor of shape `(batch_size,)`):
            The number of valid (non-padded) tokens for each batch instance.
        device: (`str` or `torch.device`, *optional*):
            torch device to place the resulting embeddings on
        padding_side: (`str`, *optional*, defaults to `"left"`):
            Whether the text tokenizer performs padding on the `"left"` or `"right"`.
        scale_factor (`int`, *optional*, defaults to `8`):
            Scaling factor to multiply the normalized hidden states by.
        eps (`float`, *optional*, defaults to `1e-6`):
            A small positive value for numerical stability when performing normalization.

    Returns:
        `torch.Tensor` of shape `(batch_size, seq_len, hidden_dim * num_layers)`:
            Normed and flattened text encoder hidden states.
    """
    batch_size, seq_len, hidden_dim, num_layers = text_hidden_states.shape
    original_dtype = text_hidden_states.dtype
    device = text_hidden_states.device

    # Create padding mask
    token_indices = torch.arange(seq_len, device=device).unsqueeze(0)
    if padding_side == "right":
        mask = token_indices < sequence_lengths[:, None]
    elif padding_side == "left":
        start_indices = seq_len - sequence_lengths[:, None]
        mask = token_indices >= start_indices
    else:
        raise ValueError(f"padding_side must be 'left' or 'right', got {padding_side}")
    mask = mask[:, :, None, None]  # [batch_size, seq_len, 1, 1]

    masked_text_hidden_states = text_hidden_states.masked_fill(~mask, 0.0)
    num_valid_positions = (sequence_lengths * hidden_dim).view(batch_size, 1, 1, 1)
    masked_mean = masked_text_hidden_states.sum(dim=(1, 2), keepdim=True) / (
        num_valid_positions + eps
    )

    x_min = text_hidden_states.masked_fill(~mask, float("inf")).amin(
        dim=(1, 2), keepdim=True
    )
    x_max = text_hidden_states.masked_fill(~mask, float("-inf")).amax(
        dim=(1, 2), keepdim=True
    )

    normalized_hidden_states = (text_hidden_states - masked_mean) / (
        x_max - x_min + eps
    )
    normalized_hidden_states = normalized_hidden_states * scale_factor

    normalized_hidden_states = normalized_hidden_states.flatten(2)
    mask_flat = mask.squeeze(-1).expand(-1, -1, hidden_dim * num_layers)
    normalized_hidden_states = normalized_hidden_states.masked_fill(~mask_flat, 0.0)
    normalized_hidden_states = normalized_hidden_states.to(dtype=original_dtype)

    return normalized_hidden_states
```
**EN:** This block defines function `pack_text_embeds`. Packs and normalizes text encoder hidden states, respecting padding. Normalization is performed per-batch and per-layer in a masked fashion (only over non-padded positions). Key calls include `torch.arange.unsqueeze`, `text_hidden_states.masked_fill`, `view`, `text_hidden_states.masked_fill.amin`, and `text_hidden_states.masked_fill.amax`. The implementation branches on conditions. Parameters such as `text_hidden_states`, `sequence_lengths`, `padding_side`, `scale_factor`, and `eps` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pack_text_embeds`。 它用于打包text embeds。 关键调用包括 `torch.arange.unsqueeze`、`text_hidden_states.masked_fill`、`view`、`text_hidden_states.masked_fill.amin` 和 `text_hidden_states.masked_fill.amax`。 实现中包含条件分支。 本段逻辑主要由 `text_hidden_states`、`sequence_lengths`、`padding_side`、`scale_factor` 和 `eps` 等参数驱动。

### Lines 98-117: `pack_text_embeds_v2` implementation / `pack_text_embeds_v2` 实现
```python
def pack_text_embeds_v2(
    text_hidden_states: torch.Tensor,
    attention_mask: torch.Tensor,
    eps: float = 1e-6,
) -> torch.Tensor:
    """
    LTX-2.3 feature extractor pre-processing.

    Upstream `FeatureExtractorV2` applies per-token RMS normalization on each
    Gemma layer and then flattens `[hidden_dim, num_layers]` into the channel
    dimension, zeroing out padded positions afterwards.
    """

    variance = torch.mean(text_hidden_states**2, dim=2, keepdim=True)
    normalized_hidden_states = text_hidden_states * torch.rsqrt(variance + eps)
    normalized_hidden_states = normalized_hidden_states.flatten(2)
    mask = attention_mask.bool().unsqueeze(-1)
    return torch.where(
        mask, normalized_hidden_states, torch.zeros_like(normalized_hidden_states)
    )
```
**EN:** This block defines function `pack_text_embeds_v2`. LTX-2.3 feature extractor pre-processing. Upstream `FeatureExtractorV2` applies per-token RMS normalization on each Gemma layer and then flattens `[hidden_dim, num_layers]` into the channel dimension, zeroing out padded positions afterwards. Key calls include `torch.mean`, `normalized_hidden_states.flatten`, `attention_mask.bool.unsqueeze`, `torch.where`, and `torch.rsqrt`. Parameters such as `text_hidden_states`, `attention_mask`, and `eps` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pack_text_embeds_v2`。 它用于打包text embeds v2。 关键调用包括 `torch.mean`、`normalized_hidden_states.flatten`、`attention_mask.bool.unsqueeze`、`torch.where` 和 `torch.rsqrt`。 本段逻辑主要由 `text_hidden_states`、`attention_mask` 和 `eps` 等参数驱动。

### Lines 120-121: `is_ltx23_native_variant` implementation / `is_ltx23_native_variant` 实现
```python
def is_ltx23_native_variant(arch_config: object) -> bool:
    return str(getattr(arch_config, "ltx_variant", "ltx_2")) == "ltx_2_3"
```
**EN:** This block defines function `is_ltx23_native_variant`. It handles is ltx23 native variant logic. Key calls include `str`, and `getattr`. Parameters such as `arch_config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `is_ltx23_native_variant`。 它用于处理 is ltx23 native variant 相关逻辑。 关键调用包括 `str` 和 `getattr`。 本段逻辑主要由 `arch_config` 等参数驱动。

### Lines 124-139: `sync_ltx23_runtime_vae_markers` implementation / `sync_ltx23_runtime_vae_markers` 实现
```python
def sync_ltx23_runtime_vae_markers(
    arch_config: object,
    loaded_vae_config: object | None,
) -> None:
    if loaded_vae_config is None:
        return
    source = getattr(loaded_vae_config, "arch_config", loaded_vae_config)
    for key in (
        "ltx_variant",
        "condition_encoder_subdir",
        "video_decoder_variant",
        "video_decoder_config",
    ):
        value = getattr(source, key, None)
        if value is not None:
            setattr(arch_config, key, value)
```
**EN:** This block defines function `sync_ltx23_runtime_vae_markers`. It handles sync ltx23 runtime vae markers logic. Key calls include `getattr`, and `setattr`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `arch_config`, and `loaded_vae_config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sync_ltx23_runtime_vae_markers`。 它用于处理 sync ltx23 runtime vae markers 相关逻辑。 关键调用包括 `getattr` 和 `setattr`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `arch_config` 和 `loaded_vae_config` 等参数驱动。

### Lines 142-162: `_gemma_postprocess_func` implementation / `_gemma_postprocess_func` 实现
```python
def _gemma_postprocess_func(
    outputs: BaseEncoderOutput,
    text_inputs: dict,
    pipeline_config: Optional["LTX2PipelineConfig"] = None,
) -> torch.Tensor:
    # LTX-2 requires all hidden states concatenated for the connector
    if hasattr(outputs, "hidden_states") and outputs.hidden_states is not None:
        hidden_states = torch.stack(outputs.hidden_states, dim=-1)
        attention_mask = text_inputs["attention_mask"]
        if (
            pipeline_config is not None
            and pipeline_config.dit_config.arch_config.caption_proj_before_connector
        ):
            return pack_text_embeds_v2(hidden_states, attention_mask)

        sequence_lengths = attention_mask.sum(dim=-1)
        return pack_text_embeds(hidden_states, sequence_lengths, padding_side="left")
    else:
        raise AttributeError(
            "Unsupported text encoder output: expected `hidden_states`."
        )
```
**EN:** This block defines function `_gemma_postprocess_func`. It handles gemma postprocess func logic. Key calls include `hasattr`, `torch.stack`, `attention_mask.sum`, `pack_text_embeds`, and `AttributeError`. The implementation branches on conditions. Parameters such as `outputs`, `text_inputs`, and `pipeline_config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_gemma_postprocess_func`。 它用于处理 gemma postprocess func 相关逻辑。 关键调用包括 `hasattr`、`torch.stack`、`attention_mask.sum`、`pack_text_embeds` 和 `AttributeError`。 实现中包含条件分支。 本段逻辑主要由 `outputs`、`text_inputs` 和 `pipeline_config` 等参数驱动。

### Lines 166-168: `LTX2PipelineConfig` class overview / `LTX2PipelineConfig` 类概览
```python
class LTX2PipelineConfig(PipelineConfig):
    """Configuration for LTX-Video pipeline."""
```
**EN:** This block defines class `LTX2PipelineConfig`. Configuration for LTX-Video pipeline. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `LTX2PipelineConfig`。 它用于封装 ltx2 pipeline config 相关行为。 它继承自 `PipelineConfig`。

### Lines 169-184: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.TI2V
    skip_input_image_preprocess: bool = True
    generator_device: str = "cpu"
    dit_config: LTX2Config = field(default_factory=LTX2Config)

    # Model architecture
    in_channels: int = 128
    out_channels: int = 128
    patch_size: int = 1
    patch_size_t: int = 1

    # Audio VAE configuration
    vae_config: LTXVideoVAEConfig = field(default_factory=LTXVideoVAEConfig)
    vae_precision: str = "bf16"
    audio_vae_config: LTXAudioVAEConfig = field(default_factory=LTXAudioVAEConfig)
    audio_vae_precision: str = "bf16"
```
**EN:** This block gathers supporting statements inside `LTX2PipelineConfig`. It updates names such as `task_type`, `skip_input_image_preprocess`, `generator_device`, `dit_config`, `in_channels`, and `out_channels`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `LTX2PipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`skip_input_image_preprocess`、`generator_device`、`dit_config`、`in_channels` 和 `out_channels` 等名称。 代码会与 `field` 协同工作。

### Lines 186-188: `vae_scale_factor` implementation / `vae_scale_factor` 实现
```python
    @property
    def vae_scale_factor(self):
        return self.vae_config.arch_config.spatial_compression_ratio
```
**EN:** This block defines method `vae_scale_factor` on `LTX2PipelineConfig`. It handles vae scale factor logic.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `vae_scale_factor`。 它用于处理 vae scale factor 相关逻辑。

### Lines 190-192: `vae_temporal_compression` implementation / `vae_temporal_compression` 实现
```python
    @property
    def vae_temporal_compression(self):
        return self.vae_config.arch_config.temporal_compression_ratio
```
**EN:** This block defines method `vae_temporal_compression` on `LTX2PipelineConfig`. It handles vae temporal compression logic.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `vae_temporal_compression`。 它用于处理 vae temporal compression 相关逻辑。

### Lines 194-198: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig(
            auto_disable_component_offload_min_available_memory_gb=70,
            auto_disable_component_offload_components=("dit",),
        )
```
**EN:** This block defines method `get_model_deployment_config` on `LTX2PipelineConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 200-204: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        """Return unpacked latent shape [B, C, F, H, W]."""
        height = batch.height // self.vae_scale_factor
        width = batch.width // self.vae_scale_factor
        return (batch_size, self.in_channels, num_frames, height, width)
```
**EN:** This block defines method `prepare_latent_shape` on `LTX2PipelineConfig`. Return unpacked latent shape [B, C, F, H, W]. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 206-228: `prepare_audio_latent_shape` implementation / `prepare_audio_latent_shape` 实现
```python
    def prepare_audio_latent_shape(self, batch, batch_size, num_frames):
        # Adapted from diffusers pipeline prepare_audio_latents
        duration_s = num_frames / batch.fps

        sample_rate = self.audio_vae_config.arch_config.sample_rate
        hop_length = self.audio_vae_config.arch_config.mel_hop_length
        temporal_compression = (
            self.audio_vae_config.arch_config.temporal_compression_ratio
        )

        latents_per_second = (
            float(sample_rate) / float(hop_length) / float(temporal_compression)
        )
        latent_length = round(duration_s * latents_per_second)

        num_mel_bins = self.audio_vae_config.arch_config.mel_bins
        mel_compression_ratio = self.audio_vae_config.arch_config.mel_compression_ratio
        latent_mel_bins = num_mel_bins // mel_compression_ratio

        # Default to 8
        num_channels_latents = self.audio_vae_config.arch_config.latent_channels

        return (batch_size, num_channels_latents, latent_length, latent_mel_bins)
```
**EN:** This block defines method `prepare_audio_latent_shape` on `LTX2PipelineConfig`. It prepares audio latent shape. Key calls include `round`, and `float`. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `prepare_audio_latent_shape`。 它用于准备audio latent shape。 关键调用包括 `round` 和 `float`。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 233-244: supporting statements / 辅助语句
```python
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Gemma3Config(),)
    )
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))
    text_encoder_extra_args: list[dict] = field(default_factory=lambda: [{}])

    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (None,)
    )
    postprocess_text_funcs: tuple[
        Callable[[BaseEncoderOutput, dict], torch.Tensor], ...
    ] = field(default_factory=lambda: (_gemma_postprocess_func,))
```
**EN:** This block gathers supporting statements inside `LTX2PipelineConfig`. It updates names such as `text_encoder_configs`, `text_encoder_precisions`, `text_encoder_extra_args`, `preprocess_text_funcs`, and `postprocess_text_funcs`. The code collaborates with `field`, and `Gemma3Config`.
**CN:** 该代码块汇集了位于 `LTX2PipelineConfig` 内部的辅助语句。 它会更新 `text_encoder_configs`、`text_encoder_precisions`、`text_encoder_extra_args`、`preprocess_text_funcs` 和 `postprocess_text_funcs` 等名称。 代码会与 `field` 和 `Gemma3Config` 协同工作。

### Lines 246-252: `prepare_sigmas` implementation / `prepare_sigmas` 实现
```python
    def prepare_sigmas(self, sigmas, num_inference_steps):
        if sigmas is None:
            steps = int(num_inference_steps)
            if steps <= 0:
                raise ValueError(f"num_inference_steps must be positive, got {steps}")
            return [1.0 - i / steps for i in range(steps)]
        return sigmas
```
**EN:** This block defines method `prepare_sigmas` on `LTX2PipelineConfig`. It prepares sigmas. Key calls include `int`, `ValueError`, and `range`. The implementation branches on conditions. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `prepare_sigmas`。 它用于准备sigmas。 关键调用包括 `int`、`ValueError` 和 `range`。 实现中包含条件分支。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 254-278: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompt: list[str], tokenizer, tok_kwargs) -> dict:
        # Adapted from diffusers_pipeline.py _get_gemma_prompt_embeds
        # But we only need tokenization here, the embedding happens in TextEncodingStage
        # Official LTX Gemma tokenizer trims surrounding whitespace before
        # tokenization.
        prompt = [text.strip() for text in prompt]

        # Gemma expects left padding for chat-style prompts
        tokenizer.padding_side = "left"
        if tokenizer.pad_token is None:
            tokenizer.pad_token = tokenizer.eos_token

        max_sequence_length = tok_kwargs.get(
            "max_length", 1024
        )  # Default from diffusers pipeline

        text_inputs = tokenizer(
            prompt,
            padding="max_length",
            max_length=max_sequence_length,
            truncation=True,
            add_special_tokens=True,
            return_tensors="pt",
        )
        return text_inputs
```
**EN:** This block defines method `tokenize_prompt` on `LTX2PipelineConfig`. It handles tokenize prompt logic. Key calls include `tok_kwargs.get`, `tokenizer`, and `text.strip`. The implementation branches on conditions. Parameters such as `prompt`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tok_kwargs.get`、`tokenizer` 和 `text.strip`。 实现中包含条件分支。 本段逻辑主要由 `prompt`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 280-304: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        # If already packed (3D shape [B, seq, C]), skip packing
        if latents.dim() == 3:
            return latents

        # Unpacked latents of shape are [B, C, F, H, W] are patched into tokens of shape [B, C, F // p_t, p_t, H // p, p, W // p, p].
        # The patch dimensions are then permuted and collapsed into the channel dimension of shape:
        # [B, F // p_t * H // p * W // p, C * p_t * p * p] (an ndim=3 tensor).
        # dim=0 is the batch size, dim=1 is the effective video sequence length, dim=2 is the effective number of input features
        batch_size, num_channels, num_frames, height, width = latents.shape
        post_patch_num_frames = num_frames // self.patch_size_t
        post_patch_height = height // self.patch_size
        post_patch_width = width // self.patch_size
        latents = latents.reshape(
            batch_size,
            -1,
            post_patch_num_frames,
            self.patch_size_t,
            post_patch_height,
            self.patch_size,
            post_patch_width,
            self.patch_size,
        )
        latents = latents.permute(0, 2, 4, 6, 1, 3, 5, 7).flatten(4, 7).flatten(1, 3)
        return latents
```
**EN:** This block defines method `maybe_pack_latents` on `LTX2PipelineConfig`. It handles maybe pack latents logic. Key calls include `latents.reshape`, `latents.permute.flatten.flatten`, `latents.dim`, `latents.permute.flatten`, and `latents.permute`. The implementation branches on conditions. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 关键调用包括 `latents.reshape`、`latents.permute.flatten.flatten`、`latents.dim`、`latents.permute.flatten` 和 `latents.permute`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 306-357: `_infer_video_latent_frames_and_tokens_per_frame` implementation / `_infer_video_latent_frames_and_tokens_per_frame` 实现
```python
    def _infer_video_latent_frames_and_tokens_per_frame(
        self, batch, seq_len: int
    ) -> tuple[int, int]:
        """Infer latent-frame count and tokens-per-frame for packed token latents [B, S, D].

        Notes:
        - This assumes `patch_size_t == 1` (no temporal patching).
        - Tokens are ordered as (frame, height, width) after packing.
        """
        if int(self.patch_size_t) != 1:
            raise ValueError(
                "LTX-2 SP time-sharding for packed token latents currently requires "
                f"{self.patch_size_t=}. (Expected 1)"
            )
        if int(seq_len) <= 0:
            raise ValueError(f"Expected {seq_len=} > 0 for packed token latents.")
        if int(self.vae_scale_factor) <= 0:
            raise ValueError(f"Invalid {self.vae_scale_factor=}. Must be > 0.")
        if int(self.patch_size) <= 0:
            raise ValueError(f"Invalid {self.patch_size=}. Must be > 0.")

        latent_height = int(batch.height) // int(self.vae_scale_factor)
        latent_width = int(batch.width) // int(self.vae_scale_factor)
        if latent_height <= 0 or latent_width <= 0:
            raise ValueError(
                "Invalid latent H/W computed from batch.height/width: "
                f"{batch.height=} {batch.width=} {self.vae_scale_factor=}"
            )
        if (latent_height % int(self.patch_size)) != 0 or (
            latent_width % int(self.patch_size)
        ) != 0:
            raise ValueError(
                "Invalid spatial patching for packed token latents. Expected latent H/W "
                "to be divisible by patch_size, got "
                f"{latent_height=} {latent_width=} {self.patch_size=}."
            )

        post_patch_h = latent_height // int(self.patch_size)
        post_patch_w = latent_width // int(self.patch_size)
        tokens_per_frame = int(post_patch_h) * int(post_patch_w)
        if tokens_per_frame <= 0:
            raise ValueError(
                f"Invalid tokens_per_frame={tokens_per_frame} from "
                f"{latent_height=} {latent_width=} {self.patch_size=}"
            )
        if int(seq_len) % int(tokens_per_frame) != 0:
            raise ValueError(
                f"LTX-2 token latents seq_len={seq_len} is not divisible by "
                f"tokens_per_frame={tokens_per_frame}. Cannot time-shard for SP."
            )
        latent_num_frames = int(seq_len) // int(tokens_per_frame)
        return int(latent_num_frames), int(tokens_per_frame)
```
**EN:** This block defines method `_infer_video_latent_frames_and_tokens_per_frame` on `LTX2PipelineConfig`. Infer latent-frame count and tokens-per-frame for packed token latents [B, S, D]. Notes: - This assumes `patch_size_t == 1` (no temporal patching). Key calls include `int`, and `ValueError`. The implementation branches on conditions. Parameters such as `batch`, and `seq_len` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_infer_video_latent_frames_and_tokens_per_frame`。 它用于推断video latent frames and tokens per frame。 关键调用包括 `int` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `seq_len` 等参数驱动。

### Lines 359-409: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        """Shard LTX-2 packed token latents across SP ranks by latent time (frame) dimension."""
        sp_world_size = get_sp_world_size()
        if sp_world_size <= 1:
            return latents, False

        # Default behavior for 5D latents.
        if isinstance(latents, torch.Tensor) and latents.ndim == 5:
            return super().shard_latents_for_sp(batch, latents)

        # LTX-2 packed token latents [B, S, D]
        if not (isinstance(latents, torch.Tensor) and latents.ndim == 3):
            return latents, False

        sp_rank = get_sp_parallel_rank()
        seq_len = int(latents.shape[1])
        latent_frames, tokens_per_frame = (
            self._infer_video_latent_frames_and_tokens_per_frame(batch, seq_len)
        )
        orig_latent_frames = int(latent_frames)

        # Pad whole frames so `latent_frames` is divisible by `sp_world_size`.
        pad_frames = (sp_world_size - (latent_frames % sp_world_size)) % sp_world_size
        if pad_frames:
            pad_tokens = int(pad_frames) * int(tokens_per_frame)
            pad = torch.zeros(
                (latents.shape[0], pad_tokens, latents.shape[2]),
                device=latents.device,
                dtype=latents.dtype,
            )
            latents = torch.cat([latents, pad], dim=1)
            latent_frames = int(latent_frames) + int(pad_frames)

        local_frames = int(latent_frames) // int(sp_world_size)
        start_frame = int(sp_rank) * int(local_frames)
        valid_local_frames = max(
            min(int(orig_latent_frames) - int(start_frame), int(local_frames)), 0
        )
        start = int(start_frame) * int(tokens_per_frame)
        end = int(start) + int(local_frames) * int(tokens_per_frame)
        latents = latents[:, start:end, :]

        # Store SP metadata for denoising (TI2V gating) and model-side RoPE shift.
        batch.sp_video_latent_num_frames = int(local_frames)
        batch.sp_video_start_frame = int(start_frame)
        batch.sp_video_tokens_per_frame = int(tokens_per_frame)
        batch.sp_video_valid_token_count = int(valid_local_frames) * int(
            tokens_per_frame
        )

        return latents, True
```
**EN:** This block defines method `shard_latents_for_sp` on `LTX2PipelineConfig`. Shard LTX-2 packed token latents across SP ranks by latent time (frame) dimension. Key calls include `get_sp_world_size`, `get_sp_parallel_rank`, `int`, `self._infer_video_latent_frames_and_tokens_per_frame`, and `max`. The implementation branches on conditions. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 关键调用包括 `get_sp_world_size`、`get_sp_parallel_rank`、`int`、`self._infer_video_latent_frames_and_tokens_per_frame` 和 `max`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 411-417: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents, batch=None):
        """Gather latents after SP. For packed token latents [B, S_local, D], gather on dim=1."""
        if get_sp_world_size() <= 1:
            return latents
        if isinstance(latents, torch.Tensor) and latents.ndim == 3:
            return self._gather_sp_tensor(latents, dim=1)
        return super().gather_latents_for_sp(latents, batch=batch)
```
**EN:** This block defines method `gather_latents_for_sp` on `LTX2PipelineConfig`. Gather latents after SP. For packed token latents [B, S_local, D], gather on dim=1. Key calls include `super.gather_latents_for_sp`, `get_sp_world_size`, `isinstance`, `self._gather_sp_tensor`, and `super`. The implementation branches on conditions. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 关键调用包括 `super.gather_latents_for_sp`、`get_sp_world_size`、`isinstance`、`self._gather_sp_tensor` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 419-455: `shard_audio_latents_for_sp` implementation / `shard_audio_latents_for_sp` 实现
```python
    def shard_audio_latents_for_sp(self, batch, audio_latents):
        sp_world_size = get_sp_world_size()
        if sp_world_size <= 1:
            return audio_latents, False
        if not (isinstance(audio_latents, torch.Tensor) and audio_latents.ndim == 3):
            return audio_latents, False

        sp_rank = get_sp_parallel_rank()
        seq_len = int(audio_latents.shape[1])
        batch.sp_audio_orig_num_frames = int(seq_len)

        pad_frames = (sp_world_size - (seq_len % sp_world_size)) % sp_world_size
        if pad_frames:
            pad = torch.zeros(
                (audio_latents.shape[0], pad_frames, audio_latents.shape[2]),
                device=audio_latents.device,
                dtype=audio_latents.dtype,
            )
            audio_latents = torch.cat([audio_latents, pad], dim=1)
            seq_len += int(pad_frames)

        local_frames = seq_len // sp_world_size
        start_frame = sp_rank * local_frames
        end_frame = start_frame + local_frames
        valid_local_frames = max(
            min(
                int(batch.sp_audio_orig_num_frames) - int(start_frame),
                int(local_frames),
            ),
            0,
        )
        audio_latents = audio_latents[:, start_frame:end_frame, :]

        batch.sp_audio_latent_num_frames = int(local_frames)
        batch.sp_audio_start_frame = int(start_frame)
        batch.sp_audio_valid_token_count = int(valid_local_frames)
        return audio_latents, True
```
**EN:** This block defines method `shard_audio_latents_for_sp` on `LTX2PipelineConfig`. It handles shard audio latents for sp logic. Key calls include `get_sp_world_size`, `get_sp_parallel_rank`, `int`, `max`, and `torch.zeros`. The implementation branches on conditions. Parameters such as `batch`, and `audio_latents` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `shard_audio_latents_for_sp`。 它用于处理 shard audio latents for sp 相关逻辑。 关键调用包括 `get_sp_world_size`、`get_sp_parallel_rank`、`int`、`max` 和 `torch.zeros`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `audio_latents` 等参数驱动。

### Lines 457-462: `can_shard_audio_latents_for_sp` implementation / `can_shard_audio_latents_for_sp` 实现
```python
    def can_shard_audio_latents_for_sp(self, audio_latents) -> bool:
        return (
            get_sp_world_size() > 1
            and isinstance(audio_latents, torch.Tensor)
            and audio_latents.ndim == 3
        )
```
**EN:** This block defines method `can_shard_audio_latents_for_sp` on `LTX2PipelineConfig`. It handles can shard audio latents for sp logic. Key calls include `isinstance`, and `get_sp_world_size`. Parameters such as `audio_latents` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `can_shard_audio_latents_for_sp`。 它用于处理 can shard audio latents for sp 相关逻辑。 关键调用包括 `isinstance` 和 `get_sp_world_size`。 本段逻辑主要由 `audio_latents` 等参数驱动。

### Lines 464-479: `gather_audio_latents_for_sp` implementation / `gather_audio_latents_for_sp` 实现
```python
    def gather_audio_latents_for_sp(self, audio_latents, batch):
        """Gather packed audio latents after SP and trim any pad-only tail tokens."""
        if get_sp_world_size() <= 1:
            return audio_latents
        if not (isinstance(audio_latents, torch.Tensor) and audio_latents.ndim == 3):
            return audio_latents

        audio_latents = self._gather_sp_tensor(
            audio_latents,
            dim=1,
        )
        return self._trim_sp_gather_padding(
            audio_latents,
            orig_len=getattr(batch, "sp_audio_orig_num_frames", None),
            dim=1,
        )
```
**EN:** This block defines method `gather_audio_latents_for_sp` on `LTX2PipelineConfig`. Gather packed audio latents after SP and trim any pad-only tail tokens. Key calls include `self._gather_sp_tensor`, `self._trim_sp_gather_padding`, `get_sp_world_size`, `isinstance`, and `getattr`. The implementation branches on conditions. Parameters such as `audio_latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `gather_audio_latents_for_sp`。 它用于处理 gather audio latents for sp 相关逻辑。 关键调用包括 `self._gather_sp_tensor`、`self._trim_sp_gather_padding`、`get_sp_world_size`、`isinstance` 和 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `audio_latents` 和 `batch` 等参数驱动。

### Lines 481-501: `prepare_video_rope_coords_for_sp` implementation / `prepare_video_rope_coords_for_sp` 实现
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
        if not batch.did_sp_shard_latents:
            return None
        return model.rope.prepare_video_coords(
            batch_size=int(latent_model_input.shape[0]),
            num_frames=num_frames,
            height=height,
            width=width,
            device=latent_model_input.device,
            fps=batch.fps,
            start_frame=int(batch.sp_video_start_frame),
        )
```
**EN:** This block defines method `prepare_video_rope_coords_for_sp` on `LTX2PipelineConfig`. It prepares video rope coords for sp. Key calls include `model.rope.prepare_video_coords`, and `int`. The implementation branches on conditions. Parameters such as `model`, `batch`, and `latent_model_input` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `prepare_video_rope_coords_for_sp`。 它用于准备video rope coords for sp。 关键调用包括 `model.rope.prepare_video_coords` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `model`、`batch` 和 `latent_model_input` 等参数驱动。

### Lines 503-518: `prepare_audio_rope_coords_for_sp` implementation / `prepare_audio_rope_coords_for_sp` 实现
```python
    def prepare_audio_rope_coords_for_sp(
        self,
        model,
        batch,
        audio_latent_model_input,
        *,
        num_frames,
    ):
        if not batch.did_sp_shard_audio_latents:
            return None
        return model.audio_rope.prepare_audio_coords(
            batch_size=int(audio_latent_model_input.shape[0]),
            num_frames=num_frames,
            device=audio_latent_model_input.device,
            start_frame=int(batch.sp_audio_start_frame),
        )
```
**EN:** This block defines method `prepare_audio_rope_coords_for_sp` on `LTX2PipelineConfig`. It prepares audio rope coords for sp. Key calls include `model.audio_rope.prepare_audio_coords`, and `int`. The implementation branches on conditions. Parameters such as `model`, `batch`, and `audio_latent_model_input` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `prepare_audio_rope_coords_for_sp`。 它用于准备audio rope coords for sp。 关键调用包括 `model.audio_rope.prepare_audio_coords` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `model`、`batch` 和 `audio_latent_model_input` 等参数驱动。

### Lines 520-532: `maybe_pack_audio_latents` implementation / `maybe_pack_audio_latents` 实现
```python
    def maybe_pack_audio_latents(self, latents, batch_size, batch):
        # If already packed (3D shape [B, T, C*F]), skip packing
        if latents.dim() == 3:
            return latents

        # Audio latents shape: [B, C, L, M], where L is the latent audio length and M is the number of mel bins
        # We need to pack them if patch_size/patch_size_t are defined for audio (not standard DiT patch size)

        # So for LTX-2 (unless we change patch sizes), we just do:
        latents = latents.transpose(1, 2).flatten(
            2, 3
        )  # [B, C, L, M] --> [B, L, C * M]
        return latents
```
**EN:** This block defines method `maybe_pack_audio_latents` on `LTX2PipelineConfig`. It handles maybe pack audio latents logic. Key calls include `latents.transpose.flatten`, `latents.dim`, and `latents.transpose`. The implementation branches on conditions. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `maybe_pack_audio_latents`。 它用于处理 maybe pack audio latents 相关逻辑。 关键调用包括 `latents.transpose.flatten`、`latents.dim` 和 `latents.transpose`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 534-540: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        # LTX-2 returns multiple prompt embed tensors (video/audio contexts).
        return (
            batch.prompt_embeds[0]
            if isinstance(batch.prompt_embeds, list)
            else batch.prompt_embeds
        )
```
**EN:** This block defines method `get_pos_prompt_embeds` on `LTX2PipelineConfig`. It retrieves pos prompt embeds. Key calls include `isinstance`. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 关键调用包括 `isinstance`。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 542-547: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return (
            batch.negative_prompt_embeds[0]
            if isinstance(batch.negative_prompt_embeds, list)
            else batch.negative_prompt_embeds
        )
```
**EN:** This block defines method `get_neg_prompt_embeds` on `LTX2PipelineConfig`. It retrieves neg prompt embeds. Key calls include `isinstance`. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 关键调用包括 `isinstance`。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 549-579: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        latents_mean = getattr(vae, "latents_mean", None)
        latents_std = getattr(vae, "latents_std", None)

        scaling_factor = (
            getattr(getattr(vae, "config", None), "scaling_factor", None)
            or getattr(vae, "scaling_factor", None)
            or getattr(self.vae_config.arch_config, "scaling_factor", None)
            or 1.0
        )
        if isinstance(scaling_factor, (int, float)) and float(scaling_factor) == 0.0:
            scaling_factor = 1.0

        if isinstance(latents_mean, torch.Tensor) and isinstance(
            latents_std, torch.Tensor
        ):
            latents_mean = latents_mean.to(device=device, dtype=dtype).view(
                1, -1, 1, 1, 1
            )
            latents_std = latents_std.to(device=device, dtype=dtype).view(
                1, -1, 1, 1, 1
            )
            sf = torch.tensor(float(scaling_factor), device=device, dtype=dtype).view(
                1, 1, 1, 1, 1
            )
            return sf / latents_std, latents_mean

        sf = torch.tensor(float(scaling_factor), device=device, dtype=dtype).view(
            1, 1, 1, 1, 1
        )
        return sf, None
```
**EN:** This block defines method `get_decode_scale_and_shift` on `LTX2PipelineConfig`. It retrieves decode scale and shift. Key calls include `getattr`, `torch.tensor.view`, `isinstance`, `latents_mean.to.view`, and `latents_std.to.view`. The implementation branches on conditions. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `getattr`、`torch.tensor.view`、`isinstance`、`latents_mean.to.view` 和 `latents_std.to.view`。 实现中包含条件分支。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 581-610: `_unpack_latents` implementation / `_unpack_latents` 实现
```python
    @staticmethod
    def _unpack_latents(
        latents: torch.Tensor,
        num_frames: int,
        height: int,
        width: int,
        patch_size: int = 1,
        patch_size_t: int = 1,
    ) -> torch.Tensor:
        # Packed latents of shape [B, S, D] (S is the effective video sequence length, D is the effective feature dimensions)
        # are unpacked and reshaped into a video tensor of shape [B, C, F, H, W]. This is the inverse operation of
        # what happens in the `_pack_latents` method.
        batch_size = latents.size(0)
        latents = latents.reshape(
            batch_size,
            num_frames,
            height,
            width,
            -1,
            patch_size_t,
            patch_size,
            patch_size,
        )
        latents = (
            latents.permute(0, 4, 1, 5, 2, 6, 3, 7)
            .flatten(6, 7)
            .flatten(4, 5)
            .flatten(2, 3)
        )
        return latents
```
**EN:** This block defines method `_unpack_latents` on `LTX2PipelineConfig`. It unpacks latents. Key calls include `latents.size`, `latents.reshape`, `latents.permute.flatten.flatten.flatten`, `latents.permute.flatten.flatten`, and `latents.permute.flatten`. Parameters such as `latents`, `num_frames`, `height`, `width`, and `patch_size` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_unpack_latents`。 它用于解包latents。 关键调用包括 `latents.size`、`latents.reshape`、`latents.permute.flatten.flatten.flatten`、`latents.permute.flatten.flatten` 和 `latents.permute.flatten`。 本段逻辑主要由 `latents`、`num_frames`、`height`、`width` 和 `patch_size` 等参数驱动。

### Lines 612-625: `_denormalize_latents` implementation / `_denormalize_latents` 实现
```python
    @staticmethod
    def _denormalize_latents(
        latents: torch.Tensor,
        latents_mean: torch.Tensor,
        latents_std: torch.Tensor,
        scaling_factor: float = 1.0,
    ) -> torch.Tensor:
        # Denormalize latents across the channel dimension [B, C, F, H, W]
        latents_mean = latents_mean.view(1, -1, 1, 1, 1).to(
            latents.device, latents.dtype
        )
        latents_std = latents_std.view(1, -1, 1, 1, 1).to(latents.device, latents.dtype)
        latents = latents * latents_std / scaling_factor + latents_mean
        return latents
```
**EN:** This block defines method `_denormalize_latents` on `LTX2PipelineConfig`. It handles denormalize latents logic. Key calls include `latents_mean.view.to`, `latents_std.view.to`, `latents_mean.view`, and `latents_std.view`. Parameters such as `latents`, `latents_mean`, `latents_std`, and `scaling_factor` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_denormalize_latents`。 它用于处理 denormalize latents 相关逻辑。 关键调用包括 `latents_mean.view.to`、`latents_std.view.to`、`latents_mean.view` 和 `latents_std.view`。 本段逻辑主要由 `latents`、`latents_mean`、`latents_std` 和 `scaling_factor` 等参数驱动。

### Lines 627-633: `_denormalize_audio_latents` implementation / `_denormalize_audio_latents` 实现
```python
    @staticmethod
    def _denormalize_audio_latents(
        latents: torch.Tensor, latents_mean: torch.Tensor, latents_std: torch.Tensor
    ):
        latents_mean = latents_mean.to(latents.device, latents.dtype)
        latents_std = latents_std.to(latents.device, latents.dtype)
        return (latents * latents_std) + latents_mean
```
**EN:** This block defines method `_denormalize_audio_latents` on `LTX2PipelineConfig`. It handles denormalize audio latents logic. Key calls include `latents_mean.to`, and `latents_std.to`. Parameters such as `latents`, `latents_mean`, and `latents_std` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_denormalize_audio_latents`。 它用于处理 denormalize audio latents 相关逻辑。 关键调用包括 `latents_mean.to` 和 `latents_std.to`。 本段逻辑主要由 `latents`、`latents_mean` 和 `latents_std` 等参数驱动。

### Lines 635-654: `_unpack_audio_latents` implementation / `_unpack_audio_latents` 实现
```python
    @staticmethod
    def _unpack_audio_latents(
        latents: torch.Tensor,
        latent_length: int,
        num_mel_bins: int,
        patch_size: int | None = None,
        patch_size_t: int | None = None,
    ) -> torch.Tensor:
        # Unpacks an audio patch sequence of shape [B, S, D] into a latent spectrogram tensor of shape [B, C, L, M],
        # where L is the latent audio length and M is the number of mel bins.
        if patch_size is not None and patch_size_t is not None:
            batch_size = latents.size(0)
            latents = latents.reshape(
                batch_size, latent_length, num_mel_bins, -1, patch_size_t, patch_size
            )
            latents = latents.permute(0, 3, 1, 4, 2, 5).flatten(4, 5).flatten(2, 3)
        else:
            # Assume [B, S, D] = [B, L, C * M], which implies that patch_size = M and patch_size_t = 1.
            latents = latents.unflatten(2, (-1, num_mel_bins)).transpose(1, 2)
        return latents
```
**EN:** This block defines method `_unpack_audio_latents` on `LTX2PipelineConfig`. It unpacks audio latents. Key calls include `latents.size`, `latents.reshape`, `latents.permute.flatten.flatten`, `latents.unflatten.transpose`, and `latents.permute.flatten`. The implementation branches on conditions. Parameters such as `latents`, `latent_length`, `num_mel_bins`, `patch_size`, and `patch_size_t` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_unpack_audio_latents`。 它用于解包audio latents。 关键调用包括 `latents.size`、`latents.reshape`、`latents.permute.flatten.flatten`、`latents.unflatten.transpose` 和 `latents.permute.flatten`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`latent_length`、`num_mel_bins`、`patch_size` 和 `patch_size_t` 等参数驱动。

### Lines 656-705: `_unpad_and_unpack_latents` implementation / `_unpad_and_unpack_latents` 实现
```python
    def _unpad_and_unpack_latents(self, latents, audio_latents, batch, vae, audio_vae):
        # Calculate latent dimensions
        # Assuming batch has height, width, num_frames
        height = batch.height
        width = batch.width
        num_frames = batch.num_frames

        # Get compression ratios
        # Default LTX-2 values if not present in config
        vae_spatial_compression_ratio = getattr(
            self.vae_config.arch_config, "spatial_compression_ratio", 32
        )
        vae_temporal_compression_ratio = getattr(
            self.vae_config.arch_config, "temporal_compression_ratio", 8
        )

        latent_height = height // vae_spatial_compression_ratio
        latent_width = width // vae_spatial_compression_ratio
        latent_num_frames = (num_frames - 1) // vae_temporal_compression_ratio + 1

        latents = self._unpack_latents(
            latents,
            latent_num_frames,
            latent_height,
            latent_width,
            self.patch_size,
            self.patch_size_t,
        )

        sample_rate = self.audio_vae_config.arch_config.sample_rate
        hop_length = self.audio_vae_config.arch_config.mel_hop_length
        temporal_compression = (
            self.audio_vae_config.arch_config.temporal_compression_ratio
        )
        duration_s = num_frames / batch.fps

        latents_per_second = (
            float(sample_rate) / float(hop_length) / float(temporal_compression)
        )
        audio_num_frames = round(duration_s * latents_per_second)

        num_mel_bins = self.audio_vae_config.arch_config.mel_bins
        mel_compression_ratio = self.audio_vae_config.arch_config.mel_compression_ratio
        latent_mel_bins = num_mel_bins // mel_compression_ratio

        audio_latents = self._unpack_audio_latents(
            audio_latents, audio_num_frames, num_mel_bins=latent_mel_bins
        )

        return latents, audio_latents
```
**EN:** This block defines method `_unpad_and_unpack_latents` on `LTX2PipelineConfig`. It handles unpad and unpack latents logic. Key calls include `getattr`, `self._unpack_latents`, `round`, `self._unpack_audio_latents`, and `float`. Parameters such as `latents`, `audio_latents`, `batch`, `vae`, and `audio_vae` drive the behavior in this section.
**CN:** 该代码块定义了 `LTX2PipelineConfig` 的方法 `_unpad_and_unpack_latents`。 它用于处理 unpad and unpack latents 相关逻辑。 关键调用包括 `getattr`、`self._unpack_latents`、`round`、`self._unpack_audio_latents` 和 `float`。 本段逻辑主要由 `latents`、`audio_latents`、`batch`、`vae` 和 `audio_vae` 等参数驱动。

### Lines 709-709: `LTX2I2VPipelineConfig` class overview / `LTX2I2VPipelineConfig` 类概览
```python
class LTX2I2VPipelineConfig(LTX2PipelineConfig):
```
**EN:** This block defines class `LTX2I2VPipelineConfig`. It encapsulates ltx2 i2 vpipeline config behavior. It inherits from `LTX2PipelineConfig`.
**CN:** 该代码块定义了类 `LTX2I2VPipelineConfig`。 它用于封装 ltx2 i2 vpipeline config 相关行为。 它继承自 `LTX2PipelineConfig`。

### Lines 710-710: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.TI2V
```
**EN:** This block gathers supporting statements inside `LTX2I2VPipelineConfig`. It updates names such as `task_type`.
**CN:** 该代码块汇集了位于 `LTX2I2VPipelineConfig` 内部的辅助语句。 它会更新 `task_type` 等名称。

## Key Concepts / 关键概念
- `pack_text_embeds`: Packs and normalizes text encoder hidden states, respecting padding. / 顶层函数，用于打包text embeds。
- `pack_text_embeds_v2`: LTX-2.3 feature extractor pre-processing. / 顶层函数，用于打包text embeds v2。
- `is_ltx23_native_variant`: Top-level function that handles is ltx23 native variant logic. / 顶层函数，用于处理 is ltx23 native variant 相关逻辑。
- `sync_ltx23_runtime_vae_markers`: Top-level function that handles sync ltx23 runtime vae markers logic. / 顶层函数，用于处理 sync ltx23 runtime vae markers 相关逻辑。
- `_gemma_postprocess_func`: Top-level function that handles gemma postprocess func logic. / 顶层函数，用于处理 gemma postprocess func 相关逻辑。
- `LTX2PipelineConfig`: Configuration for LTX-Video pipeline. / 核心类，用于封装 ltx2 pipeline config 相关行为。
- `LTX2I2VPipelineConfig`: Primary class that encapsulates ltx2 i2 vpipeline config behavior. / 核心类，用于封装 ltx2 i2 vpipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.ltx_2`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.gemma_3`, `sglang.multimodal_gen.configs.models.vaes.ltx_audio`, `sglang.multimodal_gen.configs.models.vaes.ltx_video`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config`, `sglang.multimodal_gen.runtime.distributed`

- **Total lines / 总行数**: 710
