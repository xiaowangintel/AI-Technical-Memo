# ernie_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/ernie_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `ErnieImagePipelineConfig`, `ernie_image_postprocess_text`, and `_patchify_latents`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `ErnieImagePipelineConfig`、`ernie_image_postprocess_text` 和 `_patchify_latents` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-17: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Callable

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.ernie_image import ErnieImageDitConfig
from sglang.multimodal_gen.configs.models.encoders.mistral3 import Mistral3EncoderConfig
from sglang.multimodal_gen.configs.models.vaes.ernie_image import ErnieImageVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ImagePipelineConfig,
    ModelTaskType,
    shard_rotary_emb_for_sp,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.ernie_image`, and `sglang.multimodal_gen.configs.models.encoders.mistral3`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.ernie_image` 和 `sglang.multimodal_gen.configs.models.encoders.mistral3`。这些依赖为后续实现提供所需符号。

### Lines 19-19: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 22-24: `ernie_image_postprocess_text` implementation / `ernie_image_postprocess_text` 实现
```python
def ernie_image_postprocess_text(outputs, _text_inputs, hidden_layer_index=-2):
    hidden_states = outputs.hidden_states[hidden_layer_index]
    return hidden_states
```
**EN:** This block defines function `ernie_image_postprocess_text`. It handles ernie image postprocess text logic. Parameters such as `outputs`, `_text_inputs`, and `hidden_layer_index` drive the behavior in this section.
**CN:** 该代码块定义了函数 `ernie_image_postprocess_text`。 它用于处理 ernie image postprocess text 相关逻辑。 本段逻辑主要由 `outputs`、`_text_inputs` 和 `hidden_layer_index` 等参数驱动。

### Lines 27-31: `_patchify_latents` implementation / `_patchify_latents` 实现
```python
def _patchify_latents(latents: torch.Tensor) -> torch.Tensor:
    b, c, h, w = latents.shape
    latents = latents.view(b, c, h // 2, 2, w // 2, 2)
    latents = latents.permute(0, 1, 3, 5, 2, 4).reshape(b, c * 4, h // 2, w // 2)
    return latents
```
**EN:** This block defines function `_patchify_latents`. It handles patchify latents logic. Key calls include `latents.view`, `latents.permute.reshape`, and `latents.permute`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patchify_latents`。 它用于处理 patchify latents 相关逻辑。 关键调用包括 `latents.view`、`latents.permute.reshape` 和 `latents.permute`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 34-38: `_unpatchify_latents` implementation / `_unpatchify_latents` 实现
```python
def _unpatchify_latents(latents: torch.Tensor) -> torch.Tensor:
    b, c, h, w = latents.shape
    latents = latents.reshape(b, c // 4, 2, 2, h, w)
    latents = latents.permute(0, 1, 4, 2, 5, 3).reshape(b, c // 4, h * 2, w * 2)
    return latents
```
**EN:** This block defines function `_unpatchify_latents`. It handles unpatchify latents logic. Key calls include `latents.reshape`, `latents.permute.reshape`, and `latents.permute`. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_unpatchify_latents`。 它用于处理 unpatchify latents 相关逻辑。 关键调用包括 `latents.reshape`、`latents.permute.reshape` 和 `latents.permute`。 本段逻辑主要由 `latents` 等参数驱动。

### Lines 42-44: `ErnieImagePipelineConfig` class overview / `ErnieImagePipelineConfig` 类概览
```python
class ErnieImagePipelineConfig(ImagePipelineConfig):
    """Configuration for the ErnieImage text-to-image pipeline."""
```
**EN:** This block defines class `ErnieImagePipelineConfig`. Configuration for the ErnieImage text-to-image pipeline. It inherits from `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `ErnieImagePipelineConfig`。 它用于封装 ernie image pipeline config 相关行为。 它继承自 `ImagePipelineConfig`。

### Lines 45-81: supporting statements / 辅助语句
```python
    should_use_guidance: bool = False
    task_type: ModelTaskType = ModelTaskType.T2I

    pe_model_max_length: int = None

    vae_tiling: bool = False
    vae_sp: bool = False

    dit_config: DiTConfig = field(default_factory=ErnieImageDitConfig)
    vae_config: VAEConfig = field(default_factory=ErnieImageVAEConfig)

    enable_autocast: bool = False

    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Mistral3EncoderConfig(),)
    )

    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))

    preprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (None,)
    )

    postprocess_text_funcs: tuple[Callable, ...] = field(
        default_factory=lambda: (ernie_image_postprocess_text,)
    )

    text_encoder_extra_args: list[dict] = field(
        default_factory=lambda: [
            dict(
                padding=False,
                truncation=True,
                max_length=None,
                add_special_tokens=True,
            ),
        ]
    )
