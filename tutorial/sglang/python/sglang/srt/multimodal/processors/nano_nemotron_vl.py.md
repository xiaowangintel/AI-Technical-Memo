# nano_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/nano_nemotron_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for nano nemotron vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 nano nemotron vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Comments and module notes
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 14-49: Imports dependencies
```python
import logging
import math
from math import sqrt

import numpy as np
import torch
from PIL import Image

from sglang.srt.configs.nano_nemotron_vl import (
    NemotronH_Nano_Omni_Reasoning_V3_Config,
    NemotronH_Nano_VL_V2_Config,
)
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.nano_nemotron_vl import (
    NemotronH_Nano_Omni_Reasoning_V3,
    NemotronH_Nano_VL_V2,
)
from sglang.srt.models.parakeet import ParakeetExtractor
from sglang.srt.multimodal.audio_from_video import extract_audio_from_video_bytes
from sglang.srt.multimodal.evs import EVSProcessor
from sglang.srt.multimodal.internvl_utils import (
    compute_budgeted_image_sizes,
    get_video_target_size_and_feature_size,
    image_to_pixel_values,
    resize_image_to_pixels,
    video_to_pixel_values,
)
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
from sglang.srt.utils.common import sample_video_frames
```
**EN:** This block groups related imports for the module, including logging, math, math.sqrt, numpy, torch, and 19 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, math, math.sqrt, numpy, torch 等 19 项，为后续代码准备所需名称。

### Lines 50-50: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 51-51: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-56: Declares DEFAULT_NUM_TILES, NUM_VIDEO_TILES, DESIRED_FPS, MAX_FRAMES
```python
DEFAULT_NUM_TILES = 12
NUM_VIDEO_TILES = 1
DESIRED_FPS = 2  # TODO: allow desired fps/num frames to be configurable
MAX_FRAMES = 128
```
**EN:** This block initializes a related set of values in the module, including DEFAULT_NUM_TILES, NUM_VIDEO_TILES, DESIRED_FPS, MAX_FRAMES. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 DEFAULT_NUM_TILES, NUM_VIDEO_TILES, DESIRED_FPS, MAX_FRAMES。将这些赋值集中在一起有助于理解周边配置。

### Lines 57-58: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 59-59: Declares class NanoNemotronVLImageProcessor
```python
class NanoNemotronVLImageProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `NanoNemotronVLImageProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `NanoNemotronVLImageProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 60-63: Declares models, gpu_image_decode
```python
    models = [NemotronH_Nano_VL_V2, NemotronH_Nano_Omni_Reasoning_V3]
    gpu_image_decode = (
        False  # NanoNemotronVL processes loaded image as PIL image explicitly
    )
