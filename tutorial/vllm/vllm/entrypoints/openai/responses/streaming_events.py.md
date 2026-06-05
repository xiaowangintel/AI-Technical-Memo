# streaming_events.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/streaming_events.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines streaming event objects and serialization flow. Scope: OpenAI-compatible responses. / 定义流式事件对象及其序列化流程。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 3-16 — Module overview
```python
"""
Streaming SSE event builders for the Responses API.

Pure functions that translate streaming state + delta data into
OpenAI Response API SSE events. Used by the streaming event
processors in serving.py.

The file is organized as:
  1. StreamingState dataclass + utility helpers
  2. Shared leaf helpers — delta events (take plain strings, no context)
  3. Shared leaf helpers — done events (take plain strings, no context)
  4. Harmony-specific dispatchers (route ctx/previous_item → leaf helpers)
  5. Harmony-specific tool lifecycle helpers
"""
```
**EN:** The module header documents the main contract: Streaming SSE event builders for the Responses API.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 18-76 — Imports and shared dependencies
```python
import json
from collections.abc import Callable
from dataclasses import dataclass, field
from enum import Enum, auto
from typing import Any, ClassVar, Final, NamedTuple

from openai.types.responses import (
    ResponseCodeInterpreterCallCodeDeltaEvent,
    ResponseCodeInterpreterCallCodeDoneEvent,
    ResponseCodeInterpreterCallCompletedEvent,
    ResponseCodeInterpreterCallInProgressEvent,
    ResponseCodeInterpreterCallInterpretingEvent,
    ResponseCodeInterpreterToolCallParam,
    ResponseContentPartAddedEvent,
    ResponseContentPartDoneEvent,
    ResponseFunctionCallArgumentsDeltaEvent,
    ResponseFunctionCallArgumentsDoneEvent,
    ResponseFunctionToolCall,
    ResponseFunctionToolCallItem,
    ResponseFunctionWebSearch,
    ResponseMcpCallArgumentsDeltaEvent,
    ResponseMcpCallArgumentsDoneEvent,
    ResponseMcpCallCompletedEvent,
    ResponseMcpCallInProgressEvent,
    ResponseOutputItemAddedEvent,
    ResponseOutputItemDoneEvent,
...
from vllm.entrypoints.openai.responses.context import StreamingHarmonyContext
from vllm.entrypoints.openai.responses.protocol import (
    ResponseReasoningPartAddedEvent,
    ResponseReasoningPartDoneEvent,
    StreamingResponsesResponse,
)
from vllm.outputs import CompletionOutput
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `json`, `collections`, `dataclasses`, `enum`, `typing`, uses third-party packages like `openai`, `openai_harmony`, depends on internal helpers such as `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.context`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs`.
**CN:** 该导入块引入 `json`, `collections`, `dataclasses`, `enum`, `typing` 等标准库模块，使用 `openai`, `openai_harmony` 等第三方库，依赖 `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.context`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs` 等 vLLM 内部模块。

### Lines 78-82 — Module constants
```python
TOOL_NAME_TO_MCP_SERVER_LABEL: Final[dict[str, str]] = {
    "python": "code_interpreter",
    "container": "container",
    "browser": "web_search_preview",
}
```
**EN:** This block initializes `TOOL_NAME_TO_MCP_SERVER_LABEL`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `TOOL_NAME_TO_MCP_SERVER_LABEL`，为后续逻辑准备模块级常量或共享状态。

### Lines 85-96 — Function `_resolve_mcp_name_label`
```python
def _resolve_mcp_name_label(recipient: str) -> tuple[str, str]:
    """Resolve MCP tool name and server label from a recipient string.

    - ``mcp.*`` recipients: strip prefix, use the bare name as both
      name and server_label.
    - Everything else: use the recipient as the name and look up the
      server_label in TOOL_NAME_TO_MCP_SERVER_LABEL.
    """
    if recipient.startswith("mcp."):
        name = recipient[len("mcp.") :]
        return name, name
    return recipient, TOOL_NAME_TO_MCP_SERVER_LABEL.get(recipient, recipient)
```
**EN:** This function `_resolve_mcp_name_label` is documented as: Resolve MCP tool name and server label from a recipient string.
**CN:** 这里定义函数 `_resolve_mcp_name_label`，其文档字符串说明了主要职责与调用约定。

### Lines 100-115 — Class `StreamingState`
```python
class StreamingState:
    """Mutable state for streaming event processing."""

    current_content_index: int = -1
    current_output_index: int = 0
    current_item_id: str = ""
    current_call_id: str = ""
    sent_output_item_added: bool = False
    is_first_function_call_delta: bool = False

    def reset_for_new_item(self) -> None:
        """Reset state when expecting a new output item."""
        self.current_output_index += 1
        self.sent_output_item_added = False
        self.is_first_function_call_delta = False
        self.current_call_id = ""
