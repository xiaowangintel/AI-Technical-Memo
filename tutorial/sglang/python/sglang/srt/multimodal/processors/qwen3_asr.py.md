# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/qwen3_asr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for qwen3 asr models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 qwen3 asr 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Imports dependencies
```python
import re
from typing import Union

import torch

from sglang.srt.managers.schedule_batch import Modality, MultimodalProcessorOutput
from sglang.srt.models.qwen3_asr import Qwen3ASRForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including re, typing.Union, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 re, typing.Union, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 3 项，为后续代码准备所需名称。

### Lines 12-12: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-13: Declares AUDIO_PLACEHOLDER
```python
AUDIO_PLACEHOLDER = "<|audio_start|><|audio_pad|><|audio_end|>"
```
**EN:** This statement initializes AUDIO_PLACEHOLDER in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 AUDIO_PLACEHOLDER。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 14-14: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-20: Declares DEFAULT_ASR_PROMPT
```python
DEFAULT_ASR_PROMPT = (
    f"<|im_start|>user\n"
    f"{AUDIO_PLACEHOLDER}"
    f"<|im_end|>\n"
    f"<|im_start|>assistant\n"
)
```
**EN:** This statement initializes DEFAULT_ASR_PROMPT in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 DEFAULT_ASR_PROMPT。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 21-22: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares class Qwen3ASRMultimodalProcessor
```python
class Qwen3ASRMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `Qwen3ASRMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `Qwen3ASRMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 24-24: Declares models
```python
    models = [Qwen3ASRForConditionalGeneration]
```
**EN:** This statement initializes models in the Qwen3ASRMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen3ASRMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-43: Defines function Qwen3ASRMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.AUDIO_TOKEN = AUDIO_PLACEHOLDER
        self.AUDIO_TOKEN_REGEX = re.compile(
            r"<\|audio_start\|>(?:<\|audio_pad\|>)+<\|audio_end\|>"
        )
        tokenizer = self._processor.tokenizer
        self.audio_start_id = tokenizer.convert_tokens_to_ids("<|audio_start|>")
        self.audio_token_id = tokenizer.convert_tokens_to_ids("<|audio_pad|>")
        self.audio_end_id = tokenizer.convert_tokens_to_ids("<|audio_end|>")

        self.mm_tokens = MultimodalSpecialTokens(
            audio_token=self.AUDIO_TOKEN,
            audio_token_regex=self.AUDIO_TOKEN_REGEX,
            audio_token_id=self.audio_token_id,
        ).build(_processor)

        self.ATTR_NAME_TO_MODALITY.update({"feature_attention_mask": Modality.AUDIO})
```
**EN:** This block defines function `Qwen3ASRMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Qwen3ASRMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 44-44: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 45-51: Defines function Qwen3ASRMultimodalProcessor._build_transcription_prompt
```python
    def _build_transcription_prompt(self, input_text: Union[str, list]) -> str:
        # TODO: support `force_language`
        if isinstance(input_text, list):
            input_text = self._tokenizer.decode(input_text)
        if not input_text or not input_text.strip():
            return DEFAULT_ASR_PROMPT
        return input_text
```
**EN:** This block defines function `Qwen3ASRMultimodalProcessor._build_transcription_prompt`. Parameters: self, input_text.
**CN:** 该代码块定义函数 `Qwen3ASRMultimodalProcessor._build_transcription_prompt`。 参数包括 self、input_text。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-60: Defines function Qwen3ASRMultimodalProcessor.compute_mrope_positions
```python
    def compute_mrope_positions(self, input_ids, mm_items):
        if isinstance(input_ids, list):
            seq_len = len(input_ids)
        else:
            seq_len = input_ids.shape[-1] if input_ids.dim() > 1 else input_ids.shape[0]
        positions = torch.arange(seq_len, dtype=torch.long)
        mrope_positions = positions.unsqueeze(0).expand(3, -1).clone()
        return mrope_positions, torch.tensor([0], dtype=torch.long)
```
**EN:** This block defines function `Qwen3ASRMultimodalProcessor.compute_mrope_positions`. Parameters: self, input_ids, mm_items.
**CN:** 该代码块定义函数 `Qwen3ASRMultimodalProcessor.compute_mrope_positions`。 参数包括 self、input_ids、mm_items。

### Lines 61-61: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 62-98: Defines async function Qwen3ASRMultimodalProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        audio_data=None,
        input_text=None,
        request_obj=None,
        **kwargs,
    ):
        if not audio_data:
            return None

        prompt = self._build_transcription_prompt(input_text)

        base_output = self.load_mm_data(
            prompt=prompt,
            audio_data=audio_data,
            multimodal_tokens=self.mm_tokens,
        )
        if base_output is None:
            return None

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        mrope_positions, mrope_position_delta = self.compute_mrope_positions(
            input_ids, mm_items
        )

        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            audio_start_id=self.audio_start_id,
            audio_token_id=self.audio_token_id,
            audio_end_id=self.audio_end_id,
            mrope_positions=mrope_positions,
            mrope_position_delta=mrope_position_delta,
        )
```
**EN:** This block defines async function `Qwen3ASRMultimodalProcessor.process_mm_data_async`. Parameters: self, audio_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Qwen3ASRMultimodalProcessor.process_mm_data_async`。 参数包括 self、audio_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen3ASRMultimodalProcessor`
- **Constants / 常量**: `AUDIO_PLACEHOLDER`, `DEFAULT_ASR_PROMPT`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `re`, `typing`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.qwen3_asr`, `sglang.srt.multimodal.processors.base_processor`
