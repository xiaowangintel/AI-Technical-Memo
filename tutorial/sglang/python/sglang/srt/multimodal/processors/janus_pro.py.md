# janus_pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/janus_pro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for janus pro models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 janus pro 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Imports dependencies
```python
from typing import List, Union

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.deepseek_janus_pro import MultiModalityCausalLM
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.deepseek_janus_pro.MultiModalityCausalLM, sglang.srt.multimodal.processors.base_processor.BaseMultimodalProcessor, and 1 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Union, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.deepseek_janus_pro.MultiModalityCausalLM, sglang.srt.multimodal.processors.base_processor.BaseMultimodalProcessor 等 1 项，为后续代码准备所需名称。

### Lines 9-10: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 11-11: Declares class JanusProImageProcessor
```python
class JanusProImageProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `JanusProImageProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `JanusProImageProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 12-12: Declares models
```python
    models = [MultiModalityCausalLM]
```
**EN:** This statement initializes models in the JanusProImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 JanusProImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 13-13: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JanusProImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JanusProImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 14-20: Defines function JanusProImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=_processor.image_token,
            image_token_id=_processor.image_id,
        ).build(_processor)
```
**EN:** This block defines function `JanusProImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `JanusProImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 21-21: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the JanusProImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 JanusProImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 22-45: Defines async function JanusProImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        **kwargs,
    ):
        base_out = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, _ = self.process_and_combine_mm_data(
            base_out, self.mm_tokens, prompt=base_out.input_text
        )

        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            im_start_id=self._processor.image_start_id,
            im_end_id=self._processor.image_end_id,
            im_token_id=self.mm_tokens.image_token_id,
        )
```
**EN:** This block defines async function `JanusProImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `JanusProImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `JanusProImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.deepseek_janus_pro`, `sglang.srt.multimodal.processors.base_processor`
