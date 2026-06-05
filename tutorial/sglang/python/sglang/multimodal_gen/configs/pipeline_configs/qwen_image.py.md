# qwen_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/qwen_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `QwenImagePipelineConfig`, `QwenImageEditPipelineConfig`, and `QwenImageEditPlusPipelineConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `QwenImagePipelineConfig`、`QwenImageEditPipelineConfig` 和 `QwenImageEditPlusPipelineConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-26: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Callable

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.qwenimage import (
    QwenImageDitConfig,
    QwenImageEditPlus_2511_DitConfig,
)
from sglang.multimodal_gen.configs.models.encoders.qwen_image import Qwen2_5VLConfig
from sglang.multimodal_gen.configs.models.vaes.qwenimage import QwenImageVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ImagePipelineConfig,
    ModelTaskType,
    maybe_unpad_latents,
    pad_text_embeddings_with_mask,
    shard_rotary_emb_for_sp,
)
from sglang.multimodal_gen.configs.post_training.pipeline_configs import (
    QwenImageRolloutPipelineMixin,
)
from sglang.multimodal_gen.runtime.models.vision_utils import resize
from sglang.multimodal_gen.utils import calculate_dimensions
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.qwenimage`, and `sglang.multimodal_gen.configs.models.encoders.qwen_image`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.qwenimage` 和 `sglang.multimodal_gen.configs.models.encoders.qwen_image`。这些依赖为后续实现提供所需符号。

### Lines 29-35: `_extract_masked_hidden` implementation / `_extract_masked_hidden` 实现
```python
def _extract_masked_hidden(hidden_states: torch.Tensor, mask: torch.Tensor):
    bool_mask = mask.bool()
    valid_lengths = bool_mask.sum(dim=1)
    selected = hidden_states[bool_mask]
    split_result = torch.split(selected, valid_lengths.tolist(), dim=0)

    return split_result
```
**EN:** This block defines function `_extract_masked_hidden`. It handles extract masked hidden logic. Key calls include `mask.bool`, `bool_mask.sum`, `torch.split`, and `valid_lengths.tolist`. Parameters such as `hidden_states`, and `mask` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_extract_masked_hidden`。 它用于处理 extract masked hidden 相关逻辑。 关键调用包括 `mask.bool`、`bool_mask.sum`、`torch.split` 和 `valid_lengths.tolist`。 本段逻辑主要由 `hidden_states` 和 `mask` 等参数驱动。

### Lines 38-43: `qwen_image_preprocess_text` implementation / `qwen_image_preprocess_text` 实现
```python
def qwen_image_preprocess_text(prompt):
    prompt_template_encode = "<|im_start|>system\nDescribe the image by detailing the color, shape, size, texture, quantity, text, spatial relationships of the objects and background:<|im_end|>\n<|im_start|>user\n{}<|im_end|>\n<|im_start|>assistant\n"

    template = prompt_template_encode
    txt = template.format(prompt)
    return txt
```
**EN:** This block defines function `qwen_image_preprocess_text`. It handles qwen image preprocess text logic. Key calls include `template.format`. Parameters such as `prompt` drive the behavior in this section.
**CN:** 该代码块定义了函数 `qwen_image_preprocess_text`。 它用于处理 qwen image preprocess text 相关逻辑。 关键调用包括 `template.format`。 本段逻辑主要由 `prompt` 等参数驱动。

### Lines 46-63: `qwen_image_postprocess_text` implementation / `qwen_image_postprocess_text` 实现
```python
def qwen_image_postprocess_text(
    outputs, _text_inputs, drop_idx=34, return_attention_mask=False
):
    """Postprocess Qwen text embeddings.

    Returns padded embeddings by default, or TextConditioningOutput when
    embedding-aligned masks are requested.
    """
    # squeeze the batch dim
    hidden_states = outputs.hidden_states[-1]
    split_hidden_states = _extract_masked_hidden(
        hidden_states, _text_inputs.attention_mask
    )
    split_hidden_states = [e[drop_idx:] for e in split_hidden_states]
    conditioning = pad_text_embeddings_with_mask(split_hidden_states)
    if return_attention_mask:
        return conditioning
    return conditioning.prompt_embeds
```
**EN:** This block defines function `qwen_image_postprocess_text`. Postprocess Qwen text embeddings. Returns padded embeddings by default, or TextConditioningOutput when embedding-aligned masks are requested. Key calls include `_extract_masked_hidden`, and `pad_text_embeddings_with_mask`. The implementation branches on conditions. Parameters such as `outputs`, `_text_inputs`, `drop_idx`, and `return_attention_mask` drive the behavior in this section.
**CN:** 该代码块定义了函数 `qwen_image_postprocess_text`。 它用于处理 qwen image postprocess text 相关逻辑。 关键调用包括 `_extract_masked_hidden` 和 `pad_text_embeddings_with_mask`。 实现中包含条件分支。 本段逻辑主要由 `outputs`、`_text_inputs`、`drop_idx` 和 `return_attention_mask` 等参数驱动。

### Lines 66-67: `qwen_image_edit_postprocess_text` implementation / `qwen_image_edit_postprocess_text` 实现
```python
def qwen_image_edit_postprocess_text(outputs, _text_inputs):
    return qwen_image_postprocess_text(outputs, _text_inputs, drop_idx=64)
```
**EN:** This block defines function `qwen_image_edit_postprocess_text`. It handles qwen image edit postprocess text logic. Key calls include `qwen_image_postprocess_text`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `qwen_image_edit_postprocess_text`。 它用于处理 qwen image edit postprocess text 相关逻辑。 关键调用包括 `qwen_image_postprocess_text`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 70-71: `_normalize_prompt_list` implementation / `_normalize_prompt_list` 实现
```python
def _normalize_prompt_list(prompt):
    return [prompt] if isinstance(prompt, str) else prompt
```
**EN:** This block defines function `_normalize_prompt_list`. It handles normalize prompt list logic. Key calls include `isinstance`. Parameters such as `prompt` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_normalize_prompt_list`。 它用于处理 normalize prompt list 相关逻辑。 关键调用包括 `isinstance`。 本段逻辑主要由 `prompt` 等参数驱动。

### Lines 74-77: `_normalize_image_list` implementation / `_normalize_image_list` 实现
```python
def _normalize_image_list(images):
    if images is None:
        return []
    return images if isinstance(images, list) else [images]
```
**EN:** This block defines function `_normalize_image_list`. It handles normalize image list logic. Key calls include `isinstance`. The implementation branches on conditions. Parameters such as `images` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_normalize_image_list`。 它用于处理 normalize image list 相关逻辑。 关键调用包括 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `images` 等参数驱动。

### Lines 80-82: `_build_qwen_edit_image_prompt` implementation / `_build_qwen_edit_image_prompt` 实现
```python
def _build_qwen_edit_image_prompt(num_images: int) -> str:
    img_prompt_template = "Picture {}: <|vision_start|><|image_pad|><|vision_end|>"
    return "".join(img_prompt_template.format(i + 1) for i in range(num_images))
```
**EN:** This block defines function `_build_qwen_edit_image_prompt`. It builds qwen edit image prompt. Key calls include `join`, `img_prompt_template.format`, and `range`. Parameters such as `num_images` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_qwen_edit_image_prompt`。 它用于构建qwen edit image prompt。 关键调用包括 `join`、`img_prompt_template.format` 和 `range`。 本段逻辑主要由 `num_images` 等参数驱动。

