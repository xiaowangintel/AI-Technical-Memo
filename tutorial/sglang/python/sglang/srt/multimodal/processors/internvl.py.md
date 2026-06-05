# internvl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/internvl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for internvl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 internvl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Comments and module notes
```python
# Adapted from https://huggingface.co/OpenGVLab/InternVL2-4B/blob/main/modeling_intern_vit.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 3-24: Imports dependencies
```python
import logging
from functools import lru_cache
from typing import List

import numpy as np
import torch
from PIL import Image

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.interns1 import InternS1ForConditionalGeneration
from sglang.srt.models.internvl import InternVLChatModel
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    BaseMultiModalProcessorOutput,
    MultimodalSpecialTokens,
)
from sglang.srt.utils import get_device
from sglang.srt.utils.video_decoder import VideoDecoderWrapper
```
**EN:** This block groups related imports for the module, including logging, functools.lru_cache, typing.List, numpy, torch, and 11 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, functools.lru_cache, typing.List, numpy, torch 等 11 项，为后续代码准备所需名称。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-26: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 27-28: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 29-29: Declares class InternVLProcessor
```python
class InternVLProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `InternVLProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `InternVLProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 30-31: Declares models, gpu_image_decode
```python
    models = [InternVLChatModel, InternS1ForConditionalGeneration]
    gpu_image_decode = False  # InternVL HF processor does not support tensor inputs
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 32-32: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 33-35: Declares IMAGENET_MEAN, IMAGENET_STD, IMAGE_MAX_NUM
```python
    IMAGENET_MEAN = [0.485, 0.456, 0.406]
    IMAGENET_STD = [0.229, 0.224, 0.225]
    IMAGE_MAX_NUM = 12
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including IMAGENET_MEAN, IMAGENET_STD, IMAGE_MAX_NUM. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 IMAGENET_MEAN, IMAGENET_STD, IMAGE_MAX_NUM。将这些赋值集中在一起有助于理解周边配置。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-39: Declares DEFAULT_VIDEO_NUM_FRAMES, VIDEO_MAX_NUM, VIDEO_USE_THUMBNAIL
```python
    DEFAULT_VIDEO_NUM_FRAMES = 32
    VIDEO_MAX_NUM = 1
    VIDEO_USE_THUMBNAIL = False
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including DEFAULT_VIDEO_NUM_FRAMES, VIDEO_MAX_NUM, VIDEO_USE_THUMBNAIL. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 DEFAULT_VIDEO_NUM_FRAMES, VIDEO_MAX_NUM, VIDEO_USE_THUMBNAIL。将这些赋值集中在一起有助于理解周边配置。

### Lines 40-40: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-42: Declares CONTEXT_FALLBACK, CONTEXT_RESERVED
```python
    CONTEXT_FALLBACK = 40960
    CONTEXT_RESERVED = 256
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including CONTEXT_FALLBACK, CONTEXT_RESERVED. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 CONTEXT_FALLBACK, CONTEXT_RESERVED。将这些赋值集中在一起有助于理解周边配置。

### Lines 43-44: Comments and module notes
```python

    # OpenAI multimodal placeholder tokens
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the InternVLProcessor.
**CN:** 该范围包含注释或说明，用于记录 InternVLProcessor 的假设、来源或实现备注。

### Lines 45-46: Declares IMAGE_PLACEHOLDER_TOKEN, VIDEO_PLACEHOLDER_TOKEN
```python
    IMAGE_PLACEHOLDER_TOKEN = "<image>"
    VIDEO_PLACEHOLDER_TOKEN = "<video>"
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including IMAGE_PLACEHOLDER_TOKEN, VIDEO_PLACEHOLDER_TOKEN. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 IMAGE_PLACEHOLDER_TOKEN, VIDEO_PLACEHOLDER_TOKEN。将这些赋值集中在一起有助于理解周边配置。

### Lines 47-47: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 48-50: Declares IMG_START, IMG_END, IMG_CONTEXT
```python
    IMG_START = "<img>"
    IMG_END = "</img>"
    IMG_CONTEXT = "<IMG_CONTEXT>"
```
**EN:** This block initializes a related set of values in the InternVLProcessor, including IMG_START, IMG_END, IMG_CONTEXT. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 InternVLProcessor 中初始化一组相关值，包括 IMG_START, IMG_END, IMG_CONTEXT。将这些赋值集中在一起有助于理解周边配置。

