# zimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/zimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `TransformersModelConfig`, `ZImagePipelineConfig`, and `zimage_preprocess_text`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `TransformersModelConfig`、`ZImagePipelineConfig` 和 `zimage_preprocess_text` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-30: module setup and imports / 模块初始化与导入
```python
import math
from dataclasses import dataclass, field
from typing import Callable

import torch
import torch.distributed as dist

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.zimage import ZImageDitConfig
from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.qwen3 import Qwen3TextConfig
from sglang.multimodal_gen.configs.models.vaes.flux import FluxVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ImagePipelineConfig,
    ModelTaskType,
    TextConditioningOutput,
    pad_text_embeddings_with_mask,
)
from sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config import (
    ModelDeploymentConfig,
)
from sglang.multimodal_gen.configs.post_training.pipeline_configs import (
    ZImageRolloutPipelineMixin,
)
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_sp_group,
    get_sp_parallel_rank,
    get_sp_world_size,
)
```
**EN:** This block establishes the module context and imports `math`, `dataclasses`, `typing`, `torch`, `torch.distributed`, and `sglang.multimodal_gen.configs.models`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `math`、`dataclasses`、`typing`、`torch`、`torch.distributed` 和 `sglang.multimodal_gen.configs.models`。这些依赖为后续实现提供所需符号。

### Lines 33-37: `zimage_preprocess_text` implementation / `zimage_preprocess_text` 实现
```python
def zimage_preprocess_text(prompt: str):
    messages = [
        {"role": "user", "content": prompt},
    ]
    return messages
```
**EN:** This block defines function `zimage_preprocess_text`. It handles zimage preprocess text logic. Parameters such as `prompt` drive the behavior in this section.
**CN:** 该代码块定义了函数 `zimage_preprocess_text`。 它用于处理 zimage preprocess text 相关逻辑。 本段逻辑主要由 `prompt` 等参数驱动。

### Lines 40-57: `zimage_postprocess_text` implementation / `zimage_postprocess_text` 实现
```python
def zimage_postprocess_text(
    outputs: BaseEncoderOutput, _text_inputs
) -> torch.Tensor | TextConditioningOutput:
    """Return unpadded Z-Image text embeddings.

    Batched outputs return TextConditioningOutput to preserve per-prompt text
    lengths.
    """
    device = outputs.hidden_states[-2].device
    prompt_mask = _text_inputs.attention_mask.to(device).bool()
    hidden_states = outputs.hidden_states[-2]
    if hidden_states.shape[0] == 1:
        return hidden_states[0][prompt_mask[0]]

    split_hidden_states = [
        hidden_states[idx][prompt_mask[idx]] for idx in range(hidden_states.shape[0])
    ]
    return pad_text_embeddings_with_mask(split_hidden_states)
```
**EN:** This block defines function `zimage_postprocess_text`. Return unpadded Z-Image text embeddings. Batched outputs return TextConditioningOutput to preserve per-prompt text lengths. Key calls include `_text_inputs.attention_mask.to.bool`, `pad_text_embeddings_with_mask`, `_text_inputs.attention_mask.to`, and `range`. The implementation branches on conditions. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `zimage_postprocess_text`。 它用于处理 zimage postprocess text 相关逻辑。 关键调用包括 `_text_inputs.attention_mask.to.bool`、`pad_text_embeddings_with_mask`、`_text_inputs.attention_mask.to` 和 `range`。 实现中包含条件分支。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 60-60: `TransformersModelConfig` class overview / `TransformersModelConfig` 类概览
```python
class TransformersModelConfig(EncoderConfig):
```
**EN:** This block defines class `TransformersModelConfig`. It encapsulates transformers model config behavior. It inherits from `EncoderConfig`.
**CN:** 该代码块定义了类 `TransformersModelConfig`。 它用于封装 transformers model config 相关行为。 它继承自 `EncoderConfig`。

### Lines 61-61: supporting statements / 辅助语句
```python
    tokenizer_kwargs: dict = field(default_factory=lambda: {})
```
**EN:** This block gathers supporting statements inside `TransformersModelConfig`. It updates names such as `tokenizer_kwargs`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TransformersModelConfig` 内部的辅助语句。 它会更新 `tokenizer_kwargs` 等名称。 代码会与 `field` 协同工作。

### Lines 65-65: `ZImagePipelineConfig` class overview / `ZImagePipelineConfig` 类概览
```python
class ZImagePipelineConfig(ZImageRolloutPipelineMixin, ImagePipelineConfig):
```
**EN:** This block defines class `ZImagePipelineConfig`. It encapsulates zimage pipeline config behavior. It inherits from `ZImageRolloutPipelineMixin`, and `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `ZImagePipelineConfig`。 它用于封装 zimage pipeline config 相关行为。 它继承自 `ZImageRolloutPipelineMixin` 和 `ImagePipelineConfig`。