### Lines 85-98: `_resolve_qwen_edit_per_prompt_images` implementation / `_resolve_qwen_edit_per_prompt_images` 实现
```python
def _resolve_qwen_edit_per_prompt_images(prompt_list, image_list):
    if len(prompt_list) <= 1:
        return [image_list]

    if len(image_list) <= 1:
        return [list(image_list) for _ in prompt_list]

    if len(image_list) != len(prompt_list):
        raise ValueError(
            "QwenImageEditPlus expects either one shared condition image or "
            "the same number of condition images and prompts."
        )

    return [[image] for image in image_list]
```
**EN:** This block defines function `_resolve_qwen_edit_per_prompt_images`. It resolves qwen edit per prompt images. Key calls include `len`, `ValueError`, and `list`. The implementation branches on conditions. Parameters such as `prompt_list`, and `image_list` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_resolve_qwen_edit_per_prompt_images`。 它用于解析qwen edit per prompt images。 关键调用包括 `len`、`ValueError` 和 `list`。 实现中包含条件分支。 本段逻辑主要由 `prompt_list` 和 `image_list` 等参数驱动。

### Lines 101-106: `_shard_qwen_edit_img_cache_for_sp` implementation / `_shard_qwen_edit_img_cache_for_sp` 实现
```python
def _shard_qwen_edit_img_cache_for_sp(
    img_cache: torch.Tensor, noisy_img_seq_len: int, device: torch.device
) -> torch.Tensor:
    noisy_img_cache = shard_rotary_emb_for_sp(img_cache[:noisy_img_seq_len, :])
    condition_img_cache = shard_rotary_emb_for_sp(img_cache[noisy_img_seq_len:, :])
    return torch.cat([noisy_img_cache, condition_img_cache], dim=0).to(device=device)
```
**EN:** This block defines function `_shard_qwen_edit_img_cache_for_sp`. It handles shard qwen edit img cache for sp logic. Key calls include `shard_rotary_emb_for_sp`, `torch.cat.to`, and `torch.cat`. Parameters such as `img_cache`, `noisy_img_seq_len`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_shard_qwen_edit_img_cache_for_sp`。 它用于处理 shard qwen edit img cache for sp 相关逻辑。 关键调用包括 `shard_rotary_emb_for_sp`、`torch.cat.to` 和 `torch.cat`。 本段逻辑主要由 `img_cache`、`noisy_img_seq_len` 和 `device` 等参数驱动。

### Lines 109-124: `_shard_qwen_edit_freqs_cis_for_sp` implementation / `_shard_qwen_edit_freqs_cis_for_sp` 实现
```python
def _shard_qwen_edit_freqs_cis_for_sp(freqs_cis, noisy_img_seq_len, device):
    if isinstance(freqs_cis[0], torch.Tensor) and freqs_cis[0].dim() == 2:
        img_cache, txt_cache = freqs_cis
        return (
            _shard_qwen_edit_img_cache_for_sp(img_cache, noisy_img_seq_len, device),
            txt_cache,
        )

    (img_cos, img_sin), (txt_cos, txt_sin) = freqs_cis
    return (
        (
            _shard_qwen_edit_img_cache_for_sp(img_cos, noisy_img_seq_len, device),
            _shard_qwen_edit_img_cache_for_sp(img_sin, noisy_img_seq_len, device),
        ),
        (txt_cos, txt_sin),
    )
```
**EN:** This block defines function `_shard_qwen_edit_freqs_cis_for_sp`. It handles shard qwen edit freqs cis for sp logic. Key calls include `isinstance`, `freqs_cis.dim`, and `_shard_qwen_edit_img_cache_for_sp`. The implementation branches on conditions. Parameters such as `freqs_cis`, `noisy_img_seq_len`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_shard_qwen_edit_freqs_cis_for_sp`。 它用于处理 shard qwen edit freqs cis for sp 相关逻辑。 关键调用包括 `isinstance`、`freqs_cis.dim` 和 `_shard_qwen_edit_img_cache_for_sp`。 实现中包含条件分支。 本段逻辑主要由 `freqs_cis`、`noisy_img_seq_len` 和 `device` 等参数驱动。

### Lines 128-137: `_pack_latents` implementation / `_pack_latents` 实现
```python
def _pack_latents(latents, batch_size, num_channels_latents, height, width):
    latents = latents.view(
        batch_size, num_channels_latents, height // 2, 2, width // 2, 2
    )
    latents = latents.permute(0, 2, 4, 1, 3, 5)
    latents = latents.reshape(
        batch_size, (height // 2) * (width // 2), num_channels_latents * 4
    )

    return latents
```
**EN:** This block defines function `_pack_latents`. It packs latents. Key calls include `latents.view`, `latents.permute`, and `latents.reshape`. Parameters such as `latents`, `batch_size`, `num_channels_latents`, `height`, and `width` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_pack_latents`。 它用于打包latents。 关键调用包括 `latents.view`、`latents.permute` 和 `latents.reshape`。 本段逻辑主要由 `latents`、`batch_size`、`num_channels_latents`、`height` 和 `width` 等参数驱动。

### Lines 141-143: `QwenImagePipelineConfig` class overview / `QwenImagePipelineConfig` 类概览
```python
class QwenImagePipelineConfig(QwenImageRolloutPipelineMixin, ImagePipelineConfig):
    """Configuration for the QwenImage pipeline."""
```
**EN:** This block defines class `QwenImagePipelineConfig`. Configuration for the QwenImage pipeline. It inherits from `QwenImageRolloutPipelineMixin`, and `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `QwenImagePipelineConfig`。 它用于封装 qwen image pipeline config 相关行为。 它继承自 `QwenImageRolloutPipelineMixin` 和 `ImagePipelineConfig`。

### Lines 144-182: supporting statements / 辅助语句
```python
    should_use_guidance: bool = False
    task_type: ModelTaskType = ModelTaskType.T2I

    vae_tiling: bool = False

    vae_sp: bool = False

    vae_precision: str = "bf16"

    dit_config: DiTConfig = field(default_factory=QwenImageDitConfig)
    # VAE
    vae_config: VAEConfig = field(default_factory=QwenImageVAEConfig)

    enable_autocast: bool = False

    # Text encoding stage
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Qwen2_5VLConfig(),)
    )

    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))

    preprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (qwen_image_preprocess_text,)
    )

    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (qwen_image_postprocess_text,)
    )

    text_encoder_extra_args: list[dict] = field(
        default_factory=lambda: [
            dict(
                padding=True,
                truncation=True,
            ),
            None,
        ]
    )
```
**EN:** This block gathers supporting statements inside `QwenImagePipelineConfig`. It updates names such as `should_use_guidance`, `task_type`, `vae_tiling`, `vae_sp`, `vae_precision`, and `dit_config`. The code collaborates with `field`, `Qwen2_5VLConfig`, and `dict`.
**CN:** 该代码块汇集了位于 `QwenImagePipelineConfig` 内部的辅助语句。 它会更新 `should_use_guidance`、`task_type`、`vae_tiling`、`vae_sp`、`vae_precision` 和 `dit_config` 等名称。 代码会与 `field`、`Qwen2_5VLConfig` 和 `dict` 协同工作。

### Lines 184-192: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompts: list[str], tokenizer, tok_kwargs) -> dict:
        tok_kwargs.setdefault("truncation", True)

        if tok_kwargs.get("max_length") is not None:
            tok_kwargs["padding"] = "max_length"
        else:
            tok_kwargs.setdefault("max_length", 1024)
            tok_kwargs["padding"] = True
        return tokenizer(prompts, **tok_kwargs)
```
**EN:** This block defines method `tokenize_prompt` on `QwenImagePipelineConfig`. It handles tokenize prompt logic. Key calls include `tok_kwargs.setdefault`, `tokenizer`, and `tok_kwargs.get`. The implementation branches on conditions. Parameters such as `prompts`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tok_kwargs.setdefault`、`tokenizer` 和 `tok_kwargs.get`。 实现中包含条件分支。 本段逻辑主要由 `prompts`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 194-195: `prepare_sigmas` implementation / `prepare_sigmas` 实现
```python
    def prepare_sigmas(self, sigmas, num_inference_steps):
        return self._prepare_sigmas(sigmas, num_inference_steps)
