# mistral_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/mistral_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a mistral detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Mistral 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import json
import logging
from typing import Any, List, Optional, Tuple

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)
from sglang.srt.function_call.utils import _is_complete_json

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for mistral style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Mistral风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 18-18: Class `MistralDetector` declaration
```python
class MistralDetector(BaseFormatDetector):
```
**EN:** This block declares the class `MistralDetector` and establishes its responsibility inside mistral style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MistralDetector`.
**CN:** 这一段声明了类 `MistralDetector`，并说明它在Mistral风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MistralDetector`。

### Lines 19-33: Supporting state inside `MistralDetector`
```python
    """
    Detector for Mistral tool/function call formats.

    Supported formats:

    1) JSON-array format:
       `[TOOL_CALLS] [{"name": "...", "arguments": {...}}, ...]`

    2) Compact format (common in newer templates/models, especially in streaming):
       `[TOOL_CALLS]tool_name[ARGS]{...}`
       (also tolerates missing delimiters like `]` after `[TOOL_CALLS` and/or `[ARGS]` while streaming)

    Reference: https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3?chat_template=default
    """

```
**EN:** This block adds supporting state or helper logic inside `MistralDetector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `format`.
**CN:** 这一段为 `MistralDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `format`。

### Lines 34-42: Method `__init__`
```python
    def __init__(self):
        """Initialize tokens and streaming state."""
        super().__init__()
        # Canonical Mistral prefix for JSON-array tool calls.
        self.bot_token = "[TOOL_CALLS] ["
        # Common marker shared by both JSON-array and compact formats.
        self._tool_calls_marker = "[TOOL_CALLS"
        self.eot_token = "]"
        self.tool_call_separator = ", "
```
**EN:** This block defines the method `__init__` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 44-46: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Return True if the text contains either supported tool-call marker."""
        return self._tool_calls_marker in text
```
**EN:** This block defines the method `has_tool_call` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 48-55: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: ParseResult indicating success or failure, consumed text, leftover text, and parsed calls.
        """
```
**EN:** This block defines the method `detect_and_parse` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 56-79: Method `detect_and_parse` logic (part 1)
```python
        marker_idx = text.find(self._tool_calls_marker)
        if marker_idx == -1:
            return StreamingParseResult(normal_text=text, calls=[])

        normal_text = text[:marker_idx].strip()
        tool_part = text[marker_idx:]

        # Canonical: `[TOOL_CALLS] [{...}, ...]`
        if self.bot_token in tool_part:
            json_array_str = self._extract_json_array(tool_part)
            if not json_array_str:
                return StreamingParseResult(normal_text=normal_text, calls=[])

            calls: list = []
            try:
                function_call_arr = json.loads(json_array_str)
                if not isinstance(function_call_arr, list):
                    function_call_arr = [function_call_arr]
                calls = self.parse_base_json(function_call_arr, tools)
            except json.JSONDecodeError as e:
                logger.warning(
                    f"Failed to parse JSON part: {json_array_str}, JSON parse error: {str(e)}"
                )
            json_pos = tool_part.find(json_array_str) if json_array_str else -1
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `strip`, `_extract_json_array`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`strip`、`_extract_json_array`。

### Lines 80-105: Method `detect_and_parse` logic (part 2)
```python
            trailing_text = (
                tool_part[json_pos + len(json_array_str) :].strip()
                if json_pos != -1
                else ""
            )
            combined_normal = (
                (normal_text + " " + trailing_text).strip()
                if trailing_text
                else normal_text
            )
            return StreamingParseResult(normal_text=combined_normal, calls=calls)

        # Compact: `[TOOL_CALLS]tool_name[ARGS]{...}`
        # Loop to extract all consecutive compact tool calls.
        all_calls: list = []
        remaining = tool_part
        while remaining:
            parsed = self._try_parse_compact_args_format(remaining)
            if not parsed:
                break
            func_name, args_obj, consumed = parsed
            new_calls = self.parse_base_json(
                {"name": func_name, "arguments": args_obj}, tools
            )
            all_calls.extend(new_calls)
            remaining = remaining[consumed:].strip()
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `strip`, `StreamingParseResult`, `_try_parse_compact_args_format`, `parse_base_json`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`StreamingParseResult`、`_try_parse_compact_args_format`、`parse_base_json`。

### Lines 106-113: Method `detect_and_parse` logic (part 3)
```python

        if not all_calls:
            return StreamingParseResult(normal_text=normal_text, calls=[])

        combined_normal = (
            (normal_text + " " + remaining).strip() if remaining else normal_text
        )
        return StreamingParseResult(normal_text=combined_normal, calls=all_calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `strip`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`strip`。

### Lines 115-124: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming parsing for both JSON-array and compact formats.

        For the compact format, this buffers until the JSON arguments payload is complete,
        then emits two items: tool name (with empty parameters) and a full arguments JSON
        chunk (OpenAI streaming semantics).
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `name`, `chunk`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `name`、`chunk`。

### Lines 125-157: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        # No marker: either flush as normal text or keep buffering a partial marker.
        if self._tool_calls_marker not in current_text:
            if not self._ends_with_partial_token(self._buffer, self._tool_calls_marker):
                normal_text = self._buffer
                self._buffer = ""
                if self.eot_token in normal_text:
                    normal_text = normal_text.replace(self.eot_token, "")
                return StreamingParseResult(normal_text=normal_text)
            return StreamingParseResult()

        # If there's leading normal text before the marker, stream it out first.
        marker_pos = current_text.find(self._tool_calls_marker)
        if marker_pos > 0:
            normal_text = current_text[:marker_pos]
            self._buffer = current_text[marker_pos:]
            return StreamingParseResult(normal_text=normal_text)

        # Build tool indices if not already built.
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        # Try compact first; JSON-array requires `] [` and often arrives later in streaming.
        compact = self._try_parse_compact_args_format(current_text)
        if compact:
            func_name, args_obj, consumed = compact
            if func_name not in self._tool_indices:
                # Unknown tool: treat as normal text and reset state.
                normal_text = self._buffer
                self._buffer = ""
                return StreamingParseResult(normal_text=normal_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `_ends_with_partial_token`, `replace`, `StreamingParseResult`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_ends_with_partial_token`、`replace`、`StreamingParseResult`、`find`。

### Lines 158-189: Method `parse_streaming_increment` logic (part 2)
```python

            # Initialize state if this is the first tool call.
            if self.current_tool_id == -1:
                self.current_tool_id = 0
                self.prev_tool_call_arr = []
                self.streamed_args_for_tool = []

            args_json = json.dumps(args_obj, ensure_ascii=False)
            tool_id = self.current_tool_id

            # Ensure arrays are large enough.
            while len(self.prev_tool_call_arr) <= tool_id:
                self.prev_tool_call_arr.append({})
            while len(self.streamed_args_for_tool) <= tool_id:
                self.streamed_args_for_tool.append("")

            self.prev_tool_call_arr[tool_id] = {
                "name": func_name,
                "arguments": args_obj,
            }
            self.streamed_args_for_tool[tool_id] = args_json

            calls: List[ToolCallItem] = [
                ToolCallItem(tool_index=tool_id, name=func_name, parameters=""),
                ToolCallItem(tool_index=tool_id, name=None, parameters=args_json),
            ]

            # Consume parsed content from buffer.
            self._buffer = current_text[consumed:]
            self.current_tool_id += 1
            self.current_tool_name_sent = False
            return StreamingParseResult(normal_text="", calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `dumps`, `append`, `ToolCallItem`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `dumps`、`append`、`ToolCallItem`、`StreamingParseResult`。

### Lines 190-196: Method `parse_streaming_increment` logic (part 3)
```python

        # Canonical format delegates to the BaseFormatDetector JSON streaming logic.
        if self.bot_token in current_text:
            return super().parse_streaming_increment(new_text="", tools=tools)

        # Otherwise, keep buffering.
        return StreamingParseResult()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `parse_streaming_increment`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `parse_streaming_increment`、`StreamingParseResult`。

### Lines 198-211: Method `_try_parse_compact_args_format` signature and setup
```python
    def _try_parse_compact_args_format(
        self, text: str
    ) -> Optional[Tuple[str, Any, int]]:
        """
        Parse the compact tool call format:
            `[TOOL_CALLS]tool_name[ARGS]{...}`

        Tolerates common streaming variants where delimiters are missing:
            `[TOOL_CALLStool_name[ARGS{...}`

        Returns:
            (tool_name, arguments_obj, consumed_end_index) if a complete JSON arguments
            payload is present; otherwise None.
        """
```
**EN:** This block defines the method `_try_parse_compact_args_format` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `_try_parse_compact_args_format`.
**CN:** 这一段定义了method `_try_parse_compact_args_format`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_try_parse_compact_args_format`。

### Lines 212-235: Method `_try_parse_compact_args_format` logic (part 1)
```python
        start = text.find(self._tool_calls_marker)
        if start == -1:
            return None

        i = start + len(self._tool_calls_marker)  # position after "[TOOL_CALLS"
        if i < len(text) and text[i] == "]":
            i += 1
        while i < len(text) and text[i].isspace():
            i += 1

        args_marker = "[ARGS"
        args_pos = text.find(args_marker, i)
        if args_pos == -1:
            return None

        func_name = text[i:args_pos].strip()
        if not func_name:
            return None

        j = args_pos + len(args_marker)
        if j < len(text) and text[j] == "]":
            j += 1
        while j < len(text) and text[j].isspace():
            j += 1
```
**EN:** This block continues `_try_parse_compact_args_format` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `find`, `isspace`, `strip`.
**CN:** 这一段延续了 `_try_parse_compact_args_format` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`isspace`、`strip`。

### Lines 236-251: Method `_try_parse_compact_args_format` logic (part 2)
```python

        if j >= len(text) or text[j] not in "{[":
            return None

        json_str, end_idx = self._extract_json_value(text, j)
        if not json_str:
            return None
        if not _is_complete_json(json_str):
            return None

        try:
            args_obj = json.loads(json_str)
        except json.JSONDecodeError:
            return None

        return func_name, args_obj, end_idx
```
**EN:** This block continues `_try_parse_compact_args_format` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `_extract_json_value`, `_is_complete_json`, `loads`.
**CN:** 这一段延续了 `_try_parse_compact_args_format` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_extract_json_value`、`_is_complete_json`、`loads`。

### Lines 253-262: Method `_extract_json_value` signature and setup
```python
    def _extract_json_value(
        self, text: str, json_start: int
    ) -> Tuple[Optional[str], int]:
        """
        Extract a JSON value (object or array) starting at json_start using bracket counting,
        robust to nested braces/brackets inside strings.

        Returns:
            (json_str_or_None, end_index_exclusive)
        """
```
**EN:** This block defines the method `_extract_json_value` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `_extract_json_value`. Notable operations include `value`.
**CN:** 这一段定义了method `_extract_json_value`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract_json_value`。 值得注意的操作包括 `value`。

### Lines 263-290: Method `_extract_json_value` logic (part 1)
```python
        if json_start >= len(text) or text[json_start] not in "{[":
            return None, json_start

        opening = text[json_start]
        closing = "}" if opening == "{" else "]"
        depth = 0
        in_string = False
        escape_next = False

        for k in range(json_start, len(text)):
            ch = text[k]
            if escape_next:
                escape_next = False
                continue
            if ch == "\\":
                escape_next = True
                continue
            if ch == '"' and not escape_next:
                in_string = not in_string
                continue
            if in_string:
                continue
            if ch == opening:
                depth += 1
            elif ch == closing:
                depth -= 1
                if depth == 0:
                    return text[json_start : k + 1], k + 1
```
**EN:** This block continues `_extract_json_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_extract_json_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。

### Lines 291-292: Method `_extract_json_value` logic (part 2)
```python

        return None, json_start
```
**EN:** This block continues `_extract_json_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_extract_json_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。

### Lines 294-300: Method `_extract_json_array` signature and setup
```python
    def _extract_json_array(self, text: str) -> str:
        """
        Extract the JSON array part using bracket counting to handle nested brackets.

        :param text: The complete text containing [TOOL_CALLS] [...]
        :return: The JSON array string or None if not found
        """
```
**EN:** This block defines the method `_extract_json_array` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `_extract_json_array`.
**CN:** 这一段定义了method `_extract_json_array`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract_json_array`。

### Lines 301-322: Method `_extract_json_array` logic (part 1)
```python
        start_idx = text.find(self.bot_token)
        if start_idx == -1:
            return None

        # Start from the opening bracket after [TOOL_CALLS]
        json_start = (
            start_idx + len(self.bot_token) - 1
        )  # -1 to include the opening bracket
        bracket_count = 0
        in_string = False
        escape_next = False

        for i in range(json_start, len(text)):
            char = text[i]

            if escape_next:
                escape_next = False
                continue

            if char == "\\":
                escape_next = True
                continue
```
**EN:** This block continues `_extract_json_array` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow. Notable operations include `find`.
**CN:** 这一段延续了 `_extract_json_array` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`。

### Lines 323-336: Method `_extract_json_array` logic (part 2)
```python

            if char == '"' and not escape_next:
                in_string = not in_string
                continue

            if not in_string:
                if char == "[":
                    bracket_count += 1
                elif char == "]":
                    bracket_count -= 1
                    if bracket_count == 0:
                        return text[json_start : i + 1]

        return None
```
**EN:** This block continues `_extract_json_array` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mistral style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_extract_json_array` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Mistral风格的工具调用检测与规范化工作流。

### Lines 338-343: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin='[TOOL_CALLS] [{"name":"' + name + '", "arguments":',
            end="}]",
            trigger="[TOOL_CALLS]",
        )
```
**EN:** This block defines the method `structure_info` on `MistralDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mistral style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `MistralDetector`），介绍了参数、初始化步骤，以及这部分Mistral风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `MistralDetector`: Class that encapsulates mistral detector behavior in this module. / `MistralDetector`：封装与“Mistral检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.utils`
