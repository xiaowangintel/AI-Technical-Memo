# ernie45_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/ernie45_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for ernie45 vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 ernie45 vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22: Imports dependencies
```python
import math
import os
from typing import List, Union

import numpy as np
import torch
import torchvision
from PIL import Image
from torchvision.transforms import InterpolationMode
from transformers import BaseImageProcessor

from sglang.srt.environ import envs
from sglang.srt.layers.rotary_embedding import MRotaryEmbedding
from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.ernie45_vl import Ernie4_5_VLMoeForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
from sglang.srt.utils import get_bool_env_var, is_npu, logger
```
**EN:** This block groups related imports for the module, including math, os, typing.List, typing.Union, numpy, and 14 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, os, typing.List, typing.Union, numpy 等 14 项，为后续代码准备所需名称。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-24: Declares _is_npu
```python
_is_npu = is_npu()
```
**EN:** This statement initializes _is_npu in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 _is_npu。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-26: Declares SGL_USE_CUDA_IPC
```python
SGL_USE_CUDA_IPC = get_bool_env_var("SGLANG_USE_CUDA_IPC_TRANSPORT")
```
**EN:** This statement initializes SGL_USE_CUDA_IPC in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 SGL_USE_CUDA_IPC。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 27-28: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 29-30: Declares IMAGE_FACTOR, MIN_PIXELS
```python
IMAGE_FACTOR = 28
MIN_PIXELS = 4 * 28 * 28
```
**EN:** This block initializes a related set of values in the module, including IMAGE_FACTOR, MIN_PIXELS. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 IMAGE_FACTOR, MIN_PIXELS。将这些赋值集中在一起有助于理解周边配置。

### Lines 31-31: Comments and module notes
```python
# MAX_PIXELS = envs.SGLANG_IMAGE_MAX_PIXELS.get()
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 32-34: Declares MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE
```python
MAX_PIXELS = 16384 * 28 * 28
MAX_RATIO = 200
RESIZE_RESAMPLE = getattr(Image, envs.SGLANG_RESIZE_RESAMPLE.get(), None)
```
**EN:** This block initializes a related set of values in the module, including MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 MAX_PIXELS, MAX_RATIO, RESIZE_RESAMPLE。将这些赋值集中在一起有助于理解周边配置。

### Lines 35-39: Evaluates conditional branch a runtime condition
```python
if envs.SGLANG_RESIZE_RESAMPLE.is_set() and RESIZE_RESAMPLE is None:
    logger.warning(
        f"Invalid RESIZE_RESAMPLE value: '{envs.SGLANG_RESIZE_RESAMPLE.get()}'. "
        f"Ignoring and using default."
    )
```
**EN:** This block executes conditionally based on `a runtime condition` and selects behavior for the surrounding module.
**CN:** 该代码块根据 `a runtime condition` 的结果进行条件分支，为周围的 模块 选择不同的行为。

### Lines 40-42: Declares VIDEO_TOTAL_PIXELS
```python
VIDEO_TOTAL_PIXELS = int(
    float(os.environ.get("VIDEO_MAX_PIXELS", 128000 * 28 * 28 * 0.9))
)
```
**EN:** This statement initializes VIDEO_TOTAL_PIXELS in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 VIDEO_TOTAL_PIXELS。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 43-43: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 44-49: Declares VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES, and 1 more
```python
VIDEO_MIN_PIXELS = 299 * 28 * 28
VIDEO_MAX_PIXELS = 1196 * 28 * 28
FRAME_FACTOR = 2
FPS = 2.0
FPS_MIN_FRAMES = 16
FPS_MAX_FRAMES = 180
```
**EN:** This block initializes a related set of values in the module, including VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 VIDEO_MIN_PIXELS, VIDEO_MAX_PIXELS, FRAME_FACTOR, FPS, FPS_MIN_FRAMES 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 50-51: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 52-84: Defines function smart_resize
```python
def smart_resize(
    height: int,
    width: int,
    factor: int = IMAGE_FACTOR,
    min_pixels: int = MIN_PIXELS,
    max_pixels: int = MAX_PIXELS,
):
    if max(height, width) / min(height, width) > MAX_RATIO:
        if height > width:
            new_width = max(factor, round_by_factor(width, factor))
            new_height = floor_by_factor(new_width * MAX_RATIO, factor)
        else:
            new_height = max(factor, round_by_factor(height, factor))
            new_width = floor_by_factor(new_height * MAX_RATIO, factor)

        height = new_height
        width = new_width

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

    if min_pixels > h_bar * w_bar or h_bar * w_bar > max_pixels:
        raise ValueError(f"encounter invalid h_bar: {h_bar}, w_bar: {w_bar}")

    return h_bar, w_bar