```
**EN:** This block initializes a related set of values in the NanoNemotronVLImageProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 NanoNemotronVLImageProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 64-64: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 65-133: Defines function NanoNemotronVLImageProcessor.__init__ (part 1)
```python
    def __init__(self, hf_config, server_args, _image_processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _image_processor, *args, **kwargs)
        self.evs = EVSProcessor(
            hf_config,
            {
                NemotronH_Nano_VL_V2_Config: NemotronH_Nano_VL_V2,
                NemotronH_Nano_Omni_Reasoning_V3_Config: NemotronH_Nano_Omni_Reasoning_V3,
            },
        )
        Image.MAX_IMAGE_PIXELS = None
        self.image_size = hf_config.image_size
        self.VIDEO_CONTEXT_TOKEN = hf_config.video_context_token
        self.IMG_CONTEXT_TOKEN = hf_config.img_context_token
        self.IMG_START_TOKEN = hf_config.img_start_token
        self.IMG_END_TOKEN = hf_config.img_end_token
        self.num_image_token = int(
            (self.image_size // hf_config.patch_size) ** 2
            * (hf_config.downsample_ratio**2)
        )
        if hasattr(self._processor, "tokenizer"):
            tokenizer = self._processor.tokenizer
        else:
            tokenizer = self._processor
        self.tokenizer = tokenizer

        self.img_start_token_id = tokenizer.convert_tokens_to_ids(self.IMG_START_TOKEN)
        self.img_end_token_id = tokenizer.convert_tokens_to_ids(self.IMG_END_TOKEN)

        # Audio support: initialize Parakeet extractor if sound_config is present
        self.audio_extractor: ParakeetExtractor | None = None
        self.AUDIO_CONTEXT_TOKEN = getattr(
            hf_config, "audio_context_token", "<so_embedding>"
        )
        self.AUDIO_START_TOKEN = getattr(hf_config, "audio_start_token", "<so_start>")
        self.AUDIO_END_TOKEN = getattr(hf_config, "audio_end_token", "<so_end>")

        audio_token_str = None
        audio_token_id = None
        if getattr(hf_config, "sound_config", None) is not None:
            self.audio_extractor = ParakeetExtractor(hf_config.sound_config)
            audio_token_str = self.AUDIO_CONTEXT_TOKEN
            audio_token_id = tokenizer.convert_tokens_to_ids(self.AUDIO_CONTEXT_TOKEN)
            self.audio_start_token_id = tokenizer.convert_tokens_to_ids(
                self.AUDIO_START_TOKEN
            )
            self.audio_end_token_id = tokenizer.convert_tokens_to_ids(
                self.AUDIO_END_TOKEN
            )

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMG_CONTEXT_TOKEN,
            image_token_id=tokenizer.convert_tokens_to_ids(self.IMG_CONTEXT_TOKEN),
            video_token=self.VIDEO_CONTEXT_TOKEN,
            video_token_id=tokenizer.convert_tokens_to_ids(self.VIDEO_CONTEXT_TOKEN),
            audio_token=audio_token_str,
            audio_token_id=audio_token_id,
        ).build(_image_processor)

        # Normalization config (mean/std) and tiling behavior
        self.norm_mean = hf_config.norm_mean
        self.norm_std = hf_config.norm_std
        self.use_thumbnail = hf_config.use_thumbnail

        # Dynamic resolution config
        self.dynamic_resolution = getattr(hf_config, "dynamic_resolution", False)
        self.min_num_patches = getattr(hf_config, "min_num_patches", 0)
        self.max_num_patches = getattr(hf_config, "max_num_patches", 0)
        self.patch_size = hf_config.patch_size
        self.downsample_ratio = hf_config.downsample_ratio
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _image_processor. This subsection covers lines 65-133 of the same logical block.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_image_processor。 本小节覆盖同一逻辑块中的第 65-133 行。

### Lines 134-134: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 135-146: Defines function NanoNemotronVLImageProcessor.__init__ (part 2)
```python
        # Video temporal compression config
        self.video_temporal_patch_size = getattr(
            hf_config, "video_temporal_patch_size", 1
        )
        self.video_target_num_patches = getattr(
            hf_config, "video_target_num_patches", 0
        )
        self.video_maintain_aspect_ratio = getattr(
            hf_config, "video_maintain_aspect_ratio", True
        )

        self.max_model_len = getattr(server_args, "context_length", None) or 8192
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _image_processor. This subsection covers lines 135-146 of the same logical block.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_image_processor。 本小节覆盖同一逻辑块中的第 135-146 行。

### Lines 147-147: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 148-151: Defines function NanoNemotronVLImageProcessor.__init__ (part 3)
```python
        self.PLACEHOLDER = self.tokenizer.unk_token
        assert isinstance(self.PLACEHOLDER, str)
        self.PLACEHOLDER_ID = tokenizer.convert_tokens_to_ids(self.PLACEHOLDER)
        assert isinstance(self.PLACEHOLDER_ID, int)
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _image_processor. This subsection covers lines 148-151 of the same logical block.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_image_processor。 本小节覆盖同一逻辑块中的第 148-151 行。

