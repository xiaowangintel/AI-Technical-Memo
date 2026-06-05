# gemma4_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/gemma4_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a gemma 4 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Gemma4 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
import json
import logging
from typing import List, Optional

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)

# Gemma4 special tokens for tool calls
TOOL_CALL_START = "<|tool_call>"
TOOL_CALL_END = "<tool_call|>"
STRING_DELIM = '<|"|>'


```
**EN:** This block gathers the imports and module-level setup for gemma 4 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Gemma4风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 21-42: Function `_parse_gemma4_value`
```python
def _parse_gemma4_value(value_str: str) -> object:
    """Parse a single Gemma4 value (after key:) into a Python object."""
    value_str = value_str.strip()
    if not value_str:
        return value_str

    # Boolean
    if value_str == "true":
        return True
    if value_str == "false":
        return False

    # Number (int or float)
    try:
        if "." in value_str:
            return float(value_str)
        return int(value_str)
    except ValueError:
        pass

    # Bare string (no <|"|> delimiters)
    return value_str
```
**EN:** This block defines the function `_parse_gemma4_value`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `_parse_gemma4_value`. Notable operations include `value`, `strip`, `Number`, `string`.
**CN:** 这一段定义了function `_parse_gemma4_value`，介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_gemma4_value`。 值得注意的操作包括 `value`、`strip`、`Number`、`string`。

### Lines 45-46: Function `_parse_gemma4_array` signature and setup
```python
def _parse_gemma4_array(arr_str: str) -> list:
    """Parse a Gemma4 array content string into a Python list."""
```
**EN:** This block defines the function `_parse_gemma4_array`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `_parse_gemma4_array`.
**CN:** 这一段定义了function `_parse_gemma4_array`，介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_gemma4_array`。

### Lines 47-65: Function `_parse_gemma4_array` logic (part 1)
```python
    items: list = []
    i = 0
    n = len(arr_str)

    while i < n:
        while i < n and arr_str[i] in (" ", ",", "\n", "\t"):
            i += 1
        if i >= n:
            break

        # String element
        if arr_str[i : i + len(STRING_DELIM)] == STRING_DELIM:
            i += len(STRING_DELIM)
            end_pos = arr_str.find(STRING_DELIM, i)
            if end_pos == -1:
                items.append(arr_str[i:])
                break
            items.append(arr_str[i:end_pos])
            i = end_pos + len(STRING_DELIM)
```
**EN:** This block continues `_parse_gemma4_array` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `in`, `find`, `append`.
**CN:** 这一段延续了 `_parse_gemma4_array` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `in`、`find`、`append`。

### Lines 66-83: Function `_parse_gemma4_array` logic (part 2)
```python

        # Nested object
        elif arr_str[i] == "{":
            depth = 1
            obj_start = i + 1
            i += 1
            while i < n and depth > 0:
                if arr_str[i : i + len(STRING_DELIM)] == STRING_DELIM:
                    i += len(STRING_DELIM)
                    next_delim = arr_str.find(STRING_DELIM, i)
                    i = next_delim + len(STRING_DELIM) if next_delim != -1 else n
                    continue
                if arr_str[i] == "{":
                    depth += 1
                elif arr_str[i] == "}":
                    depth -= 1
                i += 1
            items.append(_parse_gemma4_args(arr_str[obj_start : i - 1]))
```
**EN:** This block continues `_parse_gemma4_array` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `find`, `append`, `_parse_gemma4_args`.
**CN:** 这一段延续了 `_parse_gemma4_array` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`append`、`_parse_gemma4_args`。

### Lines 84-105: Function `_parse_gemma4_array` logic (part 3)
```python

        # Nested array
        elif arr_str[i] == "[":
            depth = 1
            sub_start = i + 1
            i += 1
            while i < n and depth > 0:
                if arr_str[i] == "[":
                    depth += 1
                elif arr_str[i] == "]":
                    depth -= 1
                i += 1
            items.append(_parse_gemma4_array(arr_str[sub_start : i - 1]))

        # Bare value
        else:
            val_start = i
            while i < n and arr_str[i] not in (",", "]"):
                i += 1
            items.append(_parse_gemma4_value(arr_str[val_start:i]))

    return items
```
**EN:** This block continues `_parse_gemma4_array` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `append`, `_parse_gemma4_array`, `in`, `_parse_gemma4_value`.
**CN:** 这一段延续了 `_parse_gemma4_array` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`_parse_gemma4_array`、`in`、`_parse_gemma4_value`。

### Lines 108-109: Function `_parse_gemma4_args` signature and setup
```python
def _parse_gemma4_args(args_str: str) -> dict:
    """Parse Gemma4's custom key:value format into a Python dict."""
```
**EN:** This block defines the function `_parse_gemma4_args`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `_parse_gemma4_args`.
**CN:** 这一段定义了function `_parse_gemma4_args`，介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_gemma4_args`。

