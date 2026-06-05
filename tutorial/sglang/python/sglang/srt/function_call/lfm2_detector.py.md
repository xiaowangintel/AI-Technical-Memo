# lfm2_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/lfm2_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a lfm 2 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 lfm2 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Module-level constants and helper logic
```python
"""
Detector for LFM2 (Liquid Foundation Model 2) function call format.

Format Structure (Pythonic style):
```
<|tool_call_start|>[function_name(arg1="value1", arg2="value2")]<|tool_call_end|>
```

Multiple tool calls:
```
<|tool_call_start|>[func1(arg="val"), func2(arg="val")]<|tool_call_end|>
```

Also supports JSON format:
```
<|tool_call_start|>[{"name": "func_name", "arguments": {...}}]<|tool_call_end|>
```
"""

import ast
import json
import logging
import re
from typing import Any, Dict, List, Optional, Tuple

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for lfm 2 style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `LFM2`, `Structure`, `function_name`, `func1`.
**CN:** 这一段包含与lfm2风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `LFM2`、`Structure`、`function_name`、`func1`。

### Lines 39-39: Class `Lfm2Detector` declaration
```python
class Lfm2Detector(BaseFormatDetector):
```
**EN:** This block declares the class `Lfm2Detector` and establishes its responsibility inside lfm 2 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Lfm2Detector`.
**CN:** 这一段声明了类 `Lfm2Detector`，并说明它在lfm2风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Lfm2Detector`。

### Lines 40-55: Supporting state inside `Lfm2Detector`
```python
    """
    Detector for LFM2 (Liquid Foundation Model 2) function call format.

    Supports both Pythonic and JSON formats:

    Pythonic:
    ```
    <|tool_call_start|>[calculator(expression="5 * 7")]<|tool_call_end|>
    ```

    JSON:
    ```
    <|tool_call_start|>[{"name": "calculator", "arguments": {"expression": "5 * 7"}}]<|tool_call_end|>
    ```
    """

```
**EN:** This block adds supporting state or helper logic inside `Lfm2Detector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `LFM2`, `calculator`.
**CN:** 这一段为 `Lfm2Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `LFM2`、`calculator`。

### Lines 56-63: Method `__init__`
```python
    def __init__(self):
        """
        Initializes the detector with necessary state variables.
        """
        super().__init__()
        self.bot_token = "<|tool_call_start|>"
        self.eot_token = "<|tool_call_end|>"
        self.tool_call_separator = ""
```
**EN:** This block defines the method `__init__` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 65-67: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains an LFM2 format tool call."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 69-75: Method `_get_parameter_value` signature and setup
```python
    def _get_parameter_value(self, val: ast.AST) -> Any:
        """
        Extract Python literal value from AST node.

        Handles constants, dicts, and lists recursively.
        Reuses pattern from PythonicDetector.
        """
```
**EN:** This block defines the method `_get_parameter_value` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_get_parameter_value`.
**CN:** 这一段定义了method `_get_parameter_value`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_parameter_value`。

### Lines 76-99: Method `_get_parameter_value` logic (part 1)
```python
        if isinstance(val, ast.Constant):
            return val.value
        elif isinstance(val, ast.Dict):
            return {
                self._get_parameter_value(k): self._get_parameter_value(v)
                for k, v in zip(val.keys, val.values)
                if k is not None  # Handle {**kwargs} case where key is None
            }
        elif isinstance(val, ast.List):
            return [self._get_parameter_value(v) for v in val.elts]
        elif isinstance(val, ast.Tuple):
            return tuple(self._get_parameter_value(v) for v in val.elts)
        elif isinstance(val, ast.Name):
            # Handle True, False, None as names in older Python
            if val.id == "True":
                return True
            elif val.id == "False":
                return False
            elif val.id == "None":
                return None
            else:
                raise ValueError(f"Unsupported name reference: {val.id}")
        elif isinstance(val, ast.UnaryOp) and isinstance(val.op, ast.USub):
            # Handle negative numbers like -5
```
**EN:** This block continues `_get_parameter_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `_get_parameter_value`, `ValueError`.
**CN:** 这一段延续了 `_get_parameter_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_parameter_value`、`ValueError`。

