# voxtral.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/voxtral.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for voxtral models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 voxtral 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Documents the scope
```python
"""Multimodal processor for Voxtral (speech-to-text) models."""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 2-2: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 3-18: Imports dependencies
```python
import math
import re
from typing import Dict, List, Optional

import torch

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.voxtral import VoxtralForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including math, re, typing.Dict, typing.List, typing.Optional, and 7 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, re, typing.Dict, typing.List, typing.Optional 等 7 项，为后续代码准备所需名称。

### Lines 19-20: Comments and module notes
```python

# Special token IDs for Voxtral audio (from tekken.json vocabulary)
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 21-23: Declares AUDIO_TOKEN_ID, BEGIN_AUDIO_TOKEN_ID, INST_TOKEN_ID
```python
AUDIO_TOKEN_ID = 24  # [AUDIO]
BEGIN_AUDIO_TOKEN_ID = 25  # [BEGIN_AUDIO]
INST_TOKEN_ID = 3  # [INST]
```
**EN:** This block initializes a related set of values in the module, including AUDIO_TOKEN_ID, BEGIN_AUDIO_TOKEN_ID, INST_TOKEN_ID. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 AUDIO_TOKEN_ID, BEGIN_AUDIO_TOKEN_ID, INST_TOKEN_ID。将这些赋值集中在一起有助于理解周边配置。

### Lines 24-27: Comments and module notes
```python

# Placeholder for load_mm_data regex matching.
# encode("[AUDIO]") does NOT produce token 24; actual token insertion
# is handled in _build_input_ids_with_audio.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 28-29: Declares AUDIO_PLACEHOLDER, AUDIO_PLACEHOLDER_REGEX
```python
AUDIO_PLACEHOLDER = "[AUDIO]"
AUDIO_PLACEHOLDER_REGEX = re.compile(r"\[AUDIO\]")
```
**EN:** This block initializes a related set of values in the module, including AUDIO_PLACEHOLDER, AUDIO_PLACEHOLDER_REGEX. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 AUDIO_PLACEHOLDER, AUDIO_PLACEHOLDER_REGEX。将这些赋值集中在一起有助于理解周边配置。

### Lines 30-31: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-32: Declares class VoxtralMultimodalProcessor
```python
class VoxtralMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `VoxtralMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `VoxtralMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 33-33: Declares models
```python
    models = [VoxtralForConditionalGeneration]
```
**EN:** This statement initializes models in the VoxtralMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 VoxtralMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 34-34: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 35-54: Defines function VoxtralMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        audio_config = getattr(hf_config, "audio_config", None)
        self.audio_token_id = getattr(hf_config, "audio_token_id", AUDIO_TOKEN_ID)
        self.sampling_rate = getattr(audio_config, "sampling_rate", 16000)
        self.hop_length = getattr(audio_config, "hop_length", 160)
        self.max_source_positions = getattr(audio_config, "max_source_positions", 1500)
        self.conv_downsample = 2  # conv1 stride=1 * conv2 stride=2
        self.downsample_factor = getattr(
            audio_config,
            "downsample_factor",
            getattr(audio_config, "intermediate_size", 5120)
            // getattr(audio_config, "hidden_size", 1280),
        )

        self.mm_tokens = MultimodalSpecialTokens(
            audio_token=AUDIO_PLACEHOLDER,
            audio_token_regex=AUDIO_PLACEHOLDER_REGEX,
            audio_token_id=self.audio_token_id,
        ).build(_processor)
```
**EN:** This block defines function `VoxtralMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 55-55: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 56-62: Defines function VoxtralMultimodalProcessor._compute_audio_token_count
```python
    def _compute_audio_token_count(self, n_samples: int) -> int:
        """Compute the number of [AUDIO] tokens for a given audio length."""
        mel_frames = n_samples / self.hop_length
        chunk_size = self.max_source_positions * self.conv_downsample
        n_chunks = math.ceil(mel_frames / chunk_size) if mel_frames > 0 else 1
        tokens_per_chunk = self.max_source_positions // self.downsample_factor
        return n_chunks * tokens_per_chunk
```
**EN:** This block defines function `VoxtralMultimodalProcessor._compute_audio_token_count`. Parameters: self, n_samples. Compute the number of [AUDIO] tokens for a given audio length.
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor._compute_audio_token_count`。 参数包括 self、n_samples。 文档字符串摘要：Compute the number of [AUDIO] tokens for a given audio length.

