# test_hermes_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/tool_parsers/test_hermes_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 4 test(s), 3 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 4 个测试、3 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import json

import openai
import pytest
import pytest_asyncio
from huggingface_hub import snapshot_download
from typing_extensions import TypedDict

from vllm.tool_parsers.abstract_tool_parser import ToolParser
from vllm.tool_parsers.granite4_tool_parser import Granite4ToolParser
from vllm.tool_parsers.hermes_tool_parser import Hermes2ProToolParser

from ....utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `huggingface_hub.snapshot_download`, `openai`, `pytest`, project helpers such as `vllm.tool_parsers.abstract_tool_parser.ToolParser`, `vllm.tool_parsers.granite4_tool_parser.Granite4ToolParser`, `vllm.tool_parsers.hermes_tool_parser.Hermes2ProToolParser`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `huggingface_hub.snapshot_download`、`openai`、`pytest`）、项目内辅助模块（如 `vllm.tool_parsers.abstract_tool_parser.ToolParser`、`vllm.tool_parsers.granite4_tool_parser.Granite4ToolParser`、`vllm.tool_parsers.hermes_tool_parser.Hermes2ProToolParser`）。

### Module setup / 模块级配置: LORA_MODEL, TOOLS (L18-L42)
```python
LORA_MODEL = "minpeter/LoRA-Llama-3.2-1B-tool-vllm-ci"

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA",
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                    },
                },
                "required": ["location"],
            },
        },
    }
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `LORA_MODEL`, `TOOLS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `LORA_MODEL`、`TOOLS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: ServerConfig (L45-L49)
```python
class ServerConfig(TypedDict, total=False):
    model: str
    arguments: list[str]
    model_arg: str
    tool_parser: ToolParser
```
**EN:** This class groups related scenarios in `ServerConfig`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `ServerConfig` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Module setup / 模块级配置: CONFIGS (L52-L86)
```python
CONFIGS: dict[str, ServerConfig] = {
    "llama": {
        "model": "meta-llama/Llama-3.2-1B-Instruct",
        "arguments": [
            "--enforce-eager",
            "--enable-auto-tool-choice",
            "--tool-call-parser",
            "hermes",
            "--enable-lora",
            "--lora-modules",
            f"{LORA_MODEL}={LORA_MODEL}",
            "--tokenizer",
            f"{LORA_MODEL}",
        ],
        "model_arg": LORA_MODEL,
        "tool_parser": Hermes2ProToolParser,
# ... 13 lines omitted for brevity ...
            "2",
        ],
        "model_arg": "ibm-granite/granite-4.0-h-tiny",
        "tool_parser": Granite4ToolParser,
    },
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `CONFIGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `CONFIGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server_config (L90-L96)
```python
@pytest.fixture(scope="session", params=CONFIGS.keys())
def server_config(request):
    config = CONFIGS[request.param]

    # download model and tokenizer using transformers
    snapshot_download(config["model"])
    yield CONFIGS[request.param]
```
**EN:** This fixture prepares `server_config` for dependent tests. Key inputs are `request`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server_config`。 关键输入包括 `request`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: server (L99-L104)
```python
@pytest.fixture(scope="module")
def server(request, server_config: ServerConfig):
    model = server_config["model"]
    args_for_model = server_config["arguments"]
    with RemoteOpenAIServer(model, args_for_model, max_wait_seconds=480) as server:
        yield server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `request`, `server_config`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `request`、`server_config`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L107-L110)
