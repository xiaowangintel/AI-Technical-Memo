# test_messages.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/anthropic/test_messages.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Anthropic-compatible messaging. The file defines 5 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖Anthropic 兼容消息。它定义了 5 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L8)
```python
import anthropic
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `anthropic`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `anthropic`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L10-L10)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L13-L27)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--max-model-len",
        "2048",
        "--enforce-eager",
        "--enable-auto-tool-choice",
        "--tool-call-parser",
        "hermes",
        "--served-model-name",
        "claude-3-7-sonnet-latest",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L30-L33)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client_anthropic() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_simple_messages (L36-L46)
```python
@pytest.mark.asyncio
async def test_simple_messages(client: anthropic.AsyncAnthropic):
    resp = await client.messages.create(
        model="claude-3-7-sonnet-latest",
        max_tokens=1024,
        messages=[{"role": "user", "content": "how are you!"}],
    )
    assert resp.stop_reason == "end_turn"
    assert resp.role == "assistant"

    print(f"Anthropic response: {resp.model_dump_json()}")
```
**EN:** This async test validates `test_simple_messages`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `resp.stop_reason == 'end_turn'` and `resp.role == 'assistant'`.
**CN:** 这个异步测试验证 `test_simple_messages`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `resp.stop_reason == 'end_turn'` and `resp.role == 'assistant'`。

### Test / 测试: test_system_message (L49-L60)
```python
@pytest.mark.asyncio
async def test_system_message(client: anthropic.AsyncAnthropic):
    resp = await client.messages.create(
        model="claude-3-7-sonnet-latest",
        max_tokens=1024,
        system="you are a helpful assistant",
        messages=[{"role": "user", "content": "how are you!"}],
    )
    assert resp.stop_reason == "end_turn"
    assert resp.role == "assistant"

    print(f"Anthropic response: {resp.model_dump_json()}")
```
**EN:** This async test validates `test_system_message`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `resp.stop_reason == 'end_turn'` and `resp.role == 'assistant'`.
**CN:** 这个异步测试验证 `test_system_message`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `resp.stop_reason == 'end_turn'` and `resp.role == 'assistant'`。

### Test / 测试: test_anthropic_streaming (L63-L87)
```python
@pytest.mark.asyncio
async def test_anthropic_streaming(client: anthropic.AsyncAnthropic):
    resp = await client.messages.create(
        model="claude-3-7-sonnet-latest",
        max_tokens=1024,
        messages=[{"role": "user", "content": "how are you!"}],
        stream=True,
    )

    first_chunk = None
    chunk_count = 0
    async for chunk in resp:
        chunk_count += 1
        if first_chunk is None and chunk.type == "message_start":
            first_chunk = chunk
        print(chunk.model_dump_json())

    assert chunk_count > 0
    assert first_chunk is not None, "message_start chunk was never observed"
    assert first_chunk.message is not None, "first chunk should include message"
    assert first_chunk.message.usage is not None, (
        "first chunk should include usage stats"
    )
    assert first_chunk.message.usage.output_tokens == 0
    assert first_chunk.message.usage.input_tokens > 5
```
**EN:** This async test validates `test_anthropic_streaming`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `chunk_count > 0` and `first_chunk is not None`.
**CN:** 这个异步测试验证 `test_anthropic_streaming`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chunk_count > 0` and `first_chunk is not None`。

### Test / 测试: test_anthropic_tool_call (L90-L120)
```python
@pytest.mark.asyncio
async def test_anthropic_tool_call(client: anthropic.AsyncAnthropic):
    resp = await client.messages.create(
        model="claude-3-7-sonnet-latest",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": "What's the weather like in New York today?"}
        ],
        tools=[
            {
                "name": "get_current_weather",
                "description": "Useful for querying the weather in a specified city.",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "City or region, for example: "
# ... 5 lines omitted for brevity ...
            }
        ],
        stream=False,
    )
    assert resp.stop_reason == "tool_use"
    assert resp.role == "assistant"

    print(f"Anthropic response: {resp.model_dump_json()}")
```
**EN:** This async test validates `test_anthropic_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `resp.stop_reason == 'tool_use'` and `resp.role == 'assistant'`.
**CN:** 这个异步测试验证 `test_anthropic_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `resp.stop_reason == 'tool_use'` and `resp.role == 'assistant'`。

### Test / 测试: test_anthropic_tool_call_streaming (L123-L155)
```python
@pytest.mark.asyncio
async def test_anthropic_tool_call_streaming(client: anthropic.AsyncAnthropic):
    resp = await client.messages.create(
        model="claude-3-7-sonnet-latest",
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": "What's the weather like in New York today?",
            }
        ],
        tools=[
            {
                "name": "get_current_weather",
                "description": "Useful for querying the weather in a specified city.",
                "input_schema": {
                    "type": "object",
                    "properties": {
# ... 7 lines omitted for brevity ...
                },
            }
        ],
        stream=True,
    )

    async for chunk in resp:
        print(chunk.model_dump_json())
```
**EN:** This async test validates `test_anthropic_tool_call_streaming`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_anthropic_tool_call_streaming`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The scenarios model Anthropic-style message formatting or serving semantics.
  **CN:** 这些场景模拟 Anthropic 风格的消息格式或服务语义。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `anthropic`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
