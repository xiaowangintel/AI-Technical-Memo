# sana.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/sana.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `SanaPipelineConfig`, and `sana_postprocess_text`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `SanaPipelineConfig` 和 `sana_postprocess_text` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 19-33: module setup and imports / 模块初始化与导入
```python
from collections.abc import Callable
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.sana import SanaConfig
from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.base import EncoderConfig
from sglang.multimodal_gen.configs.models.encoders.gemma2 import Gemma2Config
from sglang.multimodal_gen.configs.models.vaes.sana import SanaVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    SpatialImagePipelineConfig,
)
```
**EN:** This block establishes the module context and imports `collections.abc`, `dataclasses`, `torch`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.sana`, and `sglang.multimodal_gen.configs.models.encoders`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`dataclasses`、`torch`、`sglang.multimodal_gen.configs.models`、`sglang.multimodal_gen.configs.models.dits.sana` 和 `sglang.multimodal_gen.configs.models.encoders`。这些依赖为后续实现提供所需符号。

### Lines 36-39: `sana_postprocess_text` implementation / `sana_postprocess_text` 实现
```python
def sana_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    # SANA uses the final hidden state from Gemma2 directly as text conditioning.
    # No intermediate-layer extraction or masking needed (unlike QwenImage/ZImage).
    return outputs.last_hidden_state
```
**EN:** This block defines function `sana_postprocess_text`. It handles sana postprocess text logic. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sana_postprocess_text`。 它用于处理 sana postprocess text 相关逻辑。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 43-44: `SanaPipelineConfig` class overview / `SanaPipelineConfig` 类概览
```python
class SanaPipelineConfig(SpatialImagePipelineConfig):
```
**EN:** This block defines class `SanaPipelineConfig`. It encapsulates sana pipeline config behavior. It inherits from `SpatialImagePipelineConfig`.
**CN:** 该代码块定义了类 `SanaPipelineConfig`。 它用于封装 sana pipeline config 相关行为。 它继承自 `SpatialImagePipelineConfig`。

### Lines 45-82: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.T2I

    # should_use_guidance=False disables *embedded* guidance (timestep-conditioned
    # guidance token). Standard CFG via guidance_scale is still active.
    should_use_guidance: bool = False
    enable_autocast: bool = False

    # DC-AE does not support tiling or SP VAE decode yet.
    vae_tiling: bool = False
    vae_sp: bool = False
    vae_precision: str = "bf16"

    dit_config: DiTConfig = field(default_factory=SanaConfig)
    vae_config: VAEConfig = field(default_factory=SanaVAEConfig)

    # Single text encoder: Gemma2 (unlike Flux which uses CLIP + T5)
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Gemma2Config(),)
    )

    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))

    text_encoder_extra_args: list[dict] = field(
        default_factory=lambda: [
            {
                "padding": True,
                "return_attention_mask": True,
            }
        ]
    )

    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (None,),
    )

    postprocess_text_funcs: tuple[Callable[[str], str], ...] = field(
        default_factory=lambda: (sana_postprocess_text,)
    )
```
**EN:** This block gathers supporting statements inside `SanaPipelineConfig`. It updates names such as `task_type`, `should_use_guidance`, `enable_autocast`, `vae_tiling`, `vae_sp`, and `vae_precision`. The code collaborates with `field`, and `Gemma2Config`.
**CN:** 该代码块汇集了位于 `SanaPipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`should_use_guidance`、`enable_autocast`、`vae_tiling`、`vae_sp` 和 `vae_precision` 等名称。 代码会与 `field` 和 `Gemma2Config` 协同工作。

### Lines 84-92: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        # 4D latent shape: (B, C, H', W') — no temporal dim for T2I.
        # DC-AE compresses 1024x1024 -> 32x32 with 32 channels.
        compression = self.vae_config.arch_config.spatial_compression_ratio
        height = batch.height // compression
        width = batch.width // compression
        num_channels = self.dit_config.arch_config.num_channels_latents
        shape = (batch_size, num_channels, height, width)
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `SanaPipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 94-96: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        # Single encoder -> index [0] (Flux uses [1] because T5 is encoder #2)
        return batch.prompt_embeds[0]
```
**EN:** This block defines method `get_pos_prompt_embeds` on `SanaPipelineConfig`. It retrieves pos prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 98-99: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return batch.negative_prompt_embeds[0]
```
**EN:** This block defines method `get_neg_prompt_embeds` on `SanaPipelineConfig`. It retrieves neg prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 101-110: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        # encoder_attention_mask: batch stores list-of-tensors; diffusers' SanaTransformer
        # expects a single tensor (sglang's has list handling). Override with [0].
        out = {}
        m = batch.prompt_attention_mask
        if isinstance(m, (list, tuple)):
            out["encoder_attention_mask"] = m[0] if m else None
        elif m is not None:
            out["encoder_attention_mask"] = m
        return out
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `SanaPipelineConfig`. It prepares pos cond kwargs. Key calls include `isinstance`. The implementation branches on conditions. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 112-119: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        out = {}
        m = batch.negative_attention_mask
        if isinstance(m, (list, tuple)):
            out["encoder_attention_mask"] = m[0] if m else None
        elif m is not None:
            out["encoder_attention_mask"] = m
        return out
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `SanaPipelineConfig`. It prepares neg cond kwargs. Key calls include `isinstance`. The implementation branches on conditions. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 121-122: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        return latents
```
**EN:** This block defines method `post_denoising_loop` on `SanaPipelineConfig`. It post-processes denoising loop. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 124-127: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        # Sana's DiT uses local attention kernels and does not preserve semantics
        # when spatial latents are sequence-sharded.
        return latents, False
```
**EN:** This block defines method `shard_latents_for_sp` on `SanaPipelineConfig`. It handles shard latents for sp logic. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 129-130: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents):
        return latents
```
**EN:** This block defines method `gather_latents_for_sp` on `SanaPipelineConfig`. It handles gather latents for sp logic. Parameters such as `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `SanaPipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 本段逻辑主要由 `latents` 等参数驱动。

## Key Concepts / 关键概念
- `sana_postprocess_text`: Top-level function that handles sana postprocess text logic. / 顶层函数，用于处理 sana postprocess text 相关逻辑。
- `SanaPipelineConfig`: Primary class that encapsulates sana pipeline config behavior. / 核心类，用于封装 sana pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.sana`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.encoders.gemma2`, `sglang.multimodal_gen.configs.models.vaes.sana`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 130
