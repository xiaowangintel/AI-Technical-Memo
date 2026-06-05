# test_realtime_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/realtime/test_realtime_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 4 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 4 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import asyncio
import json
import warnings

import numpy as np
import pybase64 as base64
import pytest
import websockets

from tests.entrypoints.speech_to_text.conftest import add_attention_backend
from tests.utils import ROCM_ENV_OVERRIDES, ROCM_EXTRA_ARGS, RemoteOpenAIServer
from vllm.assets.audio import AudioAsset
from vllm.multimodal.media.audio import load_audio
```
**EN:** Imports standard-library modules such as `asyncio`, `json`, `warnings`, third-party packages like `numpy`, `pybase64`, `pytest`, project helpers such as `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`.
**CN:** 导入标准库模块（如 `asyncio`、`json`、`warnings`）、第三方包（如 `numpy`、`pybase64`、`pytest`）、项目内辅助模块（如 `tests.entrypoints.speech_to_text.conftest.add_attention_backend`、`tests.utils.ROCM_ENV_OVERRIDES`、`tests.utils.ROCM_EXTRA_ARGS`）。

### Module setup / 模块级配置: REALTIME_ENV_OVERRIDES, MISTRAL_FORMAT_ARGS, MODEL_NAME (L20-L34)
```python
REALTIME_ENV_OVERRIDES = {
    **ROCM_ENV_OVERRIDES,
    "VLLM_ENGINE_ITERATION_TIMEOUT_S": "600",
}

MISTRAL_FORMAT_ARGS = [
    "--tokenizer_mode",
    "mistral",
    "--config_format",
    "mistral",
    "--load_format",
    "mistral",
] + ROCM_EXTRA_ARGS

MODEL_NAME = "mistralai/Voxtral-Mini-4B-Realtime-2602"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `REALTIME_ENV_OVERRIDES`, `MISTRAL_FORMAT_ARGS`, `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `REALTIME_ENV_OVERRIDES`、`MISTRAL_FORMAT_ARGS`、`MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _get_websocket_url (L37-L41)
```python
def _get_websocket_url(server: RemoteOpenAIServer) -> str:
    """Convert HTTP URL to WebSocket URL for realtime endpoint."""
    http_url = server.url_root
    ws_url = http_url.replace("http://", "ws://")
    return f"{ws_url}/v1/realtime"
```
**EN:** This helper encapsulates reusable logic in `_get_websocket_url`. Key inputs are `server`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_websocket_url` 中。 关键输入包括 `server`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: receive_event (L44-L47)
```python
async def receive_event(ws, timeout: float = 60.0) -> dict:
    """Receive and parse JSON event from WebSocket."""
    message = await asyncio.wait_for(ws.recv(), timeout=timeout)
    return json.loads(message)
```
**EN:** This async helper encapsulates reusable logic in `receive_event`. Key inputs are `ws`, `timeout`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `receive_event` 中。 关键输入包括 `ws`、`timeout`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: send_event (L50-L52)
```python
async def send_event(ws, event: dict) -> None:
    """Send JSON event to WebSocket."""
    await ws.send(json.dumps(event))
```
**EN:** This async helper encapsulates reusable logic in `send_event`. Key inputs are `ws`, `event`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `send_event` 中。 关键输入包括 `ws`、`event`。

### Fixture / 夹具: mary_had_lamb_audio_chunks (L55-L70)
```python
@pytest.fixture
def mary_had_lamb_audio_chunks() -> list[str]:
    """Audio split into ~1 second chunks for streaming."""
    path = AudioAsset("mary_had_lamb").get_local_path()
    audio, _ = load_audio(str(path), sr=16000, mono=True)

    # Split into ~0.1 second chunks (1600 samples at 16kHz)
    chunk_size = 1600
    chunks = []
    for i in range(0, len(audio), chunk_size):
        chunk = audio[i : i + chunk_size]
        chunk_int16 = (chunk * 32767).astype(np.int16)
        chunk_bytes = chunk_int16.tobytes()
        chunks.append(base64.b64encode(chunk_bytes).decode("utf-8"))

    return chunks
