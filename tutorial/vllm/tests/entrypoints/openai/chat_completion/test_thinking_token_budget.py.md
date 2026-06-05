# test_thinking_token_budget.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_thinking_token_budget.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 4 test(s), 4 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 4 个测试、4 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L9-L19)
```python
import asyncio
import json
from typing import Literal

import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer, multi_gpu_only, requires_fp8
from vllm.platforms import current_platform
from vllm.tokenizers import get_tokenizer
```
**EN:** Imports standard-library modules such as `asyncio`, `json`, `typing.Literal`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `tests.utils.multi_gpu_only`, `tests.utils.requires_fp8`.
**CN:** 导入标准库模块（如 `asyncio`、`json`、`typing.Literal`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`tests.utils.multi_gpu_only`、`tests.utils.requires_fp8`）。

### Module setup / 模块级配置: MODEL_NAME, QWEN35_FP8_MTP_MODEL, MESSAGES (L21-L27)
```python
MODEL_NAME = "Qwen/Qwen3-0.6B"
QWEN35_FP8_MTP_MODEL = "Qwen/Qwen3.5-35B-A3B-FP8"
MESSAGES = [{"role": "user", "content": "What is 1+1? Be concise."}]
THINK_BUDGET = 5

REASONING_START_STR = "<think>"
REASONING_END_STR = "</think>"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `QWEN35_FP8_MTP_MODEL`, `MESSAGES`, `THINK_BUDGET`, `REASONING_START_STR`, `REASONING_END_STR`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`QWEN35_FP8_MTP_MODEL`、`MESSAGES`、`THINK_BUDGET`、`REASONING_START_STR`、`REASONING_END_STR`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _count_reasoning_decode_token_ids_between_markers (L30-L58)
```python
def _count_reasoning_decode_token_ids_between_markers(
    full_token_ids: list[int],
    reasoning_start_ids: list[int],
    reasoning_end_ids: list[int],
) -> int | None:
    """Count decode tokens in the thinking span (after last start, before first end)."""

    if not reasoning_start_ids or not reasoning_end_ids:
        raise ValueError("reasoning marker token id lists must be non-empty")

    def _last_subseq_index(haystack: list[int], needle: list[int]) -> int:
        n = len(needle)
        if n > len(haystack):
            return -1
        for i in range(len(haystack) - n, -1, -1):
            if haystack[i : i + n] == needle:
                return i
        return -1

    last_start = _last_subseq_index(full_token_ids, reasoning_start_ids)
    if last_start < 0:
        return None

    pos_after_start = last_start + len(reasoning_start_ids)
    end_n = len(reasoning_end_ids)
    for j in range(pos_after_start, len(full_token_ids) - end_n + 1):
        if full_token_ids[j : j + end_n] == reasoning_end_ids:
            return j - pos_after_start
    return len(full_token_ids) - pos_after_start
```
**EN:** This helper encapsulates reusable logic in `_count_reasoning_decode_token_ids_between_markers`. Key inputs are `full_token_ids`, `reasoning_start_ids`, `reasoning_end_ids`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_count_reasoning_decode_token_ids_between_markers` 中。 关键输入包括 `full_token_ids`、`reasoning_start_ids`、`reasoning_end_ids`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: server (L61-L76)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--reasoning-parser",
        "qwen3",
        "--reasoning-config",
        '{"reasoning_start_str": "<think>", "reasoning_end_str": "</think>"}',
        "--max-model-len",
        "2048",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.4",
        "--no-async-scheduling",
    ]
    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: server_with_auto_reasoning_config (L79-L92)
