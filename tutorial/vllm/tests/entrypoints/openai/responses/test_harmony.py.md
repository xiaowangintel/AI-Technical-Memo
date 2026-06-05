# test_harmony.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_harmony.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 33 test(s), 2 fixture(s), and 9 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 33 个测试、2 个 fixture，以及 9 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L28)
```python
from __future__ import annotations

import importlib.util
import json
import logging
import time
from typing import Any

import pytest
import pytest_asyncio
import requests
from openai import InternalServerError, NotFoundError, OpenAI
from openai_harmony import Message

from tests.utils import RemoteOpenAIServer

from .conftest import (
    BASE_TEST_ENV,
    events_contain_type,
    has_output_type,
    retry_for_tool_call,
    retry_streaming_for,
    validate_streaming_event_stack,
)
```
**EN:** Imports standard-library modules such as `__future__.annotations`, `importlib.util`, `json`, third-party packages like `openai.InternalServerError`, `openai.NotFoundError`, `openai.OpenAI`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.openai.parser.harmony_utils.get_system_message`, `.conftest.BASE_TEST_ENV`.
**CN:** 导入标准库模块（如 `__future__.annotations`、`importlib.util`、`json`）、第三方包（如 `openai.InternalServerError`、`openai.NotFoundError`、`openai.OpenAI`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.openai.parser.harmony_utils.get_system_message`、`.conftest.BASE_TEST_ENV`）。

### Module setup / 模块级配置: logger, MODEL_NAME, GET_WEATHER_SCHEMA (L30-L48)
```python
logger = logging.getLogger(__name__)

MODEL_NAME = "openai/gpt-oss-20b"

GET_WEATHER_SCHEMA = {
    "type": "function",
    "name": "get_weather",
    "description": "Get current temperature for provided coordinates in celsius.",  # noqa
    "parameters": {
        "type": "object",
        "properties": {
            "latitude": {"type": "number"},
            "longitude": {"type": "number"},
        },
        "required": ["latitude", "longitude"],
        "additionalProperties": False,
    },
    "strict": True,
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `logger`, `MODEL_NAME`, `GET_WEATHER_SCHEMA`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `logger`、`MODEL_NAME`、`GET_WEATHER_SCHEMA`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_weather (L51-L71)
```python
def get_weather(latitude, longitude):
    try:
        response = requests.get(
            f"https://api.open-meteo.com/v1/forecast?"
            f"latitude={latitude}&longitude={longitude}"
            f"&current=temperature_2m,wind_speed_10m"
            f"&hourly=temperature_2m,relative_humidity_2m,"
            f"wind_speed_10m",
            timeout=10,
        )
        data = response.json()
        return data["current"]["temperature_2m"]
    except (requests.RequestException, KeyError) as e:
        logger.warning(
            "External weather API call failed (%s), "
            "returning fake value. This does not affect "
            "test correctness — only the tool-calling "
            "protocol is under test.",
            e,
        )
        return 15.0
```
**EN:** This helper encapsulates reusable logic in `get_weather`. Key inputs are `latitude`, `longitude`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_weather` 中。 关键输入包括 `latitude`、`longitude`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_place_to_travel (L74-L75)
```python
def get_place_to_travel():
    return "Paris"
```
**EN:** This helper encapsulates reusable logic in `get_place_to_travel`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_place_to_travel` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_horoscope (L78-L79)
```python
def get_horoscope(sign):
    return f"{sign}: Next Tuesday you will befriend a baby otter."
