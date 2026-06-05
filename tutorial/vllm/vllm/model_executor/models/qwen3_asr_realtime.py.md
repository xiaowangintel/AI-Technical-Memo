# qwen3_asr_realtime.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/qwen3_asr_realtime.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Qwen3 ASR Realtime multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only Qwen3-ASR realtime model." / 实现 Qwen3 ASR Realtime 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only Qwen3-ASR realtime model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2026 The Qwen team.
# Copyright 2023 The vLLM team.
#
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
"""Inference-only Qwen3-ASR realtime model."""

# ... omitted for brevity ...
    Qwen3ASRMultiModalProcessor,
    Qwen3ASRProcessingInfo,
    _get_feat_extract_output_lengths,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.cache import _I, BaseMultiModalProcessorCache
from vllm.multimodal.inputs import MultiModalKwargsOptionalItems
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import BaseDummyInputsBuilder
from vllm.multimodal.processing.processor import (
    MultiModalPromptUpdates,
    PlaceholderFeaturesInfo,
)
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.processor import cached_processor_from_config
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as numpy, torch supply framework primitives, while internal modules like vllm.config, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 numpy, torch 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger, _PRE_ALLOCATE_BUFFER_SIZE_IN_S` (lines 50-52)
```python
logger = init_logger(__name__)

_PRE_ALLOCATE_BUFFER_SIZE_IN_S = 60
```
**EN:** This assignment block centers on `logger, _PRE_ALLOCATE_BUFFER_SIZE_IN_S` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `logger, _PRE_ALLOCATE_BUFFER_SIZE_IN_S` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `Qwen3ASRRealtimeBuffer` (lines 55-100)
```python
class Qwen3ASRRealtimeBuffer:
    """Audio buffer for Qwen3-ASR realtime streaming.

    Accumulates audio samples and yields segments when enough
    audio has been buffered for processing.
    """

    def __init__(self, sampling_rate: int, segment_duration_s: float = 5.0):
        self._sampling_rate = sampling_rate
        self._segment_size = int(segment_duration_s * sampling_rate)

        self._buffer_size = _PRE_ALLOCATE_BUFFER_SIZE_IN_S * sampling_rate
        self._buffer: np.ndarray = np.empty(self._buffer_size, dtype=np.float32)
        self._filled_len = 0

    def write_audio(self, audio: np.ndarray) -> None:
        put_end = self._filled_len + len(audio)
        if put_end > self._buffer_size:
            new_size = max(self._buffer_size * 2, put_end)
            new_buffer = np.empty(new_size, dtype=np.float32)
            new_buffer[: self._filled_len] = self._buffer[: self._filled_len]
            self._buffer = new_buffer
            self._buffer_size = new_size

        self._buffer[self._filled_len : put_end] = audio
        self._filled_len = put_end

    def read_audio(self) -> np.ndarray | None:
        if self._filled_len < self._segment_size:
            return None

        segment = self._buffer[: self._segment_size].copy()
        remaining = self._filled_len - self._segment_size
        if remaining > 0:
            self._buffer[:remaining] = self._buffer[
                self._segment_size : self._filled_len
            ]
        self._filled_len = remaining
        return segment

    def flush(self) -> np.ndarray | None:
        if self._filled_len == 0:
            return None
        audio = self._buffer[: self._filled_len].copy()
        self._filled_len = 0
        return audio
```
**EN:** Defines `Qwen3ASRRealtimeBuffer`, a supporting module used by the surrounding model implementation. Key methods such as `__init__`, `write_audio`, `read_audio`, `flush` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Audio buffer for Qwen3-ASR realtime streaming."
**CN:** 定义 `Qwen3ASRRealtimeBuffer`，它是一个被周边模型实现复用的支撑模块。 `__init__`, `write_audio`, `read_audio`, `flush` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Audio buffer for Qwen3-ASR realtime streaming。”

### Class `Qwen3ASRRealtimeMultiModalProcessor` (lines 103-167)
```python
class Qwen3ASRRealtimeMultiModalProcessor(Qwen3ASRMultiModalProcessor):
    def __init__(
        self,
        info: _I,
        dummy_inputs: BaseDummyInputsBuilder[_I],
        *,
        cache: BaseMultiModalProcessorCache | None = None,
    ) -> None:
        super().__init__(info, dummy_inputs, cache=None)

    def _maybe_apply_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        prompt_ids: list[int],
        mm_kwargs: MultiModalKwargsOptionalItems,
        mm_prompt_updates: MultiModalPromptUpdates,
        is_update_applied: bool,
    ) -> tuple[list[int], Mapping[str, list[PlaceholderFeaturesInfo]]]:
        audios = mm_kwargs.get("audio", [])
        assert len(audios) == 1, (
            f"Expected only one audio input for realtime, got {len(audios)}"
        )

        audio_data = audios[0]
        audio_feature_lengths = audio_data.get("audio_feature_lengths")
        if audio_feature_lengths is not None:
            if isinstance(audio_feature_lengths.data, torch.Tensor):
                audio_len = _get_feat_extract_output_lengths(
                    audio_feature_lengths.data
                ).item()
