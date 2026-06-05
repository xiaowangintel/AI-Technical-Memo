# voxtral_realtime.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/voxtral_realtime.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Voxtral Realtime multimodal model adapter used for inference in vLLM. / 实现 Voxtral Realtime 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-48)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import asyncio
import math
from collections.abc import AsyncGenerator, Iterable, Iterator, Mapping

import numpy as np
import torch
from mistral_common.audio import Audio
from mistral_common.protocol.instruct.chunk import RawAudio
from mistral_common.protocol.transcription.request import (
    StreamingMode,
    TranscriptionRequest,
)
from mistral_common.tokens.tokenizers.audio import AudioConfig

from vllm.compilation.decorators import support_torch_compile
# ... omitted for brevity ...
from vllm.multimodal.cache import _I, BaseMultiModalProcessorCache
from vllm.multimodal.inputs import MultiModalKwargsOptionalItems
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import BaseDummyInputsBuilder
from vllm.multimodal.processing.processor import (
    MultiModalPromptUpdates,
    PlaceholderFeaturesInfo,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.utils.torch_utils import is_torch_equal_or_newer

from .utils import (
    _flatten_embeddings,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as numpy, torch, mistral_common.audio, mistral_common.protocol.instruct.chunk supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.config.speech_to_text, vllm.engine.protocol connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 numpy, torch, mistral_common.audio, mistral_common.protocol.instruct.chunk 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.config.speech_to_text, vllm.engine.protocol 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 49-49)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `VoxtralRealtimeMultiModalProcessor` (lines 52-93)
```python
class VoxtralRealtimeMultiModalProcessor(VoxtralMultiModalProcessor):
    def __init__(
        self,
        info: _I,
        dummy_inputs: BaseDummyInputsBuilder[_I],
        *,
        cache: BaseMultiModalProcessorCache | None = None,
    ) -> None:
        # realtime can't make use of a cache yet
        super().__init__(info, dummy_inputs, cache=None)

    def _maybe_apply_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        prompt_ids: list[int],
        mm_kwargs: MultiModalKwargsOptionalItems,
        mm_prompt_updates: MultiModalPromptUpdates,
        is_update_applied: bool,
    ) -> tuple[list[int], Mapping[str, list[PlaceholderFeaturesInfo]]]:
        # there are no placeholder audio tokens for streaming
        # so we need to build the place placeholder positions manually

        # in realtime there is always only one audio input
        audios = mm_kwargs.get("audio", [])
        assert len(audios) == 1, (
            f"Expected only one audio input for realtime, got {mm_kwargs=}"
        )
        tokenizer = self.info.get_tokenizer()
        audio_config = tokenizer.instruct.audio_encoder.audio_config

        num_audio_samples = audios[0]["audio_arrays"].data.shape[0]
        length = audio_config.num_audio_tokens(num_audio_samples)

        features_info = PlaceholderFeaturesInfo(
            modality="audio",
            item_idx=0,
            start_idx=0,
            tokens=length
            * [0],  # only used for length computation, so we can take dummy inputs
            is_embed=None,
        )
        return prompt_ids, {"audio": [features_info]}
```
**EN:** Defines `VoxtralRealtimeMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from VoxtralMultiModalProcessor. Key methods such as `__init__`, `_maybe_apply_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `VoxtralRealtimeMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 VoxtralMultiModalProcessor。 `__init__`, `_maybe_apply_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TimeEmbedding` (lines 96-116)
```python
class TimeEmbedding(torch.nn.Module):
    """Sinusoidal Embedding for encoding time"""

    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.dim = dim
        self.theta = theta
        inv_freq = torch.exp(
            -math.log(self.theta)
            * torch.arange(self.dim // 2).float()
            / (self.dim // 2)
        )
        self.register_buffer("inv_freq", inv_freq, persistent=False)

    def forward(self, t: torch.Tensor) -> torch.Tensor:
        t = t[..., None]  # (B,) -> (B, 1) or (B, T) -> (B, T, 1)
        inv_freq = self.inv_freq.to(device=t.device, dtype=t.dtype)
        emb = (
            t * inv_freq
        )  # (B, 1) x (D/2,) -> (B, D/2) or (B, T, 1) x (D/2,) -> (B, T, D/2)
        return torch.cat((emb.cos(), emb.sin()), dim=-1)  # (B, D) or (B, T, D)
```
**EN:** Defines `TimeEmbedding`, a supporting module used by the surrounding model implementation. It inherits from torch.nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Sinusoidal Embedding for encoding time."
**CN:** 定义 `TimeEmbedding`，它是一个被周边模型实现复用的支撑模块。 它继承自 torch.nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Sinusoidal Embedding for encoding time。”

### Function `_expand_tensor` (lines 119-128)
```python
def _expand_tensor(input_tensor: torch.Tensor, scaling: int) -> torch.Tensor:
    # 1. Multiply by the scaling factor (e.g. 4)
    base = input_tensor * scaling

    # 2. Create the offsets, e.g. [0, 1, 2, 3]
    offsets = torch.arange(scaling, device=input_tensor.device)

    # 3. Use broadcasting, e.g. (N, 1) + (4,) results in (N, 4)
    # Then flatten back to 1D
    return (base.unsqueeze(1) + offsets).view(-1)
```
**EN:** The function `_expand_tensor` helps provide a reusable helper for the surrounding model code. Its main inputs are `input_tensor`, `scaling`.
**CN:** 函数 `_expand_tensor` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `input_tensor`、`scaling`。

### Class `VoxtralRealtimeBuffer` (lines 131-206)
```python
class VoxtralRealtimeBuffer:
    def __init__(self, config: AudioConfig, prompt_tokens: list[int]) -> None:
        self._config = config

        _look_ahead_in_ms = self._config.streaming_look_ahead_ms
        _look_back_in_ms = self._config.streaming_look_back_ms
        self._look_ahead_in_samples = self._ms_to_samples(_look_ahead_in_ms)
        self._look_back_in_samples = self._ms_to_samples(_look_back_in_ms)

        # None signals the end
        self._audio_queue: asyncio.Queue[np.ndarray | None] = asyncio.Queue()
        self._leftover: np.ndarray | None = None
        self._token_queue: asyncio.Queue[int] = asyncio.Queue()

        self._initial_end = len(prompt_tokens) * self._config.raw_audio_length_per_tok
        for token in prompt_tokens:
            self._token_queue.put_nowait(token)
# ... omitted for brevity ...
    def _generate_frame_size_and_num_tokens(self) -> Iterator[tuple[int, int]]:
        streaming_step_size = self._ms_to_samples(1000 / self._config.frame_rate)
        start = 0
        end = self._initial_end
        while True:
            frame_start = max(start - self._look_back_in_samples, 0)
            frame_end = end + self._look_ahead_in_samples
            frame_size = frame_end - frame_start
            num_tokens = (end - start) / self._config.raw_audio_length_per_tok
            assert num_tokens.is_integer()
            yield frame_size, int(num_tokens)
            start = end
            end += streaming_step_size
# ... omitted for brevity ...
    def _ms_to_samples(self, ms: float) -> int:
        len_ = self._config.sampling_rate * ms / 1000
        assert len_.is_integer(), len_
        return int(len_)
# ... omitted for brevity ...
    async def append_audio(self, audio_array: np.ndarray | None) -> None:
        await self._audio_queue.put(audio_array)
```
**EN:** Defines `VoxtralRealtimeBuffer`, a supporting module used by the surrounding model implementation. Key methods such as `__init__`, `_generate_frame_size_and_num_tokens`, `_ms_to_samples`, `append_audio`, `append_tokens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `VoxtralRealtimeBuffer`，它是一个被周边模型实现复用的支撑模块。 `__init__`, `_generate_frame_size_and_num_tokens`, `_ms_to_samples`, `append_audio`, `append_tokens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `VoxtralRealtimeGeneration` (lines 209-492)
```python
@MULTIMODAL_REGISTRY.register_processor(
    VoxtralRealtimeMultiModalProcessor,
    info=VoxtralProcessingInfo,
    dummy_inputs=VoxtralDummyInputsBuilder,
)
@support_torch_compile
class VoxtralRealtimeGeneration(VoxtralForConditionalGeneration, SupportsRealtime):
    requires_raw_input_tokens = True
    # transformers' currently has limited support for MistralCommon backend
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        assert (
            not vllm_config.compilation_config.cudagraph_mode.has_full_cudagraphs()
        ), "Voxtral realtime doesn't support full cudagraphs yet. Please use PIECEWISE."

        self.time_embedding: TimeEmbedding = TimeEmbedding(
            dim=self.config.text_config.hidden_size
        )

        audio_config = self.tokenizer.instruct.audio_encoder.audio_config
        self.n_delay_tokens = audio_config.get_num_delay_tokens()
# ... omitted for brevity ...
    def embed_input_ids(
        self,
        input_ids: torch.Tensor,
        multimodal_embeddings: MultiModalEmbeddings | None = None,
        *,
        is_multimodal: torch.Tensor | None = None,
        # Multi-modal token ID may exceed vocab size
    ) -> torch.Tensor:
        """Pass post-conv embeddings directly as input.

        For realtime models, multimodal embeddings are required at every
# ... omitted for brevity ...
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        assert inputs_embeds is not None
        assert input_ids is not None

        pool_size = self.config.audio_config.block_pool_size
        if is_torch_equal_or_newer("2.11"):
            inputs_embeds = inputs_embeds.view(
                inputs_embeds.shape[0] * pool_size, inputs_embeds.shape[1] // pool_size
            )
        else:
            # TODO Use reshape + clone to break the view chain and avoid output
            # aliasing input bug in torch.compile's AOT autograd cache.
# ... omitted for brevity ...
    def embed_multimodal(
        self, **kwargs
    ) -> list[torch.Tensor] | torch.Tensor | tuple[torch.Tensor, ...] | None:
        """Transform audio waveforms -> initial whisper post-conv embeddings"""
        audio_inputs = self._parse_and_validate_audio_arrays(**kwargs)

        if audio_inputs is None:
            logger.warning(
                "Realtime model received no audio inputs in "
                "embed_multimodal. Returning empty embeddings."
            )
            return []

        def _truncate_left(
            sample: torch.Tensor, mult_of: int, pos: int
        ) -> torch.Tensor:
            assert pos in [0, 1], pos
            if (ctx := sample.shape[pos] % mult_of) != 0:
                sample = sample[ctx:] if pos == 0 else sample[:, ctx:]
```
**EN:** Defines `VoxtralRealtimeGeneration`, a supporting module used by the surrounding model implementation. It inherits from VoxtralForConditionalGeneration, SupportsRealtime. Key methods such as `__init__`, `buffer_realtime_audio`, `audio_config`, `embed_input_ids`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `VoxtralRealtimeGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 VoxtralForConditionalGeneration、SupportsRealtime。 `__init__`, `buffer_realtime_audio`, `audio_config`, `embed_input_ids`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: asyncio, math, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: numpy, torch, mistral_common.audio, mistral_common.protocol.instruct.chunk, mistral_common.protocol.transcription.request, mistral_common.tokens.tokenizers.audio
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.config.speech_to_text, vllm.engine.protocol, vllm.envs, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
