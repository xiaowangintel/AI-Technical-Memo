# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared pytest fixtures and hooks for Responses API behavior and OpenAI-compatible serving. / [CN] 为Responses API 行为与OpenAI 兼容服务提供共享的 pytest fixture 与钩子。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L13)
```python
from __future__ import annotations

import json
import logging
from collections.abc import Callable
from typing import Any

import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `__future__.annotations`, `collections.abc.Callable`, `json`, third-party packages like `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `__future__.annotations`、`collections.abc.Callable`、`json`）、第三方包（如 `pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: logger, BASE_TEST_ENV, DEFAULT_MAX_RETRIES (L15-L21)
```python
logger = logging.getLogger(__name__)

BASE_TEST_ENV = {
    # The day vLLM said "hello world" on arxiv 🚀
    "VLLM_SYSTEM_START_DATE": "2023-09-12",
}
DEFAULT_MAX_RETRIES = 3
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `logger`, `BASE_TEST_ENV`, `DEFAULT_MAX_RETRIES`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `logger`、`BASE_TEST_ENV`、`DEFAULT_MAX_RETRIES`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: pairs_of_event_types (L24-L49)
```python
@pytest.fixture
def pairs_of_event_types() -> dict[str, str]:
    """Links the 'done' event type with the corresponding 'start' event type.

    This mapping should link all done <-> start events; if tests mean to
    restrict the allowed events, they should filter this fixture to avoid
    copy + paste errors in the mappings or unexpected KeyErrors due to missing
    events.
    """
    # fmt: off
    event_pairs = {
        "response.completed": "response.created",
        "response.output_item.done": "response.output_item.added",
        "response.content_part.done": "response.content_part.added",
        "response.output_text.done": "response.output_text.delta",
        "response.reasoning_text.done": "response.reasoning_text.delta",
        "response.reasoning_part.done": "response.reasoning_part.added",
        "response.mcp_call_arguments.done": "response.mcp_call_arguments.delta",
        "response.mcp_call.completed": "response.mcp_call.in_progress",
        "response.function_call_arguments.done": "response.function_call_arguments.delta", # noqa: E501
        "response.code_interpreter_call_code.done": "response.code_interpreter_call_code.delta", # noqa: E501
        "response.code_interpreter_call.completed": "response.code_interpreter_call.in_progress", # noqa: E501
        "response.web_search_call.completed": "response.web_search_call.in_progress",
    }
    # fmt: on
    return event_pairs
```
**EN:** This fixture prepares `pairs_of_event_types` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `pairs_of_event_types`。

### Helper / 辅助函数: retry_for_tool_call (L52-L76)
```python
async def retry_for_tool_call(
    client,
    *,
    model: str,
    expected_tool_type: str,
    max_retries: int = DEFAULT_MAX_RETRIES,
    **create_kwargs: Any,
):
    """Call ``client.responses.create`` up to *max_retries* times, returning
    the first response that contains an output item of *expected_tool_type*.

    Returns the **last** response if none match so the caller's assertions
    fire with a clear diagnostic.
    """
    last_response = None
    for attempt in range(max_retries):
        response = await client.responses.create(model=model, **create_kwargs)
        last_response = response
        if any(
            getattr(item, "type", None) == expected_tool_type
            for item in response.output
        ):
            return response
    assert last_response is not None
    return last_response
```
**EN:** This async helper encapsulates reusable logic in `retry_for_tool_call`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller. The main assertion is `last_response is not None`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `retry_for_tool_call` 中。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `last_response is not None`。