```
**EN:** This block defines method `prepare_sigmas` on `QwenImagePipelineConfig`. It prepares sigmas. Key calls include `self._prepare_sigmas`. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `prepare_sigmas`。 它用于准备sigmas。 关键调用包括 `self._prepare_sigmas`。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 197-200: `get_classifier_free_guidance_scale` implementation / `get_classifier_free_guidance_scale` 实现
```python
    def get_classifier_free_guidance_scale(self, batch, guidance_scale: float) -> float:
        if batch.true_cfg_scale is not None:
            return batch.true_cfg_scale
        return guidance_scale
```
**EN:** This block defines method `get_classifier_free_guidance_scale` on `QwenImagePipelineConfig`. It retrieves classifier free guidance scale. The implementation branches on conditions. Parameters such as `batch`, and `guidance_scale` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `get_classifier_free_guidance_scale`。 它用于获取classifier free guidance scale。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `guidance_scale` 等参数驱动。

### Lines 202-225: `postprocess_cfg_noise` implementation / `postprocess_cfg_noise` 实现
```python
    def postprocess_cfg_noise(
        self,
        batch,
        noise_pred: torch.Tensor,
        noise_pred_cond: torch.Tensor,
    ) -> torch.Tensor:
        # Qwen-Image follows the official diffusers true-CFG behavior:
        # after combining cond/uncond with true_cfg_scale, match the per-token norm
        # back to the conditional branch.
        cfg_scale = (
            batch.true_cfg_scale
            if batch.true_cfg_scale is not None
            else batch.guidance_scale
        )
        if (
            cfg_scale is None
            or cfg_scale <= 1.0
            or not batch.do_classifier_free_guidance
        ):
            return noise_pred

        cond_norm = torch.norm(noise_pred_cond, dim=-1, keepdim=True)
        noise_norm = torch.norm(noise_pred, dim=-1, keepdim=True).clamp_min(1e-12)
        return noise_pred * (cond_norm / noise_norm)
```
**EN:** This block defines method `postprocess_cfg_noise` on `QwenImagePipelineConfig`. It handles postprocess cfg noise logic. Key calls include `torch.norm`, and `torch.norm.clamp_min`. The implementation branches on conditions. Parameters such as `batch`, `noise_pred`, and `noise_pred_cond` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `postprocess_cfg_noise`。 它用于处理 postprocess cfg noise 相关逻辑。 关键调用包括 `torch.norm` 和 `torch.norm.clamp_min`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`noise_pred` 和 `noise_pred_cond` 等参数驱动。

### Lines 227-237: `prepare_image_processor_kwargs` implementation / `prepare_image_processor_kwargs` 实现
```python
    def prepare_image_processor_kwargs(self, batch, neg=False):
        prompt = batch.prompt if not neg else batch.negative_prompt
        if prompt:
            prompt_template_encode = "<|im_start|>system\nDescribe the key features of the input image (color, shape, size, texture, objects, background), then explain how the user's text instruction should alter or modify the image. Generate a new image that meets the user's requirements while maintaining consistency with the original input where appropriate.<|im_end|>\n<|im_start|>user\n<|vision_start|><|image_pad|><|vision_end|>{}<|im_end|>\n<|im_start|>assistant\n"
            prompt_list = _normalize_prompt_list(prompt)
            txt = [
                prompt_template_encode.format(cur_prompt) for cur_prompt in prompt_list
            ]
            return dict(text=txt, padding=True)
        else:
            return {}
```
**EN:** This block defines method `prepare_image_processor_kwargs` on `QwenImagePipelineConfig`. It prepares image processor kwargs. Key calls include `_normalize_prompt_list`, `dict`, and `prompt_template_encode.format`. The implementation branches on conditions. Parameters such as `batch`, and `neg` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `prepare_image_processor_kwargs`。 它用于准备image processor kwargs。 关键调用包括 `_normalize_prompt_list`、`dict` 和 `prompt_template_encode.format`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `neg` 等参数驱动。

### Lines 239-244: `get_vae_scale_factor` implementation / `get_vae_scale_factor` 实现
```python
    def get_vae_scale_factor(self):
        return getattr(
            self.vae_config.arch_config,
            "vae_scale_factor",
            self.vae_config.get_vae_scale_factor(),
        )
```
**EN:** This block defines method `get_vae_scale_factor` on `QwenImagePipelineConfig`. It retrieves vae scale factor. Key calls include `getattr`, and `self.vae_config.get_vae_scale_factor`.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `get_vae_scale_factor`。 它用于获取vae scale factor。 关键调用包括 `getattr` 和 `self.vae_config.get_vae_scale_factor`。

### Lines 246-252: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        vae_scale_factor = self.get_vae_scale_factor()
        height = 2 * (batch.height // (vae_scale_factor * 2))
        width = 2 * (batch.width // (vae_scale_factor * 2))
        num_channels_latents = self.dit_config.arch_config.in_channels // 4
        shape = (batch_size, 1, num_channels_latents, height, width)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `QwenImagePipelineConfig`. It prepares latent shape. Key calls include `self.get_vae_scale_factor`. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 关键调用包括 `self.get_vae_scale_factor`。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 254-260: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        vae_scale_factor = self.get_vae_scale_factor()
        height = 2 * (batch.height // (vae_scale_factor * 2))
        width = 2 * (batch.width // (vae_scale_factor * 2))
        num_channels_latents = self.dit_config.arch_config.in_channels // 4
        # pack latents
        return _pack_latents(latents, batch_size, num_channels_latents, height, width)
```
**EN:** This block defines method `maybe_pack_latents` on `QwenImagePipelineConfig`. It handles maybe pack latents logic. Key calls include `self.get_vae_scale_factor`, and `_pack_latents`. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 关键调用包括 `self.get_vae_scale_factor` 和 `_pack_latents`。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 262-272: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        vae_arch_config = self.vae_config.arch_config
        scaling_factor = 1.0 / torch.tensor(
            vae_arch_config.latents_std, device=device
        ).view(1, vae_arch_config.z_dim, 1, 1, 1).to(device, dtype)
        shift_factor = (
            torch.tensor(vae_arch_config.latents_mean)
            .view(1, vae_arch_config.z_dim, 1, 1, 1)
            .to(device, dtype)
        )
        return scaling_factor, shift_factor
