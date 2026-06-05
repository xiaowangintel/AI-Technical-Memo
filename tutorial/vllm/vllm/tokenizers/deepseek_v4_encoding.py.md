# deepseek_v4_encoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/deepseek_v4_encoding.py`
- **Repository**: vllm-project/vllm
- **Purpose**: DeepSeek-V4 Encoding A self-contained implementation for encoding/decoding DeepSeek-V4 chat messages with tool calling, thinking mode, and quick instruction task support / 该模块围绕 `deepseek_v4_encoding` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-101)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# ruff: noqa
# fmt: off

"""
DeepSeek-V4 Encoding

A self-contained implementation for encoding/decoding DeepSeek-V4 chat messages
with tool calling, thinking mode, and quick instruction task support.
"""

from typing import Any, Dict, List, Union, Optional, Tuple
import copy
import json

import regex as re

# ============================================================
# Special Tokens
# ============================================================

bos_token: str = "<｜begin▁of▁sentence｜>"
eos_token: str = "<｜end▁of▁sentence｜>"
    # ...
### Available Tool Schemas

{tool_schemas}

You MUST strictly follow the above defined tool name and parameter schemas to invoke tool calls.
"""
```
**EN:** Sets up the module with standard-library support such as `typing`, `copy`, `json`, external packages such as `regex`. It prepares the symbols later used by `to_json`, `tools_from_openai_format`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `to_json`, `tools_from_openai_format` 提供上下文。

### to_json (lines 107-112)
```python
def to_json(value: Any) -> str:
    """Serialize a value to JSON string."""
    try:
        return json.dumps(value, ensure_ascii=False)
    except Exception:
        return json.dumps(value, ensure_ascii=True)
```
**EN:** `to_json`: Serialize a value to JSON string. It mainly works with `value`. Inside the body, it relies on `json.dumps` to complete the main steps.
**CN:** `to_json` 负责实现本模块使用的辅助逻辑。 它主要处理 `value` 等参数。 实现过程中会调用 `json.dumps` 等函数完成关键步骤。

### tools_from_openai_format (lines 115-117)
```python
def tools_from_openai_format(tools):
    """Extract function definitions from OpenAI-format tool list."""
    return [tool["function"] for tool in tools]
```
**EN:** `tools_from_openai_format`: Extract function definitions from OpenAI-format tool list. It mainly works with `tools`.
**CN:** `tools_from_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tools` 等参数。

### tool_calls_from_openai_format (lines 120-128)
```python
def tool_calls_from_openai_format(tool_calls):
    """Convert OpenAI-format tool calls to internal format."""
    return [
        {
            "name": tool_call["function"]["name"],
            "arguments": tool_call["function"]["arguments"],
        }
        for tool_call in tool_calls
    ]
```
**EN:** `tool_calls_from_openai_format`: Convert OpenAI-format tool calls to internal format. It mainly works with `tool_calls`.
**CN:** `tool_calls_from_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tool_calls` 等参数。

### tool_calls_to_openai_format (lines 131-142)
```python
def tool_calls_to_openai_format(tool_calls):
    """Convert internal tool calls to OpenAI format."""
    return [
        {
            "type": "function",
            "function": {
                "name": tool_call["name"],
                "arguments": tool_call["arguments"],
            }
        }
        for tool_call in tool_calls
    ]
```
**EN:** `tool_calls_to_openai_format`: Convert internal tool calls to OpenAI format. It mainly works with `tool_calls`.
**CN:** `tool_calls_to_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tool_calls` 等参数。

### encode_arguments_to_dsml (lines 145-172)
```python
def encode_arguments_to_dsml(tool_call: Dict[str, Any]) -> str:
    """
    Encode tool call arguments into DSML parameter format.

    Args:
        tool_call: Dict with "name" and "arguments" keys.

    Returns:
        DSML-formatted parameter string.
    """
    p_dsml_template = '<{dsml_token}parameter name="{key}" string="{is_str}">{value}</{dsml_token}parameter>'
    P_dsml_strs = []

    if isinstance(tool_call["arguments"], str):
        arguments = json.loads(tool_call["arguments"])
    else:
        arguments = tool_call["arguments"]

    for k, v in arguments.items():
        p_dsml_str = p_dsml_template.format(
            dsml_token=dsml_token,
            key=k,
            is_str="true" if isinstance(v, str) else "false",
            value=v if isinstance(v, str) else to_json(v),
        )
        P_dsml_strs.append(p_dsml_str)

    return "\n".join(P_dsml_strs)
```
**EN:** `encode_arguments_to_dsml`: Encode tool call arguments into DSML parameter format. It mainly works with `tool_call`. Inside the body, it relies on `arguments.items`, `join`, `json.loads` to complete the main steps.
**CN:** `encode_arguments_to_dsml` 负责把数据转换为编码后的表示。 它主要处理 `tool_call` 等参数。 实现过程中会调用 `arguments.items`, `join`, `json.loads` 等函数完成关键步骤。

### decode_dsml_to_arguments (lines 175-192)
```python
def decode_dsml_to_arguments(tool_name: str, tool_args: Dict[str, Tuple[str, str]]) -> Dict[str, str]:
    """
    Decode DSML parameters back to a tool call dict.

    Args:
        tool_name: Name of the tool.
        tool_args: Dict mapping param_name -> (value, is_string_flag).

    Returns:
        Dict with "name" and "arguments" (JSON string) keys.
    """
    def _decode_value(key: str, value: str, string: str):
        if string == "true":
            value = to_json(value)
        return f"{to_json(key)}: {value}"

    tool_args_json = "{" + ", ".join([_decode_value(k, v, string=is_str) for k, (v, is_str) in tool_args.items()]) + "}"
    return dict(name=tool_name, arguments=tool_args_json)
```
**EN:** `decode_dsml_to_arguments`: Decode DSML parameters back to a tool call dict. It mainly works with `tool_name`, `tool_args`. Inside the body, it relies on `to_json`, `join`, `_decode_value` to complete the main steps.
**CN:** `decode_dsml_to_arguments` 负责把编码数据还原为可用表示。 它主要处理 `tool_name`, `tool_args` 等参数。 实现过程中会调用 `to_json`, `join`, `_decode_value` 等函数完成关键步骤。

### render_tools (lines 195-212)
```python
def render_tools(tools: List[Dict[str, Union[str, Dict[str, Any]]]]) -> str:
    """
    Render tool schemas into the system prompt format.

    Args:
        tools: List of tool schema dicts (each with name, description, parameters).

    Returns:
        Formatted tools section string.
    """
    tools_json = [to_json(t) for t in tools]

    return TOOLS_TEMPLATE.format(
        tool_schemas="\n".join(tools_json),
        dsml_token=dsml_token,
        thinking_start_token=thinking_start_token,
        thinking_end_token=thinking_end_token,
    )
```
**EN:** `render_tools`: Render tool schemas into the system prompt format. It mainly works with `tools`. Inside the body, it relies on `TOOLS_TEMPLATE.format`, `to_json`, `join` to complete the main steps.
**CN:** `render_tools` 负责实现本模块使用的辅助逻辑。 它主要处理 `tools` 等参数。 实现过程中会调用 `TOOLS_TEMPLATE.format`, `to_json`, `join` 等函数完成关键步骤。

### find_last_user_index (lines 215-222)
```python
def find_last_user_index(messages: List[Dict[str, Any]]) -> int:
    """Find the index of the last user/developer message."""
    last_user_index = -1
    for idx in range(len(messages) - 1, -1, -1):
        if messages[idx].get("role") in ["user", "developer"]:
            last_user_index = idx
            break
    return last_user_index
```
**EN:** `find_last_user_index`: Find the index of the last user/developer message. It mainly works with `messages`. Inside the body, it relies on `messages.get` to complete the main steps.
**CN:** `find_last_user_index` 负责实现本模块使用的辅助逻辑。 它主要处理 `messages` 等参数。 实现过程中会调用 `messages.get` 等函数完成关键步骤。

### render_message (lines 229-400)
```python
def render_message(index: int, messages: List[Dict[str, Any]], thinking_mode: str, drop_thinking: bool = True, reasoning_effort: Optional[str] = None) -> str:
    """
    Render a single message at the given index into its encoded string form.

    This is the core function that converts each message in the conversation
    into the DeepSeek-V4 format.

    Args:
        index: Index of the message to render.
        messages: Full list of messages in the conversation.
        thinking_mode: Either "chat" or "thinking".
        drop_thinking: Whether to drop reasoning content from earlier turns.
        reasoning_effort: Optional reasoning effort level ("max", "high", or None).

    Returns:
        Encoded string for this message.
    """
    assert 0 <= index < len(messages)
    assert thinking_mode in ["chat", "thinking"], f"Invalid thinking_mode `{thinking_mode}`"

    prompt = ""
    msg = messages[index]
    last_user_idx = find_last_user_index(messages)

    # ...
        elif drop_thinking and thinking_mode == "thinking" and index >= last_user_idx:
            prompt += thinking_start_token
        else:
            prompt += thinking_end_token

    return prompt
```
**EN:** `render_message`: Render a single message at the given index into its encoded string form. It mainly works with `index`, `messages`, `thinking_mode`, `drop_thinking`. Inside the body, it relies on `find_last_user_index`, `msg.get`, `messages.get` to complete the main steps.
**CN:** `render_message` 负责实现本模块使用的辅助逻辑。 它主要处理 `index`, `messages`, `thinking_mode`, `drop_thinking` 等参数。 实现过程中会调用 `find_last_user_index`, `msg.get`, `messages.get` 等函数完成关键步骤。

### merge_tool_messages (lines 407-463)
```python
def merge_tool_messages(messages: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """
    Merge tool messages into the preceding user message using content_blocks format.

    DeepSeek-V4 does not have a standalone "tool" role; instead, tool results
    are encoded as <tool_result> blocks within user messages.

    This function converts a standard OpenAI-format conversation (with separate
    "tool" role messages) into V4 format where tool results are merged into
    user messages.

    Args:
        messages: List of message dicts in OpenAI format.

    Returns:
        Processed message list with tool messages merged into user messages.
    """
    merged: List[Dict[str, Any]] = []

    for msg in messages:
        msg = copy.deepcopy(msg)
        role = msg.get("role")

        if role == "tool":
    # ...
                        new_msg[key] = msg[key]
                merged.append(new_msg)
        else:
            merged.append(msg)

    return merged
```
**EN:** `merge_tool_messages`: Merge tool messages into the preceding user message using content_blocks format. It mainly works with `messages`. Inside the body, it relies on `copy.deepcopy`, `msg.get`, `merged.append` to complete the main steps.
**CN:** `merge_tool_messages` 负责合并相关配置或数据并生成规范化结果。 它主要处理 `messages` 等参数。 实现过程中会调用 `copy.deepcopy`, `msg.get`, `merged.append` 等函数完成关键步骤。

### sort_tool_results_by_call_order (lines 466-505)
```python
def sort_tool_results_by_call_order(messages: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """
    Sort tool_result blocks within user messages by the order of tool_calls
    in the preceding assistant message.

    Args:
        messages: Preprocessed message list (after merge_tool_messages).

    Returns:
        Message list with sorted tool result blocks.
    """
    last_tool_call_order: Dict[str, int] = {}

    for msg in messages:
        role = msg.get("role")
        if role == "assistant" and msg.get("tool_calls"):
            last_tool_call_order = {}
            for idx, tc in enumerate(msg["tool_calls"]):
                tc_id = tc.get("id") or tc.get("function", {}).get("id", "")
                if tc_id:
                    last_tool_call_order[tc_id] = idx

        elif role == "user" and msg.get("content_blocks"):
            tool_blocks = [b for b in msg["content_blocks"] if b.get("type") == "tool_result"]
    # ...
                        sorted_idx += 1
                    else:
                        new_blocks.append(block)
                msg["content_blocks"] = new_blocks

    return messages
```
**EN:** `sort_tool_results_by_call_order`: Sort tool_result blocks within user messages by the order of tool_calls in the preceding assistant message. It mainly works with `messages`. Inside the body, it relies on `msg.get`, `tc.get`, `tc.get.get` to complete the main steps.
**CN:** `sort_tool_results_by_call_order` 负责实现本模块使用的辅助逻辑。 它主要处理 `messages` 等参数。 实现过程中会调用 `msg.get`, `tc.get`, `tc.get.get` 等函数完成关键步骤。

### encode_messages (lines 512-578)
```python
def encode_messages(
    messages: List[Dict[str, Any]],
    thinking_mode: str,
    context: Optional[List[Dict[str, Any]]] = None,
    drop_thinking: bool = True,
    add_default_bos_token: bool = True,
    reasoning_effort: Optional[str] = None,
) -> str:
    """
    Encode a list of messages into the DeepSeek-V4 prompt format.

    This is the main entry point for encoding conversations. It handles:
    - BOS token insertion
    - Thinking mode with optional reasoning content dropping
    - Tool message merging into user messages
    - Multi-turn conversation context

    Args:
        messages: List of message dicts to encode.
        thinking_mode: Either "chat" or "thinking".
        context: Optional preceding context messages (already encoded prefix).
        drop_thinking: If True, drop reasoning from earlier assistant turns
                      (only keep reasoning for messages after the last user message).
        add_default_bos_token: Whether to prepend BOS token at conversation start.
    # ...
            thinking_mode=thinking_mode,
            drop_thinking=effective_drop_thinking,
            reasoning_effort=reasoning_effort,
        )

    return prompt