### Lines 51-51: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 52-61: Defines function InternVLProcessor._get_normalize_tensors
```python
    @staticmethod
    @lru_cache(maxsize=1)
    def _get_normalize_tensors(device="cuda", dtype=torch.float32):
        mean = torch.tensor(
            InternVLProcessor.IMAGENET_MEAN, device=device, dtype=dtype
        ).view(-1, 1, 1)
        std = torch.tensor(
            InternVLProcessor.IMAGENET_STD, device=device, dtype=dtype
        ).view(-1, 1, 1)
        return mean, std
```
**EN:** This block defines function `InternVLProcessor._get_normalize_tensors`. Parameters: device, dtype. Decorators: staticmethod, lru_cache.
**CN:** 该代码块定义函数 `InternVLProcessor._get_normalize_tensors`。 参数包括 device、dtype。 装饰器包括 staticmethod、lru_cache。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-135: Defines function InternVLProcessor.__init__ (part 1)
```python
    def __init__(self, hf_config, server_args, _image_processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _image_processor, *args, **kwargs)

        image_size = (
            getattr(hf_config, "force_image_size", None)
            or hf_config.vision_config.image_size
        )
        patch_size = hf_config.vision_config.patch_size
        if isinstance(image_size, list):
            image_size = image_size[0]
        if isinstance(patch_size, list):
            patch_size = patch_size[0]

        if hasattr(self._processor, "tokenizer"):
            tokenizer = self._processor.tokenizer
        else:
            tokenizer = self._processor
        self.tokenizer = tokenizer

        # Support both InternVL (llm_config) and InternS1 (text_config).
        # Different multimodal models use different field names for the text backbone:
        # - InternVL uses: hf_config.llm_config
        # - InternS1 uses: hf_config.text_config
        # - Some store architectures at top-level
        text_cfg = (
            getattr(hf_config, "llm_config", None)
            or getattr(hf_config, "text_config", None)
            or hf_config
        )
        llm_arch = (getattr(text_cfg, "architectures", []) or [None])[0]
        self.llm_arch = llm_arch
        video_token_map = {
            "Qwen2ForCausalLM": "<|video_pad|>",
            "Qwen3ForCausalLM": "<|video_pad|>",
            "Qwen3MoeForCausalLM": "<|video_pad|>",
            "GptOssForCausalLM": "<|reserved_200000|>",
        }
        self.VIDEO_CONTEXT_TOKEN = video_token_map.get(llm_arch, None)
        self.video_token_id = (
            tokenizer.convert_tokens_to_ids(self.VIDEO_CONTEXT_TOKEN)
            if self.VIDEO_CONTEXT_TOKEN
            else None
        )

        self.image_token_id = (
            tokenizer.convert_tokens_to_ids(self.IMG_CONTEXT)
            if self.IMG_CONTEXT
            else None
        )
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (hf_config.downsample_ratio**2)
        )

        self.img_start_token_id = tokenizer.convert_tokens_to_ids(self.IMG_START)
        self.img_end_token_id = tokenizer.convert_tokens_to_ids(self.IMG_END)

        # Placeholder token use <image>/<video>
        # Offset token id use IMG_CONTEXT / VIDEO_CONTEXT
        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMAGE_PLACEHOLDER_TOKEN,
            image_token_id=self.image_token_id,
            video_token=self.VIDEO_PLACEHOLDER_TOKEN,
            video_token_id=self.video_token_id,
        ).build(_image_processor)

        # Cache token id for IMG_CONTEXT (used by both branches)
        self.img_context_token_id = tokenizer.convert_tokens_to_ids(self.IMG_CONTEXT)

        # InternLM2 legacy multimodal tokens: use <IMG_CONTEXT> as placeholder
        self.mm_tokens_internlm2 = MultimodalSpecialTokens(
            image_token=self.IMG_CONTEXT,
            image_token_id=self.img_context_token_id,
        ).build(_image_processor)
```
**EN:** This block defines function `InternVLProcessor.__init__`. Parameters: self, hf_config, server_args, _image_processor. This subsection covers lines 63-135 of the same logical block.
**CN:** 该代码块定义函数 `InternVLProcessor.__init__`。 参数包括 self、hf_config、server_args、_image_processor。 本小节覆盖同一逻辑块中的第 63-135 行。

