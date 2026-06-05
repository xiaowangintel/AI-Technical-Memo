# qwen_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/qwen_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for qwen vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 qwen vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37: Imports dependencies
```python
import math
import os
import re
import time
from typing import List, Union

import numpy as np
import torch
import torchvision
from PIL import Image
from torchvision.transforms import InterpolationMode

from sglang.srt.environ import envs
from sglang.srt.layers.rotary_embedding import MRotaryEmbedding
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.interns2preview import InternS2PreviewForConditionalGeneration
from sglang.srt.models.qwen2_5_vl import Qwen2_5_VLForConditionalGeneration
from sglang.srt.models.qwen2_vl import Qwen2VLForConditionalGeneration
from sglang.srt.models.qwen3_5 import (
    Qwen3_5ForConditionalGeneration,
    Qwen3_5MoeForConditionalGeneration,
)
from sglang.srt.models.qwen3_omni_moe import Qwen3OmniMoeForConditionalGeneration
from sglang.srt.models.qwen3_vl import Qwen3VLForConditionalGeneration
from sglang.srt.models.qwen3_vl_moe import Qwen3VLMoeForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
from sglang.srt.utils.video_decoder import VideoDecoderWrapper
from sglang.utils import logger
```
**EN:** This block groups related imports for the module, including math, os, re, time, typing.List, and 23 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, os, re, time, typing.List 等 23 项，为后续代码准备所需名称。

### Lines 38-38: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 39-43: Declares IMAGE_FACTOR, MIN_PIXELS, MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE
```python
IMAGE_FACTOR = 28
MIN_PIXELS = 4 * 28 * 28
MAX_PIXELS = envs.SGLANG_IMAGE_MAX_PIXELS.get()
MAX_RATIO = 200
RESIZE_RESAMPLE = getattr(Image, envs.SGLANG_RESIZE_RESAMPLE.get(), None)
```
**EN:** This block initializes a related set of values in the module, including IMAGE_FACTOR, MIN_PIXELS, MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 IMAGE_FACTOR, MIN_PIXELS, MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE。将这些赋值集中在一起有助于理解周边配置。

### Lines 44-48: Evaluates conditional branch a runtime condition
```python
if envs.SGLANG_RESIZE_RESAMPLE.is_set() and RESIZE_RESAMPLE is None:
    logger.warning(
        f"Invalid RESIZE_RESAMPLE value: '{envs.SGLANG_RESIZE_RESAMPLE.get()}'. "
        f"Ignoring and using default."
    )
```
**EN:** This block executes conditionally based on `a runtime condition` and selects behavior for the surrounding module.
**CN:** 该代码块根据 `a runtime condition` 的结果进行条件分支，为周围的 模块 选择不同的行为。

### Lines 49-51: Declares VIDEO_TOTAL_PIXELS
```python
VIDEO_TOTAL_PIXELS = int(
    float(os.environ.get("VIDEO_MAX_PIXELS", 128000 * 28 * 28 * 0.9))
)
```
**EN:** This statement initializes VIDEO_TOTAL_PIXELS in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 VIDEO_TOTAL_PIXELS。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-58: Declares VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES, and 1 more
```python
VIDEO_MIN_PIXELS = 128 * 28 * 28
VIDEO_MAX_PIXELS = 768 * 28 * 28
FRAME_FACTOR = 2
FPS = 2.0
FPS_MIN_FRAMES = 4
FPS_MAX_FRAMES = 768
```
**EN:** This block initializes a related set of values in the module, including VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 59-60: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 61-91: Defines function smart_resize
```python
def smart_resize(
    height: int,
    width: int,
    factor: int = IMAGE_FACTOR,
    min_pixels: int = MIN_PIXELS,
    max_pixels: int = MAX_PIXELS,
) -> tuple[int, int]:
    """
    Rescales the image so that the following conditions are met:

    1. Both dimensions (height and width) are divisible by 'factor'.

    2. The total number of pixels is within the range ['min_pixels', 'max_pixels'].

    3. The aspect ratio of the image is maintained as closely as possible.
    """
    if max(height, width) / min(height, width) > MAX_RATIO:
        raise ValueError(
            f"absolute aspect ratio must be smaller than {MAX_RATIO}, got {max(height, width) / min(height, width)}"
        )
    h_bar = max(factor, round_by_factor(height, factor))
    w_bar = max(factor, round_by_factor(width, factor))
    if h_bar * w_bar > max_pixels:
        beta = math.sqrt((height * width) / max_pixels)
        h_bar = floor_by_factor(height / beta, factor)
        w_bar = floor_by_factor(width / beta, factor)
    elif h_bar * w_bar < min_pixels:
        beta = math.sqrt(min_pixels / (height * width))
        h_bar = ceil_by_factor(height * beta, factor)
        w_bar = ceil_by_factor(width * beta, factor)
    return h_bar, w_bar
```
**EN:** This block defines function `smart_resize`. Parameters: height, width, factor, min_pixels, max_pixels. Rescales the image so that the following conditions are met: 1. Both dimensions (height and width) are divisible by 'factor'.
**CN:** 该代码块定义函数 `smart_resize`。 参数包括 height、width、factor、min_pixels、max_pixels。 文档字符串摘要：Rescales the image so that the following conditions are met: 1. Both dimensions (height and width) are divisible by 'factor'.

