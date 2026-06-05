# qwen_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/qwen_audio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for qwen audio models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 qwen audio 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Imports dependencies
```python
import re

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.qwen2_audio import Qwen2AudioForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including re, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.qwen2_audio.Qwen2AudioForConditionalGeneration, and 2 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 re, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.qwen2_audio.Qwen2AudioForConditionalGeneration 等 2 项，为后续代码准备所需名称。

### Lines 13-14: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-15: Declares class Qwen2AudioMultimodalProcessor
```python
class Qwen2AudioMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `Qwen2AudioMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `Qwen2AudioMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 16-16: Declares models
```python
    models = [Qwen2AudioForConditionalGeneration]
```
**EN:** This statement initializes models in the Qwen2AudioMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen2AudioMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 17-17: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen2AudioMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen2AudioMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 18-36: Defines function Qwen2AudioMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.AUDIO_TOKEN = "<|audio_bos|><|AUDIO|><|audio_eos|>"
        self.AUDIO_TOKEN_REGEX = re.compile(
            r"<\|audio_bos\|>(?:<\|AUDIO\|>)+<\|audio_eos\|>"
        )
        # Collect special token ids
        tokenizer = self._processor.tokenizer
        self.audio_start_id = tokenizer.convert_tokens_to_ids("<|audio_bos|>")
        self.audio_token_id = tokenizer.convert_tokens_to_ids("<|AUDIO|>")
        self.audio_end_id = tokenizer.convert_tokens_to_ids("<|audio_eos|>")

        self.mm_tokens = MultimodalSpecialTokens(
            audio_token=self.AUDIO_TOKEN,
            audio_token_regex=self.AUDIO_TOKEN_REGEX,
            audio_token_id=self.audio_token_id,
        ).build(_processor)

        self.ATTR_NAME_TO_MODALITY.update({"feature_attention_mask": Modality.AUDIO})
```
**EN:** This block defines function `Qwen2AudioMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Qwen2AudioMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 37-37: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen2AudioMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen2AudioMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 38-82: Defines function Qwen2AudioMultimodalProcessor.get_mm_data
```python
    def get_mm_data(self, prompt, embeddings, **kwargs):
        audio_feature_lens = kwargs.get("audio_feature_lens", None)

        # Convert audio_feature_lens to token counts for build_input_ids
        output_lengths = None
        input_lengths = None
        if audio_feature_lens is not None:
            if audio_feature_lens.dim() > 1:
                audio_feature_lens = audio_feature_lens.flatten()
            input_lengths = (audio_feature_lens - 1) // 2 + 1
            output_lengths = (input_lengths - 2) // 2 + 1

        input_ids, offsets, modality_list = self.build_input_ids(
            prompt,
            audio_seq_lens=output_lengths,
        )

        mm_items = []
        consumed_per_modality = {}

        for modality, offset in zip(modality_list, offsets):
            num_tokens = offset[1] - offset[0] + 1
            embedding_start = consumed_per_modality.get(modality, 0)
            embedding_slice = embeddings[modality][
                embedding_start : embedding_start + num_tokens
            ]
            consumed_per_modality[modality] = embedding_start + num_tokens
            mm_items.append(
                MultimodalDataItem(
                    modality=modality,
                    offsets=[offset],
                    precomputed_embeddings=embedding_slice,
                )
            )

        if mm_items:
            mm_items[0].audio_feature_lens = output_lengths

        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids,
            audio_start_id=self.audio_start_id,
            audio_token_id=self.audio_token_id,
            audio_end_id=self.audio_end_id,
        )
```
**EN:** This block defines function `Qwen2AudioMultimodalProcessor.get_mm_data`. Parameters: self, prompt, embeddings.
**CN:** 该代码块定义函数 `Qwen2AudioMultimodalProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。

### Lines 83-83: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen2AudioMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen2AudioMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 84-117: Defines async function Qwen2AudioMultimodalProcessor.process_mm_data_async
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

        assert (
            "feature_attention_mask" in ret
        ), "feature_attention_mask not found in processor output"
        input_lengths = ret["feature_attention_mask"].sum(dim=-1)
        input_lengths = (input_lengths - 1) // 2 + 1
        output_lengths = (input_lengths - 2) // 2 + 1

        mm_items[0].audio_feature_lens = output_lengths

        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            audio_start_id=self.audio_start_id,
            audio_token_id=self.audio_token_id,
            audio_end_id=self.audio_end_id,
        )
```
**EN:** This block defines async function `Qwen2AudioMultimodalProcessor.process_mm_data_async`. Parameters: self, audio_data, input_text.
**CN:** 该代码块定义异步函数 `Qwen2AudioMultimodalProcessor.process_mm_data_async`。 参数包括 self、audio_data、input_text。

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen2AudioMultimodalProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `re`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.qwen2_audio`, `sglang.srt.multimodal.processors.base_processor`