### Lines 136-136: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 137-143: Defines function InternVLProcessor.__init__ (part 2)
```python
        self.max_context_len = (
            getattr(server_args, "context_length", None)
            or getattr(server_args, "max_context_len", None)
            or getattr(hf_config, "max_position_embeddings", None)
            or getattr(text_cfg, "max_position_embeddings", None)
            or self.CONTEXT_FALLBACK
        )
```
**EN:** This block defines function `InternVLProcessor.__init__`. Parameters: self, hf_config, server_args, _image_processor. This subsection covers lines 137-143 of the same logical block.
**CN:** 该代码块定义函数 `InternVLProcessor.__init__`。 参数包括 self、hf_config、server_args、_image_processor。 本小节覆盖同一逻辑块中的第 137-143 行。

### Lines 144-144: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 145-208: Defines function InternVLProcessor.dynamic_preprocess
```python
    @staticmethod
    def dynamic_preprocess(
        tensor, image_size=448, max_num=IMAGE_MAX_NUM, use_thumbnail=False
    ):
        # Tensor: (C,H,W) float on GPU
        C, H, W = tensor.shape
        aspect_ratio = W / H

        # Generate all possible aspect ratios
        target_ratios = set(
            (i, j)
            for n in range(1, max_num + 1)
            for i in range(1, n + 1)
            for j in range(1, n + 1)
            if i * j <= max_num
        )
        target_ratios = sorted(target_ratios, key=lambda x: x[0] * x[1])

        # Find closest ratio
        best_ratio_diff = float("inf")
        best_ratio = (1, 1)

        for x, y in target_ratios:
            target_ar = x / y
            diff = abs(aspect_ratio - target_ar)
            blocks = x * y
            best_blocks = best_ratio[0] * best_ratio[1]

            if diff < best_ratio_diff:
                best_ratio_diff = diff
                best_ratio = (x, y)
            elif diff == best_ratio_diff and blocks > best_blocks:
                best_ratio = (x, y)

        target_w, target_h = image_size * best_ratio[0], image_size * best_ratio[1]
        blocks = best_ratio[0] * best_ratio[1]

        # Resize on GPU
        resized = torch.nn.functional.interpolate(
            tensor.unsqueeze(0),
            size=(target_h, target_w),
            mode="bicubic",
            align_corners=False,
        ).squeeze(0)

        # Split into tiles
        tiles = []
        for i in range(blocks):
            x = (i % best_ratio[0]) * image_size
            y = (i // best_ratio[0]) * image_size
            tile = resized[:, y : y + image_size, x : x + image_size]
            tiles.append(tile)

        # Add thumbnail if needed
        if use_thumbnail and len(tiles) > 1:
            thumb = torch.nn.functional.interpolate(
                tensor.unsqueeze(0),
                size=(image_size, image_size),
                mode="bicubic",
                align_corners=False,
            ).squeeze(0)
            tiles.append(thumb)

        return torch.stack(tiles).to(torch.bfloat16)
```
**EN:** This block defines function `InternVLProcessor.dynamic_preprocess`. Parameters: tensor, image_size, max_num, use_thumbnail. Decorators: staticmethod.
**CN:** 该代码块定义函数 `InternVLProcessor.dynamic_preprocess`。 参数包括 tensor、image_size、max_num、use_thumbnail。 装饰器包括 staticmethod。

### Lines 209-209: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 210-212: Defines function InternVLProcessor._open_video_reader
```python
    @staticmethod
    def _open_video_reader(path: str):
        return VideoDecoderWrapper(path)
```
**EN:** This block defines function `InternVLProcessor._open_video_reader`. Parameters: path. Decorators: staticmethod.
**CN:** 该代码块定义函数 `InternVLProcessor._open_video_reader`。 参数包括 path。 装饰器包括 staticmethod。

### Lines 213-213: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 214-230: Defines function InternVLProcessor._ensure_placeholders_before_assistant
```python
    def _ensure_placeholders_before_assistant(
        self, prompt: str, placeholder: str, want: int
    ) -> str:
        if want <= 0:
            return prompt
        have = (prompt or "").count(placeholder)
        missing = want - have
        if missing <= 0:
            return prompt

        insert = "\n" + "\n".join([placeholder] * missing) + "\n"

        marker = "<|im_start|>assistant"
        idx = (prompt or "").rfind(marker)
        if idx != -1:
            return (prompt or "")[:idx] + insert + (prompt or "")[idx:]
        return (prompt or "") + insert
```
**EN:** This block defines function `InternVLProcessor._ensure_placeholders_before_assistant`. Parameters: self, prompt, placeholder, want.
**CN:** 该代码块定义函数 `InternVLProcessor._ensure_placeholders_before_assistant`。 参数包括 self、prompt、placeholder、want。