### Lines 63-63: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 64-122: Defines async function VoxtralMultimodalProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data,
        audio_data,
        input_text,
        request_obj,
        **kwargs,
    ) -> Optional[MultimodalProcessorOutput]:
        if not audio_data:
            return None

        # Insert [AUDIO] placeholders into prompt for load_mm_data's regex
        prompt_with_placeholders = self._insert_audio_placeholders(
            input_text, len(audio_data)
        )

        # load_mm_data handles async loading, format detection, resampling.
        # process_and_combine_mm_data cannot be used: HF VoxtralProcessor.__call__
        # does not support audio (only apply_chat_template does).
        base_output = self.load_mm_data(
            prompt=prompt_with_placeholders,
            audio_data=audio_data,
            multimodal_tokens=self.mm_tokens,
            audio_sample_rate=self.sampling_rate,
        )
        if base_output is None:
            return None

        # Convert loaded audio to tensors
        waveforms: List[torch.Tensor] = []
        for audio in base_output.audios:
            wav = torch.as_tensor(audio, dtype=torch.float32)
            if wav.dim() > 1:
                wav = wav.mean(dim=0)
            waveforms.append(wav)

        # Compute audio token counts and build input_ids with audio tokens
        audio_token_counts = [
            self._compute_audio_token_count(wav.shape[-1]) for wav in waveforms
        ]
        tokenizer = getattr(self._processor, "tokenizer", self._processor)
        input_ids = self._build_input_ids_with_audio(
            tokenizer, input_text, audio_token_counts
        )

        # Find offsets of [AUDIO] token runs and build mm_items
        audio_offsets = self._find_audio_offsets(input_ids, self.audio_token_id)
        mm_items = []
        for i, wav in enumerate(waveforms):
            item = MultimodalDataItem(feature=wav, modality=Modality.AUDIO)
            if i < len(audio_offsets):
                item.offsets = [audio_offsets[i]]
            mm_items.append(item)

        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=mm_items,
            audio_token_id=self.audio_token_id,
        )
```
**EN:** This block defines async function `VoxtralMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `VoxtralMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。

### Lines 123-123: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 124-133: Defines function VoxtralMultimodalProcessor._insert_audio_placeholders
```python
    @staticmethod
    def _insert_audio_placeholders(prompt: str, n_audio: int) -> str:
        """Insert [AUDIO] placeholder texts into the prompt for load_mm_data."""
        placeholders = AUDIO_PLACEHOLDER * n_audio
        # Insert after the last [INST] marker if present
        last_inst = prompt.rfind("[INST]")
        if last_inst >= 0:
            insert_pos = last_inst + len("[INST]")
            return prompt[:insert_pos] + placeholders + prompt[insert_pos:]
        return placeholders + prompt
```
**EN:** This block defines function `VoxtralMultimodalProcessor._insert_audio_placeholders`. Parameters: prompt, n_audio. Decorators: staticmethod. Insert [AUDIO] placeholder texts into the prompt for load_mm_data.
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor._insert_audio_placeholders`。 参数包括 prompt、n_audio。 装饰器包括 staticmethod。 文档字符串摘要：Insert [AUDIO] placeholder texts into the prompt for load_mm_data.