### Lines 110-143: Function `_parse_gemma4_args` logic (part 1)
```python
    if not args_str or not args_str.strip():
        return {}

    result: dict = {}
    i = 0
    n = len(args_str)

    while i < n:
        # Skip whitespace and commas
        while i < n and args_str[i] in (" ", ",", "\n", "\t"):
            i += 1
        if i >= n:
            break

        # Parse key (unquoted, ends at ':')
        key_start = i
        while i < n and args_str[i] != ":":
            i += 1
        if i >= n:
            break
        key = args_str[key_start:i].strip()
        i += 1  # skip ':'

        # Parse value
        if i >= n:
            result[key] = ""
            break

        # Skip whitespace after ':'
        while i < n and args_str[i] in (" ", "\n", "\t"):
            i += 1
        if i >= n:
            result[key] = ""
            break
```
**EN:** This block continues `_parse_gemma4_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `strip`, `in`, `key`.
**CN:** 这一段延续了 `_parse_gemma4_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`in`、`key`。

### Lines 144-177: Function `_parse_gemma4_args` logic (part 2)
```python

        # String value: <|"|>...<|"|>
        if args_str[i : i + len(STRING_DELIM)] == STRING_DELIM:
            i += len(STRING_DELIM)
            val_start = i
            end_pos = args_str.find(STRING_DELIM, i)
            if end_pos == -1:
                # Unterminated string — take rest
                result[key] = args_str[val_start:]
                break
            result[key] = args_str[val_start:end_pos]
            i = end_pos + len(STRING_DELIM)

        # Nested object: {...}
        elif args_str[i] == "{":
            depth = 1
            obj_start = i + 1
            i += 1
            while i < n and depth > 0:
                if args_str[i : i + len(STRING_DELIM)] == STRING_DELIM:
                    # Skip over string contents
                    i += len(STRING_DELIM)
                    next_delim = args_str.find(STRING_DELIM, i)
                    if next_delim == -1:
                        i = n
                    else:
                        i = next_delim + len(STRING_DELIM)
                    continue
                if args_str[i] == "{":
                    depth += 1
                elif args_str[i] == "}":
                    depth -= 1
                i += 1
            result[key] = _parse_gemma4_args(args_str[obj_start : i - 1])
```
**EN:** This block continues `_parse_gemma4_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `find`, `_parse_gemma4_args`.
**CN:** 这一段延续了 `_parse_gemma4_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`_parse_gemma4_args`。

### Lines 178-208: Function `_parse_gemma4_args` logic (part 3)
```python

        # Array: [...]
        elif args_str[i] == "[":
            depth = 1
            arr_start = i + 1
            i += 1
            while i < n and depth > 0:
                if args_str[i : i + len(STRING_DELIM)] == STRING_DELIM:
                    i += len(STRING_DELIM)
                    next_delim = args_str.find(STRING_DELIM, i)
                    if next_delim == -1:
                        i = n
                    else:
                        i = next_delim + len(STRING_DELIM)
                    continue
                if args_str[i] == "[":
                    depth += 1
                elif args_str[i] == "]":
                    depth -= 1
                i += 1
            arr_content = args_str[arr_start : i - 1]
            result[key] = _parse_gemma4_array(arr_content)

        # Bare value (number, boolean, etc.)
        else:
            val_start = i
            while i < n and args_str[i] not in (",", "}", "]"):
                i += 1
            result[key] = _parse_gemma4_value(args_str[val_start:i])

    return result
```
**EN:** This block continues `_parse_gemma4_args` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `find`, `_parse_gemma4_array`, `value`, `in`.
**CN:** 这一段延续了 `_parse_gemma4_args` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`_parse_gemma4_array`、`value`、`in`。

