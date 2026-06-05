# test_chat_completion_with_prompt_embeds.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_chat_completion_with_prompt_embeds.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 7 test(s), 5 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 7 个测试、5 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L6-L16)
```python
import asyncio
import io

import openai
import pybase64 as base64
import pytest
import pytest_asyncio
import torch
from openai import BadRequestError

from tests.utils import VLLM_PATH, RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `asyncio`, `io`, third-party packages like `openai`, `openai.BadRequestError`, `pybase64`, project helpers such as `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`.
**CN:** 导入标准库模块（如 `asyncio`、`io`）、第三方包（如 `openai`、`openai.BadRequestError`、`pybase64`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`tests.utils.VLLM_PATH`）。

### Module setup / 模块级配置: MODEL_NAME, CHAT_TEMPLATE, SERVER_DTYPE (L18-L23)
```python
MODEL_NAME = "facebook/opt-125m"
CHAT_TEMPLATE = VLLM_PATH / "examples/template_chatml.jinja"
# Matches `--dtype` in `server_args` to avoid an implicit cast in
# `safe_load_prompt_embeds` (mismatched floating-point dtypes are cast to the
# model's dtype automatically, we match here just to skip the conversion).
SERVER_DTYPE: torch.dtype = torch.bfloat16
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `CHAT_TEMPLATE`, `SERVER_DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`CHAT_TEMPLATE`、`SERVER_DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server_args (L26-L40)
```python
@pytest.fixture(scope="module")
def server_args() -> list[str]:
    return [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "128",
        "--enforce-eager",
        "--chat-template",
        str(CHAT_TEMPLATE),
        # Prompt Embeds server args
        "--enable-prompt-embeds",
    ]
```
**EN:** This fixture prepares `server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `server_args`。

### Fixture / 夹具: server (L43-L46)
```python
@pytest.fixture(scope="module")
def server(server_args):
    with RemoteOpenAIServer(MODEL_NAME, server_args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. Key inputs are `server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 关键输入包括 `server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L49-L52)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Helper / 辅助函数: _encode_embeds (L55-L58)
```python
def _encode_embeds(embeds: torch.Tensor) -> str:
    buf = io.BytesIO()
    torch.save(embeds, buf)
    return base64.b64encode(buf.getvalue()).decode("utf-8")
```
**EN:** This helper encapsulates reusable logic in `_encode_embeds`. Key inputs are `embeds`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_encode_embeds` 中。 关键输入包括 `embeds`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: prompt_embeds_b64 (L61-L70)
```python
@pytest.fixture(scope="module")
def prompt_embeds_b64(hf_runner) -> list[str]:
    """Pre-compute embeddings for two short prompts and return as base64."""
    prompts = ["Hello, my name is", "What is an LLM?"]
    with hf_runner(MODEL_NAME) as hf_model:
        embeddings = hf_model.get_prompt_embeddings(prompts)
    # Cast to the server's dtype so `safe_load_prompt_embeds` doesn't need to
    # convert on its own, the function accepts any floating-point dtype and
    # will cast to the model's dtype, but matching up front skips the work.
    return [_encode_embeds(e.to(SERVER_DTYPE)) for e in embeddings]
```
**EN:** This fixture prepares `prompt_embeds_b64` for dependent tests. Key inputs are `hf_runner`.
**CN:** 这个 fixture 为依赖它的测试准备 `prompt_embeds_b64`。 关键输入包括 `hf_runner`。

### Test / 测试: test_single_prompt_embeds_part (L73-L95)
```python
@pytest.mark.asyncio
async def test_single_prompt_embeds_part(
    client: openai.AsyncOpenAI,
    prompt_embeds_b64: list[str],
):
    """A user message with one prompt_embeds part + text."""
    b64 = prompt_embeds_b64[0]
    chat = await client.chat.completions.create(
        model=MODEL_NAME,
        max_tokens=5,
        temperature=0.0,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "prompt_embeds", "data": b64},
                    {"type": "text", "text": "Continue:"},
                ],
            }
        ],
    )
    assert chat.choices[0].message.content is not None
    assert len(chat.choices[0].message.content) > 0