```
**EN:** Class `StreamingState` is introduced here. Its docstring describes the intent as: Mutable state for streaming event processing.
**CN:** 这里定义类 `StreamingState`。其文档字符串说明了该类的职责与使用方式。

### Lines 118-130 — Function `is_mcp_tool_by_namespace`
```python
def is_mcp_tool_by_namespace(
    recipient: str | None,
    allowed_function_tool_names: frozenset[str] | None = None,
) -> bool:
    """
    Determine if a tool call is an MCP tool based on recipient prefix.

    Inverse of :func:`is_function_recipient` — everything that is not
    a function call is an MCP tool.
    """
    if recipient is None:
        return False
    return not is_function_recipient(recipient, allowed_function_tool_names)
```
**EN:** This function `is_mcp_tool_by_namespace` is documented as: Determine if a tool call is an MCP tool based on recipient prefix.
**CN:** 这里定义函数 `is_mcp_tool_by_namespace`，其文档字符串说明了主要职责与调用约定。

### Lines 138-189 — Function `emit_text_delta_events`
```python
def emit_text_delta_events(
    delta: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for text content delta streaming."""
    events: list[StreamingResponsesResponse] = []
    if not state.sent_output_item_added:
        state.sent_output_item_added = True
        state.current_item_id = f"msg_{random_uuid()}"
        events.append(
            ResponseOutputItemAddedEvent(
                type="response.output_item.added",
                sequence_number=-1,
                output_index=state.current_output_index,
                item=ResponseOutputMessage(
                    id=state.current_item_id,
                    type="message",
                    role="assistant",
                    content=[],
                    status="in_progress",
                ),
            )
        )
        state.current_content_index += 1
        events.append(
            ResponseContentPartAddedEvent(
                type="response.content_part.added",
                sequence_number=-1,
...
            output_index=state.current_output_index,
            item_id=state.current_item_id,
            delta=delta,
            # TODO, use logprobs from ctx.last_request_output
            logprobs=[],
        )
    )
    return events
```
**EN:** This function `emit_text_delta_events` is documented as: Emit events for text content delta streaming.
**CN:** 这里定义函数 `emit_text_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 192-238 — Function `emit_reasoning_delta_events`
```python
def emit_reasoning_delta_events(
    delta: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for reasoning text delta streaming."""
    events: list[StreamingResponsesResponse] = []
    if not state.sent_output_item_added:
        state.sent_output_item_added = True
        state.current_item_id = f"msg_{random_uuid()}"
        events.append(
            ResponseOutputItemAddedEvent(
                type="response.output_item.added",
                sequence_number=-1,
                output_index=state.current_output_index,
                item=ResponseReasoningItem(
                    type="reasoning",
                    id=state.current_item_id,
                    summary=[],
                    status="in_progress",
                ),
            )
        )
        state.current_content_index += 1
        events.append(
            ResponseReasoningPartAddedEvent(
                type="response.reasoning_part.added",
                sequence_number=-1,
                output_index=state.current_output_index,
...
            item_id=state.current_item_id,
            output_index=state.current_output_index,
            content_index=state.current_content_index,
            delta=delta,
            sequence_number=-1,
        )
    )
    return events
```
**EN:** This function `emit_reasoning_delta_events` is documented as: Emit events for reasoning text delta streaming.
**CN:** 这里定义函数 `emit_reasoning_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 241-278 — Function `emit_function_call_delta_events`
```python
def emit_function_call_delta_events(
    delta: str,
    function_name: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for function call argument deltas."""
    events: list[StreamingResponsesResponse] = []
    if state.is_first_function_call_delta is False:
        state.is_first_function_call_delta = True
        state.current_item_id = f"fc_{random_uuid()}"
        state.current_call_id = f"call_{random_uuid()}"
        tool_call_item = ResponseFunctionToolCall(
            name=function_name,
            type="function_call",
            id=state.current_item_id,
            call_id=state.current_call_id,
            arguments="",
            status="in_progress",
        )
        events.append(
            ResponseOutputItemAddedEvent(
                type="response.output_item.added",
                sequence_number=-1,
                output_index=state.current_output_index,
                item=tool_call_item,
            )
        )
    # Always emit the delta (including on first call)
    events.append(
        ResponseFunctionCallArgumentsDeltaEvent(
            item_id=state.current_item_id,
            delta=delta,
            output_index=state.current_output_index,
            sequence_number=-1,
            type="response.function_call_arguments.delta",
        )
    )
    return events
```
**EN:** This function `emit_function_call_delta_events` is documented as: Emit events for function call argument deltas.
**CN:** 这里定义函数 `emit_function_call_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 281-324 — Function `emit_mcp_delta_events`
```python
def emit_mcp_delta_events(
    delta: str,
    state: StreamingState,
    recipient: str,
) -> list[StreamingResponsesResponse]:
    """Emit events for MCP tool delta streaming."""
    name, server_label = _resolve_mcp_name_label(recipient)
    events: list[StreamingResponsesResponse] = []
    if not state.sent_output_item_added:
        state.sent_output_item_added = True
        state.current_item_id = f"mcp_{random_uuid()}"
        events.append(
            ResponseOutputItemAddedEvent(
                type="response.output_item.added",
                sequence_number=-1,
                output_index=state.current_output_index,
                item=McpCall(
                    type="mcp_call",
                    id=state.current_item_id,
                    name=name,
                    arguments="",
                    server_label=server_label,
                    status="in_progress",
                ),
            )
        )
        events.append(
            ResponseMcpCallInProgressEvent(
...
            type="response.mcp_call_arguments.delta",
            sequence_number=-1,
            output_index=state.current_output_index,
            item_id=state.current_item_id,
            delta=delta,
        )
    )
    return events
```
**EN:** This function `emit_mcp_delta_events` is documented as: Emit events for MCP tool delta streaming.
**CN:** 这里定义函数 `emit_mcp_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 327-368 — Function `emit_code_interpreter_delta_events`
```python
def emit_code_interpreter_delta_events(
    delta: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for code interpreter delta streaming."""
    events: list[StreamingResponsesResponse] = []
    if not state.sent_output_item_added:
        state.sent_output_item_added = True
        state.current_item_id = f"tool_{random_uuid()}"
        events.append(
            ResponseOutputItemAddedEvent(
                type="response.output_item.added",
                sequence_number=-1,
                output_index=state.current_output_index,
                item=ResponseCodeInterpreterToolCallParam(
                    type="code_interpreter_call",
                    id=state.current_item_id,
                    code=None,
                    container_id="auto",
                    outputs=None,
                    status="in_progress",
                ),
            )
        )
        events.append(
            ResponseCodeInterpreterCallInProgressEvent(
                type="response.code_interpreter_call.in_progress",
                sequence_number=-1,
                output_index=state.current_output_index,
                item_id=state.current_item_id,
            )
        )
    events.append(
        ResponseCodeInterpreterCallCodeDeltaEvent(
            type="response.code_interpreter_call_code.delta",
            sequence_number=-1,
            output_index=state.current_output_index,
            item_id=state.current_item_id,
            delta=delta,
        )
    )
    return events
```
**EN:** This function `emit_code_interpreter_delta_events` is documented as: Emit events for code interpreter delta streaming.
**CN:** 这里定义函数 `emit_code_interpreter_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 376-422 — Function `emit_text_output_done_events`
```python
def emit_text_output_done_events(
    text: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events when a final text output item completes."""
    text_content = ResponseOutputText(
        type="output_text",
        text=text,
        annotations=[],
    )
    events: list[StreamingResponsesResponse] = []
    events.append(
        ResponseTextDoneEvent(
            type="response.output_text.done",
            sequence_number=-1,
            output_index=state.current_output_index,
            content_index=state.current_content_index,
            text=text,
            logprobs=[],
            item_id=state.current_item_id,
        )
    )
    events.append(
        ResponseContentPartDoneEvent(
            type="response.content_part.done",
            sequence_number=-1,
            item_id=state.current_item_id,
            output_index=state.current_output_index,
...
                type="message",
                role="assistant",
                content=[text_content],
                status="completed",
            ),
        )
    )
    return events
```
**EN:** This function `emit_text_output_done_events` is documented as: Emit events when a final text output item completes.
**CN:** 这里定义函数 `emit_text_output_done_events`，其文档字符串说明了主要职责与调用约定。

### Lines 425-470 — Function `emit_reasoning_done_events`
```python
def emit_reasoning_done_events(
    text: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events when a reasoning (analysis) item completes."""
    content = ResponseReasoningTextContent(
        text=text,
        type="reasoning_text",
    )
    reasoning_item = ResponseReasoningItem(
        type="reasoning",
        content=[content],
        status="completed",
        id=state.current_item_id,
        summary=[],
    )
    events: list[StreamingResponsesResponse] = []
    events.append(
        ResponseReasoningTextDoneEvent(
            type="response.reasoning_text.done",
            item_id=state.current_item_id,
            sequence_number=-1,
            output_index=state.current_output_index,
            content_index=state.current_content_index,
            text=text,
        )
    )
    events.append(
...
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
            sequence_number=-1,
            output_index=state.current_output_index,
            item=reasoning_item,
        )
    )
    return events
```
**EN:** This function `emit_reasoning_done_events` is documented as: Emit events when a reasoning (analysis) item completes.
**CN:** 这里定义函数 `emit_reasoning_done_events`，其文档字符串说明了主要职责与调用约定。

### Lines 473-508 — Function `emit_function_call_done_events`
```python
def emit_function_call_done_events(
    function_name: str,
    arguments: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events when a function call completes."""
    events: list[StreamingResponsesResponse] = []
    events.append(
        ResponseFunctionCallArgumentsDoneEvent(
            type="response.function_call_arguments.done",
            arguments=arguments,
            name=function_name,
            item_id=state.current_item_id,
            output_index=state.current_output_index,
            sequence_number=-1,
        )
    )
    function_call_item = ResponseFunctionToolCall(
        type="function_call",
        arguments=arguments,
        name=function_name,
        item_id=state.current_item_id,
        output_index=state.current_output_index,
        sequence_number=-1,
        call_id=state.current_call_id,
        status="completed",
    )
    events.append(
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
            sequence_number=-1,
            output_index=state.current_output_index,
            item=function_call_item,
        )
    )
    return events
```
**EN:** This function `emit_function_call_done_events` is documented as: Emit events when a function call completes.
**CN:** 这里定义函数 `emit_function_call_done_events`，其文档字符串说明了主要职责与调用约定。

### Lines 511-552 — Function `emit_mcp_completion_events`
```python
def emit_mcp_completion_events(
    recipient: str,
    arguments: str,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events when an MCP tool call completes."""
    name, server_label = _resolve_mcp_name_label(recipient)
    events: list[StreamingResponsesResponse] = []
    events.append(
        ResponseMcpCallArgumentsDoneEvent(
            type="response.mcp_call_arguments.done",
            arguments=arguments,
            name=name,
            item_id=state.current_item_id,
            output_index=state.current_output_index,
            sequence_number=-1,
        )
    )
    events.append(
        ResponseMcpCallCompletedEvent(
            type="response.mcp_call.completed",
            sequence_number=-1,
            output_index=state.current_output_index,
            item_id=state.current_item_id,
        )
    )
    events.append(
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
            sequence_number=-1,
            output_index=state.current_output_index,
            item=McpCall(
                type="mcp_call",
                arguments=arguments,
                name=name,
                id=state.current_item_id,
                server_label=server_label,
                status="completed",
            ),
        )
    )
    return events
```
**EN:** This function `emit_mcp_completion_events` is documented as: Emit events when an MCP tool call completes.
**CN:** 这里定义函数 `emit_mcp_completion_events`，其文档字符串说明了主要职责与调用约定。

### Lines 560-594 — Function `emit_content_delta_events`
```python
def emit_content_delta_events(
    ctx: StreamingHarmonyContext,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for content delta streaming based on channel type.

    This is a Harmony-specific dispatcher that extracts values from the
    Harmony context and delegates to shared leaf helpers.
    """
    delta = ctx.last_content_delta
    if not delta:
        return []

    channel = ctx.parser.current_channel
    recipient = ctx.parser.current_recipient

    if channel in ("final", "commentary") and recipient is None:
        # Preambles (commentary with no recipient) and final messages
        # are both user-visible text.
        return emit_text_delta_events(delta, state)
    elif channel == "analysis" and recipient is None:
        return emit_reasoning_delta_events(delta, state)
    elif recipient is not None:
        fn_names = ctx.function_tool_names
        if is_function_recipient(recipient, fn_names):
            function_name = extract_function_from_recipient(recipient)
            return emit_function_call_delta_events(delta, function_name, state)
        elif recipient == "python":
            return emit_code_interpreter_delta_events(delta, state)
        elif recipient.startswith("mcp.") or is_mcp_tool_by_namespace(
            recipient, fn_names
        ):
            return emit_mcp_delta_events(delta, state, recipient)

    return []
```
**EN:** This function `emit_content_delta_events` is documented as: Emit events for content delta streaming based on channel type.
**CN:** 这里定义函数 `emit_content_delta_events`，其文档字符串说明了主要职责与调用约定。

### Lines 597-627 — Function `emit_previous_item_done_events`
```python
def emit_previous_item_done_events(
    previous_item: HarmonyMessage,
    state: StreamingState,
    function_tool_names: frozenset[str] | None = None,
) -> list[StreamingResponsesResponse]:
    """Emit done events for the previous item when expecting a new start.

    This is a Harmony-specific dispatcher that extracts values from the
    Harmony parser's message object and delegates to shared leaf helpers.
    """
    text = previous_item.content[0].text
    if previous_item.recipient is not None:
        # Deal with tool call
        if is_function_recipient(previous_item.recipient, function_tool_names):
            function_name = extract_function_from_recipient(previous_item.recipient)
            return emit_function_call_done_events(function_name, text, state)
        elif previous_item.recipient == "python":
            return emit_code_interpreter_completion_events(previous_item, state)
        elif (
            is_mcp_tool_by_namespace(previous_item.recipient, function_tool_names)
            and state.current_item_id is not None
            and state.current_item_id.startswith("mcp_")
        ):
            return emit_mcp_completion_events(previous_item.recipient, text, state)
    elif previous_item.channel == "analysis":
        return emit_reasoning_done_events(text, state)
    elif previous_item.channel in ("commentary", "final"):
        # Preambles (commentary with no recipient) and final messages
        # are both user-visible text.
        return emit_text_output_done_events(text, state)
    return []
```
**EN:** This function `emit_previous_item_done_events` is documented as: Emit done events for the previous item when expecting a new start.
**CN:** 这里定义函数 `emit_previous_item_done_events`，其文档字符串说明了主要职责与调用约定。

### Lines 635-719 — Function `emit_browser_tool_events`
```python
def emit_browser_tool_events(
    previous_item: HarmonyMessage,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events for browser tool calls (web search)."""
    function_name = previous_item.recipient[len("browser.") :]
    parsed_args = json.loads(previous_item.content[0].text)
    action = None

    if function_name == "search":
        action = response_function_web_search.ActionSearch(
            type="search",
            query=parsed_args["query"],
        )
    elif function_name == "open":
        action = response_function_web_search.ActionOpenPage(
            type="open_page",
            # TODO: translate to url
            url=f"cursor:{parsed_args.get('cursor', '')}",
        )
    elif function_name == "find":
        action = response_function_web_search.ActionFind(
            type="find",
            pattern=parsed_args["pattern"],
            # TODO: translate to url
            url=f"cursor:{parsed_args.get('cursor', '')}",
        )
    else:
...
                type="web_search_call",
                id=state.current_item_id,
                action=action,
                status="completed",
            ),
        )
    )
    return events
