# minimax_m2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/minimax_m2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a minimax m 2 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 MiniMaxm2 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup
```python
import json
import logging
import re
from typing import Any, Dict, List, Tuple

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for minimax m 2 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与MiniMaxm2风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 17-17: Class `MinimaxM2Detector` declaration
```python
class MinimaxM2Detector(BaseFormatDetector):
```
**EN:** This block declares the class `MinimaxM2Detector` and establishes its responsibility inside minimax m 2 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MinimaxM2Detector`.
**CN:** 这一段声明了类 `MinimaxM2Detector`，并说明它在MiniMaxm2风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MinimaxM2Detector`。

### Lines 18-28: Supporting state inside `MinimaxM2Detector`
```python
    """
    Detector for MiniMax M2 models.
    Assumes function call format:
        <minimax:tool_call>
        <invoke name="func1">
        <parameter name="param1">value1</parameter>
        <parameter name="param2">value2</parameter>
        </invoke>
        </minimax:tool_call>
    """

```
**EN:** This block adds supporting state or helper logic inside `MinimaxM2Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MinimaxM2Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 29-31: Method `__init__` signature and setup
```python
    def __init__(self):
        super().__init__()
        self.tool_call_start_token: str = "<minimax:tool_call>"
```
**EN:** This block defines the method `__init__` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 32-54: Method `__init__` logic (part 1)
```python
        self.tool_call_end_token: str = "</minimax:tool_call>"
        self.tool_call_prefix: str = '<invoke name="'
        self.tool_call_function_end_token: str = "</invoke>"
        self.tool_call_regex = re.compile(
            r"<minimax:tool_call>(.*?)</minimax:tool_call>|<minimax:tool_call>(.*?)$",
            re.DOTALL,
        )
        self.tool_call_function_regex = re.compile(
            r"<invoke name=\"(.*?)</invoke>|<invoke name=\"(.*)$", re.DOTALL
        )
        self.tool_call_parameter_regex = re.compile(
            r"<parameter name=\"(.*?)</parameter>|<parameter name=\"(.*?)$", re.DOTALL
        )
        self._buf: str = ""

        # Streaming state variables
        self._current_function_name: str = ""
        self._current_parameters: Dict[str, Any] = {}
        self._streamed_parameters: Dict[str, str] = (
            {}
        )  # Track what parameter content we've streamed
        self._in_tool_call: bool = False
        self._function_name_sent: bool = False
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `compile`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `compile`。

### Lines 56-57: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.tool_call_start_token in text
```
**EN:** This block defines the method `has_tool_call` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 59-61: Method `detect_and_parse`
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        normal, calls = self._extract(text, tools)
        return StreamingParseResult(normal_text=normal, calls=calls)
```
**EN:** This block defines the method `detect_and_parse` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `_extract`, `StreamingParseResult`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `_extract`、`StreamingParseResult`。

### Lines 63-65: Method `_convert_param_value`
```python
    def _convert_param_value(self, value: str, param_type: str) -> Any:
        """Convert parameter value to the correct type (legacy single-type version)."""
        return self._convert_param_value_with_types(value, [param_type])
```
**EN:** This block defines the method `_convert_param_value` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_convert_param_value`. Notable operations include `_convert_param_value_with_types`.
**CN:** 这一段定义了method `_convert_param_value`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_param_value`。 值得注意的操作包括 `_convert_param_value_with_types`。

### Lines 67-77: Method `_extract_types_from_schema` signature and setup
```python
    def _extract_types_from_schema(self, schema: Any) -> list[str]:
        """
        Extract all possible types from a JSON schema definition.
        Handles anyOf, oneOf, allOf, type arrays, and enum fields.

        Args:
            schema: The JSON schema definition for a parameter

        Returns:
            List of type strings (e.g., ["string", "integer", "null"])
        """
```
**EN:** This block defines the method `_extract_types_from_schema` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_extract_types_from_schema`. Notable operations include `strings`.
**CN:** 这一段定义了method `_extract_types_from_schema`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract_types_from_schema`。 值得注意的操作包括 `strings`。

### Lines 78-94: Method `_extract_types_from_schema` logic (part 1)
```python
        if schema is None:
            return ["string"]

        if not isinstance(schema, dict):
            return ["string"]

        types: set[str] = set()

        # Handle direct "type" field
        if "type" in schema:
            type_value = schema["type"]
            if isinstance(type_value, str):
                types.add(type_value)
            elif isinstance(type_value, list):
                for t in type_value:
                    if isinstance(t, str):
                        types.add(t)
