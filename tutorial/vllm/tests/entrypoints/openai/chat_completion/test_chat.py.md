# test_chat.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_chat.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 35 test(s), 3 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 35 个测试、3 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L21)
```python
import json
from collections import defaultdict

import jsonschema
import openai  # use the official client for correctness check
import pytest
import pytest_asyncio
import regex as re
import requests
import torch
from openai import BadRequestError

from tests.utils import RemoteOpenAIServer
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.sampling_params import SamplingParams
```
**EN:** Imports standard-library modules such as `collections.defaultdict`, `json`, third-party packages like `huggingface_hub.snapshot_download`, `jsonschema`, `openai`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`, `vllm.envs`.
**CN:** 导入标准库模块（如 `collections.defaultdict`、`json`）、第三方包（如 `huggingface_hub.snapshot_download`、`jsonschema`、`openai`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`、`vllm.envs`）。

### Module setup / 模块级配置: MODEL_NAME (L24-L24)
```python
MODEL_NAME = "HuggingFaceH4/zephyr-7b-beta"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: zephyr_lora_files (L27-L32)
```python
@pytest.fixture(scope="module")
def zephyr_lora_files():
    """Download zephyr LoRA files once per test session."""
    from huggingface_hub import snapshot_download

    return snapshot_download(repo_id="typeof/zephyr-7b-beta-lora")
```
**EN:** This fixture prepares `zephyr_lora_files` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `zephyr_lora_files`。

### Fixture / 夹具: server (L35-L57)
```python
@pytest.fixture(scope="module")
def server(zephyr_lora_files):
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "8192",
        "--enforce-eager",
        # lora config below
        "--enable-lora",
        "--lora-modules",
        f"zephyr-lora={zephyr_lora_files}",
        "--max-lora-rank",
        "64",
        "--max-cpu-loras",
        "2",
        "--max-num-seqs",
        "128",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `zephyr_lora_files`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `zephyr_lora_files`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L60-L63)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_no_logprobs_chat (L66-L87)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    # first test base model, then test loras
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_no_logprobs_chat(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=5,
        temperature=0.0,
        logprobs=False,
    )

    choice = chat_completion.choices[0]
    assert choice.logprobs is None
```
**EN:** This async test validates `test_no_logprobs_chat`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is None`.
**CN:** 这个异步测试验证 `test_no_logprobs_chat`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is None`。

### Test / 测试: test_zero_logprobs_chat (L90-L114)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    # just test 1 lora hereafter
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_zero_logprobs_chat(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=5,
        temperature=0.0,
        logprobs=True,
        top_logprobs=0,
    )

    choice = chat_completion.choices[0]
    assert choice.logprobs is not None
    assert choice.logprobs.content is not None
    assert len(choice.logprobs.content[0].top_logprobs) == 0
```
**EN:** This async test validates `test_zero_logprobs_chat`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is not None` and `choice.logprobs.content is not None`.
**CN:** 这个异步测试验证 `test_zero_logprobs_chat`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is not None` and `choice.logprobs.content is not None`。

### Test / 测试: test_some_logprobs_chat (L117-L140)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_some_logprobs_chat(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=5,
        temperature=0.0,
        logprobs=True,
        top_logprobs=5,
    )

    choice = chat_completion.choices[0]
    assert choice.logprobs is not None
    assert choice.logprobs.content is not None
    assert len(choice.logprobs.content[0].top_logprobs) == 5
```
**EN:** This async test validates `test_some_logprobs_chat`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `choice.logprobs is not None` and `choice.logprobs.content is not None`.
**CN:** 这个异步测试验证 `test_some_logprobs_chat`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice.logprobs is not None` and `choice.logprobs.content is not None`。

### Test / 测试: test_too_many_chat_logprobs (L143-L182)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_too_many_chat_logprobs(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    # Default max_logprobs is 20, so this should raise an error
    with pytest.raises((openai.BadRequestError, openai.APIError)):
        stream = await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            logprobs=True,
# ... 14 lines omitted for brevity ...
        )

    # the server should still work afterwards
    chat_completion = await client.chat.completions.create(
        model=model_name, messages=messages, max_completion_tokens=10, stream=False
    )
    message = chat_completion.choices[0].message
    assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_too_many_chat_logprobs`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `message.content is not None and len(message.content) >= 0`.
**CN:** 这个异步测试验证 `test_too_many_chat_logprobs`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message.content is not None and len(message.content) >= 0`。

