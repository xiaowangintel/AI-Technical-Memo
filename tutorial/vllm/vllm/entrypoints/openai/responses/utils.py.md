# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides reusable helper utilities shared across related entrypoints. Scope: OpenAI-compatible responses. / 提供在相关入口之间复用的辅助工具函数。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 4-27 — Imports and shared dependencies
```python
from collections.abc import Iterable
from typing import Any

from openai.types.chat import (
    ChatCompletionAssistantMessageParam,
    ChatCompletionMessageToolCallParam,
    ChatCompletionToolMessageParam,
)
from openai.types.chat.chat_completion_message_tool_call_param import (
    Function as FunctionCallTool,
)
from openai.types.responses import ResponseFunctionToolCall, ResponseOutputItem
from openai.types.responses.response import ToolChoice
from openai.types.responses.response_function_tool_call_output_item import (
    ResponseFunctionToolCallOutputItem,
)
from openai.types.responses.response_output_message import ResponseOutputMessage
from openai.types.responses.response_reasoning_item import ResponseReasoningItem
from openai.types.responses.tool import Tool

from vllm import envs
from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionMessageParam
from vllm.entrypoints.openai.responses.protocol import ResponseInputOutputItem
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `collections`, `typing`, uses third-party packages like `openai`, depends on internal helpers such as `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`.
**CN:** 该导入块引入 `collections`, `typing` 等标准库模块，使用 `openai` 等第三方库，依赖 `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger` 等 vLLM 内部模块。

### Lines 29-29 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 32-76 — Function `should_continue_final_message`
```python
def should_continue_final_message(
    request_input: str | list[ResponseInputOutputItem],
) -> bool:
    """
    Determine if the last input message is a partial assistant message
    that should be continued rather than starting a new generation.

    This enables partial message completion similar to Anthropic's Messages API,
    where users can provide an incomplete assistant message and have the model
    continue from where it left off.

    A message is considered partial if:
    1. It's a ResponseOutputMessage or ResponseReasoningItem
    2. Its status is "in_progress" or "incomplete"

    Args:
        request_input: The input to the Responses API request

    Returns:
        True if the final message should be continued, False otherwise
    """
    if isinstance(request_input, str):
        # Simple string input is always a user message
        return False

    if not request_input:
        return False

...

    if isinstance(last_item, dict):
        # only support partial completion for messages for now
        if last_item.get("type", "message") not in ("message", "reasoning"):
            return False
        return last_item.get("status") in ("in_progress", "incomplete")

    return False
```
**EN:** This function `should_continue_final_message` is documented as: Determine if the last input message is a partial assistant message that should be continued rather than starting a new generation.
**CN:** 这里定义函数 `should_continue_final_message`，其文档字符串说明了主要职责与调用约定。

### Lines 79-121 — Function `construct_input_messages`
```python
def construct_input_messages(
    *,
    request_instructions: str | None = None,
    request_input: str | list[ResponseInputOutputItem],
    prev_msg: list[ChatCompletionMessageParam] | None = None,
    prev_response_output: list[ResponseOutputItem] | None = None,
):
    messages: list[ChatCompletionMessageParam] = []
    if request_instructions:
        messages.append(
            {
                "role": "system",
                "content": request_instructions,
            }
        )

    # Prepend the conversation history.
    if prev_msg is not None:
        # Filter out system messages from previous conversation -- per the
        # OpenAI spec, instructions should NOT carry over across responses.
        # The current request's instructions (if any) were already added above.
        messages.extend(m for m in prev_msg if m.get("role") != "system")
    if prev_response_output is not None:
        # Add the previous output.
        for output_item in prev_response_output:
            # NOTE: We skip the reasoning output.
            if isinstance(output_item, ResponseOutputMessage):
                for content in output_item.content:
...
    # Append the new input.
    # Responses API supports simple text inputs without chat format.
    if isinstance(request_input, str):
        messages.append({"role": "user", "content": request_input})
    else:
        input_messages = construct_chat_messages_with_tool_call(request_input)
        messages.extend(input_messages)
    return messages
```
**EN:** This function `construct_input_messages` implements the `construct input messages` step within the module flow.
**CN:** 该函数 `construct_input_messages` 实现了模块流程中的“constructinputmessages”步骤。

### Lines 124-139 — Function `construct_chat_messages_with_tool_call`
```python
def construct_chat_messages_with_tool_call(
    input_messages: list[ResponseInputOutputItem],
) -> list[ChatCompletionMessageParam]:
    """Build chat messages from response items.

    Some chat messages span multiple response items (e.g., reasoning + tool calls).
    """
    messages: list[ChatCompletionMessageParam] = []
    for item in input_messages:
        message = _construct_message_from_response_item(
            item, prev_msg=messages[-1] if messages else None
        )
        if message is not None:
            messages.append(message)

    return messages
