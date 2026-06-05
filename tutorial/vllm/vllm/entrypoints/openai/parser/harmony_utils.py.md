# harmony_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/parser/harmony_utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides Harmony-related parsing and formatting helpers. Scope: OpenAI-compatible parser. / 提供 Harmony 相关解析与格式化辅助逻辑。 作用域：OpenAI 兼容 / 解析器。

## Line-by-Line Analysis / 逐行分析
### Lines 4-25 — Imports and shared dependencies
```python
import datetime
from collections.abc import Iterable, Sequence

from openai.types.responses.tool import Tool
from openai_harmony import (
    Author,
    Conversation,
    DeveloperContent,
    HarmonyEncodingName,
    Message,
    ReasoningEffort,
    Role,
    StreamableParser,
    SystemContent,
    TextContent,
    ToolDescription,
    load_harmony_encoding,
)

from vllm import envs
from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionToolsParam
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `datetime`, `collections`, uses third-party packages like `openai`, `openai_harmony`, depends on internal helpers such as `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.logger`.
**CN:** 该导入块引入 `datetime`, `collections` 等标准库模块，使用 `openai`, `openai_harmony` 等第三方库，依赖 `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.logger` 等 vLLM 内部模块。

### Lines 27-27 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 30-58 — Function `is_function_recipient`
```python
def is_function_recipient(
    recipient: str,
    allowed_function_tool_names: frozenset[str] | None = None,
) -> bool:
    """Check whether *recipient* refers to a function tool call.

    The optional *allowed_function_tool_names* parameter is used by the
    Responses API to distinguish bare function-call recipients (missing the
    ``functions.`` prefix) from MCP tool calls.  When provided, a bare
    recipient is only treated as a function call if it appears in the set.
    The Chat Completions path omits this parameter so that all bare
    recipients are accepted as function calls (the heuristic fallback).
    """
    if not recipient:
        return False
    if recipient.startswith("<|"):
        return False
    if recipient.startswith("functions."):
        return len(recipient) > len("functions.")
    if recipient == "assistant":
        return False
    if recipient in BUILTIN_TOOL_TO_MCP_SERVER_LABEL:
        return False
    first_segment = recipient.split(".", 1)[0]
    if first_segment in BUILTIN_TOOL_TO_MCP_SERVER_LABEL:
        return False
    if allowed_function_tool_names is not None:
        return recipient in allowed_function_tool_names
    return True
```
**EN:** This function `is_function_recipient` is documented as: Check whether *recipient* refers to a function tool call.
**CN:** 这里定义函数 `is_function_recipient`，其文档字符串说明了主要职责与调用约定。

### Lines 61-62 — Function `extract_function_from_recipient`
```python
def extract_function_from_recipient(recipient: str) -> str:
    return recipient.removeprefix("functions.")
```
**EN:** This function `extract_function_from_recipient` implements the `extract function constructs from recipient` step within the module flow.
**CN:** 该函数 `extract_function_from_recipient` 实现了模块流程中的“extractfunction从…构造recipient”步骤。

### Lines 65-84 — Module constants
```python
REASONING_EFFORT = {
    "high": ReasoningEffort.HIGH,
    "medium": ReasoningEffort.MEDIUM,
    "low": ReasoningEffort.LOW,
}

_harmony_encoding = None

# Builtin tools that should be included in the system message when
# they are available and requested by the user.
# Tool args are provided by MCP tool descriptions. Output
# of the tools are stringified.
BUILTIN_TOOL_TO_MCP_SERVER_LABEL: dict[str, str] = {
    "python": "code_interpreter",
    "browser": "web_search_preview",
    "container": "container",
}

# Derive MCP_BUILTIN_TOOLS from the canonical mapping
MCP_BUILTIN_TOOLS: set[str] = set(BUILTIN_TOOL_TO_MCP_SERVER_LABEL.values())
```
**EN:** This block initializes `REASONING_EFFORT`, `_harmony_encoding`, `BUILTIN_TOOL_TO_MCP_SERVER_LABEL`, `MCP_BUILTIN_TOOLS`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `REASONING_EFFORT`, `_harmony_encoding`, `BUILTIN_TOOL_TO_MCP_SERVER_LABEL`, `MCP_BUILTIN_TOOLS`，为后续逻辑准备模块级常量或共享状态。

### Lines 87-92 — Function `has_custom_tools`
```python
def has_custom_tools(tool_types: set[str]) -> bool:
    """
    Checks if the given tool types are custom tools
    (i.e. any tool other than MCP builtin tools)
    """
    return not tool_types.issubset(MCP_BUILTIN_TOOLS)