### Lines 66-85: supporting statements / 辅助语句
```python
    should_use_guidance: bool = False
    task_type: ModelTaskType = ModelTaskType.T2I
    dit_config: DiTConfig = field(default_factory=ZImageDitConfig)
    vae_config: VAEConfig = field(default_factory=FluxVAEConfig)
    vae_precision: str = "bf16"
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Qwen3TextConfig(),)
    )

    preprocess_text_funcs: tuple[Callable, ...] = field(
        default_factory=lambda: (zimage_preprocess_text,)
    )
    postprocess_text_funcs: tuple[Callable, ...] = field(
        default_factory=lambda: (zimage_postprocess_text,)
    )

    SEQ_LEN_MULTIPLE: int = 32
    PATCH_SIZE: int = 2
    F_PATCH_SIZE: int = 1
```
**EN:** This block gathers supporting statements inside `ZImagePipelineConfig`. It updates names such as `should_use_guidance`, `task_type`, `dit_config`, `vae_config`, `vae_precision`, and `text_encoder_precisions`. The code collaborates with `field`, and `Qwen3TextConfig`.
**CN:** 该代码块汇集了位于 `ZImagePipelineConfig` 内部的辅助语句。 它会更新 `should_use_guidance`、`task_type`、`dit_config`、`vae_config`、`vae_precision` 和 `text_encoder_precisions` 等名称。 代码会与 `field` 和 `Qwen3TextConfig` 协同工作。

### Lines 87-88: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig(fsdp_auto_min_available_memory_gb=40)
```
**EN:** This block defines method `get_model_deployment_config` on `ZImagePipelineConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 90-108: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompts: list[str], tokenizer, tok_kwargs) -> dict:
        rendered_prompts = [
            tokenizer.apply_chat_template(
                prompt,
                tokenize=False,
                add_generation_prompt=True,
                enable_thinking=True,
            )
            for prompt in prompts
        ]

        effective_max_length = tok_kwargs.pop("max_length", 512)
        return tokenizer(
            rendered_prompts,
            padding="max_length",
            max_length=effective_max_length,
            truncation=True,
            return_tensors="pt",
        )
```
**EN:** This block defines method `tokenize_prompt` on `ZImagePipelineConfig`. It handles tokenize prompt logic. Key calls include `tok_kwargs.pop`, `tokenizer`, and `tokenizer.apply_chat_template`. Parameters such as `prompts`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tok_kwargs.pop`、`tokenizer` 和 `tokenizer.apply_chat_template`。 本段逻辑主要由 `prompts`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 110-114: `_ceil_to_multiple` implementation / `_ceil_to_multiple` 实现
```python
    @staticmethod
    def _ceil_to_multiple(x: int, m: int) -> int:
        if m <= 0:
            return x
        return int(math.ceil(x / m) * m)
```
**EN:** This block defines method `_ceil_to_multiple` on `ZImagePipelineConfig`. It handles ceil to multiple logic. Key calls include `int`, and `math.ceil`. The implementation branches on conditions. Parameters such as `x`, and `m` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_ceil_to_multiple`。 它用于处理 ceil to multiple 相关逻辑。 关键调用包括 `int` 和 `math.ceil`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `m` 等参数驱动。

### Lines 116-119: `_split_evenly` implementation / `_split_evenly` 实现
```python
    @staticmethod
    def _split_evenly(total: int, parts: int) -> list[int]:
        base, remainder = divmod(total, parts)
        return [base + int(rank < remainder) for rank in range(parts)]
```
**EN:** This block defines method `_split_evenly` on `ZImagePipelineConfig`. It splits evenly. Key calls include `divmod`, `int`, and `range`. Parameters such as `total`, and `parts` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_split_evenly`。 它用于拆分evenly。 关键调用包括 `divmod`、`int` 和 `range`。 本段逻辑主要由 `total` 和 `parts` 等参数驱动。

