# test_completion_with_function_calling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_completion_with_function_calling.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 6 test(s), 4 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 6 个测试、4 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L13)
```python
import datetime
import json

import jsonschema
import openai  # use the official client for correctness check
import pytest
import pytest_asyncio

# downloading lora to test lora requests
from tests.utils import ROCM_ENV_OVERRIDES, ROCM_EXTRA_ARGS, RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `datetime`, `json`, third-party packages like `jsonschema`, `openai`, `pytest`, project helpers such as `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `datetime`、`json`）、第三方包（如 `jsonschema`、`openai`、`pytest`）、项目内辅助模块（如 `tests.utils.ROCM_ENV_OVERRIDES`、`tests.utils.ROCM_EXTRA_ARGS`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, tools, messages (L16-L125)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"


tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "The city to find the weather for, e.g. "
                        "'Vienna'",
# ... 88 lines omitted for brevity ...
    {
        "role": "user",
        "content": "Can you tell me what the current weather is in Berlin and the "
        "forecast for the next 5 days, in fahrenheit?",
    },
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `tools`, `messages`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`tools`、`messages`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L128-L149)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "half",
        "--enable-auto-tool-choice",
        "--structured-outputs-config.backend",
        "xgrammar",
        "--tool-call-parser",
        "hermes",
        "--reasoning-parser",
        "qwen3",
        "--gpu-memory-utilization",
        "0.4",
        "--enforce-eager",
    ] + ROCM_EXTRA_ARGS

    with RemoteOpenAIServer(
        MODEL_NAME, args, env_dict=ROCM_ENV_OVERRIDES
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L152-L155)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_function_tool_use (L158-L215)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("stream", [True, False])
@pytest.mark.parametrize(
    "tool_choice",
    [
        "auto",
        "required",
        {"type": "function", "function": {"name": "get_current_weather"}},
    ],
)
@pytest.mark.parametrize("enable_thinking", [True, False])
async def test_function_tool_use(
    client: openai.AsyncOpenAI,
    model_name: str,
    stream: bool,
    tool_choice: str | dict,
    enable_thinking: bool,
# ... 32 lines omitted for brevity ...
                ):
                    reasoning.append(chunk.choices[0].delta.reasoning)
                if chunk.choices[0].delta.tool_calls:
                    output.extend(chunk.choices[0].delta.tool_calls)

        assert len(output) > 0
        if enable_thinking:
            assert len(reasoning) > 0
