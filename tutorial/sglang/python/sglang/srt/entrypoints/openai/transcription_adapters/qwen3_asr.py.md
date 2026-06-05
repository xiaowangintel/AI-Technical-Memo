# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/transcription_adapters/qwen3_asr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements qwen3 asr logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 qwen3 asr 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-15: Register helpers or handlers / 注册辅助逻辑或处理器
```python
from sglang.srt.entrypoints.openai.protocol import (
    TranscriptionRequest,
    TranscriptionUsage,
    TranscriptionVerboseResponse,
)
from sglang.srt.entrypoints.openai.transcription_adapters.base import (
    TranscriptionAdapter,
    register_transcription_adapter,
)
from sglang.srt.multimodal.processors.qwen3_asr import DEFAULT_ASR_PROMPT


@register_transcription_adapter("Qwen3ASR")
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 16-69: Define class Qwen3ASRAdapter / 定义类 Qwen3ASRAdapter
```python
class Qwen3ASRAdapter(TranscriptionAdapter):
    ASR_TEXT_TAG = "<asr_text>"

    @property
    def supports_chunked_streaming(self) -> bool:
        return True

    @property
    def chunked_streaming_config(self) -> dict:
        # Qwen3-ASR paper (arXiv:2601.21337), Table 8 uses 4 unfixed chunks.
        # We use 2 here for lower latency; tune based on quality needs.
        # TODO: allow users to override these via API request parameters.
        return {
            "chunk_size_sec": 2.0,
            "unfixed_chunk_num": 2,
            "unfixed_token_num": 5,
        }

    @property
    def prompt_template(self) -> str:
        return DEFAULT_ASR_PROMPT

    def build_sampling_params(self, request: TranscriptionRequest) -> dict:
        temperature = request.temperature
        if temperature == 0.0:
            temperature = 0.01  # Qwen3-ASR recommended near-greedy temperature
        return {
            "temperature": temperature,
            "max_new_tokens": 256,  # Qwen3-ASR default
        }

    def postprocess_text(self, text: str) -> str:
        # Qwen3-ASR outputs "language <lang><asr_text>transcription" format;
        # strip the prefix to return clean transcription text.
        if self.ASR_TEXT_TAG in text:
            return text.split(self.ASR_TEXT_TAG, 1)[-1]
        return text

    def build_verbose_response(
        self,
        request: TranscriptionRequest,
        text: str,
        ret: dict,
        tokenizer,
        usage: TranscriptionUsage,
    ) -> TranscriptionVerboseResponse:
        # TODO: Qwen3-ASR needs ForcedAligner to produce timestamps
        return TranscriptionVerboseResponse(
            language=request.language or "auto",
            duration=round(request.audio_duration_s, 2),
            text=text,
            segments=[],
            usage=usage,
        )
```
**EN:** This block declares the class `Qwen3ASRAdapter`. It centers on coordinating qwen3 asr behavior, with methods such as supports_chunked_streaming, chunked_streaming_config, prompt_template, build_sampling_params, postprocess_text, ....
**CN:** 该代码块声明类 `Qwen3ASRAdapter`。它负责承载与 qwen3 asr 相关的核心状态与行为，并通过 supports_chunked_streaming, chunked_streaming_config, prompt_template, build_sampling_params, postprocess_text, ... 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: Qwen3ASRAdapter
- **Domain focus / 领域焦点**: qwen3 asr / qwen3 asr
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: None / 无
- **Third-party / 第三方库**: __future__
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.transcription_adapters.base, sglang.srt.multimodal.processors.qwen3_asr
