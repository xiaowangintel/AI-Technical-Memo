# test_audio_in_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_audio_in_video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 3 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 3 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import json

import openai
import pybase64 as base64
import pytest
import pytest_asyncio

from tests.conftest import VideoTestAssets
from tests.utils import ROCM_EXTRA_ARGS, RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pybase64`, `pytest`, project helpers such as `tests.conftest.VideoTestAssets`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.conftest.VideoTestAssets`、`tests.utils.ROCM_EXTRA_ARGS`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L14-L14)
```python
MODEL_NAME = "Qwen/Qwen2.5-Omni-3B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L17-L32)
```python
@pytest.fixture
def server():
    args = [
        "--max-model-len",
        "16384",
        "--enforce-eager",
        "--limit-mm-per-prompt",
        json.dumps({"audio": 3, "video": 3}),
        *ROCM_EXTRA_ARGS,
    ]

    with RemoteOpenAIServer(
        MODEL_NAME,
        args,
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L35-L38)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_online_audio_in_video (L41-L88)
```python
@pytest.mark.core_model
@pytest.mark.asyncio
async def test_online_audio_in_video(
    client: openai.AsyncOpenAI, video_assets: VideoTestAssets
):
    """Test video input with `audio_in_video=True`"""

    # we don't use video_urls above because they missed audio stream.
    video_path = video_assets[0].video_path
    with open(video_path, "rb") as f:
        video_base64 = base64.b64encode(f.read()).decode("utf-8")

    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in this video?"},
                {
# ... 22 lines omitted for brevity ...
        choice = chat_completion.choices[0]
        print(
            f"[DEBUG][single-video] turn={turn} "
            f"finish_reason={choice.finish_reason!r} "
            f"content={choice.message.content!r} "
            f"usage={chat_completion.usage}"
        )
        assert choice.finish_reason == "length"
```
**EN:** This async test validates `test_online_audio_in_video`. Relevant pytest markers include `core_model`, `asyncio`. Key inputs are `client`, `video_assets`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_online_audio_in_video`。 相关的 pytest 标记包括 `core_model`、`asyncio`。 关键输入包括 `client`、`video_assets`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_online_audio_in_video_multi_videos (L91-L142)
```python
@pytest.mark.core_model
@pytest.mark.asyncio
async def test_online_audio_in_video_multi_videos(
    client: openai.AsyncOpenAI, video_assets: VideoTestAssets
):
    """Test multi-video input with `audio_in_video=True`"""

    # we don't use video_urls above because they missed audio stream.
    video_path = video_assets[0].video_path
    with open(video_path, "rb") as f:
        video_base64 = base64.b64encode(f.read()).decode("utf-8")

    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in these two videos?"},
                {
# ... 26 lines omitted for brevity ...
        choice = chat_completion.choices[0]
        print(
            f"[DEBUG][multi-video] turn={turn} "
            f"finish_reason={choice.finish_reason!r} "
            f"content={choice.message.content!r} "
            f"usage={chat_completion.usage}"
        )
        assert choice.finish_reason == "length"
```
**EN:** This async test validates `test_online_audio_in_video_multi_videos`. Relevant pytest markers include `core_model`, `asyncio`. Key inputs are `client`, `video_assets`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_online_audio_in_video_multi_videos`。 相关的 pytest 标记包括 `core_model`、`asyncio`。 关键输入包括 `client`、`video_assets`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_online_audio_in_video_interleaved (L145-L190)
```python
@pytest.mark.core_model
@pytest.mark.asyncio
async def test_online_audio_in_video_interleaved(
    client: openai.AsyncOpenAI, video_assets: VideoTestAssets
):
    """Test interleaved video/audio input with `audio_in_video=True`"""

    # we don't use video_urls above because they missed audio stream.
    video_path = video_assets[0].video_path
    with open(video_path, "rb") as f:
        video_base64 = base64.b64encode(f.read()).decode("utf-8")

    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in these two videos?"},
                {
# ... 20 lines omitted for brevity ...
            messages=messages,
            max_tokens=16,
            extra_body={
                "mm_processor_kwargs": {
                    "use_audio_in_video": True,
                }
            },
        )
```
**EN:** This async test validates `test_online_audio_in_video_interleaved`. Relevant pytest markers include `core_model`, `asyncio`. Key inputs are `client`, `video_assets`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_online_audio_in_video_interleaved`。 相关的 pytest 标记包括 `core_model`、`asyncio`。 关键输入包括 `client`、`video_assets`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `pybase64`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.conftest.VideoTestAssets`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`