```
**EN:** This function `emit_browser_tool_events` is documented as: Emit events for browser tool calls (web search).
**CN:** 这里定义函数 `emit_browser_tool_events`，其文档字符串说明了主要职责与调用约定。

### Lines 722-768 — Function `emit_code_interpreter_completion_events`
```python
def emit_code_interpreter_completion_events(
    previous_item: HarmonyMessage,
    state: StreamingState,
) -> list[StreamingResponsesResponse]:
    """Emit events when code interpreter completes."""
    events: list[StreamingResponsesResponse] = []
    events.append(
        ResponseCodeInterpreterCallCodeDoneEvent(
            type="response.code_interpreter_call_code.done",
            sequence_number=-1,
            output_index=state.current_output_index,
            item_id=state.current_item_id,
            code=previous_item.content[0].text,
        )
    )
    events.append(
        ResponseCodeInterpreterCallInterpretingEvent(
            type="response.code_interpreter_call.interpreting",
            sequence_number=-1,
            output_index=state.current_output_index,
            item_id=state.current_item_id,
        )
    )
    events.append(
        ResponseCodeInterpreterCallCompletedEvent(
            type="response.code_interpreter_call.completed",
            sequence_number=-1,
            output_index=state.current_output_index,
...
                code=previous_item.content[0].text,
                container_id="auto",
                outputs=[],
                status="completed",
            ),
        )
    )
    return events