```
**EN:** This block gathers supporting statements inside `ErnieImagePipelineConfig`. It updates names such as `should_use_guidance`, `task_type`, `pe_model_max_length`, `vae_tiling`, `vae_sp`, and `dit_config`. The code collaborates with `field`, `Mistral3EncoderConfig`, and `dict`.
**CN:** 该代码块汇集了位于 `ErnieImagePipelineConfig` 内部的辅助语句。 它会更新 `should_use_guidance`、`task_type`、`pe_model_max_length`、`vae_tiling`、`vae_sp` 和 `dit_config` 等名称。 代码会与 `field`、`Mistral3EncoderConfig` 和 `dict` 协同工作。

### Lines 83-101: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompt: list[str], tokenizer, tok_kwargs) -> dict:
        max_length = tok_kwargs.get("max_length")
        if max_length is not None:
            check = tokenizer(
                prompt,
                truncation=False,
                return_tensors="pt",
                add_special_tokens=tok_kwargs.get("add_special_tokens", True),
            )
            for i, ids in enumerate(check["input_ids"]):
                if ids.shape[-1] > max_length:
                    logger.warning(
                        "Prompt #%d has %d tokens, exceeds max_length=%d. "
                        "The tail will be silently truncated.",
                        i,
                        ids.shape[-1],
                        max_length,
                    )
        return tokenizer(prompt, **tok_kwargs)
```
**EN:** This block defines method `tokenize_prompt` on `ErnieImagePipelineConfig`. It handles tokenize prompt logic. Key calls include `tok_kwargs.get`, `tokenizer`, `enumerate`, and `logger.warning`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `prompt`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tok_kwargs.get`、`tokenizer`、`enumerate` 和 `logger.warning`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `prompt`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 103-104: `prepare_sigmas` implementation / `prepare_sigmas` 实现
```python
    def prepare_sigmas(self, sigmas, num_inference_steps):
        return self._prepare_sigmas(sigmas, num_inference_steps)
```
**EN:** This block defines method `prepare_sigmas` on `ErnieImagePipelineConfig`. It prepares sigmas. Key calls include `self._prepare_sigmas`. Parameters such as `sigmas`, and `num_inference_steps` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `prepare_sigmas`。 它用于准备sigmas。 关键调用包括 `self._prepare_sigmas`。 本段逻辑主要由 `sigmas` 和 `num_inference_steps` 等参数驱动。

### Lines 106-107: `get_vae_scale_factor` implementation / `get_vae_scale_factor` 实现
```python
    def get_vae_scale_factor(self):
        return 16