### Lines 152-152: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 153-163: Defines function NanoNemotronVLImageProcessor.preprocess_image
```python
    def preprocess_image(
        self, image: Image.Image, *, max_num_tiles: int = DEFAULT_NUM_TILES
    ) -> torch.Tensor:
        return image_to_pixel_values(
            image,
            input_size=self.image_size,
            max_num_tiles=max_num_tiles,
            use_thumbnail=self.use_thumbnail,
            mean=self.norm_mean,
            std=self.norm_std,
        ).to(dtype=torch.bfloat16)
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.preprocess_image`. Parameters: self, image.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.preprocess_image`。 参数包括 self、image。

### Lines 164-164: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 165-166: Defines function NanoNemotronVLImageProcessor.render_image
```python
    def render_image(self, *, num_tiles: int):
        return f"{self.IMG_START_TOKEN}{self.IMG_CONTEXT_TOKEN * self.num_image_token * num_tiles}{self.IMG_END_TOKEN}"
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.render_image`. Parameters: self.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.render_image`。 参数包括 self。

### Lines 167-167: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 168-169: Defines function NanoNemotronVLImageProcessor.render_image_dynamic
```python
    def render_image_dynamic(self, *, num_tokens: int):
        return f"{self.IMG_START_TOKEN}{self.IMG_CONTEXT_TOKEN * num_tokens}{self.IMG_END_TOKEN}"
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.render_image_dynamic`. Parameters: self.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.render_image_dynamic`。 参数包括 self。

### Lines 170-170: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 171-187: Defines function NanoNemotronVLImageProcessor.render_tubelet
```python
    def render_tubelet(
        self,
        tubelet_index: int,
        frame_indices: list[int],
        timestamps: list[float],
        num_tokens: int,
    ):
        """Render a tubelet (group of T frames) for temporal compression."""
        if len(frame_indices) == 1:
            return self.render_frame(
                frame_indices[0], timestamp=timestamps[0], num_tokens=num_tokens
            )
        parts = " and ".join(
            f"frame {fi + 1} sampled at {ts:.2f} seconds"
            for fi, ts in zip(frame_indices, timestamps)
        )
        return f"{parts}: {self.PLACEHOLDER}{self.IMG_CONTEXT_TOKEN * num_tokens}{self.IMG_END_TOKEN}"
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.render_tubelet`. Parameters: self, tubelet_index, frame_indices, timestamps, num_tokens. Render a tubelet (group of T frames) for temporal compression.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.render_tubelet`。 参数包括 self、tubelet_index、frame_indices、timestamps、num_tokens。 文档字符串摘要：Render a tubelet (group of T frames) for temporal compression.

### Lines 188-188: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 189-190: Defines function NanoNemotronVLImageProcessor.render_frame
```python
    def render_frame(self, frame_index: int, *, timestamp: float, num_tokens: int):
        return f"Frame {frame_index + 1} sampled at {timestamp:.2f} seconds: {self.PLACEHOLDER}{self.IMG_CONTEXT_TOKEN * num_tokens}{self.IMG_END_TOKEN}"
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.render_frame`. Parameters: self, frame_index.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.render_frame`。 参数包括 self、frame_index。

### Lines 191-191: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 192-204: Defines function NanoNemotronVLImageProcessor.parse_video
```python
    @staticmethod
    def parse_video(video) -> tuple[np.ndarray, list[float]]:
        frames = sample_video_frames(
            video, desired_fps=DESIRED_FPS, max_frames=MAX_FRAMES
        )
        video_array = video.get_frames_at(frames)
        avg_fps = video.avg_fps
        if avg_fps > 0:
            frame_duration_ms = int(1000 / avg_fps)
        else:
            frame_duration_ms = 0
        timestamps = [i * frame_duration_ms / 1000.0 for i in frames]
        return video_array, timestamps
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.parse_video`. Parameters: video. Decorators: staticmethod.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.parse_video`。 参数包括 video。 装饰器包括 staticmethod。

### Lines 205-205: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 206-211: Defines function NanoNemotronVLImageProcessor.render_audio
```python
    def render_audio(self, *, num_tokens: int):
        return (
            f"{self.AUDIO_START_TOKEN}"
            f"{self.AUDIO_CONTEXT_TOKEN * num_tokens}"
            f"{self.AUDIO_END_TOKEN}"
        )
```
**EN:** This block defines function `NanoNemotronVLImageProcessor.render_audio`. Parameters: self.
**CN:** 该代码块定义函数 `NanoNemotronVLImageProcessor.render_audio`。 参数包括 self。