```
**EN:** This function `emit_code_interpreter_completion_events` is documented as: Emit events when code interpreter completes.
**CN:** 这里定义函数 `emit_code_interpreter_completion_events`，其文档字符串说明了主要职责与调用约定。

### Lines 771-812 — Function `emit_tool_action_events`
```python
def emit_tool_action_events(
    ctx: StreamingHarmonyContext,
    state: StreamingState,
    tool_server: ToolServer | None,
) -> list[StreamingResponsesResponse]:
    """Emit events for tool action turn."""
    if not ctx.is_assistant_action_turn() or len(ctx.parser.messages) == 0:
        return []

    events: list[StreamingResponsesResponse] = []
    previous_item = ctx.parser.messages[-1]

    # Handle browser tool
    if (
        tool_server is not None
        and tool_server.has_tool("browser")
        and previous_item.recipient is not None
        and previous_item.recipient.startswith("browser.")
    ):
        events.extend(emit_browser_tool_events(previous_item, state))

    # Handle tool completion
    if (
        tool_server is not None
        and previous_item.recipient is not None
        and state.current_item_id is not None
        and state.sent_output_item_added
    ):
        recipient = previous_item.recipient
        fn_names = ctx.function_tool_names
        if recipient == "python":
            events.extend(emit_code_interpreter_completion_events(previous_item, state))
        elif recipient.startswith("mcp.") or is_mcp_tool_by_namespace(
            recipient, fn_names
        ):
            events.extend(
                emit_mcp_completion_events(
                    recipient, previous_item.content[0].text, state
                )
            )

    return events