```
**EN:** This block defines function `smart_resize`. Parameters: height, width, factor, min_pixels, max_pixels.
**CN:** 该代码块定义函数 `smart_resize`。 参数包括 height、width、factor、min_pixels、max_pixels。

### Lines 85-86: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 87-104: Defines function resize_image
```python
def resize_image(
    image,
    min_pixels: int = MIN_PIXELS,
    max_pixels: int = MAX_PIXELS,
    size_factor: int = IMAGE_FACTOR,
) -> Image.Image:
    width, height = image.size
    min_pixels = min_pixels
    max_pixels = max_pixels
    resized_height, resized_width = smart_resize(
        height,
        width,
        factor=size_factor,
        min_pixels=min_pixels,
        max_pixels=max_pixels,
    )
    image = image.resize((resized_width, resized_height), resample=RESIZE_RESAMPLE)
    return image
```
**EN:** This block defines function `resize_image`. Parameters: image, min_pixels, max_pixels, size_factor.
**CN:** 该代码块定义函数 `resize_image`。 参数包括 image、min_pixels、max_pixels、size_factor。

### Lines 105-106: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 107-108: Defines function round_by_factor
```python
def round_by_factor(number: int | float, factor: int) -> int:
    return round(number / factor) * factor
```
**EN:** This block defines function `round_by_factor`. Parameters: number, factor.
**CN:** 该代码块定义函数 `round_by_factor`。 参数包括 number、factor。

### Lines 109-110: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 111-112: Defines function ceil_by_factor
```python
def ceil_by_factor(number: int | float, factor: int) -> int:
    return math.ceil(number / factor) * factor
```
**EN:** This block defines function `ceil_by_factor`. Parameters: number, factor.
**CN:** 该代码块定义函数 `ceil_by_factor`。 参数包括 number、factor。

### Lines 113-114: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 115-116: Defines function floor_by_factor
```python
def floor_by_factor(number: int | float, factor: int) -> int:
    return math.floor(number / factor) * factor
```
**EN:** This block defines function `floor_by_factor`. Parameters: number, factor.
**CN:** 该代码块定义函数 `floor_by_factor`。 参数包括 number、factor。

### Lines 117-118: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 119-125: Defines async function resize_image_async
```python
async def resize_image_async(
    image,
    min_pixels: int = MIN_PIXELS,
    max_pixels: int = MAX_PIXELS,
    size_factor: int = IMAGE_FACTOR,
):
    return resize_image(image, min_pixels, max_pixels, size_factor)
```
**EN:** This block defines async function `resize_image_async`. Parameters: image, min_pixels, max_pixels, size_factor.
**CN:** 该代码块定义异步函数 `resize_image_async`。 参数包括 image、min_pixels、max_pixels、size_factor。

### Lines 126-127: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 128-173: Defines function smart_nframes
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

### Lines 174-176: Comments and module notes
```python


# process video, qwen-specific
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 177-220: Defines async function preprocess_video
```python
async def preprocess_video(
    vr,
    image_factor: int = IMAGE_FACTOR,
) -> torch.Tensor:

    total_frames, video_fps = len(vr), vr.get_avg_fps()
    nframes = smart_nframes({}, total_frames=total_frames, video_fps=video_fps)
    idx = np.linspace(0, total_frames - 1, num=nframes, dtype=np.int64)
    idx = np.unique(idx)
    video_np = vr.get_batch(idx).asnumpy()
    video = torch.from_numpy(video_np).pin_memory()
    video = video.permute(0, 3, 1, 2)  # Convert to TCHW format
    nframes, _, height, width = video.shape
    min_pixels = VIDEO_MIN_PIXELS
    total_pixels = VIDEO_TOTAL_PIXELS
    max_pixels = max(
        min(VIDEO_MAX_PIXELS, total_pixels / nframes * FRAME_FACTOR),
        int(min_pixels * 1.05),
    )

    resized_height, resized_width = smart_resize(
        height,
        width,
        factor=image_factor,
        min_pixels=min_pixels,
        max_pixels=max_pixels,
    )
    video = torchvision.transforms.functional.resize(
        video,
        [resized_height, resized_width],
        interpolation=InterpolationMode.BILINEAR,
    )

    video = video.permute(0, 2, 3, 1)
    video = video.pin_memory()
    video_metadata = {
        "fps": video_fps,
        "duration": total_frames / video_fps,
        "total_num_frames": total_frames,
        "frames_indices": idx,
        "video_backend": "torchvision",
    }

    return video, video_metadata
```
**EN:** This block defines async function `preprocess_video`. Parameters: vr, image_factor.
**CN:** 该代码块定义异步函数 `preprocess_video`。 参数包括 vr、image_factor。

### Lines 221-223: Comments and module notes
```python


# Compatible with Ernie-VL Series
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 224-224: Declares class Ernie4_5_VLImageProcessor
```python
class Ernie4_5_VLImageProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `Ernie4_5_VLImageProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor.
**CN:** 该代码块声明类 `Ernie4_5_VLImageProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。