```
**EN:** This function `has_custom_tools` is documented as: Checks if the given tool types are custom tools (i.e.
**CN:** 这里定义函数 `has_custom_tools`，其文档字符串说明了主要职责与调用约定。

### Lines 95-99 — Function `get_encoding`
```python
def get_encoding():
    global _harmony_encoding
    if _harmony_encoding is None:
        _harmony_encoding = load_harmony_encoding(HarmonyEncodingName.HARMONY_GPT_OSS)
    return _harmony_encoding
```
**EN:** This function `get_encoding` implements the `gets encoding` step within the module flow.
**CN:** 该函数 `get_encoding` 实现了模块流程中的“获取encoding”步骤。

### Lines 102-145 — Function `get_system_message`
```python
def get_system_message(
    model_identity: str | None = None,
    reasoning_effort: str | None = None,
    start_date: str | None = None,
    browser_description: str | None = None,
    python_description: str | None = None,
    container_description: str | None = None,
    instructions: str | None = None,
    with_custom_tools: bool = False,
) -> Message:
    sys_msg_content = SystemContent.new()
    if model_identity is not None:
        sys_msg_content = sys_msg_content.with_model_identity(model_identity)
    if instructions is not None and envs.VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS:
        current_identity = sys_msg_content.model_identity
        new_identity = (
            f"{current_identity}\n{instructions}" if current_identity else instructions
        )
        sys_msg_content = sys_msg_content.with_model_identity(new_identity)
    if reasoning_effort is not None:
        if reasoning_effort not in REASONING_EFFORT:
            supported_values = ", ".join(REASONING_EFFORT)
            raise ValueError(
                f"reasoning_effort={reasoning_effort!r} is not supported by "
                f"Harmony. Supported values are: {supported_values}."
            )
        sys_msg_content = sys_msg_content.with_reasoning_effort(
            REASONING_EFFORT[reasoning_effort]
...
    if browser_description is not None:
        sys_msg_content = sys_msg_content.with_tools(browser_description)
    if python_description is not None:
        sys_msg_content = sys_msg_content.with_tools(python_description)
    if container_description is not None:
        sys_msg_content = sys_msg_content.with_tools(container_description)
    sys_msg = Message.from_role_and_content(Role.SYSTEM, sys_msg_content)
    return sys_msg
```
**EN:** This function `get_system_message` implements the `gets system message` step within the module flow.
**CN:** 该函数 `get_system_message` 实现了模块流程中的“获取systemmessage”步骤。

### Lines 148-159 — Function `create_tool_definition`
```python
def create_tool_definition(tool: ChatCompletionToolsParam | Tool):
    if isinstance(tool, ChatCompletionToolsParam):
        return ToolDescription.new(
            name=tool.function.name,
            description=tool.function.description,
            parameters=tool.function.parameters,
        )
    return ToolDescription.new(
        name=tool.name,
        description=tool.description,
        parameters=tool.parameters,
    )
```
**EN:** This function `create_tool_definition` implements the `creates tool definition` step within the module flow.
**CN:** 该函数 `create_tool_definition` 实现了模块流程中的“创建tooldefinition”步骤。

### Lines 162-191 — Function `get_developer_message`
```python
def get_developer_message(
    instructions: str | None = None,
    tools: list[Tool | ChatCompletionToolsParam] | None = None,
) -> Message:
    dev_msg_content = DeveloperContent.new()
    if instructions is not None and not envs.VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS:
        dev_msg_content = dev_msg_content.with_instructions(instructions)
    if tools is not None:
        function_tools: list[Tool | ChatCompletionToolsParam] = []
        for tool in tools:
            if tool.type in (
                "web_search_preview",
                "code_interpreter",
                "container",
            ):
                pass

            elif tool.type == "function":
                function_tools.append(tool)
            else:
                raise ValueError(f"tool type {tool.type} not supported")
        if function_tools:
            function_tool_descriptions = [
                create_tool_definition(tool) for tool in function_tools
            ]
            dev_msg_content = dev_msg_content.with_function_tools(
                function_tool_descriptions
            )
    dev_msg = Message.from_role_and_content(Role.DEVELOPER, dev_msg_content)
    return dev_msg
```
**EN:** This function `get_developer_message` implements the `gets developer message` step within the module flow.
**CN:** 该函数 `get_developer_message` 实现了模块流程中的“获取developermessage”步骤。

### Lines 194-195 — Function `get_user_message`
```python
def get_user_message(content: str) -> Message:
    return Message.from_role_and_content(Role.USER, content)
```
**EN:** This function `get_user_message` implements the `gets user message` step within the module flow.
**CN:** 该函数 `get_user_message` 实现了模块流程中的“获取usermessage”步骤。

### Lines 198-217 — Function `parse_chat_inputs_to_harmony_messages`
```python
def parse_chat_inputs_to_harmony_messages(chat_msgs: list) -> list[Message]:
    """
    Parse a list of messages from request.messages in the Chat Completion API to
    Harmony messages.
    """
    msgs: list[Message] = []
    tool_id_names: dict[str, str] = {}

    # Collect tool id to name mappings for tool response recipient values
    for chat_msg in chat_msgs:
        for tool_call in chat_msg.get("tool_calls", []):
            tool_id_names[tool_call.get("id")] = tool_call.get("function", {}).get(
                "name"
            )

    for chat_msg in chat_msgs:
        msgs.extend(parse_chat_input_to_harmony_message(chat_msg, tool_id_names))

    msgs = auto_drop_analysis_messages(msgs)
    return msgs
```
**EN:** This function `parse_chat_inputs_to_harmony_messages` is documented as: Parse a list of messages from request.messages in the Chat Completion API to Harmony messages.
**CN:** 这里定义函数 `parse_chat_inputs_to_harmony_messages`，其文档字符串说明了主要职责与调用约定。

### Lines 220-249 — Function `auto_drop_analysis_messages`
```python
def auto_drop_analysis_messages(msgs: list[Message]) -> list[Message]:
    """
    Harmony models expect the analysis messages (representing raw chain of thought) to
    be dropped after an assistant message to the final channel is produced from the
    reasoning of those messages.

    The openai-harmony library does this if the very last assistant message is to the
    final channel, but it does not handle the case where we're in longer multi-turn
    conversations and the client gave us reasoning content from previous turns of
    the conversation with multiple assistant messages to the final channel in the
    conversation.

    So, we find the index of the last assistant message to the final channel and drop
    all analysis messages that precede it, leaving only the analysis messages that
    are relevant to the current part of the conversation.
    """
    last_assistant_final_index = -1
    for i in range(len(msgs) - 1, -1, -1):
        msg = msgs[i]
        if msg.author.role == "assistant" and msg.channel == "final":
            last_assistant_final_index = i
            break

    cleaned_msgs: list[Message] = []
    for i, msg in enumerate(msgs):
        if i < last_assistant_final_index and msg.channel == "analysis":
            continue
        cleaned_msgs.append(msg)

    return cleaned_msgs
```
**EN:** This function `auto_drop_analysis_messages` is documented as: Harmony models expect the analysis messages (representing raw chain of thought) to be dropped after an assistant message to the final channel is produced from the reasoning of those messages.
**CN:** 这里定义函数 `auto_drop_analysis_messages`，其文档字符串说明了主要职责与调用约定。

### Lines 252-263 — Function `flatten_chat_text_content`
```python
def flatten_chat_text_content(content: str | list | None) -> str | None:
    """
    Extract the text parts from a chat message content field and flatten them
    into a single string.
    """
    if isinstance(content, list):
        return "".join(
            item.get("text", "")
            for item in content
            if isinstance(item, dict) and item.get("type") == "text"
        )
    return content
```
**EN:** This function `flatten_chat_text_content` is documented as: Extract the text parts from a chat message content field and flatten them into a single string.
**CN:** 这里定义函数 `flatten_chat_text_content`，其文档字符串说明了主要职责与调用约定。

### Lines 266-357 — Function `parse_chat_input_to_harmony_message`
```python
def parse_chat_input_to_harmony_message(
    chat_msg, tool_id_names: dict[str, str] | None = None
) -> list[Message]:
    """
    Parse a message from request.messages in the Chat Completion API to
    Harmony messages.
    """
    tool_id_names = tool_id_names or {}

    if not isinstance(chat_msg, dict):
        # Handle Pydantic models
        chat_msg = chat_msg.model_dump(exclude_none=True)

    role = chat_msg.get("role")
    msgs: list[Message] = []

    # Assistant message with tool calls
    tool_calls = chat_msg.get("tool_calls", [])

    if role == "assistant" and tool_calls:
        content = flatten_chat_text_content(chat_msg.get("content"))
        if content:
            commentary_msg = Message.from_role_and_content(Role.ASSISTANT, content)
            commentary_msg = commentary_msg.with_channel("commentary")
            msgs.append(commentary_msg)

        reasoning = chat_msg.get("reasoning")
        if reasoning:
...
        msg = msg.with_channel("final")
        msgs.append(msg)
    # For user/system/developer messages, add them directly even if no content.
    elif role != "assistant":
        msg = Message.from_role_and_contents(role, contents)
        msgs.append(msg)

    return msgs
```
**EN:** This function `parse_chat_input_to_harmony_message` is documented as: Parse a message from request.messages in the Chat Completion API to Harmony messages.
**CN:** 这里定义函数 `parse_chat_input_to_harmony_message`，其文档字符串说明了主要职责与调用约定。

### Lines 360-365 — Function `render_for_completion`
```python
def render_for_completion(messages: list[Message]) -> list[int]:
    conversation = Conversation.from_messages(messages)
    token_ids = get_encoding().render_conversation_for_completion(
        conversation, Role.ASSISTANT
    )
    return token_ids
```
**EN:** This function `render_for_completion` implements the `rendering for completion` step within the module flow.
**CN:** 该函数 `render_for_completion` 实现了模块流程中的“渲染for补全”步骤。

### Lines 368-369 — Function `get_stop_tokens_for_assistant_actions`
```python
def get_stop_tokens_for_assistant_actions() -> list[int]:
    return get_encoding().stop_tokens_for_assistant_actions()
```
**EN:** This function `get_stop_tokens_for_assistant_actions` implements the `gets stop tokens for assistant actions` step within the module flow.
**CN:** 该函数 `get_stop_tokens_for_assistant_actions` 实现了模块流程中的“获取stoptokensforassistantactions”步骤。

### Lines 372-373 — Function `get_streamable_parser_for_assistant`
```python
def get_streamable_parser_for_assistant() -> StreamableParser:
    return StreamableParser(get_encoding(), role=Role.ASSISTANT)
```
**EN:** This function `get_streamable_parser_for_assistant` checks or normalizes inputs.
**CN:** 该函数 `get_streamable_parser_for_assistant` 检查或规范化输入。

### Lines 376-380 — Function `parse_output_into_messages`
```python
def parse_output_into_messages(token_ids: Iterable[int]) -> StreamableParser:
    parser = get_streamable_parser_for_assistant()
    for token_id in token_ids:
        parser.process(token_id)
    return parser
```
**EN:** This function `parse_output_into_messages` checks or normalizes inputs.
**CN:** 该函数 `parse_output_into_messages` 检查或规范化输入。

### Lines 383-435 — Function `parse_chat_output`
```python
def parse_chat_output(
    token_ids: Sequence[int],
) -> tuple[str | None, str | None, bool]:
    """
    Parse the output of a Harmony chat completion into reasoning and final content.
    Note that when the `openai` tool parser is used, serving_chat only uses this
    for the reasoning content and gets the final content from the tool call parser.

    When the `openai` tool parser is not enabled, or when `GptOssReasoningParser` is
    in use,this needs to return the final content without any tool calls parsed.

    Empty reasoning or final content is returned as None instead of an empty string.
    """
    parser = parse_output_into_messages(token_ids)
    output_msgs = parser.messages
    is_tool_call = False  # TODO: update this when tool call is supported

    # Get completed messages from the parser
    # - analysis channel: hidden reasoning
    # - commentary channel without recipient (preambles): visible to user
    # - final channel: visible to user
    # - commentary with recipient (tool calls): handled separately by tool parser
    reasoning_texts = [
        msg.content[0].text for msg in output_msgs if msg.channel == "analysis"
    ]
    final_texts = [
        msg.content[0].text
        for msg in output_msgs
...
    reasoning: str | None = "\n".join(reasoning_texts)
    final_content: str | None = "\n".join(final_texts)

    # Return None instead of empty string since existing callers check for None
    reasoning = reasoning or None
    final_content = final_content or None

    return reasoning, final_content, is_tool_call
```
**EN:** This function `parse_chat_output` is documented as: Parse the output of a Harmony chat completion into reasoning and final content.
**CN:** 这里定义函数 `parse_chat_output`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `datetime`, `collections`
- **Third-party / 第三方**: `openai`, `openai_harmony`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.logger`
