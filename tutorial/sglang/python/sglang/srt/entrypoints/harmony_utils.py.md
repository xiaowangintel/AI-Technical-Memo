# harmony_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/harmony_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements harmony utils logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 harmony 工具 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7: Import runtime dependencies / 导入运行时依赖
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from vLLM: https://github.com/vllm-project/vllm/blob/1b9902806915040ac9b3029f2ab7522ec505afc3/vllm/entrypoints/harmony_utils.py
# Slight differences in processing chat messages
import datetime
from collections.abc import Iterable
from typing import Literal, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-40: Provide supporting module logic / 提供辅助模块逻辑
```python
import orjson
from openai.types.responses import (
    ResponseOutputItem,
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseReasoningItem,
)
from openai.types.responses.response_function_tool_call import ResponseFunctionToolCall
from openai.types.responses.response_function_web_search import (
    ActionFind,
    ActionOpenPage,
    ActionSearch,
    ResponseFunctionWebSearch,
)
from openai.types.responses.response_reasoning_item import (
    Content as ResponseReasoningTextContent,
)
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
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 42-43: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.entrypoints.openai.protocol import ResponseInputOutputItem
from sglang.srt.utils import random_uuid
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 45-49: Provide supporting module logic / 提供辅助模块逻辑
```python
REASONING_EFFORT = {
    "high": ReasoningEffort.HIGH,
    "medium": ReasoningEffort.MEDIUM,
    "low": ReasoningEffort.LOW,
}
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 51-51: Provide supporting module logic / 提供辅助模块逻辑
```python
_harmony_encoding = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 54-58: Implement get encoding / 实现get encoding
```python
def get_encoding():
    global _harmony_encoding
    if _harmony_encoding is None:
        _harmony_encoding = load_harmony_encoding(HarmonyEncodingName.HARMONY_GPT_OSS)
    return _harmony_encoding
```
**EN:** This block implements the function `get_encoding()`. It focuses on handling the harmony utils responsibilities represented by `get_encoding`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_encoding()`。它围绕 `get_encoding` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 61-83: Implement get system message / 实现get system message
```python
def get_system_message(
    model_identity: Optional[str] = None,
    reasoning_effort: Optional[Literal["high", "medium", "low"]] = None,
    start_date: Optional[str] = None,
    browser_description: Optional[str] = None,
    python_description: Optional[str] = None,
) -> Message:
    sys_msg_content = SystemContent.new()
    if model_identity is not None:
        sys_msg_content = sys_msg_content.with_model_identity(model_identity)
    if reasoning_effort is not None:
        sys_msg_content = sys_msg_content.with_reasoning_effort(
            REASONING_EFFORT[reasoning_effort]
        )
    if start_date is None:
        start_date = datetime.datetime.now().strftime("%Y-%m-%d")
    sys_msg_content = sys_msg_content.with_conversation_start_date(start_date)
    if browser_description is not None:
        sys_msg_content = sys_msg_content.with_tools(browser_description)
    if python_description is not None:
        sys_msg_content = sys_msg_content.with_tools(python_description)
    sys_msg = Message.from_role_and_content(Role.SYSTEM, sys_msg_content)
    return sys_msg
```
**EN:** This block implements the function `get_system_message(model_identity, reasoning_effort, start_date, browser_description, python_description)`. It focuses on handling the harmony utils responsibilities represented by `get_system_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_system_message(model_identity, reasoning_effort, start_date, browser_description, python_description)`。它围绕 `get_system_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 86-115: Implement get developer message / 实现get developer message
```python
def get_developer_message(
    instructions: Optional[str] = None, tools: Optional[list[Tool]] = None
) -> Message:
    dev_msg_content = DeveloperContent.new()
    if instructions is not None:
        dev_msg_content = dev_msg_content.with_instructions(instructions)
    if tools is not None:
        function_tools = []
        for tool in tools:
            if tool.type in ("web_search_preview", "code_interpreter"):
                # These are built-in tools that are added to the system message.
                pass
            elif tool.type == "function":
                function_tools.append(tool)
            else:
                raise ValueError(f"tool type {tool.type} not supported")
        if function_tools:
            function_tool_descriptions = [
                ToolDescription.new(
                    name=tool.name,
                    description=tool.description,
                    parameters=tool.parameters,
                )
                for tool in function_tools
            ]
            dev_msg_content = dev_msg_content.with_function_tools(
                function_tool_descriptions
            )
    dev_msg = Message.from_role_and_content(Role.DEVELOPER, dev_msg_content)
    return dev_msg
