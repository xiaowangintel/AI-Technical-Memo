# test_transcription_validation_whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/transcription/test_transcription_validation_whisper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 15 test(s), 3 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 15 个测试、3 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L17)
```python
import asyncio
import io
import json

import numpy as np
import openai
import pytest
import pytest_asyncio
import soundfile as sf

from tests.utils import RemoteOpenAIServer
from vllm.multimodal.media.audio import load_audio
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `asyncio`, `io`, `json`, third-party packages like `numpy`, `openai`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.media.audio.load_audio`, `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `asyncio`、`io`、`json`）、第三方包（如 `numpy`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.multimodal.media.audio.load_audio`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: MODEL_NAME, _ROCM_ARGS (L19-L23)
```python
MODEL_NAME = "openai/whisper-large-v3-turbo"

# Disable prefix caching on ROCm to reduce non-determinism in
# streaming-vs-non-streaming comparisons.
_ROCM_ARGS = ["--no-enable-prefix-caching"] if current_platform.is_rocm() else []
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `_ROCM_ARGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`_ROCM_ARGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _get_attention_backend_params (L26-L50)
```python
def _get_attention_backend_params() -> list[str | None]:
    """Return attention backends to parametrize the server fixture with.

    On ROCm, we test multiple backends explicitly:
      - None: default auto-selection (ROCM_ATTN for decoder self-attention,
              falls back to ROCM_AITER_UNIFIED_ATTN or TRITON_ATTN for
              cross-attention since ROCM_ATTN doesn't support ENCODER_DECODER)
      - TRITON_ATTN: always available on ROCm
      - ROCM_AITER_UNIFIED_ATTN: only on gfx942/gfx950

    On non-ROCm platforms, we just run with the default backend.
    """
    try:
        from vllm.platforms import current_platform

        if current_platform.is_rocm():
            backends: list[str | None] = [None, "TRITON_ATTN"]
            from vllm.platforms.rocm import _ON_MI3XX

            if _ON_MI3XX:
                backends.append("ROCM_AITER_UNIFIED_ATTN")
            return backends
    except Exception:
        pass
    return [None]
```
**EN:** This helper encapsulates reusable logic in `_get_attention_backend_params`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_attention_backend_params` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: _AITER_ENV, _ATTN_BACKENDS, _ATTN_IDS (L55-L61)
```python
_AITER_ENV = {
    "VLLM_ROCM_USE_AITER": "1",
    "VLLM_ROCM_USE_AITER_MHA": "1",
}

_ATTN_BACKENDS = _get_attention_backend_params()
_ATTN_IDS = [b or "default" for b in _ATTN_BACKENDS]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `_AITER_ENV`, `_ATTN_BACKENDS`, `_ATTN_IDS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `_AITER_ENV`、`_ATTN_BACKENDS`、`_ATTN_IDS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L64-L73)
```python
@pytest.fixture(scope="module", params=_ATTN_BACKENDS, ids=_ATTN_IDS)
def server(request):
    args = [*_ROCM_ARGS]
    env_dict = None
    if request.param is not None:
        args += ["--attention-backend", request.param]
        if "AITER" in request.param:
            env_dict = _AITER_ENV
    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_dict) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: whisper_client (L76-L79)
```python
@pytest_asyncio.fixture
async def whisper_client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `whisper_client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `whisper_client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic_audio (L82-L96)
```python
@pytest.mark.asyncio
async def test_basic_audio(whisper_client, mary_had_lamb):
    # Based on https://github.com/openai/openai-cookbook/blob/main/examples/Whisper_prompting_guide.ipynb.
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
    )
    out = json.loads(transcription)
    out_text = out["text"]
    out_usage = out["usage"]
    assert "Mary had a little lamb," in out_text
    assert out_usage["seconds"] == 16, out_usage["seconds"]
```
**EN:** This async test validates `test_basic_audio`. Relevant pytest markers include `asyncio`. Key inputs are `whisper_client`, `mary_had_lamb`. It drives client-facing request creation through the API surface under test. The main assertion is `'Mary had a little lamb,' in out_text` and `out_usage['seconds'] == 16`.
**CN:** 这个异步测试验证 `test_basic_audio`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `whisper_client`、`mary_had_lamb`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'Mary had a little lamb,' in out_text` and `out_usage['seconds'] == 16`。

