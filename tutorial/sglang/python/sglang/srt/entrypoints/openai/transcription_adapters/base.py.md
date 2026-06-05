# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/transcription_adapters/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements base logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 base 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-4: Import runtime dependencies / 导入运行时依赖
```python
from abc import ABC, abstractmethod
from typing import List, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-10: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    TranscriptionRequest,
    TranscriptionUsage,
    TranscriptionVerboseResponse,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-21: Register helpers or handlers / 注册辅助逻辑或处理器
```python
class TranscriptionAdapter(ABC):
    """Abstract base for model-specific transcription logic.

    Subclass this and decorate with ``@register_transcription_adapter("Key")``
    to add support for a new ASR model.  See the sibling modules for
    the built-in Whisper and Qwen3-ASR implementations.
    """

    @abstractmethod
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 22-23: Implement build sampling params / 实现build sampling params
```python
    def build_sampling_params(self, request: TranscriptionRequest) -> dict:
        """Return the ``sampling_params`` dict for ``GenerateReqInput``."""
```
**EN:** This block implements the method `build_sampling_params(request)` on `TranscriptionAdapter`. It focuses on Return the ``sampling_params`` dict for ``GenerateReqInput``., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `build_sampling_params(request)`。它围绕 `build_sampling_params` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 24-25: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 26-32: Implement supports language detection / 实现supports language detection
```python
    def supports_language_detection(self) -> bool:
        """Whether this model supports automatic language detection.

        When True, the adapter must implement the fused autodetect methods
        and the standalone detection methods below.
        """
        return False
```
**EN:** This block implements the method `supports_language_detection()` on `TranscriptionAdapter`. It focuses on Whether this model supports automatic language detection., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `supports_language_detection()`。它围绕 `supports_language_detection` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 33-35: Import runtime dependencies / 导入运行时依赖
```python

    # -- Fused detect+transcribe (used by the server) ----------------------
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 36-43: Implement build fused autodetect params / 实现build fused autodetect params
```python
    def build_fused_autodetect_params(self, request) -> dict:
        """Return ``sampling_params`` dict for a fused detect+transcribe request.

        Uses structured generation (``regex``) to constrain the output prefix
        to a valid language + task token sequence while allowing free
        transcription afterwards — all in a single request.
        """
        raise NotImplementedError
```
**EN:** This block implements the method `build_fused_autodetect_params(request)` on `TranscriptionAdapter`. It focuses on Return ``sampling_params`` dict for a fused detect+transcribe request., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `build_fused_autodetect_params(request)`。它围绕 `build_fused_autodetect_params` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 44-45: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 46-66: Implement parse fused output / 实现parse fused output
```python
    def parse_fused_output(
        text: str, *, ts_variant: bool = False
    ) -> tuple[Optional[str], Optional[str]]:
        """Parse the fused output into ``(language_code, user_visible_text)``.

        Called by both streaming and non-streaming handlers with the same
        contract. ``ts_variant`` indicates which forced-prefix shape was
        requested (the caller knows from ``request.timestamp_granularities``);
        adapters use it to disambiguate variants whose detokenized prefix
        differs in shape from their token-id prefix.

        * ``(None, None)`` — the forced prefix is not yet locatable.
          Streaming callers keep buffering; non-streaming / end-of-stream
          callers treat this as a parse failure and fall back to
          ``strip_special_tokens`` on the raw text.
        * ``(lang, visible)`` — prefix parsed. ``visible`` is fully
          user-visible (prefix removed, embedded special tokens scrubbed).
          It must grow monotonically across cumulative streaming snapshots
          so callers can compute deltas against it directly.
        """
        raise NotImplementedError
```
**EN:** This block implements the method `parse_fused_output(text, ts_variant)` on `TranscriptionAdapter`. It focuses on Parse the fused output into ``(language_code, user_visible_text)``., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `parse_fused_output(text, ts_variant)`。它围绕 `parse_fused_output` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 67-68: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 69-77: Implement strip special tokens / 实现strip special tokens
```python
    def strip_special_tokens(text: str) -> str:
        """Best-effort scrub of model-specific special-token strings.

        Used as a fallback when ``parse_fused_output`` reports a parse
        failure (e.g. FSM abort). Default is an identity pass-through;
        adapters that request generation with ``skip_special_tokens=False``
        should override to strip their special-token syntax.
        """
        return text
```
**EN:** This block implements the method `strip_special_tokens(text)` on `TranscriptionAdapter`. It focuses on Best-effort scrub of model-specific special-token strings., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `strip_special_tokens(text)`。它围绕 `strip_special_tokens` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 78-79: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 80-82: Implement supports chunked streaming / 实现supports chunked streaming
```python
    def supports_chunked_streaming(self) -> bool:
        """Whether this model uses chunk-based streaming instead of token-level streaming."""
        return False