```
**EN:** This helper encapsulates reusable logic in `get_horoscope`. Key inputs are `sign`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_horoscope` 中。 关键输入包括 `sign`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: call_function (L82-L93)
```python
def call_function(name, args):
    logger.info("Calling function %s with args %s", name, args)
    dispatch = {
        "get_weather": lambda: get_weather(**args),
        "get_place_to_travel": lambda: get_place_to_travel(),
        "get_horoscope": lambda: get_horoscope(**args),
    }
    if name not in dispatch:
        raise ValueError(f"Unknown function: {name}")
    result = dispatch[name]()
    logger.info("Function %s returned: %s", name, result)
    return result
```
**EN:** This helper encapsulates reusable logic in `call_function`. Key inputs are `name`, `args`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `call_function` 中。 关键输入包括 `name`、`args`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: server (L96-L118)
```python
@pytest.fixture(scope="module")
def server():
    assert importlib.util.find_spec("gpt_oss") is not None, (
        "Harmony tests require gpt_oss package to be installed"
    )
    args = [
        "--enforce-eager",
        "--tool-server",
        "demo",
        "--max_model_len",
        "5000",
    ]
    env_dict = {
        **BASE_TEST_ENV,
        "VLLM_ENABLE_RESPONSES_API_STORE": "1",
        "PYTHON_EXECUTION_BACKEND": "dangerously_use_uv",
        "VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS": (
            "code_interpreter,container,web_search_preview"
        ),
        "VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS": "1",
    }
    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_dict) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes. The main assertion is `importlib.util.find_spec('gpt_oss') is not None`.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。 核心断言是 `importlib.util.find_spec('gpt_oss') is not None`。

### Fixture / 夹具: client (L121-L124)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_basic (L127-L136)
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
```
**EN:** This async test validates `test_basic`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_basic`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_basic_with_instructions (L139-L148)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_basic_with_instructions(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is 123 * 456?",
        instructions="Respond in Korean.",
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_basic_with_instructions`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_basic_with_instructions`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_basic_with_reasoning_effort (L151-L160)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_basic_with_reasoning_effort(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is the capital of South Korea?",
        reasoning={"effort": "low"},
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_basic_with_reasoning_effort`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_basic_with_reasoning_effort`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_max_tokens (L163-L174)
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

### Test / 测试: test_chat (L177-L190)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_chat(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input=[
            {"role": "system", "content": "Respond in Korean."},
            {"role": "user", "content": "Hello!"},
            {"role": "assistant", "content": "Hello! How can I help you today?"},
            {"role": "user", "content": "What is 123 * 456? Explain your answer."},
        ],
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_chat`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_chat`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_chat_with_input_type (L193-L206)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_chat_with_input_type(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input=[
            {
                "role": "user",
                "content": [{"type": "input_text", "text": "What is 123 * 456?"}],
            },
        ],
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_chat_with_input_type`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_chat_with_input_type`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_structured_output (L209-L244)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_structured_output(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input=[
            {"role": "system", "content": "Extract the event information."},
            {
                "role": "user",
                "content": "Alice and Bob are going to a science fair on Friday.",
            },
        ],
        text={
            "format": {
                "type": "json_schema",
                "name": "calendar_event",
                "schema": {
                    "type": "object",
# ... 10 lines omitted for brevity ...
                },
                "description": "A calendar event.",
                "strict": True,
            }
        },
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_structured_output`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_structured_output`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_structured_output_with_parse (L247-L264)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_structured_output_with_parse(client: OpenAI, model_name: str):
    from pydantic import BaseModel

    class CalendarEvent(BaseModel):
        name: str
        date: str
        participants: list[str]

    response = await client.responses.parse(
        model=model_name,
        input="Alice and Bob are going to a science fair on Friday",
        instructions="Extract the event information",
        text_format=CalendarEvent,
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_structured_output_with_parse`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_structured_output_with_parse`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_store (L267-L286)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_store(client: OpenAI, model_name: str):
    for store in [True, False]:
        response = await client.responses.create(
            model=model_name,
            input="What is 123 * 456?",
            store=store,
        )
        assert response is not None

        try:
            _retrieved_response = await client.responses.retrieve(response.id)
            is_not_found = False
        except NotFoundError:
            is_not_found = True

        assert is_not_found == (not store), (
            f"store={store}: expected not_found={not store}, got {is_not_found}"
        )
```
**EN:** This async test validates `test_store`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `is_not_found == (not store)`.
**CN:** 这个异步测试验证 `test_store`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `is_not_found == (not store)`。

### Test / 测试: test_background (L289-L308)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_background(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="What is 123 * 456?",
        background=True,
    )
    assert response is not None

    retries = 0
    max_retries = 30
    while retries < max_retries:
        response = await client.responses.retrieve(response.id)
        if response.status == "completed":
            break
        time.sleep(1)
        retries += 1

    assert response.status == "completed"