```python
@pytest_asyncio.fixture
async def client(server: RemoteOpenAIServer):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Module setup / 模块级配置: PRODUCT_TOOLS, MESSAGES, PRODUCT_MESSAGES (L113-L146)
```python
PRODUCT_TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "get_product_info",
            "description": "Get detailed information of a product based on its "
            "product ID.",
            "parameters": {
                "type": "object",
                "properties": {
                    "inserted": {
                        "type": "boolean",
                        "description": "inserted.",
                    },
                    "product_id": {
                        "type": "integer",
# ... 12 lines omitted for brevity ...
    {
        "role": "user",
        "content": "Hi! Do you have any detailed information about the product id "
        "7355608 and inserted true?",
    }
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `PRODUCT_TOOLS`, `MESSAGES`, `PRODUCT_MESSAGES`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `PRODUCT_TOOLS`、`MESSAGES`、`PRODUCT_MESSAGES`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_non_streaming_tool_call (L149-L179)
```python
@pytest.mark.asyncio
async def test_non_streaming_tool_call(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    """Test tool call in non-streaming mode."""

    response = await client.chat.completions.create(
        model=server_config["model_arg"],
        messages=MESSAGES,
        tools=TOOLS,
        tool_choice="auto",
        temperature=0.0,
    )

    assert response.choices
    choice = response.choices[0]
    message = choice.message

# ... 5 lines omitted for brevity ...
    assert tool_call.function.name == "get_current_weather"

    arguments = json.loads(tool_call.function.arguments)
    assert "location" in arguments
    assert "Boston" in arguments["location"]
    print("\n[Non-Streaming Test Passed]")
    print(f"Tool Call: {tool_call.function.name}")
    print(f"Arguments: {arguments}")
```
**EN:** This async test validates `test_non_streaming_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `server_config`. It drives client-facing request creation through the API surface under test. The main assertion is `response.choices` and `choice.finish_reason == 'tool_calls'`.
**CN:** 这个异步测试验证 `test_non_streaming_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`server_config`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.choices` and `choice.finish_reason == 'tool_calls'`。

### Test / 测试: test_streaming_tool_call (L182-L226)
```python
@pytest.mark.asyncio
async def test_streaming_tool_call(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    """Test tool call in streaming mode."""

    stream = await client.chat.completions.create(
        model=server_config["model_arg"],
        messages=MESSAGES,
        tools=TOOLS,
        tool_choice="auto",
        temperature=0.0,
        stream=True,
    )

    tool_call_chunks = {}
    async for chunk in stream:
        if not chunk.choices:
# ... 19 lines omitted for brevity ...
    assert reconstructed_tool_call["name"] == "get_current_weather"

    arguments = json.loads(reconstructed_tool_call["arguments"])
    assert "location" in arguments
    assert "Boston" in arguments["location"]
    print("\n[Streaming Test Passed]")
    print(f"Reconstructed Tool Call: {reconstructed_tool_call['name']}")
    print(f"Reconstructed Arguments: {arguments}")
```
**EN:** This async test validates `test_streaming_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `server_config`. It drives client-facing request creation through the API surface under test. The main assertion is `len(tool_call_chunks) == 1` and `reconstructed_tool_call['name'] == 'get_current_weather'`.
**CN:** 这个异步测试验证 `test_streaming_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`server_config`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(tool_call_chunks) == 1` and `reconstructed_tool_call['name'] == 'get_current_weather'`。

### Test / 测试: test_non_streaming_product_tool_call (L229-L268)
```python
@pytest.mark.asyncio
async def test_non_streaming_product_tool_call(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    """Test tool call integer and boolean parameters in non-streaming mode."""

    response = await client.chat.completions.create(
        model=server_config["model_arg"],
        messages=PRODUCT_MESSAGES,
        tools=PRODUCT_TOOLS,
        tool_choice="auto",
        temperature=0.66,
    )

    assert response.choices
    choice = response.choices[0]
    message = choice.message

# ... 14 lines omitted for brevity ...
    assert isinstance(product_id, int)
    assert product_id == 7355608
    assert isinstance(inserted, bool)
    assert inserted is True

    print("\n[Non-Streaming Product Test Passed]")
    print(f"Tool Call: {tool_call.function.name}")
    print(f"Arguments: {arguments}")
```
**EN:** This async test validates `test_non_streaming_product_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `server_config`. It drives client-facing request creation through the API surface under test. The main assertion is `response.choices` and `choice.finish_reason == 'tool_calls'`.
**CN:** 这个异步测试验证 `test_non_streaming_product_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`server_config`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.choices` and `choice.finish_reason == 'tool_calls'`。

### Test / 测试: test_streaming_product_tool_call (L271-L325)
```python
@pytest.mark.asyncio
async def test_streaming_product_tool_call(
    client: openai.AsyncOpenAI, server_config: ServerConfig
):
    """Test tool call integer and boolean parameters in streaming mode."""

    stream = await client.chat.completions.create(
        model=server_config["model_arg"],
        messages=PRODUCT_MESSAGES,
        tools=PRODUCT_TOOLS,
        tool_choice="auto",
        temperature=0.66,
        stream=True,
    )

    tool_call_chunks = {}
    async for chunk in stream:
        if not chunk.choices:
# ... 29 lines omitted for brevity ...
    assert isinstance(product_id, int)
    assert product_id == 7355608
    assert isinstance(inserted, bool)
    assert inserted is True

    print("\n[Streaming Product Test Passed]")
    print(f"Reconstructed Tool Call: {reconstructed_tool_call['name']}")
    print(f"Reconstructed Arguments: {arguments}")
```
**EN:** This async test validates `test_streaming_product_tool_call`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `server_config`. It drives client-facing request creation through the API surface under test. The main assertion is `len(tool_call_chunks) == 1` and `reconstructed_tool_call['name'] == 'get_product_info'`.
**CN:** 这个异步测试验证 `test_streaming_product_tool_call`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`server_config`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(tool_call_chunks) == 1` and `reconstructed_tool_call['name'] == 'get_product_info'`。

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
- **Third-party / 第三方**: `huggingface_hub.snapshot_download`, `openai`, `pytest`, `pytest_asyncio`, `typing_extensions.TypedDict`
- **Project / 项目内**: `vllm.tool_parsers.abstract_tool_parser.ToolParser`, `vllm.tool_parsers.granite4_tool_parser.Granite4ToolParser`, `vllm.tool_parsers.hermes_tool_parser.Hermes2ProToolParser`
- **Local relative imports / 本地相对导入**: `....utils.RemoteOpenAIServer`
