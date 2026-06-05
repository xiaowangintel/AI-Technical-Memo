# pythonic_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/pythonic_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a pythonic detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Python 风格 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import ast
import json
import logging
import re
from typing import List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for pythonic style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Python 风格风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 19-19: Class `PythonicDetector` declaration
```python
class PythonicDetector(BaseFormatDetector):
```
**EN:** This block declares the class `PythonicDetector` and establishes its responsibility inside pythonic style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PythonicDetector`.
**CN:** 这一段声明了类 `PythonicDetector`，并说明它在Python 风格风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PythonicDetector`。

### Lines 20-33: Supporting state inside `PythonicDetector`
```python
    """
    Detector for Llama-4 models with Pythonic tool call format.

    The Pythonic format uses Python function call syntax within square brackets,
    with arguments as Python literals rather than JSON.

    Format Structure:
    ```
    [tool1(arg1=val1, arg2=val2), tool2(arg1=val3)]
    ```

    Reference: https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct?chat_template=default
    """

```
**EN:** This block adds supporting state or helper logic inside `PythonicDetector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `tool1`, `tool2`.
**CN:** 这一段为 `PythonicDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `tool1`、`tool2`。

### Lines 34-39: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.tool_call_regex = re.compile(
            r"\[([a-zA-Z]+\w*\(([a-zA-Z]+\w*=.*,\s*)*([a-zA-Z]+\w*=.*\s)?\),\s*)*([a-zA-Z]+\w*\(([a-zA-Z]+\w*=.*,\s*)*([a-zA-Z]+\w*=.*\s*)?\)\s*)+\]",
            re.DOTALL,
        )
```
**EN:** This block defines the method `__init__` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `compile`.
**CN:** 这一段定义了method `__init__`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`compile`。

### Lines 41-47: Method `_text_strip`
```python
    @staticmethod
    def _text_strip(text: str) -> str:
        # Llama 4 model sometime will output <|python_start|> and <|python_end|> tokens
        # remove those tokens
        text = text.replace("<|python_start|>", "")
        text = text.replace("<|python_end|>", "")
        return text
```
**EN:** This block defines the method `_text_strip` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `_text_strip`. Notable operations include `replace`.
**CN:** 这一段定义了method `_text_strip`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_text_strip`。 值得注意的操作包括 `replace`。

### Lines 49-50: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return bool(self.tool_call_regex.search(self._text_strip(text.strip())))
```
**EN:** This block defines the method `has_tool_call` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `has_tool_call`. Notable operations include `search`, `_text_strip`, `strip`.
**CN:** 这一段定义了method `has_tool_call`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `search`、`_text_strip`、`strip`。

### Lines 52-54: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        # Try parsing the text as a Python list of function calls
        text = text.strip()
```
**EN:** This block defines the method `detect_and_parse` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `strip`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `strip`。

### Lines 55-81: Method `detect_and_parse` logic (part 1)
```python

        # Remove unexpected <|python_start|> and <|python_end|> for llama4
        text = self._text_strip(text)

        match = self.tool_call_regex.search(text)
        if match is None:
            return StreamingParseResult(normal_text=text, calls=[])

        # Extract the tool call part and any text before/after it
        tool_call_start = match.start()
        tool_call_end = match.end()

        normal_text_before = text[:tool_call_start] if tool_call_start > 0 else ""
        tool_call_text = text[tool_call_start:tool_call_end]
        normal_text_after = text[tool_call_end:] if tool_call_end < len(text) else ""

        # Combine normal text
        normal_text = normal_text_before + normal_text_after

        try:
            module = ast.parse(tool_call_text)
            parsed = getattr(module.body[0], "value", None)
            if not (
                isinstance(parsed, ast.List)
                and all(isinstance(e, ast.Call) for e in parsed.elts)
            ):
                return StreamingParseResult(normal_text=normal_text, calls=[])
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding pythonic style tool-call detection and normalization workflow. Notable operations include `_text_strip`, `search`, `StreamingParseResult`, `start`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Python 风格风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_text_strip`、`search`、`StreamingParseResult`、`start`。

### Lines 82-106: Method `detect_and_parse` logic (part 2)
```python

            calls = []
            tool_indices = self._get_tool_indices(tools)
            for call_index, call in enumerate(parsed.elts):
                if not isinstance(call.func, ast.Name):
                    continue
                function_name = call.func.id
                # Validate that the function exists in the tools
                if function_name not in tool_indices:
                    logger.warning(
                        f"Model attempted to call undefined function: {function_name}"
                    )
                    if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                        continue  # Skip unknown tools (default legacy behavior)

                arguments = {}
                for keyword in call.keywords:
                    arguments[keyword.arg] = self._get_parameter_value(keyword.value)
                calls.append(
                    ToolCallItem(
                        tool_index=call_index,  # Use the call index in the response, not tool position
                        name=function_name,
                        parameters=json.dumps(arguments, ensure_ascii=False),
                    )
                )
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding pythonic style tool-call detection and normalization workflow. Notable operations include `_get_tool_indices`, `warning`, `get`, `tools`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Python 风格风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_tool_indices`、`warning`、`get`、`tools`。