```
**EN:** This function `construct_chat_messages_with_tool_call` is documented as: Build chat messages from response items.
**CN:** 这里定义函数 `construct_chat_messages_with_tool_call`，其文档字符串说明了主要职责与调用约定。

### Lines 142-236 — Function `_construct_message_from_response_item`
```python
def _construct_message_from_response_item(
    item: ResponseInputOutputItem,
    prev_msg: ChatCompletionMessageParam | None = None,
) -> ChatCompletionMessageParam | None:
    """
    Returns a new message or None. If `None`, `prev_msg` might be updated.
    If `prev_msg` is `None`, a new message is always returned.
    """
    prev_assistant_msg = (
        prev_msg if prev_msg and prev_msg.get("role") == "assistant" else None
    )

    if isinstance(item, ResponseFunctionToolCall):
        tool_call = ChatCompletionMessageToolCallParam(
            id=item.call_id,
            function=FunctionCallTool(
                name=item.name,
                arguments=item.arguments,
            ),
            type="function",
        )
        if prev_assistant_msg:
            tool_calls = prev_assistant_msg.get("tool_calls")
            if tool_calls is None:
                prev_assistant_msg["tool_calls"] = [tool_call]
                return None
            if isinstance(tool_calls, list):
                tool_calls.append(tool_call)
...
    elif isinstance(item, dict) and item.get("type") == "function_call_output":
        # Append the function call output as a tool message.
        return ChatCompletionToolMessageParam(
            role="tool",
            content=item.get("output"),
            tool_call_id=item.get("call_id"),
        )
    return item  # type: ignore[arg-type]
```
**EN:** This function `_construct_message_from_response_item` is documented as: Returns a new message or None.
**CN:** 这里定义函数 `_construct_message_from_response_item`，其文档字符串说明了主要职责与调用约定。

### Lines 239-240 — Function `extract_function_tool_names`
```python
def extract_function_tool_names(tools: list[Tool]) -> frozenset[str]:
    return frozenset(tool.name for tool in tools if tool.type == "function")
```
**EN:** This function `extract_function_tool_names` implements the `extract function tool names` step within the module flow.
**CN:** 该函数 `extract_function_tool_names` 实现了模块流程中的“extractfunctiontoolnames”步骤。

### Lines 243-257 — Function `extract_tool_types`
```python
def extract_tool_types(tools: list[Tool]) -> set[str]:
    """
    Extracts the tool types from the given tools.
    """
    tool_types: set[str] = set()
    for tool in tools:
        if tool.type == "mcp":
            # Allow the MCP Tool type to enable built in tools if the
            # server_label is allowlisted in
            # envs.VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS
            if tool.server_label in envs.VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS:
                tool_types.add(tool.server_label)
        else:
            tool_types.add(tool.type)
    return tool_types
```
**EN:** This function `extract_tool_types` is documented as: Extracts the tool types from the given tools.
**CN:** 这里定义函数 `extract_tool_types`，其文档字符串说明了主要职责与调用约定。

### Lines 260-270 — Function `convert_tool_responses_to_completions_format`
```python
def convert_tool_responses_to_completions_format(tool: dict) -> dict:
    """
    Convert a flat tool schema:
        {"type": "function", "name": "...", "description": "...", "parameters": {...}}
    into:
        {"type": "function", "function": {...}}
    """
    return {
        "type": "function",
        "function": tool,
    }
```
**EN:** This function `convert_tool_responses_to_completions_format` is documented as: Convert a flat tool schema: {"type": "function", "name": "...", "description": "...", "parameters": {...}} into: {"type": "function", "function": {...}}
**CN:** 这里定义函数 `convert_tool_responses_to_completions_format`，其文档字符串说明了主要职责与调用约定。

### Lines 273-283 — Function `construct_tool_dicts`
```python
def construct_tool_dicts(
    tools: list[Tool], tool_choice: ToolChoice
) -> list[dict[str, Any]] | None:
    if not tools or (tool_choice == "none"):
        tool_dicts = None
    else:
        tool_dicts = [
            convert_tool_responses_to_completions_format(tool.model_dump())
            for tool in tools
        ]
    return tool_dicts
```
**EN:** This function `construct_tool_dicts` implements the `construct tool dicts` step within the module flow.
**CN:** 该函数 `construct_tool_dicts` 实现了模块流程中的“constructtooldicts”步骤。

## Key Concepts / 关键概念
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections`, `typing`
- **Third-party / 第三方**: `openai`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`