```
**EN:** This fixture prepares `mary_had_lamb_audio_chunks` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `mary_had_lamb_audio_chunks`。

### Test / 测试: test_multi_chunk_streaming (L73-L171)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_multi_chunk_streaming(
    model_name, mary_had_lamb_audio_chunks, rocm_aiter_fa_attention
):
    """Test streaming multiple audio chunks before committing."""
    server_args = ["--enforce-eager", "--max-model-len", "2048"]

    if model_name.startswith("mistralai"):
        server_args += MISTRAL_FORMAT_ARGS

    add_attention_backend(server_args, rocm_aiter_fa_attention)

    with RemoteOpenAIServer(
        model_name, server_args, env_dict=REALTIME_ENV_OVERRIDES
    ) as remote_server:
        ws_url = _get_websocket_url(remote_server)
        async with websockets.connect(ws_url) as ws:
# ... 73 lines omitted for brevity ...
                " it sleeps with quite a flow, and everywhere that Mary went,"
                " the lamb was sure to go."
            ) or full_text == (
                " First words I spoke in the original phonograph."
                " A little piece of practical poetry. Mary had a little lamb,"
                " it squeaked with quite a flow, and everywhere that Mary went,"
                " the lamb was sure to go."
            )
```
**EN:** This async test validates `test_multi_chunk_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `model_name`, `mary_had_lamb_audio_chunks`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `event['type'] == 'session.created'` and `event['type'] == 'transcription.done'`.
**CN:** 这个异步测试验证 `test_multi_chunk_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_name`、`mary_had_lamb_audio_chunks`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `event['type'] == 'session.created'` and `event['type'] == 'transcription.done'`。

### Test / 测试: test_empty_commit_does_not_crash_engine (L174-L271)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_empty_commit_does_not_crash_engine(
    model_name, mary_had_lamb_audio_chunks, rocm_aiter_fa_attention
):
    """Test that committing without audio does not crash the engine.

    Regression test for https://github.com/vllm-project/vllm/issues/34532.
    An empty commit (no prior input_audio_buffer.append) used to trigger
    ``AssertionError: For realtime you must provide a multimodal_embedding
    at every step`` which killed the entire engine process, disconnecting
    every connected client.
    """
    server_args = ["--enforce-eager", "--max-model-len", "2048"]

    if model_name.startswith("mistralai"):
        server_args += MISTRAL_FORMAT_ARGS

# ... 72 lines omitted for brevity ...
            done_received = False
            while not done_received:
                event = await receive_event(ws, timeout=60.0)
                if event["type"] == "transcription.done":
                    done_received = True
                elif event["type"] == "error":
                    pytest.fail(f"Engine error after empty commit: {event}")
            assert done_received
```
**EN:** This async test validates `test_empty_commit_does_not_crash_engine`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `model_name`, `mary_had_lamb_audio_chunks`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `event['type'] == 'session.created'` and `event['type'] in ('error', 'transcription.done', 'transcription.delta')`.
**CN:** 这个异步测试验证 `test_empty_commit_does_not_crash_engine`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_name`、`mary_had_lamb_audio_chunks`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `event['type'] == 'session.created'` and `event['type'] in ('error', 'transcription.done', 'transcription.delta')`。

### Test / 测试: test_session_update_invalid_model_returns_error (L274-L303)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_session_update_invalid_model_returns_error(
    model_name, rocm_aiter_fa_attention
):
    """Test that session.update with an invalid model returns an error."""
    server_args = ["--enforce-eager", "--max-model-len", "2048"]

    if model_name.startswith("mistralai"):
        server_args += MISTRAL_FORMAT_ARGS

    add_attention_backend(server_args, rocm_aiter_fa_attention)

    with RemoteOpenAIServer(
        model_name, server_args, env_dict=REALTIME_ENV_OVERRIDES
    ) as remote_server:
        ws_url = _get_websocket_url(remote_server)
        async with websockets.connect(ws_url) as ws:
            event = await receive_event(ws, timeout=30.0)
            assert event["type"] == "session.created"

            # Send session.update with a model that doesn't exist
            await send_event(
                ws,
                {"type": "session.update", "model": "nonexistent-model"},
            )

            event = await receive_event(ws, timeout=10.0)
            assert event["type"] == "error"
            assert "nonexistent-model" in event["error"]
```
**EN:** This async test validates `test_session_update_invalid_model_returns_error`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `model_name`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `event['type'] == 'session.created'` and `event['type'] == 'error'`.
**CN:** 这个异步测试验证 `test_session_update_invalid_model_returns_error`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_name`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `event['type'] == 'session.created'` and `event['type'] == 'error'`。

### Test / 测试: test_commit_without_session_update_returns_error (L306-L336)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_commit_without_session_update_returns_error(
    model_name, rocm_aiter_fa_attention
):
    """Test that committing before validating the model returns an error
    and does not fall through to processing."""
    server_args = ["--enforce-eager", "--max-model-len", "2048"]

    if model_name.startswith("mistralai"):
        server_args += MISTRAL_FORMAT_ARGS

    add_attention_backend(server_args, rocm_aiter_fa_attention)

    with RemoteOpenAIServer(
        model_name, server_args, env_dict=REALTIME_ENV_OVERRIDES
    ) as remote_server:
        ws_url = _get_websocket_url(remote_server)
# ... 5 lines omitted for brevity ...
            await send_event(
                ws,
                {"type": "input_audio_buffer.commit", "final": True},
            )

            event = await receive_event(ws, timeout=10.0)
            assert event["type"] == "error"
            assert "model_not_validated" in event.get("code", "")
```
**EN:** This async test validates `test_commit_without_session_update_returns_error`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `model_name`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `event['type'] == 'session.created'` and `event['type'] == 'error'`.
**CN:** 这个异步测试验证 `test_commit_without_session_update_returns_error`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `model_name`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `event['type'] == 'session.created'` and `event['type'] == 'error'`。

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
- **Stdlib / 标准库**: `asyncio`, `json`, `warnings`
- **Third-party / 第三方**: `numpy`, `pybase64`, `pytest`, `websockets`
- **Project / 项目内**: `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`, `vllm.assets.audio.AudioAsset`, `vllm.multimodal.media.audio.load_audio`