```
**EN:** This block defines method `get_decode_scale_and_shift` on `QwenImagePipelineConfig`. It retrieves decode scale and shift. Key calls include `torch.tensor.view.to`, `torch.tensor.view`, and `torch.tensor`. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `torch.tensor.view.to`、`torch.tensor.view` 和 `torch.tensor`。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 274-299: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    @staticmethod
    def get_freqs_cis(img_shapes, txt_seq_lens, rotary_emb, device, dtype):
        # img_shapes: for global entire image
        img_freqs, txt_freqs = rotary_emb(img_shapes, txt_seq_lens, device=device)

        max_txt_seq_len = max(txt_seq_lens) if txt_seq_lens else 0
        txt_cache_len = int(txt_freqs.shape[0])
        if max_txt_seq_len > txt_cache_len:
            overflow = max_txt_seq_len - txt_cache_len
            raise ValueError(
                "QwenImage RoPE text cache overflow before denoising: "
                f"required_txt_seq_len={max_txt_seq_len}, txt_cache_len={txt_cache_len}, "
                f"overflow={overflow}. "
                "Please reduce the number of input images, shorten the prompt, "
                "or lower the requested resolution."
            )

        # flashinfer RoPE expects a float32 cos/sin cache concatenated on the last dim
        img_cos_half = img_freqs.real.to(dtype=torch.float32).contiguous()
        img_sin_half = img_freqs.imag.to(dtype=torch.float32).contiguous()
        txt_cos_half = txt_freqs.real.to(dtype=torch.float32).contiguous()
        txt_sin_half = txt_freqs.imag.to(dtype=torch.float32).contiguous()

        img_cos_sin_cache = torch.cat([img_cos_half, img_sin_half], dim=-1)
        txt_cos_sin_cache = torch.cat([txt_cos_half, txt_sin_half], dim=-1)
        return img_cos_sin_cache, txt_cos_sin_cache
```
**EN:** This block defines method `get_freqs_cis` on `QwenImagePipelineConfig`. It retrieves freqs cis. Key calls include `rotary_emb`, `int`, `img_freqs.real.to.contiguous`, `img_freqs.imag.to.contiguous`, and `txt_freqs.real.to.contiguous`. The implementation branches on conditions. Parameters such as `img_shapes`, `txt_seq_lens`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `rotary_emb`、`int`、`img_freqs.real.to.contiguous`、`img_freqs.imag.to.contiguous` 和 `txt_freqs.real.to.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `img_shapes`、`txt_seq_lens`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 301-349: `_prepare_cond_kwargs` implementation / `_prepare_cond_kwargs` 实现
```python
    def _prepare_cond_kwargs(
        self, batch, prompt_embeds, rotary_emb, device, dtype, *, negative=False
    ):
        """Build Qwen DiT conditioning kwargs for positive or negative prompts.

        The kwargs include text lengths for RoPE construction and optional
        encoder masks for cross-attention.
        """
        batch_size = prompt_embeds[0].shape[0]
        text_seq_len = prompt_embeds[0].shape[1]
        height = batch.height
        width = batch.width
        vae_scale_factor = self.get_vae_scale_factor()

        img_shapes = [
            [
                (
                    1,
                    height // vae_scale_factor // 2,
                    width // vae_scale_factor // 2,
                )
            ]
        ] * batch_size
        txt_seq_lens, encoder_hidden_states_mask = self._prepare_text_conditioning(
            batch, 0, text_seq_len, batch_size, negative=negative
        )

        if rotary_emb is None:
            cond_kwargs = {
                "img_shapes": img_shapes,
                "txt_seq_lens": txt_seq_lens,
                "freqs_cis": None,
                "encoder_hidden_states_mask": encoder_hidden_states_mask,
            }
            return cond_kwargs

        freqs_cis = self.get_freqs_cis(
            img_shapes, txt_seq_lens, rotary_emb, device, dtype
        )

        img_cache, txt_cache = freqs_cis
        img_cache = shard_rotary_emb_for_sp(img_cache)
        cond_kwargs = {
            "txt_seq_lens": txt_seq_lens,
            "freqs_cis": (img_cache, txt_cache),
            "img_shapes": img_shapes,
            "encoder_hidden_states_mask": encoder_hidden_states_mask,
        }
        return cond_kwargs
```
**EN:** This block defines method `_prepare_cond_kwargs` on `QwenImagePipelineConfig`. Build Qwen DiT conditioning kwargs for positive or negative prompts. The kwargs include text lengths for RoPE construction and optional encoder masks for cross-attention. Key calls include `self.get_vae_scale_factor`, `self._prepare_text_conditioning`, `self.get_freqs_cis`, and `shard_rotary_emb_for_sp`. The implementation branches on conditions. Parameters such as `batch`, `prompt_embeds`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `_prepare_cond_kwargs`。 它用于准备cond kwargs。 关键调用包括 `self.get_vae_scale_factor`、`self._prepare_text_conditioning`、`self.get_freqs_cis` 和 `shard_rotary_emb_for_sp`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`prompt_embeds`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 351-379: `_prepare_text_conditioning` implementation / `_prepare_text_conditioning` 实现
```python
    def _prepare_text_conditioning(
        self,
        batch,
        encoder_index: int,
        text_seq_len: int,
        batch_size: int,
        *,
        negative: bool = False,
    ):
        """Return Qwen text lengths and an optional DiT attention mask.

        Single-request execution uses the full padded length. Batched execution
        uses stored per-request lengths and masks from text encoding.
        """
        if batch_size == 1:
            return [text_seq_len], None

        txt_seq_lens = self.require_text_seq_lens(
            batch, encoder_index, negative=negative, expected_batch_size=batch_size
        )
        encoder_hidden_states_mask = self._prepare_encoder_hidden_states_mask(
            batch,
            encoder_index,
            txt_seq_lens,
            text_seq_len,
            batch_size,
            negative=negative,
        )
        return txt_seq_lens, encoder_hidden_states_mask
```
**EN:** This block defines method `_prepare_text_conditioning` on `QwenImagePipelineConfig`. Return Qwen text lengths and an optional DiT attention mask. Single-request execution uses the full padded length. Key calls include `self.require_text_seq_lens`, and `self._prepare_encoder_hidden_states_mask`. The implementation branches on conditions. Parameters such as `batch`, `encoder_index`, `text_seq_len`, and `batch_size` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `_prepare_text_conditioning`。 它用于准备text conditioning。 关键调用包括 `self.require_text_seq_lens` 和 `self._prepare_encoder_hidden_states_mask`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`encoder_index`、`text_seq_len` 和 `batch_size` 等参数驱动。

### Lines 381-427: `_prepare_encoder_hidden_states_mask` implementation / `_prepare_encoder_hidden_states_mask` 实现
```python
    def _prepare_encoder_hidden_states_mask(
        self,
        batch,
        encoder_index: int,
        txt_seq_lens: list[int],
        text_seq_len: int,
        batch_size: int,
        *,
        negative: bool = False,
    ):
        """Return the text attention mask passed to the Qwen image DiT.

        Qwen image batches can contain prompts with different semantic text
        lengths after tokenization/postprocessing. The transformer still sees a
        padded `encoder_hidden_states` tensor with shape [batch, text_seq_len,
        dim], so we pass a [batch, text_seq_len] boolean mask to keep attention
        on real text tokens and ignore padding.

        If every request uses the full padded length, no mask is needed and this
        returns None. Otherwise, prefer the embedding-aligned mask stored by the
        text encoding stage. If that is unavailable, rebuild the same mask from
        `txt_seq_lens`: position j is valid for row i when
        `j < txt_seq_lens[i]`.
        """
        if all(seq_len == text_seq_len for seq_len in txt_seq_lens):
            return None

        masks_by_encoder = (
            batch.negative_prompt_embeds_mask if negative else batch.prompt_embeds_mask
        )
        if masks_by_encoder is not None and encoder_index < len(masks_by_encoder):
            mask = masks_by_encoder[encoder_index]
            if mask.shape != (batch_size, text_seq_len):
                raise ValueError(
                    "QwenImage text conditioning mask has shape "
                    f"{tuple(mask.shape)}, expected {(batch_size, text_seq_len)}."
                )
            return mask

        # TODO: cache positions by (device, text_seq_len) if this allocation shows up hot.
        positions = torch.arange(text_seq_len, device=batch.prompt_embeds[0].device)
        seq_lens = torch.tensor(
            txt_seq_lens,
            device=batch.prompt_embeds[0].device,
            dtype=torch.long,
        )
        return positions.unsqueeze(0) < seq_lens.unsqueeze(1)
