# deepseek_v32_encoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tokenizers/deepseek_v32_encoding.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `deepseek_v32_encoding`-related logic centered around `to_json`, `tools_from_openai_format`. / 实现与 `deepseek_v32_encoding` 相关的逻辑，核心符号包括 `to_json`, `tools_from_openai_format`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-58)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project


# copy from https://huggingface.co/deepseek-ai/DeepSeek-V3.2/blob/main/encoding/encoding_dsv32.py
import copy
import json
from typing import Any

import regex as re

# flake8: noqa: E501
TOOLS_SYSTEM_TEMPLATE = """## Tools
You have access to a set of tools you can use to answer the user's question.
You can invoke functions by writing a "<{dsml_token}function_calls>" block like the following as part of your reply to the user:
<{dsml_token}function_calls>
<{dsml_token}invoke name="$FUNCTION_NAME">
<{dsml_token}parameter name="$PARAMETER_NAME" string="true|false">$PARAMETER_VALUE</{dsml_token}parameter>
...
</{dsml_token}invoke>
<{dsml_token}invoke name="$FUNCTION_NAME2">
...
</{dsml_token}invoke>
</{dsml_token}function_calls>
    # ...
)
tool_calls_template = (
    "<{dsml_token}function_calls>\n{tool_calls}\n</{dsml_token}function_calls>"
)

tool_output_template: str = "\n<result>{content}</result>"
```
**EN:** Sets up the module with standard-library support such as `copy`, `json`, `typing`, external packages such as `regex`. It prepares the symbols later used by `to_json`, `tools_from_openai_format`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `to_json`, `tools_from_openai_format` 提供上下文。

### to_json (lines 61-65)
```python
def to_json(value: Any) -> str:
    try:
        return json.dumps(value, ensure_ascii=False)
    except Exception:
        return json.dumps(value, ensure_ascii=True)
```
**EN:** `to_json` implements helper logic used by this module. It mainly works with `value`. Inside the body, it relies on `json.dumps` to complete the main steps.
**CN:** `to_json` 负责实现本模块使用的辅助逻辑。 它主要处理 `value` 等参数。 实现过程中会调用 `json.dumps` 等函数完成关键步骤。

### tools_from_openai_format (lines 68-69)
```python
def tools_from_openai_format(tools):
    return [tool["function"] for tool in tools]
```
**EN:** `tools_from_openai_format` implements helper logic used by this module. It mainly works with `tools`.
**CN:** `tools_from_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tools` 等参数。

### tool_calls_from_openai_format (lines 72-79)
```python
def tool_calls_from_openai_format(tool_calls):
    return [
        {
            "name": tool_call["function"]["name"],
            "arguments": tool_call["function"]["arguments"],
        }
        for tool_call in tool_calls
    ]
```
**EN:** `tool_calls_from_openai_format` implements helper logic used by this module. It mainly works with `tool_calls`.
**CN:** `tool_calls_from_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tool_calls` 等参数。

### tool_calls_to_openai_format (lines 82-92)
```python
def tool_calls_to_openai_format(tool_calls):
    return [
        {
            "type": "function",
            "function": {
                "name": tool_call["name"],
                "arguments": tool_call["arguments"],
            },
        }
        for tool_call in tool_calls
    ]
```
**EN:** `tool_calls_to_openai_format` implements helper logic used by this module. It mainly works with `tool_calls`.
**CN:** `tool_calls_to_openai_format` 负责实现本模块使用的辅助逻辑。 它主要处理 `tool_calls` 等参数。

### encode_arguments_to_dsml (lines 95-113)
```python
def encode_arguments_to_dsml(tool_call: dict[str, str]) -> str:
    p_dsml_template = """<{dsml_token}parameter name="{key}" string="{is_str}">{value}</{dsml_token}parameter>"""
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
**EN:** `encode_arguments_to_dsml` converts data into an encoded representation. It mainly works with `tool_call`. Inside the body, it relies on `arguments.items`, `join`, `json.loads` to complete the main steps.
**CN:** `encode_arguments_to_dsml` 负责把数据转换为编码后的表示。 它主要处理 `tool_call` 等参数。 实现过程中会调用 `arguments.items`, `join`, `json.loads` 等函数完成关键步骤。

### decode_dsml_to_arguments (lines 116-131)
```python
def decode_dsml_to_arguments(
    tool_name: str, tool_args: dict[str, tuple[str, str]]
) -> dict[str, str]:
    def _decode_value(key: str, value: str, string: str):
        if string == "true":
            value = to_json(value)
        return f"{to_json(key)}: {value}"

    tool_args_json = (
        "{"
        + ", ".join(
            [_decode_value(k, v, string=is_str) for k, (v, is_str) in tool_args.items()]
        )
        + "}"
    )
    return dict(name=tool_name, arguments=tool_args_json)
```
**EN:** `decode_dsml_to_arguments` converts encoded data back into a usable representation. It mainly works with `tool_name`, `tool_args`. Inside the body, it relies on `to_json`, `join`, `_decode_value` to complete the main steps.
**CN:** `decode_dsml_to_arguments` 负责把编码数据还原为可用表示。 它主要处理 `tool_name`, `tool_args` 等参数。 实现过程中会调用 `to_json`, `join`, `_decode_value` 等函数完成关键步骤。

### render_tools (lines 134-142)
```python
def render_tools(tools: list[dict[str, str | dict[str, Any]]]) -> str:
    tools_json = [to_json(t) for t in tools]

    return TOOLS_SYSTEM_TEMPLATE.format(
        tool_schemas="\n".join(tools_json),
        dsml_token=dsml_token,
        thinking_start_token=thinking_start_token,
        thinking_end_token=thinking_end_token,
    )
```
**EN:** `render_tools` implements helper logic used by this module. It mainly works with `tools`. Inside the body, it relies on `TOOLS_SYSTEM_TEMPLATE.format`, `to_json`, `join` to complete the main steps.
**CN:** `render_tools` 负责实现本模块使用的辅助逻辑。 它主要处理 `tools` 等参数。 实现过程中会调用 `TOOLS_SYSTEM_TEMPLATE.format`, `to_json`, `join` 等函数完成关键步骤。

### find_last_user_index (lines 145-151)
```python
def find_last_user_index(messages: list[dict[str, Any]]) -> int:
    last_user_index = -1
    for idx in range(len(messages) - 1, -1, -1):
        if messages[idx].get("role") in ["user", "developer"]:
            last_user_index = idx
            break
    return last_user_index
```
**EN:** `find_last_user_index` implements helper logic used by this module. It mainly works with `messages`. Inside the body, it relies on `messages.get` to complete the main steps.
**CN:** `find_last_user_index` 负责实现本模块使用的辅助逻辑。 它主要处理 `messages` 等参数。 实现过程中会调用 `messages.get` 等函数完成关键步骤。

### render_message (lines 154-291)
```python
def render_message(
    index: int, messages: list[dict[str, Any]], thinking_mode: str
) -> str:
    if not (0 <= index < len(messages)):
        raise ValueError(
            f"Index {index} out of range for messages list of length {len(messages)}"
        )
    if thinking_mode not in ["chat", "thinking"]:
        raise ValueError(f"Invalid thinking_mode `{thinking_mode}`")

    prompt = ""
    msg = messages[index]
    last_user_idx = find_last_user_index(messages)

    role = msg.get("role")
    content = msg.get("content")
    tools = msg.get("tools")
    response_format = msg.get("response_format")
    tool_calls = msg.get("tool_calls")
    reasoning = msg.get("reasoning")
    is_prefix = msg.get("prefix", False)

    if tools:
        tools = tools_from_openai_format(tools)
    # ...
                tool_calls=tool_calls_content,
            )
    else:
        raise NotImplementedError(f"Unknown role: {role}")

    return prompt
```
**EN:** `render_message` implements helper logic used by this module. It mainly works with `index`, `messages`, `thinking_mode`. Inside the body, it relies on `find_last_user_index`, `msg.get`, `ValueError` to complete the main steps.
**CN:** `render_message` 负责实现本模块使用的辅助逻辑。 它主要处理 `index`, `messages`, `thinking_mode` 等参数。 实现过程中会调用 `find_last_user_index`, `msg.get`, `ValueError` 等函数完成关键步骤。

### drop_thinking_messages (lines 294-312)
```python
def drop_thinking_messages(
    messages: list[dict[str, Any]], last_user_idx: int | None = None
) -> list[dict[str, Any]]:
    messages_wo_thinking: list[dict[str, Any]] = []
    last_user_idx = (
        find_last_user_index(messages) if last_user_idx is None else last_user_idx
    )
    for idx, msg in enumerate(messages):
        role = msg.get("role")
        if role in ["user", "system", "tool"] or idx >= last_user_idx:
            messages_wo_thinking.append(msg)
            continue

        elif role == "assistant":
            msg_wo_thinking = copy.copy(msg)
            msg_wo_thinking.pop("reasoning", None)
            messages_wo_thinking.append(msg_wo_thinking)

    return messages_wo_thinking
```
**EN:** `drop_thinking_messages` implements helper logic used by this module. It mainly works with `messages`, `last_user_idx`. Inside the body, it relies on `find_last_user_index`, `msg.get`, `messages_wo_thinking.append` to complete the main steps.
**CN:** `drop_thinking_messages` 负责实现本模块使用的辅助逻辑。 它主要处理 `messages`, `last_user_idx` 等参数。 实现过程中会调用 `find_last_user_index`, `msg.get`, `messages_wo_thinking.append` 等函数完成关键步骤。

### encode_messages (lines 315-335)
```python
def encode_messages(
    messages: list[dict[str, Any]],
    thinking_mode: str,
    context: list[dict[str, Any]] | None = None,
    drop_thinking: bool = True,
    add_default_bos_token: bool = True,
) -> str:
    context = context if context else []
    full_messages = context + messages

    prompt = bos_token if add_default_bos_token and len(context) == 0 else ""

    if thinking_mode == "thinking" and drop_thinking:
        full_messages = drop_thinking_messages(full_messages)

    for idx in range(len(messages)):
        prompt += render_message(
            idx + len(context), full_messages, thinking_mode=thinking_mode
        )

    return prompt
```
**EN:** `encode_messages` converts data into an encoded representation. It mainly works with `messages`, `thinking_mode`, `context`, `drop_thinking`. Inside the body, it relies on `drop_thinking_messages`, `render_message` to complete the main steps.
**CN:** `encode_messages` 负责把数据转换为编码后的表示。 它主要处理 `messages`, `thinking_mode`, `context`, `drop_thinking` 等参数。 实现过程中会调用 `drop_thinking_messages`, `render_message` 等函数完成关键步骤。

### _read_until_stop (lines 338-355)
```python
def _read_until_stop(
    index: int, text: str, stop: list[str]
) -> tuple[int, str, None | str]:
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
**EN:** `_read_until_stop` reads and normalizes data from an input source. It mainly works with `index`, `text`, `stop`. Inside the body, it relies on `text.find` to complete the main steps.
**CN:** `_read_until_stop` 负责从输入源读取并规范化数据。 它主要处理 `index`, `text`, `stop` 等参数。 实现过程中会调用 `text.find` 等函数完成关键步骤。

### parse_tool_calls (lines 358-415)
```python
def parse_tool_calls(index: int, text: str):
    tool_calls: list[dict[str, Any]] = []
    stop_token = None
    tool_calls_end_token = f"</{dsml_token}function_calls>"

    while index < len(text):
        index, _, stop_token = _read_until_stop(
            index, text, [f"<{dsml_token}invoke", tool_calls_end_token]
        )
        if _ != ">\n":
            raise RuntimeError("Tool call format error")

        if stop_token == tool_calls_end_token:
            break

        if stop_token is None:
            raise RuntimeError("Missing special token")

        index, tool_name_content, stop_token = _read_until_stop(
            index, text, [f"<{dsml_token}parameter", f"</{dsml_token}invoke"]
        )

        p_tool_name = re.findall(
            r'^\s*name="(.*?)">\n$', tool_name_content, flags=re.DOTALL
    # ...
                raise RuntimeError("Parameter format error")

        tool_call = decode_dsml_to_arguments(tool_name=tool_name, tool_args=tool_args)
        tool_calls.append(tool_call)

    return index, stop_token, tool_calls
```
**EN:** `parse_tool_calls` parses raw inputs into structured objects. It mainly works with `index`, `text`. Inside the body, it relies on `_read_until_stop`, `re.findall`, `decode_dsml_to_arguments` to complete the main steps.
**CN:** `parse_tool_calls` 负责把原始输入解析为结构化对象。 它主要处理 `index`, `text` 等参数。 实现过程中会调用 `_read_until_stop`, `re.findall`, `decode_dsml_to_arguments` 等函数完成关键步骤。

### parse_message_from_completion_text (lines 421-471)
```python
def parse_message_from_completion_text(text: str, thinking_mode: str):
    summary_content, reasoning, tool_calls = "", "", []
    index, stop_token = 0, None
    tool_calls_start_token = f"\n\n<{dsml_token}function_calls"

    is_thinking, is_tool_calling = thinking_mode == "thinking", False

    if is_thinking:
        index, content_delta, stop_token = _read_until_stop(
            index, text, [thinking_end_token, tool_calls_start_token]
        )
        reasoning = content_delta
        if stop_token != thinking_end_token:
            raise RuntimeError("Invalid thinking format")

    index, content_delta, stop_token = _read_until_stop(
        index, text, [eos_token, tool_calls_start_token]
    )
    summary_content = content_delta
    if stop_token == tool_calls_start_token:
        is_tool_calling = True
    else:
        if stop_token != eos_token:
            raise RuntimeError("Invalid summary format")
    # ...
    return {
        "role": "assistant",
        "content": summary_content,
        "reasoning": reasoning,
        "tool_calls": tool_calls_to_openai_format(tool_calls),
    }
```
**EN:** `parse_message_from_completion_text` parses raw inputs into structured objects. It mainly works with `text`, `thinking_mode`. Inside the body, it relies on `_read_until_stop`, `parse_tool_calls`, `RuntimeError` to complete the main steps.
**CN:** `parse_message_from_completion_text` 负责把原始输入解析为结构化对象。 它主要处理 `text`, `thinking_mode` 等参数。 实现过程中会调用 `_read_until_stop`, `parse_tool_calls`, `RuntimeError` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`to_json`**: Key helper or entry point in this file. / **`to_json`**：本文件中的关键辅助函数或入口。
- **`tools_from_openai_format`**: Key helper or entry point in this file. / **`tools_from_openai_format`**：本文件中的关键辅助函数或入口。
- **`tool_calls_from_openai_format`**: Key helper or entry point in this file. / **`tool_calls_from_openai_format`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: copy, json, typing
- **Third-party / 第三方**: regex
- **Internal vLLM / vLLM 内部依赖**: None / 无
