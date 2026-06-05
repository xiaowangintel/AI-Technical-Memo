# transformers_auto.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/transformers_auto.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for transformers auto models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 transformers auto 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Imports dependencies
```python
from typing import Optional

import torch

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
from sglang.srt.utils import load_image
```
**EN:** This block groups related imports for the module, including typing.Optional, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Optional, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 3 项，为后续代码准备所需名称。

### Lines 15-16: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-22: Defines function _first_attr
```python
def _first_attr(obj, names: tuple[str, ...], default=None):
    for name in names:
        value = getattr(obj, name, None)
        if value is not None:
            return value
    return default
```
**EN:** This block defines function `_first_attr`. Parameters: obj, names, default.
**CN:** 该代码块定义函数 `_first_attr`。 参数包括 obj、names、default。

### Lines 23-24: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-31: Defines function _uses_mrope
```python
def _uses_mrope(hf_config) -> bool:
    text_config = getattr(hf_config, "text_config", hf_config)
    rope_scaling = getattr(text_config, "rope_scaling", None) or {}
    if isinstance(rope_scaling, dict) and "mrope_section" in rope_scaling:
        return True
    rope_type = str(getattr(text_config, "rope_type", "")).lower()
    return "mrope" in rope_type
```
**EN:** This block defines function `_uses_mrope`. Parameters: hf_config.
**CN:** 该代码块定义函数 `_uses_mrope`。 参数包括 hf_config。

### Lines 32-33: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 34-34: Declares class TransformersAutoMultimodalProcessor
```python
class TransformersAutoMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `TransformersAutoMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor. Generic multimodal processor for the Transformers backend. Unlike model-specific processors that rely on regex-based token matching in the raw prompt, this processor applies the HF processor directly to the prompt text + raw media.
**CN:** 该代码块声明类 `TransformersAutoMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。 文档字符串摘要：Generic multimodal processor for the Transformers backend. Unlike model-specific processors that rely on regex-based token matching in the raw prompt, this processor applies the HF processor directly to the prompt text + raw media.

### Lines 35-42: Documents the scope
```python
    """Generic multimodal processor for the Transformers backend.

    Unlike model-specific processors that rely on regex-based token matching
    in the raw prompt, this processor applies the HF processor directly to
    the prompt text + raw media.  This handles models like Gemma3 where the
    chat template uses a marker (``<start_of_image>``) that the HF processor
    internally expands into placeholder tokens.
    """
```
**EN:** This string literal serves as documentation for the TransformersAutoMultimodalProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 TransformersAutoMultimodalProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 43-43: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 44-44: Declares models
```python
    models = []
```
**EN:** This statement initializes models in the TransformersAutoMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 TransformersAutoMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 45-45: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 46-74: Defines function TransformersAutoMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.mm_tokens = MultimodalSpecialTokens(
            image_token=getattr(_processor, "image_token", None),
            video_token=getattr(_processor, "video_token", None),
            audio_token=getattr(_processor, "audio_token", None),
            image_token_id=_first_attr(
                hf_config,
                ("image_token_id", "image_token_index", "im_token_id"),
            ),
            video_token_id=_first_attr(
                hf_config,
                ("video_token_id",),
            ),
            audio_token_id=_first_attr(
                hf_config,
                ("audio_token_id",),
            ),
        ).build(_processor)

        self._is_mrope = _uses_mrope(hf_config)
        if self._is_mrope:
            vision_config = getattr(hf_config, "vision_config", None)
            self._spatial_merge_size = getattr(vision_config, "spatial_merge_size", 2)
            self._tokens_per_second = getattr(vision_config, "tokens_per_second", None)
            self._vision_start_token_id = _first_attr(
                hf_config, ("vision_start_token_id",)
            )
            self._model_type = getattr(hf_config, "model_type", "")
```
**EN:** This block defines function `TransformersAutoMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `TransformersAutoMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 75-75: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 76-96: Defines function TransformersAutoMultimodalProcessor._compute_mrope_positions
```python
    def _compute_mrope_positions(
        self,
        input_ids: list[int],
        image_grid_thw: Optional[torch.Tensor] = None,
        video_grid_thw: Optional[torch.Tensor] = None,
    ):
        from sglang.srt.layers.rotary_embedding import MRotaryEmbedding

        input_ids_tensor = torch.tensor(input_ids, dtype=torch.long).unsqueeze(0)
        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index(
            spatial_merge_size=self._spatial_merge_size,
            image_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id or -1,
            vision_start_token_id=self._vision_start_token_id,
            model_type=self._model_type,
            input_ids=input_ids_tensor,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
            tokens_per_second=self._tokens_per_second,
        )
        return mrope_positions.squeeze(1), mrope_position_delta
```
**EN:** This block defines function `TransformersAutoMultimodalProcessor._compute_mrope_positions`. Parameters: self, input_ids, image_grid_thw, video_grid_thw.
**CN:** 该代码块定义函数 `TransformersAutoMultimodalProcessor._compute_mrope_positions`。 参数包括 self、input_ids、image_grid_thw、video_grid_thw。

### Lines 97-97: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 98-108: Defines function TransformersAutoMultimodalProcessor._load_images
```python
    def _load_images(self, image_data) -> list:
        """Download / decode images from URLs, file paths, or base64."""
        if not image_data:
            return []
        images = []
        for data in image_data:
            img, _ = load_image(data)
            if img.mode != "RGB":
                img = img.convert("RGB")
            images.append(img)
        return images
```
**EN:** This block defines function `TransformersAutoMultimodalProcessor._load_images`. Parameters: self, image_data. Download / decode images from URLs, file paths, or base64.
**CN:** 该代码块定义函数 `TransformersAutoMultimodalProcessor._load_images`。 参数包括 self、image_data。 文档字符串摘要：Download / decode images from URLs, file paths, or base64.