### Lines 100-107: Method `_get_parameter_value` logic (part 2)
```python
            inner = self._get_parameter_value(val.operand)
            if isinstance(inner, (int, float)):
                return -inner
            raise ValueError(f"Cannot negate non-numeric value: {inner}")
        else:
            raise ValueError(
                f"Tool call arguments must be literals, got: {type(val).__name__}"
            )
```
**EN:** This block continues `_get_parameter_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `_get_parameter_value`, `ValueError`.
**CN:** 这一段延续了 `_get_parameter_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_parameter_value`、`ValueError`。

### Lines 109-122: Method `_parse_pythonic_call` signature and setup
```python
    def _parse_pythonic_call(
        self, call: ast.Call, call_index: int, tool_indices: Dict[str, int]
    ) -> Optional[ToolCallItem]:
        """
        Parse a single AST Call node into a ToolCallItem.

        Args:
            call: AST Call node representing a function call
            call_index: Index of this call in the list of calls
            tool_indices: Mapping of tool names to their indices

        Returns:
            ToolCallItem if successful, None if the call should be skipped
        """
```
**EN:** This block defines the method `_parse_pythonic_call` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_parse_pythonic_call`.
**CN:** 这一段定义了method `_parse_pythonic_call`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_pythonic_call`。

### Lines 123-150: Method `_parse_pythonic_call` logic (part 1)
```python
        if not isinstance(call.func, ast.Name):
            logger.warning(
                f"Tool call function must be a simple name, got: {type(call.func).__name__}"
            )
            return None

        function_name = call.func.id

        # Validate that the function exists in the tools
        if function_name not in tool_indices:
            logger.warning(
                f"Model attempted to call undefined function: {function_name}"
            )
            if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                return None  # Skip unknown tools (default legacy behavior)

        # Parse arguments
        arguments = {}
        for keyword in call.keywords:
            if keyword.arg is None:
                # **kwargs unpacking - skip for now
                logger.warning("Tool call with **kwargs unpacking is not supported")
                continue
            try:
                arguments[keyword.arg] = self._get_parameter_value(keyword.value)
            except ValueError as e:
                logger.warning(f"Failed to parse argument {keyword.arg}: {e}")
                return None
```
**EN:** This block continues `_parse_pythonic_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `warning`, `get`, `tools`, `_get_parameter_value`.
**CN:** 这一段延续了 `_parse_pythonic_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `warning`、`get`、`tools`、`_get_parameter_value`。

### Lines 151-156: Method `_parse_pythonic_call` logic (part 2)
```python

        return ToolCallItem(
            tool_index=call_index,  # Use the call index in the response, not tool position
            name=function_name,
            parameters=json.dumps(arguments, ensure_ascii=False),
        )
```
**EN:** This block continues `_parse_pythonic_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `ToolCallItem`, `dumps`.
**CN:** 这一段延续了 `_parse_pythonic_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `ToolCallItem`、`dumps`。

### Lines 158-170: Method `_parse_pythonic_content` signature and setup
```python
    def _parse_pythonic_content(
        self, content: str, tools: List[Tool]
    ) -> Tuple[List[ToolCallItem], str]:
        """
        Parse Pythonic format tool calls using AST.

        Args:
            content: The content between tool call tags (without the tags)
            tools: List of available tools

        Returns:
            Tuple of (list of parsed calls, error message if any)
        """
```
**EN:** This block defines the method `_parse_pythonic_content` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_parse_pythonic_content`. Notable operations include `tags`, `of`.
**CN:** 这一段定义了method `_parse_pythonic_content`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_pythonic_content`。 值得注意的操作包括 `tags`、`of`。

### Lines 171-194: Method `_parse_pythonic_content` logic (part 1)
```python
        content = content.strip()
        tool_indices = self._get_tool_indices(tools)

        try:
            module = ast.parse(content)
            parsed = getattr(module.body[0], "value", None) if module.body else None

            if parsed is None:
                return [], "Empty or invalid Python expression"

            # Handle both single call and list of calls
            if isinstance(parsed, ast.List):
                call_nodes = parsed.elts
            elif isinstance(parsed, ast.Call):
                call_nodes = [parsed]
            else:
                return (
                    [],
                    f"Expected function call or list, got: {type(parsed).__name__}",
                )

            # Validate all elements are calls
            if not all(isinstance(e, ast.Call) for e in call_nodes):
                return [], "Not all elements in list are function calls"