```
**EN:** This block defines method `get_vae_scale_factor` on `ErnieImagePipelineConfig`. It retrieves vae scale factor.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `get_vae_scale_factor`。 它用于获取vae scale factor。

### Lines 109-115: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        vae_scale_factor = self.get_vae_scale_factor()
        latent_h = batch.height // vae_scale_factor
        latent_w = batch.width // vae_scale_factor
        num_channels = self.dit_config.arch_config.in_channels  # 128
        shape = (batch_size, num_channels, latent_h, latent_w)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `ErnieImagePipelineConfig`. It prepares latent shape. Key calls include `self.get_vae_scale_factor`. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 关键调用包括 `self.get_vae_scale_factor`。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 117-118: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        return latents
```
**EN:** This block defines method `maybe_pack_latents` on `ErnieImagePipelineConfig`. It handles maybe pack latents logic. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 120-126: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        if hasattr(vae, "bn") and vae.bn is not None:
            bn_mean = vae.bn.running_mean.view(1, -1, 1, 1).to(device, dtype)
            bn_var = vae.bn.running_var.view(1, -1, 1, 1).to(device, dtype)
            bn_std = torch.sqrt(bn_var + 1e-5)
            return 1.0 / bn_std, bn_mean
        return 1.0, None
```
**EN:** This block defines method `get_decode_scale_and_shift` on `ErnieImagePipelineConfig`. It retrieves decode scale and shift. Key calls include `hasattr`, `vae.bn.running_mean.view.to`, `vae.bn.running_var.view.to`, `torch.sqrt`, and `vae.bn.running_mean.view`. The implementation branches on conditions. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `hasattr`、`vae.bn.running_mean.view.to`、`vae.bn.running_var.view.to`、`torch.sqrt` 和 `vae.bn.running_mean.view`。 实现中包含条件分支。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 128-144: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    @staticmethod
    def get_freqs_cis(img_shapes, txt_seq_lens, rotary_emb, device, dtype):
        freqs = rotary_emb(img_shapes, txt_seq_lens, device=device)

        if isinstance(freqs, tuple) and len(freqs) == 2:
            img_freqs, txt_freqs = freqs
            img_cos = img_freqs.real.to(dtype=torch.float32).contiguous()
            img_sin = img_freqs.imag.to(dtype=torch.float32).contiguous()
            txt_cos = txt_freqs.real.to(dtype=torch.float32).contiguous()
            txt_sin = txt_freqs.imag.to(dtype=torch.float32).contiguous()
            img_cache = torch.cat([img_cos, img_sin], dim=-1)
            txt_cache = torch.cat([txt_cos, txt_sin], dim=-1)
            return img_cache, txt_cache

        cos = freqs.real.to(dtype=torch.float32).contiguous()
        sin = freqs.imag.to(dtype=torch.float32).contiguous()
        return torch.cat([cos, sin], dim=-1)
```
**EN:** This block defines method `get_freqs_cis` on `ErnieImagePipelineConfig`. It retrieves freqs cis. Key calls include `rotary_emb`, `freqs.real.to.contiguous`, `freqs.imag.to.contiguous`, `torch.cat`, and `isinstance`. The implementation branches on conditions. Parameters such as `img_shapes`, `txt_seq_lens`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `rotary_emb`、`freqs.real.to.contiguous`、`freqs.imag.to.contiguous`、`torch.cat` 和 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `img_shapes`、`txt_seq_lens`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 146-183: `_prepare_cond_kwargs` implementation / `_prepare_cond_kwargs` 实现
```python
    def _prepare_cond_kwargs(self, batch, prompt_embeds, rotary_emb, device, dtype):
        batch_size = prompt_embeds[0].shape[0]
        height = batch.height
        width = batch.width
        vae_scale_factor = self.get_vae_scale_factor()

        img_shapes = [
            [
                (
                    1,
                    height // vae_scale_factor,
                    width // vae_scale_factor,
                )
            ]
        ] * batch_size
        txt_seq_lens = [prompt_embeds[0].shape[1]]

        if rotary_emb is None:
            return {
                "img_shapes": img_shapes,
                "txt_seq_lens": txt_seq_lens,
                "freqs_cis": None,
            }

        freqs_cis = self.get_freqs_cis(
            img_shapes, txt_seq_lens, rotary_emb, device, dtype
        )

        if isinstance(freqs_cis, tuple):
            img_cache, txt_cache = freqs_cis
            img_cache = shard_rotary_emb_for_sp(img_cache)
            freqs_cis = (img_cache, txt_cache)

        return {
            "txt_seq_lens": txt_seq_lens,
            "freqs_cis": freqs_cis,
            "img_shapes": img_shapes,
        }
```
**EN:** This block defines method `_prepare_cond_kwargs` on `ErnieImagePipelineConfig`. It prepares cond kwargs. Key calls include `self.get_vae_scale_factor`, `self.get_freqs_cis`, `isinstance`, and `shard_rotary_emb_for_sp`. The implementation branches on conditions. Parameters such as `batch`, `prompt_embeds`, `rotary_emb`, `device`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `_prepare_cond_kwargs`。 它用于准备cond kwargs。 关键调用包括 `self.get_vae_scale_factor`、`self.get_freqs_cis`、`isinstance` 和 `shard_rotary_emb_for_sp`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`prompt_embeds`、`rotary_emb`、`device` 和 `dtype` 等参数驱动。