### Test / 测试: test_basic_audio_batched (L99-L122)
```python
@pytest.mark.asyncio
async def test_basic_audio_batched(mary_had_lamb, winning_call, whisper_client):
    transcription = whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
    )
    transcription2 = whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=winning_call,
        language="en",
        response_format="text",
        temperature=0.0,
    )
    # Await both transcriptions by scheduling coroutines together
    transcription, transcription2 = await asyncio.gather(transcription, transcription2)
    out = json.loads(transcription)
    out_text = out["text"]
    assert "Mary had a little lamb," in out_text
    out2 = json.loads(transcription2)
    out_text2 = out2["text"]
    assert "Edgar Martinez" in out_text2
```
**EN:** This async test validates `test_basic_audio_batched`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `winning_call`, `whisper_client`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `'Mary had a little lamb,' in out_text` and `'Edgar Martinez' in out_text2`.
**CN:** 这个异步测试验证 `test_basic_audio_batched`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`winning_call`、`whisper_client`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'Mary had a little lamb,' in out_text` and `'Edgar Martinez' in out_text2`。

### Test / 测试: test_bad_requests (L125-L131)
```python
@pytest.mark.asyncio
async def test_bad_requests(mary_had_lamb, whisper_client):
    # invalid language
    with pytest.raises(openai.BadRequestError):
        await whisper_client.audio.transcriptions.create(
            model=MODEL_NAME, file=mary_had_lamb, language="hh", temperature=0.0
        )
```
**EN:** This async test validates `test_bad_requests`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_bad_requests`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_long_audio_request (L134-L157)
```python
@pytest.mark.asyncio
async def test_long_audio_request(mary_had_lamb, whisper_client):
    mary_had_lamb.seek(0)
    audio, sr = load_audio(mary_had_lamb)
    # Add small silence after each audio for repeatability in the split process
    audio = np.pad(audio, (0, 1600))
    repeated_audio = np.tile(audio, 10)
    # Repeated audio to buffer
    buffer = io.BytesIO()
    sf.write(buffer, repeated_audio, sr, format="WAV")
    buffer.seek(0)
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=buffer,
        language="en",
        response_format="text",
        temperature=0.0,
    )
    out = json.loads(transcription)
    out_text = out["text"]
    out_usage = out["usage"]
    counts = out_text.count("Mary had a little lamb")
    assert counts == 10, counts
    assert out_usage["seconds"] == 161, out_usage["seconds"]
```
**EN:** This async test validates `test_long_audio_request`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `counts == 10` and `out_usage['seconds'] == 161`.
**CN:** 这个异步测试验证 `test_long_audio_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `counts == 10` and `out_usage['seconds'] == 161`。

### Test / 测试: test_invalid_audio_file (L160-L174)
```python
@pytest.mark.asyncio
async def test_invalid_audio_file(whisper_client):
    """Corrupted audio should surface as HTTP 400."""
    invalid_audio = io.BytesIO(b"not a valid audio file")
    invalid_audio.name = "invalid.wav"

    with pytest.raises(openai.BadRequestError) as exc_info:
        await whisper_client.audio.transcriptions.create(
            model=MODEL_NAME,
            file=invalid_audio,
            language="en",
        )

    assert exc_info.value.status_code == 400
    assert "Invalid or unsupported audio file" in exc_info.value.message
```
**EN:** This async test validates `test_invalid_audio_file`. Relevant pytest markers include `asyncio`. Key inputs are `whisper_client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `exc_info.value.status_code == 400` and `'Invalid or unsupported audio file' in exc_info.value.message`.
**CN:** 这个异步测试验证 `test_invalid_audio_file`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `whisper_client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `exc_info.value.status_code == 400` and `'Invalid or unsupported audio file' in exc_info.value.message`。

### Test / 测试: test_completion_endpoints (L177-L187)
```python
@pytest.mark.asyncio
async def test_completion_endpoints(whisper_client):
    # text to text model
    with pytest.raises(openai.NotFoundError):
        await whisper_client.chat.completions.create(
            model=MODEL_NAME,
            messages=[{"role": "system", "content": "You are a helpful assistant."}],
        )

    with pytest.raises(openai.NotFoundError):
        await whisper_client.completions.create(model=MODEL_NAME, prompt="Hello")
```
**EN:** This async test validates `test_completion_endpoints`. Relevant pytest markers include `asyncio`. Key inputs are `whisper_client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_completion_endpoints`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `whisper_client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_streaming_response (L190-L213)
```python
@pytest.mark.asyncio
async def test_streaming_response(winning_call, whisper_client):
    transcription = ""
    res_no_stream = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=winning_call,
        response_format="json",
        language="en",
        temperature=0.0,
    )
    res = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=winning_call,
        language="en",
        temperature=0.0,
        stream=True,
        timeout=30,
    )
    # Reconstruct from chunks and validate
    async for chunk in res:
        text = chunk.choices[0]["delta"]["content"]
        transcription += text

    assert transcription == res_no_stream.text
```
**EN:** This async test validates `test_streaming_response`. Relevant pytest markers include `asyncio`. Key inputs are `winning_call`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `transcription == res_no_stream.text`.
**CN:** 这个异步测试验证 `test_streaming_response`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `winning_call`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `transcription == res_no_stream.text`。