### Lines 92-93: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 94-96: Defines function round_by_factor
```python
def round_by_factor(number: int, factor: int) -> int:
    """Returns the closest integer to 'number' that is divisible by 'factor'."""
    return round(number / factor) * factor
```
**EN:** This block defines function `round_by_factor`. Parameters: number, factor. Returns the closest integer to 'number' that is divisible by 'factor'.
**CN:** 该代码块定义函数 `round_by_factor`。 参数包括 number、factor。 文档字符串摘要：Returns the closest integer to 'number' that is divisible by 'factor'.

### Lines 97-98: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 99-101: Defines function ceil_by_factor
```python
def ceil_by_factor(number: int, factor: int) -> int:
    """Returns the smallest integer greater than or equal to 'number' that is divisible by 'factor'."""
    return math.ceil(number / factor) * factor
```
**EN:** This block defines function `ceil_by_factor`. Parameters: number, factor. Returns the smallest integer greater than or equal to 'number' that is divisible by 'factor'.
**CN:** 该代码块定义函数 `ceil_by_factor`。 参数包括 number、factor。 文档字符串摘要：Returns the smallest integer greater than or equal to 'number' that is divisible by 'factor'.

### Lines 102-103: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 104-106: Defines function floor_by_factor
```python
def floor_by_factor(number: int, factor: int) -> int:
    """Returns the largest integer less than or equal to 'number' that is divisible by 'factor'."""
    return math.floor(number / factor) * factor
```
**EN:** This block defines function `floor_by_factor`. Parameters: number, factor. Returns the largest integer less than or equal to 'number' that is divisible by 'factor'.
**CN:** 该代码块定义函数 `floor_by_factor`。 参数包括 number、factor。 文档字符串摘要：Returns the largest integer less than or equal to 'number' that is divisible by 'factor'.