### Lines 121-196: `_build_zimage_sp_plan` implementation / `_build_zimage_sp_plan` 实现
```python
    def _build_zimage_sp_plan(self, batch) -> dict:
        """Build an SP plan that preserves native spatial layout for Z-Image."""
        sp_size = get_sp_world_size()
        rank = get_sp_parallel_rank()

        raw_latent_shape = getattr(batch, "raw_latent_shape", None)
        if raw_latent_shape is not None and len(raw_latent_shape) >= 5:
            H = int(raw_latent_shape[3])
            W = int(raw_latent_shape[4])
        else:
            H = int(
                batch.height // self.vae_config.arch_config.spatial_compression_ratio
            )
            W = int(
                batch.width // self.vae_config.arch_config.spatial_compression_ratio
            )

        # ZImage patchifies [C, F, H, W] latents in native F/H/W order, so shard
        # native H or W directly.
        H_tok = H // self.PATCH_SIZE
        W_tok = W // self.PATCH_SIZE

        shard_options = []
        for shard_axis, axis_tok, other_tok, tie_break in (
            ("h", H_tok, W_tok, 0),
            ("w", W_tok, H_tok, 1),
        ):
            axis_sizes = self._split_evenly(axis_tok, sp_size)
            local_seq_lens = [axis_size * other_tok for axis_size in axis_sizes]
            img_seq_target = self._ceil_to_multiple(
                max(local_seq_lens), self.SEQ_LEN_MULTIPLE
            )
            total_pad_tokens = img_seq_target * sp_size - (H_tok * W_tok)
            shard_options.append(
                (
                    total_pad_tokens,
                    -axis_tok,
                    tie_break,
                    shard_axis,
                    axis_sizes,
                    img_seq_target,
                )
            )

        _, _, _, shard_axis, axis_sizes, img_seq_target = min(shard_options)
        axis_start_tok = sum(axis_sizes[:rank])
        axis_local_tok = axis_sizes[rank]

        if shard_axis == "h":
            h0_tok = axis_start_tok
            w0_tok = 0
            local_h_tok = axis_local_tok
            local_w_tok = W_tok
        else:
            h0_tok = 0
            w0_tok = axis_start_tok
            local_h_tok = H_tok
            local_w_tok = axis_local_tok

        plan = {
            "sp_size": sp_size,
            "rank": rank,
            "H": H,
            "W": W,
            "H_tok": H_tok,
            "W_tok": W_tok,
            "shard_axis": shard_axis,
            "shard_sizes_tok": axis_sizes,
            "h0_tok": h0_tok,
            "w0_tok": w0_tok,
            "local_h_tok": local_h_tok,
            "local_w_tok": local_w_tok,
            "img_seq_target": img_seq_target,
        }
        batch._zimage_sp_plan = plan
        return plan
```
**EN:** This block defines method `_build_zimage_sp_plan` on `ZImagePipelineConfig`. Build an SP plan that preserves native spatial layout for Z-Image. Key calls include `get_sp_world_size`, `get_sp_parallel_rank`, `getattr`, `min`, and `sum`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_build_zimage_sp_plan`。 它用于构建zimage sp plan。 关键调用包括 `get_sp_world_size`、`get_sp_parallel_rank`、`getattr`、`min` 和 `sum`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 198-203: `_get_zimage_sp_plan` implementation / `_get_zimage_sp_plan` 实现
```python
    def _get_zimage_sp_plan(self, batch) -> dict:
        plan = getattr(batch, "_zimage_sp_plan", None)
        sp_size = get_sp_world_size()
        if plan is None or plan.get("sp_size") != sp_size:
            plan = self._build_zimage_sp_plan(batch)
        return plan
