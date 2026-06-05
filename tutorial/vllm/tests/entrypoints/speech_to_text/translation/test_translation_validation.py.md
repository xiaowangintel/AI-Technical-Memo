# test_translation_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/translation/test_translation_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 8 test(s), 2 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 8 个测试、2 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L19)
```python
import io

# imports for structured outputs tests
import json

import httpx
import numpy as np
import openai
import pytest
import pytest_asyncio
import soundfile as sf

from tests.entrypoints.speech_to_text.conftest import add_attention_backend
from tests.utils import RemoteOpenAIServer
from vllm.logger import init_logger
from vllm.multimodal.media.audio import load_audio
```
**EN:** Imports standard-library modules such as `io`, `json`, third-party packages like `httpx`, `numpy`, `openai`, project helpers such as `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.RemoteOpenAIServer`, `vllm.logger.init_logger`.
**CN:** 导入标准库模块（如 `io`、`json`）、第三方包（如 `httpx`、`numpy`、`openai`）、项目内辅助模块（如 `tests.entrypoints.speech_to_text.conftest.add_attention_backend`、`tests.utils.RemoteOpenAIServer`、`vllm.logger.init_logger`）。

### Module setup / 模块级配置: logger, SERVER_ARGS (L21-L23)
```python
logger = init_logger(__name__)

SERVER_ARGS = ["--enforce-eager"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `logger`, `SERVER_ARGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `logger`、`SERVER_ARGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _get_rocm_attention_config (L26-L51)
```python
def _get_rocm_attention_config(model_name):
    """Return appropriate ROCm attention config for the given model.

    Whisper uses cross-attention (ENCODER_DECODER) which ROCM_AITER_FA does
    not support. For Whisper we use ROCM_AITER_UNIFIED_ATTN (or TRITON_ATTN
    as fallback); other models can use ROCM_AITER_FA.
    """
    from vllm.platforms import current_platform

    if not current_platform.is_rocm():
        return None

    if "whisper" in model_name.lower():
        try:
            from vllm.platforms.rocm import _ON_MI3XX

            if _ON_MI3XX:
                return {"backend": "ROCM_AITER_UNIFIED_ATTN"}
        except ImportError:
            logger.warning(
                "Could not import _ON_MI3XX from rocm platform, "
                "falling back to TRITON_ATTN for Whisper."
            )
        return {"backend": "TRITON_ATTN"}

    return {"backend": "ROCM_AITER_FA"}
```
**EN:** This helper encapsulates reusable logic in `_get_rocm_attention_config`. Key inputs are `model_name`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_rocm_attention_config` 中。 关键输入包括 `model_name`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _get_server_args (L54-L58)
```python
def _get_server_args(attention_config):
    """Get server args with attention backend if specified."""
    args = SERVER_ARGS.copy()
    add_attention_backend(args, attention_config)
    return args
```
**EN:** This helper encapsulates reusable logic in `_get_server_args`. Key inputs are `attention_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_server_args` 中。 关键输入包括 `attention_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: server (L61-L70)
```python
@pytest.fixture(
    scope="module", params=["openai/whisper-small", "google/gemma-3n-E2B-it"]
)
def server(request):
    # Parametrize over model name
    attention_config = _get_rocm_attention_config(request.param)
    with RemoteOpenAIServer(
        request.param, _get_server_args(attention_config)
    ) as remote_server:
        yield remote_server, request.param
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client_and_model (L73-L77)
```python
@pytest_asyncio.fixture
async def client_and_model(server):
    server, model_name = server
    async with server.get_async_client() as async_client:
        yield async_client, model_name
```
**EN:** This async fixture prepares `client_and_model` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client_and_model`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_non_asr_model (L80-L93)
```python
@pytest.mark.asyncio
async def test_non_asr_model(foscolo):
    # text to text model
    model_name = "JackFram/llama-68m"
    attention_config = _get_rocm_attention_config(model_name)
    with RemoteOpenAIServer(
        model_name, _get_server_args(attention_config)
    ) as remote_server:
        client = remote_server.get_async_client()

        with pytest.raises(openai.NotFoundError):
            await client.audio.translations.create(
                model=model_name, file=foscolo, temperature=0.0
            )