```
**EN:** This function `emit_tool_action_events` is documented as: Emit events for tool action turn.
**CN:** 这里定义函数 `emit_tool_action_events`，其文档字符串说明了主要职责与调用约定。

### Lines 820-824 — Class `_StateType`
```python
class _StateType(Enum):
    NONE = auto()
    CONTENT = auto()
    REASONING = auto()
    TOOL_CALL = auto()
```
**EN:** Class `_StateType` is defined here, extending `Enum`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `_StateType`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 828-837 — Class `SimpleStreamingState`
```python
class SimpleStreamingState:
    output_index: int = 0
    current_item_id: str = ""
    content_index: int = 0
    accumulated_text: str = ""
    tool_call_id: str = ""
    tool_call_name: str = ""
    tool_call_index: int | None = None
    has_emitted_tool_call_delta: bool = False
    current_state: _StateType = field(default_factory=lambda: _StateType.NONE)
```
**EN:** Class `SimpleStreamingState` is defined here, as a standalone type, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `SimpleStreamingState`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 840-873 — Function `emit_simple_content_open`
```python
def emit_simple_content_open(
    state: SimpleStreamingState,
) -> list[StreamingResponsesResponse]:
    state.current_state = _StateType.CONTENT
    state.current_item_id = random_uuid()
    state.content_index = 0
    state.accumulated_text = ""
    return [
        ResponseOutputItemAddedEvent(
            type="response.output_item.added",
            sequence_number=-1,
            output_index=state.output_index,
            item=ResponseOutputMessage(
                id=state.current_item_id,
                type="message",
                role="assistant",
                content=[],
                status="in_progress",
            ),
        ),
        ResponseContentPartAddedEvent(
            type="response.content_part.added",
            sequence_number=-1,
            output_index=state.output_index,
            item_id=state.current_item_id,
            content_index=state.content_index,
            part=ResponseOutputText(
                type="output_text",
                text="",
                annotations=[],
                logprobs=[],
            ),
        ),
    ]