### Test / 测试: test_prompt_logprobs_chat (L185-L218)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name, prompt_logprobs",
    [(MODEL_NAME, 1), (MODEL_NAME, 0), (MODEL_NAME, -1), (MODEL_NAME, None)],
)
async def test_prompt_logprobs_chat(
    client: openai.AsyncOpenAI, model_name: str, prompt_logprobs: int | None
):
    params: dict = {
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Who won the world series in 2020?"},
            {
                "role": "assistant",
                "content": "The Los Angeles Dodgers won the World Series in 2020.",
            },
            {"role": "user", "content": "Where was it played?"},
        ],
# ... 8 lines omitted for brevity ...
            await client.chat.completions.create(**params)
    else:
        completion = await client.chat.completions.create(**params)
        if prompt_logprobs is not None:
            assert completion.prompt_logprobs is not None
            assert len(completion.prompt_logprobs) > 0
        else:
            assert completion.prompt_logprobs is None
```
**EN:** This async test validates `test_prompt_logprobs_chat`. It uses parameterization over `model_name`, `prompt_logprobs`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `prompt_logprobs`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `completion.prompt_logprobs is not None` and `len(completion.prompt_logprobs) > 0`.
**CN:** 这个异步测试验证 `test_prompt_logprobs_chat`。 它通过参数化组合 `model_name`、`prompt_logprobs`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`prompt_logprobs`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion.prompt_logprobs is not None` and `len(completion.prompt_logprobs) > 0`。

### Test / 测试: test_more_than_one_prompt_logprobs_chat (L221-L249)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_more_than_one_prompt_logprobs_chat(
    client: openai.AsyncOpenAI, model_name: str
):
    params: dict = {
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Who won the world series in 2020?"},
            {
                "role": "assistant",
                "content": "The Los Angeles Dodgers won the World Series in 2020.",
            },
            {"role": "user", "content": "Where was it played?"},
        ],
        "model": model_name,
        "extra_body": {"prompt_logprobs": 1},
    }

    completion_1 = await client.chat.completions.create(**params)

    params["extra_body"] = {"prompt_logprobs": 2}
    completion_2 = await client.chat.completions.create(**params)

    assert len(completion_1.prompt_logprobs[3]) == 1
    assert len(completion_2.prompt_logprobs[3]) == 2
```
**EN:** This async test validates `test_more_than_one_prompt_logprobs_chat`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(completion_1.prompt_logprobs[3]) == 1` and `len(completion_2.prompt_logprobs[3]) == 2`.
**CN:** 这个异步测试验证 `test_more_than_one_prompt_logprobs_chat`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(completion_1.prompt_logprobs[3]) == 1` and `len(completion_2.prompt_logprobs[3]) == 2`。

### Test / 测试: test_single_chat_session (L252-L293)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_single_chat_session(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]
    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=5,
        logprobs=True,
        top_logprobs=5,
    )
# ... 16 lines omitted for brevity ...
    messages.append({"role": "user", "content": "express your result in json"})
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=5,
    )
    message = chat_completion.choices[0].message
    assert message.content is not None and len(message.content) >= 0
```
**EN:** This async test validates `test_single_chat_session`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_completion.id is not None` and `len(chat_completion.choices) == 1`.
**CN:** 这个异步测试验证 `test_single_chat_session`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_completion.id is not None` and `len(chat_completion.choices) == 1`。

### Test / 测试: test_chat_streaming (L296-L340)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    # just test 1 lora hereafter
    "model_name",
    [MODEL_NAME, "zephyr-lora"],
)
async def test_chat_streaming(client: openai.AsyncOpenAI, model_name: str):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.0,
# ... 19 lines omitted for brevity ...
            chunks.append(delta.content)
        if chunk.choices[0].finish_reason is not None:
            finish_reason_count += 1
    # finish reason should only return in last block
    assert finish_reason_count == 1
    assert chunk.choices[0].finish_reason == stop_reason
    assert delta.content
    assert "".join(chunks) == output
