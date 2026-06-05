# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `FluxPipelineConfig`, `Flux2PipelineConfig`, and `Flux2KleinPipelineConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `FluxPipelineConfig`、`Flux2PipelineConfig` 和 `Flux2KleinPipelineConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: module setup and imports / 模块初始化与导入
```python
import math
from dataclasses import dataclass, field
from typing import Callable, List, Optional

import PIL
import torch
from diffusers.image_processor import VaeImageProcessor

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.flux import FluxConfig
from sglang.multimodal_gen.configs.models.encoders import (
    BaseEncoderOutput,
    CLIPTextConfig,
    Flux2MistralTextConfig,
    T5Config,
    build_flux2_text_messages,
)
from sglang.multimodal_gen.configs.models.encoders.qwen3 import Qwen3TextConfig
from sglang.multimodal_gen.configs.models.vaes.flux import Flux2VAEConfig, FluxVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ImagePipelineConfig,
    ModelTaskType,
    shard_rotary_emb_for_sp,
)
from sglang.multimodal_gen.configs.pipeline_configs.hunyuan import (
    clip_postprocess_text,
)
from sglang.multimodal_gen.configs.pipeline_configs.qwen_image import _pack_latents
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
```
**EN:** This block establishes the module context and imports `math`, `dataclasses`, `typing`, `PIL`, `torch`, and `diffusers.image_processor`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `math`、`dataclasses`、`typing`、`PIL`、`torch` 和 `diffusers.image_processor`。这些依赖为后续实现提供所需符号。

### Lines 32-33: `t5_postprocess_text` implementation / `t5_postprocess_text` 实现
```python
def t5_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    return outputs.last_hidden_state
```
**EN:** This block defines function `t5_postprocess_text`. It handles t5 postprocess text logic. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `t5_postprocess_text`。 它用于处理 t5 postprocess text 相关逻辑。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 37-39: `FluxPipelineConfig` class overview / `FluxPipelineConfig` 类概览
```python
class FluxPipelineConfig(ImagePipelineConfig):
    """Configuration for the FLUX pipeline."""
```
**EN:** This block defines class `FluxPipelineConfig`. Configuration for the FLUX pipeline. It inherits from `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `FluxPipelineConfig`。 它用于封装 flux pipeline config 相关行为。 它继承自 `ImagePipelineConfig`。

### Lines 40-89: supporting statements / 辅助语句
```python
    embedded_cfg_scale: float = 3.5

    task_type: ModelTaskType = ModelTaskType.T2I

    vae_tiling: bool = False

    vae_sp: bool = False

    dit_config: DiTConfig = field(default_factory=FluxConfig)
    # VAE
    vae_config: VAEConfig = field(default_factory=FluxVAEConfig)
    vae_precision: str = "bf16"

    enable_autocast: bool = False

    # Text encoding stage
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (CLIPTextConfig(), T5Config())
    )

    text_encoder_precisions: tuple[str, ...] = field(
        default_factory=lambda: ("bf16", "bf16")
    )

    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (None, None),
    )

    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (clip_postprocess_text, t5_postprocess_text)
    )

    text_encoder_extra_args: list[dict] = field(
        default_factory=lambda: [
            dict(
                max_length=77,
                padding="max_length",
                truncation=True,
                return_overflowing_tokens=False,
                return_length=False,
            ),
            dict(
                max_length=512,
                padding="max_length",
                truncation=True,
                return_overflowing_tokens=False,
                return_length=False,
            ),
        ]
    )
```
**EN:** This block gathers supporting statements inside `FluxPipelineConfig`. It updates names such as `embedded_cfg_scale`, `task_type`, `vae_tiling`, `vae_sp`, `dit_config`, and `vae_config`. The code collaborates with `field`, `CLIPTextConfig`, `T5Config`, and `dict`.
**CN:** 该代码块汇集了位于 `FluxPipelineConfig` 内部的辅助语句。 它会更新 `embedded_cfg_scale`、`task_type`、`vae_tiling`、`vae_sp`、`dit_config` 和 `vae_config` 等名称。 代码会与 `field`、`CLIPTextConfig`、`T5Config` 和 `dict` 协同工作。

### Lines 91-92: `is_flux_v1` implementation / `is_flux_v1` 实现
```python
    def is_flux_v1(self) -> bool:
        return True
```
**EN:** This block defines method `is_flux_v1` on `FluxPipelineConfig`. It handles is flux v1 logic.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `is_flux_v1`。 它用于处理 is flux v1 相关逻辑。

### Lines 94-96: `get_text_encoder_attention_mask` implementation / `get_text_encoder_attention_mask` 实现
```python
    def get_text_encoder_attention_mask(self, text_inputs, encoder_index):
        # Flux v1 does not use attention masks for text encoders.
        return None
```
**EN:** This block defines method `get_text_encoder_attention_mask` on `FluxPipelineConfig`. It retrieves text encoder attention mask. Parameters such as `text_inputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `get_text_encoder_attention_mask`。 它用于获取text encoder attention mask。 本段逻辑主要由 `text_inputs` 和 `encoder_index` 等参数驱动。

### Lines 98-114: `build_text_conditioning_mask` implementation / `build_text_conditioning_mask` 实现
```python
    def build_text_conditioning_mask(
        self,
        text_inputs: dict,
        text_encoder_attention_mask: "torch.Tensor | None",
        prompt_embeds: "torch.Tensor",
        encoder_index: int,
    ) -> "torch.Tensor":
        """Use all-valid fixed-length masks for Flux v1 text embeddings."""
        if prompt_embeds.ndim < 2:
            raise ValueError(
                "prompt_embeds must have shape [batch, seq, ...] or [seq, ...]"
            )
        if prompt_embeds.ndim == 2:
            shape = (1, prompt_embeds.shape[0])
        else:
            shape = prompt_embeds.shape[:2]
        return torch.ones(shape, dtype=torch.bool)
```
**EN:** This block defines method `build_text_conditioning_mask` on `FluxPipelineConfig`. Use all-valid fixed-length masks for Flux v1 text embeddings. Key calls include `torch.ones`, and `ValueError`. The implementation branches on conditions. Parameters such as `text_inputs`, `text_encoder_attention_mask`, `prompt_embeds`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `build_text_conditioning_mask`。 它用于构建text conditioning mask。 关键调用包括 `torch.ones` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `text_inputs`、`text_encoder_attention_mask`、`prompt_embeds` 和 `encoder_index` 等参数驱动。