### Test / 测试: test_stream_options (L216-L235)
```python
@pytest.mark.asyncio
async def test_stream_options(winning_call, whisper_client):
    res = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=winning_call,
        language="en",
        temperature=0.0,
        stream=True,
        extra_body=dict(stream_include_usage=True, stream_continuous_usage_stats=True),
        timeout=30,
    )
    final = False
    continuous = True
    async for chunk in res:
        if not len(chunk.choices):
            # final usage sent
            final = True
        else:
            continuous = continuous and hasattr(chunk, "usage")
    assert final and continuous
```
**EN:** This async test validates `test_stream_options`. Relevant pytest markers include `asyncio`. Key inputs are `winning_call`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `final and continuous`.
**CN:** 这个异步测试验证 `test_stream_options`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `winning_call`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `final and continuous`。

### Test / 测试: test_sampling_params (L238-L268)
```python
@pytest.mark.asyncio
async def test_sampling_params(mary_had_lamb, whisper_client):
    """
    Compare sampling with params and greedy sampling to assert results
    are different when extreme sampling parameters values are picked.
    """
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        temperature=0.8,
        extra_body=dict(
            seed=42,
            repetition_penalty=1.9,
            top_k=12,
            top_p=0.4,
            min_p=0.5,
            frequency_penalty=1.8,
# ... 5 lines omitted for brevity ...
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        temperature=0.0,
        extra_body=dict(seed=42),
    )

    assert greedy_transcription.text != transcription.text
```
**EN:** This async test validates `test_sampling_params`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `greedy_transcription.text != transcription.text`.
**CN:** 这个异步测试验证 `test_sampling_params`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `greedy_transcription.text != transcription.text`。

### Test / 测试: test_audio_prompt (L271-L294)
```python
@pytest.mark.asyncio
async def test_audio_prompt(mary_had_lamb, whisper_client):
    prompt = "This is a speech, recorded in a phonograph."
    # Prompts should not omit the part of original prompt while transcribing.
    prefix = "The first words I spoke in the original phonograph"
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
    )
    out = json.loads(transcription)["text"]
    assert prefix in out
    transcription_wprompt = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        prompt=prompt,
        temperature=0.0,
    )
    out_prompt = json.loads(transcription_wprompt)["text"]
    assert prefix in out_prompt
```
**EN:** This async test validates `test_audio_prompt`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `prefix in out` and `prefix in out_prompt`.
**CN:** 这个异步测试验证 `test_audio_prompt`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `prefix in out` and `prefix in out_prompt`。

### Test / 测试: test_audio_with_timestamp (L297-L309)
```python
@pytest.mark.asyncio
async def test_audio_with_timestamp(mary_had_lamb, whisper_client):
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="verbose_json",
        temperature=0.0,
    )
    assert transcription.segments is not None
    assert len(transcription.segments) > 0
    assert transcription.segments[0].avg_logprob is not None
    assert transcription.segments[0].compression_ratio is not None
```
**EN:** This async test validates `test_audio_with_timestamp`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `transcription.segments is not None` and `len(transcription.segments) > 0`.
**CN:** 这个异步测试验证 `test_audio_with_timestamp`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `transcription.segments is not None` and `len(transcription.segments) > 0`。