### Lines 107-108: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 109-154: Defines function smart_nframes
```python
def smart_nframes(
    ele: dict,
    total_frames: int,
    video_fps: int | float,
) -> int:
    """calculate the number of frames for video used for model inputs.

    Args:
        ele (dict): a dict contains the configuration of video.
            support either `fps` or `nframes`:
                - nframes: the number of frames to extract for model inputs.
                - fps: the fps to extract frames for model inputs.
                    - min_frames: the minimum number of frames of the video, only used when fps is provided.
                    - max_frames: the maximum number of frames of the video, only used when fps is provided.
        total_frames (int): the original total number of frames of the video.
        video_fps (int | float): the original fps of the video.

    Raises:
        ValueError: nframes should in interval [FRAME_FACTOR, total_frames].

    Returns:
        int: the number of frames for video used for model inputs.
    """
    assert not (
        "fps" in ele and "nframes" in ele
    ), "Only accept either `fps` or `nframes`"
    if "nframes" in ele:
        nframes = round_by_factor(ele["nframes"], FRAME_FACTOR)
    else:
        fps = ele.get("fps", FPS)
        min_frames = ceil_by_factor(ele.get("min_frames", FPS_MIN_FRAMES), FRAME_FACTOR)
        max_frames = floor_by_factor(
            ele.get("max_frames", min(FPS_MAX_FRAMES, total_frames)), FRAME_FACTOR
        )
        nframes = total_frames / video_fps * fps
        if nframes > total_frames:
            logger.warning(
                f"smart_nframes: nframes[{nframes}] > total_frames[{total_frames}]"
            )
        nframes = min(min(max(nframes, min_frames), max_frames), total_frames)
        nframes = floor_by_factor(nframes, FRAME_FACTOR)
    if not (FRAME_FACTOR <= nframes and nframes <= total_frames):
        raise ValueError(
            f"nframes should in interval [{FRAME_FACTOR}, {total_frames}], but got {nframes}."
        )
    return nframes
```
**EN:** This block defines function `smart_nframes`. Parameters: ele, total_frames, video_fps. calculate the number of frames for video used for model inputs. Args: ele (dict): a dict contains the configuration of video.
**CN:** 该代码块定义函数 `smart_nframes`。 参数包括 ele、total_frames、video_fps。 文档字符串摘要：calculate the number of frames for video used for model inputs. Args: ele (dict): a dict contains the configuration of video.

### Lines 155-157: Comments and module notes
```python


# process video, qwen-specific
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 158-237: Defines async function preprocess_video
```python
async def preprocess_video(
    vr,
    image_factor: int = IMAGE_FACTOR,
    video_config: dict = {},
) -> torch.Tensor:
    # preprocessed video
    is_video_obj = isinstance(vr, VideoDecoderWrapper)
    if not is_video_obj:
        return vr, None
    entry_time = time.perf_counter()

    total_frames, video_fps = len(vr), vr.avg_fps

    nframes = smart_nframes(
        video_config, total_frames=total_frames, video_fps=video_fps
    )
    idx = np.linspace(0, total_frames - 1, num=nframes, dtype=np.int64)
    idx = np.unique(idx)

    video = vr.get_frames_as_tensor(idx.tolist())

    video = video.permute(0, 3, 1, 2)  # NHWC -> TCHW

    nframes, _, height, width = video.shape
    min_pixels = video_config.get("min_pixels", VIDEO_MIN_PIXELS)
    total_pixels = video_config.get("total_pixels", VIDEO_TOTAL_PIXELS)
    max_pixels = max(
        min(
            video_config.get("max_pixels", VIDEO_MAX_PIXELS),
            total_pixels / nframes * FRAME_FACTOR,
        ),
        int(min_pixels * 1.05),
    )

    get_batch_time = time.perf_counter()

    max_pixels_supposed = video_config.get("max_pixels", max_pixels)

    if max_pixels_supposed > max_pixels:
        logger.warning(
            f"The given max_pixels[{max_pixels_supposed}] exceeds limit[{max_pixels}]."
        )
    max_pixels = min(max_pixels_supposed, max_pixels)
    if "resized_height" in video_config and "resized_width" in video_config:
        resized_height, resized_width = smart_resize(
            video_config["resized_height"],
            video_config["resized_width"],
            factor=image_factor,
        )
    else:
        resized_height, resized_width = smart_resize(
            height,
            width,
            factor=image_factor,
            min_pixels=min_pixels,
            max_pixels=max_pixels,
        )
    smart_resize_time = time.perf_counter()
    video = torchvision.transforms.functional.resize(
        video,
        [resized_height, resized_width],
        interpolation=InterpolationMode.BILINEAR,
    )
    video = video.pin_memory()
    video_metadata = {
        "fps": video_fps,
        "duration": total_frames / video_fps,
        "total_num_frames": total_frames,
        "frames_indices": idx,
        "video_backend": "torchvision",
    }
    torchvision_resize_time = time.perf_counter()
    logger.debug(
        f"[preprocess_video Perf], "
        f"get_batch_time: {(get_batch_time - entry_time) * 1000:.2f} ms, "
        f"smart_resize_time: {(smart_resize_time - get_batch_time) * 1000:.2f} ms, "
        f"torchvision_resize_time: {(torchvision_resize_time - smart_resize_time) * 1000:.2f} ms, "
        f"total_time: {(torchvision_resize_time - entry_time) * 1000:.2f} ms"
    )
    return video, video_metadata
