# glm_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/glm_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `GlmImagePipelineConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `GlmImagePipelineConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

import torch
from diffusers.image_processor import VaeImageProcessor

from sglang.multimodal_gen.configs.models import DiTConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.glmimage import GlmImageDitConfig
from sglang.multimodal_gen.configs.models.encoders.base import EncoderConfig
from sglang.multimodal_gen.configs.models.encoders.t5 import T5Config
from sglang.multimodal_gen.configs.models.vaes.glmimage import GlmImageVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    SpatialImagePipelineConfig,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `torch`, `diffusers.image_processor`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.glmimage`, and `sglang.multimodal_gen.configs.models.encoders.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`torch`、`diffusers.image_processor`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.glmimage` 和 `sglang.multimodal_gen.configs.models.encoders.base`。这些依赖为后续实现提供所需符号。

### Lines 18-20: `GlmImagePipelineConfig` class overview / `GlmImagePipelineConfig` 类概览
```python
class GlmImagePipelineConfig(SpatialImagePipelineConfig):
    """Configuration for the GlmImage pipeline."""
```
**EN:** This block defines class `GlmImagePipelineConfig`. Configuration for the GlmImage pipeline. It inherits from `SpatialImagePipelineConfig`.
**CN:** 该代码块定义了类 `GlmImagePipelineConfig`。 它用于封装 glm image pipeline config 相关行为。 它继承自 `SpatialImagePipelineConfig`。

### Lines 21-44: supporting statements / 辅助语句
```python
    vae_precision: str = "bf16"

    should_use_guidance: bool = False
    task_type: ModelTaskType = ModelTaskType.T2I

    vae_tiling: bool = False

    vae_sp: bool = False

    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (T5Config(),)
    )

    dit_config: DiTConfig = field(default_factory=GlmImageDitConfig)
    # VAE
    vae_config: VAEConfig = field(default_factory=GlmImageVAEConfig)

    # GLM-Image uses T5 text encoder; base default is EncoderConfig() which lacks
    # parallel_folding and causes AttributeError + fallback to native T5 with missing weights.
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (T5Config(),)
    )

    enable_autocast: bool = False
```
**EN:** This block gathers supporting statements inside `GlmImagePipelineConfig`. It updates names such as `vae_precision`, `should_use_guidance`, `task_type`, `vae_tiling`, `vae_sp`, and `text_encoder_configs`. The code collaborates with `field`, and `T5Config`.
**CN:** 该代码块汇集了位于 `GlmImagePipelineConfig` 内部的辅助语句。 它会更新 `vae_precision`、`should_use_guidance`、`task_type`、`vae_tiling`、`vae_sp` 和 `text_encoder_configs` 等名称。 代码会与 `field` 和 `T5Config` 协同工作。

### Lines 46-48: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.vae_scale_factor = self.vae_config.get_vae_scale_factor()
        self.image_processor = VaeImageProcessor(vae_scale_factor=self.vae_scale_factor)
```
**EN:** This block defines method `__post_init__` on `GlmImagePipelineConfig`. It post-processes init. Key calls include `self.vae_config.get_vae_scale_factor`, and `VaeImageProcessor`.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `self.vae_config.get_vae_scale_factor` 和 `VaeImageProcessor`。

### Lines 50-55: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    def get_freqs_cis(self, batch, device, rotary_emb, dtype):
        height = batch.height // self.vae_scale_factor
        width = batch.width // self.vae_scale_factor
        hidden_states = torch.empty(1, 1, height, width, device=device, dtype=dtype)
        freqs_cis = rotary_emb(hidden_states)
        return freqs_cis
```
**EN:** This block defines method `get_freqs_cis` on `GlmImagePipelineConfig`. It retrieves freqs cis. Key calls include `torch.empty`, and `rotary_emb`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `torch.empty` 和 `rotary_emb`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 57-66: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {
            "prior_token_id": batch.prior_token_id,
            "prior_token_drop": batch.prior_token_drop_cond,
            "crop_coords": batch.crop_coords,
            "target_size": batch.target_size,
            "kv_caches": batch.kv_caches,
            "kv_caches_mode": "read",
            "freqs_cis": self.get_freqs_cis(batch, device, rotary_emb, dtype),
        }
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `GlmImagePipelineConfig`. It prepares pos cond kwargs. Key calls include `self.get_freqs_cis`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self.get_freqs_cis`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 68-77: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {
            "prior_token_id": batch.prior_token_id,
            "prior_token_drop": batch.prior_token_drop_uncond,
            "crop_coords": batch.crop_coords,
            "target_size": batch.target_size,
            "kv_caches": batch.kv_caches,
            "kv_caches_mode": "skip",
            "freqs_cis": self.get_freqs_cis(batch, device, rotary_emb, dtype),
        }
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `GlmImagePipelineConfig`. It prepares neg cond kwargs. Key calls include `self.get_freqs_cis`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self.get_freqs_cis`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 79-90: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(self, device, dtype, vae):
        latents_mean = (
            torch.tensor(self.vae_config.latents_mean)
            .view(1, self.vae_config.latent_channels, 1, 1)
            .to(device, dtype)
        )
        latents_std = (
            torch.tensor(self.vae_config.latents_std)
            .view(1, self.vae_config.latent_channels, 1, 1)
            .to(device, dtype)
        )
        return 1.0 / latents_std, latents_mean
```
**EN:** This block defines method `get_decode_scale_and_shift` on `GlmImagePipelineConfig`. It retrieves decode scale and shift. Key calls include `torch.tensor.view.to`, `torch.tensor.view`, and `torch.tensor`. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `torch.tensor.view.to`、`torch.tensor.view` 和 `torch.tensor`。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 92-95: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        if getattr(batch, "kv_caches", None) is not None:
            batch.kv_caches.clear()
        return latents.bfloat16()
```
**EN:** This block defines method `post_denoising_loop` on `GlmImagePipelineConfig`. It post-processes denoising loop. Key calls include `latents.bfloat16`, `getattr`, and `batch.kv_caches.clear`. The implementation branches on conditions. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `latents.bfloat16`、`getattr` 和 `batch.kv_caches.clear`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 97-98: `post_decoding` implementation / `post_decoding` 实现
```python
    def post_decoding(self, frames, server_args):
        return self.image_processor.postprocess(frames, output_type="latent")
```
**EN:** This block defines method `post_decoding` on `GlmImagePipelineConfig`. It post-processes decoding. Key calls include `self.image_processor.postprocess`. Parameters such as `frames`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `GlmImagePipelineConfig` 的方法 `post_decoding`。 它用于后处理decoding。 关键调用包括 `self.image_processor.postprocess`。 本段逻辑主要由 `frames` 和 `server_args` 等参数驱动。

## Key Concepts / 关键概念
- `GlmImagePipelineConfig`: Configuration for the GlmImage pipeline. / 核心类，用于封装 glm image pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `torch`, `diffusers.image_processor`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.glmimage`, `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.encoders.t5`, `sglang.multimodal_gen.configs.models.vaes.glmimage`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 98