```
**EN:** This function `emit_simple_content_open` implements the `emit simple content open` step within the module flow.
**CN:** 该函数 `emit_simple_content_open` 实现了模块流程中的“emitsimplecontentopen”步骤。

### Lines 876-892 — Function `emit_simple_content_delta`
```python
def emit_simple_content_delta(
    state: SimpleStreamingState,
    delta: str,
    logprobs: list[response_text_delta_event.Logprob] | None = None,
) -> list[StreamingResponsesResponse]:
    state.accumulated_text += delta
    return [
        ResponseTextDeltaEvent(
            type="response.output_text.delta",
            sequence_number=-1,
            content_index=state.content_index,
            output_index=state.output_index,
            item_id=state.current_item_id,
            delta=delta,
            logprobs=logprobs or [],
        )
    ]
```
**EN:** This function `emit_simple_content_delta` implements the `emit simple content delta` step within the module flow.
**CN:** 该函数 `emit_simple_content_delta` 实现了模块流程中的“emitsimplecontentdelta”步骤。

### Lines 895-937 — Function `emit_simple_content_done`
```python
def emit_simple_content_done(
    state: SimpleStreamingState,
) -> list[StreamingResponsesResponse]:
    part = ResponseOutputText(
        type="output_text",
        text=state.accumulated_text,
        annotations=[],
    )
    events: list[StreamingResponsesResponse] = [
        ResponseTextDoneEvent(
            type="response.output_text.done",
            sequence_number=-1,
            output_index=state.output_index,
            content_index=state.content_index,
            text=state.accumulated_text,
            logprobs=[],
            item_id=state.current_item_id,
        ),
        ResponseContentPartDoneEvent(
            type="response.content_part.done",
            sequence_number=-1,
            item_id=state.current_item_id,
            output_index=state.output_index,
            content_index=state.content_index,
            part=part,
        ),
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
...
                status="completed",
                summary=[],
            ),
        ),
    ]
    state.output_index += 1
    state.current_state = _StateType.NONE
    return events