### Lines 134-134: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 135-149: Defines function VoxtralMultimodalProcessor._find_audio_offsets
```python
    @staticmethod
    def _find_audio_offsets(input_ids: List[int], audio_token_id: int) -> List[tuple]:
        """Find consecutive runs of audio_token_id in input_ids."""
        offsets = []
        start = None
        for i, tok_id in enumerate(input_ids):
            if tok_id == audio_token_id:
                if start is None:
                    start = i
            elif start is not None:
                offsets.append((start, i - 1))
                start = None
        if start is not None:
            offsets.append((start, len(input_ids) - 1))
        return offsets
```
**EN:** This block defines function `VoxtralMultimodalProcessor._find_audio_offsets`. Parameters: input_ids, audio_token_id. Decorators: staticmethod. Find consecutive runs of audio_token_id in input_ids.
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor._find_audio_offsets`。 参数包括 input_ids、audio_token_id。 装饰器包括 staticmethod。 文档字符串摘要：Find consecutive runs of audio_token_id in input_ids.

### Lines 150-150: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 151-180: Defines function VoxtralMultimodalProcessor._build_input_ids_with_audio
```python
    def _build_input_ids_with_audio(
        self,
        tokenizer,
        input_text: str,
        audio_token_counts: List[int],
    ) -> List[int]:
        """Build input_ids by tokenizing text and inserting audio tokens.

        The input_text is a decoded Mistral prompt (from text-only
        apply_chat_template).  We re-tokenize to get proper special tokens
        (BOS, [INST], [/INST]), then insert [BEGIN_AUDIO] + [AUDIO]*N after
        the last [INST].
        """
        messages = self._parse_mistral_prompt(input_text)
        try:
            input_ids = tokenizer.apply_chat_template(messages, tokenize=True)
        except (ValueError, KeyError):
            # Fallback if prompt parsing produces malformed messages
            input_ids = tokenizer.encode(input_text)

        # Insert audio tokens after the last [INST]
        inst_positions = [i for i, t in enumerate(input_ids) if t == INST_TOKEN_ID]
        insert_pos = (inst_positions[-1] + 1) if inst_positions else 1

        audio_tokens = []
        for count in audio_token_counts:
            audio_tokens.append(BEGIN_AUDIO_TOKEN_ID)
            audio_tokens.extend([AUDIO_TOKEN_ID] * count)

        return input_ids[:insert_pos] + audio_tokens + input_ids[insert_pos:]
```
**EN:** This block defines function `VoxtralMultimodalProcessor._build_input_ids_with_audio`. Parameters: self, tokenizer, input_text, audio_token_counts. Build input_ids by tokenizing text and inserting audio tokens. The input_text is a decoded Mistral prompt (from text-only apply_chat_template).
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor._build_input_ids_with_audio`。 参数包括 self、tokenizer、input_text、audio_token_counts。 文档字符串摘要：Build input_ids by tokenizing text and inserting audio tokens. The input_text is a decoded Mistral prompt (from text-only apply_chat_template).

### Lines 181-181: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VoxtralMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VoxtralMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 182-217: Defines function VoxtralMultimodalProcessor._parse_mistral_prompt
```python
    @staticmethod
    def _parse_mistral_prompt(prompt: str) -> List[Dict[str, str]]:
        """Parse a Mistral-formatted prompt into a list of messages."""
        messages = []
        text = prompt.strip()

        for marker in ["<s>", "</s>"]:
            text = text.replace(marker, "")
        text = text.strip()

        # Extract system prompt
        system_match = re.search(
            r"\[SYSTEM_PROMPT\]\s*(.*?)\s*\[/SYSTEM_PROMPT\]", text, re.DOTALL
        )
        if system_match:
            messages.append(
                {"role": "system", "content": system_match.group(1).strip()}
            )
            text = text[: system_match.start()] + text[system_match.end() :]
            text = text.strip()

        # Split by [INST] / [/INST]
        parts = re.split(r"\[/?INST\]", text)
        for i, part in enumerate(parts):
            part = part.strip()
            if not part:
                continue
            if i % 2 == 1:
                messages.append({"role": "user", "content": part})
            elif i > 0:
                messages.append({"role": "assistant", "content": part})

        if not messages:
            messages.append({"role": "user", "content": text})

        return messages
```
**EN:** This block defines function `VoxtralMultimodalProcessor._parse_mistral_prompt`. Parameters: prompt. Decorators: staticmethod. Parse a Mistral-formatted prompt into a list of messages.
**CN:** 该代码块定义函数 `VoxtralMultimodalProcessor._parse_mistral_prompt`。 参数包括 prompt。 装饰器包括 staticmethod。 文档字符串摘要：Parse a Mistral-formatted prompt into a list of messages.

## Key Concepts / 关键概念
- **Classes / 类**: `VoxtralMultimodalProcessor`
- **Constants / 常量**: `AUDIO_TOKEN_ID`, `BEGIN_AUDIO_TOKEN_ID`, `INST_TOKEN_ID`, `AUDIO_PLACEHOLDER`, `AUDIO_PLACEHOLDER_REGEX`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`, `re`, `typing`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.voxtral`, `sglang.srt.multimodal.processors.base_processor`