### Lines 231-231: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 232-237: Defines function InternVLProcessor._token_len
```python
    def _token_len(self, text: str) -> int:
        try:
            ids = self.tokenizer(text, return_tensors="pt")["input_ids"].flatten()
            return int(ids.numel())
        except Exception:
            return 0
```
**EN:** This block defines function `InternVLProcessor._token_len`. Parameters: self, text.
**CN:** 该代码块定义函数 `InternVLProcessor._token_len`。 参数包括 self、text。

### Lines 238-238: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 239-257: Defines function InternVLProcessor._resolve_video_num_frames
```python
    def _resolve_video_num_frames(
        self, *, requested: int, num_videos: int, text_len: int, image_tile_cnt: int
    ) -> int:
        if num_videos <= 0:
            return 0
        if not self.VIDEO_CONTEXT_TOKEN or not self.video_token_id:
            return 0
        image_tokens = image_tile_cnt * self.num_image_token
        budget = (
            int(self.max_context_len)
            - int(text_len)
            - int(image_tokens)
            - int(self.CONTEXT_RESERVED)
        )
        if budget <= 0:
            return 1
        max_total_frames = max(1, budget // self.num_image_token)
        frames_per_video = max(1, max_total_frames // max(num_videos, 1))
        return max(1, min(int(requested), int(frames_per_video)))
```
**EN:** This block defines function `InternVLProcessor._resolve_video_num_frames`. Parameters: self.
**CN:** 该代码块定义函数 `InternVLProcessor._resolve_video_num_frames`。 参数包括 self。

### Lines 258-258: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 259-268: Defines function InternVLProcessor._has_special_format
```python
    @staticmethod
    def _has_special_format(image_data, video_data):
        """Check if any input items use processor_output or precomputed_embedding format."""
        for data in list(image_data or []) + list(video_data or []):
            if isinstance(data, dict) and data.get("format") in (
                "processor_output",
                "precomputed_embedding",
            ):
                return True
        return False
```
**EN:** This block defines function `InternVLProcessor._has_special_format`. Parameters: image_data, video_data. Decorators: staticmethod. Check if any input items use processor_output or precomputed_embedding format.
**CN:** 该代码块定义函数 `InternVLProcessor._has_special_format`。 参数包括 image_data、video_data。 装饰器包括 staticmethod。 文档字符串摘要：Check if any input items use processor_output or precomputed_embedding format.

