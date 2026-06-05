# test_chunked_prompt.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/test_chunked_prompt.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 2 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 2 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L8)
```python
import openai  # use the official client for correctness check
import pytest
import pytest_asyncio

from ...utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `...utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `...utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L11-L11)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L14-L31)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        # use half precision for speed and memory savings in CI environment
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "8192",
        "--enforce-eager",
        "--max-num-seqs",
        "128",
        "--enable-chunked-prefill",
        "--max-num-batched-tokens",
        "1000",
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L34-L37)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_completion_stream_options_and_logprobs_with_long_prompts (L40-L79)
```python
@pytest.mark.asyncio
async def test_completion_stream_options_and_logprobs_with_long_prompts(
    client: openai.AsyncOpenAI,
):
    # Test stream with long prompt
    prompt = "What is the capital of France?" * 400

    stream = await client.completions.create(
        model=MODEL_NAME,
        prompt=prompt,
        max_tokens=5,
        temperature=0.0,
        stream=True,
        stream_options={
            "include_usage": True,
            "continuous_usage_stats": True,
        },
        logprobs=5,
# ... 14 lines omitted for brevity ...
            assert chunk.choices[0].logprobs and chunk.choices[0].logprobs.tokens
            tokens_received += len(chunk.choices[0].logprobs.tokens)

            if chunk.choices[0].finish_reason is not None:
                finished = True

        if finished:
            assert chunk.usage.completion_tokens == tokens_received
```
**EN:** This async test validates `test_completion_stream_options_and_logprobs_with_long_prompts`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `chunk.usage.prompt_tokens >= 0` and `chunk.usage.completion_tokens >= 0`.
**CN:** 这个异步测试验证 `test_completion_stream_options_and_logprobs_with_long_prompts`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `chunk.usage.prompt_tokens >= 0` and `chunk.usage.completion_tokens >= 0`。

### Test / 测试: test_chat_completion_stream_options_and_logprobs_with_long_prompts (L82-L133)
```python
@pytest.mark.asyncio
async def test_chat_completion_stream_options_and_logprobs_with_long_prompts(
    client: openai.AsyncOpenAI,
):
    # Test stream with long prompt
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?" * 400},
    ]
    stream = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=messages,
        max_tokens=5,
        temperature=0.0,
        stream=True,
        stream_options={
            "include_usage": True,
            "continuous_usage_stats": True,
# ... 26 lines omitted for brevity ...

            if chunk.choices[0].finish_reason is not None:
                finished = True

        if finished:
            assert chunk.usage.completion_tokens == tokens_received

    assert empty_chunks_received <= 1
```
**EN:** This async test validates `test_chat_completion_stream_options_and_logprobs_with_long_prompts`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `empty_chunks_received <= 1` and `chunk.usage.prompt_tokens >= 0`.
**CN:** 这个异步测试验证 `test_chat_completion_stream_options_and_logprobs_with_long_prompts`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `empty_chunks_received <= 1` and `chunk.usage.prompt_tokens >= 0`。

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
- **Local relative imports / 本地相对导入**: `...utils.RemoteOpenAIServer`