```
**EN:** This async test validates `test_single_prompt_embeds_part`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `prompt_embeds_b64`. It drives client-facing request creation through the API surface under test. The main assertion is `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`.
**CN:** 这个异步测试验证 `test_single_prompt_embeds_part`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`prompt_embeds_b64`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`。

### Test / 测试: test_multiple_prompt_embeds_parts (L98-L121)
```python
@pytest.mark.asyncio
async def test_multiple_prompt_embeds_parts(
    client: openai.AsyncOpenAI,
    prompt_embeds_b64: list[str],
):
    """Multiple prompt_embeds parts in a single message."""
    b64_a, b64_b = prompt_embeds_b64
    chat = await client.chat.completions.create(
        model=MODEL_NAME,
        max_tokens=5,
        temperature=0.0,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "prompt_embeds", "data": b64_a},
                    {"type": "text", "text": " and "},
                    {"type": "prompt_embeds", "data": b64_b},
                ],
            }
        ],
    )
    assert chat.choices[0].message.content is not None
    assert len(chat.choices[0].message.content) > 0
```
**EN:** This async test validates `test_multiple_prompt_embeds_parts`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `prompt_embeds_b64`. It drives client-facing request creation through the API surface under test. The main assertion is `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`.
**CN:** 这个异步测试验证 `test_multiple_prompt_embeds_parts`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`prompt_embeds_b64`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`。

### Test / 测试: test_multi_message_conversation (L124-L153)
```python
@pytest.mark.asyncio
async def test_multi_message_conversation(
    client: openai.AsyncOpenAI,
    prompt_embeds_b64: list[str],
):
    """prompt_embeds in both system and user messages."""
    b64_sys, b64_usr = prompt_embeds_b64
    chat = await client.chat.completions.create(
        model=MODEL_NAME,
        max_tokens=5,
        temperature=0.0,
        messages=[
            {
                "role": "system",
                "content": [
                    {"type": "text", "text": "You are helpful."},
                    {"type": "prompt_embeds", "data": b64_sys},
                ],
            },
            {
                "role": "user",
                "content": [
                    {"type": "prompt_embeds", "data": b64_usr},
                    {"type": "text", "text": "Summarize."},
                ],
            },
        ],
    )
    assert chat.choices[0].message.content is not None
    assert len(chat.choices[0].message.content) > 0
```
**EN:** This async test validates `test_multi_message_conversation`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `prompt_embeds_b64`. It drives client-facing request creation through the API surface under test. The main assertion is `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`.
**CN:** 这个异步测试验证 `test_multi_message_conversation`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`prompt_embeds_b64`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chat.choices[0].message.content is not None` and `len(chat.choices[0].message.content) > 0`。

### Test / 测试: test_streaming (L156-L202)
```python
@pytest.mark.asyncio
async def test_streaming(
    client: openai.AsyncOpenAI,
    prompt_embeds_b64: list[str],
):
    """Streaming chat completion with prompt_embeds."""
    b64 = prompt_embeds_b64[0]

    # Non-streaming baseline.
    baseline = await client.chat.completions.create(
        model=MODEL_NAME,
        max_tokens=5,
        temperature=0.0,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "prompt_embeds", "data": b64},
# ... 21 lines omitted for brevity ...
        ],
    )
    chunks: list[str] = []
    async for chunk in stream:
        delta = chunk.choices[0].delta.content
        if delta:
            chunks.append(delta)
    assert "".join(chunks) == expected
```
**EN:** This async test validates `test_streaming`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `prompt_embeds_b64`. It drives client-facing request creation through the API surface under test. The main assertion is `''.join(chunks) == expected`.
**CN:** 这个异步测试验证 `test_streaming`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`prompt_embeds_b64`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `''.join(chunks) == expected`。