```
**EN:** This async test validates `test_chat_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`.
**CN:** 这个异步测试验证 `test_chat_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`。

### Test / 测试: test_chat_completion_stream_options (L343-L445)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    ["HuggingFaceH4/zephyr-7b-beta", "zephyr-lora"],
)
async def test_chat_completion_stream_options(
    client: openai.AsyncOpenAI, model_name: str
):
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"},
    ]

    # Test stream=True, stream_options={"include_usage": False}
    stream = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
# ... 77 lines omitted for brevity ...
            )
        )
        assert chunk.usage.total_tokens == (
            chunk.usage.prompt_tokens + chunk.usage.completion_tokens
        )
        last_completion_tokens = chunk.usage.completion_tokens

    assert last_completion_tokens == 10
```
**EN:** This async test validates `test_chat_completion_stream_options`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `last_completion_tokens == 10` and `chunk.usage is None`.
**CN:** 这个异步测试验证 `test_chat_completion_stream_options`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `last_completion_tokens == 10` and `chunk.usage is None`。

### Test / 测试: test_structured_outputs_choice_chat (L448-L485)
```python
@pytest.mark.asyncio
async def test_structured_outputs_choice_chat(
    client: openai.AsyncOpenAI,
    sample_structured_outputs_choices,
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": "The best language for type-safe systems programming is ",
        },
    ]
    chat_completion = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.7,
        extra_body=dict(
# ... 12 lines omitted for brevity ...
        temperature=0.7,
        extra_body=dict(
            structured_outputs={"choice": sample_structured_outputs_choices}
        ),
    )
    choice2 = chat_completion.choices[0].message.content
    assert choice2 in sample_structured_outputs_choices
    assert choice1 != choice2
```
**EN:** This async test validates `test_structured_outputs_choice_chat`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_structured_outputs_choices`. It drives client-facing request creation through the API surface under test. The main assertion is `choice1 in sample_structured_outputs_choices` and `choice2 in sample_structured_outputs_choices`.
**CN:** 这个异步测试验证 `test_structured_outputs_choice_chat`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_structured_outputs_choices`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `choice1 in sample_structured_outputs_choices` and `choice2 in sample_structured_outputs_choices`。

### Test / 测试: test_structured_outputs_json_chat (L488-L527)
```python
@pytest.mark.asyncio
async def test_structured_outputs_json_chat(
    client: openai.AsyncOpenAI,
    sample_json_schema,
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": f"Give an example JSON for an employee profile that "
            f"fits this schema: {sample_json_schema}",
        },
    ]
    chat_completion = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        max_completion_tokens=1000,
        extra_body=dict(structured_outputs={"json": sample_json_schema}),
# ... 14 lines omitted for brevity ...
        extra_body=dict(structured_outputs={"json": sample_json_schema}),
    )
    message = chat_completion.choices[0].message
    assert message.content is not None
    json2 = json.loads(message.content)
    jsonschema.validate(instance=json2, schema=sample_json_schema)
    assert json1["name"] != json2["name"]
    assert json1["age"] != json2["age"]
```
**EN:** This async test validates `test_structured_outputs_json_chat`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_json_schema`. It drives client-facing request creation through the API surface under test. The main assertion is `message.content is not None` and `message.content is not None`.
**CN:** 这个异步测试验证 `test_structured_outputs_json_chat`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_json_schema`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message.content is not None` and `message.content is not None`。

### Test / 测试: test_structured_outputs_regex_chat (L530-L563)
```python
@pytest.mark.asyncio
async def test_structured_outputs_regex_chat(
    client: openai.AsyncOpenAI,
    sample_regex,
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": f"Give an example IP address with this regex: {sample_regex}",
        },
    ]
    chat_completion = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        max_completion_tokens=20,
        extra_body=dict(structured_outputs={"regex": sample_regex}),
    )
# ... 8 lines omitted for brevity ...
        messages=messages,
        max_completion_tokens=20,
        extra_body=dict(structured_outputs={"regex": sample_regex}),
    )
    ip2 = chat_completion.choices[0].message.content
    assert ip2 is not None
    assert re.fullmatch(sample_regex, ip2) is not None
    assert ip1 != ip2
