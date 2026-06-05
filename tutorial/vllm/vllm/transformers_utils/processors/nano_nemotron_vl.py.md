# nano_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/nano_nemotron_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Nano Nemotron VL inputs. / [CN] 为 Nano Nemotron VL 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 10-31: Imports
```python
import math
from abc import ABC, abstractmethod
from collections.abc import Sequence
from dataclasses import dataclass
from functools import cached_property
from typing import Any, TypeVar

import einops
import numpy as np
import numpy.typing as npt
import regex as re
import torch
from PIL import Image
from transformers import BatchFeature, PretrainedConfig, TensorType

from vllm.model_executor.models.parakeet import ParakeetExtractor
from vllm.multimodal.evs import compute_retained_tokens_count
from vllm.multimodal.inputs import AudioItem
from vllm.multimodal.processing.processor import PromptUpdateDetails
from vllm.tokenizers.hf import HfTokenizer

from .internvl import calculate_internvl_targets, get_internvl_target_ratios
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `math`, `abc`, `collections.abc`, `dataclasses`, `functools`, `typing`, external APIs such as `einops`, `numpy`, `numpy.typing`, `regex`, `torch`, `PIL`, `transformers`, and internal vLLM modules such as `vllm.model_executor.models.parakeet`, `vllm.multimodal.evs`, `vllm.multimodal.inputs`, `vllm.multimodal.processing.processor`, `vllm.tokenizers.hf`, `.internvl`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `math`, `abc`, `collections.abc`, `dataclasses`, `functools`, `typing`，外部 API 如 `einops`, `numpy`, `numpy.typing`, `regex`, `torch`, `PIL`, `transformers`，以及 vLLM 内部模块如 `vllm.model_executor.models.parakeet`, `vllm.multimodal.evs`, `vllm.multimodal.inputs`, `vllm.multimodal.processing.processor`, `vllm.tokenizers.hf`, `.internvl`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 33-49: Module state and constants
```python
_T = TypeVar("_T")


IMG_START = "<img>"
IMG_END = "</img>"
IMG_CONTEXT = "<image>"
AUDIO_START = "<so_start>"
AUDIO_END = "<so_end>"
AUDIO_CONTEXT = "<so_embedding>"

# Profiling
# MAX_FRAMES = 16
DEFAULT_NUM_TILES = 12

# Configure PIL to handle large images without warnings
# This prevents DecompressionBombWarning for legitimate large images
Image.MAX_IMAGE_PIXELS = None  # Disable the limit entirely
```
**EN:** This block defines module-level constants/defaults such as `_T`, `IMG_START`, `IMG_END`, `IMG_CONTEXT`, `AUDIO_START`, `AUDIO_END`, ... (+2 more). They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `_T`, `IMG_START`, `IMG_END`, `IMG_CONTEXT`, `AUDIO_START`, `AUDIO_END`, ... (+2 more)。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 54-62: Function `calculate_timestamps`
```python
def calculate_timestamps(
    indices: list[int] | torch.Tensor,
    frame_duration_ms: int,
):
    if not isinstance(indices, list):
        indices = indices.tolist()

    timestamps = [int(i) * frame_duration_ms / 1000.0 for i in indices]
    return timestamps