```
**EN:** This block defines async function `preprocess_video`. Parameters: vr, image_factor, video_config.
**CN:** 该代码块定义异步函数 `preprocess_video`。 参数包括 vr、image_factor、video_config。

### Lines 238-240: Comments and module notes
```python


# Compatible with Qwen-VL & Qwen-Omni Series
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 241-241: Declares class QwenVLImageProcessor
```python
class QwenVLImageProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `QwenVLImageProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor.
**CN:** 该代码块声明类 `QwenVLImageProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。

### Lines 242-252: Declares supports_transformers_backend, models
```python
    supports_transformers_backend = True
    models = [
        Qwen2VLForConditionalGeneration,
        Qwen2_5_VLForConditionalGeneration,
        Qwen3VLForConditionalGeneration,
        Qwen3VLMoeForConditionalGeneration,
        Qwen3_5ForConditionalGeneration,
        Qwen3_5MoeForConditionalGeneration,
        InternS2PreviewForConditionalGeneration,
        Qwen3OmniMoeForConditionalGeneration,
    ]
```
**EN:** This block initializes a related set of values in the QwenVLImageProcessor, including supports_transformers_backend, models. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 QwenVLImageProcessor 中初始化一组相关值，包括 supports_transformers_backend, models。将这些赋值集中在一起有助于理解周边配置。

### Lines 253-253: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the QwenVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 QwenVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 254-281: Defines function QwenVLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        self.model_type = hf_config.model_type
        if hf_config.model_type == "qwen3_omni_moe":
            hf_config = hf_config.thinker_config

        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.IM_START_TOKEN_ID = hf_config.vision_start_token_id
        self.IM_END_TOKEN_ID = hf_config.vision_end_token_id
        self.IM_TOKEN_ID = hf_config.image_token_id
        self.VIDEO_TOKEN_ID = hf_config.video_token_id

        self.vision_start_token_id = hf_config.vision_start_token_id
        self.vision_end_token_id = getattr(hf_config, "vision_end_token_id", None)

        self.audio_start_token_id = getattr(hf_config, "audio_start_token_id", None)
        self.audio_token_id = getattr(hf_config, "audio_token_id", None)

        self.mm_tokens = MultimodalSpecialTokens(
            image_token="<|vision_start|><|image_pad|><|vision_end|>",
            image_token_id=hf_config.image_token_id,
            # The regex that matches expanded image tokens.
            image_token_regex=re.compile(
                r"<\|vision_start\|>(?:<\|image_pad\|>)+<\|vision_end\|>"
            ),
            video_token_id=self.VIDEO_TOKEN_ID,
            audio_token_id=self.audio_token_id,
        ).build(_processor)
```
**EN:** This block defines function `QwenVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 282-282: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the QwenVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 QwenVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 283-309: Defines function QwenVLImageProcessor.build_input_ids_with_timestamps (part 1)
```python
    def build_input_ids_with_timestamps(
        self, prompt, embeddings, img_grid_thw, video_grid_thw, video_timestamps
    ):
        """
        Build input_ids with timestamps for qwen3_vl models.
        """
        if not isinstance(prompt, list):
            prompt = self._processor.tokenizer.encode(prompt)

        img_token_id = getattr(self, "IM_TOKEN_ID", None)
        video_token_id = getattr(self, "VIDEO_TOKEN_ID", None)
        audio_token_id = getattr(self, "audio_token_id", None)
        spatial_merge_size = getattr(self, "spatial_merge_size", 1)
        vision_start_token_id = getattr(self, "vision_start_token_id", None)
        vision_end_token_id = getattr(self, "vision_end_token_id", None)

        input_ids = []
        offsets = []
        modality_list = []
        cur_idx = 0

        vision_start_indices = []
        for i in range(len(prompt) - 1):
            if img_token_id is not None and prompt[i + 1] == img_token_id:
                vision_start_indices.append((i, Modality.IMAGE))
            elif video_token_id is not None and prompt[i + 1] == video_token_id:
                vision_start_indices.append((i, Modality.VIDEO))