```
**EN:** This block defines method `_get_zimage_sp_plan` on `ZImagePipelineConfig`. It retrieves zimage sp plan. Key calls include `getattr`, `get_sp_world_size`, `self._build_zimage_sp_plan`, and `plan.get`. The implementation branches on conditions. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_get_zimage_sp_plan`。 它用于获取zimage sp plan。 关键调用包括 `getattr`、`get_sp_world_size`、`self._build_zimage_sp_plan` 和 `plan.get`。 实现中包含条件分支。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 205-235: `_split_text_embeds_for_dit` implementation / `_split_text_embeds_for_dit` 实现
```python
    def _split_text_embeds_for_dit(self, batch, *, negative: bool = False):
        """Return per-request text tensors, trimming padded batched embeddings."""
        embeds = batch.negative_prompt_embeds if negative else batch.prompt_embeds
        if embeds is None:
            return None

        if isinstance(embeds, (list, tuple)):
            if not embeds:
                return []
            embeds = embeds[0]

        if not torch.is_tensor(embeds):
            return embeds

        if embeds.ndim == 2:
            return [embeds]

        if embeds.ndim != 3:
            raise ValueError(
                "Z-Image text embeddings must have shape [seq, dim] or [batch, seq, dim]"
            )

        seq_lens = self.require_text_seq_lens(
            batch,
            0,
            negative=negative,
            expected_batch_size=int(embeds.shape[0]),
        )
        return [
            embeds[idx, :seq_len].contiguous() for idx, seq_len in enumerate(seq_lens)
        ]
```
**EN:** This block defines method `_split_text_embeds_for_dit` on `ZImagePipelineConfig`. Return per-request text tensors, trimming padded batched embeddings. Key calls include `isinstance`, `self.require_text_seq_lens`, `torch.is_tensor`, `ValueError`, and `embeds.contiguous`. The implementation branches on conditions. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_split_text_embeds_for_dit`。 它用于拆分text embeds for dit。 关键调用包括 `isinstance`、`self.require_text_seq_lens`、`torch.is_tensor`、`ValueError` 和 `embeds.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 237-264: `_caption_rope_length` implementation / `_caption_rope_length` 实现
```python
    def _caption_rope_length(self, prompt_embeds, batch, *, negative: bool = False):
        """Return the shared caption RoPE length for current text embeddings."""
        if torch.is_tensor(prompt_embeds):
            if prompt_embeds.ndim == 2:
                return int(prompt_embeds.shape[0])
            if prompt_embeds.ndim == 3:
                seq_lens = self.require_text_seq_lens(
                    batch,
                    0,
                    negative=negative,
                    expected_batch_size=int(prompt_embeds.shape[0]),
                )
                return max(seq_lens) if seq_lens else int(prompt_embeds.shape[1])

        if isinstance(prompt_embeds, (list, tuple)) and prompt_embeds:
            first = prompt_embeds[0]
            if torch.is_tensor(first):
                if first.ndim == 3:
                    seq_lens = self.require_text_seq_lens(
                        batch,
                        0,
                        negative=negative,
                        expected_batch_size=int(first.shape[0]),
                    )
                    return max(seq_lens) if seq_lens else int(first.shape[1])
                return max(int(item.shape[0]) for item in prompt_embeds)

        raise ValueError("Unable to infer Z-Image caption length for rotary embeddings")
```
**EN:** This block defines method `_caption_rope_length` on `ZImagePipelineConfig`. Return the shared caption RoPE length for current text embeddings. Key calls include `torch.is_tensor`, `ValueError`, `isinstance`, `int`, and `self.require_text_seq_lens`. The implementation branches on conditions. Parameters such as `prompt_embeds`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `_caption_rope_length`。 它用于处理 caption rope length 相关逻辑。 关键调用包括 `torch.is_tensor`、`ValueError`、`isinstance`、`int` 和 `self.require_text_seq_lens`。 实现中包含条件分支。 本段逻辑主要由 `prompt_embeds` 和 `batch` 等参数驱动。

### Lines 266-267: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        return self._split_text_embeds_for_dit(batch, negative=False)
```
**EN:** This block defines method `get_pos_prompt_embeds` on `ZImagePipelineConfig`. It retrieves pos prompt embeds. Key calls include `self._split_text_embeds_for_dit`. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 关键调用包括 `self._split_text_embeds_for_dit`。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 269-270: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return self._split_text_embeds_for_dit(batch, negative=True)
```
**EN:** This block defines method `get_neg_prompt_embeds` on `ZImagePipelineConfig`. It retrieves neg prompt embeds. Key calls include `self._split_text_embeds_for_dit`. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 关键调用包括 `self._split_text_embeds_for_dit`。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 272-275: `get_latent_dtype` implementation / `get_latent_dtype` 实现
```python
    def get_latent_dtype(self, prompt_dtype: torch.dtype) -> torch.dtype:
        # Match the official diffusers Z-Image pipeline, which samples latents in fp32
        # and keeps scheduler state in fp32.
        return torch.float32
```
**EN:** This block defines method `get_latent_dtype` on `ZImagePipelineConfig`. It retrieves latent dtype. Parameters such as `prompt_dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `get_latent_dtype`。 它用于获取latent dtype。 本段逻辑主要由 `prompt_dtype` 等参数驱动。

### Lines 277-290: `shard_latents_for_sp` implementation / `shard_latents_for_sp` 实现
```python
    def shard_latents_for_sp(self, batch, latents):
        sp_size = get_sp_world_size()
        if sp_size <= 1 or latents.dim() != 5:
            return latents, False

        plan = self._get_zimage_sp_plan(batch)
        if plan["shard_axis"] == "h":
            h0 = plan["h0_tok"] * self.PATCH_SIZE
            h1 = (plan["h0_tok"] + plan["local_h_tok"]) * self.PATCH_SIZE
            return latents[:, :, :, h0:h1, :].contiguous(), True

        w0 = plan["w0_tok"] * self.PATCH_SIZE
        w1 = (plan["w0_tok"] + plan["local_w_tok"]) * self.PATCH_SIZE
        return latents[:, :, :, :, w0:w1].contiguous(), True
