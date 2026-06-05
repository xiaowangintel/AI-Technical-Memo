# test_function_call.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_function_call.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 6 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 6 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L7)
```python
import json

import openai  # use the official client for correctness check
import pytest
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`）。

### Module setup / 模块级配置: MODEL_NAME, tools (L9-L98)
```python
MODEL_NAME = "Qwen/Qwen3-1.7B"
tools = [
    {
        "type": "function",
        "name": "get_current_weather",
        "description": "Get the current weather in a given location",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city to find the weather for, e.g. 'Vienna'",
                    "default": "Vienna",
                },
                "country": {
                    "type": "string",
# ... 68 lines omitted for brevity ...
                },
            },
            "required": ["country", "days", "unit"],
        },
    },
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `tools`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`tools`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_function_tool_use (L101-L137)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("tool_choice", ["auto", "required"])
async def test_function_tool_use(
    client: openai.AsyncOpenAI, model_name: str, tool_choice: str
):
    prompt = [
        {
            "role": "user",
            "content": "Can you tell me what the current weather is in Berlin and the "
            "forecast for the next 5 days, in fahrenheit?",
        },
    ]
    response = await client.responses.create(
        model=model_name,
        input=prompt,
        tools=tools,
        tool_choice=tool_choice,
# ... 11 lines omitted for brevity ...
        assert tool_call is not None
        assert tool_call.type == "function_call"
        assert json.loads(tool_call.arguments) is not None
        assert reasoning is not None
        assert reasoning.type == "reasoning"
    else:
        print(response.model_dump_json(indent=2))
        assert response.incomplete_details.reason == "max_output_tokens"
```
**EN:** This async test validates `test_function_tool_use`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `tool_choice`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output) >= 1` and `tool_call is not None`.
**CN:** 这个异步测试验证 `test_function_tool_use`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`tool_choice`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output) >= 1` and `tool_call is not None`。

### Test / 测试: test_max_tokens_with_tool_choice_required (L140-L165)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_max_tokens_with_tool_choice_required(
    client: openai.AsyncOpenAI, model_name: str
):
    prompt = [
        {
            "role": "user",
            "content": "Can you tell me what the current weather is in Berlin and the "
            "forecast for the next 5 days, in fahrenheit?",
        },
    ]
    response = await client.responses.create(
        model=model_name,
        input=prompt,
        tools=tools,
        tool_choice="required",
        max_output_tokens=10,
    )
    assert len(response.output) >= 1
    for out in response.output:
        # When `tool_choice="required"` and the tokens of `tools`
        # exceed `max_output_tokens`,`function_call` should be empty.
        # This behavior should be consistent with OpenAI
        assert out.type != "function_call"
    assert response.incomplete_details.reason == "max_output_tokens"
```
**EN:** This async test validates `test_max_tokens_with_tool_choice_required`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output) >= 1` and `response.incomplete_details.reason == 'max_output_tokens'`.
**CN:** 这个异步测试验证 `test_max_tokens_with_tool_choice_required`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output) >= 1` and `response.incomplete_details.reason == 'max_output_tokens'`。

### Test / 测试: test_named_tool_use (L168-L230)
```python
@pytest.mark.asyncio
async def test_named_tool_use(client: openai.AsyncOpenAI):
    def get_weather(latitude: float, longitude: float) -> str:
        """
        Mock function to simulate getting weather data.
        In a real application, this would call an external weather API.
        """
        return f"Current temperature at ({latitude}, {longitude}) is 20°C."

    tools = [
        {
            "type": "function",
            "name": "get_weather",
            "description": (
                "Get current temperature for provided coordinates in celsius."
            ),
            "parameters": {
                "type": "object",
# ... 37 lines omitted for brevity ...
            "call_id": tool_call.call_id,
            "output": str(result),
        }
    )
    # create a new response with the tool call result
    response_2 = await client.responses.create(model=MODEL_NAME, input=input_messages)
    # check the output
    assert len(response_2.output_text) > 0
```
**EN:** This async test validates `test_named_tool_use`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output) >= 1` and `tool_call is not None`.
**CN:** 这个异步测试验证 `test_named_tool_use`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output) >= 1` and `tool_call is not None`。

### Test / 测试: test_function_calling_with_streaming_expected_arguments (L233-L319)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_with_streaming_expected_arguments(
    client: openai.AsyncOpenAI, model_name: str
):
    tools = [
        {
            "type": "function",
            "name": "get_weather",
            "description": "Get current temperature for provided location in celsius.",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                },
                "required": ["location"],
                "additionalProperties": False,
            },
# ... 61 lines omitted for brevity ...
        assert added_item.type == "function_call"
        assert added_item.arguments == arguments_done_event.arguments
        assert added_item.arguments == completed_item.arguments
        assert added_item.name == arguments_done_event.name
        assert added_item.name == completed_item.name
        args = json.loads(added_item.arguments)
        assert "location" in args
        assert args["location"] is not None
```
**EN:** This async test validates `test_function_calling_with_streaming_expected_arguments`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(tool_call_items) >= 2` and `len(arguments_done_events) >= 2`.
**CN:** 这个异步测试验证 `test_function_calling_with_streaming_expected_arguments`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(tool_call_items) >= 2` and `len(arguments_done_events) >= 2`。

