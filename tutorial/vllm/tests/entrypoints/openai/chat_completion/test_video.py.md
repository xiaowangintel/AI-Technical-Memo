# test_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 9 test(s), 3 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 9 个测试、3 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import json

import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
from vllm.multimodal.utils import encode_video_url, fetch_video
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.utils.encode_video_url`, `vllm.multimodal.utils.fetch_video`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.multimodal.utils.encode_video_url`、`vllm.multimodal.utils.fetch_video`）。

### Module setup / 模块级配置: MODEL_NAME, MAXIMUM_VIDEOS, TEST_VIDEO_URLS (L14-L21)
```python
MODEL_NAME = "llava-hf/llava-onevision-qwen2-0.5b-ov-hf"
MAXIMUM_VIDEOS = 3

TEST_VIDEO_URLS = [
    "https://www.bogotobogo.com/python/OpenCV_Python/images/mean_shift_tracking/slow_traffic_small.mp4",
    "https://github.com/opencv/opencv/raw/refs/tags/4.12.0/samples/data/vtest.avi",
    "https://github.com/opencv/opencv/raw/refs/tags/4.12.0/samples/data/Megamind.avi",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAXIMUM_VIDEOS`, `TEST_VIDEO_URLS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAXIMUM_VIDEOS`、`TEST_VIDEO_URLS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L24-L51)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--runner",
        "generate",
        "--max-model-len",
        "32768",
        "--max-num-seqs",
        "2",
        "--enforce-eager",
        "--trust-remote-code",
        "--limit-mm-per-prompt",
        json.dumps({"video": MAXIMUM_VIDEOS}),
        "--media-io-kwargs",
        json.dumps({"video": {"num_frames": 32}}),
    ]

    # ROCm: Increase timeouts to handle potential network delays and slower
    # video processing when downloading multiple videos from external sources
    env_overrides = {}
    if current_platform.is_rocm():
        env_overrides = {
            "VLLM_VIDEO_FETCH_TIMEOUT": "120",
            "VLLM_ENGINE_ITERATION_TIMEOUT_S": "300",
        }

    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_overrides) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L54-L57)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: url_encoded_video (L60-L65)
```python
@pytest.fixture(scope="session")
def url_encoded_video() -> dict[str, str]:
    return {
        video_url: encode_video_url(fetch_video(video_url)[0])
        for video_url in TEST_VIDEO_URLS
    }
```
**EN:** This fixture prepares `url_encoded_video` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `url_encoded_video`。

### Helper / 辅助函数: dummy_messages_from_video_url (L68-L86)
```python
def dummy_messages_from_video_url(
    video_urls: str | list[str],
    content_text: str = "What's in this video?",
):
    if isinstance(video_urls, str):
        video_urls = [video_urls]

    return [
        {
            "role": "user",
            "content": [
                *(
                    {"type": "video_url", "video_url": {"url": video_url}}
                    for video_url in video_urls
                ),
                {"type": "text", "text": content_text},
            ],
        }
    ]
