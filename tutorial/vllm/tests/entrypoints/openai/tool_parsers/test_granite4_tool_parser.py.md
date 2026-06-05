# test_granite4_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/tool_parsers/test_granite4_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 1 test(s), 1 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 1 个测试、1 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L8)
```python
import json

import openai
import pytest

from ....utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`, project helpers such as `....utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`）、项目内辅助模块（如 `....utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL (L10-L10)
```python
MODEL = "ibm-granite/granite-4.0-h-tiny"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L13-L29)
```python
@pytest.fixture(scope="module")
def server():
    model = MODEL
    args_for_model = [
        "--enforce-eager",
        "--enable-auto-tool-choice",
        "--tool-call-parser",
        "granite4",
        "--tokenizer",
        "ibm-granite/granite-4.0-h-tiny",
        "--max-model-len",
        "4096",
        "--max-num-seqs",
        "2",
    ]
    with RemoteOpenAIServer(model, args_for_model, max_wait_seconds=480) as server:
        yield server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Module setup / 模块级配置: tools, tools2, messages (L32-L161)
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_acme_region_name_for_transaction_id",
            "description": "Returns ACME transaction/transaction ID information"
            " including ACME regions\n\nArgs:\n    start_time "
            "(str): Start date and time in datetime format "
            '"%Y-%m-%dT%H:%M:%S.%f"\n    end_time (str): End '
            "date and time in datetime format "
            '"%Y-%m-%dT%H:%M:%S.%f"\n    size (int, optional): '
            "Number of ACME Transaction IDs to return\n    "
            "order (str, optional): Sort by most run "
            "transaction IDs. The value can be 'asc' for "
            "ascending or 'desc' for descending\n    "
            "transaction_id (str, optional): ACME Transaction "
# ... 108 lines omitted for brevity ...
        "tool_call_id": "time_range_tool",
    },
]
messages2 = [{"role": "user", "content": "What's stock price for IBM?"}]

messages3 = [{"role": "user", "content": "What's the current weather in New York?"}]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `tools`, `tools2`, `messages`, `messages2`, `messages3`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `tools`、`tools2`、`messages`、`messages2`、`messages3`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_args (L164-L175)
```python
def get_args(client: openai.OpenAI, _tools, _messages, _stop):
    response = client.chat.completions.create(
        model=MODEL,
        messages=_messages,
        temperature=0,
        tools=_tools,
        max_tokens=200,
        stop=_stop,
        tool_choice="auto",
    )

    return response.choices[0].message.tool_calls[0].function.arguments
```
**EN:** This helper encapsulates reusable logic in `get_args`. Key inputs are `client`, `_tools`, `_messages`, `_stop`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_args` 中。 关键输入包括 `client`、`_tools`、`_messages`、`_stop`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_args_streaming (L178-L196)
```python
async def get_args_streaming(
    async_client: openai.AsyncOpenAI, _tools, _messages, _stop
):
    stream = await async_client.chat.completions.create(
        model=MODEL,
        messages=_messages,
        temperature=0,
        tools=_tools,
        max_tokens=200,
        stop=_stop,
        tool_choice="auto",
        stream=True,
    )
    full_call = []
    async for chunk in stream:
        tc = chunk.choices[0].delta.tool_calls
        if tc and tc[0].function.arguments:
            full_call.append(tc[0].function.arguments)
    return "".join(full_call)
```
**EN:** This async helper encapsulates reusable logic in `get_args_streaming`. Key inputs are `async_client`, `_tools`, `_messages`, `_stop`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `get_args_streaming` 中。 关键输入包括 `async_client`、`_tools`、`_messages`、`_stop`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_scenario (L199-L206)
```python
async def run_scenario(server: RemoteOpenAIServer, _tools, _messages, _stop):
    non_streaming = get_args(server.get_client(), _tools, _messages, _stop)
    json.loads(non_streaming)  # verify that it is json loadable
    streaming = await get_args_streaming(
        server.get_async_client(), _tools, _messages, _stop
    )
    json.loads(streaming)
    assert non_streaming == streaming, f"{non_streaming=}, {streaming=}"
```
**EN:** This async helper encapsulates reusable logic in `run_scenario`. Key inputs are `server`, `_tools`, `_messages`, `_stop`. The main assertion is `non_streaming == streaming`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `run_scenario` 中。 关键输入包括 `server`、`_tools`、`_messages`、`_stop`。 核心断言是 `non_streaming == streaming`。

### Test / 测试: test_stop_sequence_interference (L209-L220)
```python
@pytest.mark.asyncio
async def test_stop_sequence_interference(server: RemoteOpenAIServer):
    print("Testing scenario 1")
    await run_scenario(server, tools, messages, "veroniqueprattyushveroniqueprattyush")

    print("Testing scenario 2")
    await run_scenario(
        server, tools2, messages2, "veroniqueprattyushveroniqueprattyush"
    )

    print("Testing scenario 3")
    await run_scenario(server, tools2, messages3, "prattyush")
```
**EN:** This async test validates `test_stop_sequence_interference`. Relevant pytest markers include `asyncio`. Key inputs are `server`.
**CN:** 这个异步测试验证 `test_stop_sequence_interference`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`。

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
- **Third-party / 第三方**: `openai`, `pytest`
- **Local relative imports / 本地相对导入**: `....utils.RemoteOpenAIServer`