### Helper / 辅助函数: retry_streaming_for (L79-L102)
```python
async def retry_streaming_for(
    client,
    *,
    model: str,
    validate_events: Callable[[list], bool],
    max_retries: int = DEFAULT_MAX_RETRIES,
    **create_kwargs: Any,
) -> list:
    """Call ``client.responses.create(stream=True)`` up to *max_retries*
    times, returning the first event list where *validate_events* returns
    ``True``.
    """
    last_events: list = []
    for attempt in range(max_retries):
        stream = await client.responses.create(
            model=model, stream=True, **create_kwargs
        )
        events: list = []
        async for event in stream:
            events.append(event)
        last_events = events
        if validate_events(events):
            return events
    return last_events
```
**EN:** This async helper encapsulates reusable logic in `retry_streaming_for`. Key inputs are `client`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `retry_streaming_for` 中。 关键输入包括 `client`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: has_output_type (L105-L107)
```python
def has_output_type(response, type_name: str) -> bool:
    """Return True if *response* has at least one output item of *type_name*."""
    return any(getattr(item, "type", None) == type_name for item in response.output)
```
**EN:** This helper encapsulates reusable logic in `has_output_type`. Key inputs are `response`, `type_name`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `has_output_type` 中。 关键输入包括 `response`、`type_name`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: events_contain_type (L110-L112)
```python
def events_contain_type(events: list, type_substring: str) -> bool:
    """Return True if any event's type contains *type_substring*."""
    return any(type_substring in getattr(e, "type", "") for e in events)
```
**EN:** This helper encapsulates reusable logic in `events_contain_type`. Key inputs are `events`, `type_substring`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `events_contain_type` 中。 关键输入包括 `events`、`type_substring`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _validate_event_pairing (L115-L142)
```python
def _validate_event_pairing(events: list, pairs_of_event_types: dict[str, str]) -> None:
    """Validate that streaming events are properly nested/paired.

    Derives push/pop sets from *pairs_of_event_types* so that every
    start/end pair in the dict is handled automatically.
    """
    start_events = set(pairs_of_event_types.values())
    end_events = set(pairs_of_event_types.keys())

    stack: list[str] = []
    for event in events:
        etype = event.type
        if etype in end_events:
            expected_start = pairs_of_event_types[etype]
            assert stack and stack[-1] == expected_start, (
                f"Stack mismatch for {etype}: "
                f"expected {expected_start}, "
                f"got {stack[-1] if stack else '<empty>'}"
            )
            stack.pop()
        elif etype in start_events:
            # Consecutive deltas of the same type share a single stack slot.
            if etype.endswith("delta") and stack and stack[-1] == etype:
                continue
            stack.append(etype)
        # else: passthrough event (e.g. response.in_progress,
        # web_search_call.searching, code_interpreter_call.interpreting)
    assert len(stack) == 0, f"Unclosed events on stack: {stack}"
```
**EN:** This helper encapsulates reusable logic in `_validate_event_pairing`. Key inputs are `events`, `pairs_of_event_types`. The main assertion is `len(stack) == 0` and `stack and stack[-1] == expected_start`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_validate_event_pairing` 中。 关键输入包括 `events`、`pairs_of_event_types`。 核心断言是 `len(stack) == 0` and `stack and stack[-1] == expected_start`。

### Helper / 辅助函数: _validate_event_ordering (L145-L176)
```python
def _validate_event_ordering(events: list) -> None:
    """Validate that envelope events appear in the correct positions."""
    assert len(events) >= 2, f"Expected at least 2 events, got {len(events)}"

    # First event must be response.created
    assert events[0].type == "response.created", (
        f"First event must be response.created, got {events[0].type}"
    )
    # Last event must be response.completed
    assert events[-1].type == "response.completed", (
        f"Last event must be response.completed, got {events[-1].type}"
    )

    # response.in_progress, if present, must be the second event
    in_progress_indices = [
        i for i, e in enumerate(events) if e.type == "response.in_progress"
    ]
    if in_progress_indices:
# ... 6 lines omitted for brevity ...
    created_count = sum(1 for e in events if e.type == "response.created")
    completed_count = sum(1 for e in events if e.type == "response.completed")
    assert created_count == 1, (
        f"Expected exactly 1 response.created, got {created_count}"
    )
    assert completed_count == 1, (
        f"Expected exactly 1 response.completed, got {completed_count}"
    )
```
**EN:** This helper encapsulates reusable logic in `_validate_event_ordering`. Key inputs are `events`. The main assertion is `len(events) >= 2` and `events[0].type == 'response.created'`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_validate_event_ordering` 中。 关键输入包括 `events`。 核心断言是 `len(events) >= 2` and `events[0].type == 'response.created'`。

### Helper / 辅助函数: _validate_field_consistency (L179-L266)
```python
def _validate_field_consistency(events: list) -> None:
    """Validate item_id, output_index, and content_index consistency.

    Tracks the active output item established by ``output_item.added``
    and verifies that all subsequent events for that item carry matching
    identifiers until ``output_item.done`` closes it.
    """
    _SESSION_EVENTS = {
        "response.created",
        "response.in_progress",
        "response.completed",
    }

    active_item_id: str | None = None
    active_output_index: int | None = None
    last_output_index: int = -1
    active_content_index: int | None = None

# ... 62 lines omitted for brevity ...

        # content_index (only meaningful on events that carry it)
        content_index = getattr(event, "content_index", None)
        if content_index is not None and active_content_index is not None:
            assert content_index == active_content_index, (
                f"{etype} content_index mismatch: "
                f"expected {active_content_index}, got {content_index}"
            )
```
**EN:** This helper encapsulates reusable logic in `_validate_field_consistency`. Key inputs are `events`. The main assertion is `item is not None` and `item_id`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_validate_field_consistency` 中。 关键输入包括 `events`。 核心断言是 `item is not None` and `item_id`。

### Helper / 辅助函数: _assert_item_fields (L269-L287)
```python
def _assert_item_fields(
    event,
    etype: str,
    active_item_id: str | None,
    active_output_index: int | None,
) -> None:
    """Check that *event*'s item_id and output_index match the active item."""
    event_item_id = getattr(event, "item_id", None)
    output_index = getattr(event, "output_index", None)

    if active_item_id is not None and event_item_id is not None:
        assert event_item_id == active_item_id, (
            f"{etype} item_id mismatch: expected {active_item_id}, got {event_item_id}"
        )
    if active_output_index is not None and output_index is not None:
        assert output_index == active_output_index, (
            f"{etype} output_index mismatch: "
            f"expected {active_output_index}, got {output_index}"
        )
