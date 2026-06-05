# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for OpenAI-compatible serving. The file exposes 3 helper/class block(s) used by nearby tests. / [CN] 为OpenAI 兼容服务提供共享测试工具。该文件暴露了 3 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L13)
```python
import json
from collections.abc import AsyncGenerator
from typing import Any

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionResponse,
    ChatCompletionResponseChoice,
    ChatCompletionStreamResponse,
    ChatMessage,
)
from vllm.entrypoints.openai.engine.protocol import UsageInfo
```
**EN:** Imports standard-library modules such as `collections.abc.AsyncGenerator`, `json`, `typing.Any`, project helpers such as `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponse`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponseChoice`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionStreamResponse`.
**CN:** 导入标准库模块（如 `collections.abc.AsyncGenerator`、`json`、`typing.Any`）、项目内辅助模块（如 `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponse`、`vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponseChoice`、`vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionStreamResponse`）。

### Helper / 辅助函数: accumulate_streaming_response (L16-L138)
```python
async def accumulate_streaming_response(
    stream_generator: AsyncGenerator[str, None],
) -> ChatCompletionResponse:
    """
    Accumulate streaming SSE chunks into a complete ChatCompletionResponse.

    This helper parses the SSE format and builds up the complete response
    by combining all the delta chunks.
    """
    accumulated_content = ""
    accumulated_reasoning = None
    accumulated_tool_calls: list[dict[str, Any]] = []
    role = None
    finish_reason = None
    response_id = None
    created = None
    model = None
    index = 0
# ... 97 lines omitted for brevity ...
        object="chat.completion",
        created=created or 0,
        model=model or "test-model",
        choices=[choice],
        usage=usage,
    )

    return response
```
**EN:** This async helper encapsulates reusable logic in `accumulate_streaming_response`. Key inputs are `stream_generator`. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `accumulate_streaming_response` 中。 关键输入包括 `stream_generator`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: verify_harmony_messages (L141-L161)
```python
def verify_harmony_messages(
    messages: list[Any], expected_messages: list[dict[str, Any]]
):
    assert len(messages) == len(expected_messages)
    for msg, expected in zip(messages, expected_messages):
        if "role" in expected:
            assert msg.author.role == expected["role"]
        if "author_name" in expected:
            assert msg.author.name == expected["author_name"]
        if "channel" in expected:
            assert msg.channel == expected["channel"]
        if "recipient" in expected:
            assert msg.recipient == expected["recipient"]
        if "content" in expected:
            assert msg.content[0].text == expected["content"]
        if "content_type" in expected:
            assert msg.content_type == expected["content_type"]
        if "tool_definitions" in expected:
            # Check that the tool definitions match the expected list of tool names
            actual_tools = [t.name for t in msg.content[0].tools["functions"].tools]
            assert actual_tools == expected["tool_definitions"]
```
**EN:** This helper encapsulates reusable logic in `verify_harmony_messages`. Key inputs are `messages`, `expected_messages`. The main assertion is `len(messages) == len(expected_messages)` and `msg.author.role == expected['role']`.
**CN:** 这个辅助函数将可复用逻辑封装在 `verify_harmony_messages` 中。 关键输入包括 `messages`、`expected_messages`。 核心断言是 `len(messages) == len(expected_messages)` and `msg.author.role == expected['role']`。

### Helper / 辅助函数: verify_chat_response (L164-L190)
```python
def verify_chat_response(
    response: ChatCompletionResponse,
    content: str | None = None,
    reasoning: str | None = None,
    tool_calls: list[tuple[str, str]] | None = None,
):
    assert len(response.choices) == 1
    message = response.choices[0].message

    if content is not None:
        assert message.content == content
    else:
        assert not message.content

    if reasoning is not None:
        assert message.reasoning == reasoning
    else:
        assert not message.reasoning

    if tool_calls:
        assert message.tool_calls is not None
        assert len(message.tool_calls) == len(tool_calls)
        for tc, (expected_name, expected_args) in zip(message.tool_calls, tool_calls):
            assert tc.function.name == expected_name
            assert tc.function.arguments == expected_args
    else:
        assert not message.tool_calls
```
**EN:** This helper encapsulates reusable logic in `verify_chat_response`. Key inputs are `response`, `content`, `reasoning`, `tool_calls`. The main assertion is `len(response.choices) == 1` and `message.content == content`.
**CN:** 这个辅助函数将可复用逻辑封装在 `verify_chat_response` 中。 关键输入包括 `response`、`content`、`reasoning`、`tool_calls`。 核心断言是 `len(response.choices) == 1` and `message.content == content`。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.AsyncGenerator`, `json`, `typing.Any`
- **Project / 项目内**: `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponse`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionResponseChoice`, `vllm.entrypoints.openai.chat_completion.protocol.ChatCompletionStreamResponse`, `vllm.entrypoints.openai.chat_completion.protocol.ChatMessage`, `vllm.entrypoints.openai.engine.protocol.UsageInfo`
