# test_basic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/instrumentator/test_basic.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 6 test(s), 3 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 6 个测试、3 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import asyncio
from http import HTTPStatus
from unittest.mock import AsyncMock, Mock

import openai
import pytest
import pytest_asyncio
import requests
from fastapi import Request

from tests.utils import RemoteOpenAIServer
from vllm.v1.engine.exceptions import EngineDeadError
from vllm.version import __version__ as VLLM_VERSION
```
**EN:** Imports standard-library modules such as `asyncio`, `http.HTTPStatus`, `unittest.mock.AsyncMock`, third-party packages like `fastapi.Request`, `openai`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.serve.instrumentator.health.health`, `vllm.v1.engine.exceptions.EngineDeadError`.
**CN:** 导入标准库模块（如 `asyncio`、`http.HTTPStatus`、`unittest.mock.AsyncMock`）、第三方包（如 `fastapi.Request`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.serve.instrumentator.health.health`、`vllm.v1.engine.exceptions.EngineDeadError`）。

### Module setup / 模块级配置: MODEL_NAME (L18-L18)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server_args (L21-L54)
```python
@pytest.fixture(scope="module")
def server_args(request: pytest.FixtureRequest) -> list[str]:
    """Provide extra arguments to the server via indirect parametrization

    Usage:

    >>> @pytest.mark.parametrize(
    >>>     "server_args",
    >>>     [
    >>>         ["--max-model-len", "10100"],
    >>>         [
    >>>             "--model=NousResearch/Hermes-3-Llama-3.1-70B",
    >>>             "--enable-auto-tool-choice",
    >>>         ],
    >>>     ],
    >>>     indirect=True,
    >>> )
    >>> def test_foo(server, client):
# ... 8 lines omitted for brevity ...
        return []

    val = request.param

    if isinstance(val, str):
        return [val]

    return request.param
```
**EN:** This fixture prepares `server_args` for dependent tests. Key inputs are `request`.
**CN:** 这个 fixture 为依赖它的测试准备 `server_args`。 关键输入包括 `request`。

### Fixture / 夹具: server (L57-L72)
```python
@pytest.fixture(scope="module")
def server(server_args):
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "8192",
        "--enforce-eager",
        "--max-num-seqs",
        "128",
        *server_args,
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L75-L78)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_show_version (L81-L86)
```python
@pytest.mark.asyncio
async def test_show_version(server: RemoteOpenAIServer):
    response = requests.get(server.url_for("version"))
    response.raise_for_status()

    assert response.json() == {"version": VLLM_VERSION}
```
**EN:** This async test validates `test_show_version`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `response.json() == {'version': VLLM_VERSION}`.
**CN:** 这个异步测试验证 `test_show_version`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `response.json() == {'version': VLLM_VERSION}`。

### Test / 测试: test_check_health (L89-L93)
```python
@pytest.mark.asyncio
async def test_check_health(server: RemoteOpenAIServer):
    response = requests.get(server.url_for("health"))

    assert response.status_code == HTTPStatus.OK
```
**EN:** This async test validates `test_check_health`. Relevant pytest markers include `asyncio`. Key inputs are `server`. The main assertion is `response.status_code == HTTPStatus.OK`.
**CN:** 这个异步测试验证 `test_check_health`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 核心断言是 `response.status_code == HTTPStatus.OK`。

### Test / 测试: test_request_cancellation (L96-L141)
```python
@pytest.mark.parametrize(
    "server_args",
    [
        pytest.param(["--max-model-len", "10100"]),
    ],
    indirect=True,
)
@pytest.mark.asyncio
async def test_request_cancellation(server: RemoteOpenAIServer):
    # clunky test: send an ungodly amount of load in with short timeouts
    # then ensure that it still responds quickly afterwards

    chat_input = [{"role": "user", "content": "Write a long story"}]
    client = server.get_async_client(timeout=0.5)
    tasks = []
    # Request about 2 million tokens
    for _ in range(200):
        task = asyncio.create_task(
# ... 20 lines omitted for brevity ...
    # If the server had not cancelled all the other requests, then it would not
    # be able to respond to this one within the timeout
    client = server.get_async_client(timeout=5)
    response = await client.chat.completions.create(
        messages=chat_input, model=MODEL_NAME, max_tokens=10, temperature=0.0
    )

    assert len(response.choices) == 1
```
**EN:** This async test validates `test_request_cancellation`. It uses parameterization over `server_args`. Relevant pytest markers include `asyncio`. Key inputs are `server`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `len(pending) == 0` and `len(response.choices) == 1`.
**CN:** 这个异步测试验证 `test_request_cancellation`。 它通过参数化组合 `server_args`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(pending) == 0` and `len(response.choices) == 1`。

### Test / 测试: test_request_wrong_content_type (L144-L155)
```python
@pytest.mark.asyncio
async def test_request_wrong_content_type(server: RemoteOpenAIServer):
    chat_input = [{"role": "user", "content": "Write a long story"}]
    client = server.get_async_client()

    with pytest.raises(openai.APIStatusError):
        await client.chat.completions.create(
            messages=chat_input,
            model=MODEL_NAME,
            max_tokens=10000,
            extra_headers={"Content-Type": "application/x-www-form-urlencoded"},
        )
```
**EN:** This async test validates `test_request_wrong_content_type`. Relevant pytest markers include `asyncio`. Key inputs are `server`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_request_wrong_content_type`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_server_load (L158-L201)
```python
@pytest.mark.parametrize(
    "server_args",
    [pytest.param(["--enable-server-load-tracking"], id="enable-server-load-tracking")],
    indirect=True,
)
@pytest.mark.asyncio
async def test_server_load(server: RemoteOpenAIServer):
    # Check initial server load
    response = requests.get(server.url_for("load"))
    assert response.status_code == HTTPStatus.OK
    assert response.json().get("server_load") == 0

    def make_long_completion_request():
        return requests.post(
            server.url_for("v1/completions"),
            headers={"Content-Type": "application/json"},
            json={
                "prompt": "Give me a long story",
# ... 18 lines omitted for brevity ...
    # Wait for the completion request to finish.
    await completion_future
    await asyncio.sleep(0.1)

    # Check server load after the completion request has finished.
    response = requests.get(server.url_for("load"))
    assert response.status_code == HTTPStatus.OK
    assert response.json().get("server_load") == 0
```
**EN:** This async test validates `test_server_load`. It uses parameterization over `server_args`. Relevant pytest markers include `asyncio`. Key inputs are `server`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status_code == HTTPStatus.OK` and `response.json().get('server_load') == 0`.
**CN:** 这个异步测试验证 `test_server_load`。 它通过参数化组合 `server_args`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status_code == HTTPStatus.OK` and `response.json().get('server_load') == 0`。

### Test / 测试: test_health_check_engine_dead_error (L204-L222)
```python
@pytest.mark.asyncio
async def test_health_check_engine_dead_error():
    # Import the health function directly to test it in isolation
    from vllm.entrypoints.serve.instrumentator.health import health

    # Create a mock request that simulates what FastAPI would provide
    mock_request = Mock(spec=Request)
    mock_app_state = Mock()
    mock_engine_client = AsyncMock()
    mock_engine_client.check_health.side_effect = EngineDeadError()
    mock_app_state.engine_client = mock_engine_client
    mock_request.app.state = mock_app_state

    # Test the health function directly with our mocked request
    # This simulates what would happen if the engine dies
    response = await health(mock_request)

    # Assert that it returns 503 Service Unavailable
    assert response.status_code == 503
```
**EN:** This async test validates `test_health_check_engine_dead_error`. Relevant pytest markers include `asyncio`. The main assertion is `response.status_code == 503`.
**CN:** 这个异步测试验证 `test_health_check_engine_dead_error`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `response.status_code == 503`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `http.HTTPStatus`, `unittest.mock.AsyncMock`, `unittest.mock.Mock`
- **Third-party / 第三方**: `fastapi.Request`, `openai`, `pytest`, `pytest_asyncio`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.serve.instrumentator.health.health`, `vllm.v1.engine.exceptions.EngineDeadError`, `vllm.version.__version__`