```
**EN:** This async test validates `test_non_asr_model`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_non_asr_model`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_basic_audio_with_lora (L96-L134)
```python
@pytest.mark.asyncio
async def test_basic_audio_with_lora(mary_had_lamb):
    """Ensure STT (translate) requests can pass LoRA through to generate."""
    # ROCm SPECIFIC CONFIGURATION:
    # To ensure the test passes on ROCm, we modify the max model length to 512.
    # We DO NOT apply this to other platforms to maintain strict upstream parity.
    from vllm.platforms import current_platform

    # NOTE - careful to call this test before the module scoped server
    # fixture, otherwise it'll OOMkill the CI
    model_name = "ibm-granite/granite-speech-3.3-2b"
    lora_model_name = "speech"
    server_args = [
        "--enforce-eager",
        "--enable-lora",
        "--max-lora-rank",
        "64",
        "--lora-modules",
# ... 13 lines omitted for brevity ...
            model=lora_model_name,
            file=mary_had_lamb,
            extra_body=dict(language="en", to_language="es"),
            response_format="text",
            temperature=0.0,
        )
    out = json.loads(translation)["text"].strip().lower()
    assert "pequeño" in out.split(" ")
```
**EN:** This async test validates `test_basic_audio_with_lora`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. It drives client-facing request creation through the API surface under test. The main assertion is `'pequeño' in out.split(' ')`.
**CN:** 这个异步测试验证 `test_basic_audio_with_lora`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'pequeño' in out.split(' ')`。

### Test / 测试: test_basic_audio (L138-L150)
```python
@pytest.mark.asyncio
async def test_basic_audio(foscolo, client_and_model):
    client, model_name = client_and_model
    translation = await client.audio.translations.create(
        model=model_name,
        file=foscolo,
        response_format="text",
        # TODO remove `language="it"` once language detection is implemented
        extra_body=dict(language="it", to_language="en"),
        temperature=0.0,
    )
    out = json.loads(translation)["text"].strip().lower()
    assert "greek sea" in out
```
**EN:** This async test validates `test_basic_audio`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `client_and_model`. It drives client-facing request creation through the API surface under test. The main assertion is `'greek sea' in out`.
**CN:** 这个异步测试验证 `test_basic_audio`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`client_and_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'greek sea' in out`。

### Test / 测试: test_audio_prompt (L153-L168)
```python
@pytest.mark.asyncio
async def test_audio_prompt(foscolo, client_and_model):
    client, model_name = client_and_model
    # Condition whisper on starting text
    prompt = "Nor have I ever"
    transcription = await client.audio.translations.create(
        model=model_name,
        file=foscolo,
        prompt=prompt,
        extra_body=dict(language="it", to_language="en"),
        response_format="text",
        temperature=0.0,
    )
    out = json.loads(transcription)["text"]
    assert "Nor will I ever touch the sacred" not in out
    assert prompt not in out