### Lines 269-269: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 270-348: Defines async function InternVLProcessor._process_special_format
```python
    async def _process_special_format(
        self, image_data, video_data, input_text, request_obj, **kwargs
    ):
        """Handle processor_output and precomputed_embedding input formats.

        Delegates to the base class process_and_combine_mm_data which has
        built-in support for these formats.
        """
        # When user provides input_ids directly, input_text may be a list of ints
        if isinstance(input_text, list):
            user_input_ids = input_text
            prompt = ""
        else:
            user_input_ids = None
            prompt = input_text or ""

        # When the prompt is empty (user provided input_ids directly),
        # load_mm_data can't match multimodal tokens to data items.
        # Build BaseMultiModalProcessorOutput directly from the dict items.
        if not prompt and (image_data or video_data):
            images = [d for d in (image_data or []) if isinstance(d, dict)]
            videos = [d for d in (video_data or []) if isinstance(d, dict)]

            # Raise if raw (non-dict) images/videos were silently filtered out.
            # InternVL cannot process raw images without a text prompt because
            # dynamic tiling and placeholder expansion require the prompt string.
            raw_img_dropped = len(image_data or []) - len(images)
            raw_vid_dropped = len(video_data or []) - len(videos)
            if raw_img_dropped > 0 or raw_vid_dropped > 0:
                raise ValueError(
                    f"[internvl] Cannot process raw images/videos with pre-tokenized "
                    f"input_ids. Provide multimodal data in 'processor_output' or "
                    f"'precomputed_embedding' format, or use a text prompt instead. "
                    f"(raw images dropped: {raw_img_dropped}, "
                    f"raw videos dropped: {raw_vid_dropped})"
                )

            base_output = BaseMultiModalProcessorOutput(
                input_text=prompt,
                images=images,
                videos=videos,
            )
        else:
            base_output = self.load_mm_data(
                prompt=prompt,
                image_data=image_data,
                video_data=video_data,
                multimodal_tokens=self.mm_tokens,
                discard_alpha_channel=True,
            )

        mm_items, input_ids_tensor, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        # If user provided input_ids directly, use those and recompute offsets
        if user_input_ids is not None:
            input_ids_tensor = torch.tensor(user_input_ids, dtype=torch.long)
            for mm_item in mm_items:
                if (
                    mm_item.modality == Modality.VIDEO
                    and self.video_token_id is not None
                ):
                    mm_token_id = self.video_token_id
                else:
                    mm_token_id = self.img_context_token_id
                mm_item.offsets = self.get_mm_items_offset(
                    input_ids=input_ids_tensor,
                    mm_token_id=mm_token_id,
                )

        return MultimodalProcessorOutput(
            input_ids=input_ids_tensor.flatten().tolist(),
            mm_items=mm_items,
            im_start_id=self.img_start_token_id,
            im_end_id=self.img_end_token_id,
            im_token_id=self.img_context_token_id,
            video_token_id=self.video_token_id,
        )
```
**EN:** This block defines async function `InternVLProcessor._process_special_format`. Parameters: self, image_data, video_data, input_text, request_obj. Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats.
**CN:** 该代码块定义异步函数 `InternVLProcessor._process_special_format`。 参数包括 self、image_data、video_data、input_text、request_obj。 文档字符串摘要：Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats.

### Lines 349-349: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 350-383: Defines async function InternVLProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self, image_data, input_text, request_obj, **kwargs
    ):
        video_data = getattr(request_obj, "video_data", None) or []

        # Handle processor_output and precomputed_embedding formats
        if isinstance(input_text, list) or self._has_special_format(
            image_data, video_data
        ):
            return await self._process_special_format(
                image_data=image_data,
                video_data=video_data,
                input_text=input_text,
                request_obj=request_obj,
                **kwargs,
            )

        is_internlm2 = self.llm_arch == "InternLM2ForCausalLM"

        if is_internlm2:
            return await self.process_internlm2_mm_data_async(
                image_data=image_data,
                input_text=input_text,
                request_obj=request_obj,
                **kwargs,
            )
        else:
            # Default branch uses OpenAI-style placeholders
            return await self.process_qwen_mm_data_async(
                image_data=image_data,
                input_text=input_text,
                request_obj=request_obj,
                **kwargs,
            )
```
**EN:** This block defines async function `InternVLProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

### Lines 384-384: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 385-461: Defines async function InternVLProcessor.process_qwen_mm_data_async (part 1)
```python
    async def process_qwen_mm_data_async(
        self, image_data, input_text, request_obj, **kwargs
    ):

        img_max_num = (
            getattr(request_obj, "image_max_dynamic_patch", None)
            or getattr(request_obj, "max_dynamic_patch", None)
            or kwargs.get("image_max_dynamic_patch")
            or kwargs.get("max_dynamic_patch")
            or self.IMAGE_MAX_NUM
        )
        img_max_num = max(1, int(img_max_num))

        vid_max_num = (
            getattr(request_obj, "video_max_dynamic_patch", None)
            or getattr(request_obj, "max_dynamic_patch", None)
            or kwargs.get("video_max_dynamic_patch")
            or kwargs.get("max_dynamic_patch")
            or self.VIDEO_MAX_NUM
        )
        vid_max_num = max(1, int(vid_max_num))

        # Qwen/Qwen3 branch: OpenAI-style placeholders <image>/<video>
        prompt = input_text or ""
        video_data = getattr(request_obj, "video_data", None) or []

        if image_data:
            prompt = self._ensure_placeholders_before_assistant(
                prompt, self.IMAGE_PLACEHOLDER_TOKEN, len(image_data)
            )
        if video_data:
            prompt = self._ensure_placeholders_before_assistant(
                prompt, self.VIDEO_PLACEHOLDER_TOKEN, len(video_data)
            )

        logger.info(
            "[internvl][qwen] placeholders image=%d video=%d",
            prompt.count(self.IMAGE_PLACEHOLDER_TOKEN),
            prompt.count(self.VIDEO_PLACEHOLDER_TOKEN),
        )

        base_output = self.load_mm_data(
            prompt=prompt,
            image_data=image_data,
            video_data=video_data,
            multimodal_tokens=self.mm_tokens,  # expects <image>/<video>
            discard_alpha_channel=True,
        )

        logger.info(
            "[internvl][qwen] loaded images=%d videos=%d",
            len(base_output.images),
            len(base_output.videos),
        )

        mean, std = self._get_normalize_tensors(device=get_device())

        # ----- Images -> tiles -----
        num_patches_list: List[int] = []
        pixel_values_list: List[torch.Tensor] = []

        for image in base_output.images:
            if isinstance(image, Image.Image):
                img_np = np.array(image.convert("RGB"))
                tensor = (
                    torch.from_numpy(img_np).permute(2, 0, 1).to(get_device()).float()
                    / 255.0
                )
            else:
                tensor = image.to(get_device())

            tensor = (tensor - mean) / std
            tiles = self.dynamic_preprocess(
                tensor, image_size=448, max_num=img_max_num, use_thumbnail=True
            )
            pixel_values_list.append(tiles)
            num_patches_list.append(int(tiles.shape[0]))
```
**EN:** This block defines async function `InternVLProcessor.process_qwen_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 385-461 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_qwen_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 385-461 行。

### Lines 462-462: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 463-541: Defines async function InternVLProcessor.process_qwen_mm_data_async (part 2)
```python
        if image_data and not pixel_values_list:
            raise ValueError(
                "[internvl][qwen] image_data provided but no images parsed from prompt placeholders"
            )

        image_tensor = (
            torch.cat(pixel_values_list, dim=0) if pixel_values_list else None
        )

        # ----- Videos -> frame tiles (optional) -----
        video_tensor = None
        video_patch_lists = []
        video_pixel_values = []

        requested_frames = int(
            kwargs.get("video_num_frames", self.DEFAULT_VIDEO_NUM_FRAMES)
        )
        num_frames = self._resolve_video_num_frames(
            requested=requested_frames,
            num_videos=len(base_output.videos),
            text_len=self._token_len(base_output.input_text or prompt),
            image_tile_cnt=int(sum(num_patches_list)) if num_patches_list else 0,
        )

        if base_output.videos and num_frames > 0 and self.video_token_id is not None:
            for video in base_output.videos:
                is_video_obj = isinstance(video, VideoDecoderWrapper)
                vr = video if is_video_obj else self._open_video_reader(str(video))
                max_frame = len(vr) - 1
                frame_indices = (
                    [0]
                    if num_frames == 1
                    else np.linspace(0, max_frame, num=num_frames, dtype=int).tolist()
                )

                per_video_tiles = []
                per_video_patch_cnt = []
                for fi in frame_indices:
                    img_np = vr[int(fi)]
                    frame_t = (
                        torch.from_numpy(img_np)
                        .permute(2, 0, 1)
                        .to(get_device())
                        .float()
                        / 255.0
                    )
                    frame_t = (frame_t - mean) / std

                    tiles = self.dynamic_preprocess(
                        frame_t,
                        image_size=448,
                        max_num=vid_max_num,
                        use_thumbnail=self.VIDEO_USE_THUMBNAIL,
                    )
                    per_video_tiles.append(tiles)
                    per_video_patch_cnt.append(int(tiles.shape[0]))

                pv = torch.cat(per_video_tiles, dim=0)
                video_pixel_values.append(pv)
                video_patch_lists.append(per_video_patch_cnt)

            video_tensor = (
                torch.cat(video_pixel_values, dim=0) if video_pixel_values else None
            )

        # ----- Build prompt text with <img> + CONTEXT*n + </img> -----
        img_ph = "<<<__IMG_PLACEHOLDER__>>>"
        vid_ph = "<<<__VID_PLACEHOLDER__>>>"

        input_text_mid = base_output.input_text or prompt
        input_text_mid = input_text_mid.replace(self.IMAGE_PLACEHOLDER_TOKEN, img_ph)
        input_text_mid = input_text_mid.replace(self.IMG_CONTEXT, img_ph)

        if self.VIDEO_CONTEXT_TOKEN and self.video_token_id is not None:
            input_text_mid = input_text_mid.replace(
                self.VIDEO_PLACEHOLDER_TOKEN, vid_ph
            )
        else:
            input_text_mid = input_text_mid.replace(self.VIDEO_PLACEHOLDER_TOKEN, "")
