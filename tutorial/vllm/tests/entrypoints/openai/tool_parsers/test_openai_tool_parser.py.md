# test_openai_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/tool_parsers/test_openai_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 7 test(s), 2 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 7 个测试、2 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import json

import jsonschema
import openai
import pytest
import pytest_asyncio
from rapidfuzz import fuzz

from ....utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `jsonschema`, `openai`, `pytest`, project helpers such as `....utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `jsonschema`、`openai`、`pytest`）、项目内辅助模块（如 `....utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L14-L14)
```python
MODEL_NAME = "openai/gpt-oss-20b"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L17-L28)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--max-model-len",
        "8192",
        "--enforce-eager",
        "--enable-auto-tool-choice",
        "--tool-call-parser",
        "openai",
    ]
    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L31-L35)
```python
@pytest_asyncio.fixture
async def client(server):
    """Async fixture providing an OpenAI-compatible vLLM client."""
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Module setup / 模块级配置: TOOLS, MESSAGES_CALC, MESSAGES_GET_TIME (L41-L123)
```python
TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "Performs basic arithmetic calculations.",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": (
                            "Arithmetic expression to evaluate, e.g. '123 + 456'."
                        ),
                    }
                },
# ... 61 lines omitted for brevity ...
# Expected outputs
FUNC_CALC = "calculator"
FUNC_ARGS_CALC = '{"expression":"123 + 456"}'

FUNC_TIME = "get_time"
FUNC_ARGS_TIME = '{"city": "New York"}'
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `TOOLS`, `MESSAGES_CALC`, `MESSAGES_GET_TIME`, `MESSAGES_MULTIPLE_CALLS`, `MESSAGES_INVALID_CALL`, `FUNC_CALC`, `FUNC_ARGS_CALC`, `FUNC_TIME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `TOOLS`、`MESSAGES_CALC`、`MESSAGES_GET_TIME`、`MESSAGES_MULTIPLE_CALLS`、`MESSAGES_INVALID_CALL`、`FUNC_CALC`、`FUNC_ARGS_CALC`、`FUNC_TIME`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: extract_reasoning_and_calls (L129-L159)
```python
def extract_reasoning_and_calls(chunks: list) -> tuple[str, list[str], list[str]]:
    """
    Extract accumulated reasoning text and tool call arguments
    from streaming chunks.
    """
    reasoning: str = ""
    tool_calls: dict[int, dict[str, str]] = {}

    for chunk in chunks:
        choice = getattr(chunk.choices[0], "delta", None)
        if not choice:
            continue

        if hasattr(choice, "reasoning") and choice.reasoning:
            reasoning += choice.reasoning

        for tc in getattr(choice, "tool_calls", []) or []:
            idx = getattr(tc, "index", 0)
# ... 5 lines omitted for brevity ...
                    tool_entry["name"] = func.name
                if getattr(func, "arguments", None):
                    tool_entry["arguments"] += func.arguments

    function_names: list[str] = [v["name"] for _, v in sorted(tool_calls.items())]
    arguments: list[str] = [v["arguments"] for _, v in sorted(tool_calls.items())]

    return reasoning, arguments, function_names
```
**EN:** This helper encapsulates reusable logic in `extract_reasoning_and_calls`. Key inputs are `chunks`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `extract_reasoning_and_calls` 中。 关键输入包括 `chunks`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_calculator_tool_call_and_argument_accuracy (L165-L202)
```python
@pytest.mark.asyncio
async def test_calculator_tool_call_and_argument_accuracy(client: openai.AsyncOpenAI):
    """Verify calculator tool call is made and arguments are accurate."""

    response = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_CALC,
        tools=TOOLS,
        temperature=0.0,
        stream=False,
    )

    message = response.choices[0].message
    tool_calls = getattr(message, "tool_calls", [])
    assert tool_calls, "No tool calls detected"

    calc_call = next((c for c in tool_calls if c.function.name == FUNC_CALC), None)
    assert calc_call, "Calculator function not called"
# ... 12 lines omitted for brevity ...
    expected_expr = "123 + 456"
    actual_expr = parsed_args.get("expression", "")
    similarity = fuzz.ratio(actual_expr, expected_expr)

    assert similarity > 90, (
        f"Expression mismatch: expected '{expected_expr}' "
        f"got '{actual_expr}' (similarity={similarity}%)"
    )
```
**EN:** This async test validates `test_calculator_tool_call_and_argument_accuracy`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `tool_calls` and `calc_call`.
**CN:** 这个异步测试验证 `test_calculator_tool_call_and_argument_accuracy`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `tool_calls` and `calc_call`。

### Test / 测试: test_streaming_tool_call_get_time_with_reasoning (L205-L230)
```python
@pytest.mark.asyncio
async def test_streaming_tool_call_get_time_with_reasoning(client: openai.AsyncOpenAI):
    """Verify streamed reasoning and tool call behavior for get_time."""

    stream = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_GET_TIME,
        tools=TOOLS,
        temperature=0.0,
        stream=True,
    )

    chunks = [chunk async for chunk in stream]
    reasoning, arguments, function_names = extract_reasoning_and_calls(chunks)

    assert FUNC_TIME in function_names, "get_time function not called"

    assert any("New York" in arg for arg in arguments), (
        f"Expected get_time arguments for New York not found in {arguments}"
    )

    assert len(reasoning) > 0, "Expected reasoning content missing"

    assert any(keyword in reasoning for keyword in ["New York", "time", "current"]), (
        f"Reasoning is not relevant to the request: {reasoning}"
    )
```
**EN:** This async test validates `test_streaming_tool_call_get_time_with_reasoning`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `FUNC_TIME in function_names` and `any(('New York' in arg for arg in arguments))`.
**CN:** 这个异步测试验证 `test_streaming_tool_call_get_time_with_reasoning`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `FUNC_TIME in function_names` and `any(('New York' in arg for arg in arguments))`。