```
**EN:** This block continues `_extract_types_from_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `add`.
**CN:** 这一段延续了 `_extract_types_from_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `add`。

### Lines 95-119: Method `_extract_types_from_schema` logic (part 2)
```python

        # Handle enum - infer types from enum values
        if "enum" in schema and isinstance(schema["enum"], list) and schema["enum"]:
            for value in schema["enum"]:
                if value is None:
                    types.add("null")
                elif isinstance(value, bool):
                    types.add("boolean")
                elif isinstance(value, int):
                    types.add("integer")
                elif isinstance(value, float):
                    types.add("number")
                elif isinstance(value, str):
                    types.add("string")
                elif isinstance(value, list):
                    types.add("array")
                elif isinstance(value, dict):
                    types.add("object")

        # Handle anyOf, oneOf, allOf - recursively extract types
        for choice_field in ("anyOf", "oneOf", "allOf"):
            if choice_field in schema and isinstance(schema[choice_field], list):
                for choice in schema[choice_field]:
                    extracted = self._extract_types_from_schema(choice)
                    types.update(extracted)
```
**EN:** This block continues `_extract_types_from_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `add`, `in`, `_extract_types_from_schema`, `update`.
**CN:** 这一段延续了 `_extract_types_from_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `add`、`in`、`_extract_types_from_schema`、`update`。

### Lines 120-125: Method `_extract_types_from_schema` logic (part 3)
```python

        # If no types found, default to string
        if not types:
            return ["string"]

        return list(types)
```
**EN:** This block continues `_extract_types_from_schema` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_extract_types_from_schema` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。

### Lines 127-140: Method `_convert_param_value_with_types` signature and setup
```python
    def _convert_param_value_with_types(
        self, value: str, param_types: list[str]
    ) -> Any:
        """
        Convert parameter value to the correct type based on a list of possible types.
        Tries each type in order until one succeeds.

        Args:
            value: The string value to convert
            param_types: List of possible type strings

        Returns:
            The converted value
        """
```
**EN:** This block defines the method `_convert_param_value_with_types` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_convert_param_value_with_types`.
**CN:** 这一段定义了method `_convert_param_value_with_types`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_param_value_with_types`。

### Lines 141-165: Method `_convert_param_value_with_types` logic (part 1)
```python
        if value.lower() == "null":
            return None

        # Normalize types
        normalized_types = [t.lower() for t in param_types]

        # Try null first if it's in the list
        if "null" in normalized_types or value.lower() in ("null", "none", "nil"):
            return None

        # Try each type in order of preference (most specific first, string as fallback)
        # Priority: integer > number > boolean > object > array > string
        type_priority = [
            "integer",
            "int",
            "number",
            "float",
            "boolean",
            "bool",
            "object",
            "array",
            "string",
            "str",
            "text",
        ]
```
**EN:** This block continues `_convert_param_value_with_types` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `lower`, `in`, `preference`.
**CN:** 这一段延续了 `_convert_param_value_with_types` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `lower`、`in`、`preference`。

### Lines 166-195: Method `_convert_param_value_with_types` logic (part 2)
```python

        for param_type in type_priority:
            if param_type not in normalized_types:
                continue

            if param_type in ["string", "str", "text"]:
                return value
            elif param_type in ["integer", "int"]:
                try:
                    return int(value)
                except (ValueError, TypeError):
                    continue
            elif param_type in ["number", "float"]:
                try:
                    val = float(value)
                    return val if val != int(val) else int(val)
                except (ValueError, TypeError):
                    continue
            elif param_type in ["boolean", "bool"]:
                lower_val = value.lower().strip()
                if lower_val in ["true", "1", "yes", "on"]:
                    return True
                elif lower_val in ["false", "0", "no", "off"]:
                    return False
                continue
            elif param_type in ["object", "array"]:
                try:
                    return json.loads(value)
                except json.JSONDecodeError:
                    continue
```
**EN:** This block continues `_convert_param_value_with_types` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `lower`, `strip`, `loads`.
**CN:** 这一段延续了 `_convert_param_value_with_types` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `lower`、`strip`、`loads`。

### Lines 196-201: Method `_convert_param_value_with_types` logic (part 3)
```python

        # Fallback: try JSON parse, then return as string
        try:
            return json.loads(value)
        except json.JSONDecodeError:
            return value
```
**EN:** This block continues `_convert_param_value_with_types` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `loads`.
**CN:** 这一段延续了 `_convert_param_value_with_types` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`。

### Lines 203-224: Method `_get_param_types_from_config`
```python
    def _get_param_types_from_config(
        self, param_name: str, param_config: dict
    ) -> list[str]:
        """
        Get parameter types from parameter configuration.
        Handles anyOf, oneOf, allOf, and direct type definitions.

        Args:
            param_name: The name of the parameter
            param_config: The properties dict from the tool schema

        Returns:
            List of type strings
        """
        if param_name not in param_config:
            return ["string"]

        param_schema = param_config[param_name]
        if not isinstance(param_schema, dict):
            return ["string"]

        return self._extract_types_from_schema(param_schema)
