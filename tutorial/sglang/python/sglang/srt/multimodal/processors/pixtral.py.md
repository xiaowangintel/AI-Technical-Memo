# pixtral.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/pixtral.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for pixtral models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 pixtral 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Imports dependencies
```python
import math
from typing import List, Union

from transformers import PreTrainedTokenizerBase
from transformers.models.pixtral.image_processing_pixtral import (
    _num_image_tokens as _get_pixtral_hf_num_image_tokens,
)

from sglang.srt.managers.schedule_batch import Modality, MultimodalProcessorOutput
from sglang.srt.models.pixtral import (
    PixtralForConditionalGeneration,
    PixtralVisionModel,
)
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including math, typing.List, typing.Union, transformers.PreTrainedTokenizerBase, transformers.models.pixtral.image_processing_pixtral._num_image_tokens, and 6 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, typing.List, typing.Union, transformers.PreTrainedTokenizerBase, transformers.models.pixtral.image_processing_pixtral._num_image_tokens 等 6 项，为后续代码准备所需名称。

### Lines 18-19: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-20: Declares class PixtralProcessor
```python
class PixtralProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `PixtralProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `PixtralProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 21-22: Declares models, gpu_image_decode
```python
    models = [PixtralVisionModel, PixtralForConditionalGeneration]
    gpu_image_decode = False  # Pixtral processes loaded image as PIL image explicitly
```
**EN:** This block initializes a related set of values in the PixtralProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 PixtralProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the PixtralProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 PixtralProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-25: Declares PAD_TOKEN, DEFAULT_IMAGE_TOKEN
```python
    PAD_TOKEN = "<pad>"
    DEFAULT_IMAGE_TOKEN = "[IMG]"
```
**EN:** This block initializes a related set of values in the PixtralProcessor, including PAD_TOKEN, DEFAULT_IMAGE_TOKEN. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 PixtralProcessor 中初始化一组相关值，包括 PAD_TOKEN, DEFAULT_IMAGE_TOKEN。将这些赋值集中在一起有助于理解周边配置。

### Lines 26-26: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the PixtralProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 PixtralProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 27-64: Defines function PixtralProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.IM_TOKEN_ID = getattr(
            hf_config, "image_token_index", PixtralVisionModel.DEFAULT_IMAGE_TOKEN_ID
        )

        self.vision_config = hf_config.vision_config
        self.image_size = self.vision_config.image_size
        self.patch_size = self.vision_config.patch_size

        # spatial_merge_size may live on vision_config (Mistral native) or
        # on the top-level config (HF native Mistral3Config).
        self._spatial_merge_size = getattr(
            self.vision_config,
            "spatial_merge_size",
            getattr(hf_config, "spatial_merge_size", 1),
        )

        self._processor.patch_size = self.patch_size
        if self._spatial_merge_size > 1:
            self._processor.spatial_merge_size = self._spatial_merge_size

        tokenizer = (
            _processor
            if isinstance(_processor, PreTrainedTokenizerBase)
            else _processor.tokenizer
        )
        self.image_token = getattr(_processor, "image_token", self.DEFAULT_IMAGE_TOKEN)

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.image_token,
            image_token_id=self.IM_TOKEN_ID,
        ).build(_processor)
        tokenizer.add_special_tokens(
            {
                "pad_token": getattr(hf_config, "pad_token", self.PAD_TOKEN),
            }
        )
```
**EN:** This block defines function `PixtralProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `PixtralProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 65-65: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the PixtralProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 PixtralProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 66-134: Defines async function PixtralProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        mm_data = self.load_mm_data(
            prompt=input_text,
            multimodal_tokens=self.mm_tokens,
            image_data=image_data,
            return_text=True,
        )
        if mm_data.images:
            effective_patch = self.patch_size * self._spatial_merge_size
            image_nrows = []
            for img in mm_data.images:
                w, h = img.size
                ratio = max(w / self.image_size, h / self.image_size)
                if ratio > 1:
                    w = int(math.floor(w / ratio))
                    h = int(math.floor(h / ratio))
                nrows, _ = _get_pixtral_hf_num_image_tokens(
                    (h, w), (effective_patch, effective_patch)
                )
                image_nrows.append(nrows)

            mm_items, input_ids, _ = self.process_and_combine_mm_data(
                mm_data, self.mm_tokens
            )

            # For multi-image: split single IMAGE mm_item into per-image items
            if len(mm_data.images) > 1:
                from sglang.srt.managers.schedule_batch import MultimodalDataItem

                old_item = next(
                    item for item in mm_items if item.modality == Modality.IMAGE
                )
                all_offsets = old_item.offsets
                old_feature = old_item.feature
                old_image_sizes = getattr(old_item, "image_sizes", None)

                mm_items = [
                    item for item in mm_items if item.modality != Modality.IMAGE
                ]
                offset_idx = 0
                for i, img in enumerate(mm_data.images):
                    nr = image_nrows[i]
                    item_offsets = all_offsets[offset_idx : offset_idx + nr]
                    offset_idx += nr
                    new_item = MultimodalDataItem(modality=Modality.IMAGE)
                    new_item.feature = old_feature[i : i + 1]
                    new_item.offsets = item_offsets
                    if old_image_sizes is not None:
                        new_item.model_specific_data["image_sizes"] = old_image_sizes[
                            i : i + 1
                        ]
                    mm_items.append(new_item)
        else:
            mm_items, input_ids, _ = self.process_and_combine_mm_data(
                mm_data, self.mm_tokens
            )

        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            im_token_id=self.IM_TOKEN_ID,
        )
```
**EN:** This block defines async function `PixtralProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `PixtralProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `PixtralProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`, `typing`
- **Third-Party / 第三方**: `transformers`, `transformers.models.pixtral.image_processing_pixtral`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.pixtral`, `sglang.srt.multimodal.processors.base_processor`