### Lines 116-120: `seq_lens_from_text_conditioning_mask` implementation / `seq_lens_from_text_conditioning_mask` 实现
```python
    @staticmethod
    def seq_lens_from_text_conditioning_mask(mask: "torch.Tensor") -> list[int]:
        if mask.ndim != 2:
            raise ValueError("text conditioning mask must have shape [batch, seq]")
        return [int(mask.shape[1])] * int(mask.shape[0])
```
**EN:** This block defines method `seq_lens_from_text_conditioning_mask` on `FluxPipelineConfig`. It handles seq lens from text conditioning mask logic. Key calls include `ValueError`, and `int`. The implementation branches on conditions. Parameters such as `mask` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `seq_lens_from_text_conditioning_mask`。 它用于处理 seq lens from text conditioning mask 相关逻辑。 关键调用包括 `ValueError` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `mask` 等参数驱动。

### Lines 122-123: `get_text_encoder_pooler_output` implementation / `get_text_encoder_pooler_output` 实现
```python
    def get_text_encoder_pooler_output(self, outputs, encoder_index):
        return outputs.pooler_output
```
**EN:** This block defines method `get_text_encoder_pooler_output` on `FluxPipelineConfig`. It retrieves text encoder pooler output. Parameters such as `outputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `get_text_encoder_pooler_output`。 它用于获取text encoder pooler output。 本段逻辑主要由 `outputs` 和 `encoder_index` 等参数驱动。

### Lines 125-126: `prepare_sigmas` implementation / `prepare_sigmas` 实现
```python
    def prepare_sigmas(self, sigmas, num_inference_steps):
        return self._prepare_sigmas(sigmas, num_inference_steps)