```
**EN:** This function implements `calculate_timestamps`. Main inputs include `indices`, `frame_duration_ms`.
**CN:** 该函数实现 `calculate_timestamps` 相关逻辑。 主要输入参数包括 `indices`, `frame_duration_ms`。

### Lines 99-148: Function `dynamic_preprocess`
```python
def dynamic_preprocess(
    image: Image.Image,
    *,
    image_size: int = 512,
    max_num_tiles: int = 12,
    use_thumbnail: bool = True,
    norm_mean: torch.Tensor | None = None,
    norm_std: torch.Tensor | None = None,
    dtype: torch.dtype = torch.float32,
) -> torch.Tensor:
    orig_width, orig_height = image.size

    target_ratios = get_internvl_target_ratios(1, max_num_tiles)

    blocks, target_width, target_height = calculate_internvl_targets(
        orig_width=orig_width,
        orig_height=orig_height,
        target_ratios=target_ratios,
        image_size=image_size,
        use_thumbnail=False,
    )

    tensor = _pil_to_nhwc_tensor(image)

    resized_img = _bicubic_resize_and_normalize(
        tensor,
        size=(target_height, target_width),
        norm_mean=norm_mean,
        norm_std=norm_std,
        dtype=dtype,
    )
    B, C, H, W = resized_img.shape
    hp, wp = H // image_size, W // image_size
    patches = (
        resized_img.reshape(B, C, hp, image_size, wp, image_size)
        .permute(0, 2, 4, 1, 3, 5)
        .reshape(B * hp * wp, C, image_size, image_size)
    )

    if use_thumbnail and patches.shape[0] > 1:
        thumb = _bicubic_resize_and_normalize(
            tensor,
            size=(image_size, image_size),
            norm_mean=norm_mean,
            norm_std=norm_std,
            dtype=dtype,
        )
        patches = torch.cat([patches, thumb], dim=0)

    return patches
```
**EN:** This function implements `dynamic_preprocess`. Main inputs include `image`, `image_size`, `max_num_tiles`, `use_thumbnail`, `norm_mean`, ... (+2 more).
**CN:** 该函数实现 `dynamic_preprocess` 相关逻辑。 主要输入参数包括 `image`, `image_size`, `max_num_tiles`, `use_thumbnail`, `norm_mean`, ... (+2 more)。

### Lines 151-186: Function `_compute_aspect_preserving_size`
```python
def _compute_aspect_preserving_size(
    orig_w: int,
    orig_h: int,
    target_num_patches: int,
    patch_size: int,
    downsample_ratio: float,
) -> tuple[int, int]:
    """Compute target pixel dimensions that preserve aspect ratio.

    Mirrors Megatron-LM image_processing.py video frame resizing:
    target area in patch-grid space is *target_num_patches*, distributed
    according to the source aspect ratio, then snapped to a multiple of
    the required divisor (2 for pixel-shuffle).
    """
    aspect_wh = orig_w / max(orig_h, 1)
    ph = round(math.sqrt(target_num_patches / aspect_wh))
    pw = round(math.sqrt(target_num_patches * aspect_wh))
    ph = max(ph, 1)
    pw = max(pw, 1)

    reduction_factor = int(round(1 / downsample_ratio))
    required_divisor = reduction_factor  # 2 for pixel-shuffle
    if required_divisor > 1:
        rem_h = ph % required_divisor
        rem_w = pw % required_divisor
        ph_up = ph + (required_divisor - rem_h if rem_h else 0)
        ph_down = ph - rem_h
        pw_up = pw + (required_divisor - rem_w if rem_w else 0)
        pw_down = pw - rem_w
        if ph_up * pw_up <= target_num_patches:
            ph, pw = ph_up, pw_up
        else:
            ph = max(required_divisor, ph_down)
            pw = max(required_divisor, pw_down)

    return pw * patch_size, ph * patch_size  # (width, height) in pixels