```
**EN:** This async test validates `test_structured_outputs_regex_chat`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_regex`. It drives client-facing request creation through the API surface under test. The main assertion is `ip1 is not None` and `re.fullmatch(sample_regex, ip1) is not None`.
**CN:** 这个异步测试验证 `test_structured_outputs_regex_chat`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_regex`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `ip1 is not None` and `re.fullmatch(sample_regex, ip1) is not None`。

### Test / 测试: test_structured_outputs_type_error (L566-L581)
```python
@pytest.mark.asyncio
async def test_structured_outputs_type_error(client: openai.AsyncOpenAI):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": "The best language for type-safe systems programming is ",
        },
    ]

    with pytest.raises(openai.BadRequestError):
        _ = await client.chat.completions.create(
            model=MODEL_NAME,
            messages=messages,
            extra_body=dict(structured_outputs={"regex": {1: "Python", 2: "C++"}}),
        )
```
**EN:** This async test validates `test_structured_outputs_type_error`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_structured_outputs_type_error`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_structured_outputs_choice_chat_logprobs (L584-L612)
```python
@pytest.mark.asyncio
async def test_structured_outputs_choice_chat_logprobs(
    client: openai.AsyncOpenAI, sample_structured_outputs_choices
):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {
            "role": "user",
            "content": "The best language for type-safe systems programming is ",
        },
    ]
    chat_completion = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        max_completion_tokens=10,
        logprobs=True,
        top_logprobs=5,
        extra_body=dict(
            structured_outputs={"choice": sample_structured_outputs_choices}
        ),
    )

    assert chat_completion.choices[0].logprobs is not None
    assert chat_completion.choices[0].logprobs.content is not None
    top_logprobs = chat_completion.choices[0].logprobs.content[0].top_logprobs

    # -9999.0 is the minimum logprob returned by OpenAI
    for item in top_logprobs:
        assert item.logprob >= -9999.0, f"Failed (top_logprobs={top_logprobs})"
```
**EN:** This async test validates `test_structured_outputs_choice_chat_logprobs`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `sample_structured_outputs_choices`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_completion.choices[0].logprobs is not None` and `chat_completion.choices[0].logprobs.content is not None`.
**CN:** 这个异步测试验证 `test_structured_outputs_choice_chat_logprobs`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`sample_structured_outputs_choices`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_completion.choices[0].logprobs is not None` and `chat_completion.choices[0].logprobs.content is not None`。

### Test / 测试: test_response_format_json_object (L615-L636)
```python
@pytest.mark.asyncio
async def test_response_format_json_object(client: openai.AsyncOpenAI):
    for _ in range(2):
        resp = await client.chat.completions.create(
            model=MODEL_NAME,
            messages=[
                {
                    "role": "user",
                    "content": (
                        "what is 1+1? please respond with a JSON object, "
                        'the format is {"result": 2}'
                    ),
                }
            ],
            response_format={"type": "json_object"},
        )

        content = resp.choices[0].message.content
        assert content is not None

        loaded = json.loads(content)
        assert loaded == {"result": 2}, loaded
```
**EN:** This async test validates `test_response_format_json_object`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `content is not None` and `loaded == {'result': 2}`.
**CN:** 这个异步测试验证 `test_response_format_json_object`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content is not None` and `loaded == {'result': 2}`。

### Test / 测试: test_response_format_json_schema (L639-L676)
```python
@pytest.mark.asyncio
async def test_response_format_json_schema(client: openai.AsyncOpenAI):
    prompt = 'what is 1+1? The format is "result": 2'
    # Check that this prompt cannot lead to a valid JSON without json_schema
    for _ in range(2):
        resp = await client.chat.completions.create(
            model=MODEL_NAME,
            messages=[{"role": "user", "content": prompt}],
        )
        content = resp.choices[0].message.content
        assert content is not None
        with pytest.raises((json.JSONDecodeError, AssertionError)):
            loaded = json.loads(content)
            assert loaded == {"result": 2}, loaded

    for _ in range(2):
        resp = await client.chat.completions.create(
            model=MODEL_NAME,
# ... 12 lines omitted for brevity ...
            },
        )

        content = resp.choices[0].message.content
        assert content is not None

        loaded = json.loads(content)
        assert loaded == {"result": 2}, loaded
```
**EN:** This async test validates `test_response_format_json_schema`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `content is not None` and `content is not None`.
**CN:** 这个异步测试验证 `test_response_format_json_schema`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content is not None` and `content is not None`。