```
**EN:** This block defines method `prepare_sigmas` on `FluxPipelineConfig`. It prepares sigmas. Key calls include `self._prepare_sigmas`. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `prepare_sigmas`。 它用于准备sigmas。 关键调用包括 `self._prepare_sigmas`。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 128-135: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        height = 2 * (
            batch.height // (self.vae_config.arch_config.vae_scale_factor * 2)
        )
        width = 2 * (batch.width // (self.vae_config.arch_config.vae_scale_factor * 2))
        num_channels_latents = self.dit_config.arch_config.in_channels // 4
        shape = (batch_size, num_channels_latents, height, width)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `FluxPipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 137-144: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        height = 2 * (
            batch.height // (self.vae_config.arch_config.vae_scale_factor * 2)
        )
        width = 2 * (batch.width // (self.vae_config.arch_config.vae_scale_factor * 2))
        num_channels_latents = self.dit_config.arch_config.in_channels // 4
        # pack latents
        return _pack_latents(latents, batch_size, num_channels_latents, height, width)
```
**EN:** This block defines method `maybe_pack_latents` on `FluxPipelineConfig`. It handles maybe pack latents logic. Key calls include `_pack_latents`. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 关键调用包括 `_pack_latents`。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 146-147: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        return batch.prompt_embeds[1]
```
**EN:** This block defines method `get_pos_prompt_embeds` on `FluxPipelineConfig`. It retrieves pos prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 149-150: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return batch.negative_prompt_embeds[1]
```
**EN:** This block defines method `get_neg_prompt_embeds` on `FluxPipelineConfig`. It retrieves neg prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 152-172: `_prepare_latent_image_ids` implementation / `_prepare_latent_image_ids` 实现
```python
    def _prepare_latent_image_ids(self, original_height, original_width, device):
        vae_scale_factor = self.vae_config.arch_config.vae_scale_factor
        height = int(original_height) // (vae_scale_factor * 2)
        width = int(original_width) // (vae_scale_factor * 2)
        latent_image_ids = torch.zeros(height, width, 3, device=device)
        latent_image_ids[..., 1] = (
            latent_image_ids[..., 1] + torch.arange(height, device=device)[:, None]
        )
        latent_image_ids[..., 2] = (
            latent_image_ids[..., 2] + torch.arange(width, device=device)[None, :]
        )

        latent_image_id_height, latent_image_id_width, latent_image_id_channels = (
            latent_image_ids.shape
        )

        latent_image_ids = latent_image_ids.reshape(
            latent_image_id_height * latent_image_id_width, latent_image_id_channels
        )

        return latent_image_ids
```
**EN:** This block defines method `_prepare_latent_image_ids` on `FluxPipelineConfig`. It prepares latent image ids. Key calls include `torch.zeros`, `latent_image_ids.reshape`, `int`, and `torch.arange`. Parameters such as `original_height`, `original_width`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `_prepare_latent_image_ids`。 它用于准备latent image ids。 关键调用包括 `torch.zeros`、`latent_image_ids.reshape`、`int` 和 `torch.arange`。 本段逻辑主要由 `original_height`、`original_width` 和 `device` 等参数驱动。

### Lines 174-189: `_validate_fixed_text_seq_lens` implementation / `_validate_fixed_text_seq_lens` 实现
```python
    @staticmethod
    def _validate_fixed_text_seq_lens(prompt_embeds, txt_seq_lens):
        if prompt_embeds.ndim < 3:
            raise ValueError(
                "Flux text conditioning expects prompt_embeds with shape [batch, seq, dim]"
            )
        batch_size, seq_len = prompt_embeds.shape[:2]
        if len(txt_seq_lens) != batch_size:
            raise ValueError(
                f"Flux text sequence lengths have {len(txt_seq_lens)} entries, expected {batch_size}."
            )
        if any(int(seq_len_i) != seq_len for seq_len_i in txt_seq_lens):
            raise ValueError(
                "Flux currently requires fixed-length text conditioning; "
                f"got seq_lens={txt_seq_lens}, expected all {seq_len}."
            )
```
**EN:** This block defines method `_validate_fixed_text_seq_lens` on `FluxPipelineConfig`. It validates fixed text seq lens. Key calls include `any`, `ValueError`, `len`, and `int`. The implementation branches on conditions. Parameters such as `prompt_embeds`, and `txt_seq_lens` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `_validate_fixed_text_seq_lens`。 它用于校验fixed text seq lens。 关键调用包括 `any`、`ValueError`、`len` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `prompt_embeds` 和 `txt_seq_lens` 等参数驱动。

### Lines 191-211: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    def get_freqs_cis(
        self, prompt_embeds, width, height, device, rotary_emb, batch, txt_seq_lens
    ):
        self._validate_fixed_text_seq_lens(prompt_embeds, txt_seq_lens)
        txt_ids = torch.zeros(prompt_embeds.shape[1], 3, device=device)
        img_ids = self._prepare_latent_image_ids(
            original_height=height,
            original_width=width,
            device=device,
        )

        # NOTE(mick): prepare it here, to avoid unnecessary computations
        img_cos, img_sin = rotary_emb.forward(img_ids)
        img_cos = shard_rotary_emb_for_sp(img_cos)
        img_sin = shard_rotary_emb_for_sp(img_sin)

        txt_cos, txt_sin = rotary_emb.forward(txt_ids)

        cos = torch.cat([txt_cos, img_cos], dim=0).to(device=device)
        sin = torch.cat([txt_sin, img_sin], dim=0).to(device=device)
        return cos, sin
```
**EN:** This block defines method `get_freqs_cis` on `FluxPipelineConfig`. It retrieves freqs cis. Key calls include `self._validate_fixed_text_seq_lens`, `torch.zeros`, `self._prepare_latent_image_ids`, `rotary_emb.forward`, and `shard_rotary_emb_for_sp`. Parameters such as `prompt_embeds`, `width`, `height`, `device`, and `rotary_emb` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `self._validate_fixed_text_seq_lens`、`torch.zeros`、`self._prepare_latent_image_ids`、`rotary_emb.forward` 和 `shard_rotary_emb_for_sp`。 本段逻辑主要由 `prompt_embeds`、`width`、`height`、`device` 和 `rotary_emb` 等参数驱动。

### Lines 213-223: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        # unpack latents for flux
        (
            latents,
            batch_size,
            channels,
            height,
            width,
        ) = self._unpad_and_unpack_latents(latents, batch)
        latents = latents.reshape(batch_size, channels // (2 * 2), height, width)
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `FluxPipelineConfig`. It post-processes denoising loop. Key calls include `self._unpad_and_unpack_latents`, and `latents.reshape`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `self._unpad_and_unpack_latents` 和 `latents.reshape`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 225-254: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        """Build Flux positive-conditioning kwargs from encoded text state.

        Flux v1 uses encoder index 1 (the T5 encoder) as the token stream that
        is concatenated with image tokens for rotary position embeddings. The
        text encoding stage stores per-request sequence lengths in
        batch.prompt_seq_lens; read them here instead of inferring from padded
        embeddings so grouped multi-output requests preserve their explicit
        text-conditioning contract.
        """
        txt_seq_lens = self.require_text_seq_lens(
            batch,
            1,
            negative=False,
            expected_batch_size=batch.prompt_embeds[1].shape[0],
        )
        return {
            "freqs_cis": self.get_freqs_cis(
                batch.prompt_embeds[1],
                batch.width,
                batch.height,
                device,
                rotary_emb,
                batch,
                txt_seq_lens,
            ),
            "pooled_projections": (
                batch.pooled_embeds[0] if batch.pooled_embeds else None
            ),
        }
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `FluxPipelineConfig`. Build Flux positive-conditioning kwargs from encoded text state. Flux v1 uses encoder index 1 (the T5 encoder) as the token stream that is concatenated with image tokens for rotary position embeddings. Key calls include `self.require_text_seq_lens`, and `self.get_freqs_cis`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self.require_text_seq_lens` 和 `self.get_freqs_cis`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 256-277: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        """Build Flux negative-conditioning kwargs using T5 sequence lengths."""
        txt_seq_lens = self.require_text_seq_lens(
            batch,
            1,
            negative=True,
            expected_batch_size=batch.negative_prompt_embeds[1].shape[0],
        )
        return {
            "freqs_cis": self.get_freqs_cis(
                batch.negative_prompt_embeds[1],
                batch.width,
                batch.height,
                device,
                rotary_emb,
                batch,
                txt_seq_lens,
            ),
            "pooled_projections": (
                batch.neg_pooled_embeds[0] if batch.neg_pooled_embeds else None
            ),
        }
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `FluxPipelineConfig`. Build Flux negative-conditioning kwargs using T5 sequence lengths. Key calls include `self.require_text_seq_lens`, and `self.get_freqs_cis`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `FluxPipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self.require_text_seq_lens` 和 `self.get_freqs_cis`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 280-308: `_prepare_latent_ids` implementation / `_prepare_latent_ids` 实现
```python
def _prepare_latent_ids(
    latents: torch.Tensor,  # (B, C, H, W)
):
    r"""
    Generates 4D position coordinates (T, H, W, L) for latent tensors.

    Args:
        latents (torch.Tensor):
            Latent tensor of shape (B, C, H, W)

    Returns:
        torch.Tensor:
            Position IDs tensor of shape (B, H*W, 4) All batches share the same coordinate structure: T=0,
            H=[0..H-1], W=[0..W-1], L=0
    """

    batch_size, _, height, width = latents.shape

    t = torch.arange(1)  # [0] - time dimension
    h = torch.arange(height)
    w = torch.arange(width)
    layer = torch.arange(1)  # [0] - layer dimension

    # Create position IDs: (H*W, 4)
    latent_ids = torch.cartesian_prod(t, h, w, layer)

    # Expand to batch: (B, H*W, 4)
    latent_ids = latent_ids.unsqueeze(0).expand(batch_size, -1, -1)
    return latent_ids
```
**EN:** This block defines function `_prepare_latent_ids`. Generates 4D position coordinates (T, H, W, L) for latent tensors. Args: latents (torch.Tensor): Latent tensor of shape (B, C, H, W) Returns: torch.Tensor: Position IDs tensor of shape (B, H*W, 4) All batches share the same coordinate structure: T=0, H=[0..H-1], W=[0..W-1], L=0 Key calls include `torch.arange`, `torch.cartesian_prod`, `latent_ids.unsqueeze.expand`, and `latent_ids.unsqueeze`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_prepare_latent_ids`。 它用于准备latent ids。 关键调用包括 `torch.arange`、`torch.cartesian_prod`、`latent_ids.unsqueeze.expand` 和 `latent_ids.unsqueeze`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 311-337: `_unpack_latents_with_ids` implementation / `_unpack_latents_with_ids` 实现
```python
def _unpack_latents_with_ids(
    x: torch.Tensor, x_ids: torch.Tensor
) -> list[torch.Tensor]:
    """
    using position ids to scatter tokens into place
    """
    x_list = []
    x_ids = x_ids.to(device=x.device)
    for data, pos in zip(x, x_ids):
        _, ch = data.shape  # noqa: F841
        h_ids = pos[:, 1].to(torch.int64)
        w_ids = pos[:, 2].to(torch.int64)

        h = torch.max(h_ids) + 1
        w = torch.max(w_ids) + 1

        flat_ids = h_ids * w + w_ids

        out = torch.zeros((h * w, ch), device=data.device, dtype=data.dtype)
        out.scatter_(0, flat_ids.unsqueeze(1).expand(-1, ch), data)

        # reshape from (H * W, C) to (H, W, C) and permute to (C, H, W)

        out = out.view(h, w, ch).permute(2, 0, 1)
        x_list.append(out)

    return torch.stack(x_list, dim=0)
```
**EN:** This block defines function `_unpack_latents_with_ids`. using position ids to scatter tokens into place Key calls include `x_ids.to`, `zip`, `torch.stack`, `pos.to`, and `torch.zeros`. The implementation iterates over collections or steps. Parameters such as `x`, and `x_ids` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_unpack_latents_with_ids`。 它用于解包latents with ids。 关键调用包括 `x_ids.to`、`zip`、`torch.stack`、`pos.to` 和 `torch.zeros`。 实现中会遍历集合或步骤。 本段逻辑主要由 `x` 和 `x_ids` 等参数驱动。

### Lines 340-349: `_patchify_latents` implementation / `_patchify_latents` 实现
```python
def _patchify_latents(latents):
    batch_size, num_channels_latents, height, width = latents.shape
    latents = latents.view(
        batch_size, num_channels_latents, height // 2, 2, width // 2, 2
    )
    latents = latents.permute(0, 1, 3, 5, 2, 4)
    latents = latents.reshape(
        batch_size, num_channels_latents * 4, height // 2, width // 2
    )
    return latents
```
**EN:** This block defines function `_patchify_latents`. It handles patchify latents logic. Key calls include `latents.view`, `latents.permute`, and `latents.reshape`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patchify_latents`。 它用于处理 patchify latents 相关逻辑。 关键调用包括 `latents.view`、`latents.permute` 和 `latents.reshape`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 352-361: `_unpatchify_latents` implementation / `_unpatchify_latents` 实现
```python
def _unpatchify_latents(latents):
    batch_size, num_channels_latents, height, width = latents.shape
    latents = latents.reshape(
        batch_size, num_channels_latents // (2 * 2), 2, 2, height, width
    )
    latents = latents.permute(0, 1, 4, 2, 5, 3)
    latents = latents.reshape(
        batch_size, num_channels_latents // (2 * 2), height * 2, width * 2
    )
    return latents
```
**EN:** This block defines function `_unpatchify_latents`. It handles unpatchify latents logic. Key calls include `latents.reshape`, and `latents.permute`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_unpatchify_latents`。 它用于处理 unpatchify latents 相关逻辑。 关键调用包括 `latents.reshape` 和 `latents.permute`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 364-380: `_prepare_text_ids` implementation / `_prepare_text_ids` 实现
```python
def _prepare_text_ids(
    x: torch.Tensor,  # (B, L, D) or (L, D)
    t_coord: Optional[torch.Tensor] = None,
):
    B, L, _ = x.shape
    out_ids = []

    for i in range(B):
        t = torch.arange(1) if t_coord is None else t_coord[i]
        h = torch.arange(1)
        w = torch.arange(1)
        layer = torch.arange(L)

        coords = torch.cartesian_prod(t, h, w, layer)
        out_ids.append(coords)

    return torch.stack(out_ids)
```
**EN:** This block defines function `_prepare_text_ids`. It prepares text ids. Key calls include `range`, `torch.stack`, `torch.arange`, `torch.cartesian_prod`, and `out_ids.append`. The implementation iterates over collections or steps. Parameters such as `x`, and `t_coord` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_prepare_text_ids`。 它用于准备text ids。 关键调用包括 `range`、`torch.stack`、`torch.arange`、`torch.cartesian_prod` 和 `out_ids.append`。 实现中会遍历集合或步骤。 本段逻辑主要由 `x` 和 `t_coord` 等参数驱动。

### Lines 383-409: `_prepare_image_ids` implementation / `_prepare_image_ids` 实现
```python
def _prepare_image_ids(
    image_latents: List[torch.Tensor],  # [(1, C, H, W), (1, C, H, W), ...]
    scale: int = 10,
):
    if not isinstance(image_latents, list):
        raise ValueError(
            f"Expected `image_latents` to be a list, got {type(image_latents)}."
        )

    # create time offset for each reference image
    t_coords = [scale + scale * t for t in torch.arange(0, len(image_latents))]
    t_coords = [t.view(-1) for t in t_coords]

    image_latent_ids = []
    for x, t in zip(image_latents, t_coords):
        x = x.squeeze(0)
        _, height, width = x.shape

        x_ids = torch.cartesian_prod(
            t, torch.arange(height), torch.arange(width), torch.arange(1)
        )
        image_latent_ids.append(x_ids)

    image_latent_ids = torch.cat(image_latent_ids, dim=0)
    image_latent_ids = image_latent_ids.unsqueeze(0)

    return image_latent_ids
```
**EN:** This block defines function `_prepare_image_ids`. It prepares image ids. Key calls include `zip`, `torch.cat`, `image_latent_ids.unsqueeze`, `isinstance`, and `ValueError`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `image_latents`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_prepare_image_ids`。 它用于准备image ids。 关键调用包括 `zip`、`torch.cat`、`image_latent_ids.unsqueeze`、`isinstance` 和 `ValueError`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `image_latents` 和 `scale` 等参数驱动。

### Lines 412-421: `flux2_postprocess_text` implementation / `flux2_postprocess_text` 实现
```python
def flux2_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    hidden_states_layers: list[int] = [10, 20, 30]

    out = torch.stack([outputs.hidden_states[k] for k in hidden_states_layers], dim=1)
    batch_size, num_channels, seq_len, hidden_dim = out.shape
    prompt_embeds = out.permute(0, 2, 1, 3).reshape(
        batch_size, seq_len, num_channels * hidden_dim
    )

    return prompt_embeds
```
**EN:** This block defines function `flux2_postprocess_text`. It handles flux2 postprocess text logic. Key calls include `torch.stack`, `out.permute.reshape`, and `out.permute`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flux2_postprocess_text`。 它用于处理 flux2 postprocess text 相关逻辑。 关键调用包括 `torch.stack`、`out.permute.reshape` 和 `out.permute`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 424-435: `flux2_klein_postprocess_text` implementation / `flux2_klein_postprocess_text` 实现
```python
def flux2_klein_postprocess_text(
    outputs: BaseEncoderOutput, _text_inputs
) -> torch.Tensor:
    hidden_states_layers: list[int] = [9, 18, 27]

    out = torch.stack([outputs.hidden_states[k] for k in hidden_states_layers], dim=1)
    batch_size, num_channels, seq_len, hidden_dim = out.shape
    prompt_embeds = out.permute(0, 2, 1, 3).reshape(
        batch_size, seq_len, num_channels * hidden_dim
    )

    return prompt_embeds
```
**EN:** This block defines function `flux2_klein_postprocess_text`. It handles flux2 klein postprocess text logic. Key calls include `torch.stack`, `out.permute.reshape`, and `out.permute`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flux2_klein_postprocess_text`。 它用于处理 flux2 klein postprocess text 相关逻辑。 关键调用包括 `torch.stack`、`out.permute.reshape` 和 `out.permute`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 439-443: `flux2_pack_latents` implementation / `flux2_pack_latents` 实现
```python
def flux2_pack_latents(latents):
    batch_size, num_channels, height, width = latents.shape
    latents = latents.reshape(batch_size, num_channels, height * width).permute(0, 2, 1)

    return latents
```
**EN:** This block defines function `flux2_pack_latents`. It handles flux2 pack latents logic. Key calls include `latents.reshape.permute`, and `latents.reshape`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flux2_pack_latents`。 它用于处理 flux2 pack latents 相关逻辑。 关键调用包括 `latents.reshape.permute` 和 `latents.reshape`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 447-447: `Flux2PipelineConfig` class overview / `Flux2PipelineConfig` 类概览
```python
class Flux2PipelineConfig(FluxPipelineConfig):
```
**EN:** This block defines class `Flux2PipelineConfig`. It encapsulates flux2 pipeline config behavior. It inherits from `FluxPipelineConfig`.
**CN:** 该代码块定义了类 `Flux2PipelineConfig`。 它用于封装 flux2 pipeline config 相关行为。 它继承自 `FluxPipelineConfig`。

### Lines 448-477: supporting statements / 辅助语句
```python
    embedded_cfg_scale: float = 4.0

    task_type: ModelTaskType = ModelTaskType.TI2I

    vae_precision: str = "bf16"

    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))

    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Flux2MistralTextConfig(),)
    )
    preprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (None,),
    )

    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (flux2_postprocess_text,)
    )
    vae_config: VAEConfig = field(default_factory=Flux2VAEConfig)
    text_encoder_extra_args: list[dict] = field(
        default_factory=lambda: [
            dict(
                max_length=512,
                padding="max_length",
                truncation=True,
                return_overflowing_tokens=False,
                return_length=False,
            )
        ]
    )
```
**EN:** This block gathers supporting statements inside `Flux2PipelineConfig`. It updates names such as `embedded_cfg_scale`, `task_type`, `vae_precision`, `text_encoder_precisions`, `text_encoder_configs`, and `preprocess_text_funcs`. The code collaborates with `field`, `Flux2MistralTextConfig`, and `dict`.
**CN:** 该代码块汇集了位于 `Flux2PipelineConfig` 内部的辅助语句。 它会更新 `embedded_cfg_scale`、`task_type`、`vae_precision`、`text_encoder_precisions`、`text_encoder_configs` 和 `preprocess_text_funcs` 等名称。 代码会与 `field`、`Flux2MistralTextConfig` 和 `dict` 协同工作。

### Lines 479-480: `is_flux_v1` implementation / `is_flux_v1` 实现
```python
    def is_flux_v1(self) -> bool:
        return False
```
**EN:** This block defines method `is_flux_v1` on `Flux2PipelineConfig`. It handles is flux v1 logic.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `is_flux_v1`。 它用于处理 is flux v1 相关逻辑。

### Lines 482-484: `get_text_encoder_attention_mask` implementation / `get_text_encoder_attention_mask` 实现
```python
    def get_text_encoder_attention_mask(self, text_inputs, encoder_index):
        # Flux2 uses standard attention masks (unlike Flux v1).
        return text_inputs.get("attention_mask")
```
**EN:** This block defines method `get_text_encoder_attention_mask` on `Flux2PipelineConfig`. It retrieves text encoder attention mask. Key calls include `text_inputs.get`. Parameters such as `text_inputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_text_encoder_attention_mask`。 它用于获取text encoder attention mask。 关键调用包括 `text_inputs.get`。 本段逻辑主要由 `text_inputs` 和 `encoder_index` 等参数驱动。

### Lines 486-488: `get_text_encoder_pooler_output` implementation / `get_text_encoder_pooler_output` 实现
```python
    def get_text_encoder_pooler_output(self, outputs, encoder_index):
        # Flux2 does not use pooler output.
        return None
```
**EN:** This block defines method `get_text_encoder_pooler_output` on `Flux2PipelineConfig`. It retrieves text encoder pooler output. Parameters such as `outputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_text_encoder_pooler_output`。 它用于获取text encoder pooler output。 本段逻辑主要由 `outputs` 和 `encoder_index` 等参数驱动。

### Lines 490-496: `supports_dynamic_batching` implementation / `supports_dynamic_batching` 实现
```python
    def supports_dynamic_batching(self):
        """Allow batching for Flux2 text-only requests.

        Flux2 is a TI2I pipeline, so image-input requests are rejected by the
        scheduler's request-level batching checks.
        """
        return True
```
**EN:** This block defines method `supports_dynamic_batching` on `Flux2PipelineConfig`. Allow batching for Flux2 text-only requests. Flux2 is a TI2I pipeline, so image-input requests are rejected by the scheduler's request-level batching checks.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `supports_dynamic_batching`。 它用于处理 supports dynamic batching 相关逻辑。

### Lines 498-513: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompts: list[str], tokenizer, tok_kwargs) -> dict:
        messages = build_flux2_text_messages(prompts)
        effective_max_length = tok_kwargs.pop("max_length", 512)
        inputs = tokenizer.apply_chat_template(
            messages,
            add_generation_prompt=False,
            tokenize=True,
            return_dict=True,
            return_tensors="pt",
            padding="max_length",
            truncation=True,
            # 2048 from official github repo, 512 from diffusers
            max_length=effective_max_length,
        )

        return inputs
```
**EN:** This block defines method `tokenize_prompt` on `Flux2PipelineConfig`. It handles tokenize prompt logic. Key calls include `build_flux2_text_messages`, `tok_kwargs.pop`, and `tokenizer.apply_chat_template`. Parameters such as `prompts`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `build_flux2_text_messages`、`tok_kwargs.pop` 和 `tokenizer.apply_chat_template`。 本段逻辑主要由 `prompts`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 515-522: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        height = 2 * (
            batch.height // (self.vae_config.arch_config.vae_scale_factor * 2)
        )
        width = 2 * (batch.width // (self.vae_config.arch_config.vae_scale_factor * 2))
        num_channels_latents = self.dit_config.arch_config.in_channels
        shape = (batch_size, num_channels_latents, height // 2, width // 2)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `Flux2PipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 524-525: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        return batch.prompt_embeds[0]
```
**EN:** This block defines method `get_pos_prompt_embeds` on `Flux2PipelineConfig`. It retrieves pos prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 527-528: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return batch.negative_prompt_embeds[0]
```
**EN:** This block defines method `get_neg_prompt_embeds` on `Flux2PipelineConfig`. It retrieves neg prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 530-551: `calculate_condition_image_size` implementation / `calculate_condition_image_size` 实现
```python
    def calculate_condition_image_size(
        self, image, width, height
    ) -> Optional[tuple[int, int]]:
        vae_scale_factor = self.vae_config.arch_config.vae_scale_factor
        multiple_of = vae_scale_factor * 2

        target_area: int = 1024 * 1024
        if width is not None and height is not None:
            new_width, new_height = width, height
            if width * height > target_area:
                scale = math.sqrt(target_area / (width * height))
                new_width = int(width * scale)
                new_height = int(height * scale)

            # Flux requires multiples of (VAE scale 8 * Patch size 2)
            new_width = (new_width // multiple_of) * multiple_of
            new_height = (new_height // multiple_of) * multiple_of

            if new_width != width or new_height != height:
                return new_width, new_height

        return None
```
**EN:** This block defines method `calculate_condition_image_size` on `Flux2PipelineConfig`. It handles calculate condition image size logic. Key calls include `math.sqrt`, and `int`. The implementation branches on conditions. Parameters such as `image`, `width`, and `height` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `calculate_condition_image_size`。 它用于处理 calculate condition image size 相关逻辑。 关键调用包括 `math.sqrt` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `image`、`width` 和 `height` 等参数驱动。

### Lines 553-580: `preprocess_condition_image` implementation / `preprocess_condition_image` 实现
```python
    def preprocess_condition_image(
        self, image, target_width, target_height, vae_image_processor: VaeImageProcessor
    ):
        target_area = 1024 * 1024
        img = image
        if image.width * image.height > target_area:
            resize_to_target_area = getattr(
                vae_image_processor, "_resize_to_target_area", None
            )
            if callable(resize_to_target_area):
                img = resize_to_target_area(image, target_area)
            else:
                scale = math.sqrt(target_area / (image.width * image.height))
                resized_width = int(image.width * scale)
                resized_height = int(image.height * scale)
                img = image.resize(
                    (resized_width, resized_height), PIL.Image.Resampling.LANCZOS
                )

        image_width, image_height = img.size
        vae_scale_factor = self.vae_config.arch_config.vae_scale_factor
        multiple_of = vae_scale_factor * 2
        image_width = (image_width // multiple_of) * multiple_of
        image_height = (image_height // multiple_of) * multiple_of
        img = vae_image_processor.preprocess(
            img, height=image_height, width=image_width, resize_mode="crop"
        )
        return img, (image_width, image_height)
```
**EN:** This block defines method `preprocess_condition_image` on `Flux2PipelineConfig`. It handles preprocess condition image logic. Key calls include `vae_image_processor.preprocess`, `getattr`, `callable`, `resize_to_target_area`, and `math.sqrt`. The implementation branches on conditions. Parameters such as `image`, `target_width`, `target_height`, and `vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `preprocess_condition_image`。 它用于处理 preprocess condition image 相关逻辑。 关键调用包括 `vae_image_processor.preprocess`、`getattr`、`callable`、`resize_to_target_area` 和 `math.sqrt`。 实现中包含条件分支。 本段逻辑主要由 `image`、`target_width`、`target_height` 和 `vae_image_processor` 等参数驱动。

### Lines 582-593: `postprocess_image_latent` implementation / `postprocess_image_latent` 实现
```python
    def postprocess_image_latent(self, latent_condition, batch):
        batch_size = batch.batch_size
        # latent: (1, 128, 32, 32)
        packed = self.maybe_pack_latents(
            latent_condition, None, batch
        )  # (1, 1024, 128)
        packed = packed.squeeze(0)  # (1024, 128) - remove batch dim

        # Concatenate all reference tokens along sequence dimension
        image_latents = packed.unsqueeze(0)  # (1, N*1024, 128)
        image_latents = image_latents.repeat(batch_size, 1, 1)
        return image_latents
```
**EN:** This block defines method `postprocess_image_latent` on `Flux2PipelineConfig`. It handles postprocess image latent logic. Key calls include `self.maybe_pack_latents`, `packed.squeeze`, `packed.unsqueeze`, and `image_latents.repeat`. Parameters such as `latent_condition`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `postprocess_image_latent`。 它用于处理 postprocess image latent 相关逻辑。 关键调用包括 `self.maybe_pack_latents`、`packed.squeeze`、`packed.unsqueeze` 和 `image_latents.repeat`。 本段逻辑主要由 `latent_condition` 和 `batch` 等参数驱动。

### Lines 595-598: `prepare_condition_image_latent_ids` implementation / `prepare_condition_image_latent_ids` 实现
```python
    def prepare_condition_image_latent_ids(self, image_latents, batch):
        image_latent_ids = _prepare_image_ids(image_latents)
        image_latent_ids = image_latent_ids.repeat(batch.batch_size, 1, 1)
        batch.condition_image_latent_ids = image_latent_ids.to(get_local_torch_device())
```
**EN:** This block defines method `prepare_condition_image_latent_ids` on `Flux2PipelineConfig`. It prepares condition image latent ids. Key calls include `_prepare_image_ids`, `image_latent_ids.repeat`, `image_latent_ids.to`, and `get_local_torch_device`. Parameters such as `image_latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `prepare_condition_image_latent_ids`。 它用于准备condition image latent ids。 关键调用包括 `_prepare_image_ids`、`image_latent_ids.repeat`、`image_latent_ids.to` 和 `get_local_torch_device`。 本段逻辑主要由 `image_latents` 和 `batch` 等参数驱动。

### Lines 600-631: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    def get_freqs_cis(
        self, prompt_embeds, width, height, device, rotary_emb, batch, txt_seq_lens
    ):
        self._validate_fixed_text_seq_lens(prompt_embeds, txt_seq_lens)
        txt_ids = _prepare_text_ids(prompt_embeds).to(device=device)

        img_ids = batch.latent_ids
        if img_ids.ndim == 3:
            img_ids = img_ids[0]
        if txt_ids.ndim == 3:
            txt_ids = txt_ids[0]

        # NOTE(mick): prepare it here, to avoid unnecessary computations
        img_cos, img_sin = rotary_emb.forward(img_ids)
        img_cos = shard_rotary_emb_for_sp(img_cos)
        img_sin = shard_rotary_emb_for_sp(img_sin)

        if batch.image_latent is not None:
            cond_ids = batch.condition_image_latent_ids
            if cond_ids.ndim == 3:
                cond_ids = cond_ids[0]
            cond_cos, cond_sin = rotary_emb.forward(cond_ids)
            cond_cos = shard_rotary_emb_for_sp(cond_cos)
            cond_sin = shard_rotary_emb_for_sp(cond_sin)
            img_cos = torch.cat([img_cos, cond_cos], dim=0)
            img_sin = torch.cat([img_sin, cond_sin], dim=0)

        txt_cos, txt_sin = rotary_emb.forward(txt_ids)

        cos = torch.cat([txt_cos, img_cos], dim=0).to(device=device)
        sin = torch.cat([txt_sin, img_sin], dim=0).to(device=device)
        return cos, sin
```
**EN:** This block defines method `get_freqs_cis` on `Flux2PipelineConfig`. It retrieves freqs cis. Key calls include `self._validate_fixed_text_seq_lens`, `_prepare_text_ids.to`, `rotary_emb.forward`, `shard_rotary_emb_for_sp`, and `torch.cat.to`. The implementation branches on conditions. Parameters such as `prompt_embeds`, `width`, `height`, `device`, and `rotary_emb` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `self._validate_fixed_text_seq_lens`、`_prepare_text_ids.to`、`rotary_emb.forward`、`shard_rotary_emb_for_sp` 和 `torch.cat.to`。 实现中包含条件分支。 本段逻辑主要由 `prompt_embeds`、`width`、`height`、`device` 和 `rotary_emb` 等参数驱动。

### Lines 633-657: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        """Build Flux2 positive-conditioning kwargs from encoded text state.

        Flux2 uses encoder index 0 for the Mistral text stream. The stored
        sequence lengths are passed through to rotary-position preparation so
        grouped requests use the same text-length metadata that was produced
        during text encoding.
        """
        txt_seq_lens = self.require_text_seq_lens(
            batch,
            0,
            negative=False,
            expected_batch_size=batch.prompt_embeds[0].shape[0],
        )
        return {
            "freqs_cis": self.get_freqs_cis(
                batch.prompt_embeds[0],
                batch.width,
                batch.height,
                device,
                rotary_emb,
                batch,
                txt_seq_lens,
            )
        }
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `Flux2PipelineConfig`. Build Flux2 positive-conditioning kwargs from encoded text state. Flux2 uses encoder index 0 for the Mistral text stream. Key calls include `self.require_text_seq_lens`, and `self.get_freqs_cis`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self.require_text_seq_lens` 和 `self.get_freqs_cis`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 659-660: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {}
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `Flux2PipelineConfig`. It prepares neg cond kwargs. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 662-663: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        return flux2_pack_latents(latents)
```
**EN:** This block defines method `maybe_pack_latents` on `Flux2PipelineConfig`. It handles maybe pack latents logic. Key calls include `flux2_pack_latents`. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 关键调用包括 `flux2_pack_latents`。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 665-666: `maybe_prepare_latent_ids` implementation / `maybe_prepare_latent_ids` 实现
```python
    def maybe_prepare_latent_ids(self, latents):
        return _prepare_latent_ids(latents)
```
**EN:** This block defines method `maybe_prepare_latent_ids` on `Flux2PipelineConfig`. It handles maybe prepare latent ids logic. Key calls include `_prepare_latent_ids`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `maybe_prepare_latent_ids`。 它用于处理 maybe prepare latent ids 相关逻辑。 关键调用包括 `_prepare_latent_ids`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 668-671: `postprocess_vae_encode` implementation / `postprocess_vae_encode` 实现
```python
    def postprocess_vae_encode(self, image_latents, vae):
        # patchify
        image_latents = _patchify_latents(image_latents)
        return image_latents
```
**EN:** This block defines method `postprocess_vae_encode` on `Flux2PipelineConfig`. It handles postprocess vae encode logic. Key calls include `_patchify_latents`. Parameters such as `image_latents`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `postprocess_vae_encode`。 它用于处理 postprocess vae encode 相关逻辑。 关键调用包括 `_patchify_latents`。 本段逻辑主要由 `image_latents` 和 `vae` 等参数驱动。

### Lines 673-684: `normalize_vae_encode` implementation / `normalize_vae_encode` 实现
```python
    def normalize_vae_encode(self, image_latents, vae):
        if not self._check_vae_has_bn(vae):
            return None

        latents_bn_mean = vae.bn.running_mean.view(1, -1, 1, 1).to(
            image_latents.device, image_latents.dtype
        )
        latents_bn_std = torch.sqrt(
            vae.bn.running_var.view(1, -1, 1, 1)
            + self.vae_config.arch_config.batch_norm_eps
        ).to(image_latents.device, image_latents.dtype)
        return (image_latents - latents_bn_mean) / latents_bn_std
```
**EN:** This block defines method `normalize_vae_encode` on `Flux2PipelineConfig`. It handles normalize vae encode logic. Key calls include `vae.bn.running_mean.view.to`, `torch.sqrt.to`, `self._check_vae_has_bn`, `vae.bn.running_mean.view`, and `torch.sqrt`. The implementation branches on conditions. Parameters such as `image_latents`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `normalize_vae_encode`。 它用于处理 normalize vae encode 相关逻辑。 关键调用包括 `vae.bn.running_mean.view.to`、`torch.sqrt.to`、`self._check_vae_has_bn`、`vae.bn.running_mean.view` 和 `torch.sqrt`。 实现中包含条件分支。 本段逻辑主要由 `image_latents` 和 `vae` 等参数驱动。

### Lines 686-690: `_check_vae_has_bn` implementation / `_check_vae_has_bn` 实现
```python
    def _check_vae_has_bn(self, vae):
        """Check if VAE has bn attribute (cached check to avoid repeated hasattr calls)."""
        if not hasattr(self, "_vae_has_bn_cache"):
            self._vae_has_bn_cache = hasattr(vae, "bn") and vae.bn is not None
        return self._vae_has_bn_cache
```
**EN:** This block defines method `_check_vae_has_bn` on `Flux2PipelineConfig`. Check if VAE has bn attribute (cached check to avoid repeated hasattr calls). Key calls include `hasattr`. The implementation branches on conditions. Parameters such as `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `_check_vae_has_bn`。 它用于检查vae has bn。 关键调用包括 `hasattr`。 实现中包含条件分支。 本段逻辑主要由 `vae` 等参数驱动。

### Lines 692-701: `preprocess_decoding` implementation / `preprocess_decoding` 实现
```python
    def preprocess_decoding(self, latents, server_args=None, vae=None):
        """Preprocess latents before decoding.

        Dynamically adapts based on VAE type:
        - Standard Flux2 VAE (has bn): needs unpatchify (128 channels -> 32 channels)
        - Distilled VAE (no bn): keeps patchified latents (128 channels)
        """
        if vae is not None and self._check_vae_has_bn(vae):
            return _unpatchify_latents(latents)
        return latents
```
**EN:** This block defines method `preprocess_decoding` on `Flux2PipelineConfig`. Preprocess latents before decoding. Dynamically adapts based on VAE type: - Standard Flux2 VAE (has bn): needs unpatchify (128 channels -> 32 channels) - Distilled VAE (no bn): keeps patchified latents (128 channels) Key calls include `self._check_vae_has_bn`, and `_unpatchify_latents`. The implementation branches on conditions. Parameters such as `latents`, `server_args`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `preprocess_decoding`。 它用于处理 preprocess decoding 相关逻辑。 关键调用包括 `self._check_vae_has_bn` 和 `_unpatchify_latents`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`server_args` 和 `vae` 等参数驱动。

### Lines 703-730: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        """Get scale and shift for decoding.

        Dynamically adapts based on VAE type:
        - Standard Flux2 VAE (has bn): uses BatchNorm statistics
        - Distilled VAE (no bn): uses scaling_factor from config
        """
        vae_arch_config = self.vae_config.arch_config

        if self._check_vae_has_bn(vae):
            # Standard Flux2 VAE: use BatchNorm statistics
            latents_bn_mean = vae.bn.running_mean.view(1, -1, 1, 1).to(device, dtype)
            latents_bn_std = torch.sqrt(
                vae.bn.running_var.view(1, -1, 1, 1) + vae_arch_config.batch_norm_eps
            ).to(device, dtype)
            return 1 / latents_bn_std, latents_bn_mean

        # Distilled VAE or unknown: use scaling_factor
        scaling_factor = (
            getattr(vae.config, "scaling_factor", None)
            if hasattr(vae, "config")
            else getattr(vae, "scaling_factor", None)
        ) or getattr(vae_arch_config, "scaling_factor", 0.13025)

        scale = torch.tensor(scaling_factor, device=device, dtype=dtype).view(
            1, 1, 1, 1
        )
        return 1 / scale, None
```
**EN:** This block defines method `get_decode_scale_and_shift` on `Flux2PipelineConfig`. Get scale and shift for decoding. Dynamically adapts based on VAE type: - Standard Flux2 VAE (has bn): uses BatchNorm statistics - Distilled VAE (no bn): uses scaling_factor from config Key calls include `self._check_vae_has_bn`, `torch.tensor.view`, `vae.bn.running_mean.view.to`, `torch.sqrt.to`, and `getattr`. The implementation branches on conditions. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `self._check_vae_has_bn`、`torch.tensor.view`、`vae.bn.running_mean.view.to`、`torch.sqrt.to` 和 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 732-736: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        latent_ids = batch.latent_ids
        latents = _unpack_latents_with_ids(latents, latent_ids)

        return latents
```
**EN:** This block defines method `post_denoising_loop` on `Flux2PipelineConfig`. It post-processes denoising loop. Key calls include `_unpack_latents_with_ids`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `_unpack_latents_with_ids`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 738-741: `slice_noise_pred` implementation / `slice_noise_pred` 实现
```python
    def slice_noise_pred(self, noise, latents):
        # remove noise over input image
        noise = noise[:, : latents.size(1) :]
        return noise
```
**EN:** This block defines method `slice_noise_pred` on `Flux2PipelineConfig`. It handles slice noise pred logic. Key calls include `latents.size`. Parameters such as `noise`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2PipelineConfig` 的方法 `slice_noise_pred`。 它用于处理 slice noise pred 相关逻辑。 关键调用包括 `latents.size`。 本段逻辑主要由 `noise` 和 `latents` 等参数驱动。

### Lines 745-746: `Flux2KleinPipelineConfig` class overview / `Flux2KleinPipelineConfig` 类概览
```python
class Flux2KleinPipelineConfig(Flux2PipelineConfig):
    # Klein is distilled, so no guidance embeddings
```
**EN:** This block defines class `Flux2KleinPipelineConfig`. It encapsulates flux2 klein pipeline config behavior. It inherits from `Flux2PipelineConfig`.
**CN:** 该代码块定义了类 `Flux2KleinPipelineConfig`。 它用于封装 flux2 klein pipeline config 相关行为。 它继承自 `Flux2PipelineConfig`。

### Lines 747-761: supporting statements / 辅助语句
```python
    should_use_guidance: bool = False

    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))

    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Qwen3TextConfig(),)
    )

    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (None,),
    )

    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (flux2_klein_postprocess_text,)
    )
```
**EN:** This block gathers supporting statements inside `Flux2KleinPipelineConfig`. It updates names such as `should_use_guidance`, `text_encoder_precisions`, `text_encoder_configs`, `preprocess_text_funcs`, and `postprocess_text_funcs`. The code collaborates with `field`, and `Qwen3TextConfig`.
**CN:** 该代码块汇集了位于 `Flux2KleinPipelineConfig` 内部的辅助语句。 它会更新 `should_use_guidance`、`text_encoder_precisions`、`text_encoder_configs`、`preprocess_text_funcs` 和 `postprocess_text_funcs` 等名称。 代码会与 `field` 和 `Qwen3TextConfig` 协同工作。

### Lines 763-800: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompts: list[str], tokenizer, tok_kwargs) -> dict:
        if prompts and isinstance(prompts[0], list):
            prompts = [p for prompt in prompts for p in prompt]

        def _apply_chat_template(prompt: str) -> str:
            messages = [{"role": "user", "content": prompt}]
            try:
                return tokenizer.apply_chat_template(
                    messages,
                    tokenize=False,
                    add_generation_prompt=True,
                    enable_thinking=False,
                )
            except TypeError:
                return tokenizer.apply_chat_template(
                    messages,
                    tokenize=False,
                    add_generation_prompt=True,
                )

        texts = [_apply_chat_template(prompt) for prompt in prompts]

        tok_kwargs = dict(tok_kwargs or {})
        tok_kwargs.pop("max_length", None)
        # Flux2 Klein uses max_length 512.
        max_length = 512
        padding = tok_kwargs.pop("padding", "max_length")
        truncation = tok_kwargs.pop("truncation", True)
        return_tensors = tok_kwargs.pop("return_tensors", "pt")

        return tokenizer(
            texts,
            padding=padding,
            truncation=truncation,
            max_length=max_length,
            return_tensors=return_tensors,
            **tok_kwargs,
        )
```
**EN:** This block defines method `tokenize_prompt` on `Flux2KleinPipelineConfig`. It handles tokenize prompt logic. Key calls include `dict`, `tok_kwargs.pop`, `tokenizer`, `isinstance`, and `_apply_chat_template`. The implementation branches on conditions, handles exceptional paths. Parameters such as `prompts`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `Flux2KleinPipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `dict`、`tok_kwargs.pop`、`tokenizer`、`isinstance` 和 `_apply_chat_template`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `prompts`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

## Key Concepts / 关键概念
- `t5_postprocess_text`: Top-level function that handles t5 postprocess text logic. / 顶层函数，用于处理 t5 postprocess text 相关逻辑。
- `FluxPipelineConfig`: Configuration for the FLUX pipeline. / 核心类，用于封装 flux pipeline config 相关行为。
- `_prepare_latent_ids`: Generates 4D position coordinates (T, H, W, L) for latent tensors. / 顶层函数，用于准备latent ids。
- `_unpack_latents_with_ids`: using position ids to scatter tokens into place / 顶层函数，用于解包latents with ids。
- `_patchify_latents`: Top-level function that handles patchify latents logic. / 顶层函数，用于处理 patchify latents 相关逻辑。
- `_unpatchify_latents`: Top-level function that handles unpatchify latents logic. / 顶层函数，用于处理 unpatchify latents 相关逻辑。
- `_prepare_text_ids`: Top-level function that prepares text ids. / 顶层函数，用于准备text ids。
- `_prepare_image_ids`: Top-level function that prepares image ids. / 顶层函数，用于准备image ids。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `PIL`, `torch`, `diffusers.image_processor`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.flux`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.qwen3`, `sglang.multimodal_gen.configs.models.vaes.flux`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.hunyuan`, `sglang.multimodal_gen.configs.pipeline_configs.qwen_image`, `sglang.multimodal_gen.runtime.distributed`

- **Total lines / 总行数**: 800