```
**EN:** This block defines method `shard_latents_for_sp` on `ZImagePipelineConfig`. It handles shard latents for sp logic. Key calls include `get_sp_world_size`, `self._get_zimage_sp_plan`, `latents.contiguous`, and `latents.dim`. The implementation branches on conditions. Parameters such as `batch`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `shard_latents_for_sp`。 它用于处理 shard latents for sp 相关逻辑。 关键调用包括 `get_sp_world_size`、`self._get_zimage_sp_plan`、`latents.contiguous` 和 `latents.dim`。 实现中包含条件分支。 本段逻辑主要由 `batch` 和 `latents` 等参数驱动。

### Lines 292-326: `gather_latents_for_sp` implementation / `gather_latents_for_sp` 实现
```python
    def gather_latents_for_sp(self, latents, batch):
        # Gather native H/W shards by padding to a common collective shape, then crop.
        latents = latents.contiguous()
        if get_sp_world_size() <= 1 or latents.dim() not in (4, 5, 6):
            return latents

        assert batch is not None
        plan = self._get_zimage_sp_plan(batch)
        if latents.dim() == 4:
            shard_dim = 2 if plan["shard_axis"] == "h" else 3
        elif latents.dim() == 5:
            shard_dim = 3 if plan["shard_axis"] == "h" else 4
        else:
            shard_dim = 4 if plan["shard_axis"] == "h" else 5
        max_axis_tok = max(plan["shard_sizes_tok"])
        max_axis_lat = max_axis_tok * self.PATCH_SIZE

        pad_shape = list(latents.shape)
        pad_shape[shard_dim] = max_axis_lat
        padded = latents.new_zeros(pad_shape)
        axis_len = latents.shape[shard_dim]
        padded_slices = [slice(None)] * latents.dim()
        padded_slices[shard_dim] = slice(axis_len)
        padded[tuple(padded_slices)] = latents

        gathered = [torch.empty_like(padded) for _ in range(plan["sp_size"])]
        dist.all_gather(gathered, padded, group=get_sp_group().device_group)

        pieces = []
        for rank, tensor in enumerate(gathered):
            axis_lat = plan["shard_sizes_tok"][rank] * self.PATCH_SIZE
            gather_slices = [slice(None)] * latents.dim()
            gather_slices[shard_dim] = slice(axis_lat)
            pieces.append(tensor[tuple(gather_slices)])
        return torch.cat(pieces, dim=shard_dim)
```
**EN:** This block defines method `gather_latents_for_sp` on `ZImagePipelineConfig`. It handles gather latents for sp logic. Key calls include `latents.contiguous`, `self._get_zimage_sp_plan`, `max`, `list`, and `latents.new_zeros`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `gather_latents_for_sp`。 它用于处理 gather latents for sp 相关逻辑。 关键调用包括 `latents.contiguous`、`self._get_zimage_sp_plan`、`max`、`list` 和 `latents.new_zeros`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 328-329: `gather_noise_pred_for_sp` implementation / `gather_noise_pred_for_sp` 实现
```python
    def gather_noise_pred_for_sp(self, batch, noise_pred):
        return self.gather_latents_for_sp(noise_pred, batch=batch)