```
**EN:** This async test validates `test_background`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_background`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_background_cancel (L311-L323)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_background_cancel(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="Write a long story about a cat.",
        background=True,
    )
    assert response is not None
    time.sleep(1)

    cancelled_response = await client.responses.cancel(response.id)
    assert cancelled_response is not None
```
**EN:** This async test validates `test_background_cancel`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `cancelled_response is not None`.
**CN:** 这个异步测试验证 `test_background_cancel`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `cancelled_response is not None`。

### Test / 测试: test_stateful_multi_turn (L326-L346)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_stateful_multi_turn(client: OpenAI, model_name: str):
    response1 = await client.responses.create(
        model=model_name, input="What is 123 * 456?"
    )
    assert response1.status == "completed"

    response2 = await client.responses.create(
        model=model_name,
        input="What if I increase both numbers by 1?",
        previous_response_id=response1.id,
    )
    assert response2.status == "completed"

    response3 = await client.responses.create(
        model=model_name,
        input="Divide the result by 2.",
        previous_response_id=response2.id,
    )
    assert response3.status == "completed"
```
**EN:** This async test validates `test_stateful_multi_turn`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response1.status == 'completed'` and `response2.status == 'completed'`.
**CN:** 这个异步测试验证 `test_stateful_multi_turn`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response1.status == 'completed'` and `response2.status == 'completed'`。

### Test / 测试: test_streaming_types (L349-L366)
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

### Test / 测试: test_function_calling_with_streaming_types (L369-L388)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_with_streaming_types(
    pairs_of_event_types: dict[str, str], client: OpenAI, model_name: str
):
    """Streaming event nesting for function-calling responses."""

    def _has_function_events(evts: list) -> bool:
        return events_contain_type(evts, "function_call_arguments")

    events = await retry_streaming_for(
        client,
        model=model_name,
        validate_events=_has_function_events,
        input=[{"role": "user", "content": "What's the weather like in Paris today?"}],
        tools=[GET_WEATHER_SCHEMA],
        temperature=0.0,
    )

    validate_streaming_event_stack(events, pairs_of_event_types)
