# test_simple.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_simple.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 10 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 10 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L10)
```python
import pytest
import pytest_asyncio
from openai import OpenAI

from tests.utils import RemoteOpenAIServer

from .conftest import validate_streaming_event_stack
```
**EN:** Imports third-party packages like `openai.OpenAI`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `.conftest.BASE_TEST_ENV`, `.conftest.validate_streaming_event_stack`.
**CN:** 导入第三方包（如 `openai.OpenAI`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`.conftest.BASE_TEST_ENV`、`.conftest.validate_streaming_event_stack`）。

### Module setup / 模块级配置: MODEL_NAME (L12-L12)
```python
MODEL_NAME = "Qwen/Qwen3-8B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L15-L27)
```python
@pytest.fixture(scope="module")
def server():
    from .conftest import BASE_TEST_ENV

    args = ["--reasoning-parser", "qwen3", "--max_model_len", "5000"]
    env_dict = {
        **BASE_TEST_ENV,
        "VLLM_ENABLE_RESPONSES_API_STORE": "1",
        # uncomment for tool calling
        # PYTHON_EXECUTION_BACKEND: "dangerously_use_uv",
    }
    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_dict) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L30-L33)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L36-L46)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_basic(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is 123 * 456?",
    )
    assert response is not None
    print("response: ", response)
    assert response.status == "completed"
    assert response.incomplete_details is None
```
**EN:** This async test validates `test_basic`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_basic`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_enable_response_messages (L49-L64)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_enable_response_messages(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="Hello?",
        extra_body={"enable_response_messages": True},
    )
    assert response.status == "completed"
    assert response.input_messages[0]["type"] == "raw_message_tokens"
    assert type(response.input_messages[0]["message"]) is str
    assert len(response.input_messages[0]["message"]) > 10
    assert type(response.input_messages[0]["tokens"][0]) is int
    assert type(response.output_messages[0]["message"]) is str
    assert len(response.output_messages[0]["message"]) > 10
    assert type(response.output_messages[0]["tokens"][0]) is int
```
**EN:** This async test validates `test_enable_response_messages`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `response.input_messages[0]['type'] == 'raw_message_tokens'`.
**CN:** 这个异步测试验证 `test_enable_response_messages`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `response.input_messages[0]['type'] == 'raw_message_tokens'`。

### Test / 测试: test_reasoning_item (L67-L93)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_reasoning_item(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input=[
            {"type": "message", "content": "Hello.", "role": "user"},
            {
                "type": "reasoning",
                "id": "lol",
                "content": [
                    {
                        "type": "reasoning_text",
                        "text": "We need to respond: greeting.",
                    }
                ],
                "summary": [],
            },
        ],
        temperature=0.0,
    )
    assert response is not None
    assert response.status == "completed"
    # make sure we get a reasoning and text output
    assert response.output[0].type == "reasoning"
    assert response.output[1].type == "message"
    assert type(response.output[1].content[0].text) is str
```
**EN:** This async test validates `test_reasoning_item`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_reasoning_item`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_streaming_output_consistency (L96-L138)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_streaming_output_consistency(client: OpenAI, model_name: str):
    """Test that streaming delta text matches the final response output_text.

    This test verifies that when using streaming mode:
    1. The concatenated text from all 'response.output_text.delta' events
    2. Matches the 'output_text' in the final 'response.completed' event
    """
    response = await client.responses.create(
        model=model_name,
        input="Say hello in one sentence.",
        stream=True,
    )

    events = []
    async for event in response:
        events.append(event)
# ... 17 lines omitted for brevity ...
    assert len(response_completed_event.response.output) > 0

    # Verify streaming text matches final output_text
    assert streaming_text == final_output_text, (
        f"Streaming text does not match final output_text.\n"
        f"Streaming: {streaming_text!r}\n"
        f"Final: {final_output_text!r}"
    )
```
**EN:** This async test validates `test_streaming_output_consistency`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(events) > 0` and `response_completed_event.type == 'response.completed'`.
**CN:** 这个异步测试验证 `test_streaming_output_consistency`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(events) > 0` and `response_completed_event.type == 'response.completed'`。

### Test / 测试: test_streaming_logprobs (L141-L191)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_streaming_logprobs(client: OpenAI, model_name: str):
    """Test that streaming with logprobs returns valid logprob data on
    output_text.delta events and that top_logprobs has the requested count."""
    response = await client.responses.create(
        model=model_name,
        input="Say hello.",
        stream=True,
        top_logprobs=3,
        include=["message.output_text.logprobs"],
    )

    events = []
    async for event in response:
        events.append(event)

    assert len(events) > 0