```
**EN:** This function `emit_simple_content_done` implements the `emit simple content done` step within the module flow.
**CN:** 该函数 `emit_simple_content_done` 实现了模块流程中的“emitsimplecontentdone”步骤。

### Lines 940-970 — Function `emit_simple_reasoning_open`
```python
def emit_simple_reasoning_open(
    state: SimpleStreamingState,
) -> list[StreamingResponsesResponse]:
    state.current_state = _StateType.REASONING
    state.current_item_id = random_uuid()
    state.content_index = 0
    state.accumulated_text = ""
    return [
        ResponseOutputItemAddedEvent(
            type="response.output_item.added",
            sequence_number=-1,
            output_index=state.output_index,
            item=ResponseReasoningItem(
                type="reasoning",
                id=state.current_item_id,
                summary=[],
                status="in_progress",
            ),
        ),
        ResponseReasoningPartAddedEvent(
            type="response.reasoning_part.added",
            sequence_number=-1,
            output_index=state.output_index,
            item_id=state.current_item_id,
            content_index=state.content_index,
            part=ResponseReasoningTextContent(
                text="",
                type="reasoning_text",
            ),
        ),
    ]
```
**EN:** This function `emit_simple_reasoning_open` implements the `emit simple reasoning open` step within the module flow.
**CN:** 该函数 `emit_simple_reasoning_open` 实现了模块流程中的“emitsimplereasoningopen”步骤。

### Lines 973-987 — Function `emit_simple_reasoning_delta`
```python
def emit_simple_reasoning_delta(
    state: SimpleStreamingState,
    delta: str,
) -> list[StreamingResponsesResponse]:
    state.accumulated_text += delta
    return [
        ResponseReasoningTextDeltaEvent(
            type="response.reasoning_text.delta",
            item_id=state.current_item_id,
            sequence_number=-1,
            output_index=state.output_index,
            content_index=state.content_index,
            delta=delta,
        )
    ]
```
**EN:** This function `emit_simple_reasoning_delta` implements the `emit simple reasoning delta` step within the module flow.
**CN:** 该函数 `emit_simple_reasoning_delta` 实现了模块流程中的“emitsimplereasoningdelta”步骤。

### Lines 990-1029 — Function `emit_simple_reasoning_done`
```python
def emit_simple_reasoning_done(
    state: SimpleStreamingState,
) -> list[StreamingResponsesResponse]:
    part = ResponseReasoningTextContent(
        text=state.accumulated_text,
        type="reasoning_text",
    )
    events: list[StreamingResponsesResponse] = [
        ResponseReasoningTextDoneEvent(
            type="response.reasoning_text.done",
            item_id=state.current_item_id,
            sequence_number=-1,
            output_index=state.output_index,
            content_index=state.content_index,
            text=state.accumulated_text,
        ),
        ResponseReasoningPartDoneEvent(
            type="response.reasoning_part.done",
            sequence_number=-1,
            item_id=state.current_item_id,
            output_index=state.output_index,
            content_index=state.content_index,
            part=part,
        ),
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
            sequence_number=-1,
            output_index=state.output_index,
            item=ResponseReasoningItem(
                type="reasoning",
                content=[part],
                status="completed",
                id=state.current_item_id,
                summary=[],
            ),
        ),
    ]
    state.output_index += 1
    state.current_state = _StateType.NONE
    return events
```
**EN:** This function `emit_simple_reasoning_done` implements the `emit simple reasoning done` step within the module flow.
**CN:** 该函数 `emit_simple_reasoning_done` 实现了模块流程中的“emitsimplereasoningdone”步骤。

### Lines 1032-1058 — Function `emit_simple_tool_call_open`
```python
def emit_simple_tool_call_open(
    state: SimpleStreamingState,
    name: str,
    index: int | None,
) -> list[StreamingResponsesResponse]:
    state.current_state = _StateType.TOOL_CALL
    state.current_item_id = random_uuid()
    state.tool_call_id = f"call_{random_uuid()}"
    state.tool_call_name = name
    state.tool_call_index = index
    state.accumulated_text = ""
    state.has_emitted_tool_call_delta = False
    return [
        ResponseOutputItemAddedEvent(
            type="response.output_item.added",
            sequence_number=-1,
            output_index=state.output_index,
            item=ResponseFunctionToolCallItem(
                type="function_call",
                id=state.current_item_id,
                call_id=state.tool_call_id,
                name=name,
                arguments="",
                status="in_progress",
            ),
        ),
    ]
