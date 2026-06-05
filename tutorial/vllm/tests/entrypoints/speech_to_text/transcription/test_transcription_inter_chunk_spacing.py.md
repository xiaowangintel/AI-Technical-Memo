# test_transcription_inter_chunk_spacing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/transcription/test_transcription_inter_chunk_spacing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 7 test(s), 0 fixture(s), and 13 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 7 个测试、0 个 fixture，以及 13 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L11-L36)
```python
from __future__ import annotations

import json
from collections.abc import AsyncGenerator
from types import SimpleNamespace
from unittest.mock import AsyncMock, MagicMock, patch

import pytest

from vllm.config import ModelConfig
from vllm.config.speech_to_text import SpeechToTextConfig
from vllm.entrypoints.openai.engine.protocol import (
    ErrorResponse,
    RequestResponseMetadata,
)
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.entrypoints.speech_to_text.base.serving import (
    OpenAISpeechToText,
# ... 4 lines omitted for brevity ...
    OpenAIServingTranscription,
)
from vllm.model_executor.models.interfaces import SupportsTranscription
from vllm.outputs import CompletionOutput, RequestOutput
```
**EN:** Imports standard-library modules such as `__future__.annotations`, `collections.abc.AsyncGenerator`, `json`, third-party packages like `pytest`, project helpers such as `vllm.config.ModelConfig`, `vllm.config.speech_to_text.SpeechToTextConfig`, `vllm.entrypoints.openai.engine.protocol.ErrorResponse`.
**CN:** 导入标准库模块（如 `__future__.annotations`、`collections.abc.AsyncGenerator`、`json`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.config.ModelConfig`、`vllm.config.speech_to_text.SpeechToTextConfig`、`vllm.entrypoints.openai.engine.protocol.ErrorResponse`）。

### Test / 测试: test_default_no_space_languages_includes_zh_and_ja (L41-L42)
```python
def test_default_no_space_languages_includes_zh_and_ja():
    assert SupportsTranscription.no_space_languages == {"ja", "zh"}
```
**EN:** This test validates `test_default_no_space_languages_includes_zh_and_ja`. The main assertion is `SupportsTranscription.no_space_languages == {'ja', 'zh'}`.
**CN:** 这个测试验证 `test_default_no_space_languages_includes_zh_and_ja`。 核心断言是 `SupportsTranscription.no_space_languages == {'ja', 'zh'}`。

### Test / 测试: test_asr_inter_chunk_separator_matches_protocol (L45-L58)
```python
@pytest.mark.parametrize(
    ("language", "expected_sep"),
    [
        ("en", " "),
        ("EN", " "),
        ("zh", ""),
        ("ZH", ""),
        ("ja", ""),
        (None, " "),
    ],
)
def test_asr_inter_chunk_separator_matches_protocol(language, expected_sep):
    sep = asr_inter_chunk_separator(language, SupportsTranscription.no_space_languages)
    assert sep == expected_sep
```
**EN:** This test validates `test_asr_inter_chunk_separator_matches_protocol`. It uses parameterization over `language`, `expected_sep`. Key inputs are `language`, `expected_sep`. The main assertion is `sep == expected_sep`.
**CN:** 这个测试验证 `test_asr_inter_chunk_separator_matches_protocol`。 它通过参数化组合 `language`、`expected_sep`。 关键输入包括 `language`、`expected_sep`。 核心断言是 `sep == expected_sep`。

### Test / 测试: test_joined_chunks_english_has_space_between (L61-L63)
```python
def test_joined_chunks_english_has_space_between():
    sep = asr_inter_chunk_separator("en", SupportsTranscription.no_space_languages)
    assert sep.join(["hello", "world"]) == "hello world"
```
**EN:** This test validates `test_joined_chunks_english_has_space_between`. The main assertion is `sep.join(['hello', 'world']) == 'hello world'`.
**CN:** 这个测试验证 `test_joined_chunks_english_has_space_between`。 核心断言是 `sep.join(['hello', 'world']) == 'hello world'`。

### Test / 测试: test_joined_chunks_chinese_has_no_space_between (L66-L68)
```python
def test_joined_chunks_chinese_has_no_space_between():
    sep = asr_inter_chunk_separator("zh", SupportsTranscription.no_space_languages)
    assert sep.join(["你好", "世界"]) == "你好世界"