### Lines 212-212: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NanoNemotronVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NanoNemotronVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 213-265: Defines async function NanoNemotronVLImageProcessor.process_mm_data_async (part 1)
```python
    async def process_mm_data_async(
        self, image_data, audio_data, input_text, request_obj, **kwargs
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            audio_data=audio_data if self.audio_extractor else None,
            multimodal_tokens=self.mm_tokens,
            discard_alpha_channel=True,
            audio_sample_rate=(
                self.audio_extractor.sampling_rate if self.audio_extractor else None
            ),
        )

        videos = [self.parse_video(video) for video in base_output.videos]

        T = self.video_temporal_patch_size

        if T > 1:
            tubelets_per_video = [math.ceil(len(frames) / T) for frames, _ in videos]
            if self.video_target_num_patches > 0 and videos:
                frame_h, frame_w = videos[0][0][0].shape[:2]
                target_w, target_h, tokens_per_tubelet = (
                    get_video_target_size_and_feature_size(
                        frame_w,
                        frame_h,
                        self.video_target_num_patches,
                        self.video_maintain_aspect_ratio,
                        self.patch_size,
                        self.downsample_ratio,
                    )
                )
                ds = int(1 / self.downsample_ratio)
                rows = target_h // self.patch_size // ds
                cols = target_w // self.patch_size // ds
            else:
                tokens_per_tubelet = self.num_image_token
                rows = cols = int(sqrt(tokens_per_tubelet))
            create_data_items, tokens_per_frame = self.evs.static_size_data_items(
                frames_per_video=tubelets_per_video,
                num_images=len(base_output.images),
                rows=rows,
                cols=cols,
            )
        else:
            rows = cols = int(sqrt(self.num_image_token))
            create_data_items, tokens_per_frame = self.evs.static_size_data_items(
                frames_per_video=[len(frames) for frames, _ in videos],
                num_images=len(base_output.images),
                rows=rows,
                cols=cols,
            )
```
**EN:** This block defines async function `NanoNemotronVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 213-265 of the same logical block.
**CN:** 该代码块定义异步函数 `NanoNemotronVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 213-265 行。

### Lines 266-266: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 267-345: Defines async function NanoNemotronVLImageProcessor.process_mm_data_async (part 2)
```python
        prompt = input_text
        image_is_dynamic = False
        num_tokens_per_image = []
        image_feature = None
        if base_output.images and self.dynamic_resolution:
            image_is_dynamic = True
            image_sizes = [(img.width, img.height) for img in base_output.images]
            text_only = input_text.replace(self.IMG_CONTEXT_TOKEN, "")
            text_tokens = len(
                self.tokenizer(text_only, add_special_tokens=False)["input_ids"]
            )
            total_token_budget = self.max_model_len - text_tokens
            budgeted_sizes = compute_budgeted_image_sizes(
                image_sizes,
                total_token_budget,
                self.patch_size,
                self.downsample_ratio,
                self.min_num_patches,
                self.max_num_patches,
            )
            preprocessed_images = []
            for image, (target_w, target_h, n_tokens) in zip(
                base_output.images, budgeted_sizes
            ):
                pv = resize_image_to_pixels(
                    image,
                    target_w,
                    target_h,
                    mean=self.norm_mean,
                    std=self.norm_std,
                )
                preprocessed_images.append(pv.to(dtype=torch.bfloat16))
                num_tokens_per_image.append(n_tokens)
            rendered_images = [
                self.render_image_dynamic(num_tokens=nt) for nt in num_tokens_per_image
            ]
            prompt = prompt.replace(self.IMG_CONTEXT_TOKEN, "".join(rendered_images), 1)
            image_feature = preprocessed_images
        elif base_output.images:
            preprocessed_images = [
                self.preprocess_image(image) for image in base_output.images
            ]
            rendered_images = [
                self.render_image(num_tiles=image.shape[0])
                for image in preprocessed_images
            ]
            prompt = prompt.replace(self.IMG_CONTEXT_TOKEN, "".join(rendered_images), 1)
            image_feature = torch.cat(preprocessed_images, dim=0)

        video_feature = None
        T = self.video_temporal_patch_size
        if base_output.videos:
            preprocessed_videos = []
            for (video_array, timestamps), tpf in zip(
                videos, tokens_per_frame, strict=True
            ):
                if self.video_target_num_patches > 0:
                    frames_tensors = []
                    for frame in video_array:
                        pv, _ = video_to_pixel_values(
                            Image.fromarray(frame, mode="RGB"),
                            patch_size=self.patch_size,
                            downsample_ratio=self.downsample_ratio,
                            target_num_patches=self.video_target_num_patches,
                            maintain_aspect_ratio=self.video_maintain_aspect_ratio,
                            mean=self.norm_mean,
                            std=self.norm_std,
                        )
                        frames_tensors.append(pv.to(dtype=torch.bfloat16))
                else:
                    frames_tensors = [
                        self.preprocess_image(
                            Image.fromarray(frame, mode="RGB"),
                            max_num_tiles=NUM_VIDEO_TILES,
                        )
                        for frame in video_array
                    ]
                preprocessed_video = torch.cat(frames_tensors, dim=0)
                preprocessed_videos.append(preprocessed_video)
```
**EN:** This block defines async function `NanoNemotronVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 267-345 of the same logical block.
**CN:** 该代码块定义异步函数 `NanoNemotronVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 267-345 行。

### Lines 346-346: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 347-423: Defines async function NanoNemotronVLImageProcessor.process_mm_data_async (part 3)
```python
                if T > 1:
                    num_frames = len(video_array)
                    num_tubelets = math.ceil(num_frames / T)
                    rendered_parts = []
                    for ti in range(num_tubelets):
                        start_fi = ti * T
                        end_fi = min(start_fi + T, num_frames)
                        fi_list = list(range(start_fi, end_fi))
                        ts_list = [timestamps[fi] for fi in fi_list]
                        rendered_parts.append(
                            self.render_tubelet(
                                ti, fi_list, ts_list, num_tokens=tpf[ti]
                            )
                        )
                    prompt = prompt.replace(
                        self.VIDEO_CONTEXT_TOKEN, "\n".join(rendered_parts), 1
                    )
                else:
                    rendered_frames = [
                        self.render_frame(
                            i,
                            timestamp=timestamp,
                            num_tokens=num_tokens,
                        )
                        for i, (timestamp, num_tokens) in enumerate(
                            zip(timestamps, tpf, strict=True)
                        )
                    ]
                    prompt = prompt.replace(
                        self.VIDEO_CONTEXT_TOKEN, "".join(rendered_frames), 1
                    )
            video_feature = torch.cat(preprocessed_videos, dim=0)

        # Extract audio from video if requested and no explicit audio provided
        use_audio_in_video = getattr(request_obj, "use_audio_in_video", False)
        extracted_audios: list[np.ndarray] = []
        if (
            use_audio_in_video
            and base_output.videos
            and not base_output.audios
            and self.audio_extractor is not None
        ):
            for video_wrapper in base_output.videos:
                video_bytes = video_wrapper.source_bytes
                if video_bytes is not None:
                    audio_array = extract_audio_from_video_bytes(
                        video_bytes,
                        target_sr=self.audio_extractor.sampling_rate,
                    )
                    if audio_array is not None:
                        extracted_audios.append(audio_array)

        all_audios: list[np.ndarray] = (
            list(base_output.audios) if base_output.audios else []
        )
        all_audios.extend(extracted_audios)

        # Process audio data through the Parakeet feature extractor
        audio_items: list[MultimodalDataItem] = []
        if all_audios and self.audio_extractor is not None:
            extractor = self.audio_extractor
            for audio in all_audios:
                num_tokens = extractor.audio_token_count(len(audio))
                rendered = self.render_audio(num_tokens=num_tokens)
                if self.AUDIO_CONTEXT_TOKEN in prompt:
                    prompt = prompt.replace(self.AUDIO_CONTEXT_TOKEN, rendered, 1)
                else:
                    prompt = prompt + rendered

            extracted = extractor(
                all_audios,
                sampling_rate=extractor.sampling_rate,
                return_tensors="pt",
            )
            input_features = extracted.input_features
            attention_mask = extracted.attention_mask
            clip_counts = extracted.audio_num_clips
```
**EN:** This block defines async function `NanoNemotronVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 347-423 of the same logical block.
**CN:** 该代码块定义异步函数 `NanoNemotronVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 347-423 行。