```
**EN:** This block defines method `gather_noise_pred_for_sp` on `ZImagePipelineConfig`. It handles gather noise pred for sp logic. Key calls include `self.gather_latents_for_sp`. Parameters such as `batch`, and `noise_pred` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `gather_noise_pred_for_sp`。 它用于处理 gather noise pred for sp 相关逻辑。 关键调用包括 `self.gather_latents_for_sp`。 本段逻辑主要由 `batch` 和 `noise_pred` 等参数驱动。

### Lines 331-342: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        raw_latent_shape = getattr(batch, "raw_latent_shape", None)
        if raw_latent_shape is not None and latents.dim() == 5:
            latents = latents[:, :, :, : raw_latent_shape[3], : raw_latent_shape[4]]

        bs, channels, num_frames, height, width = latents.shape
        if raw_latent_shape is not None and num_frames > raw_latent_shape[2]:
            latents = latents[:, :, : raw_latent_shape[2], :, :]
            num_frames = raw_latent_shape[2]
        if num_frames != 1:
            return latents[:, :, 0, :, :]
        return latents.view(bs, channels, height, width)
```
**EN:** This block defines method `post_denoising_loop` on `ZImagePipelineConfig`. It post-processes denoising loop. Key calls include `getattr`, `latents.view`, and `latents.dim`. The implementation branches on conditions. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `getattr`、`latents.view` 和 `latents.dim`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 344-451: `get_freqs_cis` implementation / `get_freqs_cis` 实现
```python
    def get_freqs_cis(
        self,
        prompt_embeds,
        width,
        height,
        device,
        rotary_emb,
        batch,
        *,
        negative: bool = False,
    ):
        """Build caption and image RoPE caches for Z-Image conditioning.

        Batched prompts use stored text lengths. SP mode builds image caches for
        the local spatial shard.
        """

        def create_coordinate_grid(size, start=None, device=None):
            if start is None:
                start = (0 for _ in size)

            axes = [
                torch.arange(x0, x0 + span, dtype=torch.int32, device=device)
                for x0, span in zip(start, size)
            ]
            grids = torch.meshgrid(axes, indexing="ij")
            return torch.stack(grids, dim=-1)

        sp_size = get_sp_world_size()
        if sp_size > 1:
            # SP path: keep caption replicated on every rank and build local-only
            # image freqs_cis matching the spatial shard.
            plan = self._get_zimage_sp_plan(batch)
            cap_ori_len = self._caption_rope_length(
                prompt_embeds, batch, negative=negative
            )
            cap_padding_len = (-cap_ori_len) % self.SEQ_LEN_MULTIPLE

            # caption (replicated prefix)
            cap_pos_ids = create_coordinate_grid(
                size=(cap_ori_len + cap_padding_len, 1, 1),
                start=(1, 0, 0),
                device=device,
            ).flatten(0, 2)
            cap_freqs_cis = rotary_emb(cap_pos_ids)

            # Build image positions for the local native shard.
            F_tokens = 1
            H_tokens_local = plan["local_h_tok"]
            W_tokens_local = plan["local_w_tok"]
            img_pos_ids = create_coordinate_grid(
                size=(F_tokens, H_tokens_local, W_tokens_local),
                start=(
                    cap_ori_len + cap_padding_len + 1,
                    plan["h0_tok"],
                    plan["w0_tok"],
                ),
                device=device,
            ).flatten(0, 2)
            img_pad_len = plan["img_seq_target"] - img_pos_ids.shape[0]
            if img_pad_len:
                pad_ids = create_coordinate_grid(
                    size=(1, 1, 1), start=(0, 0, 0), device=device
                ).flatten(0, 2)
                img_pos_ids = torch.cat(
                    [img_pos_ids, pad_ids.repeat(img_pad_len, 1)], dim=0
                )
            x_freqs_cis = rotary_emb(img_pos_ids)
            return (cap_freqs_cis, x_freqs_cis)

        cap_ori_len = self._caption_rope_length(prompt_embeds, batch, negative=negative)
        cap_padding_len = (-cap_ori_len) % self.SEQ_LEN_MULTIPLE
        cap_padded_pos_ids = create_coordinate_grid(
            size=(cap_ori_len + cap_padding_len, 1, 1),
            start=(1, 0, 0),
            device=device,
        ).flatten(0, 2)

        F = 1
        H = height // self.vae_config.arch_config.spatial_compression_ratio
        W = width // self.vae_config.arch_config.spatial_compression_ratio

        pH, pW = self.PATCH_SIZE, self.PATCH_SIZE
        pF = self.F_PATCH_SIZE
        F_tokens, H_tokens, W_tokens = F // pF, H // pH, W // pW
        image_ori_len = F_tokens * H_tokens * W_tokens
        image_padding_len = (-image_ori_len) % self.SEQ_LEN_MULTIPLE

        image_ori_pos_ids = create_coordinate_grid(
            size=(F_tokens, H_tokens, W_tokens),
            start=(cap_ori_len + cap_padding_len + 1, 0, 0),
            device=device,
        ).flatten(0, 2)
        image_padding_pos_ids = (
            create_coordinate_grid(
                size=(1, 1, 1),
                start=(0, 0, 0),
                device=device,
            )
            .flatten(0, 2)
            .repeat(image_padding_len, 1)
        )
        image_padded_pos_ids = torch.cat(
            [image_ori_pos_ids, image_padding_pos_ids], dim=0
        )
        cap_freqs_cis = rotary_emb(cap_padded_pos_ids)
        x_freqs_cis = rotary_emb(image_padded_pos_ids)
        return (cap_freqs_cis, x_freqs_cis)
```
**EN:** This block defines method `get_freqs_cis` on `ZImagePipelineConfig`. Build caption and image RoPE caches for Z-Image conditioning. Batched prompts use stored text lengths. Key calls include `get_sp_world_size`, `self._caption_rope_length`, `create_coordinate_grid.flatten`, `create_coordinate_grid.flatten.repeat`, and `torch.cat`. The implementation branches on conditions. Parameters such as `prompt_embeds`, `width`, `height`, `device`, and `rotary_emb` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `get_freqs_cis`。 它用于获取freqs cis。 关键调用包括 `get_sp_world_size`、`self._caption_rope_length`、`create_coordinate_grid.flatten`、`create_coordinate_grid.flatten.repeat` 和 `torch.cat`。 实现中包含条件分支。 本段逻辑主要由 `prompt_embeds`、`width`、`height`、`device` 和 `rotary_emb` 等参数驱动。

