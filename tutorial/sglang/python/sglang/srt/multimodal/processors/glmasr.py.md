# glmasr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/glmasr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for glmasr models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 glmasr 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Imports dependencies
```python
import re

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.glmasr import GlmAsrForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including re, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.glmasr.GlmAsrForConditionalGeneration, sglang.srt.multimodal.processors.base_processor.BaseMultimodalProcessor, sglang.srt.multimodal.processors.base_processor.MultimodalSpecialTokens. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 re, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.glmasr.GlmAsrForConditionalGeneration, sglang.srt.multimodal.processors.base_processor.BaseMultimodalProcessor, sglang.srt.multimodal.processors.base_processor.MultimodalSpecialTokens，为后续代码准备所需名称。

### Lines 9-10: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 11-11: Declares class GlmAsrProcessor
```python
class GlmAsrProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `GlmAsrProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `GlmAsrProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 12-12: Declares models
```python
    models = [GlmAsrForConditionalGeneration]
```
**EN:** This statement initializes models in the GlmAsrProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 GlmAsrProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 13-13: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GlmAsrProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GlmAsrProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 14-30: Defines function GlmAsrProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.AUDIO_TOKEN = "<|begin_of_audio|><|pad|><|end_of_audio|>"
        self.AUDIO_TOKEN_REGEX = re.compile(
            r"<\|begin_of_audio\|><\|pad\|><\|end_of_audio\|>"
        )
        # Collect special token ids
        tokenizer = self._processor.tokenizer
        self.audio_start_id = tokenizer.convert_tokens_to_ids("<|begin_of_audio|>")
        self.audio_token_id = tokenizer.convert_tokens_to_ids("<|pad|>")
        self.audio_end_id = tokenizer.convert_tokens_to_ids("<|end_of_audio|>")

        self.mm_tokens = MultimodalSpecialTokens(
            audio_token=self.AUDIO_TOKEN,
            audio_token_regex=self.AUDIO_TOKEN_REGEX,
            audio_token_id=self.audio_token_id,
        ).build(_processor)
```
**EN:** This block defines function `GlmAsrProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `GlmAsrProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 31-31: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GlmAsrProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GlmAsrProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-54: Defines async function GlmAsrProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        audio_data,
        input_text,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            audio_data=audio_data,
            multimodal_tokens=self.mm_tokens,
        )
        if base_output is None:
            return None
        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )
        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            audio_start_id=self.audio_start_id,
            audio_token_id=self.audio_token_id,
            audio_end_id=self.audio_end_id,
        )
```
**EN:** This block defines async function `GlmAsrProcessor.process_mm_data_async`. Parameters: self, audio_data, input_text.
**CN:** 该代码块定义异步函数 `GlmAsrProcessor.process_mm_data_async`。 参数包括 self、audio_data、input_text。

## Key Concepts / 关键概念
- **Classes / 类**: `GlmAsrProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `re`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.glmasr`, `sglang.srt.multimodal.processors.base_processor`