```
**EN:** This block defines function `QwenVLImageProcessor.build_input_ids_with_timestamps`. Parameters: self, prompt, embeddings, img_grid_thw, video_grid_thw, video_timestamps. Build input_ids with timestamps for qwen3_vl models. This subsection covers lines 283-309 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.build_input_ids_with_timestamps`。 参数包括 self、prompt、embeddings、img_grid_thw、video_grid_thw、video_timestamps。 文档字符串摘要：Build input_ids with timestamps for qwen3_vl models. 本小节覆盖同一逻辑块中的第 283-309 行。

### Lines 310-310: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 311-372: Defines function QwenVLImageProcessor.build_input_ids_with_timestamps (part 2)
```python
        img_idx = 0
        video_idx = 0
        model_type = getattr(self, "model_type", None)
        for mm_start_idx, modality in vision_start_indices:
            modality_list.append(modality)
            video_tokens = None
            if modality == Modality.IMAGE:
                mm_token_num = img_grid_thw[img_idx].prod() // (spatial_merge_size**2)
                mm_token_id = img_token_id
                img_idx += 1
            elif modality == Modality.VIDEO:
                curr_timestamps = video_timestamps[video_idx]
                num_frames = video_grid_thw[video_idx][0]
                frame_seqlen = video_grid_thw[video_idx][1:].prod().item() // (
                    spatial_merge_size**2
                )
                video_tokens = []
                _current_offset = len(input_ids) + mm_start_idx + 1 - cur_idx
                # take single frame as one mm_item
                for frame_idx in range(num_frames):
                    if frame_idx > 0:
                        modality_list.append(Modality.VIDEO)
                    curr_time = curr_timestamps[frame_idx]
                    timestamp_text = f"<{curr_time:.1f} seconds>"
                    timestamp_tokens = self._processor.tokenizer.encode(
                        timestamp_text, add_special_tokens=False
                    )
                    video_tokens.extend(timestamp_tokens)
                    _current_offset += len(timestamp_tokens)
                    if vision_start_token_id is not None:
                        video_tokens.append(vision_start_token_id)
                        _current_offset += 1
                    video_tokens.extend([video_token_id] * frame_seqlen)
                    if vision_end_token_id is not None:
                        video_tokens.append(vision_end_token_id)
                    offsets.append(
                        (_current_offset, _current_offset + frame_seqlen - 1)
                    )
                    _current_offset += (
                        frame_seqlen + 1
                        if vision_end_token_id is not None
                        else frame_seqlen
                    )  # for vision_end_token_id
                mm_token_num = len(video_tokens)
                mm_token_id = None
                video_idx += 1
            else:
                logger.warning(
                    f"{modality} modality is not supported for qwen3_vl models with timestamps."
                )
                continue
            assert cur_idx <= mm_start_idx
            input_ids.extend(prompt[cur_idx : mm_start_idx + 1])
            if modality == Modality.VIDEO:
                input_ids.extend(video_tokens)
            else:
                mm_offset_start = len(input_ids)
                input_ids.extend([mm_token_id] * mm_token_num)
                offsets.append((mm_offset_start, len(input_ids) - 1))
            cur_idx = mm_start_idx + 2  # jump to vision_end_id
        else:
            input_ids.extend(prompt[cur_idx:])
```
**EN:** This block defines function `QwenVLImageProcessor.build_input_ids_with_timestamps`. Parameters: self, prompt, embeddings, img_grid_thw, video_grid_thw, video_timestamps. Build input_ids with timestamps for qwen3_vl models. This subsection covers lines 311-372 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.build_input_ids_with_timestamps`。 参数包括 self、prompt、embeddings、img_grid_thw、video_grid_thw、video_timestamps。 文档字符串摘要：Build input_ids with timestamps for qwen3_vl models. 本小节覆盖同一逻辑块中的第 311-372 行。

### Lines 373-373: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 374-374: Defines function QwenVLImageProcessor.build_input_ids_with_timestamps (part 3)
```python
        return input_ids, offsets, modality_list