```
**EN:** This block defines method `_prepare_encoder_hidden_states_mask` on `QwenImagePipelineConfig`. Return the text attention mask passed to the Qwen image DiT. Qwen image batches can contain prompts with different semantic text lengths after tokenization/postprocessing. Key calls include `all`, `torch.arange`, `torch.tensor`, `positions.unsqueeze`, and `seq_lens.unsqueeze`. The implementation branches on conditions. Parameters such as `batch`, `encoder_index`, `txt_seq_lens`, `text_seq_len`, and `batch_size` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `_prepare_encoder_hidden_states_mask`。 它用于准备encoder hidden states mask。 关键调用包括 `all`、`torch.arange`、`torch.tensor`、`positions.unsqueeze` 和 `seq_lens.unsqueeze`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`encoder_index`、`txt_seq_lens`、`text_seq_len` 和 `batch_size` 等参数驱动。

### Lines 429-432: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_cond_kwargs(
            batch, batch.prompt_embeds, rotary_emb, device, dtype, negative=False
        )
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `QwenImagePipelineConfig`. It prepares pos cond kwargs. Key calls include `self._prepare_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self._prepare_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 434-442: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_cond_kwargs(
            batch,
            batch.negative_prompt_embeds,
            rotary_emb,
            device,
            dtype,
            negative=True,
        )
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `QwenImagePipelineConfig`. It prepares neg cond kwargs. Key calls include `self._prepare_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self._prepare_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 444-454: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        # unpack latents for qwen-image
        (
            latents,
            batch_size,
            channels,
            height,
            width,
        ) = self._unpad_and_unpack_latents(latents, batch)
        latents = latents.reshape(batch_size, channels // (2 * 2), 1, height, width)
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `QwenImagePipelineConfig`. It post-processes denoising loop. Key calls include `self._unpad_and_unpack_latents`, and `latents.reshape`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImagePipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `self._unpad_and_unpack_latents` 和 `latents.reshape`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 458-460: `QwenImageEditPipelineConfig` class overview / `QwenImageEditPipelineConfig` 类概览
```python
class QwenImageEditPipelineConfig(QwenImagePipelineConfig):
    """Configuration for the QwenImageEdit pipeline."""
```
**EN:** This block defines class `QwenImageEditPipelineConfig`. Configuration for the QwenImageEdit pipeline. It inherits from `QwenImagePipelineConfig`.
**CN:** 该代码块定义了类 `QwenImageEditPipelineConfig`。 它用于封装 qwen image edit pipeline config 相关行为。 它继承自 `QwenImagePipelineConfig`。

### Lines 461-464: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2I
    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (qwen_image_edit_postprocess_text,)
    )
```
**EN:** This block gathers supporting statements inside `QwenImageEditPipelineConfig`. It updates names such as `task_type`, and `postprocess_text_funcs`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `QwenImageEditPipelineConfig` 内部的辅助语句。 它会更新 `task_type` 和 `postprocess_text_funcs` 等名称。 代码会与 `field` 协同工作。

### Lines 466-525: `_prepare_edit_cond_kwargs` implementation / `_prepare_edit_cond_kwargs` 实现
```python
    def _prepare_edit_cond_kwargs(
        self, batch, prompt_embeds, rotary_emb, device, dtype, *, negative=False
    ):
        batch_size = batch.latents.shape[0]
        assert batch_size == 1
        text_seq_len = prompt_embeds[0].shape[1]
        height = batch.height
        width = batch.width
        image_size = batch.original_condition_image_size
        edit_width, edit_height, _ = calculate_dimensions(
            1024 * 1024, image_size[0] / image_size[1]
        )
        vae_scale_factor = self.get_vae_scale_factor()

        img_shapes = [
            [
                (
                    1,
                    height // vae_scale_factor // 2,
                    width // vae_scale_factor // 2,
                ),
                (
                    1,
                    edit_height // vae_scale_factor // 2,
                    edit_width // vae_scale_factor // 2,
                ),
            ],
        ] * batch_size
        txt_seq_lens, encoder_hidden_states_mask = self._prepare_text_conditioning(
            batch, 0, text_seq_len, batch_size, negative=negative
        )

        if rotary_emb is None:
            cond_kwargs = {
                "img_shapes": img_shapes,
                "txt_seq_lens": txt_seq_lens,
                "freqs_cis": None,
                "encoder_hidden_states_mask": encoder_hidden_states_mask,
            }
            return cond_kwargs

        freqs_cis = QwenImagePipelineConfig.get_freqs_cis(
            img_shapes, txt_seq_lens, rotary_emb, device, dtype
        )

        # perform sp shard on noisy image tokens
        noisy_img_seq_len = (
            1 * (height // vae_scale_factor // 2) * (width // vae_scale_factor // 2)
        )

        img_cache, txt_cache = _shard_qwen_edit_freqs_cis_for_sp(
            freqs_cis, noisy_img_seq_len, device
        )
        cond_kwargs = {
            "txt_seq_lens": txt_seq_lens,
            "freqs_cis": (img_cache, txt_cache),
            "img_shapes": img_shapes,
            "encoder_hidden_states_mask": encoder_hidden_states_mask,
        }
        return cond_kwargs
```
**EN:** This block defines method `_prepare_edit_cond_kwargs` on `QwenImageEditPipelineConfig`. It prepares edit cond kwargs. Key calls include `calculate_dimensions`, `self.get_vae_scale_factor`, `self._prepare_text_conditioning`, `QwenImagePipelineConfig.get_freqs_cis`, and `_shard_qwen_edit_freqs_cis_for_sp`. The implementation branches on conditions. Parameters such as `batch`, `prompt_embeds`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `_prepare_edit_cond_kwargs`。 它用于准备edit cond kwargs。 关键调用包括 `calculate_dimensions`、`self.get_vae_scale_factor`、`self._prepare_text_conditioning`、`QwenImagePipelineConfig.get_freqs_cis` 和 `_shard_qwen_edit_freqs_cis_for_sp`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`prompt_embeds`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 527-533: `preprocess_condition_image` implementation / `preprocess_condition_image` 实现
```python
    def preprocess_condition_image(
        self, image, target_width, target_height, _vae_image_processor
    ):
        return resize(image, target_height, target_width, resize_mode="default"), (
            target_width,
            target_height,
        )
```
**EN:** This block defines method `preprocess_condition_image` on `QwenImageEditPipelineConfig`. It handles preprocess condition image logic. Key calls include `resize`. Parameters such as `image`, `target_width`, `target_height`, and `_vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `preprocess_condition_image`。 它用于处理 preprocess condition image 相关逻辑。 关键调用包括 `resize`。 本段逻辑主要由 `image`、`target_width`、`target_height` 和 `_vae_image_processor` 等参数驱动。

### Lines 535-560: `postprocess_image_latent` implementation / `postprocess_image_latent` 实现
```python
    def postprocess_image_latent(self, latent_condition, batch):
        batch_size = batch.batch_size
        if batch_size > latent_condition.shape[0]:
            if batch_size % latent_condition.shape[0] == 0:
                # expand init_latents for batch_size
                additional_image_per_prompt = batch_size // latent_condition.shape[0]
                image_latents = latent_condition.repeat(
                    additional_image_per_prompt, 1, 1, 1
                )
            else:
                raise ValueError(
                    f"Cannot duplicate `image` of batch size {latent_condition.shape[0]} to {batch_size} text prompts."
                )
        else:
            image_latents = latent_condition
        image_latent_height, image_latent_width = image_latents.shape[3:]
        num_channels_latents = self.dit_config.arch_config.in_channels // 4
        image_latents = _pack_latents(
            image_latents,
            batch_size,
            num_channels_latents,
            image_latent_height,
            image_latent_width,
        )

        return image_latents