```
**EN:** This block defines the method `_get_param_types_from_config` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_get_param_types_from_config`. Notable operations include `_extract_types_from_schema`.
**CN:** 这一段定义了method `_get_param_types_from_config`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_param_types_from_config`。 值得注意的操作包括 `_extract_types_from_schema`。

### Lines 226-228: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
```
**EN:** This block defines the method `parse_streaming_increment` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 229-259: Method `parse_streaming_increment` logic (part 1)
```python
        self._buf += new_text
        normal = ""
        calls: List[ToolCallItem] = []

        # Build tool indices for validation
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        while True:
            # If we're not in a tool call and don't see a start token, return normal text
            if not self._in_tool_call and self.tool_call_start_token not in self._buf:
                normal += self._buf
                self._buf = ""
                break

            # Look for tool call start
            if not self._in_tool_call:
                s = self._buf.find(self.tool_call_start_token)
                if s == -1:
                    normal += self._buf
                    self._buf = ""
                    break

                normal += self._buf[:s]
                self._buf = self._buf[s:]

                self._in_tool_call = True
                self._function_name_sent = False
                self._current_function_name = ""
                self._current_parameters = {}
                self._streamed_parameters = {}
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `_get_tool_indices`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_tool_indices`、`find`。

### Lines 260-291: Method `parse_streaming_increment` logic (part 2)
```python

                # Remove the start token
                self._buf = self._buf[len(self.tool_call_start_token) :]
                continue

            # We're in a tool call, try to parse function name if not sent yet
            if not self._function_name_sent:
                # Look for function name pattern: <invoke name=name>
                function_match = re.search(r"<invoke name=\"([^>]+)\">", self._buf)
                if function_match:
                    function_name = function_match.group(1).strip()

                    # Validate function name
                    if function_name in self._tool_indices:
                        self._current_function_name = function_name
                        self._function_name_sent = True

                        # Initialize tool call tracking
                        if self.current_tool_id == -1:
                            self.current_tool_id = 0

                        # Ensure tracking arrays are large enough
                        while len(self.prev_tool_call_arr) <= self.current_tool_id:
                            self.prev_tool_call_arr.append({})
                        while len(self.streamed_args_for_tool) <= self.current_tool_id:
                            self.streamed_args_for_tool.append("")

                        # Store tool call info
                        self.prev_tool_call_arr[self.current_tool_id] = {
                            "name": function_name,
                            "arguments": {},
                        }
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `search`, `group`, `strip`, `append`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`group`、`strip`、`append`。

### Lines 292-320: Method `parse_streaming_increment` logic (part 3)
```python

                        # Send tool name with empty parameters
                        calls.append(
                            ToolCallItem(
                                tool_index=self.current_tool_id,
                                name=function_name,
                                parameters="",
                            )
                        )

                        # Remove the processed function declaration
                        self._buf = self._buf[function_match.end() :]
                        continue
                    else:
                        # Invalid function name, reset state
                        logger.warning(f"Invalid function name: {function_name}")
                        self._reset_streaming_state()
                        normal += self._buf
                        self._buf = ""
                        break
                else:
                    # Function name not complete yet, wait for more text
                    break

            # Parse parameters incrementally
            if self._function_name_sent:
                # Process parameters and get any calls to emit
                parameter_calls = self._parse_and_stream_parameters(self._buf, tools)
                calls.extend(parameter_calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `end`, `warning`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`end`、`warning`。

### Lines 321-353: Method `parse_streaming_increment` logic (part 4)
```python

                # Check if tool call is complete
                if self.tool_call_function_end_token in self._buf:
                    end_pos = self._buf.find(self.tool_call_function_end_token)

                    # Add closing brace to complete the JSON object
                    current_streamed = self.streamed_args_for_tool[self.current_tool_id]
                    if current_streamed:
                        # Count opening and closing braces to check if JSON is complete
                        open_braces = current_streamed.count("{")
                        close_braces = current_streamed.count("}")
                        if open_braces > close_braces:
                            calls.append(
                                ToolCallItem(
                                    tool_index=self.current_tool_id,
                                    name=None,
                                    parameters="}",
                                )
                            )
                            self.streamed_args_for_tool[self.current_tool_id] = (
                                current_streamed + "}"
                            )

                    # Complete the tool call
                    self._buf = self._buf[
                        end_pos + len(self.tool_call_function_end_token) :
                    ]
                    self._reset_streaming_state(True)
                    self.current_tool_id += 1
                    continue
                else:
                    # Tool call not complete yet, wait for more text
                    break
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `find`, `count`, `append`, `ToolCallItem`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`count`、`append`、`ToolCallItem`。

### Lines 354-355: Method `parse_streaming_increment` logic (part 5)
```python

        return StreamingParseResult(normal_text=normal, calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`。

### Lines 357-374: Method `_parse_and_stream_parameters` signature and setup
```python
    def _parse_and_stream_parameters(
        self, text_to_parse: str, tools: List[Tool]
    ) -> List[ToolCallItem]:
        """
        Parse complete parameter blocks from text and return any tool call items to emit.

        This method:
        1. Finds all complete <parameter> blocks
        2. Parses them into a dictionary
        3. Compares with current parameters and generates diff if needed
        4. Updates internal state

        Args:
            text_to_parse: The text to search for parameter blocks

        Returns:
            List of ToolCallItem objects to emit (may be empty)
        """
```
**EN:** This block defines the method `_parse_and_stream_parameters` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_parse_and_stream_parameters`. Notable operations include `emit`.
**CN:** 这一段定义了method `_parse_and_stream_parameters`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_and_stream_parameters`。 值得注意的操作包括 `emit`。

### Lines 375-407: Method `_parse_and_stream_parameters` logic (part 1)
```python
        calls: List[ToolCallItem] = []

        # Find all complete parameter patterns
        param_matches = list(
            re.finditer(
                r"<parameter name=\"([^>]+)\">(.*?)</parameter>",
                text_to_parse,
                re.DOTALL,
            )
        )

        # Build new parameters dictionary
        new_params = {}
        for match in param_matches:
            param_name = match.group(1).strip()
            param_value = match.group(2)
            new_params[param_name] = self._parse_parameter(
                self._current_function_name, param_name, param_value, tools
            )

        # Calculate parameter diff to stream with proper incremental JSON building
        if new_params != self._current_parameters:
            previous_args_json = self.streamed_args_for_tool[self.current_tool_id]

            # Build incremental JSON properly
            if not self._current_parameters:
                # First parameter(s) - start JSON object but don't close it yet
                items = []
                for key, value in new_params.items():
                    items.append(
                        f"{json.dumps(key, ensure_ascii=False)}: {json.dumps(value, ensure_ascii=False)}"
                    )
                json_fragment = "{" + ", ".join(items)
```
**EN:** This block continues `_parse_and_stream_parameters` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `finditer`, `group`, `strip`, `_parse_parameter`.
**CN:** 这一段延续了 `_parse_and_stream_parameters` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `finditer`、`group`、`strip`、`_parse_parameter`。

### Lines 408-441: Method `_parse_and_stream_parameters` logic (part 2)
```python

                calls.append(
                    ToolCallItem(
                        tool_index=self.current_tool_id,
                        name=None,
                        parameters=json_fragment,
                    )
                )
                self.streamed_args_for_tool[self.current_tool_id] = json_fragment

            else:
                # Additional parameters - add them incrementally
                new_keys = set(new_params.keys()) - set(self._current_parameters.keys())
                if new_keys:
                    # Build the continuation part (no closing brace yet)
                    continuation_parts = []
                    for key in new_keys:
                        value = new_params[key]
                        continuation_parts.append(
                            f"{json.dumps(key, ensure_ascii=False)}: {json.dumps(value, ensure_ascii=False)}"
                        )

                    json_fragment = ", " + ", ".join(continuation_parts)

                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=None,
                            parameters=json_fragment,
                        )
                    )
                    self.streamed_args_for_tool[self.current_tool_id] = (
                        previous_args_json + json_fragment
                    )