### Test / 测试: test_audio_with_max_tokens (L312-L341)
```python
@pytest.mark.asyncio
async def test_audio_with_max_tokens(whisper_client, mary_had_lamb):
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
        extra_body={"max_completion_tokens": 1},
    )
    out = json.loads(transcription)
    out_text = out["text"]
    from transformers import AutoTokenizer

    tok = AutoTokenizer.from_pretrained(MODEL_NAME)
    out_tokens = tok(out_text, add_special_tokens=False)["input_ids"]
    assert len(out_tokens) == 1
    # max_completion_tokens > max_model_len
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
        extra_body={"max_completion_tokens": int(1e6)},
    )
    out = json.loads(transcription)
    out_text = out["text"]
    out_tokens = tok(out_text, add_special_tokens=False)["input_ids"]
    assert len(out_tokens) < 450  # ~Whisper max output len
```
**EN:** This async test validates `test_audio_with_max_tokens`. Relevant pytest markers include `asyncio`. Key inputs are `whisper_client`, `mary_had_lamb`. It drives client-facing request creation through the API surface under test. The main assertion is `len(out_tokens) == 1` and `len(out_tokens) < 450`.
**CN:** 这个异步测试验证 `test_audio_with_max_tokens`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `whisper_client`、`mary_had_lamb`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(out_tokens) == 1` and `len(out_tokens) < 450`。

### Fixture / 夹具: test_language_auto_detect (L344-L368)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    ("fixture_name", "expected_lang", "expected_text"),
    [
        ("mary_had_lamb", "en", ["Mary had a little lamb"]),
        ("foscolo", "it", ["zacinto", "sacre"]),
    ],
    ids=["english", "italian"],
)
async def test_language_auto_detect(
    whisper_client, fixture_name, expected_lang, expected_text, request
):
    """Auto-detect language when no language param is provided."""
    audio_file = request.getfixturevalue(fixture_name)
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=audio_file,
        response_format="verbose_json",
        temperature=0.0,
    )
    assert transcription.language == expected_lang
    text_lower = transcription.text.lower()
    assert any(word.lower() in text_lower for word in expected_text), (
        f"Expected {expected_lang} text but got: {transcription.text}"
    )
```
**EN:** This async fixture prepares `test_language_auto_detect` for dependent tests. It uses parameterization to cover `english`, `italian`. Relevant pytest markers include `asyncio`. Key inputs are `whisper_client`, `fixture_name`, `expected_lang`, `expected_text`, `request`. It drives client-facing request creation through the API surface under test. The main assertion is `transcription.language == expected_lang` and `any((word.lower() in text_lower for word in expected_text))`.
**CN:** 这个异步 fixture 为依赖它的测试准备 `test_language_auto_detect`。 它通过参数化覆盖 `english`、`italian` 等场景。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `whisper_client`、`fixture_name`、`expected_lang`、`expected_text`、`request`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `transcription.language == expected_lang` and `any((word.lower() in text_lower for word in expected_text))`。

### Test / 测试: test_whisper_beam_search_single_beam (L371-L397)
```python
@pytest.mark.asyncio
async def test_whisper_beam_search_single_beam(mary_had_lamb, whisper_client):
    """Test beam search with encoder-decoder model (Whisper) on transcriptions with
    one beam aligns with greedy decoding.
    """
    beam_transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
        extra_body=dict(
            use_beam_search=True,
            n=1,
        ),
    )

    greedy_transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        response_format="text",
        temperature=0.0,
    )

    greedy_res = json.loads(greedy_transcription)["text"]
    beam_res = json.loads(beam_transcription)["text"]
    assert greedy_res == beam_res
```
**EN:** This async test validates `test_whisper_beam_search_single_beam`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `greedy_res == beam_res`.
**CN:** 这个异步测试验证 `test_whisper_beam_search_single_beam`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `greedy_res == beam_res`。

### Test / 测试: test_whisper_beam_search_multibeam (L400-L421)
```python
@pytest.mark.asyncio
async def test_whisper_beam_search_multibeam(mary_had_lamb, whisper_client):
    """Test n>1 for beam search returns one transcription (best beam)."""
    transcription = await whisper_client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=mary_had_lamb,
        language="en",
        response_format="text",
        temperature=0.0,
        extra_body=dict(
            use_beam_search=True,
            n=2,
        ),
    )

    result = json.loads(transcription)

    text = result["text"]

    assert text is not None
    assert len(text) > 0
    assert "mary had a little lamb" in text.lower()
```
**EN:** This async test validates `test_whisper_beam_search_multibeam`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `whisper_client`. It drives client-facing request creation through the API surface under test. The main assertion is `text is not None` and `len(text) > 0`.
**CN:** 这个异步测试验证 `test_whisper_beam_search_multibeam`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`whisper_client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `text is not None` and `len(text) > 0`。

### Test / 测试: test_stream_with_beams_raises (L424-L437)
```python
@pytest.mark.asyncio
async def test_stream_with_beams_raises(winning_call, whisper_client):
    """Test that stream=True + beam search raises bad request for now."""
    with pytest.raises(openai.BadRequestError):
        await whisper_client.audio.transcriptions.create(
            model=MODEL_NAME,
            file=winning_call,
            language="en",
            stream=True,
            extra_body=dict(
                use_beam_search=True,
                n=2,
            ),
        )
```
**EN:** This async test validates `test_stream_with_beams_raises`. Relevant pytest markers include `asyncio`. Key inputs are `winning_call`, `whisper_client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_stream_with_beams_raises`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `winning_call`、`whisper_client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `io`, `json`
- **Third-party / 第三方**: `numpy`, `openai`, `pytest`, `pytest_asyncio`, `soundfile`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.media.audio.load_audio`, `vllm.platforms.current_platform`, `vllm.platforms.rocm._ON_MI3XX`