```
**EN:** This test validates `test_joined_chunks_chinese_has_no_space_between`. The main assertion is `sep.join(['你好', '世界']) == '你好世界'`.
**CN:** 这个测试验证 `test_joined_chunks_chinese_has_no_space_between`。 核心断言是 `sep.join(['你好', '世界']) == '你好世界'`。

### Class / 类: _StubTranscriptionModel (L74-L91)
```python
class _StubTranscriptionModel:
    """Minimal stand-in for a SupportsTranscription implementation (no torch)."""

    no_space_languages: set[str] = {"ja", "zh"}
    supports_segment_timestamp = False

    @classmethod
    def get_speech_to_text_config(
        cls, model_config: ModelConfig, task_type: str
    ) -> SpeechToTextConfig:
        return SpeechToTextConfig(
            sample_rate=16000.0,
            max_audio_clip_s=5.0,
        )

    @classmethod
    def post_process_output(cls, text: str) -> str:
        return text
```
**EN:** This class groups related scenarios in `_StubTranscriptionModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `get_speech_to_text_config`, `post_process_output`.
**CN:** 该类将与 `_StubTranscriptionModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `get_speech_to_text_config`、`post_process_output`。

### Helper method / 辅助方法: _StubTranscriptionModel.get_speech_to_text_config (L80-L87)
```python
    @classmethod
    def get_speech_to_text_config(
        cls, model_config: ModelConfig, task_type: str
    ) -> SpeechToTextConfig:
        return SpeechToTextConfig(
            sample_rate=16000.0,
            max_audio_clip_s=5.0,
        )
```
**EN:** This helper encapsulates reusable logic in `_StubTranscriptionModel.get_speech_to_text_config`. Key inputs are `model_config`, `task_type`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_StubTranscriptionModel.get_speech_to_text_config` 中。 关键输入包括 `model_config`、`task_type`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: _StubTranscriptionModel.post_process_output (L89-L91)
```python
    @classmethod
    def post_process_output(cls, text: str) -> str:
        return text
```
**EN:** This helper encapsulates reusable logic in `_StubTranscriptionModel.post_process_output`. Key inputs are `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_StubTranscriptionModel.post_process_output` 中。 关键输入包括 `text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _request_output (L94-L111)
```python
def _request_output(text: str) -> RequestOutput:
    return RequestOutput(
        request_id="rid",
        prompt=None,
        prompt_token_ids=None,
        prompt_logprobs=None,
        outputs=[
            CompletionOutput(
                index=0,
                text=text,
                token_ids=(1, 2, 3),
                cumulative_logprob=None,
                logprobs=None,
                finish_reason="stop",
            )
        ],
        finished=True,
    )
```
**EN:** This helper encapsulates reusable logic in `_request_output`. Key inputs are `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_request_output` 中。 关键输入包括 `text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _sse_delta_contents (L114-L128)
```python
def _sse_delta_contents(sse_body: str) -> list[str]:
    """Extract ``choices[0].delta.content`` from each ``data:`` line (streaming API)."""
    contents: list[str] = []
    for line in sse_body.splitlines():
        if not line.startswith("data: "):
            continue
        payload = line.removeprefix("data: ").strip()
        if payload == "[DONE]":
            continue
        obj = json.loads(payload)
        for choice in obj.get("choices") or []:
            delta = choice.get("delta") or {}
            if "content" in delta:
                contents.append(delta["content"])
    return contents
```
**EN:** This helper encapsulates reusable logic in `_sse_delta_contents`. Key inputs are `sse_body`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_sse_delta_contents` 中。 关键输入包括 `sse_body`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_transcription_stream_generator_english_inserts_space_between_chunks (L131-L167)
```python
@pytest.mark.asyncio
async def test_transcription_stream_generator_english_inserts_space_between_chunks():
    """Online streaming: first output per audio chunk is prefixed with *separator*."""

    async def gen_hello() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("hello")

    async def gen_world() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("world")

    serving = OpenAIServingTranscription.__new__(OpenAIServingTranscription)
    serving.enable_force_include_usage = False
    serving.model_cls = _StubTranscriptionModel
    serving.task_type = "transcribe"
    request = SimpleNamespace(
        model="stub-model",
        stream_include_usage=False,
        stream_continuous_usage_stats=False,
# ... 11 lines omitted for brevity ...
        audio_duration_s=1.0,
        separator=sep,
    )
    async for line in agen:
        out_lines.append(line)
    sse = "".join(out_lines)
    combined = "".join(_sse_delta_contents(sse))
    assert combined.strip() == "hello world"