```
**EN:** This block defines async function `InternVLProcessor.process_qwen_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 463-541 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_qwen_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 463-541 行。

### Lines 542-542: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 543-612: Defines async function InternVLProcessor.process_qwen_mm_data_async (part 3)
```python
        input_text_updated = input_text_mid

        # Expand images
        for num_patches in num_patches_list:
            image_tokens = (
                self.IMG_START
                + (self.IMG_CONTEXT * (self.num_image_token * int(num_patches)))
                + self.IMG_END
            )
            input_text_updated = input_text_updated.replace(img_ph, image_tokens, 1)

        # Expand videos (each frame is one <img>...</img>)
        if video_patch_lists and self.VIDEO_CONTEXT_TOKEN:
            for frame_patch_list in video_patch_lists:
                frame_lines = []
                for i, patch_cnt in enumerate(frame_patch_list):
                    ctx_cnt = int(self.num_image_token) * int(patch_cnt)
                    frame_tokens = (
                        self.IMG_START
                        + (self.VIDEO_CONTEXT_TOKEN * ctx_cnt)
                        + self.IMG_END
                    )
                    frame_lines.append(f"Frame {i+1}: {frame_tokens}")
                video_tokens = "\n".join(frame_lines) + "\n"
                input_text_updated = input_text_updated.replace(vid_ph, video_tokens, 1)

        # Tokenize
        input_ids_tensor = self.tokenizer(input_text_updated, return_tensors="pt")[
            "input_ids"
        ].flatten()
        input_ids = input_ids_tensor.tolist()

        # Offsets
        image_offsets = []
        if image_tensor is not None:
            image_offsets = self.get_mm_items_offset(
                input_ids=input_ids_tensor.to(get_device()),
                mm_token_id=self.img_context_token_id,
            )

        video_offsets = []
        if video_tensor is not None and self.video_token_id is not None:
            video_offsets = self.get_mm_items_offset(
                input_ids=input_ids_tensor.to(get_device()),
                mm_token_id=self.video_token_id,
            )

        items = []
        if image_tensor is not None:
            # Split per-image for better cache granularity
            assert len(num_patches_list) == len(image_offsets), (
                f"InternVL: num_patches_list ({len(num_patches_list)}) != "
                f"image_offsets ({len(image_offsets)})"
            )
            cumulative = 0
            for i, num_patches in enumerate(num_patches_list):
                items.append(
                    MultimodalDataItem(
                        feature=image_tensor[cumulative : cumulative + num_patches],
                        modality=Modality.IMAGE,
                        offsets=[image_offsets[i]],
                    )
                )
                cumulative += num_patches
        if video_tensor is not None:
            items.append(
                MultimodalDataItem(
                    feature=video_tensor, modality=Modality.VIDEO, offsets=video_offsets
                )
            )
```
**EN:** This block defines async function `InternVLProcessor.process_qwen_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 543-612 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_qwen_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 543-612 行。

### Lines 613-613: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 614-621: Defines async function InternVLProcessor.process_qwen_mm_data_async (part 4)
```python
        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=items,
            im_start_id=self.img_start_token_id,
            im_end_id=self.img_end_token_id,
            im_token_id=self.img_context_token_id,
            video_token_id=self.video_token_id,
        )
```
**EN:** This block defines async function `InternVLProcessor.process_qwen_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 614-621 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_qwen_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 614-621 行。

