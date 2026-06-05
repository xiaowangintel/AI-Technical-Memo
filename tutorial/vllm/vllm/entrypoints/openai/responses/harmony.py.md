# harmony.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/harmony.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements model- or format-specific harmony handling logic. Scope: OpenAI-compatible responses. / 实现特定模型或格式的 Harmony 处理逻辑。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 3-9 — Module overview
```python
"""
Harmony ↔ Responses API conversion utilities.

Handles two directions:
  1. Response Input → Harmony Messages  (input parsing)
  2. Harmony Messages → Response Output Items  (output parsing)
"""
```
**EN:** The module header documents the main contract: Harmony ↔ Responses API conversion utilities.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 11-43 — Imports and shared dependencies
```python
import json

from openai.types.responses import (
    ResponseFunctionToolCall,
    ResponseOutputItem,
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseReasoningItem,
)
from openai.types.responses.response_function_web_search import (
    ActionFind,
    ActionOpenPage,
    ActionSearch,
    ResponseFunctionWebSearch,
)
from openai.types.responses.response_output_item import McpCall
from openai.types.responses.response_reasoning_item import (
    Content as ResponseReasoningTextContent,
)
from openai_harmony import Author, Message, Role, StreamableParser, TextContent

from vllm.entrypoints.openai.parser.harmony_utils import (
    BUILTIN_TOOL_TO_MCP_SERVER_LABEL,
    extract_function_from_recipient,
    flatten_chat_text_content,
    is_function_recipient,
)
from vllm.entrypoints.openai.responses.protocol import (
    ResponseInputOutputItem,
    ResponsesRequest,
)
from vllm.logger import init_logger
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `json`, uses third-party packages like `openai`, `openai_harmony`, depends on internal helpers such as `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.utils`.
**CN:** 该导入块引入 `json` 等标准库模块，使用 `openai`, `openai_harmony` 等第三方库，依赖 `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.utils` 等 vLLM 内部模块。

### Lines 45-45 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 52-83 — Function `_parse_harmony_format_message`
```python
def _parse_harmony_format_message(chat_msg: dict) -> Message:
    """Reconstruct a Message from Harmony-format dict,
    preserving channel, recipient, and content_type."""
    author_dict = chat_msg["author"]
    role = author_dict.get("role")
    name = author_dict.get("name")

    raw_content = chat_msg.get("content", "")
    if isinstance(raw_content, list):
        # TODO: Support refusal and non-text content types.
        contents = [TextContent(text=c.get("text", "")) for c in raw_content]
    elif isinstance(raw_content, str):
        contents = [TextContent(text=raw_content)]
    else:
        contents = [TextContent(text="")]

    if name:
        msg = Message.from_author_and_contents(Author.new(Role(role), name), contents)
    else:
        msg = Message.from_role_and_contents(Role(role), contents)

    channel = chat_msg.get("channel")
    if channel:
        msg = msg.with_channel(channel)
    recipient = chat_msg.get("recipient")
    if recipient:
        msg = msg.with_recipient(recipient)
    content_type = chat_msg.get("content_type")
    if content_type:
        msg = msg.with_content_type(content_type)

    return msg
```
**EN:** This function `_parse_harmony_format_message` is documented as: Reconstruct a Message from Harmony-format dict, preserving channel, recipient, and content_type.
**CN:** 这里定义函数 `_parse_harmony_format_message`，其文档字符串说明了主要职责与调用约定。

### Lines 86-132 — Function `_parse_chat_format_message`
```python
def _parse_chat_format_message(chat_msg: dict) -> list[Message]:
    """Parse an OpenAI chat-format dict into Harmony messages."""
    role = chat_msg.get("role")
    if role is None:
        raise ValueError(f"Message has no 'role' key: {chat_msg}")

    # Assistant message with tool calls
    tool_calls = chat_msg.get("tool_calls")
    if role == "assistant" and tool_calls:
        msgs: list[Message] = []
        for call in tool_calls:
            func = call.get("function", {})
            name = func.get("name", "")
            arguments = func.get("arguments", "") or ""
            msg = Message.from_role_and_content(Role.ASSISTANT, arguments)
            msg = msg.with_channel("commentary")
            msg = msg.with_recipient(f"functions.{name}")
            msg = msg.with_content_type("json")
            msgs.append(msg)
        return msgs

    # Tool role message (tool output)
    if role == "tool":
        name = chat_msg.get("name", "")
        if name and not name.startswith("functions."):
            name = f"functions.{name}"
        content = chat_msg.get("content", "") or ""
        content = flatten_chat_text_content(content)
