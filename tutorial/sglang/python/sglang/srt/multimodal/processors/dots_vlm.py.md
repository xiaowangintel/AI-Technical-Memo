# dots_vlm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/dots_vlm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for dots vlm models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 dots vlm 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10: Imports dependencies
```python
import re
from typing import Dict, List, Union

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.dots_ocr import DotsOCRForCausalLM
from sglang.srt.models.dots_vlm import DotsVLMForCausalLM
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including re, typing.Dict, typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 4 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 re, typing.Dict, typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 4 项，为后续代码准备所需名称。

### Lines 11-12: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-13: Declares class DotsVLMImageProcessor
```python
class DotsVLMImageProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `DotsVLMImageProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `DotsVLMImageProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 14-14: Declares models
```python
    models = [DotsVLMForCausalLM, DotsOCRForCausalLM]
```
**EN:** This statement initializes models in the DotsVLMImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 DotsVLMImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 15-15: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DotsVLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DotsVLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 16-43: Defines function DotsVLMImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        # The single, pre-expanded image token.
        self.IMAGE_TOKEN = "<|img|><|imgpad|><|endofimg|>"
        # The regex that matches expanded image tokens.
        self.IMAGE_TOKEN_REGEX = re.compile(r"<\|img\|>(?:<\|imgpad\|>)+<\|endofimg\|>")

        assert len(_processor.tokenizer.encode("<|img|>")) == 1
        self.im_start_id = _processor.tokenizer.encode("<|img|>")[0]
        self.im_end_id = _processor.tokenizer.encode("<|endofimg|>")[0]
        self.image_token_id = _processor.tokenizer.encode("<|imgpad|>")[0]
        self.IM_TOKEN_ID = self.image_token_id
        self.IM_START_TOKEN_ID = self.im_start_id
        self.IM_END_TOKEN_ID = self.im_end_id

        vision_config = hf_config.vision_config
        patch_size = vision_config.patch_size
        merge_size = vision_config.spatial_merge_size

        self.IMAGE_FACTOR = patch_size * merge_size
        self.MIN_PIXELS = _processor.image_processor.min_pixels
        self.MAX_PIXELS = _processor.image_processor.max_pixels
        self.MAX_RATIO = 200
        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMAGE_TOKEN,
            image_token_id=self.image_token_id,
            image_token_regex=self.IMAGE_TOKEN_REGEX,
        ).build(_processor)
```
**EN:** This block defines function `DotsVLMImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `DotsVLMImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 44-44: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DotsVLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DotsVLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 45-82: Defines async function DotsVLMImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes, Dict]],
        input_text,
        request_obj,
        max_req_input_len,
        *args,
        **kwargs,
    ):
        if isinstance(image_data, str):
            image_data = [image_data]

        if (
            isinstance(image_data, list)
            and image_data
            and isinstance(image_data[0], list)
        ):
            image_data = sum(image_data, [])

        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )

        combined_mm_item, input_ids, _ = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )
        if combined_mm_item is None:
            return None

        return MultimodalProcessorOutput(
            mm_items=combined_mm_item,
            input_ids=input_ids.tolist(),
            im_token_id=self.image_token_id,
            im_start_id=self.im_start_id,
            im_end_id=self.im_end_id,
        )
```
**EN:** This block defines async function `DotsVLMImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj, max_req_input_len.
**CN:** 该代码块定义异步函数 `DotsVLMImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj、max_req_input_len。

## Key Concepts / 关键概念
- **Classes / 类**: `DotsVLMImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `re`, `typing`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.dots_ocr`, `sglang.srt.models.dots_vlm`, `sglang.srt.multimodal.processors.base_processor`