```
**EN:** This async test validates `test_transcription_stream_generator_english_inserts_space_between_chunks`. Relevant pytest markers include `asyncio`. The main assertion is `sep == ' '` and `combined.strip() == 'hello world'`.
**CN:** 这个异步测试验证 `test_transcription_stream_generator_english_inserts_space_between_chunks`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `sep == ' '` and `combined.strip() == 'hello world'`。

### Test / 测试: test_transcription_stream_generator_chinese_no_space_between_chunks (L170-L203)
```python
@pytest.mark.asyncio
async def test_transcription_stream_generator_chinese_no_space_between_chunks():
    async def gen_a() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("你好")

    async def gen_b() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("世界")

    serving = OpenAIServingTranscription.__new__(OpenAIServingTranscription)
    serving.enable_force_include_usage = False
    serving.model_cls = _StubTranscriptionModel
    serving.task_type = "transcribe"
    request = SimpleNamespace(
        model="stub-model",
        stream_include_usage=False,
        stream_continuous_usage_stats=False,
    )
    sep = asr_inter_chunk_separator("zh", _StubTranscriptionModel.no_space_languages)
# ... 8 lines omitted for brevity ...
        request_metadata=RequestResponseMetadata(request_id="test-req-zh"),
        audio_duration_s=1.0,
        separator=sep,
    )
    async for line in agen:
        out_lines.append(line)
    combined = "".join(_sse_delta_contents("".join(out_lines)))
    assert combined == "你好世界"
```
**EN:** This async test validates `test_transcription_stream_generator_chinese_no_space_between_chunks`. Relevant pytest markers include `asyncio`. The main assertion is `sep == ''` and `combined == '你好世界'`.
**CN:** 这个异步测试验证 `test_transcription_stream_generator_chinese_no_space_between_chunks`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `sep == ''` and `combined == '你好世界'`。

### Test / 测试: test_create_transcription_non_streaming_joins_chunks_by_language (L206-L273)
```python
@pytest.mark.asyncio
async def test_create_transcription_non_streaming_joins_chunks_by_language():
    """``create_transcription`` uses the same separator logic as the helper."""

    async def gen_hello() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("hello")

    async def gen_world() -> AsyncGenerator[RequestOutput, None]:
        yield _request_output("world")

    engine_client = MagicMock()
    engine_client.model_config = MagicMock()
    engine_client.model_config.get_diff_sampling_param.return_value = {
        "max_tokens": 256,
        "temperature": 0.0,
    }
    engine_client.model_config.max_model_len = 8192
    engine_client.errored = False
# ... 42 lines omitted for brevity ...
            stream=False,
            response_format="json",
        )
        out_zh = await serving.create_transcription(
            b"\x00\x00", req_zh, raw_request=None
        )
        assert not isinstance(out_zh, ErrorResponse)
        assert out_zh.text == "你好世界"
```
**EN:** This async test validates `test_create_transcription_non_streaming_joins_chunks_by_language`. Relevant pytest markers include `asyncio`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It drives client-facing request creation through the API surface under test. The main assertion is `not isinstance(out_en, ErrorResponse)` and `out_en.text == 'hello world'`.
**CN:** 这个异步测试验证 `test_create_transcription_non_streaming_joins_chunks_by_language`。 相关的 pytest 标记包括 `asyncio`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `not isinstance(out_en, ErrorResponse)` and `out_en.text == 'hello world'`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `__future__.annotations`, `collections.abc.AsyncGenerator`, `json`, `types.SimpleNamespace`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`, `unittest.mock.patch`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.config.ModelConfig`, `vllm.config.speech_to_text.SpeechToTextConfig`, `vllm.entrypoints.openai.engine.protocol.ErrorResponse`, `vllm.entrypoints.openai.engine.protocol.RequestResponseMetadata`, `vllm.entrypoints.openai.models.serving.OpenAIServingModels`, `vllm.entrypoints.speech_to_text.base.serving.OpenAISpeechToText`, `vllm.entrypoints.speech_to_text.base.serving.asr_inter_chunk_separator`, `vllm.entrypoints.speech_to_text.transcription.protocol.TranscriptionRequest`, `vllm.entrypoints.speech_to_text.transcription.serving.OpenAIServingTranscription`, `vllm.model_executor.models.interfaces.SupportsTranscription`, `vllm.outputs.CompletionOutput`, `vllm.outputs.RequestOutput`