```
**EN:** This block defines method `postprocess_image_latent` on `QwenImageEditPipelineConfig`. It handles postprocess image latent logic. Key calls include `_pack_latents`, `latent_condition.repeat`, and `ValueError`. The implementation branches on conditions. Parameters such as `latent_condition`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `postprocess_image_latent`。 它用于处理 postprocess image latent 相关逻辑。 关键调用包括 `_pack_latents`、`latent_condition.repeat` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `latent_condition` 和 `batch` 等参数驱动。

### Lines 562-565: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_edit_cond_kwargs(
            batch, batch.prompt_embeds, rotary_emb, device, dtype, negative=False
        )
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `QwenImageEditPipelineConfig`. It prepares pos cond kwargs. Key calls include `self._prepare_edit_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self._prepare_edit_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 567-575: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_edit_cond_kwargs(
            batch,
            batch.negative_prompt_embeds,
            rotary_emb,
            device,
            dtype,
            negative=True,
        )
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `QwenImageEditPipelineConfig`. It prepares neg cond kwargs. Key calls include `self._prepare_edit_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self._prepare_edit_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 577-581: `calculate_condition_image_size` implementation / `calculate_condition_image_size` 实现
```python
    def calculate_condition_image_size(self, image, width, height) -> tuple[int, int]:
        calculated_width, calculated_height, _ = calculate_dimensions(
            1024 * 1024, width / height
        )
        return calculated_width, calculated_height
```
**EN:** This block defines method `calculate_condition_image_size` on `QwenImageEditPipelineConfig`. It handles calculate condition image size logic. Key calls include `calculate_dimensions`. Parameters such as `image`, `width`, and `height` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `calculate_condition_image_size`。 它用于处理 calculate condition image size 相关逻辑。 关键调用包括 `calculate_dimensions`。 本段逻辑主要由 `image`、`width` 和 `height` 等参数驱动。

### Lines 583-586: `slice_noise_pred` implementation / `slice_noise_pred` 实现
```python
    def slice_noise_pred(self, noise, latents):
        # remove noise over input image
        noise = noise[:, : latents.size(1)]
        return noise
```
**EN:** This block defines method `slice_noise_pred` on `QwenImageEditPipelineConfig`. It handles slice noise pred logic. Key calls include `latents.size`. Parameters such as `noise`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPipelineConfig` 的方法 `slice_noise_pred`。 它用于处理 slice noise pred 相关逻辑。 关键调用包括 `latents.size`。 本段逻辑主要由 `noise` 和 `latents` 等参数驱动。

### Lines 589-590: supporting statements / 辅助语句
```python
CONDITION_IMAGE_SIZE = 384 * 384
VAE_IMAGE_SIZE = 1024 * 1024
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `CONDITION_IMAGE_SIZE`, and `VAE_IMAGE_SIZE`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `CONDITION_IMAGE_SIZE` 和 `VAE_IMAGE_SIZE` 等名称。

### Lines 594-594: `QwenImageEditPlusPipelineConfig` class overview / `QwenImageEditPlusPipelineConfig` 类概览
```python
class QwenImageEditPlusPipelineConfig(QwenImageEditPipelineConfig):
```
**EN:** This block defines class `QwenImageEditPlusPipelineConfig`. It encapsulates qwen image edit plus pipeline config behavior. It inherits from `QwenImageEditPipelineConfig`.
**CN:** 该代码块定义了类 `QwenImageEditPlusPipelineConfig`。 它用于封装 qwen image edit plus pipeline config 相关行为。 它继承自 `QwenImageEditPipelineConfig`。

### Lines 595-595: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2I
```
**EN:** This block gathers supporting statements inside `QwenImageEditPlusPipelineConfig`. It updates names such as `task_type`.
**CN:** 该代码块汇集了位于 `QwenImageEditPlusPipelineConfig` 内部的辅助语句。 它会更新 `task_type` 等名称。

### Lines 597-610: `_get_condition_image_sizes` implementation / `_get_condition_image_sizes` 实现
```python
    def _get_condition_image_sizes(self, batch) -> list[tuple[int, int]]:
        image = batch.condition_image
        if not isinstance(image, list):
            image = [image]

        condition_image_sizes = []
        for img in image:
            image_width, image_height = img.size
            edit_width, edit_height, _ = calculate_dimensions(
                VAE_IMAGE_SIZE, image_width / image_height
            )
            condition_image_sizes.append((edit_width, edit_height))

        return condition_image_sizes
```
**EN:** This block defines method `_get_condition_image_sizes` on `QwenImageEditPlusPipelineConfig`. It retrieves condition image sizes. Key calls include `isinstance`, `calculate_dimensions`, and `condition_image_sizes.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `_get_condition_image_sizes`。 它用于获取condition image sizes。 关键调用包括 `isinstance`、`calculate_dimensions` 和 `condition_image_sizes.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 612-639: `prepare_image_processor_kwargs` implementation / `prepare_image_processor_kwargs` 实现
```python
    def prepare_image_processor_kwargs(self, batch, neg=False) -> dict:
        prompt = batch.prompt if not neg else batch.negative_prompt
        if not prompt:
            return {}

        prompt_list = _normalize_prompt_list(prompt)
        image_list = _normalize_image_list(batch.condition_image)
        per_prompt_images = _resolve_qwen_edit_per_prompt_images(
            prompt_list, image_list
        )

        prompt_template_encode = (
            "<|im_start|>system\nDescribe the key features of the input image "
            "(color, shape, size, texture, objects, background), then explain how "
            "the user's text instruction should alter or modify the image. Generate "
            "a new image that meets the user's requirements while maintaining "
            "consistency with the original input where appropriate.<|im_end|>\n"
            "<|im_start|>user\n{}<|im_end|>\n"
            "<|im_start|>assistant\n"
        )
        txt = [
            prompt_template_encode.format(
                _build_qwen_edit_image_prompt(len(prompt_images)) + prompt_text
            )
            for prompt_text, prompt_images in zip(prompt_list, per_prompt_images)
        ]

        return dict(text=txt, padding=True, per_prompt_images=per_prompt_images)
```
**EN:** This block defines method `prepare_image_processor_kwargs` on `QwenImageEditPlusPipelineConfig`. It prepares image processor kwargs. Key calls include `_normalize_prompt_list`, `_normalize_image_list`, `_resolve_qwen_edit_per_prompt_images`, `dict`, and `prompt_template_encode.format`. The implementation branches on conditions. Parameters such as `batch`, and `neg` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `prepare_image_processor_kwargs`。 它用于准备image processor kwargs。 关键调用包括 `_normalize_prompt_list`、`_normalize_image_list`、`_resolve_qwen_edit_per_prompt_images`、`dict` 和 `prompt_template_encode.format`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `neg` 等参数驱动。

### Lines 641-642: `prepare_calculated_size` implementation / `prepare_calculated_size` 实现
```python
    def prepare_calculated_size(self, image):
        return self.calculate_vae_image_size(image, image.width, image.height)
```
**EN:** This block defines method `prepare_calculated_size` on `QwenImageEditPlusPipelineConfig`. It prepares calculated size. Key calls include `self.calculate_vae_image_size`. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `prepare_calculated_size`。 它用于准备calculated size。 关键调用包括 `self.calculate_vae_image_size`。 本段逻辑主要由 `image` 等参数驱动。

### Lines 644-650: `resize_condition_image` implementation / `resize_condition_image` 实现
```python
    def resize_condition_image(self, images, target_width, target_height):
        if not isinstance(images, list):
            images = [images]
        new_images = []
        for img, width, height in zip(images, target_width, target_height):
            new_images.append(resize(img, height, width, resize_mode="default"))
        return new_images
```
**EN:** This block defines method `resize_condition_image` on `QwenImageEditPlusPipelineConfig`. It handles resize condition image logic. Key calls include `zip`, `isinstance`, `new_images.append`, and `resize`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `images`, `target_width`, and `target_height` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `resize_condition_image`。 它用于处理 resize condition image 相关逻辑。 关键调用包括 `zip`、`isinstance`、`new_images.append` 和 `resize`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `images`、`target_width` 和 `target_height` 等参数驱动。

### Lines 652-656: `calculate_condition_image_size` implementation / `calculate_condition_image_size` 实现
```python
    def calculate_condition_image_size(self, image, width, height) -> tuple[int, int]:
        calculated_width, calculated_height, _ = calculate_dimensions(
            CONDITION_IMAGE_SIZE, width / height
        )
        return calculated_width, calculated_height
```
**EN:** This block defines method `calculate_condition_image_size` on `QwenImageEditPlusPipelineConfig`. It handles calculate condition image size logic. Key calls include `calculate_dimensions`. Parameters such as `image`, `width`, and `height` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `calculate_condition_image_size`。 它用于处理 calculate condition image size 相关逻辑。 关键调用包括 `calculate_dimensions`。 本段逻辑主要由 `image`、`width` 和 `height` 等参数驱动。

### Lines 658-662: `calculate_vae_image_size` implementation / `calculate_vae_image_size` 实现
```python
    def calculate_vae_image_size(self, image, width, height) -> tuple[int, int]:
        calculated_width, calculated_height, _ = calculate_dimensions(
            VAE_IMAGE_SIZE, width / height
        )
        return calculated_width, calculated_height
```
**EN:** This block defines method `calculate_vae_image_size` on `QwenImageEditPlusPipelineConfig`. It handles calculate vae image size logic. Key calls include `calculate_dimensions`. Parameters such as `image`, `width`, and `height` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `calculate_vae_image_size`。 它用于处理 calculate vae image size 相关逻辑。 关键调用包括 `calculate_dimensions`。 本段逻辑主要由 `image`、`width` 和 `height` 等参数驱动。