### Test / 测试: test_streaming_multiple_tools (L233-L256)
```python
@pytest.mark.asyncio
async def test_streaming_multiple_tools(client: openai.AsyncOpenAI):
    """Test streamed multi-tool response with reasoning."""
    stream = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_MULTIPLE_CALLS,
        tools=TOOLS,
        temperature=0.0,
        stream=True,
    )

    chunks = [chunk async for chunk in stream]
    reasoning, arguments, function_names = extract_reasoning_and_calls(chunks)

    try:
        assert FUNC_CALC in function_names, (
            f"Calculator tool missing — found {function_names}"
        )
        assert FUNC_TIME in function_names, (
            f"Time tool missing — found {function_names}"
        )
        assert len(reasoning) > 0, "Expected reasoning content in streamed response"
    except AssertionError as e:
        print(f"ERROR: {e}")
```
**EN:** This async test validates `test_streaming_multiple_tools`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `FUNC_CALC in function_names` and `FUNC_TIME in function_names`.
**CN:** 这个异步测试验证 `test_streaming_multiple_tools`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `FUNC_CALC in function_names` and `FUNC_TIME in function_names`。

### Test / 测试: test_invalid_tool_call (L259-L281)
```python
@pytest.mark.asyncio
async def test_invalid_tool_call(client: openai.AsyncOpenAI):
    """
    Verify that ambiguous instructions that should not trigger a tool
    do not produce any tool calls.
    """
    response = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_INVALID_CALL,
        tools=TOOLS,
        temperature=0.0,
        stream=False,
    )

    message = response.choices[0].message

    assert message is not None, "Expected message in response"
    assert hasattr(message, "content"), "Expected 'content' field in message"

    tool_calls = getattr(message, "tool_calls", [])
    assert not tool_calls, (
        f"Model unexpectedly attempted a tool call on invalid input: {tool_calls}"
    )
```
**EN:** This async test validates `test_invalid_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `message is not None` and `hasattr(message, 'content')`.
**CN:** 这个异步测试验证 `test_invalid_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message is not None` and `hasattr(message, 'content')`。

### Test / 测试: test_tool_call_with_temperature (L284-L305)
```python
@pytest.mark.asyncio
async def test_tool_call_with_temperature(client: openai.AsyncOpenAI):
    """
    Verify model produces valid tool or text output
    under non-deterministic sampling.
    """
    response = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_CALC,
        tools=TOOLS,
        temperature=0.7,
        stream=False,
    )

    message = response.choices[0].message
    assert message is not None, "Expected non-empty message in response"
    assert message.tool_calls or message.content, (
        "Response missing both text and tool calls"
    )

    print(f"\nTool calls: {message.tool_calls}")
    print(f"Text: {message.content}")
```
**EN:** This async test validates `test_tool_call_with_temperature`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `message is not None` and `message.tool_calls or message.content`.
**CN:** 这个异步测试验证 `test_tool_call_with_temperature`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message is not None` and `message.tool_calls or message.content`。

### Test / 测试: test_tool_response_schema_accuracy (L308-L338)
```python
@pytest.mark.asyncio
async def test_tool_response_schema_accuracy(client: openai.AsyncOpenAI):
    """Validate that tool call arguments adhere to their declared JSON schema."""
    response = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES_MULTIPLE_CALLS,
        tools=TOOLS,
        temperature=0.0,
    )

    calls = response.choices[0].message.tool_calls
    assert calls, "No tool calls produced"

    for call in calls:
        func_name = call.function.name
        args = json.loads(call.function.arguments)

        schema: dict[str, object] | None = None
# ... 5 lines omitted for brevity ...
                and function_def.get("name") == func_name
            ):
                schema = function_def.get("parameters")
                break

        assert schema is not None, f"No matching tool schema found for {func_name}"

        jsonschema.validate(instance=args, schema=schema)
```
**EN:** This async test validates `test_tool_response_schema_accuracy`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `calls` and `schema is not None`.
**CN:** 这个异步测试验证 `test_tool_response_schema_accuracy`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `calls` and `schema is not None`。

### Test / 测试: test_semantic_consistency_with_temperature (L341-L359)
```python
@pytest.mark.asyncio
async def test_semantic_consistency_with_temperature(client: openai.AsyncOpenAI):
    """Test that temperature variation doesn't cause contradictory reasoning."""
    responses = []
    for temp in [0.0, 0.5, 1.0]:
        resp = await client.chat.completions.create(
            model=MODEL_NAME,
            messages=MESSAGES_CALC,
            tools=TOOLS,
            temperature=temp,
        )
        text = (resp.choices[0].message.content or "").strip()
        responses.append(text)

    # Compare fuzzy similarity between low- and mid-temperature outputs
    low_mid_sim = fuzz.ratio(responses[0], responses[1])
    assert low_mid_sim > 60, (
        f"Semantic drift too large between T=0.0 and T=0.5 ({low_mid_sim}%)"
    )
```
**EN:** This async test validates `test_semantic_consistency_with_temperature`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `low_mid_sim > 60`.
**CN:** 这个异步测试验证 `test_semantic_consistency_with_temperature`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `low_mid_sim > 60`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `jsonschema`, `openai`, `pytest`, `pytest_asyncio`, `rapidfuzz.fuzz`
- **Local relative imports / 本地相对导入**: `....utils.RemoteOpenAIServer`