### Lines 453-468: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {
            "freqs_cis": self.get_freqs_cis(
                batch.prompt_embeds[0],
                batch.width,
                batch.height,
                device,
                rotary_emb,
                batch,
            ),
            "image_seq_len_target": (
                self._get_zimage_sp_plan(batch)["img_seq_target"]
                if get_sp_world_size() > 1
                else None
            ),
        }
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `ZImagePipelineConfig`. It prepares pos cond kwargs. Key calls include `self.get_freqs_cis`, `get_sp_world_size`, and `self._get_zimage_sp_plan`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 关键调用包括 `self.get_freqs_cis`、`get_sp_world_size` 和 `self._get_zimage_sp_plan`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 470-492: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        use_negative_embeds = batch.negative_prompt_embeds is not None
        prompt_embeds = (
            batch.negative_prompt_embeds[0]
            if use_negative_embeds
            else batch.prompt_embeds[0]
        )
        return {
            "freqs_cis": self.get_freqs_cis(
                prompt_embeds,
                batch.width,
                batch.height,
                device,
                rotary_emb,
                batch,
                negative=use_negative_embeds,
            ),
            "image_seq_len_target": (
                self._get_zimage_sp_plan(batch)["img_seq_target"]
                if get_sp_world_size() > 1
                else None
            ),
        }
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `ZImagePipelineConfig`. It prepares neg cond kwargs. Key calls include `self.get_freqs_cis`, `get_sp_world_size`, and `self._get_zimage_sp_plan`. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ZImagePipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 关键调用包括 `self.get_freqs_cis`、`get_sp_world_size` 和 `self._get_zimage_sp_plan`。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

## Key Concepts / 关键概念
- `zimage_preprocess_text`: Top-level function that handles zimage preprocess text logic. / 顶层函数，用于处理 zimage preprocess text 相关逻辑。
- `zimage_postprocess_text`: Return unpadded Z-Image text embeddings. / 顶层函数，用于处理 zimage postprocess text 相关逻辑。
- `TransformersModelConfig`: Primary class that encapsulates transformers model config behavior. / 核心类，用于封装 transformers model config 相关行为。
- `ZImagePipelineConfig`: Primary class that encapsulates zimage pipeline config behavior. / 核心类，用于封装 zimage pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.zimage`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.qwen3`, `sglang.multimodal_gen.configs.models.vaes.flux`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config`, `sglang.multimodal_gen.configs.post_training.pipeline_configs`, `sglang.multimodal_gen.runtime.distributed.parallel_state`

- **Total lines / 总行数**: 492