### Test / 测试: test_function_calling_with_streaming_types (L322-L463)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "tool_choice",
    ["auto", "required", {"type": "function", "name": "get_current_weather"}],
)
@pytest.mark.parametrize(
    "enable_thinking",
    [True, False],
)
async def test_function_calling_with_streaming_types(
    client: openai.AsyncOpenAI, model_name: str, tool_choice, enable_thinking: bool
):
    # this links the "done" type with the "start" type
    # so every "done" type should have a corresponding "start" type
    # and every open block should be closed by the end of the stream
    #
    # stream of events for a response with function call could look like this:
# ... 116 lines omitted for brevity ...
        elif event.type.endswith("delta"):
            if stack_of_event_types[-1] == event.type:
                continue
            stack_of_event_types.append(event.type)
        elif event.type.endswith("done"):
            assert stack_of_event_types[-1] == pairs_of_event_types[event.type]
            stack_of_event_types.pop()
    assert len(stack_of_event_types) == 0
```
**EN:** This async test validates `test_function_calling_with_streaming_types`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `tool_choice`, `enable_thinking`. It drives client-facing request creation through the API surface under test. The main assertion is `len(stack_of_event_types) == 0` and `stack_of_event_types[-1] == pairs_of_event_types[event.type]`.
**CN:** 这个异步测试验证 `test_function_calling_with_streaming_types`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`tool_choice`、`enable_thinking`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(stack_of_event_types) == 0` and `stack_of_event_types[-1] == pairs_of_event_types[event.type]`。

### Test / 测试: test_function_calling_with_streaming_forced_tool_choice (L466-L529)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "tool_choice",
    ["required", "auto", {"type": "function", "name": "get_weather"}],
)
async def test_function_calling_with_streaming_forced_tool_choice(
    client: openai.AsyncOpenAI, model_name: str, tool_choice: str
):
    tools = [
        {
            "type": "function",
            "name": "get_weather",
            "description": "Get current temperature for provided location in celsius.",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
# ... 38 lines omitted for brevity ...
    assert completed_event is not None
    assert tool_call_item.arguments == completed_event.item.arguments
    assert tool_call_item.name == completed_event.item.name
    args = json.loads(tool_call_item.arguments)
    assert "location" in args
    assert args["location"] is not None
    # Forced tool choice should not leak tool-call JSON via output_text delta.
    assert "".join(text_deltas).strip() == ""
```
**EN:** This async test validates `test_function_calling_with_streaming_forced_tool_choice`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `tool_choice`. It drives client-facing request creation through the API surface under test. The main assertion is `tool_call_item is not None` and `tool_call_item.type == 'function_call'`.
**CN:** 这个异步测试验证 `test_function_calling_with_streaming_forced_tool_choice`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`tool_choice`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `tool_call_item is not None` and `tool_call_item.type == 'function_call'`。

## Key Concepts / 关键概念
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
- **Third-party / 第三方**: `openai`, `pytest`