```
**EN:** This block implements the function `get_developer_message(instructions, tools)`. It focuses on handling the harmony utils responsibilities represented by `get_developer_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_developer_message(instructions, tools)`。它围绕 `get_developer_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 118-119: Implement get user message / 实现get user message
```python
def get_user_message(content: str) -> Message:
    return Message.from_role_and_content(Role.USER, content)
```
**EN:** This block implements the function `get_user_message(content)`. It focuses on handling the harmony utils responsibilities represented by `get_user_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_user_message(content)`。它围绕 `get_user_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 122-157: Implement parse response input / 实现parse 响应 input
```python
def parse_response_input(
    response_msg: ResponseInputOutputItem,
    prev_responses: list[Union[ResponseOutputItem, ResponseReasoningItem]],
) -> Message:
    if not isinstance(response_msg, dict):
        response_msg = response_msg.model_dump()
    if "type" not in response_msg or response_msg["type"] == "message":
        role = response_msg["role"]
        content = response_msg["content"]
        if role == "system":
            # User is trying to set a system message. Change it to:
            # <|start|>developer<|message|># Instructions
            # {instructions}<|end|>
            role = "developer"
            text_prefix = "Instructions:\n"
        else:
            text_prefix = ""
        if isinstance(content, str):
            msg = Message.from_role_and_content(role, text_prefix + content)
        else:
            contents = [TextContent(text=text_prefix + c["text"]) for c in content]
            msg = Message.from_role_and_contents(role, contents)
    elif response_msg["type"] == "function_call_output":
        call_id = response_msg["call_id"]
        call_response: Optional[ResponseFunctionToolCall] = None
        for prev_response in reversed(prev_responses):
            if (
                isinstance(prev_response, ResponseFunctionToolCall)
                and prev_response.call_id == call_id
            ):
                call_response = prev_response
                break
        if call_response is None:
            raise ValueError(f"No call message found for {call_id}")
        msg = Message.from_author_and_content(
            Author.new(Role.TOOL, f"functions.{call_response.name}"),
```
**EN:** This block implements the function `parse_response_input(response_msg, prev_responses)`. It focuses on handling the harmony utils responsibilities represented by `parse_response_input`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_response_input(response_msg, prev_responses)`。它围绕 `parse_response_input` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 158-171: Continue parse response input / 继续说明parse 响应 input
```python
            response_msg["output"],
        )
    elif response_msg["type"] == "reasoning":
        content = response_msg["content"]
        assert len(content) == 1
        msg = Message.from_role_and_content(Role.ASSISTANT, content[0]["text"])
    elif response_msg["type"] == "function_call":
        msg = Message.from_role_and_content(Role.ASSISTANT, response_msg["arguments"])
        msg = msg.with_channel("commentary")
        msg = msg.with_recipient(f"functions.{response_msg['name']}")
        msg = msg.with_content_type("json")
    else:
        raise ValueError(f"Unknown input type: {response_msg['type']}")
    return msg
```
**EN:** This block implements the function `parse_response_input(response_msg, prev_responses)`. It focuses on handling the harmony utils responsibilities represented by `parse_response_input`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_response_input(response_msg, prev_responses)`。它围绕 `parse_response_input` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 174-187: Implement parse response output / 实现parse 响应 output
```python
def parse_response_output(output: ResponseOutputItem) -> Message:
    if isinstance(output, ResponseOutputMessage):
        role = output.role
        contents = [TextContent(text=c.text) for c in output.content]
        msg = Message.from_role_and_contents(role, contents)
        return msg
    elif isinstance(output, ResponseFunctionToolCall):
        msg = Message.from_role_and_content(Role.ASSISTANT, output.arguments)
        msg = msg.with_channel("commentary")
        msg = msg.with_recipient(output.name)
        msg = msg.with_content_type("json")
        return msg
    else:
        raise ValueError(f"Unknown output type: {type(output)}")
```
**EN:** This block implements the function `parse_response_output(output)`. It focuses on handling the harmony utils responsibilities represented by `parse_response_output`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_response_output(output)`。它围绕 `parse_response_output` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 190-199: Implement parse chat input / 实现parse 聊天 input
```python
def parse_chat_input(chat_msg) -> Message:
    role = chat_msg.role
    content = chat_msg.content
    if isinstance(content, str):
        contents = [TextContent(text=content)]
    else:
        # TODO: Support refusal.
        contents = [TextContent(text=c.text) for c in content]
    msg = Message.from_role_and_contents(role, contents)
    return msg
```
**EN:** This block implements the function `parse_chat_input(chat_msg)`. It focuses on handling the harmony utils responsibilities represented by `parse_chat_input`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_chat_input(chat_msg)`。它围绕 `parse_chat_input` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 202-207: Implement render for completion / 实现render for 补全
```python
def render_for_completion(messages: list[Message]) -> list[int]:
    conversation = Conversation.from_messages(messages)
    token_ids = get_encoding().render_conversation_for_completion(
        conversation, Role.ASSISTANT
    )
    return token_ids
```
**EN:** This block implements the function `render_for_completion(messages)`. It focuses on handling the harmony utils responsibilities represented by `render_for_completion`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `render_for_completion(messages)`。它围绕 `render_for_completion` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 210-211: Implement get stop tokens for assistant actions / 实现get stop tokens for assistant actions
```python
def get_stop_tokens_for_assistant_actions() -> list[int]:
    return get_encoding().stop_tokens_for_assistant_actions()
```
**EN:** This block implements the function `get_stop_tokens_for_assistant_actions()`. It focuses on handling the harmony utils responsibilities represented by `get_stop_tokens_for_assistant_actions`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_stop_tokens_for_assistant_actions()`。它围绕 `get_stop_tokens_for_assistant_actions` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 214-215: Implement get streamable parser for assistant / 实现get streamable parser for assistant
```python
def get_streamable_parser_for_assistant() -> StreamableParser:
    return StreamableParser(get_encoding(), role=Role.ASSISTANT)
```
**EN:** This block implements the function `get_streamable_parser_for_assistant()`. It focuses on handling the harmony utils responsibilities represented by `get_streamable_parser_for_assistant`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `get_streamable_parser_for_assistant()`。它围绕 `get_streamable_parser_for_assistant` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 218-253: Implement parse output message / 实现parse output message
```python
def parse_output_message(message: Message):
    if message.author.role != "assistant":
        # This is a message from a tool to the assistant (e.g., search result).
        # Don't include it in the final output for now. This aligns with
        # OpenAI's behavior on models like o4-mini.
        return []

    output_items = []
    recipient = message.recipient
    if recipient is not None and recipient.startswith("browser."):
        if len(message.content) != 1:
            raise ValueError("Invalid number of contents in browser message")
        content = message.content[0]
        browser_call = orjson.loads(content.text)
        # TODO: translate to url properly!
        if recipient == "browser.search":
            action = ActionSearch(
                query=f"cursor:{browser_call.get('query', '')}", type="search"
            )
        elif recipient == "browser.open":
            action = ActionOpenPage(
                url=f"cursor:{browser_call.get('url', '')}", type="open_page"
            )
        elif recipient == "browser.find":
            action = ActionFind(
                pattern=browser_call["pattern"],
                url=f"cursor:{browser_call.get('url', '')}",
                type="find",
            )
        else:
            raise ValueError(f"Unknown browser action: {recipient}")
        web_search_item = ResponseFunctionWebSearch(
            id=f"ws_{random_uuid()}",
            action=action,
            status="completed",
            type="web_search_call",
```
**EN:** This block implements the function `parse_output_message(message)`. It focuses on handling the harmony utils responsibilities represented by `parse_output_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_output_message(message)`。它围绕 `parse_output_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 254-289: Continue parse output message / 继续说明parse output message
```python
        )
        output_items.append(web_search_item)
    elif message.channel == "analysis":
        for content in message.content:
            reasoning_item = ResponseReasoningItem(
                id=f"rs_{random_uuid()}",
                type="reasoning",
                summary=[],
                content=[
                    ResponseReasoningTextContent(
                        text=content.text, type="reasoning_text"
                    )
                ],
                status=None,
            )
            output_items.append(reasoning_item)
    elif message.channel == "commentary":
        if message.recipient.startswith("functions."):
            function_name = message.recipient.split(".")[-1]
            for content in message.content:
                random_id = random_uuid()
                response_item = ResponseFunctionToolCall(
                    arguments=content.text,
                    call_id=f"call_{random_id}",
                    type="function_call",
                    name=function_name,
                    id=f"ft_{random_id}",
                )
                output_items.append(response_item)
        elif message.recipient.startswith("python") or message.recipient.startswith(
            "browser"
        ):
            for content in message.content:
                reasoning_item = ResponseReasoningItem(
                    id=f"rs_{random_uuid()}",
                    type="reasoning",
```
**EN:** This block implements the function `parse_output_message(message)`. It focuses on handling the harmony utils responsibilities represented by `parse_output_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_output_message(message)`。它围绕 `parse_output_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 290-321: Continue parse output message / 继续说明parse output message
```python
                    summary=[],
                    content=[
                        ResponseReasoningTextContent(
                            text=content.text, type="reasoning_text"
                        )
                    ],
                    status=None,
                )
                output_items.append(reasoning_item)
        else:
            raise ValueError(f"Unknown recipient: {message.recipient}")
    elif message.channel == "final":
        contents = []
        for content in message.content:
            output_text = ResponseOutputText(
                text=content.text,
                annotations=[],  # TODO
                type="output_text",
                logprobs=None,  # TODO
            )
            contents.append(output_text)
        text_item = ResponseOutputMessage(
            id=f"msg_{random_uuid()}",
            content=contents,
            role=message.author.role,
            status="completed",
            type="message",
        )
        output_items.append(text_item)
    else:
        raise ValueError(f"Unknown channel: {message.channel}")
    return output_items
```
**EN:** This block implements the function `parse_output_message(message)`. It focuses on handling the harmony utils responsibilities represented by `parse_output_message`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_output_message(message)`。它围绕 `parse_output_message` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 324-361: Implement parse remaining state / 实现parse remaining 状态
```python
def parse_remaining_state(parser: StreamableParser):
    if not parser.current_content:
        return []
    if parser.current_role != Role.ASSISTANT:
        return []
    current_recipient = parser.current_recipient
    if current_recipient is not None and current_recipient.startswith("browser."):
        return []

    if parser.current_channel == "analysis":
        reasoning_item = ResponseReasoningItem(
            id=f"rs_{random_uuid()}",
            type="reasoning",
            summary=[],
            content=[
                ResponseReasoningTextContent(
                    text=parser.current_content, type="reasoning_text"
                )
            ],
            status=None,
        )
        return [reasoning_item]
    elif parser.current_channel == "final":
        output_text = ResponseOutputText(
            text=parser.current_content,
            annotations=[],  # TODO
            type="output_text",
            logprobs=None,  # TODO
        )
        text_item = ResponseOutputMessage(
            id=f"msg_{random_uuid()}",
            content=[output_text],
            role="assistant",
            status="completed",
            type="message",
        )
        return [text_item]
    return []
```
**EN:** This block implements the function `parse_remaining_state(parser)`. It focuses on handling the harmony utils responsibilities represented by `parse_remaining_state`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_remaining_state(parser)`。它围绕 `parse_remaining_state` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 364-368: Implement parse output into messages / 实现parse output into messages
```python
def parse_output_into_messages(token_ids: Iterable[int]):
    parser = get_streamable_parser_for_assistant()
    for token_id in token_ids:
        parser.process(token_id)
    return parser
```
**EN:** This block implements the function `parse_output_into_messages(token_ids)`. It focuses on handling the harmony utils responsibilities represented by `parse_output_into_messages`, providing reusable behavior for the harmony utils pipeline.
**CN:** 该代码块实现函数 `parse_output_into_messages(token_ids)`。它围绕 `parse_output_into_messages` 所承担的 harmony 工具 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Main callables / 主要可调用对象**: get_encoding, get_system_message, get_developer_message, get_user_message, parse_response_input, parse_response_output, parse_chat_input, render_for_completion, ...
- **Domain focus / 领域焦点**: harmony utils / harmony 工具
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: collections.abc, typing
- **Third-party / 第三方库**: datetime, openai.types.responses, openai.types.responses.response_function_tool_call, openai.types.responses.response_function_web_search, openai.types.responses.response_reasoning_item, openai.types.responses.tool, openai_harmony, orjson
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.utils
