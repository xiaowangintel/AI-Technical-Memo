# test_parsable_context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_parsable_context.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 5 test(s), 2 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 5 个测试、2 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L19)
```python
import importlib.util
import json
import logging

import pytest
import pytest_asyncio
from openai import OpenAI

from tests.utils import RemoteOpenAIServer

from .conftest import (
    BASE_TEST_ENV,
    has_output_type,
    log_response_diagnostics,
    retry_for_tool_call,
)
```
**EN:** Imports standard-library modules such as `importlib.util`, `json`, `logging`, third-party packages like `openai.OpenAI`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `.conftest.BASE_TEST_ENV`, `.conftest.has_output_type`.
**CN:** 导入标准库模块（如 `importlib.util`、`json`、`logging`）、第三方包（如 `openai.OpenAI`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`.conftest.BASE_TEST_ENV`、`.conftest.has_output_type`）。

### Module setup / 模块级配置: logger, MODEL_NAME, _PYTHON_TOOL_INSTRUCTION (L21-L28)
```python
logger = logging.getLogger(__name__)

MODEL_NAME = "Qwen/Qwen3-8B"

_PYTHON_TOOL_INSTRUCTION = (
    "You must use the Python tool to execute code. "
    "Never simulate execution. You must print the final answer."
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `logger`, `MODEL_NAME`, `_PYTHON_TOOL_INSTRUCTION`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `logger`、`MODEL_NAME`、`_PYTHON_TOOL_INSTRUCTION`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L31-L57)
```python
@pytest.fixture(scope="module")
def server():
    assert importlib.util.find_spec("gpt_oss") is not None, (
        "Harmony tests require gpt_oss package to be installed"
    )

    args = [
        "--reasoning-parser",
        "qwen3",
        "--max_model_len",
        "5000",
        "--structured-outputs-config.backend",
        "xgrammar",
        "--enable-auto-tool-choice",
        "--tool-call-parser",
        "hermes",
        "--tool-server",
        "demo",
    ]
    env_dict = {
        **BASE_TEST_ENV,
        "VLLM_ENABLE_RESPONSES_API_STORE": "1",
        "VLLM_USE_EXPERIMENTAL_PARSER_CONTEXT": "1",
        "PYTHON_EXECUTION_BACKEND": "dangerously_use_uv",
    }
    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_dict) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes. The main assertion is `importlib.util.find_spec('gpt_oss') is not None`.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。 核心断言是 `importlib.util.find_spec('gpt_oss') is not None`。

### Fixture / 夹具: client (L60-L63)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L66-L77)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_basic(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is 123 * 456?",
        temperature=0.0,
    )
    assert response is not None
    print("response: ", response)
    assert response.status == "completed"
    assert response.incomplete_details is None
```
**EN:** This async test validates `test_basic`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_basic`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_reasoning_and_function_items (L80-L126)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_reasoning_and_function_items(client: OpenAI, model_name: str):
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
# ... 21 lines omitted for brevity ...
    output_types = [getattr(o, "type", None) for o in response.output]
    assert "reasoning" in output_types, (
        f"Expected reasoning in output, got: {output_types}"
    )
    assert "message" in output_types, f"Expected message in output, got: {output_types}"

    msg = next(o for o in response.output if o.type == "message")
    assert type(msg.content[0].text) is str
```
**EN:** This async test validates `test_reasoning_and_function_items`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_reasoning_and_function_items`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Helper / 辅助函数: get_horoscope (L129-L130)
```python
def get_horoscope(sign):
    return f"{sign}: Next Tuesday you will befriend a baby otter."
```
**EN:** This helper encapsulates reusable logic in `get_horoscope`. Key inputs are `sign`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_horoscope` 中。 关键输入包括 `sign`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: call_function (L133-L137)
```python
def call_function(name, args):
    logger.info("Calling function %s with args %s", name, args)
    if name == "get_horoscope":
        return get_horoscope(**args)
    raise ValueError(f"Unknown function: {name}")
```
**EN:** This helper encapsulates reusable logic in `call_function`. Key inputs are `name`, `args`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `call_function` 中。 关键输入包括 `name`、`args`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_function_call_first_turn (L140-L184)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_call_first_turn(client: OpenAI, model_name: str):
    tools = [
        {
            "type": "function",
            "name": "get_horoscope",
            "description": "Get today's horoscope for an astrological sign.",
            "parameters": {
                "type": "object",
                "properties": {
                    "sign": {"type": "string"},
                },
                "required": ["sign"],
                "additionalProperties": False,
            },
            "strict": True,
        }
# ... 19 lines omitted for brevity ...
    )

    function_call = next(o for o in response.output if o.type == "function_call")
    assert function_call.name == "get_horoscope"
    assert function_call.call_id is not None

    args = json.loads(function_call.arguments)
    assert "sign" in args
```
**EN:** This async test validates `test_function_call_first_turn`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_function_call_first_turn`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_mcp_tool_call (L187-L265)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_mcp_tool_call(client: OpenAI, model_name: str):
    """MCP tool calling with code_interpreter.

    The model may make one or more tool calls before producing a final
    message.  We validate server invariants (mcp_call items have correct
    fields) with hard assertions.  Output indices are never hardcoded
    since the model can produce multiple tool-call rounds.
    """
    # MCP + container init + code execution can be slow
    client_with_timeout = client.with_options(timeout=client.timeout * 3)

    response = await retry_for_tool_call(
        client_with_timeout,
        model=model_name,
        expected_tool_type="mcp_call",
        input=(
# ... 53 lines omitted for brevity ...
    assert len(response.output_messages) >= 1, "Expected at least 1 output message"
    assert any(
        any(s in str(msg) for s in ("56088", "56,088"))
        for msg in response.output_messages
    ), (
        f"Expected 56088 in at least one output_message, "
        f"got {len(response.output_messages)} messages"
    )
```
**EN:** This async test validates `test_mcp_tool_call`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_mcp_tool_call`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_max_tokens (L268-L280)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_max_tokens(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is the first paragraph of Moby Dick?",
        reasoning={"effort": "low"},
        max_output_tokens=30,
        temperature=0.0,
    )
    assert response is not None
    assert response.status == "incomplete"
    assert response.incomplete_details.reason == "max_output_tokens"
```
**EN:** This async test validates `test_max_tokens`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'incomplete'`.
**CN:** 这个异步测试验证 `test_max_tokens`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'incomplete'`。

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
- **Stdlib / 标准库**: `importlib.util`, `json`, `logging`
- **Third-party / 第三方**: `openai.OpenAI`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
- **Local relative imports / 本地相对导入**: `.conftest.BASE_TEST_ENV`, `.conftest.has_output_type`, `.conftest.log_response_diagnostics`, `.conftest.retry_for_tool_call`