```
**EN:** `encode_messages`: Encode a list of messages into the DeepSeek-V4 prompt format. It mainly works with `messages`, `thinking_mode`, `context`, `drop_thinking`. Inside the body, it relies on `merge_tool_messages`, `any`, `sort_tool_results_by_call_order` to complete the main steps.
**CN:** `encode_messages` 负责把数据转换为编码后的表示。 它主要处理 `messages`, `thinking_mode`, `context`, `drop_thinking` 等参数。 实现过程中会调用 `merge_tool_messages`, `any`, `sort_tool_results_by_call_order` 等函数完成关键步骤。

### _drop_thinking_messages (lines 581-605)
```python
def _drop_thinking_messages(messages: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
    """
    Drop reasoning and non-essential messages before the last user message.

    Behavior:
    - Messages with role in ["user", "system", "tool", "latest_reminder"] are always kept.
    - Messages at or after the last user index are always kept.
    - Assistant messages before the last user get reasoning removed.
    - Developer messages before the last user are dropped entirely.
    """
    last_user_idx = find_last_user_index(messages)
    result = []
    keep_roles = {"user", "system", "tool", "latest_reminder", "direct_search_results"}

    for idx, msg in enumerate(messages):
        role = msg.get("role")
        if role in keep_roles or idx >= last_user_idx:
            result.append(msg)
        elif role == "assistant":
            msg = copy.copy(msg)
            msg.pop("reasoning", None)
            result.append(msg)
        # developer and other roles before last_user_idx are dropped

    return result
```
**EN:** `_drop_thinking_messages`: Drop reasoning and non-essential messages before the last user message. It mainly works with `messages`. Inside the body, it relies on `find_last_user_index`, `msg.get`, `result.append` to complete the main steps.
**CN:** `_drop_thinking_messages` 负责实现本模块使用的辅助逻辑。 它主要处理 `messages` 等参数。 实现过程中会调用 `find_last_user_index`, `msg.get`, `result.append` 等函数完成关键步骤。

### _read_until_stop (lines 612-633)
```python
def _read_until_stop(index: int, text: str, stop: List[str]) -> Tuple[int, str, Optional[str]]:
    """
    Read text from index until one of the stop strings is found.

    Returns:
        Tuple of (new_index, content_before_stop, matched_stop_string_or_None).
    """
    min_pos = len(text)
    matched_stop = None

    for s in stop:
        pos = text.find(s, index)
        if pos != -1 and pos < min_pos:
            min_pos = pos
            matched_stop = s

    if matched_stop:
        content = text[index:min_pos]
        return min_pos + len(matched_stop), content, matched_stop
    else:
        content = text[index:]
        return len(text), content, None
```
**EN:** `_read_until_stop`: Read text from index until one of the stop strings is found. It mainly works with `index`, `text`, `stop`. Inside the body, it relies on `text.find` to complete the main steps.
**CN:** `_read_until_stop` 负责从输入源读取并规范化数据。 它主要处理 `index`, `text`, `stop` 等参数。 实现过程中会调用 `text.find` 等函数完成关键步骤。

### parse_tool_calls (lines 636-690)
```python
def parse_tool_calls(index: int, text: str) -> Tuple[int, Optional[str], List[Dict[str, str]]]:
    """
    Parse DSML tool calls from text starting at the given index.

    Args:
        index: Starting position in text.
        text: The full text to parse.

    Returns:
        Tuple of (new_index, last_stop_token, list_of_tool_call_dicts).
        Each tool call dict has "name" and "arguments" keys.
    """
    tool_calls: List[Dict[str, Any]] = []
    stop_token = None
    tool_calls_end_token = f"</{dsml_token}{tool_calls_block_name}>"

    while index < len(text):
        index, content_before, stop_token = _read_until_stop(index, text, [f"<{dsml_token}invoke", tool_calls_end_token])
        if content_before != ">\n":
            raise ValueError(f"Tool call format error: expected '>\\n' but got '{content_before}'")

        if stop_token == tool_calls_end_token:
            break

    # ...
                raise ValueError(f"Parameter format error: expected '>\\n' but got '{content}'")

        tool_call = decode_dsml_to_arguments(tool_name=tool_name, tool_args=tool_args)
        tool_calls.append(tool_call)

    return index, stop_token, tool_calls
```
**EN:** `parse_tool_calls`: Parse DSML tool calls from text starting at the given index. It mainly works with `index`, `text`. Inside the body, it relies on `_read_until_stop`, `re.findall`, `decode_dsml_to_arguments` to complete the main steps.
**CN:** `parse_tool_calls` 负责把原始输入解析为结构化对象。 它主要处理 `index`, `text` 等参数。 实现过程中会调用 `_read_until_stop`, `re.findall`, `decode_dsml_to_arguments` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`to_json`**: Key helper or entry point in this file. / **`to_json`**：本文件中的关键辅助函数或入口。
- **`tools_from_openai_format`**: Key helper or entry point in this file. / **`tools_from_openai_format`**：本文件中的关键辅助函数或入口。
- **`tool_calls_from_openai_format`**: Key helper or entry point in this file. / **`tool_calls_from_openai_format`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: typing, copy, json
- **Third-party / 第三方**: regex
- **Internal vLLM / vLLM 内部依赖**: None / 无