...
    content = chat_msg.get("content", "")
    if isinstance(content, str):
        contents = [TextContent(text=content)]
    else:
        # TODO: Support refusal.
        contents = [TextContent(text=c.get("text", "")) for c in content]
    msg = Message.from_role_and_contents(role, contents)
    return [msg]
```
**EN:** This function `_parse_chat_format_message` is documented as: Parse an OpenAI chat-format dict into Harmony messages.
**CN:** 这里定义函数 `_parse_chat_format_message`，其文档字符串说明了主要职责与调用约定。

### Lines 140-197 — Function `response_input_to_harmony`
```python
def response_input_to_harmony(
    response_msg: ResponseInputOutputItem,
    prev_responses: list[ResponseOutputItem | ResponseReasoningItem],
) -> Message | None:
    """Convert a single ResponseInputOutputItem into a Harmony Message.

    Returns None for reasoning items with empty or absent content so
    the caller can skip them.
    """
    if not isinstance(response_msg, dict):
        response_msg = response_msg.model_dump()
    if "type" not in response_msg or response_msg["type"] == "message":
        role = response_msg["role"]
        content = response_msg["content"]
        # Add prefix for developer messages.
        # <|start|>developer<|message|># Instructions {instructions}<|end|>
        text_prefix = "Instructions:\n" if role == "developer" else ""
        if isinstance(content, str):
            msg = Message.from_role_and_content(role, text_prefix + content)
        else:
            contents = [TextContent(text=text_prefix + c["text"]) for c in content]
            msg = Message.from_role_and_contents(role, contents)
        if role == "assistant":
            msg = msg.with_channel("final")
    elif response_msg["type"] == "function_call_output":
        call_id = response_msg["call_id"]
        call_response: ResponseFunctionToolCall | None = None
        for prev_response in reversed(prev_responses):
...
    elif response_msg["type"] == "function_call":
        msg = Message.from_role_and_content(Role.ASSISTANT, response_msg["arguments"])
        msg = msg.with_channel("commentary")
        msg = msg.with_recipient(f"functions.{response_msg['name']}")
        msg = msg.with_content_type("json")
    else:
        raise ValueError(f"Unknown input type: {response_msg['type']}")
    return msg
```
**EN:** This function `response_input_to_harmony` is documented as: Convert a single ResponseInputOutputItem into a Harmony Message.
**CN:** 这里定义函数 `response_input_to_harmony`，其文档字符串说明了主要职责与调用约定。

### Lines 200-213 — Function `response_previous_input_to_harmony`
```python
def response_previous_input_to_harmony(chat_msg) -> list[Message]:
    """Parse a message from request.previous_input_messages
    into Harmony messages.

    Supports both OpenAI chat format ({"role": "..."}) and
    Harmony format ({"author": {"role": "..."}}).
    """
    if not isinstance(chat_msg, dict):
        chat_msg = chat_msg.model_dump(exclude_none=True)

    if "author" in chat_msg and isinstance(chat_msg.get("author"), dict):
        return [_parse_harmony_format_message(chat_msg)]

    return _parse_chat_format_message(chat_msg)
```
**EN:** This function `response_previous_input_to_harmony` is documented as: Parse a message from request.previous_input_messages into Harmony messages.
**CN:** 这里定义函数 `response_previous_input_to_harmony`，其文档字符串说明了主要职责与调用约定。

### Lines 216-240 — Function `construct_harmony_previous_input_messages`
```python
def construct_harmony_previous_input_messages(
    request: ResponsesRequest,
) -> list[Message]:
    """Build a Harmony message list from request.previous_input_messages.

    Filters out system/developer messages to match OpenAI behavior where
    instructions are always taken from the most recent Responses API request.
    """
    messages: list[Message] = []
    if request.previous_input_messages:
        for message in request.previous_input_messages:
            # Handle both Message objects and dictionary inputs
            if isinstance(message, Message):
                message_role = message.author.role
                if message_role == Role.SYSTEM or message_role == Role.DEVELOPER:
                    continue
                messages.append(message)
            else:
                harmony_messages = response_previous_input_to_harmony(message)
                for harmony_msg in harmony_messages:
                    message_role = harmony_msg.author.role
                    if message_role == Role.SYSTEM or message_role == Role.DEVELOPER:
                        continue
                    messages.append(harmony_msg)
    return messages