```
**EN:** This helper encapsulates reusable logic in `_assert_item_fields`. Key inputs are `event`, `etype`, `active_item_id`, `active_output_index`. The main assertion is `event_item_id == active_item_id` and `output_index == active_output_index`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_item_fields` 中。 关键输入包括 `event`、`etype`、`active_item_id`、`active_output_index`。 核心断言是 `event_item_id == active_item_id` and `output_index == active_output_index`。

### Helper / 辅助函数: validate_streaming_event_stack (L290-L306)
```python
def validate_streaming_event_stack(
    events: list, pairs_of_event_types: dict[str, str]
) -> None:
    """Validate streaming events: pairing, ordering, and field consistency.

    Checks three aspects:
    1. **Event pairing** — start/end events are properly nested
       (stack-based matching derived from *pairs_of_event_types*).
    2. **Event ordering** — envelope events (``created``,
       ``in_progress``, ``completed``) appear at the correct positions.
    3. **Field consistency** — ``item_id``, ``output_index``, and
       ``content_index`` are consistent across related events within
       each output item's lifecycle.
    """
    _validate_event_pairing(events, pairs_of_event_types)
    _validate_event_ordering(events)
    _validate_field_consistency(events)
```
**EN:** This helper encapsulates reusable logic in `validate_streaming_event_stack`. Key inputs are `events`, `pairs_of_event_types`.
**CN:** 这个辅助函数将可复用逻辑封装在 `validate_streaming_event_stack` 中。 关键输入包括 `events`、`pairs_of_event_types`。

### Helper / 辅助函数: log_response_diagnostics (L309-L366)
```python
def log_response_diagnostics(
    response,
    *,
    label: str = "Response Diagnostics",
) -> dict[str, Any]:
    """Extract and log diagnostic info from a Responses API response.

    Logs reasoning, tool-call attempts, MCP items, and output types so
    that CI output (``pytest -s`` or ``--log-cli-level=INFO``) gives
    full visibility into model behaviour even on passing runs.

    Returns the extracted data so callers can make additional assertions
    if needed.
    """
    reasoning_texts = [
        text
        for item in response.output
        if getattr(item, "type", None) == "reasoning"
# ... 32 lines omitted for brevity ...

    logger.info(
        "\n====== %s ======\n%s\n==============================",
        label,
        json.dumps(diagnostics, indent=2, default=str),
    )

    return diagnostics
```
**EN:** This helper encapsulates reusable logic in `log_response_diagnostics`. Key inputs are `response`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `log_response_diagnostics` 中。 关键输入包括 `response`。 它把计算得到的状态或辅助对象返回给调用方。

### Fixture / 夹具: default_server_args (L369-L382)
```python
@pytest.fixture(scope="module")
def default_server_args():
    return [
        "--max-model-len",
        "18192",
        "--enforce-eager",  # For faster startup.
        "--enable-auto-tool-choice",
        "--structured-outputs-config.backend",
        "xgrammar",
        "--tool-call-parser",
        "hermes",
        "--reasoning-parser",
        "qwen3",
    ]
```
**EN:** This fixture prepares `default_server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `default_server_args`。

### Fixture / 夹具: server_with_store (L385-L395)
```python
@pytest.fixture(scope="module")
def server_with_store(default_server_args):
    with RemoteOpenAIServer(
        "Qwen/Qwen3-1.7B",
        default_server_args,
        env_dict={
            "VLLM_ENABLE_RESPONSES_API_STORE": "1",
            "VLLM_SERVER_DEV_MODE": "1",
        },
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server_with_store` for dependent tests. Key inputs are `default_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server_with_store`。 关键输入包括 `default_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L398-L401)
```python
@pytest_asyncio.fixture
async def client(server_with_store):
    async with server_with_store.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server_with_store`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server_with_store`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `__future__.annotations`, `collections.abc.Callable`, `json`, `logging`, `typing.Any`
- **Third-party / 第三方**: `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
