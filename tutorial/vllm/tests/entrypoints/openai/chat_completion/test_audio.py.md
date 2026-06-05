# test_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_audio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 7 test(s), 4 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 7 个测试、4 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import json

import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
from vllm.assets.audio import AudioAsset
from vllm.multimodal.utils import encode_audio_base64, encode_audio_url, fetch_audio
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.assets.audio.AudioAsset`, `vllm.multimodal.utils.encode_audio_base64`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.assets.audio.AudioAsset`、`vllm.multimodal.utils.encode_audio_base64`）。

### Module setup / 模块级配置: MODEL_NAME, TEST_AUDIO_URLS, MAXIMUM_AUDIOS (L14-L19)
```python
MODEL_NAME = "fixie-ai/ultravox-v0_5-llama-3_2-1b"
TEST_AUDIO_URLS = [
    AudioAsset("winning_call").url,
    AudioAsset("mary_had_lamb").url,
]
MAXIMUM_AUDIOS = 2
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `TEST_AUDIO_URLS`, `MAXIMUM_AUDIOS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`TEST_AUDIO_URLS`、`MAXIMUM_AUDIOS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L22-L38)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--dtype",
        "float32",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "5",
        "--enforce-eager",
        "--trust-remote-code",
        "--limit-mm-per-prompt",
        json.dumps({"audio": MAXIMUM_AUDIOS}),
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L41-L44)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: base64_encoded_audio (L47-L52)
```python
@pytest.fixture(scope="session")
def base64_encoded_audio() -> dict[str, str]:
    return {
        audio_url: encode_audio_base64(*fetch_audio(audio_url))
        for audio_url in TEST_AUDIO_URLS
    }
```
**EN:** This fixture prepares `base64_encoded_audio` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `base64_encoded_audio`。

### Fixture / 夹具: url_encoded_audio (L55-L60)
```python
@pytest.fixture(scope="session")
def url_encoded_audio() -> dict[str, str]:
    return {
        audio_url: encode_audio_url(*fetch_audio(audio_url))
        for audio_url in TEST_AUDIO_URLS
    }
```
**EN:** This fixture prepares `url_encoded_audio` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `url_encoded_audio`。

### Helper / 辅助函数: dummy_messages_from_audio_url (L63-L81)
```python
def dummy_messages_from_audio_url(
    audio_urls: str | list[str],
    content_text: str = "What's happening in this audio?",
):
    if isinstance(audio_urls, str):
        audio_urls = [audio_urls]

    return [
        {
            "role": "user",
            "content": [
                *(
                    {"type": "audio_url", "audio_url": {"url": audio_url}}
                    for audio_url in audio_urls
                ),
                {"type": "text", "text": content_text},
            ],
        }
    ]
```
**EN:** This helper encapsulates reusable logic in `dummy_messages_from_audio_url`. Key inputs are `audio_urls`, `content_text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `dummy_messages_from_audio_url` 中。 关键输入包括 `audio_urls`、`content_text`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_single_chat_session_audio (L84-L123)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", [TEST_AUDIO_URLS[0]])
async def test_single_chat_session_audio(
    client: openai.AsyncOpenAI, model_name: str, audio_url: str
):
    messages = dummy_messages_from_audio_url(audio_url)

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        logprobs=True,
        temperature=0.0,
        top_logprobs=5,
    )
    assert len(chat_completion.choices) == 1
# ... 14 lines omitted for brevity ...
    messages.append({"role": "user", "content": "express your result in json"})
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
    )
    message = chat_completion.choices[0].message
    assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_single_chat_session_audio`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_audio`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_error_on_invalid_audio_url_type (L126-L149)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", [TEST_AUDIO_URLS[0]])
async def test_error_on_invalid_audio_url_type(
    client: openai.AsyncOpenAI, model_name: str, audio_url: str
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "audio_url", "audio_url": audio_url},
                {"type": "text", "text": "What's happening in this audio?"},
            ],
        }
    ]

    # audio_url should be a dict {"url": "some url"}, not directly a string
    with pytest.raises(openai.BadRequestError):
        _ = await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            temperature=0.0,
        )