```
**EN:** This async test validates `test_audio_prompt`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `client_and_model`. It drives client-facing request creation through the API surface under test. The main assertion is `'Nor will I ever touch the sacred' not in out` and `prompt not in out`.
**CN:** 这个异步测试验证 `test_audio_prompt`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`client_and_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'Nor will I ever touch the sacred' not in out` and `prompt not in out`。

### Test / 测试: test_streaming_response (L171-L219)
```python
@pytest.mark.asyncio
async def test_streaming_response(foscolo, client_and_model, server):
    client, model_name = client_and_model
    translation = ""
    res_no_stream = await client.audio.translations.create(
        model=model_name,
        file=foscolo,
        response_format="json",
        extra_body=dict(language="it", to_language="en", seed=42),
        temperature=0.0,
    )

    # Stream via HTTPX since OpenAI translation client doesn't expose streaming
    server, model_name = server
    url = server.url_for("v1/audio/translations")
    headers = {"Authorization": f"Bearer {server.DUMMY_API_KEY}"}
    data = {
        "model": model_name,
# ... 23 lines omitted for brevity ...
    res_stream = translation.split()
    # NOTE There's a small non-deterministic issue here, likely in the attn
    # computation, which will cause a few tokens to be different, while still
    # being very close semantically.
    assert (
        sum([x == y for x, y in zip(res_stream, res_no_stream.text.split())])
        >= len(res_stream) * 0.87
    )
```
**EN:** This async test validates `test_streaming_response`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `client_and_model`, `server`. It drives client-facing request creation through the API surface under test. The main assertion is `sum([x == y for x, y in zip(res_stream, res_no_stream.text.split())]) >= len(res_stream) * 0.87`.
**CN:** 这个异步测试验证 `test_streaming_response`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`client_and_model`、`server`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `sum([x == y for x, y in zip(res_stream, res_no_stream.text.split())]) >= len(res_stream) * 0.87`。

### Test / 测试: test_stream_options (L222-L258)
```python
@pytest.mark.asyncio
async def test_stream_options(foscolo, server):
    server, model_name = server
    url = server.url_for("v1/audio/translations")
    headers = {"Authorization": f"Bearer {server.DUMMY_API_KEY}"}
    data = {
        "model": model_name,
        "language": "it",
        "to_language": "en",
        "stream": True,
        "stream_include_usage": True,
        "stream_continuous_usage_stats": True,
        "temperature": 0.0,
    }
    foscolo.seek(0)
    final = False
    continuous = True
    async with httpx.AsyncClient() as http_client:
# ... 11 lines omitted for brevity ...
                chunk = json.loads(line)
                choices = chunk.get("choices", [])
                if not choices:
                    # final usage sent
                    final = True
                else:
                    continuous = continuous and ("usage" in chunk)
    assert final and continuous
```
**EN:** This async test validates `test_stream_options`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `server`. The main assertion is `final and continuous`.
**CN:** 这个异步测试验证 `test_stream_options`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`server`。 核心断言是 `final and continuous`。

### Test / 测试: test_long_audio_request (L261-L281)
```python
@pytest.mark.asyncio
async def test_long_audio_request(foscolo, client_and_model):
    client, model_name = client_and_model
    if model_name == "google/gemma-3n-E2B-it":
        pytest.skip("Gemma3n does not support long audio requests")
    foscolo.seek(0)
    audio, sr = load_audio(foscolo)
    repeated_audio = np.tile(audio, 2)
    # Repeated audio to buffer
    buffer = io.BytesIO()
    sf.write(buffer, repeated_audio, sr, format="WAV")
    buffer.seek(0)
    translation = await client.audio.translations.create(
        model=model_name,
        file=buffer,
        extra_body=dict(language="it", to_language="en"),
        response_format="text",
        temperature=0.0,
    )
    out = json.loads(translation)["text"].strip().lower()
    assert out.count("greek sea") == 2
```
**EN:** This async test validates `test_long_audio_request`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `client_and_model`. It drives client-facing request creation through the API surface under test. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `out.count('greek sea') == 2`.
**CN:** 这个异步测试验证 `test_long_audio_request`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`client_and_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `out.count('greek sea') == 2`。

### Test / 测试: test_audio_with_max_tokens (L284-L318)
```python
@pytest.mark.asyncio
async def test_audio_with_max_tokens(mary_had_lamb, client_and_model):
    client, model_name = client_and_model
    transcription = await client.audio.translations.create(
        model=model_name,
        file=mary_had_lamb,
        response_format="text",
        temperature=0.0,
        extra_body={"max_completion_tokens": 1},
    )
    out = json.loads(transcription)
    out_text = out["text"]
    print(out_text)
    from transformers import AutoTokenizer

    tok = AutoTokenizer.from_pretrained(model_name)
    out_tokens = tok(out_text, add_special_tokens=False)["input_ids"]
    assert len(out_tokens) == 1
# ... 9 lines omitted for brevity ...
            "repetition_penalty": 1.3,
        },
    )
    out = json.loads(transcription)
    out_text = out["text"]
    print(out_text)
    out_tokens = tok(out_text, add_special_tokens=False)["input_ids"]
    assert len(out_tokens) < 450  # ~Whisper max output len
```
**EN:** This async test validates `test_audio_with_max_tokens`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `client_and_model`. It drives client-facing request creation through the API surface under test. The main assertion is `len(out_tokens) == 1` and `len(out_tokens) < 450`.
**CN:** 这个异步测试验证 `test_audio_with_max_tokens`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`client_and_model`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(out_tokens) == 1` and `len(out_tokens) < 450`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `io`, `json`
- **Third-party / 第三方**: `httpx`, `numpy`, `openai`, `pytest`, `pytest_asyncio`, `soundfile`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.RemoteOpenAIServer`, `vllm.logger.init_logger`, `vllm.multimodal.media.audio.load_audio`, `vllm.platforms.current_platform`, `vllm.platforms.rocm._ON_MI3XX`