```
**EN:** This block continues `_parse_pythonic_content` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `strip`, `_get_tool_indices`, `parse`.
**CN:** 这一段延续了 `_parse_pythonic_content` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`_get_tool_indices`、`parse`。

### Lines 195-208: Method `_parse_pythonic_content` logic (part 2)
```python

            calls = []
            for call_index, call in enumerate(call_nodes):
                item = self._parse_pythonic_call(call, call_index, tool_indices)
                if item is not None:
                    calls.append(item)

            return calls, ""

        except SyntaxError as e:
            return [], f"Python syntax error: {e}"
        except Exception as e:
            logger.exception("Unexpected error in pythonic tool call parsing")
            return [], f"Unexpected error: {e}"
```
**EN:** This block continues `_parse_pythonic_content` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `_parse_pythonic_call`, `append`, `exception`.
**CN:** 这一段延续了 `_parse_pythonic_content` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_parse_pythonic_call`、`append`、`exception`。

### Lines 210-225: Method `_parse_json_content` signature and setup
```python
    def _parse_json_content(
        self, content: str, tools: List[Tool]
    ) -> Tuple[List[ToolCallItem], str]:
        """
        Parse JSON format tool calls.

        Uses parse_base_json from BaseFormatDetector for consistent handling
        of SGLANG_FORWARD_UNKNOWN_TOOLS and tool validation.

        Args:
            content: The content between tool call tags (without the tags)
            tools: List of available tools

        Returns:
            Tuple of (list of parsed calls, error message if any)
        """
```
**EN:** This block defines the method `_parse_json_content` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_parse_json_content`. Notable operations include `tags`, `of`.
**CN:** 这一段定义了method `_parse_json_content`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_json_content`。 值得注意的操作包括 `tags`、`of`。

### Lines 226-236: Method `_parse_json_content` logic (part 1)
```python
        content = content.strip()

        try:
            parsed = json.loads(content)
            # parse_base_json handles list/dict normalization, tool validation,
            # and SGLANG_FORWARD_UNKNOWN_TOOLS consistently with other detectors
            calls = self.parse_base_json(parsed, tools)
            return calls, ""

        except json.JSONDecodeError as e:
            return [], f"JSON parse error: {e}"
```
**EN:** This block continues `_parse_json_content` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `strip`, `loads`, `parse_base_json`.
**CN:** 这一段延续了 `_parse_json_content` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`loads`、`parse_base_json`。

### Lines 238-244: Method `_parse_tool_calls_content` signature and setup
```python
    def _parse_tool_calls_content(
        self, content: str, tools: List[Tool]
    ) -> List[ToolCallItem]:
        """
        Parse the content between tool call tags.
        Handles both JSON and Pythonic formats.
        """
```
**EN:** This block defines the method `_parse_tool_calls_content` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_parse_tool_calls_content`.
**CN:** 这一段定义了method `_parse_tool_calls_content`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_tool_calls_content`。

### Lines 245-264: Method `_parse_tool_calls_content` logic (part 1)
```python
        content = content.strip()

        # First, try JSON format (faster check)
        if content.startswith("[{") or content.startswith("{"):
            calls, error = self._parse_json_content(content, tools)
            if calls:
                return calls
            # If JSON parsing failed but it looked like JSON, log the error
            if error:
                logger.debug(f"JSON parsing failed: {error}, trying Pythonic format")

        # Try Pythonic format
        calls, error = self._parse_pythonic_content(content, tools)
        if calls:
            return calls

        if error:
            logger.warning(f"Failed to parse tool calls: {error}")

        return []
```
**EN:** This block continues `_parse_tool_calls_content` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `strip`, `format`, `startswith`, `_parse_json_content`.
**CN:** 这一段延续了 `_parse_tool_calls_content` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`format`、`startswith`、`_parse_json_content`。