### Lines 109-109: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 110-123: Defines function TransformersAutoMultimodalProcessor._apply_hf_processor
```python
    def _apply_hf_processor(self, text: str, images=None, videos=None):
        """Run the HF processor on text + media and return the full output.

        This is the key method that makes the generic processor work for
        models with non-trivial token expansion (Gemma3, PaliGemma, etc.).
        The HF processor handles chat-template expansion, image token
        insertion, and tokenization in one shot.
        """
        kwargs = {}
        if images:
            kwargs["images"] = images
        if videos:
            kwargs["videos"] = videos
        return self._processor(text=text, return_tensors="pt", **kwargs)
```
**EN:** This block defines function `TransformersAutoMultimodalProcessor._apply_hf_processor`. Parameters: self, text, images, videos. Run the HF processor on text + media and return the full output. This is the key method that makes the generic processor work for models with non-trivial token expansion (Gemma3, PaliGemma, etc.).
**CN:** 该代码块定义函数 `TransformersAutoMultimodalProcessor._apply_hf_processor`。 参数包括 self、text、images、videos。 文档字符串摘要：Run the HF processor on text + media and return the full output. This is the key method that makes the generic processor work for models with non-trivial token expansion (Gemma3, PaliGemma, etc.).

### Lines 124-124: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 125-143: Defines function TransformersAutoMultimodalProcessor._build_mm_items
```python
    def _build_mm_items(
        self, processor_output: dict, input_ids: torch.Tensor
    ) -> list[MultimodalDataItem]:
        """Extract MultimodalDataItem objects from the HF processor output."""
        items = self.collect_mm_items_from_processor_output(processor_output)

        modality_to_token_id = {
            Modality.IMAGE: self.mm_tokens.image_token_id,
            Modality.MULTI_IMAGES: self.mm_tokens.image_token_id,
            Modality.VIDEO: self.mm_tokens.video_token_id,
            Modality.AUDIO: self.mm_tokens.audio_token_id,
        }

        for item in items:
            token_id = modality_to_token_id.get(item.modality)
            if token_id is not None:
                item.offsets = self.get_mm_items_offset(input_ids, token_id)

        return items
```
**EN:** This block defines function `TransformersAutoMultimodalProcessor._build_mm_items`. Parameters: self, processor_output, input_ids. Extract MultimodalDataItem objects from the HF processor output.
**CN:** 该代码块定义函数 `TransformersAutoMultimodalProcessor._build_mm_items`。 参数包括 self、processor_output、input_ids。 文档字符串摘要：Extract MultimodalDataItem objects from the HF processor output.

### Lines 144-144: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the TransformersAutoMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 TransformersAutoMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 145-219: Defines async function TransformersAutoMultimodalProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data,
        audio_data,
        input_text,
        request_obj,
        **kwargs,
    ):
        video_data = getattr(request_obj, "video_data", None)
        if video_data is not None and not isinstance(video_data, list):
            video_data = [video_data]

        # Load raw media
        images = self._load_images(image_data)
        # TODO: video / audio loading when needed

        # Apply HF processor — handles token expansion internally
        processor_output = self._apply_hf_processor(
            text=input_text,
            images=images or None,
            videos=video_data or None,
        )

        input_ids = processor_output["input_ids"].flatten()

        # Build mm_items from processor output
        mm_items = self._build_mm_items(processor_output, input_ids)

        ret = MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
        )

        # Propagate token_type_ids for models that need it (Gemma3, PaliGemma)
        token_type_key = (
            "mm_token_type_ids"
            if "mm_token_type_ids" in processor_output
            else "token_type_ids"
        )
        if token_type_key in processor_output:
            ret.token_type_ids = processor_output[token_type_key].flatten().tolist()

        if self.mm_tokens.image_token_id is not None:
            ret.im_token_id = self.mm_tokens.image_token_id
        if self.mm_tokens.video_token_id is not None:
            ret.video_token_id = self.mm_tokens.video_token_id
        if self.mm_tokens.audio_token_id is not None:
            ret.audio_token_id = self.mm_tokens.audio_token_id

        image_start_id = _first_attr(
            self.hf_config,
            ("image_start_token_id", "vision_start_token_id", "im_start_id"),
        )
        image_end_id = _first_attr(
            self.hf_config,
            ("image_end_token_id", "vision_end_token_id", "im_end_id"),
        )
        if image_start_id is not None:
            ret.im_start_id = image_start_id
        if image_end_id is not None:
            ret.im_end_id = image_end_id

        # M-RoPE positions (Qwen2.5-VL, Qwen3-VL)
        if self._is_mrope:
            image_grid_thw = processor_output.get("image_grid_thw")
            video_grid_thw = processor_output.get("video_grid_thw")
            mrope_positions, mrope_position_delta = self._compute_mrope_positions(
                ret.input_ids,
                image_grid_thw=image_grid_thw,
                video_grid_thw=video_grid_thw,
            )
            ret.mrope_positions = mrope_positions
            ret.mrope_position_delta = mrope_position_delta

        return ret
```
**EN:** This block defines async function `TransformersAutoMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `TransformersAutoMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `TransformersAutoMultimodalProcessor`
- **Functions / 函数**: `_first_attr`, `_uses_mrope`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.layers.rotary_embedding`, `sglang.srt.managers.schedule_batch`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils`