```
**EN:** This private function implements `_compute_aspect_preserving_size`. The docstring states that Compute target pixel dimensions that preserve aspect ratio. Main inputs include `orig_w`, `orig_h`, `target_num_patches`, `patch_size`, `downsample_ratio`.
**CN:** 该私有函数实现 `_compute_aspect_preserving_size` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `orig_w`, `orig_h`, `target_num_patches`, `patch_size`, `downsample_ratio`。

### Lines 189-220: Function `get_video_target_size_and_feature_size`
```python
def get_video_target_size_and_feature_size(
    orig_w: int,
    orig_h: int,
    target_patches: int,
    maintain_aspect_ratio: bool,
    patch_size: int,
    downsample_ratio: float,
) -> tuple[int, int, int]:
    """Compute target (width, height) and feature_size for video resize and token count.

    Used by video_to_pixel_values (resize) and get_video_replacement_internvl
    (seq length calc) so both use the same dimensions.
    """
    if maintain_aspect_ratio:
        target_w, target_h = _compute_aspect_preserving_size(
            orig_w=orig_w,
            orig_h=orig_h,
            target_num_patches=target_patches,
            patch_size=patch_size,
            downsample_ratio=downsample_ratio,
        )
    else:
        reduction_factor = int(round(1 / downsample_ratio))
        side = int(math.sqrt(target_patches))
        side = max(reduction_factor, (side // reduction_factor) * reduction_factor)
        target_w = side * patch_size
        target_h = side * patch_size

    feature_size = int((target_h // patch_size) * downsample_ratio) * int(
        (target_w // patch_size) * downsample_ratio
    )
    return target_w, target_h, feature_size
```
**EN:** This function retrieves video target size and feature size. The docstring states that Compute target (width, height) and feature_size for video resize and token count. Main inputs include `orig_w`, `orig_h`, `target_patches`, `maintain_aspect_ratio`, `patch_size`, ... (+1 more).
**CN:** 该函数负责完成 `get_video_target_size_and_feature_size` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `orig_w`, `orig_h`, `target_patches`, `maintain_aspect_ratio`, `patch_size`, ... (+1 more)。

### Lines 223-254: Function `video_to_pixel_values`
```python
def video_to_pixel_values(
    video: npt.NDArray,
    *,
    input_size: int,
    video_target_num_patches: int | None = None,
    video_maintain_aspect_ratio: bool = False,
    patch_size: int = 16,
    downsample_ratio: float = 0.5,
    norm_mean: torch.Tensor | None = None,
    norm_std: torch.Tensor | None = None,
    dtype: torch.dtype = torch.float32,
) -> torch.Tensor:
    """Convert video ndarray (T, H, W, C) to normalized pixel tensor (T, C, H, W)."""
    orig_h, orig_w = video.shape[1], video.shape[2]
    size: tuple[int, int] | None = None

    if video_target_num_patches is not None:
        tw, th, _ = get_video_target_size_and_feature_size(
            orig_w=orig_w,
            orig_h=orig_h,
            target_patches=video_target_num_patches,
            maintain_aspect_ratio=video_maintain_aspect_ratio,
            patch_size=patch_size,
            downsample_ratio=downsample_ratio,
        )
        if orig_h != th or orig_w != tw:
            size = (th, tw)
    elif orig_h != input_size or orig_w != input_size:
        size = (input_size, input_size)

    tensor = torch.from_numpy(video)
    return _bicubic_resize_and_normalize(tensor, size, norm_mean, norm_std, dtype)
```
**EN:** This function implements `video_to_pixel_values`. The docstring states that Convert video ndarray (T, H, W, C) to normalized pixel tensor (T, C, H, W). Main inputs include `video`, `input_size`, `video_target_num_patches`, `video_maintain_aspect_ratio`, `patch_size`, ... (+4 more).
**CN:** 该函数实现 `video_to_pixel_values` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `video`, `input_size`, `video_target_num_patches`, `video_maintain_aspect_ratio`, `patch_size`, ... (+4 more)。

### Lines 257-575: Class `DynamicResolutionImageTiler`
```python
class DynamicResolutionImageTiler:
    CONV_MERGING = False
    PIXEL_SHUFFLE = True
    USE_THUMBNAIL = False

    def __init__(
        self,
        *,
        max_model_len: int,
        patch_size: int,
        min_num_patches: int,
        max_num_patches: int,
        downsample_ratio: int,
        norm_mean: Sequence[float],
        norm_std: Sequence[float],
        factor_max: float = 1.0,
        use_thumbnail: bool = False,
    ) -> None:
        assert use_thumbnail is False, "use_thumbnail is not supported"
        self._patch_size: int = patch_size
        self._max_model_len = max_model_len
        self._min_num_patches = min_num_patches
        self._max_num_patches = max_num_patches if max_num_patches > 0 else float("inf")
        self._factor_max = factor_max
        self.norm_mean = torch.tensor(norm_mean).reshape(3, 1, 1)
        self.norm_std = torch.tensor(norm_std).reshape(3, 1, 1)
        assert downsample_ratio < 1
        reduction_factor = 1 / downsample_ratio
        assert reduction_factor == 2.0
        self._downsample_ratio = int(reduction_factor) ** (
            self.PIXEL_SHUFFLE + self.CONV_MERGING
        )
        assert self._downsample_ratio == 2

    def _get_num_embeddings(self, width: int, height: int) -> int:
        num_patches = (width // self._patch_size) * (height // self._patch_size)
        num_tokens = num_patches // (self._downsample_ratio**2)
        return num_tokens

    def width_and_height_for_max_num_tokens_available(
# ... omitted for brevity ...
                x,
                "c (py yy) (px xx) -> (py px) (c yy xx)",
                py=py,
                yy=patch_size,
                px=px,
                xx=patch_size,
            )
            return x

        imgs = [rearrange_img(img) for img in images]
        pixel_values_flat = torch.cat(imgs, dim=0).unsqueeze(0)
        return pixel_values_flat
```
**EN:** Defines `DynamicResolutionImageTiler`, a processor-related class. Key methods include `__init__`, `_get_num_embeddings`, `width_and_height_for_max_num_tokens_available`, `max_num_tokens_available`, `_images_to_pixel_values_lst`, ... (+4 more).
**CN:** 定义 `DynamicResolutionImageTiler`，这是一个处理器相关类。 关键方法包括 `__init__`, `_get_num_embeddings`, `width_and_height_for_max_num_tokens_available`, `max_num_tokens_available`, `_images_to_pixel_values_lst`, ... (+4 more)。

### Lines 578-760: Class `BaseNanoNemotronVLProcessor`
```python
class BaseNanoNemotronVLProcessor(ABC):
    """
    This model doesn't define its own HF processor,
    so we implement our own one here.

    The code to insert image tokens is based on:
    https://huggingface.co/OpenGVLab/InternVL2-1B/blob/main/modeling_internvl_chat.py#L252
    """

    def __init__(
        self,
        config: PretrainedConfig,
        tokenizer: HfTokenizer,
        *args,
        max_model_len: int,
        max_num_tiles: int | None = None,
        **kwargs,
    ) -> None:
        super().__init__()

        self.config = config
        self.tokenizer = tokenizer

        self.max_num_tiles = max_num_tiles or DEFAULT_NUM_TILES
        image_size: int = config.force_image_size
        patch_size: int = config.patch_size
        downsample_ratio: int = config.downsample_ratio

        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (downsample_ratio**2)
        )
        self.image_size = image_size
        self.use_thumbnail: bool = config.use_thumbnail
        self.norm_mean = torch.Tensor(config.norm_mean).reshape(1, 3, 1, 1)
        self.norm_std = torch.Tensor(config.norm_std).reshape(1, 3, 1, 1)

        self.dynamic_tiler: DynamicResolutionImageTiler | None = None
        if self.use_dynamic_resolution(config):
            self.dynamic_tiler = DynamicResolutionImageTiler(
                max_model_len=max_model_len,
# ... omitted for brevity ...

    @abstractmethod
    def __call__(
        self,
        text: str | list[str] | None = None,
        images: Image.Image | list[Image.Image] | None = None,
        *,
        return_tensors: str | TensorType | None = None,
        max_num_tiles: int | None = None,
        **kwargs,
    ) -> BatchFeature:
        raise NotImplementedError
```
**EN:** Defines `BaseNanoNemotronVLProcessor`, a processor-related class derived from `ABC`. The class docstring highlights that This model doesn't define its own HF processor, Key methods include `__init__`, `use_dynamic_resolution`, `image_token_id`, `get_image_repl`, `get_num_image_tokens`, ... (+4 more).
**CN:** 定义 `BaseNanoNemotronVLProcessor`，这是一个处理器相关类，继承自 `ABC`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `use_dynamic_resolution`, `image_token_id`, `get_image_repl`, `get_num_image_tokens`, ... (+4 more)。

### Lines 763-1207: Class `NanoNemotronVLProcessor`
```python
class NanoNemotronVLProcessor(BaseNanoNemotronVLProcessor):
    """
    HF Processor with extended video processing logic.
    Code for video processing is adapted from video example:
    https://huggingface.co/OpenGVLab/InternVL3-1B#inference-with-transformers
    """

    def __init__(
        self,
        config: PretrainedConfig,
        tokenizer: HfTokenizer,
        *,
        max_model_len: int,
        max_num_tiles: int | None = None,
        video_token: str | None = None,
        video_pruning_rate: float | None = None,
        use_audio_in_video: bool = False,
    ) -> None:
        super().__init__(
            config=config,
            tokenizer=tokenizer,
            max_model_len=max_model_len,
            max_num_tiles=max_num_tiles,
        )
        # add extra video token for video processing
        self.video_token = video_token
        self.video_pruning_rate = video_pruning_rate
        self.use_audio_in_video = use_audio_in_video

        # Video params live exclusively in vision_config
        vision_config = getattr(config, "vision_config", config)
        self.video_temporal_patch_size: int = getattr(
            vision_config, "video_temporal_patch_size", 1
        )
        self.video_maintain_aspect_ratio: bool = getattr(
            vision_config, "video_maintain_aspect_ratio", False
        )

        # Resolve video frame target size: exactly one of video_target_num_patches
        # or video_target_img_size may be set (mirrors Megatron's
# ... omitted for brevity ...

        # Tokenize each component independently to avoid tokenizer merging tokens
        # across boundaries. This ensures consistent tokenization regardless of
        # num_tokens_per_frame values.
        all_token_ids = []
        for i, num_tokens in enumerate(tokens_per_frame):
            all_token_ids.extend(frame_separators_tokenized[i])
            all_token_ids.extend(img_start_token_ids)
            all_token_ids.extend(img_context_token_ids * num_tokens)
            all_token_ids.extend(img_end_token_ids)

        return PromptUpdateDetails.from_seq(all_token_ids)
```
**EN:** Defines `NanoNemotronVLProcessor`, a processor-related class derived from `BaseNanoNemotronVLProcessor`. The class docstring highlights that HF Processor with extended video processing logic. Key methods include `__init__`, `num_video_token`, `supports_video`, `video_token_id`, `image_token_id`, ... (+7 more).
**CN:** 定义 `NanoNemotronVLProcessor`，这是一个处理器相关类，继承自 `BaseNanoNemotronVLProcessor`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `num_video_token`, `supports_video`, `video_token_id`, `image_token_id`, ... (+7 more)。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `math`, `abc`, `collections.abc`, `dataclasses`, `functools`, `typing`.
- **CN:** 标准库模块：`math`, `abc`, `collections.abc`, `dataclasses`, `functools`, `typing`。
- **EN:** External packages: `einops`, `numpy`, `numpy.typing`, `regex`, `torch`, `PIL`, `transformers`.
- **CN:** 外部依赖包：`einops`, `numpy`, `numpy.typing`, `regex`, `torch`, `PIL`, `transformers`。
- **EN:** Internal modules: `vllm.model_executor.models.parakeet`, `vllm.multimodal.evs`, `vllm.multimodal.inputs`, `vllm.multimodal.processing.processor`, `vllm.tokenizers.hf`, `.internvl`.
- **CN:** 内部模块：`vllm.model_executor.models.parakeet`, `vllm.multimodal.evs`, `vllm.multimodal.inputs`, `vllm.multimodal.processing.processor`, `vllm.tokenizers.hf`, `.internvl`。