### Test / 测试: test_response_format_text (L679-L695)
```python
@pytest.mark.asyncio
async def test_response_format_text(client: openai.AsyncOpenAI):
    for _ in range(2):
        resp = await client.chat.completions.create(
            model=MODEL_NAME,
            messages=[
                {
                    "role": "user",
                    "content": "what is 1+1?",
                }
            ],
            max_completion_tokens=10,
            response_format={"type": "text"},
        )

        content = resp.choices[0].message.content
        assert content is not None
```
**EN:** This async test validates `test_response_format_text`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `content is not None`.
**CN:** 这个异步测试验证 `test_response_format_text`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content is not None`。

### Test / 测试: test_extra_fields_allowed (L698-L714)
```python
@pytest.mark.asyncio
async def test_extra_fields_allowed(client: openai.AsyncOpenAI):
    resp = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=[
            {
                "role": "user",
                "content": "what is 1+1?",
                "extra_field": "0",
            }
        ],  # type: ignore
        temperature=0,
        seed=0,
    )

    content = resp.choices[0].message.content
    assert content is not None
```
**EN:** This async test validates `test_extra_fields_allowed`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `content is not None`.
**CN:** 这个异步测试验证 `test_extra_fields_allowed`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content is not None`。

### Test / 测试: test_complex_message_content (L717-L737)
```python
@pytest.mark.asyncio
async def test_complex_message_content(client: openai.AsyncOpenAI):
    content = [
        {
            "type": "text",
            "text": "what is 1+1? please provide the result without any other text.",
        }
    ]
    resp = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=[
            {
                "role": "user",
                "content": content,
            }
        ],
        temperature=0,
        seed=0,
    )
    content = resp.choices[0].message.content
    assert content == "2"
```
**EN:** This async test validates `test_complex_message_content`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `content == '2'`.
**CN:** 这个异步测试验证 `test_complex_message_content`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content == '2'`。

### Test / 测试: test_custom_role (L740-L771)
```python
@pytest.mark.asyncio
async def test_custom_role(client: openai.AsyncOpenAI):
    # Not sure how the model handles custom roles so we just check that
    # both string and complex message content are handled in the same way

    resp1 = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=[
            {
                "role": "my-custom-role",
                "content": "what is 1+1?",
            }
        ],  # type: ignore
        temperature=0,
        seed=0,
    )

    resp2 = await client.chat.completions.create(
# ... 6 lines omitted for brevity ...
        ],  # type: ignore
        temperature=0,
        seed=0,
    )

    content1 = resp1.choices[0].message.content
    content2 = resp2.choices[0].message.content
    assert content1 == content2
```
**EN:** This async test validates `test_custom_role`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `content1 == content2`.
**CN:** 这个异步测试验证 `test_custom_role`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `content1 == content2`。

### Test / 测试: test_long_seed (L774-L793)
```python
@pytest.mark.asyncio
async def test_long_seed(client: openai.AsyncOpenAI):
    for seed in [torch.iinfo(torch.long).min - 1, torch.iinfo(torch.long).max + 1]:
        with pytest.raises(BadRequestError) as exc_info:
            await client.chat.completions.create(
                model=MODEL_NAME,
                messages=[
                    {
                        "role": "system",
                        "content": "You are a helpful assistant.",
                    }
                ],
                temperature=0,
                seed=seed,
            )

        assert (
            "greater_than_equal" in exc_info.value.message
            or "less_than_equal" in exc_info.value.message
        )
```
**EN:** This async test validates `test_long_seed`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `'greater_than_equal' in exc_info.value.message or 'less_than_equal' in exc_info.value.message`.
**CN:** 这个异步测试验证 `test_long_seed`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'greater_than_equal' in exc_info.value.message or 'less_than_equal' in exc_info.value.message`。