```
**EN:** This async test validates `test_error_on_invalid_audio_url_type`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_error_on_invalid_audio_url_type`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_single_chat_session_audio_base64encoded (L152-L195)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", [TEST_AUDIO_URLS[0]])
async def test_single_chat_session_audio_base64encoded(
    client: openai.AsyncOpenAI,
    model_name: str,
    audio_url: str,
    url_encoded_audio: dict[str, str],
):
    messages = dummy_messages_from_audio_url(url_encoded_audio[audio_url])

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        logprobs=True,
        temperature=0.0,
# ... 18 lines omitted for brevity ...
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.0,
    )
    message = chat_completion.choices[0].message
    assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_single_chat_session_audio_base64encoded`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`, `url_encoded_audio`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_audio_base64encoded`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`、`url_encoded_audio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_single_chat_session_input_audio (L198-L253)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", [TEST_AUDIO_URLS[0]])
async def test_single_chat_session_input_audio(
    client: openai.AsyncOpenAI,
    model_name: str,
    audio_url: str,
    base64_encoded_audio: dict[str, str],
):
    messages = [
        {
            "role": "user",
            "content": [
                {
                    "type": "input_audio",
                    "input_audio": {
                        "data": base64_encoded_audio[audio_url],
                        "format": "wav",
# ... 30 lines omitted for brevity ...
    messages.append({"role": "user", "content": "express your result in json"})
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
    )
    message = chat_completion.choices[0].message
    assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_single_chat_session_input_audio`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`, `base64_encoded_audio`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_input_audio`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`、`base64_encoded_audio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_chat_streaming_audio (L256-L298)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", TEST_AUDIO_URLS)
async def test_chat_streaming_audio(
    client: openai.AsyncOpenAI, model_name: str, audio_url: str
):
    messages = dummy_messages_from_audio_url(
        audio_url, "What's a short title for this audio?"
    )

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=8,
        temperature=0.0,
    )
    output = chat_completion.choices[0].message.content
# ... 17 lines omitted for brevity ...
            chunks.append(delta.content)
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    assert chunk.choices[0].finish_reason == stop_reason
    assert delta.content
    assert "".join(chunks) == output
```
**EN:** This async test validates `test_chat_streaming_audio`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`.
**CN:** 这个异步测试验证 `test_chat_streaming_audio`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`。

### Test / 测试: test_chat_streaming_input_audio (L301-L358)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("audio_url", TEST_AUDIO_URLS)
async def test_chat_streaming_input_audio(
    client: openai.AsyncOpenAI,
    model_name: str,
    audio_url: str,
    base64_encoded_audio: dict[str, str],
):
    messages = [
        {
            "role": "user",
            "content": [
                {
                    "type": "input_audio",
                    "input_audio": {
                        "data": base64_encoded_audio[audio_url],
                        "format": "wav",
# ... 32 lines omitted for brevity ...
            chunks.append(delta.content)
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    assert chunk.choices[0].finish_reason == stop_reason
    assert delta.content
    assert "".join(chunks) == output
```
**EN:** This async test validates `test_chat_streaming_input_audio`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_url`, `base64_encoded_audio`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`.
**CN:** 这个异步测试验证 `test_chat_streaming_input_audio`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_url`、`base64_encoded_audio`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`。

### Test / 测试: test_multi_audio_input (L361-L397)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "audio_urls", [TEST_AUDIO_URLS, TEST_AUDIO_URLS + [TEST_AUDIO_URLS[0]]]
)
async def test_multi_audio_input(
    client: openai.AsyncOpenAI, model_name: str, audio_urls: list[str]
):
    messages = dummy_messages_from_audio_url(audio_urls)

    if len(audio_urls) > MAXIMUM_AUDIOS:
        with pytest.raises(openai.BadRequestError):  # test multi-audio input
            await client.chat.completions.create(
                model=model_name,
                messages=messages,
                max_completion_tokens=10,
                temperature=0.0,
            )
# ... 11 lines omitted for brevity ...
        chat_completion = await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            temperature=0.0,
        )
        message = chat_completion.choices[0].message
        assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_multi_audio_input`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `audio_urls`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `completion is not None and len(completion) >= 0` and `message.content is not None and len(message.content) >= 0`.
**CN:** 这个异步测试验证 `test_multi_audio_input`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`audio_urls`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion is not None and len(completion) >= 0` and `message.content is not None and len(message.content) >= 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.assets.audio.AudioAsset`, `vllm.multimodal.utils.encode_audio_base64`, `vllm.multimodal.utils.encode_audio_url`, `vllm.multimodal.utils.fetch_audio`
