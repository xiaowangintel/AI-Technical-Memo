# test_orca_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/instrumentator/test_orca_metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 3 test(s), 3 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 3 个测试、3 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L8)
```python
import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `_pytest.monkeypatch.MonkeyPatch`, `openai`, `pytest`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `_pytest.monkeypatch.MonkeyPatch`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L11-L11)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: monkeypatch_module (L14-L20)
```python
@pytest.fixture(scope="module")
def monkeypatch_module():
    from _pytest.monkeypatch import MonkeyPatch

    mpatch = MonkeyPatch()
    yield mpatch
    mpatch.undo()
```
**EN:** This fixture prepares `monkeypatch_module` for dependent tests. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `monkeypatch_module`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: server (L23-L34)
```python
@pytest.fixture(scope="module", params=[True])
def server(request, monkeypatch_module):
    args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "8192",
        "--enforce-eager",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`, `monkeypatch_module`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`、`monkeypatch_module`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L37-L40)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_chat_completion_with_orca_header (L43-L72)
```python
@pytest.mark.asyncio
async def test_chat_completion_with_orca_header(server: RemoteOpenAIServer):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    client = openai.OpenAI(
        api_key="EMPTY",
        base_url=f"http://localhost:{server.port}/v1",
        default_headers={"endpoint-load-metrics-format": "TEXT"},
    )

    # 1. Use raw client to get response headers.
    raw_client = client.with_raw_response

    # 2. Make the API call using the raw_client
    response_with_raw = raw_client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        extra_headers={"endpoint-load-metrics-format": "TEXT"},
    )

    # 3. Access the raw httpx.Response object
    raw_http_response = response_with_raw.http_response

    # 4. Get the headers from the httpx.Response object
    response_headers = raw_http_response.headers

    assert "endpoint-load-metrics" in response_headers
```
**EN:** This async test validates `test_chat_completion_with_orca_header`. Relevant pytest markers include `asyncio`. Key inputs are `server`. It drives client-facing request creation through the API surface under test. The main assertion is `'endpoint-load-metrics' in response_headers`.
**CN:** 这个异步测试验证 `test_chat_completion_with_orca_header`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'endpoint-load-metrics' in response_headers`。

### Test / 测试: test_completion_with_orca_header (L75-L94)
```python
@pytest.mark.asyncio
async def test_completion_with_orca_header(client: openai.AsyncOpenAI):
    # 1. Use raw client to get response headers.
    raw_client = client.with_raw_response

    # 2. Make the API call using the raw_client
    completion = await raw_client.completions.create(
        model=MODEL_NAME,
        prompt="Hello, my name is",
        max_tokens=5,
        extra_headers={"endpoint-load-metrics-format": "JSON"},
    )

    # 3. Access the raw httpx.Response object
    raw_http_response = completion.http_response

    # 4. Get the headers from the httpx.Response object
    response_headers = raw_http_response.headers

    assert "endpoint-load-metrics" in response_headers
```
**EN:** This async test validates `test_completion_with_orca_header`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `'endpoint-load-metrics' in response_headers`.
**CN:** 这个异步测试验证 `test_completion_with_orca_header`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'endpoint-load-metrics' in response_headers`。

### Test / 测试: test_single_completion (L97-L126)
```python
@pytest.mark.asyncio
async def test_single_completion(client: openai.AsyncOpenAI):
    completion = await client.completions.create(
        model=MODEL_NAME,
        prompt="Hello, my name is",
        max_tokens=5,
        extra_headers={"endpoint-load-metrics-format": "JSON"},
        temperature=0.0,
    )

    assert completion.id is not None
    assert completion.choices is not None and len(completion.choices) == 1

    choice = completion.choices[0]
    assert len(choice.text) >= 5
    assert choice.finish_reason == "length"
    # When using Qwen3-0.6B, prompt tokens=[9707, 11, 847, 829, 374]
    assert completion.usage == openai.types.CompletionUsage(
        completion_tokens=5, prompt_tokens=5, total_tokens=10
    )

    # test using token IDs
    completion = await client.completions.create(
        model=MODEL_NAME,
        prompt=[0, 0, 0, 0, 0],
        max_tokens=5,
        temperature=0.0,
    )
    assert len(completion.choices[0].text) >= 1
    assert completion.choices[0].prompt_logprobs is None
```
**EN:** This async test validates `test_single_completion`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `completion.id is not None` and `completion.choices is not None and len(completion.choices) == 1`.
**CN:** 这个异步测试验证 `test_single_completion`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion.id is not None` and `completion.choices is not None and len(completion.choices) == 1`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `_pytest.monkeypatch.MonkeyPatch`, `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