### Lines 622-622: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternVLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternVLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 623-698: Defines async function InternVLProcessor.process_internlm2_mm_data_async (part 1)
```python
    async def process_internlm2_mm_data_async(
        self, image_data, input_text, request_obj, **kwargs
    ):
        # InternLM2 branch: legacy placeholder <IMG_CONTEXT> (stable for InternLM2 prompt behavior)
        prompt = input_text or ""
        video_data = getattr(request_obj, "video_data", None) or []
        if video_data:
            logger.warning(
                "[internvl][internlm2] video input ignored for InternLM2 branch"
            )

        # Convert any OpenAI-style <image> into <IMG_CONTEXT>
        prompt = prompt.replace(self.IMAGE_PLACEHOLDER_TOKEN, self.IMG_CONTEXT)

        if image_data:
            prompt = self._ensure_placeholders_before_assistant(
                prompt, self.IMG_CONTEXT, len(image_data)
            )

        logger.info(
            "[internvl][internlm2] placeholders img_context=%d",
            prompt.count(self.IMG_CONTEXT),
        )

        base_output = self.load_mm_data(
            prompt=prompt,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens_internlm2,  # expects <IMG_CONTEXT>
            discard_alpha_channel=True,
        )

        mean, std = self._get_normalize_tensors(device=get_device())

        num_patches_list: List[int] = []
        pixel_values_list: List[torch.Tensor] = []

        for image in base_output.images:
            if isinstance(image, Image.Image):
                img_np = np.array(image.convert("RGB"))
                tensor = (
                    torch.from_numpy(img_np).permute(2, 0, 1).to(get_device()).float()
                    / 255.0
                )
            else:
                tensor = image.to(get_device())

            tensor = (tensor - mean) / std
            tiles = self.dynamic_preprocess(
                tensor, image_size=448, max_num=12, use_thumbnail=True
            )
            pixel_values_list.append(tiles)
            num_patches_list.append(int(tiles.shape[0]))

        if image_data and not pixel_values_list:
            raise ValueError(
                "[internvl][internlm2] image_data provided but no images parsed from prompt placeholders"
            )

        pixel_values = (
            torch.cat(pixel_values_list, dim=0) if pixel_values_list else None
        )

        # Expand each <IMG_CONTEXT> into <img> + <IMG_CONTEXT>*N + </img>
        ph = "<<<__IMG_CONTEXT_PLACEHOLDER__>>>"
        input_text_base = (base_output.input_text or prompt).replace(
            self.IMG_CONTEXT, ph
        )

        input_text_updated = input_text_base
        for num_patches in num_patches_list:
            image_tokens = (
                self.IMG_START
                + (self.IMG_CONTEXT * (self.num_image_token * int(num_patches)))
                + self.IMG_END
            )
            input_text_updated = input_text_updated.replace(ph, image_tokens, 1)
```
**EN:** This block defines async function `InternVLProcessor.process_internlm2_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 623-698 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_internlm2_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 623-698 行。

### Lines 699-699: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 700-730: Defines async function InternVLProcessor.process_internlm2_mm_data_async (part 2)
```python
        # Tokenize
        input_ids_tensor = self.tokenizer(input_text_updated, return_tensors="pt")[
            "input_ids"
        ].flatten()
        input_ids = input_ids_tensor.tolist()

        # Offsets
        image_offsets = []
        if pixel_values is not None:
            image_offsets = self.get_mm_items_offset(
                input_ids=input_ids_tensor.to(get_device()),
                mm_token_id=self.img_context_token_id,
            )

        items = []
        if pixel_values is not None:
            # Split per-image for better cache granularity
            assert len(num_patches_list) == len(image_offsets), (
                f"InternVL: num_patches_list ({len(num_patches_list)}) != "
                f"image_offsets ({len(image_offsets)})"
            )
            cumulative = 0
            for i, num_patches in enumerate(num_patches_list):
                items.append(
                    MultimodalDataItem(
                        feature=pixel_values[cumulative : cumulative + num_patches],
                        modality=Modality.IMAGE,
                        offsets=[image_offsets[i]],
                    )
                )
                cumulative += num_patches
```
**EN:** This block defines async function `InternVLProcessor.process_internlm2_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 700-730 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_internlm2_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 700-730 行。

### Lines 731-731: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 732-739: Defines async function InternVLProcessor.process_internlm2_mm_data_async (part 3)
```python
        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=items,
            im_start_id=self.img_start_token_id,
            im_end_id=self.img_end_token_id,
            im_token_id=self.img_context_token_id,
            video_token_id=self.video_token_id,
        )
```
**EN:** This block defines async function `InternVLProcessor.process_internlm2_mm_data_async`. Parameters: self, image_data, input_text, request_obj. This subsection covers lines 732-739 of the same logical block.
**CN:** 该代码块定义异步函数 `InternVLProcessor.process_internlm2_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 732-739 行。

## Key Concepts / 关键概念
- **Classes / 类**: `InternVLProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `functools`, `logging`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.interns1`, `sglang.srt.models.internvl`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils`, `sglang.srt.utils.video_decoder`