### Test / 测试: test_invocations (L796-L822)
```python
@pytest.mark.asyncio
async def test_invocations(server: RemoteOpenAIServer, client: openai.AsyncOpenAI):
    messages = [
        {"role": "system", "content": "you are a helpful assistant"},
        {"role": "user", "content": "what is 1+1?"},
    ]

    request_args = {
        "model": MODEL_NAME,
        "messages": messages,
        "max_completion_tokens": 5,
        "temperature": 0.0,
        "logprobs": False,
    }

    chat_completion = await client.chat.completions.create(**request_args)

    invocation_response = requests.post(
        server.url_for("invocations"), json=request_args
    )
    invocation_response.raise_for_status()

    chat_output = chat_completion.model_dump()
    invocation_output = invocation_response.json()

    assert chat_output.keys() == invocation_output.keys()
    assert chat_output["choices"] == invocation_output["choices"]
```
**EN:** This async test validates `test_invocations`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `client`. It drives client-facing request creation through the API surface under test. The main assertion is `chat_output.keys() == invocation_output.keys()` and `chat_output['choices'] == invocation_output['choices']`.
**CN:** 这个异步测试验证 `test_invocations`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat_output.keys() == invocation_output.keys()` and `chat_output['choices'] == invocation_output['choices']`。

### Test / 测试: test_chat_completion_n_parameter_non_streaming (L830-L864)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_chat_completion_n_parameter_non_streaming(
    client: openai.AsyncOpenAI, model_name: str
):
    """Test that n parameter returns multiple choices for non-streaming requests."""
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the opposite of big?"},
    ]

    # Test with n=3
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
# ... 9 lines omitted for brevity ...
    # Verify each choice has content and correct index
    for i, choice in enumerate(chat_completion.choices):
        assert choice.index == i
        assert choice.message.content is not None
        assert len(choice.message.content) > 0

    contents = [choice.message.content for choice in chat_completion.choices]
    assert len(set(contents)) > 1, "Expected different responses with n=3"
```
**EN:** This async test validates `test_chat_completion_n_parameter_non_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 3` and `len(set(contents)) > 1`.
**CN:** 这个异步测试验证 `test_chat_completion_n_parameter_non_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 3` and `len(set(contents)) > 1`。

### Test / 测试: test_chat_completion_n_parameter_streaming (L867-L906)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_chat_completion_n_parameter_streaming(
    client: openai.AsyncOpenAI, model_name: str
):
    """Test that n parameter returns multiple choices for streaming requests."""
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"},
    ]

    stream = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=15,
# ... 14 lines omitted for brevity ...
    assert len(chunks_by_index[1]) > 0, "Choice 1 received no content chunks"

    # Reconstruct full responses
    response_0 = "".join(chunks_by_index[0])
    response_1 = "".join(chunks_by_index[1])

    assert len(response_0) > 0, "Choice 0 has empty response"
    assert len(response_1) > 0, "Choice 1 has empty response"
```
**EN:** This async test validates `test_chat_completion_n_parameter_streaming`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chunks_by_index[0]) > 0` and `len(chunks_by_index[1]) > 0`.
**CN:** 这个异步测试验证 `test_chat_completion_n_parameter_streaming`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chunks_by_index[0]) > 0` and `len(chunks_by_index[1]) > 0`。

### Test / 测试: test_chat_completion_n_with_seed (L909-L938)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_chat_completion_n_with_seed(client: openai.AsyncOpenAI, model_name: str):
    """Test that n parameter works correctly with seed parameter."""
    messages = [
        {"role": "user", "content": "Say hello."},
    ]

    # Test that seed parameter is accepted and works with n > 1
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.8,
        n=2,
        seed=42,
        stream=False,
    )

    # Verify we get n=2 choices
    assert len(chat_completion.choices) == 2

    # Verify both choices have valid content
    for i, choice in enumerate(chat_completion.choices):
        assert choice.index == i
        assert choice.message.content is not None
        assert len(choice.message.content) > 0
```
**EN:** This async test validates `test_chat_completion_n_with_seed`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 2` and `choice.index == i`.
**CN:** 这个异步测试验证 `test_chat_completion_n_with_seed`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 2` and `choice.index == i`。

### Test / 测试: test_chat_completion_n_equals_1 (L941-L963)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name",
    [MODEL_NAME],
)
async def test_chat_completion_n_equals_1(client: openai.AsyncOpenAI, model_name: str):
    """Test that n=1 (default) still works correctly."""
    messages = [
        {"role": "user", "content": "Hello!"},
    ]

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.7,
        n=1,
        stream=False,
    )

    assert len(chat_completion.choices) == 1
    assert chat_completion.choices[0].index == 0
    assert chat_completion.choices[0].message.content is not None