### Lines 266-285: Method `detect_and_parse`
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.
        """
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text

        if self.bot_token not in text:
            return StreamingParseResult(normal_text=normal_text, calls=[])

        # Find all <|tool_call_start|>...<|tool_call_end|> blocks
        pattern = rf"{re.escape(self.bot_token)}(.*?){re.escape(self.eot_token)}"
        match_result_list = re.findall(pattern, text, re.DOTALL)

        calls = []
        for match_result in match_result_list:
            parsed_calls = self._parse_tool_calls_content(match_result, tools)
            calls.extend(parsed_calls)

        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block defines the method `detect_and_parse` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `find`, `strip`, `StreamingParseResult`, `escape`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `find`、`strip`、`StreamingParseResult`、`escape`。

### Lines 287-289: Method `_strip_special_tokens`
```python
    def _strip_special_tokens(self, text: str) -> str:
        """Remove special tokens from text."""
        return text.replace(self.bot_token, "").replace(self.eot_token, "")
```
**EN:** This block defines the method `_strip_special_tokens` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `_strip_special_tokens`. Notable operations include `replace`.
**CN:** 这一段定义了method `_strip_special_tokens`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_strip_special_tokens`。 值得注意的操作包括 `replace`。

### Lines 291-303: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for LFM2 tool calls.

        This implementation properly handles Pythonic format by:
        1. Buffering until we see complete <|tool_call_start|>[...]<|tool_call_end|>
        2. Emitting normal text before tool calls immediately
        3. Parsing complete tool call blocks using detect_and_parse

        Based on PythonicDetector streaming logic.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 304-327: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        # Check for partial bot_token at the end
        partial_bot = self._ends_with_partial_token(self._buffer, self.bot_token)
        partial_eot = self._ends_with_partial_token(self._buffer, self.eot_token)

        # Find bot_token position
        bot_pos = self._buffer.find(self.bot_token)

        if bot_pos == -1:
            # No tool call start found
            if partial_bot:
                # Might be partial bot_token, hold back that part
                safe_text = self._buffer[:-partial_bot]
                self._buffer = self._buffer[-partial_bot:]
                return StreamingParseResult(normal_text=safe_text)
            else:
                # No tool call, emit all as normal text
                normal_text = self._strip_special_tokens(self._buffer)
                self._buffer = ""
                return StreamingParseResult(normal_text=normal_text)

        # We have bot_token - extract any normal text before it
        normal_text_before = self._buffer[:bot_pos] if bot_pos > 0 else ""
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `_ends_with_partial_token`, `find`, `StreamingParseResult`, `_strip_special_tokens`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_ends_with_partial_token`、`find`、`StreamingParseResult`、`_strip_special_tokens`。

### Lines 328-349: Method `parse_streaming_increment` logic (part 2)
```python

        # Look for the end token
        eot_pos = self._buffer.find(self.eot_token, bot_pos + len(self.bot_token))

        if eot_pos == -1:
            # No end token yet - check if we might have a partial one
            if partial_eot:
                # Hold back the partial token, but we need to keep buffering
                # Just emit any normal text before the tool call
                if normal_text_before:
                    self._buffer = self._buffer[bot_pos:]
                    return StreamingParseResult(normal_text=normal_text_before)
                # Keep buffering
                return StreamingParseResult(normal_text="")

            # No end token and no partial - keep buffering but emit normal text
            if normal_text_before:
                self._buffer = self._buffer[bot_pos:]
                return StreamingParseResult(normal_text=normal_text_before)

            # Just keep buffering
            return StreamingParseResult(normal_text="")
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`。

### Lines 350-365: Method `parse_streaming_increment` logic (part 3)
```python

        # We have a complete tool call block
        tool_call_block = self._buffer[bot_pos : eot_pos + len(self.eot_token)]
        remaining = self._buffer[eot_pos + len(self.eot_token) :]

        # Parse the complete block
        result = self.detect_and_parse(tool_call_block, tools)

        # Update buffer with remaining text
        self._buffer = remaining

        # Add any normal text before the tool call
        if normal_text_before:
            result.normal_text = normal_text_before + (result.normal_text or "")

        return result
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding lfm 2 style tool-call detection and normalization workflow. Notable operations include `detect_and_parse`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的lfm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `detect_and_parse`。

### Lines 367-374: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        """
        Return False because LFM2 uses Pythonic format which is not JSON-compatible.

        structural_tag only supports JSON-compatible content between begin and end,
        so it cannot parse Pythonic function call syntax like `func(arg="val")`.
        """
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`. Notable operations include `func`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。 值得注意的操作包括 `func`。

### Lines 376-387: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        """
        Return structure info for constrained generation.

        Note: This is provided for completeness but won't be used since
        supports_structural_tag() returns False.
        """
        return lambda name: StructureInfo(
            begin="<|tool_call_start|>[" + name + "(",
            end=")]<|tool_call_end|>",
            trigger="<|tool_call_start|>",
        )
```
**EN:** This block defines the method `structure_info` on `Lfm2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of lfm 2 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `supports_structural_tag`, `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `Lfm2Detector`），介绍了参数、初始化步骤，以及这部分lfm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `supports_structural_tag`、`StructureInfo`。

## Key Concepts / 关键概念
- `Lfm2Detector`: Class that encapsulates lfm 2 detector behavior in this module. / `Lfm2Detector`：封装与“lfm2检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