```
**EN:** This async test validates `test_function_calling_with_streaming_types`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `pairs_of_event_types`, `client`, `model_name`.
**CN:** 这个异步测试验证 `test_function_calling_with_streaming_types`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `pairs_of_event_types`、`client`、`model_name`。

### Test / 测试: test_streaming (L391-L491)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("background", [True, False])
async def test_streaming(client: OpenAI, model_name: str, background: bool):
    # TODO: Add back when web search and code interpreter are available in CI
    prompts = [
        "tell me a story about a cat in 20 words",
        "What is 123 * 456? Use python to calculate the result.",
        # "When did Jensen found NVIDIA? Search it and answer the year only.",
    ]

    for prompt in prompts:
        stream = await client.responses.create(
            model=model_name,
            input=prompt,
            reasoning={"effort": "low"},
            tools=[
                # {
# ... 75 lines omitted for brevity ...
            async with await client.responses.retrieve(
                response_id=resp_id, stream=True, starting_after=starting_after
            ) as replay_stream:
                counter = starting_after
                async for event in replay_stream:
                    counter += 1
                    assert event == events[counter]
            assert counter == len(events) - 1
```
**EN:** This async test validates `test_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `background`. It drives client-facing request creation through the API surface under test. The main assertion is `len(events) > 0` and `events[-1].response.output`.
**CN:** 这个异步测试验证 `test_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`background`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(events) > 0` and `events[-1].response.output`。

### Test / 测试: test_web_search (L494-L504)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.skip(reason="Web search tool is not available in CI yet.")
async def test_web_search(client: OpenAI, model_name: str):
    response = await client.responses.create(
        model=model_name,
        input="Who is the president of South Korea as of now?",
        tools=[{"type": "web_search_preview"}],
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_web_search`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`, `skip`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_web_search`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`、`skip`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_code_interpreter (L507-L533)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_code_interpreter(client: OpenAI, model_name: str):
    timeout_value = client.timeout * 3
    client_with_timeout = client.with_options(timeout=timeout_value)

    response = await client_with_timeout.responses.create(
        model=model_name,
        input=(
            "What's the first 4 digits after the decimal point of "
            "cube root of `19910212 * 20250910`? "
            "Show only the digits. The python interpreter is not stateful "
            "and you must print to see the output."
        ),
        tools=[{"type": "code_interpreter", "container": {"type": "auto"}}],
        temperature=0.0,
    )
    assert response is not None
    assert response.status == "completed"
    assert response.usage.output_tokens_details.tool_output_tokens > 0

    for item in response.output:
        if item.type == "message":
            output_string = item.content[0].text
            assert "5846" in output_string, (
                f"Expected '5846' in output, got: {output_string}"
            )
```
**EN:** This async test validates `test_code_interpreter`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_code_interpreter`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_reasoning_item (L536-L555)
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
                    {"type": "reasoning_text", "text": "We need to respond: greeting."}
                ],
                "summary": [],
            },
        ],
        temperature=0.0,
    )
    assert response is not None
    assert response.status == "completed"
```
**EN:** This async test validates `test_reasoning_item`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_reasoning_item`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_function_calling (L558-L607)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling(client: OpenAI, model_name: str):
    tools = [GET_WEATHER_SCHEMA]

    response = await retry_for_tool_call(
        client,
        model=model_name,
        expected_tool_type="function_call",
        input="What's the weather like in Paris today?",
        tools=tools,
        temperature=0.0,
        extra_body={"request_id": "test_function_calling_non_resp"},
    )
    assert response.status == "completed"
    assert has_output_type(response, "function_call"), (
        f"Expected function_call in output, got: "
        f"{[getattr(o, 'type', None) for o in response.output]}"
# ... 24 lines omitted for brevity ...
        model=model_name,
        input="What's the weather like in Paris today?",
        tools=tools,
        previous_response_id=response_2.id,
        temperature=0.0,
    )
    assert response_3.status == "completed"
    assert response_3.output_text is not None
```
**EN:** This async test validates `test_function_calling`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `has_output_type(response, 'function_call')`.
**CN:** 这个异步测试验证 `test_function_calling`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `has_output_type(response, 'function_call')`。