### Lines 424-424: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 425-497: Defines async function NanoNemotronVLImageProcessor.process_mm_data_async (part 4)
```python
            clip_offset = 0
            for audio_idx, num_clips in enumerate(clip_counts):
                audio_features = input_features[clip_offset : clip_offset + num_clips]
                audio_mask = attention_mask[clip_offset : clip_offset + num_clips]
                clip_offset += num_clips
                audio_items.append(
                    MultimodalDataItem(
                        modality=Modality.AUDIO,
                        feature=audio_features,
                        model_specific_data={
                            "feature_attention_mask": audio_mask,
                            "audio_num_clips": num_clips,
                        },
                    )
                )

        prompt_ids = self.tokenizer(
            prompt, add_special_tokens=False, return_tensors="pt"
        )["input_ids"].flatten()
        offsets = self.get_mm_items_offset(prompt_ids, self.mm_tokens.image_token_id)
        img_offsets = [
            (start, end)
            for start, end in offsets
            if prompt_ids[start - 1] == self.img_start_token_id
        ]
        video_offsets = [
            (start, end)
            for start, end in offsets
            if prompt_ids[start - 1] == self.PLACEHOLDER_ID
        ]
        # Cleanup:
        prompt_ids[prompt_ids == self.PLACEHOLDER_ID] = self.img_start_token_id

        # Compute audio offsets
        if audio_items:
            audio_token_id = self.mm_tokens.audio_token_id
            audio_offsets_list = self.get_mm_items_offset(prompt_ids, audio_token_id)
            for item, offset in zip(audio_items, audio_offsets_list):
                item.offsets = [offset]

        prompt_ids_list = prompt_ids.tolist()

        if image_is_dynamic and image_feature is not None:
            items = []
            for i, (pv, offset) in enumerate(zip(image_feature, img_offsets)):
                items.append(
                    MultimodalDataItem(
                        modality=Modality.IMAGE,
                        feature=pv,
                        offsets=[offset],
                        model_specific_data={
                            "num_tokens": num_tokens_per_image[i],
                            "is_dynamic": True,
                        },
                    )
                )
            if video_feature is not None:
                items.append(
                    MultimodalDataItem(
                        modality=Modality.VIDEO,
                        feature=video_feature,
                        offsets=video_offsets,
                    )
                )
        else:
            items = create_data_items(
                image=image_feature,
                image_offsets=img_offsets,
                video=video_feature,
                video_offsets=video_offsets,
                input_ids_list=prompt_ids_list,
            )
        items.extend(audio_items)
```
**EN:** This block defines async function `NanoNemotronVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 425-497 of the same logical block.
**CN:** 该代码块定义异步函数 `NanoNemotronVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 425-497 行。

### Lines 498-498: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 499-509: Defines async function NanoNemotronVLImageProcessor.process_mm_data_async (part 5)
```python
        return MultimodalProcessorOutput(
            input_ids=prompt_ids_list,
            mm_items=items,
            im_start_id=self.img_start_token_id,
            im_end_id=self.img_end_token_id,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.image_token_id,
            audio_token_id=self.mm_tokens.audio_token_id if audio_items else None,
            audio_start_id=(self.audio_start_token_id if audio_items else None),
            audio_end_id=(self.audio_end_token_id if audio_items else None),
        )
```
**EN:** This block defines async function `NanoNemotronVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 499-509 of the same logical block.
**CN:** 该代码块定义异步函数 `NanoNemotronVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 499-509 行。

## Key Concepts / 关键概念
- **Classes / 类**: `NanoNemotronVLImageProcessor`
- **Constants / 常量**: `DEFAULT_NUM_TILES`, `NUM_VIDEO_TILES`, `DESIRED_FPS`, `MAX_FRAMES`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `logging`, `math`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`
- **Local Modules / 本地模块**: `sglang.srt.configs.nano_nemotron_vl`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.nano_nemotron_vl`, `sglang.srt.models.parakeet`, `sglang.srt.multimodal.audio_from_video`, `sglang.srt.multimodal.evs`, `sglang.srt.multimodal.internvl_utils`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils.common`
