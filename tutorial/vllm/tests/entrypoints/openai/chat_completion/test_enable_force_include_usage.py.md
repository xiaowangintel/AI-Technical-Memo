# test_enable_force_include_usage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_enable_force_include_usage.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 1 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 1 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L7)
```python
import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Fixture / 夹具: chat_server_with_force_include_usage (L10-L29)
```python
@pytest.fixture(scope="module")
def chat_server_with_force_include_usage(request):
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "128",
        "--enforce-eager",
        "--max-num-seqs",
        "4",
        "--enable-force-include-usage",
        "--port",
        "55857",
        "--gpu-memory-utilization",
        "0.2",
    ]

    with RemoteOpenAIServer("Qwen/Qwen3-0.6B", args, auto_port=False) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `chat_server_with_force_include_usage` for dependent tests. Key inputs are `request`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `chat_server_with_force_include_usage`。 关键输入包括 `request`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: chat_client_with_force_include_usage (L32-L35)
```python
@pytest_asyncio.fixture
async def chat_client_with_force_include_usage(chat_server_with_force_include_usage):
    async with chat_server_with_force_include_usage.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `chat_client_with_force_include_usage` for dependent tests. Key inputs are `chat_server_with_force_include_usage`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `chat_client_with_force_include_usage`。 关键输入包括 `chat_server_with_force_include_usage`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_chat_with_enable_force_include_usage (L38-L69)
```python
@pytest.mark.asyncio
async def test_chat_with_enable_force_include_usage(
    chat_client_with_force_include_usage: openai.AsyncOpenAI,
):
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"},
    ]

    stream = await chat_client_with_force_include_usage.chat.completions.create(
        model="Qwen/Qwen3-0.6B",
        messages=messages,
        max_completion_tokens=10,
        extra_body=dict(min_tokens=10),
        temperature=0.0,
        stream=True,
    )
    last_completion_tokens = 0
# ... 6 lines omitted for brevity ...
                not chunk.choices
                and chunk.usage.completion_tokens == last_completion_tokens
            )
        )
        assert chunk.usage.total_tokens == (
            chunk.usage.prompt_tokens + chunk.usage.completion_tokens
        )
        last_completion_tokens = chunk.usage.completion_tokens
```
**EN:** This async test validates `test_chat_with_enable_force_include_usage`. Relevant pytest markers include `asyncio`. Key inputs are `chat_client_with_force_include_usage`. It drives client-facing request creation through the API surface under test. The main assertion is `chunk.usage.prompt_tokens >= 0` and `last_completion_tokens == 0 or chunk.usage.completion_tokens > last_completion_tokens or (not chunk.choices and chunk.usage.completion_to...`.
**CN:** 这个异步测试验证 `test_chat_with_enable_force_include_usage`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `chat_client_with_force_include_usage`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chunk.usage.prompt_tokens >= 0` and `last_completion_tokens == 0 or chunk.usage.completion_tokens > last_completion_tokens or (not chunk.choices and chunk.usage.completion_to...`。

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
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