### Test / 测试: test_function_calling_multi_turn (L610-L694)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_multi_turn(client: OpenAI, model_name: str):
    """Multi-tool, multi-turn function calling with retry at API level."""
    tools = [
        {
            "type": "function",
            "name": "get_place_to_travel",
            "description": "Get a random place to travel",
            "parameters": {
                "type": "object",
                "properties": {},
                "required": [],
                "additionalProperties": False,
            },
            "strict": True,
        },
        GET_WEATHER_SCHEMA,
# ... 59 lines omitted for brevity ...
        # Log as warning so it shows up in CI without failing the test.
        assert response_2.output_text is not None
        pytest.xfail(
            "Model went straight to answering instead of calling a "
            "second tool. Valid behaviour but not the expected path."
            "If this happens consistently, the prompt or model may have "
            "changed behaviour."
        )
```
**EN:** This async test validates `test_function_calling_multi_turn`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `has_output_type(response, 'function_call')`.
**CN:** 这个异步测试验证 `test_function_calling_multi_turn`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `has_output_type(response, 'function_call')`。

### Test / 测试: test_function_calling_required (L697-L708)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_required(client: OpenAI, model_name: str):
    tools = [GET_WEATHER_SCHEMA]

    with pytest.raises(InternalServerError):
        await client.responses.create(
            model=model_name,
            input="What's the weather like in Paris today?",
            tools=tools,
            tool_choice="required",
        )
```
**EN:** This async test validates `test_function_calling_required`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_function_calling_required`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_system_message_with_tools (L711-L723)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_system_message_with_tools(client: OpenAI, model_name: str):
    from vllm.entrypoints.openai.parser.harmony_utils import get_system_message

    # Commentary channel should always be present (needed for preambles)
    # regardless of whether custom tools are enabled
    for with_tools in (True, False):
        sys_msg = get_system_message(with_custom_tools=with_tools)
        valid_channels = sys_msg.content[0].channel_config.valid_channels
        assert "commentary" in valid_channels, (
            f"commentary channel missing when with_custom_tools={with_tools}"
        )
```
**EN:** This async test validates `test_system_message_with_tools`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `'commentary' in valid_channels`.
**CN:** 这个异步测试验证 `test_system_message_with_tools`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `'commentary' in valid_channels`。

### Test / 测试: test_function_calling_full_history (L726-L769)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_full_history(client: OpenAI, model_name: str):
    tools = [GET_WEATHER_SCHEMA]

    input_messages = [
        {"role": "user", "content": "What's the weather like in Paris today?"}
    ]

    response = await retry_for_tool_call(
        client,
        model=model_name,
        expected_tool_type="function_call",
        input=input_messages,
        tools=tools,
        temperature=0.0,
    )
    assert response.status == "completed"
# ... 18 lines omitted for brevity ...
    response_2 = await client.responses.create(
        model=model_name,
        input=input_messages,
        tools=tools,
        temperature=0.0,
    )
    assert response_2.status == "completed"
    assert response_2.output_text is not None
```
**EN:** This async test validates `test_function_calling_full_history`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `tool_call is not None`.
**CN:** 这个异步测试验证 `test_function_calling_full_history`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `tool_call is not None`。

### Test / 测试: test_function_calling_with_stream (L772-L850)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_with_stream(client: OpenAI, model_name: str):
    """Function calling via streaming, with retry for non-determinism."""
    tools = [GET_WEATHER_SCHEMA]
    input_list = [
        {"role": "user", "content": "What's the weather like in Paris today?"},
    ]

    def _has_function_call(evts: list) -> bool:
        return any(
            getattr(e, "type", "") == "response.output_item.added"
            and getattr(getattr(e, "item", None), "type", None) == "function_call"
            for e in evts
        )

    events = await retry_streaming_for(
        client,
# ... 53 lines omitted for brevity ...
    async for event in response:
        # check that no function call events in the stream
        assert event.type != "response.function_call_arguments.delta"
        assert event.type != "response.function_call_arguments.done"
        # check that the response contains output text
        if event.type == "response.completed":
            assert len(event.response.output) > 0
            assert event.response.output_text is not None
```
**EN:** This async test validates `test_function_calling_with_stream`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `tool_call is not None` and `event.type != 'response.function_call_arguments.delta'`.
**CN:** 这个异步测试验证 `test_function_calling_with_stream`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `tool_call is not None` and `event.type != 'response.function_call_arguments.delta'`。

### Test / 测试: test_function_calling_no_code_interpreter_events (L853-L904)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_calling_no_code_interpreter_events(
    client: OpenAI, model_name: str
):
    """Verify that function calls don't trigger code_interpreter events.

    Uses retry_streaming_for to handle non-determinism: the model might not
    always produce a function_call, but if it does, code_interpreter events
    should NEVER appear.
    """
    tools = [GET_WEATHER_SCHEMA]
    input_list = [
        {"role": "user", "content": "What's the weather like in Paris today?"},
    ]

    def _has_function_call(evts: list) -> bool:
        return any(
# ... 26 lines omitted for brevity ...
            "Function calls should only emit function_call events."
        )

    # Verify we saw the correct function call event types
    assert (
        "response.function_call_arguments.delta" in event_types_seen
        or "response.function_call_arguments.done" in event_types_seen
    ), "Expected to see function_call_arguments events"