```
**EN:** This block defines function `QwenVLImageProcessor.build_input_ids_with_timestamps`. Parameters: self, prompt, embeddings, img_grid_thw, video_grid_thw, video_timestamps. Build input_ids with timestamps for qwen3_vl models. This subsection covers lines 374-374 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.build_input_ids_with_timestamps`。 参数包括 self、prompt、embeddings、img_grid_thw、video_grid_thw、video_timestamps。 文档字符串摘要：Build input_ids with timestamps for qwen3_vl models. 本小节覆盖同一逻辑块中的第 374-374 行。

### Lines 375-375: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the QwenVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 QwenVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 376-399: Defines function QwenVLImageProcessor.compute_mrope_positions
```python
    def compute_mrope_positions(self, input_ids, mm_items):
        image_grid_thw = None
        video_grid_thw = None
        for item in mm_items:
            if "image_grid_thw" in item.model_specific_data:
                image_grid_thw = item.model_specific_data["image_grid_thw"]
            if "video_grid_thw" in item.model_specific_data:
                video_grid_thw = item.model_specific_data["video_grid_thw"]

        input_ids_tensor = torch.tensor(input_ids, dtype=torch.long).unsqueeze(0)
        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index(
            spatial_merge_size=self.hf_config.vision_config.spatial_merge_size,
            image_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            vision_start_token_id=self.vision_start_token_id,
            model_type=self.model_type,
            tokens_per_second=getattr(
                self.hf_config.vision_config, "tokens_per_second", None
            ),
            input_ids=input_ids_tensor,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
        )
        return mrope_positions.squeeze(1), mrope_position_delta
```
**EN:** This block defines function `QwenVLImageProcessor.compute_mrope_positions`. Parameters: self, input_ids, mm_items.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.compute_mrope_positions`。 参数包括 self、input_ids、mm_items。

### Lines 400-400: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the QwenVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 QwenVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 401-469: Defines function QwenVLImageProcessor.get_mm_data (part 1)
```python
    def get_mm_data(self, prompt, embeddings, **kwargs):
        img_grid_thw = kwargs.get("img_grid_thw", None)
        video_grid_thw = kwargs.get("video_grid_thw", None)
        audio_feature_lens = kwargs.get("audio_feature_lens", None)
        video_timestamps = kwargs.get("video_timestamps", None)
        second_per_grid_ts = kwargs.get("second_per_grid_ts", None)

        audio_seq_lens = None
        if audio_feature_lens is not None:
            if self.model_type == "qwen3_omni_moe":
                # apply _get_feat_extract_lengths to get seq_lens
                input_lengths_leave = audio_feature_lens % 100
                feat_lengths = (input_lengths_leave - 1) // 2 + 1
                audio_seq_lens = (
                    ((feat_lengths - 1) // 2 + 1 - 1) // 2
                    + 1
                    + (audio_feature_lens // 100) * 13
                )
            elif self.model_type == "qwen2_5_omni":
                audio_seq_lens = (audio_feature_lens - 1) // 2 + 1
                audio_seq_lens = (audio_seq_lens - 2) // 2 + 1

        if (
            self.model_type
            in [
                "qwen3_vl",
                "qwen3_vl_moe",
                "qwen3_5",
                "qwen3_5_moe",
                "intern_s2_preview",
            ]
            and video_timestamps is not None
        ):
            input_ids, offsets, modality_list = self.build_input_ids_with_timestamps(
                prompt, embeddings, img_grid_thw, video_grid_thw, video_timestamps
            )
        else:
            input_ids, offsets, modality_list = self.build_input_ids(
                prompt, img_grid_thw, video_grid_thw, audio_seq_lens=audio_seq_lens
            )
        assert all(isinstance(modality, Modality) for modality in modality_list)

        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index(
            spatial_merge_size=self.hf_config.vision_config.spatial_merge_size,
            image_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            vision_start_token_id=self.vision_start_token_id,
            model_type=self.model_type,
            input_ids=torch.tensor(input_ids, dtype=torch.long).unsqueeze(0),
            image_grid_thw=img_grid_thw,
            video_grid_thw=video_grid_thw,
            second_per_grid_ts=second_per_grid_ts,
            use_audio_in_video=False,
            audio_seqlens=(
                audio_feature_lens if self.model_type == "qwen3_omni_moe" else None
            ),
            audio_token_id=getattr(self.hf_config, "audio_token_id", None),
            audio_start_token_id=self.audio_start_token_id,
            position_id_per_seconds=getattr(
                self.hf_config, "position_id_per_seconds", None
            ),
            tokens_per_second=getattr(
                self.hf_config.vision_config, "tokens_per_second", None
            ),
        )
        mrope_positions = mrope_positions.squeeze(1)

        mm_items = []
        consumed_per_modality = {}
```
**EN:** This block defines function `QwenVLImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings. This subsection covers lines 401-469 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。 本小节覆盖同一逻辑块中的第 401-469 行。

### Lines 470-470: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 471-485: Defines function QwenVLImageProcessor.get_mm_data (part 2)
```python
        for modality, offset in zip(modality_list, offsets):
            num_tokens = offset[1] - offset[0] + 1
            embedding_start = consumed_per_modality.get(modality, 0)
            embedding_slice = embeddings[modality][
                embedding_start : embedding_start + num_tokens
            ]
            consumed_per_modality[modality] = embedding_start + num_tokens
            logger.info(f"Get embedding slice for {modality}, num_tokens={num_tokens}")
            mm_items.append(
                MultimodalDataItem(
                    modality=modality,
                    offsets=[offset],
                    precomputed_embeddings=embedding_slice,
                )
            )
```
**EN:** This block defines function `QwenVLImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings. This subsection covers lines 471-485 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。 本小节覆盖同一逻辑块中的第 471-485 行。

### Lines 486-486: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 487-497: Defines function QwenVLImageProcessor.get_mm_data (part 3)
```python
        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=mm_items,
            im_start_id=self.IM_START_TOKEN_ID,
            im_end_id=self.IM_END_TOKEN_ID,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
            mrope_positions=mrope_positions,
            mrope_position_delta=mrope_position_delta,
        )
```
**EN:** This block defines function `QwenVLImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings. This subsection covers lines 487-497 of the same logical block.
**CN:** 该代码块定义函数 `QwenVLImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。 本小节覆盖同一逻辑块中的第 487-497 行。

### Lines 498-498: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the QwenVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 QwenVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 499-573: Defines async function QwenVLImageProcessor.process_mm_data_async (part 1)
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        entry_time = time.perf_counter()
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            audio_data=request_obj.audio_data,
            multimodal_tokens=self.mm_tokens,
        )
        load_time = time.perf_counter()
        rid = getattr(request_obj, "rid", "anonymous_rid")

        video_metadata = None
        if base_output.videos:
            videos_processed = [
                await preprocess_video(video, video_config=self.video_config)
                for video in base_output.videos
            ]
            base_output.videos, video_metadata = map(list, zip(*videos_processed))

        preprocess_time = time.perf_counter()

        # NOTE: for qwen3-vl, video_meta need to be passed in, since do_sample_frames is already done in preprocess_video
        if self.hf_config.model_type in (
            "qwen3_vl",
            "qwen3_vl_moe",
            "qwen3_5",
            "qwen3_5_moe",
            "intern_s2_preview",
        ):
            mm_items, input_ids, ret = self.process_and_combine_mm_data(
                base_output,
                self.mm_tokens,
                video_metadata=video_metadata,
                do_sample_frames=False,
            )
        else:
            mm_items, input_ids, ret = self.process_and_combine_mm_data(
                base_output, self.mm_tokens
            )

        audio_feature_lengths = None

        if self.model_type == "qwen3_omni_moe":
            audio_item = next((mm for mm in mm_items if mm.is_audio()), None)
            if audio_item:
                audio_feature_lengths = torch.sum(
                    audio_item.feature_attention_mask, dim=1
                )

        second_per_grid_ts = getattr(ret, "second_per_grid_ts", None)
        if second_per_grid_ts is None:
            second_per_grid_ts = getattr(ret, "video_second_per_grid", None)

        process_time = time.perf_counter()

        input_ids = input_ids.flatten()

        image_grid_thw = None
        if hasattr(ret, "image_grid_thw"):
            image_grid_thw = ret.image_grid_thw

        if image_grid_thw is None and image_data and isinstance(image_data[0], dict):
            image_grid_thw = image_data[0].get("image_grid_thw")

        video_grid_thw = None
        if hasattr(ret, "video_grid_thw"):
            video_grid_thw = ret.video_grid_thw
```
**EN:** This block defines async function `QwenVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 499-573 of the same logical block.
**CN:** 该代码块定义异步函数 `QwenVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 499-573 行。

### Lines 574-574: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 575-611: Defines async function QwenVLImageProcessor.process_mm_data_async (part 2)
```python
        if video_grid_thw is None and request_obj.video_data:
            first_video = request_obj.video_data[0]
            if isinstance(first_video, dict):
                video_grid_thw = first_video.get("video_grid_thw")

        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index(
            spatial_merge_size=self.hf_config.vision_config.spatial_merge_size,
            image_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            vision_start_token_id=self.vision_start_token_id,
            model_type=self.model_type,
            tokens_per_second=getattr(
                self.hf_config.vision_config, "tokens_per_second", None
            ),
            # use the expanded token ids
            input_ids=input_ids.unsqueeze(0),
            image_grid_thw=getattr(ret, "image_grid_thw", None),
            video_grid_thw=getattr(ret, "video_grid_thw", None),
            second_per_grid_ts=second_per_grid_ts,
            use_audio_in_video=False,
            audio_seqlens=audio_feature_lengths,
            audio_token_id=getattr(self.hf_config, "audio_token_id", None),
            audio_start_token_id=self.audio_start_token_id,
            position_id_per_seconds=getattr(
                self.hf_config, "position_id_per_seconds", None
            ),
        )
        mrope_positions = mrope_positions.squeeze(1)
        get_rope_index_time = time.perf_counter()
        logger.debug(
            f"[QwenVLProcessor Perf] {rid=}, "
            f"load_time: {(load_time - entry_time) * 1000:.2f} ms, "
            f"preprocess_time: {(preprocess_time - load_time) * 1000:.2f} ms, "
            f"process_time: {(process_time - preprocess_time) * 1000:.2f} ms, "
            f"get_rope_index_time: {(get_rope_index_time - process_time) * 1000:.2f} ms, "
            f"total_time: {(get_rope_index_time - entry_time) * 1000:.2f} ms"
        )
```
**EN:** This block defines async function `QwenVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 575-611 of the same logical block.
**CN:** 该代码块定义异步函数 `QwenVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 575-611 行。

### Lines 612-612: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 613-623: Defines async function QwenVLImageProcessor.process_mm_data_async (part 3)
```python
        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_start_id=self.vision_start_token_id,
            im_end_id=self.vision_end_token_id,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
            mrope_positions=mrope_positions,
            mrope_position_delta=mrope_position_delta,
        )
```
**EN:** This block defines async function `QwenVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 613-623 of the same logical block.
**CN:** 该代码块定义异步函数 `QwenVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 613-623 行。

## Key Concepts / 关键概念
- **Classes / 类**: `QwenVLImageProcessor`
- **Functions / 函数**: `smart_resize`, `round_by_factor`, `ceil_by_factor`, `floor_by_factor`, `smart_nframes`, `preprocess_video`
- **Constants / 常量**: `IMAGE_FACTOR`, `MIN_PIXELS`, `MAX_PIXELS`, `MAX_RATIO`, `RESIZE_RESAMPLE`, `VIDEO_TOTAL_PIXELS`, `VIDEO_MIN_PIXELS`, `VIDEO_MAX_PIXELS`, `FRAME_FACTOR`, `FPS`, `FPS_MIN_FRAMES`, `FPS_MAX_FRAMES`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`, `os`, `re`, `time`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`, `torchvision`, `torchvision.transforms`
- **Local Modules / 本地模块**: `sglang.srt.environ`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.interns2preview`, `sglang.srt.models.qwen2_5_vl`, `sglang.srt.models.qwen2_vl`, `sglang.srt.models.qwen3_5`, `sglang.srt.models.qwen3_omni_moe`, `sglang.srt.models.qwen3_vl`, `sglang.srt.models.qwen3_vl_moe`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils.video_decoder`, `sglang.utils`