### Lines 225-225: Declares models
```python
    models = [Ernie4_5_VLMoeForConditionalGeneration]
```
**EN:** This statement initializes models in the Ernie4_5_VLImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Ernie4_5_VLImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 226-226: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Ernie4_5_VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Ernie4_5_VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 227-248: Defines function Ernie4_5_VLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.hf_config = hf_config
        self.model_type = hf_config.model_type
        self.image_start_token_id = hf_config.image_start_token_id
        self.image_end_token_id = hf_config.image_end_token_id
        self.video_start_token_id = hf_config.video_start_token_id
        self.video_end_token_id = hf_config.video_end_token_id

        self.IMAGE_FACTOR = 28
        self.MIN_PIXELS = 4 * 28 * 28
        self.MAX_PIXELS = 16384 * 28 * 28
        self.MAX_RATIO = 200
        self.mm_tokens = MultimodalSpecialTokens(
            image_token="<|IMAGE_START|><|image@placeholder|><|IMAGE_END|>",
            video_token="<|VIDEO_START|><|video@placeholder|><|VIDEO_END|>",
            image_token_id=hf_config.im_patch_id,
            video_token_id=hf_config.im_patch_id,  # image and video use the same token_id
        ).build(_processor)

        self.tokenizer = self._processor.tokenizer
        self.image_processor = self._processor.image_processor
```
**EN:** This block defines function `Ernie4_5_VLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Ernie4_5_VLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 249-249: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Ernie4_5_VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Ernie4_5_VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 250-285: Defines function Ernie4_5_VLImageProcessor._pixel_values_norm
```python
    def _pixel_values_norm(
        self,
        pixel_values: torch.Tensor,
        mm_kwargs: object,
    ) -> torch.Tensor:
        hf_config = self.hf_config
        vision_config = hf_config.vision_config
        image_processor = self.image_processor
        image_mean_tensor = torch.tensor(
            image_processor.image_mean, dtype=torch.float32
        ).reshape([1, 3, 1, 1])
        image_std_tensor = torch.tensor(
            image_processor.image_std, dtype=torch.float32
        ).reshape([1, 3, 1, 1])
        rescale_factor = torch.tensor(
            image_processor.rescale_factor, dtype=torch.float32
        )
        patch_size_squared = vision_config.patch_size**2

        image_mean_tensor = image_mean_tensor.squeeze([-2, -1]).repeat_interleave(
            patch_size_squared, -1
        )
        image_std_tensor = image_std_tensor.squeeze([-2, -1]).repeat_interleave(
            patch_size_squared, -1
        )

        if not image_mean_tensor.is_contiguous():
            image_mean_tensor = image_mean_tensor.contiguous()
        if not image_std_tensor.is_contiguous():
            image_std_tensor = image_std_tensor.contiguous()

        pixel_values = (
            rescale_factor * pixel_values.to(torch.float32) - image_mean_tensor
        ) / image_std_tensor
        pixel_values = pixel_values.to(hf_config.dtype)
        return pixel_values
```
**EN:** This block defines function `Ernie4_5_VLImageProcessor._pixel_values_norm`. Parameters: self, pixel_values, mm_kwargs.
**CN:** 该代码块定义函数 `Ernie4_5_VLImageProcessor._pixel_values_norm`。 参数包括 self、pixel_values、mm_kwargs。

