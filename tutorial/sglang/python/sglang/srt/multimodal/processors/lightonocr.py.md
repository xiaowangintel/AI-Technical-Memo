# lightonocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/lightonocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for lightonocr models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 lightonocr 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
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
# ==============================================================================

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 15-22: Documents the scope
```python
"""
Multimodal processor for lightonai/LightOnOCR-2-1B.

Key difference from Pixtral: LightOnOCR does NOT use image break/end tokens.
The parent PixtralProcessor inserts row-break and image-end tokens between
image patch rows. This processor removes them after the parent processing
to produce a single contiguous range of image tokens per image.
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-27: Imports dependencies
```python
from typing import List, Union

from sglang.srt.models.lightonocr import LightOnOCRForConditionalGeneration
from sglang.srt.multimodal.processors.pixtral import PixtralProcessor
```
**EN:** This block groups related imports for the module, including typing.List, typing.Union, sglang.srt.models.lightonocr.LightOnOCRForConditionalGeneration, sglang.srt.multimodal.processors.pixtral.PixtralProcessor. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Union, sglang.srt.models.lightonocr.LightOnOCRForConditionalGeneration, sglang.srt.multimodal.processors.pixtral.PixtralProcessor，为后续代码准备所需名称。

### Lines 28-29: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 30-30: Declares class LightOnOCRProcessor
```python
class LightOnOCRProcessor(PixtralProcessor):
```
**EN:** This block introduces class `LightOnOCRProcessor` as a reusable abstraction inside the module. It inherits from PixtralProcessor. Processor for LightOnOCR model.
**CN:** 该代码块声明类 `LightOnOCRProcessor`，作为模块中的可复用抽象。 它继承自 PixtralProcessor。 文档字符串摘要：Processor for LightOnOCR model.

### Lines 31-31: Documents the scope
```python
    """Processor for LightOnOCR model."""
```
**EN:** This string literal serves as documentation for the LightOnOCRProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 LightOnOCRProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 32-32: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LightOnOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LightOnOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 33-33: Declares models
```python
    models = [LightOnOCRForConditionalGeneration]
```
**EN:** This statement initializes models in the LightOnOCRProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 LightOnOCRProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 34-34: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LightOnOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LightOnOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 35-61: Defines function LightOnOCRProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        # LightOnOCR uses image_token_id instead of image_token_index
        if not hasattr(hf_config, "image_token_index"):
            hf_config.image_token_index = getattr(hf_config, "image_token_id", 151655)

        # Propagate spatial_merge_size from root config to vision_config
        spatial_merge_size = getattr(hf_config, "spatial_merge_size", 2)
        if hasattr(hf_config, "vision_config"):
            vc = hf_config.vision_config
            if not hasattr(vc, "spatial_merge_size") or vc.spatial_merge_size is None:
                vc.spatial_merge_size = spatial_merge_size

        if hasattr(_processor, "patch_size"):
            _processor.spatial_merge_size = spatial_merge_size

        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        # Identify break/end token IDs for removal
        self._break_token_ids = set()
        for attr in ("image_break_token_id", "image_break_id"):
            tid = getattr(_processor, attr, None)
            if tid is not None:
                self._break_token_ids.add(tid)
        for attr in ("image_end_token_id", "image_end_id"):
            tid = getattr(_processor, attr, None)
            if tid is not None:
                self._break_token_ids.add(tid)
```
**EN:** This block defines function `LightOnOCRProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `LightOnOCRProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LightOnOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LightOnOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-110: Defines async function LightOnOCRProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        result = await super().process_mm_data_async(
            image_data=image_data,
            input_text=input_text,
            request_obj=request_obj,
            *args,
            **kwargs,
        )

        if not result or not self._break_token_ids:
            return result

        # Remove break/end tokens and fix multimodal item offsets
        input_ids = result.input_ids or []
        mm_items = result.mm_items or []

        new_input_ids = []
        old_to_new = {}
        for old_idx, token_id in enumerate(input_ids):
            if token_id not in self._break_token_ids:
                old_to_new[old_idx] = len(new_input_ids)
                new_input_ids.append(token_id)

        if len(new_input_ids) == len(input_ids):
            return result

        # Remap multimodal item offsets to account for removed tokens
        for mm_item in mm_items:
            if not mm_item.offsets:
                continue
            new_indices = sorted(
                old_to_new[idx]
                for start, end in mm_item.offsets
                for idx in range(start, end + 1)
                if idx in old_to_new
            )
            if new_indices:
                mm_item.offsets = [(new_indices[0], new_indices[-1])]

        result.input_ids = new_input_ids
        return result
```
**EN:** This block defines async function `LightOnOCRProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `LightOnOCRProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `LightOnOCRProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Local Modules / 本地模块**: `sglang.srt.models.lightonocr`, `sglang.srt.multimodal.processors.pixtral`