```
**EN:** This async test validates `test_function_calling_no_code_interpreter_events`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. The main assertion is `function_call_found` and `'response.function_call_arguments.delta' in event_types_seen or 'response.function_call_arguments.done' in event_types_seen`.
**CN:** 这个异步测试验证 `test_function_calling_no_code_interpreter_events`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 核心断言是 `function_call_found` and `'response.function_call_arguments.delta' in event_types_seen or 'response.function_call_arguments.done' in event_types_seen`。

### Test / 测试: test_code_interpreter_streaming (L907-L972)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.skip(
    reason="This test is flaky in CI, needs investigation and "
    "potential fixes in the code interpreter MCP implementation."
)
async def test_code_interpreter_streaming(
    client: OpenAI,
    model_name: str,
    pairs_of_event_types: dict[str, str],
):
    tools = [{"type": "code_interpreter", "container": {"type": "auto"}}]
    input_text = (
        "Calculate 123 * 456 using python. "
        "The python interpreter is not stateful and you must "
        "print to see the output."
    )

# ... 40 lines omitted for brevity ...
            assert event.code is not None
        elif (
            event.type == "response.output_item.done"
            and hasattr(event.item, "type")
            and event.item.type == "code_interpreter_call"
        ):
            assert event.item.status == "completed"
            assert event.item.code is not None
```
**EN:** This async test validates `test_code_interpreter_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`, `skip`. Key inputs are `client`, `model_name`, `pairs_of_event_types`. The main assertion is `event.item.status == 'in_progress'` and `event.code is not None`.
**CN:** 这个异步测试验证 `test_code_interpreter_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`、`skip`。 关键输入包括 `client`、`model_name`、`pairs_of_event_types`。 核心断言是 `event.item.status == 'in_progress'` and `event.code is not None`。

### Test / 测试: test_mcp_tool_multi_turn (L975-L1030)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_mcp_tool_multi_turn(client: OpenAI, model_name: str, server):
    """MCP tools work across multiple turns via previous_response_id."""
    tools = [{"type": "mcp", "server_label": "code_interpreter"}]
    instructions = (
        "You must use the Python tool to execute code. Never simulate execution."
    )

    # First turn
    response1 = await retry_for_tool_call(
        client,
        model=model_name,
        expected_tool_type="mcp_call",
        input="Calculate 1234 * 4567 using python tool and print the result.",
        tools=tools,
        temperature=0.0,
        instructions=instructions,
# ... 30 lines omitted for brevity ...
        input="Now divide that result by 2.",
        tools=tools,
        temperature=0.0,
        instructions=instructions,
        previous_response_id=response1.id,
        extra_body={"enable_response_messages": True},
    )
    assert response2.status == "completed"
```
**EN:** This async test validates `test_mcp_tool_multi_turn`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `server`. It drives client-facing request creation through the API surface under test. The main assertion is `response1.status == 'completed'` and `tool_call_found`.
**CN:** 这个异步测试验证 `test_mcp_tool_multi_turn`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`server`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response1.status == 'completed'` and `tool_call_found`。

### Test / 测试: test_output_messages_enabled (L1033-L1045)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_output_messages_enabled(client: OpenAI, model_name: str, server):
    response = await client.responses.create(
        model=model_name,
        input="What is the capital of South Korea?",
        extra_body={"enable_response_messages": True},
    )

    assert response is not None
    assert response.status == "completed"
    assert len(response.input_messages) > 0
    assert len(response.output_messages) > 0
```
**EN:** This async test validates `test_output_messages_enabled`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `server`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_output_messages_enabled`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`server`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