```
**EN:** This block continues `_parse_and_stream_parameters` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `keys`, `part`.
**CN:** 这一段延续了 `_parse_and_stream_parameters` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`keys`、`part`。

### Lines 442-447: Method `_parse_and_stream_parameters` logic (part 3)
```python

            # Update current state
            self._current_parameters = new_params
            self.prev_tool_call_arr[self.current_tool_id]["arguments"] = new_params

        return calls
```
**EN:** This block continues `_parse_and_stream_parameters` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_parse_and_stream_parameters` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。

### Lines 449-456: Method `_reset_streaming_state`
```python
    def _reset_streaming_state(self, still_in_tool_call: bool = False):
        """Reset streaming state for the next tool call"""
        self._in_tool_call = still_in_tool_call
        self._function_name_sent = False
        self._current_function_name = ""
        self._current_parameters = {}
        self._streamed_parameters = {}
        self.current_tool_name_sent = False
```
**EN:** This block defines the method `_reset_streaming_state` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_reset_streaming_state`.
**CN:** 这一段定义了method `_reset_streaming_state`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_reset_streaming_state`。

### Lines 458-475: Method `_extract`
```python
    def _extract(self, text: str, tools: List[Tool]) -> Tuple[str, List[ToolCallItem]]:
        normal_parts: List[str] = []
        calls: List[ToolCallItem] = []
        cursor = 0
        while True:
            s = text.find(self.tool_call_start_token, cursor)
            if s == -1:
                normal_parts.append(text[cursor:])
                break
            normal_parts.append(text[cursor:s])
            e = text.find(self.tool_call_end_token, s)
            if e == -1:
                normal_parts.append(text[s:])
                break
            block = text[s : e + len(self.tool_call_end_token)]
            cursor = e + len(self.tool_call_end_token)
            calls.extend(self._parse_block(block, tools))
        return "".join(normal_parts), calls
```
**EN:** This block defines the method `_extract` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_extract`. Notable operations include `find`, `append`, `extend`, `_parse_block`.
**CN:** 这一段定义了method `_extract`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract`。 值得注意的操作包括 `find`、`append`、`extend`、`_parse_block`。