```
**EN:** This block implements the method `supports_chunked_streaming()` on `TranscriptionAdapter`. It focuses on Whether this model uses chunk-based streaming instead of token-level streaming., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `supports_chunked_streaming()`。它围绕 `supports_chunked_streaming` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 83-84: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 85-91: Implement prompt template / 实现prompt template
```python
    def prompt_template(self) -> str:
        """Prompt template for chunked streaming requests.

        Only used when ``supports_chunked_streaming`` is True.
        The default returns an empty string.
        """
        return ""
```
**EN:** This block implements the method `prompt_template()` on `TranscriptionAdapter`. It focuses on Prompt template for chunked streaming requests., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `prompt_template()`。它围绕 `prompt_template` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 92-93: Provide supporting module logic / 提供辅助模块逻辑
```python

    @property
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 94-100: Implement chunked streaming config / 实现chunked streaming 配置
```python
    def chunked_streaming_config(self) -> dict:
        """Parameters for ``StreamingASRState`` when using chunked streaming.

        Only used when ``supports_chunked_streaming`` is True.
        Keys: ``chunk_size_sec``, ``unfixed_chunk_num``, ``unfixed_token_num``.
        """
        return {}
```
**EN:** This block implements the method `chunked_streaming_config()` on `TranscriptionAdapter`. It focuses on Parameters for ``StreamingASRState`` when using chunked streaming., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `chunked_streaming_config()`。它围绕 `chunked_streaming_config` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 102-107: Implement postprocess text / 实现postprocess text
```python
    def postprocess_text(self, text: str) -> str:
        """Strip model-specific markers from raw decoded text.

        The default implementation is a no-op pass-through.
        """
        return text
```
**EN:** This block implements the method `postprocess_text(text)` on `TranscriptionAdapter`. It focuses on Strip model-specific markers from raw decoded text., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `postprocess_text(text)`。它围绕 `postprocess_text` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 108-109: Provide supporting module logic / 提供辅助模块逻辑
```python

    @abstractmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 110-118: Implement build verbose response / 实现build verbose 响应
```python
    def build_verbose_response(
        self,
        request: TranscriptionRequest,
        text: str,
        ret: dict,
        tokenizer,
        usage: TranscriptionUsage,
    ) -> TranscriptionVerboseResponse:
        """Build a ``verbose_json`` response with segments / timestamps."""
```
**EN:** This block implements the method `build_verbose_response(request, text, ret, tokenizer, usage)` on `TranscriptionAdapter`. It focuses on Build a ``verbose_json`` response with segments / timestamps., so the class can advance the base workflow in a self-contained way.
**CN:** 该代码块实现 `TranscriptionAdapter` 上的方法 `build_verbose_response(request, text, ret, tokenizer, usage)`。它围绕 `build_verbose_response` 所承担的 base 相关职责展开，使该类能够独立推进相应流程。

### Lines 121-122: Provide supporting module logic / 提供辅助模块逻辑
```python
_ADAPTER_REGISTRY: dict[str, type[TranscriptionAdapter]] = {}
_DEFAULT_ADAPTER_KEY = "Whisper"
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 125-139: Implement register transcription adapter / 实现register transcription adapter
```python
def register_transcription_adapter(
    key: str,
) -> callable:
    """Class decorator that registers a ``TranscriptionAdapter`` subclass.

    *key* is matched as a substring against the model's HF ``architectures``
    list at init time (e.g. ``"Whisper"`` matches
    ``"WhisperForConditionalGeneration"``).
    """

    def decorator(cls: type[TranscriptionAdapter]) -> type[TranscriptionAdapter]:
        _ADAPTER_REGISTRY[key] = cls
        return cls

    return decorator
```
**EN:** This block implements the function `register_transcription_adapter(key)`. It focuses on Class decorator that registers a ``TranscriptionAdapter`` subclass., providing reusable behavior for the base pipeline.
**CN:** 该代码块实现函数 `register_transcription_adapter(key)`。它围绕 `register_transcription_adapter` 所承担的 base 相关职责展开，为对应处理链路提供可复用能力。

### Lines 142-154: Implement resolve adapter / 实现resolve adapter
```python
def resolve_adapter(architectures: List[str]) -> TranscriptionAdapter:
    """Pick the right adapter by matching architecture names against the registry."""
    for arch in architectures or []:
        for key, adapter_cls in _ADAPTER_REGISTRY.items():
            if key in arch:
                return adapter_cls()
    default_cls = _ADAPTER_REGISTRY.get(_DEFAULT_ADAPTER_KEY)
    if default_cls is None:
        raise RuntimeError(
            "No transcription adapters registered. "
            "Make sure 'transcription_adapters' package is importable."
        )
    return default_cls()
```
**EN:** This block implements the function `resolve_adapter(architectures)`. It focuses on Pick the right adapter by matching architecture names against the registry., providing reusable behavior for the base pipeline.
**CN:** 该代码块实现函数 `resolve_adapter(architectures)`。它围绕 `resolve_adapter` 所承担的 base 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: TranscriptionAdapter
- **Main callables / 主要可调用对象**: register_transcription_adapter, resolve_adapter
- **Domain focus / 领域焦点**: base / base
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing
- **Third-party / 第三方库**: __future__, abc
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol
