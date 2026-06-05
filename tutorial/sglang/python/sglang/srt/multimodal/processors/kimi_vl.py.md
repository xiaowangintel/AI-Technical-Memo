# kimi_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/kimi_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for kimi vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 kimi vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Imports dependencies
```python
import re
from typing import Dict, List, Union

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.kimi_vl import KimiVLForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
from sglang.srt.multimodal.processors.kimi_common import KimiGridMMDataMixin
```
**EN:** This block groups related imports for the module, including re, typing.Dict, typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 4 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 re, typing.Dict, typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 4 项，为后续代码准备所需名称。

### Lines 13-15: Comments and module notes
```python


# Compatible with KimiVLForConditionalGeneration
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 16-16: Declares class KimiVLImageProcessor
```python
class KimiVLImageProcessor(KimiGridMMDataMixin, SGLangBaseProcessor):
```
**EN:** This block introduces class `KimiVLImageProcessor` as a reusable abstraction inside the module. It inherits from KimiGridMMDataMixin, SGLangBaseProcessor.
**CN:** 该代码块声明类 `KimiVLImageProcessor`，作为模块中的可复用抽象。 它继承自 KimiGridMMDataMixin、SGLangBaseProcessor。

### Lines 17-18: Declares models, gpu_image_decode
```python
    models = [KimiVLForConditionalGeneration]
    gpu_image_decode = False  # KimiVL HF processor does not support tensor inputs
```
**EN:** This block initializes a related set of values in the KimiVLImageProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 KimiVLImageProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 19-19: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-27: Defines function KimiVLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.mm_tokens = MultimodalSpecialTokens(
            image_token="<|media_pad|>",
            # TODO: could we convert in MultimodalSpecialTokens?
            image_token_id=hf_config.media_placeholder_token_id,
            image_token_regex=re.compile(r"(?:<\|media_pad\|>)+"),
        ).build(_processor)
```
**EN:** This block defines function `KimiVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `KimiVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 28-28: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 29-51: Defines async function KimiVLImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes, Dict]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, _ = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
        )
```
**EN:** This block defines async function `KimiVLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `KimiVLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-60: Defines function KimiVLImageProcessor.get_mm_data
```python
    def get_mm_data(self, prompt, embeddings, **kwargs):
        img_grid_thw = kwargs.get("img_grid_thw", None)
        return self._build_kimi_mm_data_from_grids(
            prompt=prompt,
            embeddings=embeddings,
            image_token_id=self.mm_tokens.image_token_id,
            img_grid_thw=img_grid_thw,
        )
```
**EN:** This block defines function `KimiVLImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings.
**CN:** 该代码块定义函数 `KimiVLImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。

## Key Concepts / 关键概念
- **Classes / 类**: `KimiVLImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `re`, `typing`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.kimi_vl`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.multimodal.processors.kimi_common`
