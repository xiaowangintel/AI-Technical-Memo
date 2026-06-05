# lfm2_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/lfm2_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for lfm2 vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 lfm2 vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Comments and module notes
```python
# Copyright 2026 Liquid AI. All rights reserved.
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

### Lines 13-13: Documents the scope
```python
"""Multimodal processor for LFM2-VL models with SigLip2 NaFlex support."""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 14-14: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-24: Imports dependencies
```python
from typing import List, Union

from sglang.srt.managers.schedule_batch import Modality, MultimodalProcessorOutput
from sglang.srt.models.lfm2_vl import Lfm2VlForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including typing.List, typing.Union, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.lfm2_vl.Lfm2VlForConditionalGeneration, and 2 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Union, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.lfm2_vl.Lfm2VlForConditionalGeneration 等 2 项，为后续代码准备所需名称。

### Lines 25-26: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 27-27: Declares class Lfm2VlImageProcessor
```python
class Lfm2VlImageProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `Lfm2VlImageProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor. Multimodal processor for LFM2-VL vision-language models. Uses the base class load_mm_data + process_and_combine_mm_data flow.
**CN:** 该代码块声明类 `Lfm2VlImageProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。 文档字符串摘要：Multimodal processor for LFM2-VL vision-language models. Uses the base class load_mm_data + process_and_combine_mm_data flow.

### Lines 28-32: Documents the scope
```python
    """Multimodal processor for LFM2-VL vision-language models.

    Uses the base class load_mm_data + process_and_combine_mm_data flow.
    The HF processor handles NaFlex variable-resolution tiling internally.
    """
```
**EN:** This string literal serves as documentation for the Lfm2VlImageProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Lfm2VlImageProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 33-33: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 34-35: Declares models, gpu_image_decode
```python
    models = [Lfm2VlForConditionalGeneration]
    gpu_image_decode = False
```
**EN:** This block initializes a related set of values in the Lfm2VlImageProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Lfm2VlImageProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-51: Defines function Lfm2VlImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.IMAGE_TOKEN_ID = hf_config.image_token_id
        self.IMAGE_TOKEN = "<image>"

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMAGE_TOKEN,
            image_token_id=hf_config.image_token_id,
        ).build(_processor)

        # Register NaFlex-specific HF processor outputs so
        # collect_mm_items_from_processor_output picks them up
        self.ATTR_NAME_TO_MODALITY["pixel_attention_mask"] = Modality.IMAGE
        self.ATTR_NAME_TO_MODALITY["spatial_shapes"] = Modality.IMAGE
```
**EN:** This block defines function `Lfm2VlImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Lfm2VlImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Lfm2VlImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Lfm2VlImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-85: Defines async function Lfm2VlImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        audio_data,
        input_text: str,
        request_obj,
        **kwargs,
    ):
        if not image_data:
            input_ids = self._tokenizer(
                input_text, return_tensors="pt", add_special_tokens=False
            ).input_ids
            return {
                "input_ids": input_ids.squeeze(0).tolist(),
                "mm_items": [],
                "im_token_id": self.IMAGE_TOKEN_ID,
            }

        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.IMAGE_TOKEN_ID,
        )
```
**EN:** This block defines async function `Lfm2VlImageProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Lfm2VlImageProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `Lfm2VlImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.lfm2_vl`, `sglang.srt.multimodal.processors.base_processor`