# ... 25 lines omitted for brevity ...
    assert len(all_top_logprobs) > 0, (
        "Expected at least one top_logprobs entry across all delta events"
    )

    # Verify the completed event still has valid output
    completed = events[-1]
    assert completed.type == "response.completed"
    assert completed.response.status == "completed"
```
**EN:** This async test validates `test_streaming_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(events) > 0` and `len(text_delta_events) > 0`.
**CN:** 这个异步测试验证 `test_streaming_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(events) > 0` and `len(text_delta_events) > 0`。

### Test / 测试: test_streaming_reasoning_tokens_e2e (L194-L217)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_streaming_reasoning_tokens_e2e(client: OpenAI, model_name: str):
    """Verify final usage includes reasoning_tokens in streaming mode."""
    response = await client.responses.create(
        model=model_name,
        input="Compute 17 * 19 and explain briefly.",
        reasoning={"effort": "low"},
        temperature=0.0,
        stream=True,
    )

    completed_event = None
    async for event in response:
        if event.type == "response.completed":
            completed_event = event

    assert completed_event is not None
    assert completed_event.response.status == "completed"
    assert completed_event.response.usage is not None
    assert completed_event.response.usage.output_tokens_details is not None
    assert completed_event.response.usage.output_tokens_details.reasoning_tokens > 0, (
        "Expected reasoning_tokens > 0 for streamed Qwen3 response."
    )
```
**EN:** This async test validates `test_streaming_reasoning_tokens_e2e`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `completed_event is not None` and `completed_event.response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_streaming_reasoning_tokens_e2e`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completed_event is not None` and `completed_event.response.status == 'completed'`。

### Test / 测试: test_non_streaming_reasoning_tokens_e2e (L220-L238)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_non_streaming_reasoning_tokens_e2e(client: OpenAI, model_name: str):
    """Verify usage includes reasoning_tokens in non-streaming mode."""
    response = await client.responses.create(
        model=model_name,
        input="Compute 23 * 17 and explain briefly.",
        reasoning={"effort": "low"},
        temperature=0.0,
        stream=False,
    )

    assert response is not None
    assert response.status == "completed"
    assert response.usage is not None
    assert response.usage.output_tokens_details is not None
    assert response.usage.output_tokens_details.reasoning_tokens > 0, (
        "Expected reasoning_tokens > 0 for non-streamed Qwen3 response."
    )
```
**EN:** This async test validates `test_non_streaming_reasoning_tokens_e2e`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_non_streaming_reasoning_tokens_e2e`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_max_tokens (L241-L252)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_max_tokens(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is the first paragraph of Moby Dick?",
        reasoning={"effort": "low"},
        max_output_tokens=30,
    )
    assert response is not None
    assert response.status == "incomplete"
    assert response.incomplete_details.reason == "max_output_tokens"
```
**EN:** This async test validates `test_max_tokens`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'incomplete'`.
**CN:** 这个异步测试验证 `test_max_tokens`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'incomplete'`。

### Test / 测试: test_extra_sampling_params (L255-L276)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_extra_sampling_params(client: OpenAI, model_name: str):
    """Test that extra sampling parameters are accepted and work."""
    # Test with multiple sampling parameters - just verify they're accepted
    response = await client.responses.create(
        model=model_name,
        input="Write a short sentence",
        max_output_tokens=50,
        temperature=0.7,
        top_p=0.9,
        extra_body={
            "top_k": 40,
            "repetition_penalty": 1.2,
            "seed": 42,
        },
    )

    # Verify request succeeded and parameters were accepted
    assert response.status in ["completed", "incomplete"]
    assert len(response.output) > 0
    assert response.output[0].content[0].text  # Has text output
```
**EN:** This async test validates `test_extra_sampling_params`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status in ['completed', 'incomplete']` and `len(response.output) > 0`.
**CN:** 这个异步测试验证 `test_extra_sampling_params`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status in ['completed', 'incomplete']` and `len(response.output) > 0`。

### Test / 测试: test_streaming_types (L279-L296)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_streaming_types(
    pairs_of_event_types: dict[str, str], client: OpenAI, model_name: str
):
    stream = await client.responses.create(
        model=model_name,
        input="tell me a story about a cat in 20 words",
        reasoning={"effort": "low"},
        tools=[],
        stream=True,
        background=False,
    )
    events = []
    async for event in stream:
        events.append(event)

    validate_streaming_event_stack(events, pairs_of_event_types)
```
**EN:** This async test validates `test_streaming_types`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `pairs_of_event_types`, `client`, `model_name`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_streaming_types`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `pairs_of_event_types`、`client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。

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
- **Third-party / 第三方**: `openai.OpenAI`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
- **Local relative imports / 本地相对导入**: `.conftest.BASE_TEST_ENV`, `.conftest.validate_streaming_event_stack`