### Lines 286-286: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Ernie4_5_VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Ernie4_5_VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 287-363: Defines function Ernie4_5_VLImageProcessor.process_mm_data
```python
    def process_mm_data(
        self, input_text, images=None, videos=None, audios=None, **kwargs
    ) -> dict:
        """
        process multimodal data with transformers AutoProcessor
        """
        if images:
            kwargs["images"] = images
            if self.image_config:
                kwargs.setdefault("images_kwargs", {}).update(self.image_config)
        if videos:
            kwargs["videos"] = videos
            if self.video_config:
                kwargs.setdefault("videos_kwargs", {}).update(self.video_config)

        processor = self._processor
        if (
            hasattr(processor, "image_processor")
            and isinstance(processor.image_processor, BaseImageProcessor)
            and not self.server_args.disable_fast_image_processor
        ):
            if not _is_npu:
                kwargs["device"] = "cuda"

        result = processor.__call__(
            text=[input_text],
            padding=True,
            return_tensors="pt",
            **kwargs,
        )

        # Divide the processor_output into two modalities: image and video.
        if result is not None:
            pixel_values = result["images"]
            if pixel_values is not None:
                result["images"] = self._pixel_values_norm(pixel_values, kwargs)
            for key in list(result.keys()):
                if result[key] is None:
                    del result[key]
                    continue
                if key == "grid_thw":
                    grid_thw = result["grid_thw"]
                    pixel_values_all = result["images"]
                    # Identify elements where the first
                    # dimension is greater than 1 and
                    # treat them as the video modality
                    mask = grid_thw[:, 0] > 1
                    result["video_grid_thw"] = grid_thw[mask]
                    result["image_grid_thw"] = grid_thw[~mask]
                    image_patch_num = result["image_grid_thw"].prod(dim=1).sum()
                    result["pixel_values"] = pixel_values_all[:image_patch_num]
                    result["pixel_values_videos"] = pixel_values_all[image_patch_num:]
                    del result["images"]
                    del result["grid_thw"]

                    # del empty result
                    if result["image_grid_thw"].numel() == 0:
                        del result["image_grid_thw"]
                    if result["pixel_values"].numel() == 0:
                        del result["pixel_values"]
                    if result["video_grid_thw"].numel() == 0:
                        del result["video_grid_thw"]
                    if result["pixel_values_videos"].numel() == 0:
                        del result["pixel_values_videos"]

        if not self.server_args.keep_mm_feature_on_device:
            # move feature tensors to cpu
            for feature_name in self.FEATURE_NAMES:
                if SGL_USE_CUDA_IPC:
                    pass
                else:
                    if feature_name in result and isinstance(
                        result[feature_name], torch.Tensor
                    ):
                        result[feature_name] = result[feature_name].to("cpu")

        return result
```
**EN:** This block defines function `Ernie4_5_VLImageProcessor.process_mm_data`. Parameters: self, input_text, images, videos, audios. process multimodal data with transformers AutoProcessor
**CN:** 该代码块定义函数 `Ernie4_5_VLImageProcessor.process_mm_data`。 参数包括 self、input_text、images、videos、audios。 文档字符串摘要：process multimodal data with transformers AutoProcessor

### Lines 364-364: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Ernie4_5_VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Ernie4_5_VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 365-381: Defines function Ernie4_5_VLImageProcessor.compute_mrope_positions
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
        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index_ernie45(
            input_ids=input_ids_tensor,
            hf_config=self.hf_config,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
        )
        return mrope_positions.squeeze(1), mrope_position_delta
```
**EN:** This block defines function `Ernie4_5_VLImageProcessor.compute_mrope_positions`. Parameters: self, input_ids, mm_items.
**CN:** 该代码块定义函数 `Ernie4_5_VLImageProcessor.compute_mrope_positions`。 参数包括 self、input_ids、mm_items。

### Lines 382-382: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Ernie4_5_VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Ernie4_5_VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 383-440: Defines async function Ernie4_5_VLImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            audio_data=request_obj.audio_data,
            multimodal_tokens=self.mm_tokens,
        )

        # resize images if they are raw Image objects
        resized_images = []
        if base_output.images and isinstance(base_output.images[0], Image.Image):
            for image in base_output.images:
                resized_image = resize_image(image)
                resized_images.append(resized_image)
            base_output.images = resized_images

        if base_output.videos:
            videos_processed = [
                await preprocess_video(video) for video in base_output.videos
            ]
            base_output.videos, _ = map(list, zip(*videos_processed))

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        input_ids = input_ids.flatten()

        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index_ernie45(
            input_ids=input_ids.unsqueeze(0),
            hf_config=self.hf_config,
            image_grid_thw=getattr(ret, "image_grid_thw", None),
            video_grid_thw=getattr(ret, "video_grid_thw", None),
        )
        mrope_positions = mrope_positions.squeeze(1)

        assert (
            input_ids.shape[0] == mrope_positions.shape[-1]
        ), "input_ids and mrope_positions should have the same length"

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_start_id=self.image_start_token_id,
            im_end_id=self.image_end_token_id,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            mrope_positions=mrope_positions,
            mrope_position_delta=mrope_position_delta,
        )
```
**EN:** This block defines async function `Ernie4_5_VLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Ernie4_5_VLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `Ernie4_5_VLImageProcessor`
- **Functions / 函数**: `smart_resize`, `resize_image`, `round_by_factor`, `ceil_by_factor`, `floor_by_factor`, `resize_image_async`, `smart_nframes`, `preprocess_video`
- **Constants / 常量**: `SGL_USE_CUDA_IPC`, `IMAGE_FACTOR`, `MIN_PIXELS`, `MAX_PIXELS`, `MAX_RATIO`, `RESIZE_RESAMPLE`, `VIDEO_TOTAL_PIXELS`, `VIDEO_MIN_PIXELS`, `VIDEO_MAX_PIXELS`, `FRAME_FACTOR`, `FPS`, `FPS_MIN_FRAMES`, `FPS_MAX_FRAMES`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`, `os`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`, `torchvision`, `torchvision.transforms`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.environ`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.ernie45_vl`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils`