### Lines 211-234: Function `_find_matching_brace`
```python
def _find_matching_brace(text: str) -> int:
    """Find index of matching '}' in text, respecting STRING_DELIM and nesting.

    Assumes text starts just after the opening '{'.
    Returns index of closing brace, or -1 if not found (incomplete).
    """
    depth = 1
    i = 0
    n = len(text)
    delim_len = len(STRING_DELIM)
    while i < n and depth > 0:
        if text[i : i + delim_len] == STRING_DELIM:
            i += delim_len
            next_delim = text.find(STRING_DELIM, i)
            if next_delim == -1:
                return -1
            i = next_delim + delim_len
            continue
        if text[i] == "{":
            depth += 1
        elif text[i] == "}":
            depth -= 1
        i += 1
    return (i - 1) if depth == 0 else -1
```
**EN:** This block defines the function `_find_matching_brace`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `_find_matching_brace`. Notable operations include `found`, `find`.
**CN:** 这一段定义了function `_find_matching_brace`，介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_find_matching_brace`。 值得注意的操作包括 `found`、`find`。

### Lines 237-237: Class `Gemma4Detector` declaration
```python
class Gemma4Detector(BaseFormatDetector):
```
**EN:** This block declares the class `Gemma4Detector` and establishes its responsibility inside gemma 4 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Gemma4Detector`.
**CN:** 这一段声明了类 `Gemma4Detector`，并说明它在Gemma4风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Gemma4Detector`。

### Lines 238-247: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.tool_call_start_token = TOOL_CALL_START
        self.tool_call_end_token = TOOL_CALL_END

        # Streaming state
        self.parsed_pos: int = 0
        self.is_inside_tool_call: bool = False
        self.current_func_name: Optional[str] = None
        self._tool_indices: Optional[dict] = None
```
**EN:** This block defines the method `__init__` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 249-251: Method `_extract_tool_calls` signature and setup
```python
    @staticmethod
    def _extract_tool_calls(text: str) -> list:
        """Extract (func_name, args_str) pairs using brace-balanced parsing."""
```
**EN:** This block defines the method `_extract_tool_calls` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `_extract_tool_calls`. Notable operations include `Extract`.
**CN:** 这一段定义了method `_extract_tool_calls`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract_tool_calls`。 值得注意的操作包括 `Extract`。

### Lines 252-273: Method `_extract_tool_calls` logic (part 1)
```python
        results = []
        search_from = 0
        while True:
            start = text.find(TOOL_CALL_START, search_from)
            if start == -1:
                break
            end = text.find(TOOL_CALL_END, start)
            if end == -1:
                break
            inner = text[start + len(TOOL_CALL_START) : end]
            if inner.startswith("call:"):
                brace = inner.find("{")
                if brace != -1:
                    func_name = inner[5:brace]
                    args_content = inner[brace + 1 :]
                    match_idx = _find_matching_brace(args_content)
                    args_str = (
                        args_content[:match_idx] if match_idx != -1 else args_content
                    )
                    results.append((func_name, args_str))
            search_from = end + len(TOOL_CALL_END)
        return results
```
**EN:** This block continues `_extract_tool_calls` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `find`, `startswith`, `_find_matching_brace`, `append`.
**CN:** 这一段延续了 `_extract_tool_calls` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`startswith`、`_find_matching_brace`、`append`。

### Lines 275-276: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.tool_call_start_token in text
```
**EN:** This block defines the method `has_tool_call` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 278-280: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        if self.tool_call_start_token not in text:
            return StreamingParseResult(normal_text=text)
```
**EN:** This block defines the method `detect_and_parse` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `StreamingParseResult`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `StreamingParseResult`。

### Lines 281-303: Method `detect_and_parse` logic (part 1)
```python

        calls = []
        try:
            matches = self._extract_tool_calls(text)
            if not matches:
                return StreamingParseResult(normal_text=text)

            tool_indices = self._get_tool_indices(tools)
            for func_name, args_str in matches:
                arguments = _parse_gemma4_args(args_str)
                calls.append(
                    ToolCallItem(
                        tool_index=tool_indices.get(func_name, -1),
                        name=func_name,
                        parameters=json.dumps(arguments, ensure_ascii=False),
                    )
                )

            # Content = text before first tool call
            content_end = text.find(self.tool_call_start_token)
            normal_text = text[:content_end] if content_end > 0 else ""

            return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `_extract_tool_calls`, `StreamingParseResult`, `_get_tool_indices`, `_parse_gemma4_args`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_extract_tool_calls`、`StreamingParseResult`、`_get_tool_indices`、`_parse_gemma4_args`。

### Lines 304-307: Method `detect_and_parse` logic (part 2)
```python

        except (ValueError, IndexError, TypeError, KeyError) as e:
            logger.error(f"Error in detect_and_parse: {e}", exc_info=True)
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `error`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `error`、`StreamingParseResult`。