```
**EN:** This helper encapsulates reusable logic in `dummy_messages_from_video_url`. Key inputs are `video_urls`, `content_text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `dummy_messages_from_video_url` 中。 关键输入包括 `video_urls`、`content_text`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_single_chat_session_video (L89-L128)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_single_chat_session_video(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = dummy_messages_from_video_url(video_url)

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
**EN:** This async test validates `test_single_chat_session_video`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_video`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_request_media_io_kwargs_override_uses_fewer_video_frames (L131-L161)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", [TEST_VIDEO_URLS[0]])
async def test_request_media_io_kwargs_override_uses_fewer_video_frames(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = dummy_messages_from_video_url(video_url)

    default_resp = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=1,
        temperature=0.0,
    )
    override_resp = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=1,
# ... 5 lines omitted for brevity ...
                }
            }
        },
    )

    assert default_resp.usage is not None
    assert override_resp.usage is not None
    assert override_resp.usage.prompt_tokens < default_resp.usage.prompt_tokens
```
**EN:** This async test validates `test_request_media_io_kwargs_override_uses_fewer_video_frames`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It drives client-facing request creation through the API surface under test. The main assertion is `default_resp.usage is not None` and `override_resp.usage is not None`.
**CN:** 这个异步测试验证 `test_request_media_io_kwargs_override_uses_fewer_video_frames`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `default_resp.usage is not None` and `override_resp.usage is not None`。

### Test / 测试: test_invalid_num_frames_request_recoverable (L164-L195)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", [TEST_VIDEO_URLS[0]])
async def test_invalid_num_frames_request_recoverable(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = dummy_messages_from_video_url(video_url)

    with pytest.raises((openai.BadRequestError, openai.APIStatusError)):
        await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=1,
            temperature=0.0,
            extra_body={
                "media_io_kwargs": {
                    "video": {
                        "num_frames": "invalid",
# ... 6 lines omitted for brevity ...
    recovery_resp = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=1,
        temperature=0.0,
    )
    recovery_msg = recovery_resp.choices[0].message
    assert recovery_msg.content is not None and len(recovery_msg.content) >= 0
```
**EN:** This async test validates `test_invalid_num_frames_request_recoverable`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `recovery_msg.content is not None and len(recovery_msg.content) >= 0`.
**CN:** 这个异步测试验证 `test_invalid_num_frames_request_recoverable`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `recovery_msg.content is not None and len(recovery_msg.content) >= 0`。

### Test / 测试: test_error_on_invalid_video_url_type (L198-L221)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_error_on_invalid_video_url_type(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "video_url", "video_url": video_url},
                {"type": "text", "text": "What's in this video?"},
            ],
        }
    ]

    # video_url should be a dict {"url": "some url"}, not directly a string
    with pytest.raises(openai.BadRequestError):
        _ = await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            temperature=0.0,
        )
```
**EN:** This async test validates `test_error_on_invalid_video_url_type`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_error_on_invalid_video_url_type`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_single_chat_session_video_beamsearch (L224-L245)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_single_chat_session_video_beamsearch(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = dummy_messages_from_video_url(video_url)

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        n=2,
        max_completion_tokens=10,
        logprobs=True,
        top_logprobs=5,
        extra_body=dict(use_beam_search=True),
    )
    assert len(chat_completion.choices) == 2
    assert (
        chat_completion.choices[0].message.content
        != chat_completion.choices[1].message.content
    )
```
**EN:** This async test validates `test_single_chat_session_video_beamsearch`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 2` and `chat_completion.choices[0].message.content != chat_completion.choices[1].message.content`.
**CN:** 这个异步测试验证 `test_single_chat_session_video_beamsearch`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 2` and `chat_completion.choices[0].message.content != chat_completion.choices[1].message.content`。

### Test / 测试: test_single_chat_session_video_base64encoded (L248-L291)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_single_chat_session_video_base64encoded(
    client: openai.AsyncOpenAI,
    model_name: str,
    video_url: str,
    url_encoded_video: dict[str, str],
):
    messages = dummy_messages_from_video_url(url_encoded_video[video_url])

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
**EN:** This async test validates `test_single_chat_session_video_base64encoded`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`, `url_encoded_video`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_video_base64encoded`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`、`url_encoded_video`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_single_chat_session_video_base64encoded_beamsearch (L294-L316)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_single_chat_session_video_base64encoded_beamsearch(
    client: openai.AsyncOpenAI,
    model_name: str,
    video_url: str,
    url_encoded_video: dict[str, str],
):
    messages = dummy_messages_from_video_url(url_encoded_video[video_url])

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        n=2,
        max_completion_tokens=10,
        extra_body=dict(use_beam_search=True),
    )
    assert len(chat_completion.choices) == 2
    assert (
        chat_completion.choices[0].message.content
        != chat_completion.choices[1].message.content
    )
```
**EN:** This async test validates `test_single_chat_session_video_base64encoded_beamsearch`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`, `url_encoded_video`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 2` and `chat_completion.choices[0].message.content != chat_completion.choices[1].message.content`.
**CN:** 这个异步测试验证 `test_single_chat_session_video_base64encoded_beamsearch`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`、`url_encoded_video`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 2` and `chat_completion.choices[0].message.content != chat_completion.choices[1].message.content`。

### Test / 测试: test_chat_streaming_video (L319-L359)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("video_url", TEST_VIDEO_URLS)
async def test_chat_streaming_video(
    client: openai.AsyncOpenAI, model_name: str, video_url: str
):
    messages = dummy_messages_from_video_url(video_url)

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.0,
    )
    output = chat_completion.choices[0].message.content
    stop_reason = chat_completion.choices[0].finish_reason

# ... 15 lines omitted for brevity ...
            chunks.append(delta.content)
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    assert chunk.choices[0].finish_reason == stop_reason
    assert delta.content
    assert "".join(chunks) == output
```
**EN:** This async test validates `test_chat_streaming_video`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `video_url`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`.
**CN:** 这个异步测试验证 `test_chat_streaming_video`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`video_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`。

### Test / 测试: test_multi_video_input (L362-L403)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "video_urls", [TEST_VIDEO_URLS[:i] for i in range(2, len(TEST_VIDEO_URLS))]
)
@pytest.mark.flaky(
    reruns=2,
    reruns_delay=5,
    condition=current_platform.is_rocm(),
)
async def test_multi_video_input(
    client: openai.AsyncOpenAI, model_name: str, video_urls: list[str]
):
    messages = dummy_messages_from_video_url(video_urls)

    if len(video_urls) > MAXIMUM_VIDEOS:
        with pytest.raises(openai.BadRequestError):  # test multi-video input
            await client.chat.completions.create(
# ... 16 lines omitted for brevity ...
        chat_completion = await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            temperature=0.0,
        )
        message = chat_completion.choices[0].message
        assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_multi_video_input`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`, `flaky`. Key inputs are `client`, `model_name`, `video_urls`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `completion is not None and len(completion) >= 0` and `message.content is not None and len(message.content) >= 0`.
**CN:** 这个异步测试验证 `test_multi_video_input`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`、`flaky`。 关键输入包括 `client`、`model_name`、`video_urls`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion is not None and len(completion) >= 0` and `message.content is not None and len(message.content) >= 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.utils.encode_video_url`, `vllm.multimodal.utils.fetch_video`, `vllm.platforms.current_platform`