### Test / 测试: test_function_call_with_previous_input_messages (L1048-L1143)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_function_call_with_previous_input_messages(
    client: OpenAI, model_name: str
):
    """Multi-turn function calling using previous_input_messages."""
    tools = [
        {
            "type": "function",
            "name": "get_horoscope",
            "description": "Get today's horoscope for an astrological sign.",
            "parameters": {
                "type": "object",
                "properties": {"sign": {"type": "string"}},
                "required": ["sign"],
                "additionalProperties": False,
            },
            "strict": True,
# ... 70 lines omitted for brevity ...
    assert num_system == 1, f"Expected 1 system message, got {num_system}"
    assert num_developer == 1, f"Expected 1 developer message, got {num_developer}"
    assert num_tool == 1, f"Expected 1 tool message, got {num_tool}"

    output_text = response_2.output_text.lower()
    assert any(kw in output_text for kw in ["aquarius", "otter", "tuesday"]), (
        f"Expected horoscope-related content, got: {response_2.output_text}"
    )
```
**EN:** This async test validates `test_function_call_with_previous_input_messages`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `function_call is not None`.
**CN:** 这个异步测试验证 `test_function_call_with_previous_input_messages`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `function_call is not None`。

### Test / 测试: test_chat_truncation_content_not_null (L1146-L1168)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_chat_truncation_content_not_null(client: OpenAI, model_name: str):
    response = await client.chat.completions.create(
        model=model_name,
        messages=[
            {
                "role": "user",
                "content": (
                    "What is the role of AI in medicine? "
                    "The response must exceed 350 words."
                ),
            }
        ],
        temperature=0.0,
        max_tokens=350,
    )
    choice = response.choices[0]
    assert choice.finish_reason == "length", (
        f"Expected finish_reason='length', got {choice.finish_reason}"
    )
    assert choice.message.content is not None, "Content should not be None"
    assert len(choice.message.content) > 0, "Content should not be empty"
```
**EN:** This async test validates `test_chat_truncation_content_not_null`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.finish_reason == 'length'` and `choice.message.content is not None`.
**CN:** 这个异步测试验证 `test_chat_truncation_content_not_null`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.finish_reason == 'length'` and `choice.message.content is not None`。

### Test / 测试: test_system_prompt_override_no_duplication (L1171-L1191)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_system_prompt_override_no_duplication(client: OpenAI, model_name: str):
    """Hard check: custom system message must not be duplicated."""
    response = await client.responses.create(
        model=model_name,
        input=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Hello"},
        ],
        extra_body={"enable_response_messages": True},
        temperature=0.0,
    )
    assert response.status == "completed"
    assert response.output_text is not None

    num_system = 0
    for message in (Message.from_dict(msg) for msg in response.input_messages):
        if message.author.role == "system":
            num_system += 1
    assert num_system == 1, f"Expected 1 system message, got {num_system}"
```
**EN:** This async test validates `test_system_prompt_override_no_duplication`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `response.output_text is not None`.
**CN:** 这个异步测试验证 `test_system_prompt_override_no_duplication`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `response.output_text is not None`。

### Test / 测试: test_system_prompt_override_follows_personality (L1194-L1225)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.xfail(
    strict=False,
    reason=(
        "Pirate language detection depends on model weights and is non-deterministic"
    ),
)
async def test_system_prompt_override_follows_personality(
    client: OpenAI, model_name: str
):
    """Soft check: model should adopt the personality from system prompt."""
    response = await client.responses.create(
        model=model_name,
        input=[
            {
                "role": "system",
                "content": (
# ... 6 lines omitted for brevity ...
        temperature=0.0,
    )
    assert response.status == "completed"
    output_text = response.output_text.lower()
    pirate_indicators = ["arrr", "matey", "ahoy", "ye", "sea", "aye", "sail"]
    assert any(kw in output_text for kw in pirate_indicators), (
        f"Expected pirate language, got: {response.output_text}"
    )
```
**EN:** This async test validates `test_system_prompt_override_follows_personality`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`, `xfail`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `any((kw in output_text for kw in pirate_indicators))`.
**CN:** 这个异步测试验证 `test_system_prompt_override_follows_personality`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`、`xfail`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `any((kw in output_text for kw in pirate_indicators))`。

### Test / 测试: test_system_prompt_structured_content (L1228-L1247)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_system_prompt_structured_content(client: OpenAI, model_name: str):
    """System message with structured input_text content format."""
    response = await client.responses.create(
        model=model_name,
        input=[
            {
                "role": "system",
                "content": [
                    {"type": "input_text", "text": "You are a helpful assistant."}
                ],
            },
            {"role": "user", "content": "What is 2 + 2?"},
        ],
        temperature=0.0,
    )
    assert response is not None
    assert response.status == "completed"
    assert response.output_text is not None
```
**EN:** This async test validates `test_system_prompt_structured_content`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `response is not None` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_system_prompt_structured_content`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response is not None` and `response.status == 'completed'`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `__future__.annotations`, `importlib.util`, `json`, `logging`, `time`, `typing.Any`
- **Third-party / 第三方**: `openai.InternalServerError`, `openai.NotFoundError`, `openai.OpenAI`, `openai_harmony.Message`, `pydantic.BaseModel`, `pytest`, `pytest_asyncio`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.openai.parser.harmony_utils.get_system_message`
- **Local relative imports / 本地相对导入**: `.conftest.BASE_TEST_ENV`, `.conftest.events_contain_type`, `.conftest.has_output_type`, `.conftest.retry_for_tool_call`, `.conftest.retry_streaming_for`, `.conftest.validate_streaming_event_stack`