```
**EN:** This function `construct_harmony_previous_input_messages` is documented as: Build a Harmony message list from request.previous_input_messages.
**CN:** 这里定义函数 `construct_harmony_previous_input_messages`，其文档字符串说明了主要职责与调用约定。

### Lines 248-294 — Function `_parse_browser_tool_call`
```python
def _parse_browser_tool_call(message: Message, recipient: str) -> ResponseOutputItem:
    """Parse browser tool calls (search, open, find) into web search items."""
    if len(message.content) != 1:
        raise ValueError("Invalid number of contents in browser message")
    content = message.content[0]

    # Parse JSON args (with retry detection)
    try:
        browser_call = json.loads(content.text)
    except json.JSONDecodeError:
        logger.warning(
            "Invalid JSON in browser tool call, using error placeholder: %s",
            content.text,
        )
        json_retry_output_message = (
            f"Invalid JSON args, caught and retried: {content.text}"
        )
        browser_call = {
            "query": json_retry_output_message,
            "url": json_retry_output_message,
            "pattern": json_retry_output_message,
        }

    # Create appropriate action based on recipient
    if recipient == "browser.search":
        action = ActionSearch(
            query=f"cursor:{browser_call.get('query', '')}", type="search"
        )
...
        raise ValueError(f"Unknown browser action: {recipient}")

    return ResponseFunctionWebSearch(
        id=f"ws_{random_uuid()}",
        action=action,
        status="completed",
        type="web_search_call",
    )
```
**EN:** This function `_parse_browser_tool_call` is documented as: Parse browser tool calls (search, open, find) into web search items.
**CN:** 这里定义函数 `_parse_browser_tool_call`，其文档字符串说明了主要职责与调用约定。

### Lines 297-311 — Function `_parse_function_call`
```python
def _parse_function_call(message: Message, recipient: str) -> list[ResponseOutputItem]:
    """Parse function calls into function tool call items."""
    function_name = extract_function_from_recipient(recipient)
    output_items = []
    for content in message.content:
        random_id = random_uuid()
        response_item = ResponseFunctionToolCall(
            arguments=content.text,
            call_id=f"call_{random_id}",
            type="function_call",
            name=function_name,
            id=f"fc_{random_id}",
        )
        output_items.append(response_item)
    return output_items
```
**EN:** This function `_parse_function_call` is documented as: Parse function calls into function tool call items.
**CN:** 这里定义函数 `_parse_function_call`，其文档字符串说明了主要职责与调用约定。

### Lines 314-328 — Function `_parse_reasoning`
```python
def _parse_reasoning(message: Message) -> list[ResponseOutputItem]:
    """Parse reasoning/analysis content into reasoning items."""
    output_items = []
    for content in message.content:
        reasoning_item = ResponseReasoningItem(
            id=f"rs_{random_uuid()}",
            summary=[],
            type="reasoning",
            content=[
                ResponseReasoningTextContent(text=content.text, type="reasoning_text")
            ],
            status=None,
        )
        output_items.append(reasoning_item)
    return output_items
```
**EN:** This function `_parse_reasoning` is documented as: Parse reasoning/analysis content into reasoning items.
**CN:** 这里定义函数 `_parse_reasoning`，其文档字符串说明了主要职责与调用约定。

### Lines 331-348 — Function `_parse_final_message`
```python
def _parse_final_message(message: Message) -> ResponseOutputItem:
    """Parse final channel messages into output message items."""
    contents = []
    for content in message.content:
        output_text = ResponseOutputText(
            text=content.text,
            annotations=[],  # TODO
            type="output_text",
            logprobs=None,  # TODO
        )
        contents.append(output_text)
    return ResponseOutputMessage(
        id=f"msg_{random_uuid()}",
        content=contents,
        role=message.author.role,
        status="completed",
        type="message",
    )
```
**EN:** This function `_parse_final_message` is documented as: Parse final channel messages into output message items.
**CN:** 这里定义函数 `_parse_final_message`，其文档字符串说明了主要职责与调用约定。

### Lines 351-368 — Function `_parse_mcp_recipient`
```python
def _parse_mcp_recipient(recipient: str) -> tuple[str, str]:
    """Parse MCP recipient into (server_label, tool_name).

    For dotted recipients like "repo_browser.list":
        - server_label: "repo_browser" (namespace/server)
        - tool_name: "list" (specific tool)

    For simple recipients like "filesystem":
        - server_label: "filesystem"
        - tool_name: "filesystem"
    """
    if "." in recipient:
        server_label = recipient.split(".")[0]
        tool_name = recipient.split(".")[-1]
    else:
        server_label = recipient
        tool_name = recipient
    return server_label, tool_name
