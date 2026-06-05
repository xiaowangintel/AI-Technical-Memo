# image_encoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/image_encoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for image encoding, connecting stages, tensors, and runtime metadata. Key symbols include `ImageEncodingFingerprint`, `LTX2ImageEncodingFingerprint`, `ImageVAEEncodingFingerprint`. / 该模块负责 image encoding 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `ImageEncodingFingerprint`, `LTX2ImageEncodingFingerprint`, `ImageVAEEncodingFingerprint`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Image encoding stages for I2V diffusion pipelines.

This module contains implementations of image encoding stages for diffusion pipelines.
"""

import inspect
from dataclasses import dataclass
from typing import Any

import numpy as np
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 52-60: Class `ImageEncodingFingerprint` / 类 `ImageEncodingFingerprint`
```python
@dataclass(frozen=True)
class ImageEncodingFingerprint:
    image_source: Any
    prompt: Any
    negative_prompt: Any
    do_classifier_free_guidance: bool
    height: int | None
    width: int | None
    num_frames: int | None
```
**EN:** This class models `ImageEncodingFingerprint`.
**CN:** 该类实现 `ImageEncodingFingerprint`。

### Lines 63-71: Class `LTX2ImageEncodingFingerprint` / 类 `LTX2ImageEncodingFingerprint`
```python
@dataclass(frozen=True)
class LTX2ImageEncodingFingerprint:
    image_source: Any
    height: int | None
    width: int | None
    num_frames: int | None
    latent_dtype: str
    condition_encoder_subdir: str
    encode_sample_mode: str
```
**EN:** This class models `LTX2ImageEncodingFingerprint`.
**CN:** 该类实现 `LTX2ImageEncodingFingerprint`。

### Lines 74-82: Class `ImageVAEEncodingFingerprint` / 类 `ImageVAEEncodingFingerprint`
```python
@dataclass(frozen=True)
class ImageVAEEncodingFingerprint:
    image_source: Any
    height: int | None
    width: int | None
    num_frames: int | None
    encode_sample_mode: str
    vae_precision: Any
    vae_tiling: bool
```
**EN:** This class models `ImageVAEEncodingFingerprint`.
**CN:** 该类实现 `ImageVAEEncodingFingerprint`。

### Lines 85-98: Function `_freeze_image_source_value` / 函数 `_freeze_image_source_value`
```python
def _freeze_image_source_value(value):
    """Build a hashable identity fragment for image inputs.

    Image inputs are often PIL/numpy/tensor objects. For file paths we can use
    the path value; for in-memory objects we only dedup when the exact same
    object instance is shared by multiple requests. This avoids expensive image
    hashing and avoids treating two mutable image objects as equivalent just
    because they currently have the same shape.
    """
    if isinstance(value, (list, tuple)):
        return tuple(_freeze_image_source_value(item) for item in value)
    if isinstance(value, (str, int, float, bool, type(None))):
        return value
    return ("object", id(value))
```
**EN:** This function drives `_freeze_image_source_value` with inputs such as `value`. Build a hashable identity fragment for image inputs.
**CN:** 这个函数负责 `_freeze_image_source_value`，主要处理 `value` 等输入。 文档字符串说明：Build a hashable identity fragment for image inputs.

### Lines 101-110: Function `_build_image_source_fingerprint` / 函数 `_build_image_source_fingerprint`
```python
def _build_image_source_fingerprint(batch: Req, *, prefer_vae_image: bool = False):
    """Return the image input fragment used by image encoding fingerprints."""
    if batch.image_path is not None:
        return ("path", PipelineStage.freeze_for_dedup(batch.image_path))
    image = (
        batch.vae_image if prefer_vae_image and batch.vae_image is not None else None
    )
    if image is None:
        image = batch.condition_image
    return ("image", _freeze_image_source_value(image))
```
**EN:** This function drives `_build_image_source_fingerprint` with inputs such as `batch`, `prefer_vae_image`. Return the image input fragment used by image encoding fingerprints.
**CN:** 这个函数负责 `_build_image_source_fingerprint`，主要处理 `batch`, `prefer_vae_image` 等输入。 文档字符串说明：Return the image input fragment used by image encoding fingerprints.

### Lines 113-412: Class `ImageEncodingStage` / 类 `ImageEncodingStage`
```python
class ImageEncodingStage(PipelineStage):
    """
    Stage for encoding image prompts into embeddings for diffusion models.

    This stage handles the encoding of image prompts into the embedding space
    expected by the diffusion model.
    """

    deduplicated_output_fields = (
        "image_embeds",
        "prompt_embeds",
        "negative_prompt_embeds",
        "prompt_embeds_mask",
        "negative_prompt_embeds_mask",
# ...
        """Verify image encoding stage outputs."""
        result = VerificationResult()
        # result.add_check("image_embeds", batch.image_embeds, V.list_of_tensors_dims(3))
        return result
```
**EN:** This class models `ImageEncodingStage` as a specialization of `PipelineStage`. Stage for encoding image prompts into embeddings for diffusion models. Important methods include `__init__`, `component_uses`, `encoding_image_edit`, `_split_text_conditioning_output`.
**CN:** 该类实现 `ImageEncodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for encoding image prompts into embeddings for diffusion models. 其中较重要的方法包括 `__init__`, `component_uses`, `encoding_image_edit`, `_split_text_conditioning_output`。

### Lines 415-789: Class `LTX2ImageEncodingStage` / 类 `LTX2ImageEncodingStage`
```python
class LTX2ImageEncodingStage(PipelineStage):
    """Encode ``batch.image_path`` into packed token latents for LTX-2 TI2V.

    Runs before denoising. Populates:
      - ``batch.condition_image`` (resized PIL image)
      - ``batch.image_latent``    (packed [B, S0, D] token latents)
      - ``batch.ltx2_num_image_tokens``
    """

    deduplicated_output_fields = (
        "condition_image",
        "image_latent",
        "ltx2_num_image_tokens",
    )
# ...
            latent_dtype=str(latent_dtype),
            condition_encoder_subdir=encoder_subdir,
            encode_sample_mode=sample_mode,
        )
```
**EN:** This class models `LTX2ImageEncodingStage` as a specialization of `PipelineStage`. Encode ``batch.image_path`` into packed token latents for LTX-2 TI2V. Important methods include `__init__`, `component_uses`, `_ensure_condition_image_encoder`, `_apply_video_codec_compression`.
**CN:** 该类实现 `LTX2ImageEncodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Encode ``batch.image_path`` into packed token latents for LTX-2 TI2V. 其中较重要的方法包括 `__init__`, `component_uses`, `_ensure_condition_image_encoder`, `_apply_video_codec_compression`。

### Lines 792-1033: Class `ImageVAEEncodingStage` / 类 `ImageVAEEncodingStage`
```python
class ImageVAEEncodingStage(PipelineStage):
    """
    Stage for encoding pixel representations into latent space.

    This stage handles the encoding of pixel representations into the final
    input format (e.g., image_latents).
    """

    deduplicated_output_fields = (
        "image_latent",
        "condition_image_latent_ids",
        "vae_image_sizes",
    )

# ...
        # result.add_check(
        #     "image_latent", batch.image_latent, [V.is_tensor, V.with_dims(5)]
        # )
        return result
```
**EN:** This class models `ImageVAEEncodingStage` as a specialization of `PipelineStage`. Stage for encoding pixel representations into latent space. Important methods include `__init__`, `component_uses`, `forward`, `build_dedup_fingerprint`.
**CN:** 该类实现 `ImageVAEEncodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for encoding pixel representations into latent space. 其中较重要的方法包括 `__init__`, `component_uses`, `forward`, `build_dedup_fingerprint`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.models.vaes.common`, `sglang.multimodal_gen.runtime.models.vision_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`
- **External / 外部**: `numpy`, `PIL`, `PIL.Image`, `torch`, `diffusers.models.autoencoders.vae`, `diffusers.models.modeling_outputs`, `safetensors.torch`, `av`
- **Stdlib / 标准库**: `inspect`, `dataclasses`, `typing`, `json`, `os`, `io`, `math`