### Lines 107-111: Method `detect_and_parse` logic (part 3)
```python

            return StreamingParseResult(normal_text=normal_text, calls=calls)
        except Exception:
            logger.exception("Error in pythonic tool call parsing.")
            return StreamingParseResult(normal_text=normal_text, calls=[])
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding pythonic style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `exception`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Python 风格风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`exception`。

### Lines 113-133: Method `_find_matching_bracket`
```python
    def _find_matching_bracket(self, buffer: str, start: int) -> int:
        """
        Find the matching closing bracket for the opening bracket at start position.
        Properly handles nested brackets.

        Args:
            buffer: The text buffer to search in
            start: Position of the opening bracket '['

        Returns:
            Position of the matching closing bracket ']', or -1 if not found
        """
        bracket_count = 0
        for i in range(start, len(buffer)):
            if buffer[i] == "[":
                bracket_count += 1
            elif buffer[i] == "]":
                bracket_count -= 1
                if bracket_count == 0:
                    return i
        return -1  # No matching bracket found
```
**EN:** This block defines the method `_find_matching_bracket` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `_find_matching_bracket`.
**CN:** 这一段定义了method `_find_matching_bracket`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_find_matching_bracket`。

### Lines 135-157: Method `_strip_and_split_buffer`
```python
    def _strip_and_split_buffer(self, buffer: str) -> tuple[str, str]:
        """
        Strip special tokens from buffer and split into safe_text and held_back_text.

        Returns:
            tuple of (safe_text_to_output, text_to_hold_in_buffer)
        """
        # Check if original buffer ends with a partial token at the end
        special_tokens = ["<|python_start|>", "<|python_end|>"]

        for token in special_tokens:
            partial_length = self._ends_with_partial_token(buffer, token)
            if partial_length > 0:
                # Split buffer: safe part + held back partial token
                safe_text = buffer[:-partial_length]
                held_back = buffer[-partial_length:]
                # Strip complete special tokens from safe part only
                safe_text = self._text_strip(safe_text)
                return safe_text, held_back

        # No partial tokens found, strip complete tokens from entire buffer
        safe_text = self._text_strip(buffer)
        return safe_text, ""
```
**EN:** This block defines the method `_strip_and_split_buffer` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `_strip_and_split_buffer`. Notable operations include `of`, `_ends_with_partial_token`, `_text_strip`.
**CN:** 这一段定义了method `_strip_and_split_buffer`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_strip_and_split_buffer`。 值得注意的操作包括 `of`、`_ends_with_partial_token`、`_text_strip`。

### Lines 159-166: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for pythonic tool calls.
        Buffers input until a complete pythonic tool call (from [ to ]) is found,
        then parses and emits any detected calls.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `call`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `call`。

### Lines 167-189: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        # Strip special tokens from entire buffer and handle partial tokens
        stripped_buffer, held_back = self._strip_and_split_buffer(self._buffer)

        start = stripped_buffer.find("[")

        if start == -1:
            # No tool call bracket found
            self._buffer = held_back
            return StreamingParseResult(normal_text=stripped_buffer)

        normal_text = stripped_buffer[:start] if start > 0 else ""

        end = self._find_matching_bracket(stripped_buffer, start)
        if end != -1:
            # Found complete tool call
            call_text = stripped_buffer[start : end + 1]
            result = self.detect_and_parse(call_text, tools)

            # Update buffer with remaining text after tool call plus any held back text
            remaining_text = stripped_buffer[end + 1 :] + held_back
            self._buffer = remaining_text
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding pythonic style tool-call detection and normalization workflow. Notable operations include `_strip_and_split_buffer`, `find`, `StreamingParseResult`, `_find_matching_bracket`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Python 风格风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_strip_and_split_buffer`、`find`、`StreamingParseResult`、`_find_matching_bracket`。

### Lines 190-205: Method `parse_streaming_increment` logic (part 2)
```python

            # If we had normal text before the tool call, add it to the result
            if normal_text:
                result.normal_text = normal_text + (result.normal_text or "")

            return result

        # We have an opening bracket but no closing bracket yet
        # Put back everything from the bracket onwards plus held back text
        self._buffer = stripped_buffer[start:] + held_back

        if normal_text:
            return StreamingParseResult(normal_text=normal_text)

        # Otherwise, we're still accumulating a potential tool call
        return StreamingParseResult(normal_text="")
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding pythonic style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Python 风格风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`。

### Lines 207-218: Method `_get_parameter_value`
```python
    def _get_parameter_value(self, val):
        if isinstance(val, ast.Constant):
            return val.value
        elif isinstance(val, ast.Dict):
            return {
                k.value: self._get_parameter_value(v)
                for k, v in zip(val.keys, val.values)
            }
        elif isinstance(val, ast.List):
            return [self._get_parameter_value(v) for v in val.elts]
        else:
            raise ValueError("Tool call arguments must be literals")
```
**EN:** This block defines the method `_get_parameter_value` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `_get_parameter_value`. Notable operations include `_get_parameter_value`, `ValueError`.
**CN:** 这一段定义了method `_get_parameter_value`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_parameter_value`。 值得注意的操作包括 `_get_parameter_value`、`ValueError`。

### Lines 220-221: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 223-224: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError
```
**EN:** This block defines the method `structure_info` on `PythonicDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of pythonic style tool-call detection and normalization. Definitions introduced here include `structure_info`.
**CN:** 这一段定义了method `structure_info`（属于 `PythonicDetector`），介绍了参数、初始化步骤，以及这部分Python 风格风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。

## Key Concepts / 关键概念
- `PythonicDetector`: Class that encapsulates pythonic detector behavior in this module. / `PythonicDetector`：封装与“Python 风格检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