```
**EN:** This async test validates `test_chat_completion_n_equals_1`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `chat_completion.choices[0].index == 0`.
**CN:** 这个异步测试验证 `test_chat_completion_n_equals_1`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `chat_completion.choices[0].index == 0`。

### Test / 测试: test_chat_completion_request_n_parameter_to_sampling_params (L967-L983)
```python
def test_chat_completion_request_n_parameter_to_sampling_params():
    """Test that n parameter is correctly passed to SamplingParams."""
    # Test with n=3
    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Hello"}],
        n=3,
        max_tokens=10,
    )

    sampling_params = request.to_sampling_params(
        max_tokens=10,
        default_sampling_params={},
    )

    assert isinstance(sampling_params, SamplingParams)
    assert sampling_params.n == 3, f"Expected n=3, got n={sampling_params.n}"
```
**EN:** This test validates `test_chat_completion_request_n_parameter_to_sampling_params`. The main assertion is `isinstance(sampling_params, SamplingParams)` and `sampling_params.n == 3`.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_to_sampling_params`。 核心断言是 `isinstance(sampling_params, SamplingParams)` and `sampling_params.n == 3`。

### Test / 测试: test_chat_completion_request_n_parameter_default (L986-L1002)
```python
def test_chat_completion_request_n_parameter_default():
    """Test that n parameter defaults to 1."""
    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Hello"}],
        # n not specified, should default to 1
        max_tokens=10,
    )

    assert request.n == 1, "n should default to 1"
    sampling_params = request.to_sampling_params(
        max_tokens=10,
        default_sampling_params={},
    )

    # SamplingParams.from_optional converts None to 1
    assert sampling_params.n == 1, f"Expected n=1 (default), got n={sampling_params.n}"
```
**EN:** This test validates `test_chat_completion_request_n_parameter_default`. The main assertion is `request.n == 1` and `sampling_params.n == 1`.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_default`。 核心断言是 `request.n == 1` and `sampling_params.n == 1`。

### Test / 测试: test_chat_completion_request_accepts_model_specific_reasoning_effort (L1005-L1018)
```python
def test_chat_completion_request_accepts_model_specific_reasoning_effort():
    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Hello"}],
        reasoning_effort="max",
    )

    chat_params = request.build_chat_params(
        default_template=None,
        default_template_content_format="auto",
    )

    assert request.reasoning_effort == "max"
    assert chat_params.chat_template_kwargs["reasoning_effort"] == "max"
```
**EN:** This test validates `test_chat_completion_request_accepts_model_specific_reasoning_effort`. The main assertion is `request.reasoning_effort == 'max'` and `chat_params.chat_template_kwargs['reasoning_effort'] == 'max'`.
**CN:** 这个测试验证 `test_chat_completion_request_accepts_model_specific_reasoning_effort`。 核心断言是 `request.reasoning_effort == 'max'` and `chat_params.chat_template_kwargs['reasoning_effort'] == 'max'`。

### Test / 测试: test_chat_completion_request_rejects_unknown_reasoning_effort (L1021-L1027)
```python
def test_chat_completion_request_rejects_unknown_reasoning_effort():
    with pytest.raises(ValueError, match="Input should be"):
        ChatCompletionRequest(
            model="test-model",
            messages=[{"role": "user", "content": "Hello"}],
            reasoning_effort="extra_high",
        )
```
**EN:** This test validates `test_chat_completion_request_rejects_unknown_reasoning_effort`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_chat_completion_request_rejects_unknown_reasoning_effort`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_chat_completion_request_n_parameter_various_values (L1030-L1047)
```python
def test_chat_completion_request_n_parameter_various_values():
    """Test n parameter with various values."""
    for n_value in [1, 2, 5, 10]:
        request = ChatCompletionRequest(
            model="test-model",
            messages=[{"role": "user", "content": "Test"}],
            n=n_value,
            max_tokens=10,
        )

        sampling_params = request.to_sampling_params(
            max_tokens=10,
            default_sampling_params={},
        )

        assert sampling_params.n == n_value, (
            f"Expected n={n_value}, got n={sampling_params.n}"
        )
```
**EN:** This test validates `test_chat_completion_request_n_parameter_various_values`. The main assertion is `sampling_params.n == n_value`.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_various_values`。 核心断言是 `sampling_params.n == n_value`。

### Test / 测试: test_chat_completion_request_n_parameter_exceeds_default_limit (L1050-L1072)
```python
def test_chat_completion_request_n_parameter_exceeds_default_limit(
    monkeypatch: pytest.MonkeyPatch,
):
    """Test that n values exceeding the default limit are rejected."""
    import vllm.envs as envs

    monkeypatch.delenv("VLLM_MAX_N_SEQUENCES", raising=False)
    if hasattr(envs.__getattr__, "cache_clear"):
        envs.__getattr__.cache_clear()

    max_n = envs.VLLM_MAX_N_SEQUENCES
    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Test"}],
        n=max_n + 1,
        max_tokens=10,
    )

    with pytest.raises(ValueError, match="n must be at most"):
        request.to_sampling_params(
            max_tokens=10,
            default_sampling_params={},
        )
