# test_truncation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/basic/test_truncation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 3 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 3 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L10)
```python
from typing import Any

import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: MODEL_NAME, max_model_len, input (L12-L27)
```python
MODEL_NAME = "sentence-transformers/all-MiniLM-L12-v2"
max_model_len = 128

input = """Immerse yourself in the enchanting chronicle of calculus, a 
    mathematical domain that has radically transformed our comprehension of 
    change and motion. Despite its roots in ancient civilizations, the 
    formal birth of calculus predominantly occurred in the 17th century, 
    primarily under the influential guidance of Sir Isaac Newton and Gottfried 
    Wilhelm Leibniz. The earliest traces of calculus concepts are found in 
    ancient Greek mathematics,most notably in the works of Eudoxus and 
    Archimedes, around 300 BCE. They utilized the 'method of exhaustion'—a 
    technique for computing areas and volumes through the use of finite sums. 
    This methodology laid crucial foundational work for integral calculus. 
    In the 17th century, both Newton and Leibniz independently pioneered 
    calculus, each contributing unique perspectives that would shape this new 
    field."""
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `max_model_len`, `input`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`max_model_len`、`input`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L30-L47)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--runner",
        "pooling",
        "--dtype",
        "bfloat16",
        "--enforce-eager",
        "--max-model-len",
        str(max_model_len),
    ]

    # ROCm: Use Flex Attention to support encoder-only self-attention.
    if current_platform.is_rocm():
        args.extend(["--attention-backend", "FLEX_ATTENTION"])

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L50-L53)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_smaller_truncation_size (L56-L67)
```python
@pytest.mark.asyncio
async def test_smaller_truncation_size(client: openai.AsyncOpenAI):
    truncation_size = 10
    kwargs: dict[str, Any] = {
        "model": MODEL_NAME,
        "input": input,
        "truncate_prompt_tokens": truncation_size,
    }

    response = await client.post(path="embeddings", cast_to=object, body={**kwargs})

    assert response["usage"]["prompt_tokens"] == truncation_size
```
**EN:** This async test validates `test_smaller_truncation_size`. Relevant pytest markers include `asyncio`. Key inputs are `client`. The main assertion is `response['usage']['prompt_tokens'] == truncation_size`.
**CN:** 这个异步测试验证 `test_smaller_truncation_size`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 核心断言是 `response['usage']['prompt_tokens'] == truncation_size`。

### Test / 测试: test_bigger_truncation_size (L70-L90)
```python
@pytest.mark.asyncio
async def test_bigger_truncation_size(client: openai.AsyncOpenAI):
    truncation_size = max_model_len + 1
    kwargs: dict[str, Any] = {
        "model": MODEL_NAME,
        "input": input,
        "truncate_prompt_tokens": truncation_size,
    }

    with pytest.raises(openai.BadRequestError) as err:
        await client.post(path="embeddings", cast_to=object, body={**kwargs})

    assert err.value.status_code == 400
    error_details = err.value.response.json()["error"]
    assert error_details["type"] == "BadRequestError"
    expected_message = (
        "truncate_prompt_tokens value is "
        "greater than max_model_len."
        " Please request a smaller truncation size."
    )
    assert error_details["message"] == expected_message
```
**EN:** This async test validates `test_bigger_truncation_size`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It checks an expected failure path with `pytest.raises`. The main assertion is `err.value.status_code == 400` and `error_details['type'] == 'BadRequestError'`.
**CN:** 这个异步测试验证 `test_bigger_truncation_size`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `err.value.status_code == 400` and `error_details['type'] == 'BadRequestError'`。

### Test / 测试: test_max_truncation_size (L93-L104)
```python
@pytest.mark.asyncio
async def test_max_truncation_size(client: openai.AsyncOpenAI):
    truncation_size = -1
    kwargs: dict[str, Any] = {
        "model": MODEL_NAME,
        "input": input,
        "truncate_prompt_tokens": truncation_size,
    }

    response = await client.post(path="embeddings", cast_to=object, body={**kwargs})

    assert response["usage"]["prompt_tokens"] == max_model_len
```
**EN:** This async test validates `test_max_truncation_size`. Relevant pytest markers include `asyncio`. Key inputs are `client`. The main assertion is `response['usage']['prompt_tokens'] == max_model_len`.
**CN:** 这个异步测试验证 `test_max_truncation_size`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 核心断言是 `response['usage']['prompt_tokens'] == max_model_len`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`