```
**EN:** This function `_parse_mcp_recipient` is documented as: Parse MCP recipient into (server_label, tool_name).
**CN:** 这里定义函数 `_parse_mcp_recipient`，其文档字符串说明了主要职责与调用约定。

### Lines 371-391 — Function `_parse_mcp_call`
```python
def _parse_mcp_call(message: Message, recipient: str) -> list[ResponseOutputItem]:
    """Parse MCP calls into MCP call items."""
    # Handle built-in tools that need server_label mapping
    if recipient in BUILTIN_TOOL_TO_MCP_SERVER_LABEL:
        server_label = BUILTIN_TOOL_TO_MCP_SERVER_LABEL[recipient]
        tool_name = recipient
    else:
        server_label, tool_name = _parse_mcp_recipient(recipient)

    output_items = []
    for content in message.content:
        response_item = McpCall(
            arguments=content.text,
            type="mcp_call",
            name=tool_name,
            server_label=server_label,
            id=f"mcp_{random_uuid()}",
            status="completed",
        )
        output_items.append(response_item)
    return output_items
```
**EN:** This function `_parse_mcp_call` is documented as: Parse MCP calls into MCP call items.
**CN:** 这里定义函数 `_parse_mcp_call`，其文档字符串说明了主要职责与调用约定。

### Lines 394-407 — Function `_parse_message_no_recipient`
```python
def _parse_message_no_recipient(
    message: Message,
) -> list[ResponseOutputItem]:
    """Parse a Harmony message with no recipient based on its channel."""
    if message.channel == "analysis":
        return _parse_reasoning(message)

    if message.channel in ("commentary", "final"):
        # Per Harmony format, preambles (commentary with no recipient) and
        # final channel content are both intended to be shown to end-users.
        # See: https://cookbook.openai.com/articles/openai-harmony
        return [_parse_final_message(message)]

    raise ValueError(f"Unknown channel: {message.channel}")
```
**EN:** This function `_parse_message_no_recipient` is documented as: Parse a Harmony message with no recipient based on its channel.
**CN:** 这里定义函数 `_parse_message_no_recipient`，其文档字符串说明了主要职责与调用约定。

### Lines 415-453 — Function `harmony_to_response_output`
```python
def harmony_to_response_output(
    message: Message,
    function_tool_names: frozenset[str] | None = None,
) -> list[ResponseOutputItem]:
    """Parse a Harmony message into a list of output response items.

    This is the main dispatcher that routes based on channel and recipient.
    """
    if message.author.role != "assistant":
        # This is a message from a tool to the assistant (e.g., search result).
        # Don't include it in the final output for now. This aligns with
        # OpenAI's behavior on models like o4-mini.
        return []

    output_items: list[ResponseOutputItem] = []
    recipient = message.recipient

    if recipient is not None:
        # Browser tool calls (browser.search, browser.open, browser.find)
        if recipient.startswith("browser."):
            output_items.append(_parse_browser_tool_call(message, recipient))

        # Function calls (with or without "functions." prefix)
        elif is_function_recipient(recipient, function_tool_names):
            output_items.extend(_parse_function_call(message, recipient))

        # Built-in MCP tools (python, browser, container)
        elif recipient in BUILTIN_TOOL_TO_MCP_SERVER_LABEL:
            output_items.extend(_parse_reasoning(message))

        # All other recipients are MCP calls
        else:
            output_items.extend(_parse_mcp_call(message, recipient))

    # No recipient - handle based on channel for non-tool messages
    else:
        output_items.extend(_parse_message_no_recipient(message))

    return output_items
```
**EN:** This function `harmony_to_response_output` is documented as: Parse a Harmony message into a list of output response items.
**CN:** 这里定义函数 `harmony_to_response_output`，其文档字符串说明了主要职责与调用约定。

### Lines 456-568 — Function `parser_state_to_response_output`
```python
def parser_state_to_response_output(
    parser: StreamableParser,
    function_tool_names: frozenset[str] | None = None,
) -> list[ResponseOutputItem]:
    """Extract in-progress response items from incomplete parser state.

    Called when the parser has buffered content that hasn't formed a
    complete message yet (e.g., generation was cut short).
    """
    if not parser.current_content:
        return []
    if parser.current_role != Role.ASSISTANT:
        return []
    current_recipient = parser.current_recipient
    if current_recipient is not None and current_recipient.startswith("browser."):
        return []

    if current_recipient:
        if is_function_recipient(current_recipient, function_tool_names):
            rid = random_uuid()
            return [
                ResponseFunctionToolCall(
                    arguments=parser.current_content,
                    call_id=f"call_{rid}",
                    type="function_call",
                    name=extract_function_from_recipient(current_recipient),
                    id=f"fc_{rid}",
                    status="in_progress",
...
            # if the parser still has messages (ie if the generator got cut
            # abruptly), this should be incomplete
            status="incomplete",
            type="message",
        )
        return [text_item]

    return []
```
**EN:** This function `parser_state_to_response_output` is documented as: Extract in-progress response items from incomplete parser state.
**CN:** 这里定义函数 `parser_state_to_response_output`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `openai_harmony`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.utils`
