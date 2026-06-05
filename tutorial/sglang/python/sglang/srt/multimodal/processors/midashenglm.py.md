# midashenglm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/midashenglm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for midashenglm models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 midashenglm 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Imports dependencies
```python
import logging
import re

import torch

from sglang.srt.managers.schedule_batch import Modality, MultimodalProcessorOutput
from sglang.srt.models.midashenglm import MiDashengLMModel
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including logging, re, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, re, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput 等 3 项，为后续代码准备所需名称。

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

### Lines 14-15: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 16-16: Declares class MiDashengLMMultimodalProcessor
```python
class MiDashengLMMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `MiDashengLMMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor. Multimodal processor for MiDashengLM audio-language model.
**CN:** 该代码块声明类 `MiDashengLMMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。 文档字符串摘要：Multimodal processor for MiDashengLM audio-language model.

### Lines 17-17: Documents the scope
```python
    """Multimodal processor for MiDashengLM audio-language model."""
```
**EN:** This string literal serves as documentation for the MiDashengLMMultimodalProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 MiDashengLMMultimodalProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 18-18: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiDashengLMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiDashengLMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 19-19: Declares models
```python
    models = [MiDashengLMModel]
```
**EN:** This statement initializes models in the MiDashengLMMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 MiDashengLMMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 20-20: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiDashengLMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiDashengLMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 21-48: Defines function MiDashengLMMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.AUDIO_TOKEN = "<|audio_bos|><|AUDIO|><|audio_eos|>"
        self.AUDIO_TOKEN_REGEX = re.compile(
            r"<\|audio_bos\|>(?:<\|AUDIO\|>)+<\|audio_eos\|>"
        )

        tokenizer = self._processor.tokenizer
        self.audio_start_id = tokenizer.convert_tokens_to_ids("<|audio_bos|>")
        self.audio_token_id = tokenizer.convert_tokens_to_ids("<|AUDIO|>")
        self.audio_end_id = tokenizer.convert_tokens_to_ids("<|audio_eos|>")

        self.mm_tokens = MultimodalSpecialTokens(
            audio_token=self.AUDIO_TOKEN,
            audio_token_regex=self.AUDIO_TOKEN_REGEX,
            audio_token_id=self.audio_token_id,
        ).build(_processor)

        self.ATTR_NAME_TO_MODALITY.update(
            {
                "input_values": Modality.AUDIO,
                "audio_length": Modality.AUDIO,
            }
        )

        if "input_values" not in self.FEATURE_NAMES:
            self.FEATURE_NAMES.append("input_values")
```
**EN:** This block defines function `MiDashengLMMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `MiDashengLMMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 49-49: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiDashengLMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiDashengLMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 50-78: Defines function MiDashengLMMultimodalProcessor.process_mm_data
```python
    def process_mm_data(
        self, input_text, images=None, videos=None, audios=None, **kwargs
    ):
        """Override to use correct audio parameter name for MiDashengLM processor."""
        if images:
            kwargs["images"] = images
        if videos:
            kwargs["videos"] = videos
        if audios:
            kwargs["audio"] = audios
            kwargs.setdefault("audio_kwargs", {})
            kwargs["audio_kwargs"].setdefault("truncation", False)
            if self.audio_config:
                kwargs["audio_kwargs"].update(self.audio_config)

        processor = self._processor
        result = processor.__call__(
            text=[input_text],
            padding=True,
            return_tensors="pt",
            **kwargs,
        )

        if not getattr(self.server_args, "keep_mm_feature_on_device", False):
            for feature_name in ["input_values"]:
                if feature_name in result:
                    result[feature_name] = result[feature_name].cpu()

        return result
