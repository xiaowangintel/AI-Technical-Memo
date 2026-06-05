# test_stateful.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_stateful.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 7 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 7 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L6)
```python
import asyncio

import openai
import pytest
```
**EN:** Imports standard-library modules such as `asyncio`, third-party packages like `openai`, `pytest`.
**CN:** 导入标准库模块（如 `asyncio`）、第三方包（如 `openai`、`pytest`）。

### Test / 测试: test_store (L9-L28)
```python
@pytest.mark.asyncio
async def test_store(client: openai.AsyncOpenAI):
    # By default, store is True.
    response = await client.responses.create(input="Hello!")
    assert response.status == "completed"

    # Retrieve the response.
    response = await client.responses.retrieve(response.id)
    assert response.status == "completed"

    # Test store=False.
    response = await client.responses.create(
        input="Hello!",
        store=False,
    )
    assert response.status == "completed"

    # The response should not be found.
    with pytest.raises(openai.NotFoundError, match="Response with id .* not found."):
        await client.responses.retrieve(response.id)
```
**EN:** This async test validates `test_store`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_store`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'` and `response.status == 'completed'`。

### Test / 测试: test_background (L31-L49)
```python
@pytest.mark.asyncio
async def test_background(client: openai.AsyncOpenAI):
    # NOTE: This query should be easy enough for the model to answer
    # within the 10 seconds.
    response = await client.responses.create(
        input="Hello!",
        background=True,
    )
    assert response.status == "queued"

    max_retries = 10
    for _ in range(max_retries):
        await asyncio.sleep(1)
        response = await client.responses.retrieve(response.id)
        if response.status != "queued":
            break
    print(response)

    assert response.status == "completed"
```
**EN:** This async test validates `test_background`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'queued'` and `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_background`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'queued'` and `response.status == 'completed'`。

### Test / 测试: test_background_error (L52-L61)
```python
@pytest.mark.asyncio
async def test_background_error(client: openai.AsyncOpenAI):
    with pytest.raises(
        openai.BadRequestError, match="background can only be used when `store` is true"
    ):
        _ = await client.responses.create(
            input="What is 13 * 24?",
            background=True,
            store=False,
        )
```
**EN:** This async test validates `test_background_error`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_background_error`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_background_cancel (L64-L94)
```python
@pytest.mark.asyncio
async def test_background_cancel(client: openai.AsyncOpenAI):
    response = await client.responses.create(
        input="Write a long story about a cat.",
        background=True,
    )
    assert response.status == "queued"

    # Cancel the response before it is completed.
    # Poll until the response is no longer queued (started processing) or timeout
    loop = asyncio.get_running_loop()
    start_time = loop.time()
    max_wait_seconds = 5.0
    poll_interval = 0.1
    while loop.time() - start_time < max_wait_seconds:
        response = await client.responses.retrieve(response.id)
        if response.status != "queued":
            # Started processing or completed - try to cancel
# ... 5 lines omitted for brevity ...

    # Make sure the response status remains unchanged after some time.
    max_retries = 10
    for _ in range(max_retries):
        await asyncio.sleep(0.5)
        response = await client.responses.retrieve(response.id)
        # Verify status is still cancelled
        assert response.status == "cancelled"
```
**EN:** This async test validates `test_background_cancel`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'queued'` and `response.status == 'cancelled'`.
**CN:** 这个异步测试验证 `test_background_cancel`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'queued'` and `response.status == 'cancelled'`。

### Test / 测试: test_cancel_completed (L97-L105)
```python
@pytest.mark.asyncio
async def test_cancel_completed(client: openai.AsyncOpenAI):
    response = await client.responses.create(input="Hello")
    assert response.status == "completed"

    with pytest.raises(
        openai.BadRequestError, match="Cannot cancel a synchronous response."
    ):
        await client.responses.cancel(response.id)
```
**EN:** This async test validates `test_cancel_completed`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `response.status == 'completed'`.
**CN:** 这个异步测试验证 `test_cancel_completed`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `response.status == 'completed'`。

### Test / 测试: test_previous_response_id (L108-L127)
```python
@pytest.mark.asyncio
async def test_previous_response_id(client: openai.AsyncOpenAI):
    response1 = await client.responses.create(
        instructions="You are tested on your ability to retrieve the correct "
        "information from the previous response.",
        input="Hello, my name is John.",
    )

    response2 = await client.responses.create(
        input="Actually, my name is not John. My real name is Mark.",
        previous_response_id=response1.id,
    )

    response3 = await client.responses.create(
        input="What is my real name again? Answer in one word.",
        previous_response_id=response2.id,
    )
    print(response3)
    assert "Mark" in response3.output[-1].content[0].text
    assert "John" not in response3.output[-1].content[0].text
```
**EN:** This async test validates `test_previous_response_id`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `'Mark' in response3.output[-1].content[0].text` and `'John' not in response3.output[-1].content[0].text`.
**CN:** 这个异步测试验证 `test_previous_response_id`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'Mark' in response3.output[-1].content[0].text` and `'John' not in response3.output[-1].content[0].text`。

### Test / 测试: test_two_responses_with_same_prev_id (L130-L152)
```python
@pytest.mark.asyncio
async def test_two_responses_with_same_prev_id(client: openai.AsyncOpenAI):
    response1 = await client.responses.create(
        instructions="You are tested on your ability to retrieve the correct "
        "information from the previous response.",
        input="Hello, my name is John.",
    )

    # Both response 2 and 3 use response 1 as the previous response.
    response2 = client.responses.create(
        input="Actually, my name is not John. My name is Mark.",
        previous_response_id=response1.id,
    )
    response3 = client.responses.create(
        input="What is my name again? Answer in one word.",
        previous_response_id=response1.id,
    )

    _ = await response2
    response3_result = await response3
    print(response3_result)
    assert "John" in response3_result.output[-1].content[0].text
    assert "Mark" not in response3_result.output[-1].content[0].text
```
**EN:** This async test validates `test_two_responses_with_same_prev_id`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `'John' in response3_result.output[-1].content[0].text` and `'Mark' not in response3_result.output[-1].content[0].text`.
**CN:** 这个异步测试验证 `test_two_responses_with_same_prev_id`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'John' in response3_result.output[-1].content[0].text` and `'Mark' not in response3_result.output[-1].content[0].text`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`
- **Third-party / 第三方**: `openai`, `pytest`