```python
@pytest.fixture(scope="module")
def server_with_auto_reasoning_config():
    args = [
        "--reasoning-parser",
        "qwen3",
        "--max-model-len",
        "2048",
        "--enforce-eager",
        "--gpu-memory-utilization",
        "0.4",
        "--no-async-scheduling",
    ]
    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server_with_auto_reasoning_config` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server_with_auto_reasoning_config`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: server_qwen35_fp8_mtp_tp2 (L95-L137)
```python
@pytest.fixture(scope="module")
def server_qwen35_fp8_mtp_tp2():
    """Qwen3.5-35B FP8 with MTP speculative decoding and tensor parallel size 2."""
    if current_platform.device_count() < 2:
        pytest.skip("Need at least 2 GPUs for --tensor-parallel-size 2")
    if not current_platform.supports_fp8():
        pytest.skip("FP8 is not supported on this platform")

    spec_cfg = {
        "method": "mtp",
        "num_speculative_tokens": 2,
        "max_model_len": 32768,
    }
    args = [
        "--tensor-parallel-size",
        "2",
        "--max-model-len",
        "32768",
# ... 17 lines omitted for brevity ...

    with RemoteOpenAIServer(
        QWEN35_FP8_MTP_MODEL,
        args,
        max_wait_seconds=3000,
        env_dict=env_dict,
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server_qwen35_fp8_mtp_tp2` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The logic can skip unsupported environments when prerequisites are not satisfied. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server_qwen35_fp8_mtp_tp2`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L140-L148)
```python
@pytest_asyncio.fixture
async def client(request, server, server_with_auto_reasoning_config):
    server_map = {
        "default": server,
        "auto_config": server_with_auto_reasoning_config,
    }
    target_server = server_map[request.param]
    async with target_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `request`, `server`, `server_with_auto_reasoning_config`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `request`、`server`、`server_with_auto_reasoning_config`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_thinking_token_budget_mixed_requests (L151-L173)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("client", ["default", "auto_config"], indirect=True)
async def test_thinking_token_budget_mixed_requests(client: openai.AsyncOpenAI):
    """Test that mixed requests (some with thinking_token_budget, some without)
    complete successfully without errors."""

    response_with_budget = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES,
        max_tokens=100,
        extra_body={"thinking_token_budget": THINK_BUDGET},
    )
    response_without_budget = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES,
        max_tokens=100,
    )

    msg_with = response_with_budget.choices[0].message
    msg_without = response_without_budget.choices[0].message

    assert msg_with.content or getattr(msg_with, "reasoning", None)
    assert msg_without.content or getattr(msg_without, "reasoning", None)
```
**EN:** This async test validates `test_thinking_token_budget_mixed_requests`. It uses parameterization over `client`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `msg_with.content or getattr(msg_with, 'reasoning', None)` and `msg_without.content or getattr(msg_without, 'reasoning', None)`.
**CN:** 这个异步测试验证 `test_thinking_token_budget_mixed_requests`。 它通过参数化组合 `client`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `msg_with.content or getattr(msg_with, 'reasoning', None)` and `msg_without.content or getattr(msg_without, 'reasoning', None)`。

### Test / 测试: test_thinking_token_budget_limits_reasoning (L176-L203)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("client", ["default", "auto_config"], indirect=True)
async def test_thinking_token_budget_limits_reasoning(client: openai.AsyncOpenAI):
    """Test that thinking_token_budget limits the number of reasoning tokens.

    Counts non-empty streaming ``delta.reasoning`` chunks (coarse proxy; each
    chunk may represent multiple decode tokens — see
    ``_count_reasoning_decode_token_ids_between_markers`` and the Qwen3.5 MTP
    test for id-based checks).
    """

    reasoning_token_count = 0
    stream = await client.chat.completions.create(
        model=MODEL_NAME,
        messages=MESSAGES,
        max_tokens=100,
        stream=True,
        extra_body={"thinking_token_budget": THINK_BUDGET},
    )
    async for chunk in stream:
        delta = chunk.choices[0].delta
        if getattr(delta, "reasoning", None):
            reasoning_token_count += 1

    assert reasoning_token_count == THINK_BUDGET, (
        f"reasoning tokens ({reasoning_token_count}) exceeded "
        f"thinking_token_budget ({THINK_BUDGET})"
    )
```
**EN:** This async test validates `test_thinking_token_budget_limits_reasoning`. It uses parameterization over `client`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `reasoning_token_count == THINK_BUDGET`.
**CN:** 这个异步测试验证 `test_thinking_token_budget_limits_reasoning`。 它通过参数化组合 `client`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `reasoning_token_count == THINK_BUDGET`。

### Test / 测试: test_thinking_token_budget_qwen35_fp8_mtp_concurrent_mixed_budget_and_plain (L206-L289)
```python
@pytest.mark.asyncio
@multi_gpu_only(num_gpus=2)
@requires_fp8
async def test_thinking_token_budget_qwen35_fp8_mtp_concurrent_mixed_budget_and_plain(
    server_qwen35_fp8_mtp_tp2,
):
    """Concurrent chat requests: some with ``thinking_token_budget``, some without.

    Exercises the scheduler / input processor under a mixed batch on the same
    Qwen3.5 FP8 + MTP (TP=2) server. Budgeted calls are checked with
    ``_count_reasoning_decode_token_ids_between_markers`` on full token ids.
    """

    _batch_spec: list[tuple[Literal["budget"], int] | tuple[Literal["plain"], None]] = [
        ("budget", 1),
        ("budget", 12),
        ("plain", None),
        ("budget", 20),
# ... 58 lines omitted for brevity ...
            n_reason = _count_reasoning_decode_token_ids_between_markers(
                full_ids, start_ids, end_ids
            )
            assert n_reason is not None, f"index {i}: missing reasoning start in ids"
            assert n_reason == expected_budget, (
                f"index {i}: reasoning decode token ids ({n_reason}) != "
                f"thinking_token_budget ({expected_budget})"
            )
```
**EN:** This async test validates `test_thinking_token_budget_qwen35_fp8_mtp_concurrent_mixed_budget_and_plain`. Relevant pytest markers include `asyncio`. Key inputs are `server_qwen35_fp8_mtp_tp2`. The body fans out concurrent work via `asyncio.gather`. It drives client-facing request creation through the API surface under test. The main assertion is `msg.content or getattr(msg, 'reasoning', None)` and `expected_budget is not None`.
**CN:** 这个异步测试验证 `test_thinking_token_budget_qwen35_fp8_mtp_concurrent_mixed_budget_and_plain`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server_qwen35_fp8_mtp_tp2`。 函数体通过 `asyncio.gather` 并发展开工作负载。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `msg.content or getattr(msg, 'reasoning', None)` and `expected_budget is not None`。

### Test / 测试: test_streaming_with_thinking_disabled_stays_in_content (L292-L333)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("client", ["default", "auto_config"], indirect=True)
async def test_streaming_with_thinking_disabled_stays_in_content(
    client: openai.AsyncOpenAI,
):
    request_kwargs = {
        "model": MODEL_NAME,
        "messages": [
            {
                "role": "user",
                "content": "Which is larger, 4 or 12?"
                " Output exactly one token: 4 or 12.",
            }
        ],
        "max_tokens": 16,
        "temperature": 0.0,
        "extra_body": {"chat_template_kwargs": {"enable_thinking": False}},
    }
# ... 16 lines omitted for brevity ...
        delta = chunk.choices[0].delta
        if getattr(delta, "content", None):
            content_chunks.append(delta.content)
        if getattr(delta, "reasoning", None):
            reasoning_chunks.append(delta.reasoning)

    assert "".join(content_chunks).strip() != ""
    assert reasoning_chunks == []
```
**EN:** This async test validates `test_streaming_with_thinking_disabled_stays_in_content`. It uses parameterization over `client`. Relevant pytest markers include `asyncio`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. The main assertion is `message.content is not None and message.content.strip() != ''` and `getattr(message, 'reasoning', None) in (None, '')`.
**CN:** 这个异步测试验证 `test_streaming_with_thinking_disabled_stays_in_content`。 它通过参数化组合 `client`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `message.content is not None and message.content.strip() != ''` and `getattr(message, 'reasoning', None) in (None, '')`。

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
- **Stdlib / 标准库**: `asyncio`, `json`, `typing.Literal`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `tests.utils.multi_gpu_only`, `tests.utils.requires_fp8`, `vllm.platforms.current_platform`, `vllm.tokenizers.get_tokenizer`