### Lines 477-479: Method `_parse_block` signature and setup
```python
    def _parse_block(self, block: str, tools: List[Tool]) -> List[ToolCallItem]:
        res: List[ToolCallItem] = []
        for m in self.tool_call_function_regex.findall(block):
```
**EN:** This block defines the method `_parse_block` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_parse_block`. Notable operations include `findall`.
**CN:** 这一段定义了method `_parse_block`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_block`。 值得注意的操作包括 `findall`。

### Lines 480-502: Method `_parse_block` logic (part 1)
```python
            txt = m[0] if m[0] else m[1]
            if '">' not in txt:
                continue
            idx = txt.index('">')
            fname = txt[:idx].strip()
            body = txt[idx + 2 :]
            params: Dict[str, Any] = {}
            for pm in self.tool_call_parameter_regex.findall(body):
                ptxt = pm[0] if pm[0] else pm[1]
                if '">' not in ptxt:
                    continue
                pidx = ptxt.index('">')
                pname = ptxt[:pidx].strip()
                pval = ptxt[pidx + 2 :].lstrip("\n").rstrip("\n")
                params[pname] = self._parse_parameter(fname, pname, pval, tools)
            raw = {"name": fname, "arguments": params}
            try:
                # TODO: fix idx in function call, the index for a function
                # call will always be -1 in parse_base_json
                res.extend(self.parse_base_json(raw, tools))
            except Exception:
                logger.warning("invalid tool call for %s dropped", fname)
        return res
```
**EN:** This block continues `_parse_block` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding minimax m 2 style tool-call detection and normalization workflow. Notable operations include `index`, `strip`, `findall`, `lstrip`.
**CN:** 这一段延续了 `_parse_block` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的MiniMaxm2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `index`、`strip`、`findall`、`lstrip`。

### Lines 504-516: Method `_parse_parameter`
```python
    def _parse_parameter(
        self, fname: str, pname: str, pval: str, tools: List[Tool]
    ) -> Any:
        param_config = {}
        for tool in tools:
            if tool.function.name == fname and tool.function.parameters is not None:
                parameters = tool.function.parameters
                if isinstance(parameters, dict) and "properties" in parameters:
                    param_config = parameters["properties"]
                    break

        param_type = self._get_param_types_from_config(pname, param_config)
        return self._convert_param_value_with_types(pval, param_type)
```
**EN:** This block defines the method `_parse_parameter` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `_parse_parameter`. Notable operations include `_get_param_types_from_config`, `_convert_param_value_with_types`.
**CN:** 这一段定义了method `_parse_parameter`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_parameter`。 值得注意的操作包括 `_get_param_types_from_config`、`_convert_param_value_with_types`。

### Lines 518-519: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 521-522: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError
```
**EN:** This block defines the method `structure_info` on `MinimaxM2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of minimax m 2 style tool-call detection and normalization. Definitions introduced here include `structure_info`.
**CN:** 这一段定义了method `structure_info`（属于 `MinimaxM2Detector`），介绍了参数、初始化步骤，以及这部分MiniMaxm2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。

## Key Concepts / 关键概念
- `MinimaxM2Detector`: Class that encapsulates minimax m 2 detector behavior in this module. / `MinimaxM2Detector`：封装与“MiniMaxm2检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