```
**EN:** This test validates `test_chat_completion_request_n_parameter_exceeds_default_limit`. Key inputs are `monkeypatch`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_exceeds_default_limit`。 关键输入包括 `monkeypatch`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_chat_completion_request_n_parameter_at_limit (L1075-L1097)
```python
def test_chat_completion_request_n_parameter_at_limit(
    monkeypatch: pytest.MonkeyPatch,
):
    """Test that n at exactly the limit is accepted."""
    import vllm.envs as envs

    monkeypatch.delenv("VLLM_MAX_N_SEQUENCES", raising=False)
    if hasattr(envs.__getattr__, "cache_clear"):
        envs.__getattr__.cache_clear()

    max_n = envs.VLLM_MAX_N_SEQUENCES
    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Test"}],
        n=max_n,
        max_tokens=10,
    )

    sampling_params = request.to_sampling_params(
        max_tokens=10,
        default_sampling_params={},
    )
    assert sampling_params.n == max_n
```
**EN:** This test validates `test_chat_completion_request_n_parameter_at_limit`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `sampling_params.n == max_n`.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_at_limit`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `sampling_params.n == max_n`。

### Test / 测试: test_chat_completion_request_n_parameter_custom_limit (L1100-L1134)
```python
def test_chat_completion_request_n_parameter_custom_limit(
    monkeypatch: pytest.MonkeyPatch,
):
    """Test that VLLM_MAX_N_SEQUENCES env var overrides the default limit."""
    import vllm.envs as envs

    monkeypatch.setenv("VLLM_MAX_N_SEQUENCES", "128")
    if hasattr(envs.__getattr__, "cache_clear"):
        envs.__getattr__.cache_clear()

    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Test"}],
        n=128,
        max_tokens=10,
    )

    sampling_params = request.to_sampling_params(
# ... 9 lines omitted for brevity ...
        max_tokens=10,
    )

    with pytest.raises(ValueError, match="n must be at most 128"):
        request_over.to_sampling_params(
            max_tokens=10,
            default_sampling_params={},
        )
```
**EN:** This test validates `test_chat_completion_request_n_parameter_custom_limit`. Key inputs are `monkeypatch`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `sampling_params.n == 128`.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_custom_limit`。 关键输入包括 `monkeypatch`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `sampling_params.n == 128`。

### Test / 测试: test_chat_completion_request_n_parameter_massive_value (L1137-L1158)
```python
def test_chat_completion_request_n_parameter_massive_value(
    monkeypatch: pytest.MonkeyPatch,
):
    """Test that astronomically large n values are rejected (CVE fix)."""
    import vllm.envs as envs

    monkeypatch.delenv("VLLM_MAX_N_SEQUENCES", raising=False)
    if hasattr(envs.__getattr__, "cache_clear"):
        envs.__getattr__.cache_clear()

    request = ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "Test"}],
        n=100_000_000,
        max_tokens=1,
    )

    with pytest.raises(ValueError, match="n must be at most"):
        request.to_sampling_params(
            max_tokens=1,
            default_sampling_params={},
        )
```
**EN:** This test validates `test_chat_completion_request_n_parameter_massive_value`. Key inputs are `monkeypatch`. It checks an expected failure path with `pytest.raises`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_chat_completion_request_n_parameter_massive_value`。 关键输入包括 `monkeypatch`。 它使用 `pytest.raises` 检查预期失败路径。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

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
- **Stdlib / 标准库**: `collections.defaultdict`, `json`
- **Third-party / 第三方**: `huggingface_hub.snapshot_download`, `jsonschema`, `openai`, `openai.BadRequestError`, `pytest`, `pytest_asyncio`, `regex`, `requests`, `torch`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionRequest`, `vllm.envs`, `vllm.sampling_params.SamplingParams`