```
**EN:** This function `emit_simple_tool_call_open` implements the `emit simple tool call open` step within the module flow.
**CN:** 该函数 `emit_simple_tool_call_open` 实现了模块流程中的“emitsimpletoolcallopen”步骤。

### Lines 1061-1075 — Function `emit_simple_tool_call_delta`
```python
def emit_simple_tool_call_delta(
    state: SimpleStreamingState,
    delta: str,
) -> list[StreamingResponsesResponse]:
    state.accumulated_text += delta
    state.has_emitted_tool_call_delta = True
    return [
        ResponseFunctionCallArgumentsDeltaEvent(
            type="response.function_call_arguments.delta",
            sequence_number=-1,
            output_index=state.output_index,
            item_id=state.current_item_id,
            delta=delta,
        )
    ]
```
**EN:** This function `emit_simple_tool_call_delta` implements the `emit simple tool call delta` step within the module flow.
**CN:** 该函数 `emit_simple_tool_call_delta` 实现了模块流程中的“emitsimpletoolcalldelta”步骤。

### Lines 1078-1110 — Function `emit_simple_tool_call_done`
```python
def emit_simple_tool_call_done(
    state: SimpleStreamingState,
) -> list[StreamingResponsesResponse]:
    events: list[StreamingResponsesResponse] = []
    if state.has_emitted_tool_call_delta:
        events.append(
            ResponseFunctionCallArgumentsDoneEvent(
                type="response.function_call_arguments.done",
                sequence_number=-1,
                output_index=state.output_index,
                item_id=state.current_item_id,
                arguments=state.accumulated_text,
                name=state.tool_call_name,
            )
        )
    events.append(
        ResponseOutputItemDoneEvent(
            type="response.output_item.done",
            sequence_number=-1,
            output_index=state.output_index,
            item=ResponseFunctionToolCall(
                type="function_call",
                name=state.tool_call_name,
                arguments=state.accumulated_text,
                status="completed",
                id=state.current_item_id,
                call_id=state.tool_call_id,
            ),
        ),
    )
    state.output_index += 1
    state.current_state = _StateType.NONE
    return events
```
**EN:** This function `emit_simple_tool_call_done` implements the `emit simple tool call done` step within the module flow.
**CN:** 该函数 `emit_simple_tool_call_done` 实现了模块流程中的“emitsimpletoolcalldone”步骤。

### Lines 1113-1118 — Class `_StateHandlers`
```python
class _StateHandlers(NamedTuple):
    """Tuple for each state: open(start), delta(chunk), done(finish)."""

    open_fn: Callable[..., list[StreamingResponsesResponse]]
    delta_fn: Callable[..., list[StreamingResponsesResponse]]
    done_fn: Callable[..., list[StreamingResponsesResponse]]
```
**EN:** Class `_StateHandlers` is introduced here. Its docstring describes the intent as: Tuple for each state: open(start), delta(chunk), done(finish).
**CN:** 这里定义类 `_StateHandlers`。其文档字符串说明了该类的职责与使用方式。

### Lines 1121-1266 — Class `SimpleStreamingEventProcessor`
```python
class SimpleStreamingEventProcessor:
    """
    State-machine processor for the simple (non-Harmony) streaming path.

    Core flow:
      1. Resolve the target state from the delta_message
         (CONTENT / REASONING / TOOL_CALL).
      2. If the target state differs from the current one,
         close_current() then open() the new state.
      3. emit_delta() produces the incremental events for the state.

    State lifecycle:
      open()  ->  repeated emit_delta()  ->  close_current()
    """

    _STATE_HANDLERS: ClassVar[dict[_StateType, _StateHandlers]] = {
        _StateType.CONTENT: _StateHandlers(
            emit_simple_content_open,
            emit_simple_content_delta,
            emit_simple_content_done,
        ),
        _StateType.REASONING: _StateHandlers(
            emit_simple_reasoning_open,
            emit_simple_reasoning_delta,
            emit_simple_reasoning_done,
        ),
        _StateType.TOOL_CALL: _StateHandlers(
            emit_simple_tool_call_open,
...
        elif self.state.current_state == _StateType.REASONING:
            assert delta_message.reasoning is not None
            return handlers.delta_fn(self.state, delta_message.reasoning)
        elif self.state.current_state == _StateType.CONTENT:
            assert delta_message.content is not None
            logprobs = get_logprobs(output) if get_logprobs else []
            return handlers.delta_fn(self.state, delta_message.content, logprobs)
        return []
```
**EN:** Class `SimpleStreamingEventProcessor` is introduced here. Its docstring describes the intent as: State-machine processor for the simple (non-Harmony) streaming path.
**CN:** 这里定义类 `SimpleStreamingEventProcessor`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- Streaming responses / 流式响应
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections`, `dataclasses`, `enum`, `typing`
- **Third-party / 第三方**: `openai`, `openai_harmony`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.context`, `vllm.entrypoints.openai.responses.protocol`, `vllm.outputs`, `vllm.utils`
