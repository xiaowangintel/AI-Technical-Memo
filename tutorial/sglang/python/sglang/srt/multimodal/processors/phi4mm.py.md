# phi4mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/phi4mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for phi4mm models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 phi4mm 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Imports dependencies
```python
import logging
from typing import List, Union

from transformers.processing_utils import ProcessorMixin

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.phi4mm import Phi4MMForCausalLM
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including logging, typing.List, typing.Union, transformers.processing_utils.ProcessorMixin, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, typing.List, typing.Union, transformers.processing_utils.ProcessorMixin, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 3 项，为后续代码准备所需名称。

### Lines 12-12: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 13-13: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 14-17: Comments and module notes
```python


# It is an adapter of hf phi4 mm processor to make it work for sglang
# Ref: https://huggingface.co/microsoft/Phi-4-multimodal-instruct/blob/main/processing_phi4mm.py#L693
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 18-18: Declares class Phi4MMProcessorAdapter
```python
class Phi4MMProcessorAdapter(ProcessorMixin):
```
**EN:** This block introduces class `Phi4MMProcessorAdapter` as a reusable abstraction inside the module. It inherits from ProcessorMixin.
**CN:** 该代码块声明类 `Phi4MMProcessorAdapter`，作为模块中的可复用抽象。 它继承自 ProcessorMixin。

### Lines 19-20: Defines function Phi4MMProcessorAdapter.__init__
```python
    def __init__(self, _processor) -> None:
        self._processor = _processor
```
**EN:** This block defines function `Phi4MMProcessorAdapter.__init__`. Parameters: self, _processor.
**CN:** 该代码块定义函数 `Phi4MMProcessorAdapter.__init__`。 参数包括 self、_processor。

### Lines 21-21: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Phi4MMProcessorAdapter, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Phi4MMProcessorAdapter 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 22-44: Defines function Phi4MMProcessorAdapter.__call__
```python
    def __call__(self, **kwargs):
        result = self._processor(**kwargs)

        # Map HuggingFace output keys to sglang standard keys
        key_mapping = {
            "input_image_embeds": "pixel_values",
            "input_audio_embeds": "audio_features",
            "audio_embed_sizes": "audio_feature_lens",
        }
        for hf_key, sglang_key in key_mapping.items():
            if hf_key in result:
                result[sglang_key] = result[hf_key]
                del result[hf_key]

        # Filter out None or empty tensors from the result.
        # This prevents the sglang function base_processor.collect_mm_items_from_processor_output()
        # from misclassifying audio content as image content, and vice versa.
        filtered_result = {
            k: v
            for k, v in result.items()
            if v is not None and (not hasattr(v, "numel") or v.numel() > 0)
        }
        return filtered_result
```
**EN:** This block defines function `Phi4MMProcessorAdapter.__call__`. Parameters: self.
**CN:** 该代码块定义函数 `Phi4MMProcessorAdapter.__call__`。 参数包括 self。

### Lines 45-46: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-47: Declares class Phi4MMMultimodalProcessor
```python
class Phi4MMMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `Phi4MMMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `Phi4MMMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 48-48: Declares models
```python
    models = [Phi4MMForCausalLM]
```
**EN:** This statement initializes models in the Phi4MMMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Phi4MMMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 49-49: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Phi4MMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Phi4MMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 50-67: Defines function Phi4MMMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        self.processor = Phi4MMProcessorAdapter(_processor)
        super().__init__(hf_config, server_args, self.processor, *args, **kwargs)

        # the following CONSTANTS come from hugging-face microsoft/Phi-4-multimodal-instruct's processing_phi4mm.py file
        # ref: https://huggingface.co/microsoft/Phi-4-multimodal-instruct/blob/main/processing_phi4mm.py
        self.IMAGE_TOKEN = "<|endoftext10|>"
        self.AUDIO_TOKEN = "<|endoftext11|>"
        self.IM_TOKEN_ID = 200010
        self.AUDIO_TOKEN_ID = 200011
        self.AUDIO_SAMPLE_RATE = 16000

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMAGE_TOKEN,
            image_token_id=self.IM_TOKEN_ID,
            audio_token=self.AUDIO_TOKEN,
            audio_token_id=self.AUDIO_TOKEN_ID,
        ).build(self.processor)
```
**EN:** This block defines function `Phi4MMMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Phi4MMMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 68-68: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Phi4MMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Phi4MMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 69-101: Defines async function Phi4MMMultimodalProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        audio_data,
        input_text,
        request_obj,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            audio_data=audio_data,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
            audio_sample_rate=self.AUDIO_SAMPLE_RATE,
        )

        if base_output.audios is not None:
            # hugging-face microsoft/Phi-4-multimodal-instruct's processing_phi4mm.py file requires the audio input to be tuple of (audio, sample_rate)
            # ref: https://huggingface.co/microsoft/Phi-4-multimodal-instruct/blob/main/processing_phi4mm.py
            base_output.audios = [
                (audio, self.AUDIO_SAMPLE_RATE) for audio in base_output.audios
            ]

        mm_items, input_ids, _ = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
        )
```
**EN:** This block defines async function `Phi4MMMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Phi4MMMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `Phi4MMProcessorAdapter`, `Phi4MMMultimodalProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `logging`, `typing`
- **Third-Party / 第三方**: `transformers.processing_utils`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.phi4mm`, `sglang.srt.multimodal.processors.base_processor`