```
**EN:** This async test validates `test_function_tool_use`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `stream`, `tool_choice`, `enable_thinking`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_completion.choices[0].message.tool_calls is not None` and `len(chat_completion.choices[0].message.tool_calls) > 0`.
**CN:** 这个异步测试验证 `test_function_tool_use`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`stream`、`tool_choice`、`enable_thinking`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_completion.choices[0].message.tool_calls is not None` and `len(chat_completion.choices[0].message.tool_calls) > 0`。

### Fixture / 夹具: k2_server (L218-L243)
```python
@pytest.fixture(scope="module")
def k2_server():
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "half",
        "--enable-auto-tool-choice",
        "--structured-outputs-config.backend",
        "xgrammar",
        "--tool-call-parser",
        "hermes",
        "--reasoning-parser",
        "qwen3",
        "--gpu-memory-utilization",
        "0.4",
    ] + ROCM_EXTRA_ARGS
    # Test kimi_k2 tool use tool_id format by overriding model_type.
    # is_deepseek_mla safely returns False via getattr when kv_lora_rank
    # is absent from the underlying config.
    with RemoteOpenAIServer(
        MODEL_NAME,
        args,
        env_dict=ROCM_ENV_OVERRIDES,
        override_hf_configs={"model_type": "kimi_k2"},
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `k2_server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `k2_server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: k2_client (L246-L249)
```python
@pytest_asyncio.fixture
async def k2_client(k2_server):
    async with k2_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `k2_client` for dependent tests. Key inputs are `k2_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `k2_client`。 关键输入包括 `k2_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_tool_id_kimi_k2 (L252-L288)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("stream", [True, False])
@pytest.mark.parametrize("tool_choice", ["required"])
async def test_tool_id_kimi_k2(
    k2_client: openai.AsyncOpenAI, model_name: str, stream: bool, tool_choice: str
):
    if not stream:
        # Non-streaming test
        chat_completion = await k2_client.chat.completions.create(
            messages=messages, model=model_name, tools=tools, tool_choice=tool_choice
        )
        assert chat_completion.choices[0].message.tool_calls is not None
        assert len(chat_completion.choices[0].message.tool_calls) > 0
        assert chat_completion.choices[0].message.tool_calls[0].id in [
            "functions.get_current_weather:0",
            "functions.get_forecast:1",
        ]
# ... 11 lines omitted for brevity ...
        async for chunk in output_stream:
            if chunk.choices and chunk.choices[0].delta.tool_calls:
                output.extend(chunk.choices[0].delta.tool_calls)
        for o in output:
            assert o.id is None or o.id in [
                "functions.get_current_weather:0",
                "functions.get_forecast:1",
            ]
```
**EN:** This async test validates `test_tool_id_kimi_k2`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `k2_client`, `model_name`, `stream`, `tool_choice`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_completion.choices[0].message.tool_calls is not None` and `len(chat_completion.choices[0].message.tool_calls) > 0`.
**CN:** 这个异步测试验证 `test_tool_id_kimi_k2`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `k2_client`、`model_name`、`stream`、`tool_choice`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_completion.choices[0].message.tool_calls is not None` and `len(chat_completion.choices[0].message.tool_calls) > 0`。

### Test / 测试: test_no_args_tool_call (L291-L376)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("arguments", ["{}", ""])
async def test_no_args_tool_call(
    client: openai.AsyncOpenAI, model_name: str, arguments: str
):
    # Step 1: Define a tool that requires no parameters
    tools = [
        {
            "type": "function",
            "function": {
                "name": "get_current_time",
                "description": (
                    "Get the current date and time. Call this when the user "
                    "asks what time or date it is. No parameters needed."
                ),
                "parameters": {
                    "type": "object",
# ... 60 lines omitted for brevity ...
            assert (
                final_response.choices[0].message.content is not None
                and final_response.choices[0].message.content.strip() != ""
            )

    else:
        # No tool called — just print model's direct reply
        assert message.content is not None
```
**EN:** This async test validates `test_no_args_tool_call`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `arguments`. It drives client-facing request creation through the API surface under test. The main assertion is `message.content is not None` and `final_response.choices[0].message.content is not None and final_response.choices[0].message.content.strip() != ''`.
**CN:** 这个异步测试验证 `test_no_args_tool_call`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`arguments`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message.content is not None` and `final_response.choices[0].message.content is not None and final_response.choices[0].message.content.strip() != ''`。

### Test / 测试: test_named_tool_use (L379-L454)
```python
@pytest.mark.asyncio
async def test_named_tool_use(
    client: openai.AsyncOpenAI,
    sample_json_schema,
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": (
                "Give an example JSON for an employee profile using the specified tool."
            ),
        },
    ]
    tools = [
        {
            "type": "function",
            "function": {
# ... 50 lines omitted for brevity ...
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    json2 = json.loads("".join(output))
    jsonschema.validate(instance=json2, schema=sample_json_schema)
    assert json1["name"] != json2["name"]
    assert json1["age"] != json2["age"]
```
**EN:** This async test validates `test_named_tool_use`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_json_schema`. It drives client-facing request creation through the API surface under test. The main assertion is `len(message.content) == 0` and `finish_reason_count == 1`.
**CN:** 这个异步测试验证 `test_named_tool_use`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_json_schema`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(message.content) == 0` and `finish_reason_count == 1`。

### Test / 测试: test_inconsistent_tool_choice_and_tools (L457-L517)
```python
@pytest.mark.asyncio
async def test_inconsistent_tool_choice_and_tools(
    client: openai.AsyncOpenAI, sample_json_schema
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": f"Give an example JSON for an employee profile that "
            f"fits this schema: {sample_json_schema}",
        },
    ]

    with pytest.raises(openai.BadRequestError):
        await client.chat.completions.create(
            model=MODEL_NAME,
            messages=messages,
            max_completion_tokens=1000,
# ... 35 lines omitted for brevity ...
                        "name": "dummy_function_name",
                        "description": "This is a dummy function",
                        "parameters": sample_json_schema,
                    },
                }
            ],
            tool_choice={},
        )
```
**EN:** This async test validates `test_inconsistent_tool_choice_and_tools`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_json_schema`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_inconsistent_tool_choice_and_tools`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_json_schema`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_max_tokens_with_tool_choice_required (L520-L546)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "tool_choice",
    ["required", {"type": "function", "function": {"name": "get_current_weather"}}],
)
async def test_max_tokens_with_tool_choice_required(
    client: openai.AsyncOpenAI, tool_choice
):
    """ """
    models = await client.models.list()
    model_name: str = models.data[0].id

    # This combination previously crashed the engine
    chat_completion = await client.chat.completions.create(
        messages=messages,
        temperature=0,
        max_completion_tokens=1,
        model=model_name,
        tools=tools,
        tool_choice=tool_choice,
    )
    # When `tool_choice="required"` and the tokens of `tools` exceed `max_tokens`,
    # both `tool_calls` and `content` should be empty.
    # This behavior should be consistent with OpenAI.
    choice = chat_completion.choices[0]
    assert choice.finish_reason == "length"
    assert len(choice.message.tool_calls) == 0
```
**EN:** This async test validates `test_max_tokens_with_tool_choice_required`. It uses parameterization over `tool_choice`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `tool_choice`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.finish_reason == 'length'` and `len(choice.message.tool_calls) == 0`.
**CN:** 这个异步测试验证 `test_max_tokens_with_tool_choice_required`。 它通过参数化组合 `tool_choice`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`tool_choice`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.finish_reason == 'length'` and `len(choice.message.tool_calls) == 0`。

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
- **Stdlib / 标准库**: `datetime`, `json`
- **Third-party / 第三方**: `jsonschema`, `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`