### Fixture / 夹具: aligned_content_and_embeds_b64 (L205-L218)
```python
@pytest.fixture(scope="module")
def aligned_content_and_embeds_b64(hf_runner) -> tuple[str, str]:
    """Return `(content, base64_embeds)` where the embeddings are the model's
    embedding of `content` tokenized WITHOUT special tokens.
    """
    content = "Hello, my name is"
    with hf_runner(MODEL_NAME) as hf_model:
        ids = hf_model.tokenizer(
            content, add_special_tokens=False, return_tensors="pt"
        ).input_ids
        ids = hf_model.wrap_device({"input_ids": ids})["input_ids"]
        embed_layer = hf_model.model.get_input_embeddings()
        embeds = embed_layer(ids).squeeze(0).to(SERVER_DTYPE).cpu()
    return content, _encode_embeds(embeds)
```
**EN:** This fixture prepares `aligned_content_and_embeds_b64` for dependent tests. Key inputs are `hf_runner`.
**CN:** 这个 fixture 为依赖它的测试准备 `aligned_content_and_embeds_b64`。 关键输入包括 `hf_runner`。

### Test / 测试: test_text_content_and_prompt_embeds_match (L221-L255)
```python
@pytest.mark.asyncio
async def test_text_content_and_prompt_embeds_match(
    client: openai.AsyncOpenAI,
    aligned_content_and_embeds_b64: tuple[str, str],
):
    """Equal content in text and `prompt_embeds` should yield identical
    Chat Completions output under greedy decoding.
    """
    content, encoded_embeds = aligned_content_and_embeds_b64

    text_resp, embeds_resp = await asyncio.gather(
        client.chat.completions.create(
            model=MODEL_NAME,
            max_tokens=10,
            temperature=0.0,
            messages=[{"role": "user", "content": content}],
        ),
        client.chat.completions.create(
# ... 9 lines omitted for brevity ...
        ),
    )

    text_out = text_resp.choices[0].message.content
    embeds_out = embeds_resp.choices[0].message.content
    assert text_out is not None and len(text_out) > 0
    assert embeds_out is not None and len(embeds_out) > 0
    assert text_out == embeds_out
```
**EN:** This async test validates `test_text_content_and_prompt_embeds_match`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `aligned_content_and_embeds_b64`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`.
**CN:** 这个异步测试验证 `test_text_content_and_prompt_embeds_match`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`aligned_content_and_embeds_b64`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `text_out is not None and len(text_out) > 0` and `embeds_out is not None and len(embeds_out) > 0`。

### Test / 测试: test_missing_data_field (L258-L273)
```python
@pytest.mark.asyncio
async def test_missing_data_field(
    client: openai.AsyncOpenAI,
):
    """A prompt_embeds part without `data` should return a clear error."""
    with pytest.raises(BadRequestError):
        await client.chat.completions.create(
            model=MODEL_NAME,
            max_tokens=5,
            messages=[
                {
                    "role": "user",
                    "content": [{"type": "prompt_embeds"}],
                }
            ],
        )
```
**EN:** This async test validates `test_missing_data_field`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_missing_data_field`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_invalid_base64 (L276-L293)
```python
@pytest.mark.asyncio
async def test_invalid_base64(
    client: openai.AsyncOpenAI,
):
    """Invalid base64 in the `data` field should return a clear error."""
    with pytest.raises(BadRequestError):
        await client.chat.completions.create(
            model=MODEL_NAME,
            max_tokens=5,
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "prompt_embeds", "data": "not_valid_base64!!"},
                    ],
                }
            ],
        )
```
**EN:** This async test validates `test_invalid_base64`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_invalid_base64`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

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
- **Stdlib / 标准库**: `asyncio`, `io`
- **Third-party / 第三方**: `openai`, `openai.BadRequestError`, `pybase64`, `pytest`, `pytest_asyncio`, `torch`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`