```
**EN:** This block defines function `MiDashengLMMultimodalProcessor.process_mm_data`. Parameters: self, input_text, images, videos, audios. Override to use correct audio parameter name for MiDashengLM processor.
**CN:** 该代码块定义函数 `MiDashengLMMultimodalProcessor.process_mm_data`。 参数包括 self、input_text、images、videos、audios。 文档字符串摘要：Override to use correct audio parameter name for MiDashengLM processor.

### Lines 79-79: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiDashengLMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiDashengLMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 80-157: Defines async function MiDashengLMMultimodalProcessor.process_mm_data_async (part 1)
```python
    async def process_mm_data_async(
        self,
        audio_data,
        input_text,
        **kwargs,
    ):
        """Process audio data for MiDashengLM model.

        Args:
            audio_data: Audio input data
            input_text: Text prompt
            **kwargs: Additional arguments

        Returns:
            Dictionary containing processed multimodal data
        """
        logger.info("=" * 80)
        logger.info("process_mm_data_async called")
        logger.info(f"audio_data is not None: {audio_data is not None}")
        logger.info(f"input_text: {input_text}")
        logger.info("=" * 80)

        if audio_data and not self.AUDIO_TOKEN_REGEX.search(input_text):
            input_text = f"{self.AUDIO_TOKEN}{input_text}"
            logger.info("Auto-prepended audio token")

        base_output = self.load_mm_data(
            prompt=input_text,
            audio_data=audio_data,
            multimodal_tokens=self.mm_tokens,
        )
        if base_output is None:
            logger.info("base_output is None")
            return None

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )
        logger.info(f"mm_items count: {len(mm_items)}")
        logger.info(f"ret keys: {list(ret.keys())}")
        logger.info(f"input_ids shape: {input_ids.shape}")
        logger.info(
            f"audio_token_id={self.audio_token_id}, audio_start_id={self.audio_start_id}, audio_end_id={self.audio_end_id}"
        )
        logger.info(
            f"Count of audio_token_id in input_ids: {(input_ids == self.audio_token_id).sum().item()}"
        )
        for i, item in enumerate(mm_items):
            logger.info(f"mm_item[{i}] modality: {item.modality}")
            logger.info(
                f"mm_item[{i}] pad_value: {getattr(item, 'pad_value', 'NOT SET')}"
            )
            logger.info(f"mm_item[{i}] offsets: {getattr(item, 'offsets', 'NOT SET')}")
            logger.info(f"mm_item[{i}] has feature: {hasattr(item, 'feature')}")
            if hasattr(item, "feature") and item.feature is not None:
                logger.info(f"mm_item[{i}] feature shape: {item.feature.shape}")

        if "audio_length" in ret and len(mm_items) > 0:
            audio_length = ret["audio_length"]
            if isinstance(audio_length, torch.Tensor):
                audio_length = (
                    audio_length.item()
                    if audio_length.numel() == 1
                    else audio_length[0].item()
                )
            mm_items[0].audio_length = audio_length
            logger.info(
                f"Set audio_length={audio_length} (from processor, mel frame count)"
            )
        elif "input_values" in ret and len(mm_items) > 0:
            input_values = ret["input_values"]
            audio_length = (
                input_values.shape[-1]
                if input_values.ndim >= 2
                else input_values.shape[0]
            )
            mm_items[0].audio_length = audio_length
            logger.info(f"Set audio_length={audio_length} (fallback, waveform length)")
```
**EN:** This block defines async function `MiDashengLMMultimodalProcessor.process_mm_data_async`. Parameters: self, audio_data, input_text. Process audio data for MiDashengLM model. Args: audio_data: Audio input data input_text: Text prompt **kwargs: Additional arguments Returns: Dictionary containing processed multimodal data This subsection covers lines 80-157 of the same logical block.
**CN:** 该代码块定义异步函数 `MiDashengLMMultimodalProcessor.process_mm_data_async`。 参数包括 self、audio_data、input_text。 文档字符串摘要：Process audio data for MiDashengLM model. Args: audio_data: Audio input data input_text: Text prompt **kwargs: Additional arguments Returns: Dictionary containing processed multimodal data 本小节覆盖同一逻辑块中的第 80-157 行。

### Lines 158-158: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 159-167: Defines async function MiDashengLMMultimodalProcessor.process_mm_data_async (part 2)
```python
        result = MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids.tolist(),
            audio_start_id=self.audio_start_id,
            audio_token_id=self.audio_token_id,
            audio_end_id=self.audio_end_id,
        )
        logger.info(f"Returning {len(result.mm_items)} mm_items")
        return result
```
**EN:** This block defines async function `MiDashengLMMultimodalProcessor.process_mm_data_async`. Parameters: self, audio_data, input_text. Process audio data for MiDashengLM model. Args: audio_data: Audio input data input_text: Text prompt **kwargs: Additional arguments Returns: Dictionary containing processed multimodal data This subsection covers lines 159-167 of the same logical block.
**CN:** 该代码块定义异步函数 `MiDashengLMMultimodalProcessor.process_mm_data_async`。 参数包括 self、audio_data、input_text。 文档字符串摘要：Process audio data for MiDashengLM model. Args: audio_data: Audio input data input_text: Text prompt **kwargs: Additional arguments Returns: Dictionary containing processed multimodal data 本小节覆盖同一逻辑块中的第 159-167 行。

## Key Concepts / 关键概念
- **Classes / 类**: `MiDashengLMMultimodalProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `logging`, `re`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.midashenglm`, `sglang.srt.multimodal.processors.base_processor`