### Lines 185-188: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_cond_kwargs(
            batch, batch.prompt_embeds, rotary_emb, device, dtype
        )
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `ErnieImagePipelineConfig`. It prepares pos cond kwargs. Key calls include `self._prepare_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self._prepare_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 190-193: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return self._prepare_cond_kwargs(
            batch, batch.negative_prompt_embeds, rotary_emb, device, dtype
        )
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `ErnieImagePipelineConfig`. It prepares neg cond kwargs. Key calls include `self._prepare_cond_kwargs`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self._prepare_cond_kwargs`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 195-198: `_check_vae_has_bn` implementation / `_check_vae_has_bn` 实现
```python
    def _check_vae_has_bn(self, vae):
        if not hasattr(self, "_vae_has_bn_cache"):
            self._vae_has_bn_cache = hasattr(vae, "bn") and vae.bn is not None
        return self._vae_has_bn_cache
```
**EN:** This block defines method `_check_vae_has_bn` on `ErnieImagePipelineConfig`. It checks vae has bn. Key calls include `hasattr`. The implementation branches on conditions. Parameters such as `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `_check_vae_has_bn`。 它用于检查vae has bn。 关键调用包括 `hasattr`。 实现中包含条件分支。 本段逻辑主要由 `vae` 等参数驱动。

### Lines 200-203: `preprocess_decoding` implementation / `preprocess_decoding` 实现
```python
    def preprocess_decoding(self, latents, server_args=None, vae=None):
        if vae is not None and self._check_vae_has_bn(vae):
            latents = _unpatchify_latents(latents)
        return latents
```
**EN:** This block defines method `preprocess_decoding` on `ErnieImagePipelineConfig`. It handles preprocess decoding logic. Key calls include `self._check_vae_has_bn`, and `_unpatchify_latents`. The implementation branches on conditions. Parameters such as `latents`, `server_args`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `preprocess_decoding`。 它用于处理 preprocess decoding 相关逻辑。 关键调用包括 `self._check_vae_has_bn` 和 `_unpatchify_latents`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`server_args` 和 `vae` 等参数驱动。

### Lines 205-206: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `ErnieImagePipelineConfig`. It post-processes denoising loop. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ErnieImagePipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

## Key Concepts / 关键概念
- `ernie_image_postprocess_text`: Top-level function that handles ernie image postprocess text logic. / 顶层函数，用于处理 ernie image postprocess text 相关逻辑。
- `_patchify_latents`: Top-level function that handles patchify latents logic. / 顶层函数，用于处理 patchify latents 相关逻辑。
- `_unpatchify_latents`: Top-level function that handles unpatchify latents logic. / 顶层函数，用于处理 unpatchify latents 相关逻辑。
- `ErnieImagePipelineConfig`: Configuration for the ErnieImage text-to-image pipeline. / 核心类，用于封装 ernie image pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.ernie_image`, `sglang.multimodal_gen.configs.models.encoders.mistral3`, `sglang.multimodal_gen.configs.models.vaes.ernie_image`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 206