### Lines 664-675: `preprocess_vae_image` implementation / `preprocess_vae_image` 实现
```python
    def preprocess_vae_image(self, batch, vae_image_processor):
        if not isinstance(batch.condition_image, list):
            batch.condition_image = [batch.condition_image]
        new_images = []
        vae_image_sizes = []
        for img in batch.condition_image:
            width, height = self.calculate_vae_image_size(img, img.width, img.height)
            new_images.append(vae_image_processor.preprocess(img, height, width))
            vae_image_sizes.append((width, height))
        batch.vae_image = new_images
        batch.vae_image_sizes = vae_image_sizes
        return batch
```
**EN:** This block defines method `preprocess_vae_image` on `QwenImageEditPlusPipelineConfig`. It handles preprocess vae image logic. Key calls include `isinstance`, `self.calculate_vae_image_size`, `new_images.append`, `vae_image_sizes.append`, and `vae_image_processor.preprocess`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `batch`, and `vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `preprocess_vae_image`。 它用于处理 preprocess vae image 相关逻辑。 关键调用包括 `isinstance`、`self.calculate_vae_image_size`、`new_images.append`、`vae_image_sizes.append` 和 `vae_image_processor.preprocess`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `batch` 和 `vae_image_processor` 等参数驱动。

### Lines 677-722: `_prepare_edit_cond_kwargs` implementation / `_prepare_edit_cond_kwargs` 实现
```python
    def _prepare_edit_cond_kwargs(
        self, batch, prompt_embeds, rotary_emb, device, dtype, *, negative=False
    ):
        batch_size = batch.latents.shape[0]
        assert batch_size == 1
        text_seq_len = prompt_embeds[0].shape[1]
        height = batch.height
        width = batch.width

        vae_scale_factor = self.get_vae_scale_factor()

        img_shapes = [
            [
                (1, height // vae_scale_factor // 2, width // vae_scale_factor // 2),
                *[
                    (
                        1,
                        vae_height // vae_scale_factor // 2,
                        vae_width // vae_scale_factor // 2,
                    )
                    for vae_width, vae_height in batch.vae_image_sizes
                ],
            ],
        ] * batch_size
        txt_seq_lens, encoder_hidden_states_mask = self._prepare_text_conditioning(
            batch, 0, text_seq_len, batch_size, negative=negative
        )

        freqs_cis = QwenImageEditPlusPipelineConfig.get_freqs_cis(
            img_shapes, txt_seq_lens, rotary_emb, device, dtype
        )

        # perform sp shard on noisy image tokens
        noisy_img_seq_len = (
            1 * (height // vae_scale_factor // 2) * (width // vae_scale_factor // 2)
        )

        cond_kwargs = {
            "txt_seq_lens": txt_seq_lens,
            "freqs_cis": _shard_qwen_edit_freqs_cis_for_sp(
                freqs_cis, noisy_img_seq_len, device
            ),
            "img_shapes": img_shapes,
            "encoder_hidden_states_mask": encoder_hidden_states_mask,
        }
        return cond_kwargs
```
**EN:** This block defines method `_prepare_edit_cond_kwargs` on `QwenImageEditPlusPipelineConfig`. It prepares edit cond kwargs. Key calls include `self.get_vae_scale_factor`, `self._prepare_text_conditioning`, `QwenImageEditPlusPipelineConfig.get_freqs_cis`, and `_shard_qwen_edit_freqs_cis_for_sp`. Parameters such as `batch`, `prompt_embeds`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageEditPlusPipelineConfig` 的方法 `_prepare_edit_cond_kwargs`。 它用于准备edit cond kwargs。 关键调用包括 `self.get_vae_scale_factor`、`self._prepare_text_conditioning`、`QwenImageEditPlusPipelineConfig.get_freqs_cis` 和 `_shard_qwen_edit_freqs_cis_for_sp`。 本段逻辑主要由 `batch`、`prompt_embeds`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 726-726: `QwenImageEditPlus_2511_PipelineConfig` class overview / `QwenImageEditPlus_2511_PipelineConfig` 类概览
```python
class QwenImageEditPlus_2511_PipelineConfig(QwenImageEditPlusPipelineConfig):
```
**EN:** This block defines class `QwenImageEditPlus_2511_PipelineConfig`. It encapsulates qwen image edit plus 2511 pipeline config behavior. It inherits from `QwenImageEditPlusPipelineConfig`.
**CN:** 该代码块定义了类 `QwenImageEditPlus_2511_PipelineConfig`。 它用于封装 qwen image edit plus 2511 pipeline config 相关行为。 它继承自 `QwenImageEditPlusPipelineConfig`。

### Lines 727-727: supporting statements / 辅助语句
```python
    dit_config: DiTConfig = field(default_factory=QwenImageEditPlus_2511_DitConfig)
```
**EN:** This block gathers supporting statements inside `QwenImageEditPlus_2511_PipelineConfig`. It updates names such as `dit_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `QwenImageEditPlus_2511_PipelineConfig` 内部的辅助语句。 它会更新 `dit_config` 等名称。 代码会与 `field` 协同工作。

### Lines 731-731: `QwenImageLayeredPipelineConfig` class overview / `QwenImageLayeredPipelineConfig` 类概览
```python
class QwenImageLayeredPipelineConfig(QwenImageEditPipelineConfig):
```
**EN:** This block defines class `QwenImageLayeredPipelineConfig`. It encapsulates qwen image layered pipeline config behavior. It inherits from `QwenImageEditPipelineConfig`.
**CN:** 该代码块定义了类 `QwenImageLayeredPipelineConfig`。 它用于封装 qwen image layered pipeline config 相关行为。 它继承自 `QwenImageEditPipelineConfig`。

### Lines 732-733: supporting statements / 辅助语句
```python
    resolution: int = 640
    vae_precision: str = "bf16"
```
**EN:** This block gathers supporting statements inside `QwenImageLayeredPipelineConfig`. It updates names such as `resolution`, and `vae_precision`.
**CN:** 该代码块汇集了位于 `QwenImageLayeredPipelineConfig` 内部的辅助语句。 它会更新 `resolution` 和 `vae_precision` 等名称。

### Lines 735-743: `postprocess_cfg_noise` implementation / `postprocess_cfg_noise` 实现
```python
    def postprocess_cfg_noise(
        self,
        batch,
        noise_pred: torch.Tensor,
        noise_pred_cond: torch.Tensor,
    ) -> torch.Tensor:
        if not batch.cfg_normalize:
            return noise_pred
        return super().postprocess_cfg_noise(batch, noise_pred, noise_pred_cond)
```
**EN:** This block defines method `postprocess_cfg_noise` on `QwenImageLayeredPipelineConfig`. It handles postprocess cfg noise logic. Key calls include `super.postprocess_cfg_noise`, and `super`. The implementation branches on conditions. Parameters such as `batch`, `noise_pred`, and `noise_pred_cond` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageLayeredPipelineConfig` 的方法 `postprocess_cfg_noise`。 它用于处理 postprocess cfg noise 相关逻辑。 关键调用包括 `super.postprocess_cfg_noise` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`noise_pred` 和 `noise_pred_cond` 等参数驱动。

### Lines 745-783: `_prepare_edit_cond_kwargs` implementation / `_prepare_edit_cond_kwargs` 实现
```python
    def _prepare_edit_cond_kwargs(
        self, batch, prompt_embeds, rotary_emb, device, dtype, *, negative=False
    ):
        batch_size = batch.latents.shape[0]
        assert batch_size == 1
        text_seq_len = prompt_embeds[0].shape[1]
        height = batch.height
        width = batch.width

        vae_scale_factor = self.get_vae_scale_factor()

        img_shapes = batch.img_shapes
        txt_seq_lens, encoder_hidden_states_mask = self._prepare_text_conditioning(
            batch, 0, text_seq_len, batch_size, negative=negative
        )

        freqs_cis = QwenImageEditPlusPipelineConfig.get_freqs_cis(
            img_shapes, txt_seq_lens, rotary_emb, device, dtype
        )

        # perform sp shard on noisy image tokens
        noisy_img_seq_len = (
            1 * (height // vae_scale_factor // 2) * (width // vae_scale_factor // 2)
        )

        img_cache, txt_cache = freqs_cis
        noisy_img_cache = shard_rotary_emb_for_sp(img_cache[:noisy_img_seq_len, :])
        img_cache = torch.cat(
            [noisy_img_cache, img_cache[noisy_img_seq_len:, :]], dim=0
        ).to(device=device)

        cond_kwargs = {
            "txt_seq_lens": txt_seq_lens,
            "img_shapes": img_shapes,
            "freqs_cis": (img_cache, txt_cache),
            "additional_t_cond": torch.tensor([0], device=device, dtype=torch.long),
            "encoder_hidden_states_mask": encoder_hidden_states_mask,
        }
        return cond_kwargs
```
**EN:** This block defines method `_prepare_edit_cond_kwargs` on `QwenImageLayeredPipelineConfig`. It prepares edit cond kwargs. Key calls include `self.get_vae_scale_factor`, `self._prepare_text_conditioning`, `QwenImageEditPlusPipelineConfig.get_freqs_cis`, `shard_rotary_emb_for_sp`, and `torch.cat.to`. Parameters such as `batch`, `prompt_embeds`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageLayeredPipelineConfig` 的方法 `_prepare_edit_cond_kwargs`。 它用于准备edit cond kwargs。 关键调用包括 `self.get_vae_scale_factor`、`self._prepare_text_conditioning`、`QwenImageEditPlusPipelineConfig.get_freqs_cis`、`shard_rotary_emb_for_sp` 和 `torch.cat.to`。 本段逻辑主要由 `batch`、`prompt_embeds`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 785-804: `_unpad_and_unpack_latents` implementation / `_unpad_and_unpack_latents` 实现
```python
    def _unpad_and_unpack_latents(self, latents, batch):
        vae_scale_factor = self.get_vae_scale_factor()
        channels = self.dit_config.arch_config.in_channels
        batch_size = latents.shape[0]
        layers = batch.num_frames

        height = 2 * (int(batch.height) // (vae_scale_factor * 2))
        width = 2 * (int(batch.width) // (vae_scale_factor * 2))

        latents = maybe_unpad_latents(latents, batch)
        latents = latents.view(
            batch_size, layers + 1, height // 2, width // 2, channels // 4, 2, 2
        )
        latents = latents.permute(0, 1, 4, 2, 5, 3, 6)

        latents = latents.reshape(
            batch_size, layers + 1, channels // (2 * 2), height, width
        )
        latents = latents.permute(0, 2, 1, 3, 4)  # (b, c, f, h, w)
        return latents, batch_size, channels, height, width
```
**EN:** This block defines method `_unpad_and_unpack_latents` on `QwenImageLayeredPipelineConfig`. It handles unpad and unpack latents logic. Key calls include `self.get_vae_scale_factor`, `maybe_unpad_latents`, `latents.view`, `latents.permute`, and `latents.reshape`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageLayeredPipelineConfig` 的方法 `_unpad_and_unpack_latents`。 它用于处理 unpad and unpack latents 相关逻辑。 关键调用包括 `self.get_vae_scale_factor`、`maybe_unpad_latents`、`latents.view`、`latents.permute` 和 `latents.reshape`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 806-807: `allow_set_num_frames` implementation / `allow_set_num_frames` 实现
```python
    def allow_set_num_frames(self):
        return True
```
**EN:** This block defines method `allow_set_num_frames` on `QwenImageLayeredPipelineConfig`. It handles allow set num frames logic.
**CN:** 该代码块定义了 `QwenImageLayeredPipelineConfig` 的方法 `allow_set_num_frames`。 它用于处理 allow set num frames 相关逻辑。

### Lines 809-822: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        # unpack latents for qwen-image
        (
            latents,
            batch_size,
            channels,
            height,
            width,
        ) = self._unpad_and_unpack_latents(latents, batch)
        b, c, f, h, w = latents.shape
        latents = latents[:, :, 1:]  # remove the first frame as it is the origin input
        latents = latents.permute(0, 2, 1, 3, 4).view(-1, c, 1, h, w)
        # latents = latents.reshape(batch_size, channels // (2 * 2), 1, height, width)
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `QwenImageLayeredPipelineConfig`. It post-processes denoising loop. Key calls include `self._unpad_and_unpack_latents`, `latents.permute.view`, and `latents.permute`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `QwenImageLayeredPipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `self._unpad_and_unpack_latents`、`latents.permute.view` 和 `latents.permute`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

## Key Concepts / 关键概念
- `_extract_masked_hidden`: Top-level function that handles extract masked hidden logic. / 顶层函数，用于处理 extract masked hidden 相关逻辑。
- `qwen_image_preprocess_text`: Top-level function that handles qwen image preprocess text logic. / 顶层函数，用于处理 qwen image preprocess text 相关逻辑。
- `qwen_image_postprocess_text`: Postprocess Qwen text embeddings. / 顶层函数，用于处理 qwen image postprocess text 相关逻辑。
- `qwen_image_edit_postprocess_text`: Top-level function that handles qwen image edit postprocess text logic. / 顶层函数，用于处理 qwen image edit postprocess text 相关逻辑。
- `_normalize_prompt_list`: Top-level function that handles normalize prompt list logic. / 顶层函数，用于处理 normalize prompt list 相关逻辑。
- `_normalize_image_list`: Top-level function that handles normalize image list logic. / 顶层函数，用于处理 normalize image list 相关逻辑。
- `_build_qwen_edit_image_prompt`: Top-level function that builds qwen edit image prompt. / 顶层函数，用于构建qwen edit image prompt。
- `_resolve_qwen_edit_per_prompt_images`: Top-level function that resolves qwen edit per prompt images. / 顶层函数，用于解析qwen edit per prompt images。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.qwenimage`, `sglang.multimodal_gen.configs.models.encoders.qwen_image`, `sglang.multimodal_gen.configs.models.vaes.qwenimage`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.post_training.pipeline_configs`, `sglang.multimodal_gen.runtime.models.vision_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 822