### Lines 309-311: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
```
**EN:** This block defines the method `parse_streaming_increment` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 312-343: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        if not self._buffer:
            return StreamingParseResult()

        calls = []
        normal_text_chunks = []
        if self._tool_indices is None:
            self._tool_indices = self._get_tool_indices(tools)

        try:
            while True:
                current_slice = self._buffer[self.parsed_pos :]
                if not current_slice:
                    break

                if not self.is_inside_tool_call:
                    # Outside tool call block
                    next_start = current_slice.find(self.tool_call_start_token)
                    if next_start == -1:
                        # Check for partial match at the end
                        partial_len = self._ends_with_partial_token(
                            current_slice, self.tool_call_start_token
                        )
                        if partial_len > 0:
                            text_to_append = current_slice[:-partial_len]
                            if text_to_append:
                                normal_text_chunks.append(text_to_append)
                            self.parsed_pos += len(text_to_append)
                            break
                        else:
                            normal_text_chunks.append(current_slice)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `_get_tool_indices`, `find`, `_ends_with_partial_token`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`_get_tool_indices`、`find`、`_ends_with_partial_token`。

### Lines 344-376: Method `parse_streaming_increment` logic (part 2)
```python
                            self.parsed_pos += len(current_slice)
                            continue
                    elif next_start == 0:
                        self.parsed_pos += len(self.tool_call_start_token)
                        self.is_inside_tool_call = True
                        continue
                    else:
                        normal_text_chunks.append(current_slice[:next_start])
                        self.parsed_pos += next_start
                        continue
                else:
                    # Inside tool call block

                    # Check for TOOL_CALL_END first
                    if current_slice.startswith(self.tool_call_end_token):
                        self.parsed_pos += len(self.tool_call_end_token)
                        self.is_inside_tool_call = False
                        self.current_func_name = None
                        continue

                    if not self.current_func_name:
                        # Skip leading whitespace
                        if current_slice[0] in (" ", "\n", "\t"):
                            self.parsed_pos += 1
                            continue

                        if current_slice.startswith("call:"):
                            brace_pos = current_slice.find("{")
                            if brace_pos != -1:
                                func_name = current_slice[5:brace_pos]
                                self.current_tool_id += 1
                                self.current_func_name = func_name
                                self.current_tool_name_sent = True
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `append`, `startswith`, `in`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`startswith`、`in`、`find`。

### Lines 377-407: Method `parse_streaming_increment` logic (part 3)
```python

                                calls.append(
                                    ToolCallItem(
                                        tool_index=self._tool_indices.get(
                                            func_name, -1
                                        ),
                                        name=func_name,
                                        parameters="",
                                    )
                                )
                                self.parsed_pos += brace_pos + 1
                                continue
                            else:
                                # Incomplete call:name{
                                break
                        else:
                            # Check for partial matches
                            if "call:".startswith(
                                current_slice
                            ) or self.tool_call_end_token.startswith(current_slice):
                                break

                            # Unexpected content, skip
                            self.parsed_pos += 1
                            continue
                    else:
                        # Parsing arguments (looking for balancing })
                        match_idx = _find_matching_brace(current_slice)
                        if match_idx != -1:
                            args_str = current_slice[:match_idx]
                            arguments = _parse_gemma4_args(args_str)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `get`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`get`、`startswith`。

### Lines 408-439: Method `parse_streaming_increment` logic (part 4)
```python

                            calls.append(
                                ToolCallItem(
                                    tool_index=self._tool_indices.get(
                                        self.current_func_name, -1
                                    ),
                                    parameters=json.dumps(
                                        arguments, ensure_ascii=False
                                    ),
                                )
                            )
                            self.parsed_pos += match_idx + 1
                            self.current_func_name = None
                            continue
                        else:
                            # Incomplete arguments block
                            break

        except (ValueError, IndexError, TypeError, KeyError) as e:
            logger.error(f"Error in parse_streaming_increment: {e}", exc_info=True)
            # Reset parser state to prevent corruption
            self.is_inside_tool_call = False
            self.current_func_name = None
            self._buffer = ""
            self.parsed_pos = 0

        if self.parsed_pos > 0:
            self._buffer = self._buffer[self.parsed_pos :]
            self.parsed_pos = 0

        normal_text = "".join(normal_text_chunks) if normal_text_chunks else ""
        return StreamingParseResult(calls=calls, normal_text=normal_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gemma 4 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `get`, `dumps`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Gemma4风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`get`、`dumps`。

### Lines 441-442: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 444-445: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError
```
**EN:** This block defines the method `structure_info` on `Gemma4Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gemma 4 style tool-call detection and normalization. Definitions introduced here include `structure_info`.
**CN:** 这一段定义了method `structure_info`（属于 `Gemma4Detector`），介绍了参数、初始化步骤，以及这部分Gemma4风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。

## Key Concepts / 关键概念
- `_parse_gemma4_value`: Function that performs parse gemma 4 value for the surrounding workflow. / `_parse_gemma4_value`：在周边工作流中执行“解析Gemma4value”相关任务的函数。
- `_parse_gemma4_array`: Function that performs parse gemma 4 array for the surrounding workflow. / `_parse_gemma4_array`：在周边工作流中执行“解析Gemma4数组”相关任务的函数。
- `_parse_gemma4_args`: Function that performs parse gemma 4 args for the surrounding workflow. / `_parse_gemma4_args`：在周边工作流中执行“解析Gemma4args”相关任务的函数。
- `_find_matching_brace`: Function that performs find matching brace for the surrounding workflow. / `_find_matching_brace`：在周边工作流中执行“findmatchingbrace”相关任务的函数。
- `Gemma4Detector`: Class that encapsulates gemma 4 detector behavior in this module. / `Gemma4Detector`：封装与“Gemma4检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
