# streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/streaming.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements shared helpers for turning partial model output into streaming OpenAI-compatible tool-call deltas. / 实现共享的流式辅助逻辑，用于把模型的部分输出转换成兼容 OpenAI 的工具调用增量。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-19)
```python
import json
from typing import TYPE_CHECKING

import partial_json_parser
import regex as re
from partial_json_parser.core.options import Allow

from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.engine.protocol import (
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
)
from vllm.tool_parsers.mistral_tool_parser import MistralToolCall
from vllm.tool_parsers.utils import partial_json_loads
from vllm.utils.mistral import is_mistral_tokenizer
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Function `_bracket_level` (lines 27-35)
```python
def _bracket_level(s: str, opening: str = "{", closing: str = "}") -> int:
    """Calculate the current level of nested brackets in a string."""
    level = 0
    for char in s:
        if char == opening:
            level += 1
        elif char == closing:
            level -= 1
    return level
```
**EN:** Calculate the current level of nested brackets in a string.
**CN:** `_bracket_level` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `filter_delta_text` (lines 38-59)
```python
def filter_delta_text(
    delta_text: str,
    previous_text: str,
) -> tuple[str, bool]:
    """Trim trailing tool-list delimiters from required-tool streaming text."""
    bracket_level = _bracket_level(previous_text)
    updated_delta = ""
    passed_zero = False
    for char in delta_text:
        if char == "{":
            bracket_level += 1
            passed_zero = bracket_level == 0
        elif char == "}":
            bracket_level -= 1
            passed_zero = bracket_level == 0

        if bracket_level != 0:
            updated_delta += char
        else:
            if char == ",":
                break
    return updated_delta, passed_zero
```
**EN:** Trim trailing tool-list delimiters from required-tool streaming text.
**CN:** `filter_delta_text` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `extract_named_tool_call_streaming` (lines 62-100)
```python
def extract_named_tool_call_streaming(
    *,
    delta_text: str,
    function_name: str,
    function_name_returned: bool,
    tool_call_idx: int | None,
    tool_call_id_type: str,
    tokenizer: "TokenizerLike",
    tool_call_array_index: int = 0,
) -> tuple[DeltaMessage | None, bool]:
    """Build a streaming tool-call delta for forced named tool choice."""
    if function_name_returned:
        delta_tool_call = DeltaToolCall(
            function=DeltaFunctionCall(arguments=delta_text),
            index=tool_call_array_index,
        )
    else:
        if is_mistral_tokenizer(tokenizer):
            tool_call_id = MistralToolCall.generate_random_id()
        else:
            tool_call_id = make_tool_call_id(
                id_type=tool_call_id_type,
                func_name=function_name,
                idx=tool_call_idx,
            )
        delta_tool_call = DeltaToolCall(
            id=tool_call_id,
            type="function",
            function=DeltaFunctionCall(
                name=function_name,
                arguments=delta_text,
            ),
            index=tool_call_array_index,
        )
        function_name_returned = True
    return (
        DeltaMessage(tool_calls=[delta_tool_call]),
        function_name_returned,
    )
```
**EN:** Build a streaming tool-call delta for forced named tool choice.
**CN:** `extract_named_tool_call_streaming` 函数是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Function `extract_required_tool_call_streaming` (lines 103-195)
```python
def extract_required_tool_call_streaming(
    *,
    previous_text: str,
    current_text: str | None,
    delta_text: str,
    function_name_returned: bool,
    tool_call_idx: int | None,
    tool_call_id_type: str,
) -> tuple[DeltaMessage | None, bool]:
    if current_text is None or current_text == "":
        # if the current text is empty, we cannot parse it
        return None, function_name_returned
    try:
        flags = Allow.ALL
        obj, _ = partial_json_loads(current_text, flags)
    except (
        partial_json_parser.core.exceptions.MalformedJSON,
        json.JSONDecodeError,
    ):
        obj = None

    # check if the current text is a valid array
    # containing a partial tool calling object
    # if not repeat
    if obj is None or not isinstance(obj, list) or not len(obj) > 0:
        function_name_returned = False
        delta_message = None
    else:
        _, finishes_previous_tool = filter_delta_text(delta_text, previous_text)
        # take the last tool call from the generated list
        current_tool_call = obj[-1]

        # once parameters have been generated the name is complete as well
        if not finishes_previous_tool and (
            "name" not in current_tool_call or "parameters" not in current_tool_call
        ):
            function_name_returned = False
            delta_message = None
        else:
            if not function_name_returned:
                # get partly generated arguments from the latest tool call
                param_match = re.search(
                    r'.*"parameters":\s*(.*)', current_text, re.DOTALL
                )
                arguments = param_match.group(1) if param_match else ""
                arguments, _ = filter_delta_text(arguments, previous_text)

                # if this iteration finishes a previous tool call but a
                # new incomplete tool is already generated, take the
                # previous from the list
# ... omitted for brevity / 为简洁省略 ...
                                    # instead of name every time
                                    name=None,
                                    arguments=delta_text,
                                ),
                                index=len(obj) - 1,
                            )
                        ]
                    )
                else:
                    delta_message = None

    return delta_message, function_name_returned
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `extract_required_tool_call_streaming` 函数是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Partial JSON recovery**: Partial JSON recovery. / 在 JSON 尚未闭合时尽量提取稳定的中间结果。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `typing`
- **Third-party / 第三方**: `partial_json_parser`, `regex`, `partial_json_parser.core.options`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tool_parsers.mistral_tool_parser`, `vllm.tool_parsers.utils`, `vllm.utils.mistral`