# ... omitted for brevity ...
        else:
            audio_len = 0

        # Get audio_pad token ID and expand placeholder in prompt_ids
        # so that MRoPE position computation matches seq_len.
        tokenizer = self.info.get_tokenizer()
        audio_pad_id = tokenizer.convert_tokens_to_ids("<|audio_pad|>")

        # Find the audio_pad token position and expand it to audio_len tokens
        expanded_ids = list[int]()
        pad_start_idx = -1
        for i, tid in enumerate(prompt_ids):
            if tid == audio_pad_id and pad_start_idx == -1:
                pad_start_idx = i
                expanded_ids.extend([audio_pad_id] * audio_len)
            else:
                expanded_ids.append(tid)

        if pad_start_idx == -1:
            pad_start_idx = 0

        features_info = PlaceholderFeaturesInfo(
            modality="audio",
            item_idx=0,
            start_idx=pad_start_idx,
            tokens=audio_len * [audio_pad_id],
            is_embed=None,
        )
        return expanded_ids, {"audio": [features_info]}
```
**EN:** Defines `Qwen3ASRRealtimeMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from Qwen3ASRMultiModalProcessor. Key methods such as `__init__`, `_maybe_apply_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRRealtimeMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3ASRMultiModalProcessor。 `__init__`, `_maybe_apply_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3ASRRealtimeGeneration` (lines 174-237)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3ASRRealtimeMultiModalProcessor,
    info=Qwen3ASRProcessingInfo,
    dummy_inputs=Qwen3ASRDummyInputsBuilder,
)
class Qwen3ASRRealtimeGeneration(Qwen3ASRForConditionalGeneration, SupportsRealtime):
    realtime_max_tokens = 64

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

    @classmethod
    async def buffer_realtime_audio(
        cls,
        audio_stream: AsyncGenerator[np.ndarray, None],
        input_stream: asyncio.Queue[list[int]],
        model_config: ModelConfig,
    ) -> AsyncGenerator[PromptType, None]:
        processor = cached_processor_from_config(model_config)
        feature_extractor = processor.feature_extractor
        sampling_rate = feature_extractor.sampling_rate
        tokenizer = cached_tokenizer_from_config(model_config)

        # Use a small segment size for low-latency streaming.
        segment_duration_s = 5.0
        buffer = Qwen3ASRRealtimeBuffer(
            sampling_rate=sampling_rate,
            segment_duration_s=segment_duration_s,
        )

# ... omitted for brevity ...
        prompt_token_ids = tokenizer.encode(prompt_template)

        async for audio_chunk in audio_stream:
            buffer.write_audio(audio_chunk)

            while (segment := buffer.read_audio()) is not None:
                yield TokensPrompt(
                    prompt_token_ids=prompt_token_ids,
                    multi_modal_data={"audio": segment},
                )

        remaining = buffer.flush()
        if remaining is not None and len(remaining) > 0:
            yield TokensPrompt(
                prompt_token_ids=prompt_token_ids,
                multi_modal_data={"audio": remaining},
            )

    @classmethod
    def get_speech_to_text_config(
        cls, model_config: ModelConfig, task_type: str
    ) -> SpeechToTextConfig:
        processor = cached_processor_from_config(model_config)
        feature_extractor = processor.feature_extractor
        return SpeechToTextConfig(
            max_audio_clip_s=None,
            sample_rate=feature_extractor.sampling_rate,
            min_energy_split_window_size=None,
        )
```
**EN:** Defines `Qwen3ASRRealtimeGeneration`, a supporting module used by the surrounding model implementation. It inherits from Qwen3ASRForConditionalGeneration, SupportsRealtime. Key methods such as `__init__`, `buffer_realtime_audio`, `get_speech_to_text_config` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3ASRRealtimeGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3ASRForConditionalGeneration、SupportsRealtime。 `__init__`, `buffer_realtime_audio`, `get_speech_to_text_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: asyncio, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: numpy, torch
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces, vllm.model_executor.models.qwen3_asr, vllm.multimodal, vllm.multimodal.cache, vllm.multimodal.inputs
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
