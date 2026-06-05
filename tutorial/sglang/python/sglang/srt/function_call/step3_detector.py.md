# step3_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/step3_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a step 3 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 步骤3 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import ast
import json
import logging
import re
from typing import Any, Dict, List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for step 3 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与步骤3风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 18-28: Function `get_argument_type`
```python
def get_argument_type(func_name: str, arg_key: str, defined_tools: List[Tool]) -> str:
    """Get the expected type for a function argument from tool schema."""
    name2tool = {tool.function.name: tool for tool in defined_tools}
    if func_name not in name2tool:
        return None
    tool = name2tool[func_name]
    parameters = tool.function.parameters or {}
    properties = parameters.get("properties", {})
    if arg_key not in properties:
        return None
    return properties[arg_key].get("type", None)
```
**EN:** This block defines the function `get_argument_type`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `get_argument_type`. Notable operations include `get`.
**CN:** 这一段定义了function `get_argument_type`，介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_argument_type`。 值得注意的操作包括 `get`。

### Lines 31-40: Function `parse_arguments`
```python
def parse_arguments(value: str) -> tuple[Any, bool]:
    """Parse a string value to appropriate type. Returns (parsed_value, success)."""
    try:
        try:
            parsed_value = json.loads(value)
        except:
            parsed_value = ast.literal_eval(value)
        return parsed_value, True
    except:
        return value, False
```
**EN:** This block defines the function `parse_arguments`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `parse_arguments`. Notable operations include `Returns`, `loads`, `literal_eval`.
**CN:** 这一段定义了function `parse_arguments`，介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_arguments`。 值得注意的操作包括 `Returns`、`loads`、`literal_eval`。

### Lines 43-43: Class `Step3Detector` declaration
```python
class Step3Detector(BaseFormatDetector):
```
**EN:** This block declares the class `Step3Detector` and establishes its responsibility inside step 3 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Step3Detector`.
**CN:** 这一段声明了类 `Step3Detector`，并说明它在步骤3风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Step3Detector`。

### Lines 44-60: Supporting state inside `Step3Detector`
```python
    """
    Detector for Step3 model function call format.

    The Step3 format uses special Unicode tokens to delimit function calls
    with steptml XML format for invocations.

    Format Structure:
    ```
    <｜tool_calls_begin｜>
    <｜tool_call_begin｜>function<｜tool_sep｜><steptml:invoke name="function_name">
    <steptml:parameter name="param1">value1</steptml:parameter>
    <steptml:parameter name="param2">value2</steptml:parameter>
    </steptml:invoke><｜tool_call_end｜>
    <｜tool_calls_end｜>
    ```
    """

```
**EN:** This block adds supporting state or helper logic inside `Step3Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `Step3Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 61-83: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<｜tool_calls_begin｜>"
        self.eot_token = "<｜tool_calls_end｜>"
        self.tool_call_begin = "<｜tool_call_begin｜>"
        self.tool_call_end = "<｜tool_call_end｜>"
        self.tool_sep = "<｜tool_sep｜>"

        # Regex for parsing steptml invocations
        self.invoke_regex = re.compile(
            r'<steptml:invoke name="([^"]+)">(.+?)</steptml:invoke>', re.DOTALL
        )
        self.param_regex = re.compile(
            r'<steptml:parameter name="([^"]+)">([^<]*)</steptml:parameter>', re.DOTALL
        )

        # Streaming state variables
        self._in_tool_block: bool = False
        self._tool_block_finished: bool = False
        self._current_function_name: str = ""
        self._current_parameters: Dict[str, Any] = {}
        self._in_tool_call: bool = False
        self._function_name_sent: bool = False
```
**EN:** This block defines the method `__init__` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `compile`.
**CN:** 这一段定义了method `__init__`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`compile`。

### Lines 85-87: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a Step3 format tool call."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 89-92: Method `_parse_steptml_invoke` signature and setup
```python
    def _parse_steptml_invoke(
        self, text: str, tools: List[Tool] = None
    ) -> tuple[str, dict]:
        """Parse steptml invoke format to extract function name and parameters."""
```
**EN:** This block defines the method `_parse_steptml_invoke` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `_parse_steptml_invoke`.
**CN:** 这一段定义了method `_parse_steptml_invoke`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_steptml_invoke`。

### Lines 93-116: Method `_parse_steptml_invoke` logic (part 1)
```python
        invoke_match = self.invoke_regex.search(text)
        if not invoke_match:
            return None, {}

        func_name = invoke_match.group(1)
        params_text = invoke_match.group(2)

        params = {}
        for param_match in self.param_regex.finditer(params_text):
            param_name = param_match.group(1)
            param_value = param_match.group(2).strip()

            # If tools provided, use schema-aware parsing
            if tools:
                arg_type = get_argument_type(func_name, param_name, tools)
                if arg_type and arg_type != "string":
                    parsed_value, _ = parse_arguments(param_value)
                    params[param_name] = parsed_value
                else:
                    params[param_name] = param_value
            else:
                # Fallback to generic parsing if no tools provided
                parsed_value, _ = parse_arguments(param_value)
                params[param_name] = parsed_value
```
**EN:** This block continues `_parse_steptml_invoke` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `search`, `group`, `finditer`, `strip`.
**CN:** 这一段延续了 `_parse_steptml_invoke` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`group`、`finditer`、`strip`。

### Lines 117-118: Method `_parse_steptml_invoke` logic (part 2)
```python

        return func_name, params
```
**EN:** This block continues `_parse_steptml_invoke` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `_parse_steptml_invoke` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。

### Lines 120-123: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.
        """
```
**EN:** This block defines the method `detect_and_parse` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 124-147: Method `detect_and_parse` logic (part 1)
```python
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=text, calls=[])

        try:
            pre_text, rest = text.split(self.bot_token, 1)

            # If no end token, return everything as normal text
            if self.eot_token not in rest:
                return StreamingParseResult(normal_text=text, calls=[])

            tool_section, post_text = rest.split(self.eot_token, 1)

            # Find all individual tool calls using regex
            calls = []
            tool_call_pattern = (
                f"{re.escape(self.tool_call_begin)}(.*?){re.escape(self.tool_call_end)}"
            )

            for match in re.finditer(tool_call_pattern, tool_section, re.DOTALL):
                call_content = match.group(1)

                # Check if it's a function call
                if self.tool_sep not in call_content:
                    continue
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `split`, `escape`, `finditer`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`split`、`escape`、`finditer`。

### Lines 148-167: Method `detect_and_parse` logic (part 2)
```python

                type_part, invoke_part = call_content.split(self.tool_sep, 1)
                if type_part.strip() != "function":
                    continue

                func_name, params = self._parse_steptml_invoke(invoke_part, tools)
                if func_name:
                    # Use parse_base_json to create the ToolCallItem
                    action = {"name": func_name, "arguments": params}
                    calls.extend(self.parse_base_json(action, tools))

            # Combine pre and post text
            normal_text = pre_text + post_text

            return StreamingParseResult(normal_text=normal_text, calls=calls)

        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}")
            # Return the original text if parsing fails
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `split`, `strip`, `_parse_steptml_invoke`, `extend`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `split`、`strip`、`_parse_steptml_invoke`、`extend`。

### Lines 169-174: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for Step3 format.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 175-205: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        # Build tool indices for validation
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        # If we've finished the tool block, everything is normal text
        if self._tool_block_finished:
            normal_text = self._buffer
            self._buffer = ""
            return StreamingParseResult(normal_text=normal_text)

        # Check if tool block hasn't started yet
        if not self._in_tool_block:
            if self.bot_token in self._buffer:
                idx = self._buffer.find(self.bot_token)
                normal_text = self._buffer[:idx]
                self._buffer = self._buffer[idx + len(self.bot_token) :]
                self._in_tool_block = True
                return StreamingParseResult(normal_text=normal_text)
            else:
                # Check if we might have a partial bot_token
                partial_len = self._ends_with_partial_token(
                    self._buffer, self.bot_token
                )
                if partial_len:
                    return StreamingParseResult()  # Wait for more text
                else:
                    normal_text = self._buffer
                    self._buffer = ""
                    return StreamingParseResult(normal_text=normal_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `_get_tool_indices`, `StreamingParseResult`, `find`, `_ends_with_partial_token`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_get_tool_indices`、`StreamingParseResult`、`find`、`_ends_with_partial_token`。

### Lines 206-233: Method `parse_streaming_increment` logic (part 2)
```python

        # We're inside the tool block
        calls: List[ToolCallItem] = []

        # Check if tool block is ending
        if self.eot_token in self._buffer:
            idx = self._buffer.find(self.eot_token)

            # If we're in the middle of a tool call, we need to handle it
            if self._in_tool_call:
                # The buffer before eot_token might contain the end of the current tool call
                before_eot = self._buffer[:idx]
                if self.tool_call_end in before_eot:
                    # Parse this final tool call
                    result = self._parse_partial_tool_call(tools)
                    calls.extend(result.calls)
                else:
                    # Incomplete tool call - log warning
                    logger.warning("Tool block ended with incomplete tool call")

            remaining = self._buffer[idx + len(self.eot_token) :]
            self._buffer = ""
            self._tool_block_finished = True

            # Reset any partial tool call state
            self._reset_streaming_state()

            return StreamingParseResult(normal_text=remaining, calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `find`, `_parse_partial_tool_call`, `extend`, `warning`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`_parse_partial_tool_call`、`extend`、`warning`。

### Lines 234-254: Method `parse_streaming_increment` logic (part 3)
```python

        # Check if we're in a tool call or need to start one
        if not self._in_tool_call:
            if self.tool_call_begin in self._buffer:
                idx = self._buffer.find(self.tool_call_begin)
                # Remove any content before tool call begin (shouldn't happen but be safe)
                self._buffer = self._buffer[idx + len(self.tool_call_begin) :]
                self._in_tool_call = True
                self._function_name_sent = False
                self._current_function_name = ""
                self._current_parameters = {}
                # Fall through to parse the partial tool call
            else:
                # Wait for tool call to begin
                return StreamingParseResult()

        # Parse partial tool call
        if self._in_tool_call:
            return self._parse_partial_tool_call(tools)

        return StreamingParseResult()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `find`, `begin`, `StreamingParseResult`, `_parse_partial_tool_call`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`begin`、`StreamingParseResult`、`_parse_partial_tool_call`。

### Lines 256-257: Method `_parse_partial_tool_call` signature and setup
```python
    def _parse_partial_tool_call(self, tools: List[Tool]) -> StreamingParseResult:
        """Parse partial tool call for streaming scenarios."""
```
**EN:** This block defines the method `_parse_partial_tool_call` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `_parse_partial_tool_call`.
**CN:** 这一段定义了method `_parse_partial_tool_call`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_partial_tool_call`。

### Lines 258-289: Method `_parse_partial_tool_call` logic (part 1)
```python
        calls = []

        # Check if we have tool_sep (means we're past the type declaration)
        if self.tool_sep not in self._buffer:
            return StreamingParseResult(calls=calls)  # Wait for more text

        type_part, invoke_part = self._buffer.split(self.tool_sep, 1)
        if type_part.strip() != "function":
            # Invalid tool type, skip this tool call
            self._reset_streaming_state()
            return StreamingParseResult(calls=calls)

        # Try to extract function name if not sent yet
        if not self._function_name_sent:
            name_match = re.search(r'<steptml:invoke name="([^"]+)">', invoke_part)
            if name_match:
                func_name = name_match.group(1)

                # Validate function name
                if func_name in self._tool_indices:
                    self._current_function_name = func_name
                    self._function_name_sent = True

                    # Initialize tool tracking
                    if self.current_tool_id == -1:
                        self.current_tool_id = 0

                    # Ensure tracking arrays are large enough
                    while len(self.prev_tool_call_arr) <= self.current_tool_id:
                        self.prev_tool_call_arr.append({})
                    while len(self.streamed_args_for_tool) <= self.current_tool_id:
                        self.streamed_args_for_tool.append("")
```
**EN:** This block continues `_parse_partial_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `tool_sep`, `StreamingParseResult`, `split`, `strip`.
**CN:** 这一段延续了 `_parse_partial_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `tool_sep`、`StreamingParseResult`、`split`、`strip`。

### Lines 290-320: Method `_parse_partial_tool_call` logic (part 2)
```python

                    # Store tool call info
                    self.prev_tool_call_arr[self.current_tool_id] = {
                        "name": func_name,
                        "arguments": {},
                    }

                    # Send tool name with empty parameters
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=func_name,
                            parameters="",
                        )
                    )
                else:
                    # Invalid function name
                    logger.warning(f"Invalid function name: {func_name}")
                    self._reset_streaming_state()
                    return StreamingParseResult(calls=calls)
            else:
                # Function name not complete yet
                return StreamingParseResult(calls=calls)

        # Parse parameters incrementally
        if self._function_name_sent:
            # Extract all complete parameters
            new_params = {}
            for param_match in self.param_regex.finditer(invoke_part):
                param_name = param_match.group(1)
                param_value = param_match.group(2).strip()
```
**EN:** This block continues `_parse_partial_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `warning`, `_reset_streaming_state`.
**CN:** 这一段延续了 `_parse_partial_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`warning`、`_reset_streaming_state`。

### Lines 321-350: Method `_parse_partial_tool_call` logic (part 3)
```python

                # Use schema-aware parsing
                arg_type = get_argument_type(
                    self._current_function_name, param_name, tools
                )
                if arg_type and arg_type != "string":
                    parsed_value, _ = parse_arguments(param_value)
                    new_params[param_name] = parsed_value
                else:
                    new_params[param_name] = param_value

            # Check if we have new parameters to stream
            if new_params != self._current_parameters:
                # Build the JSON content without the closing brace for streaming
                if not self._current_parameters:
                    # First parameters - send opening brace and content
                    params_content = json.dumps(new_params, ensure_ascii=False)
                    if len(params_content) > 2:  # More than just "{}"
                        # Send everything except the closing brace
                        diff = params_content[:-1]
                    else:
                        diff = "{"
                else:
                    # Subsequent parameters - calculate the incremental diff
                    old_json = json.dumps(self._current_parameters, ensure_ascii=False)
                    new_json = json.dumps(new_params, ensure_ascii=False)

                    # Remove closing braces for comparison
                    old_without_brace = old_json[:-1]
                    new_without_brace = new_json[:-1]
```
**EN:** This block continues `_parse_partial_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `get_argument_type`, `parse_arguments`, `dumps`.
**CN:** 这一段延续了 `_parse_partial_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get_argument_type`、`parse_arguments`、`dumps`。

### Lines 351-382: Method `_parse_partial_tool_call` logic (part 4)
```python

                    # The new content should extend the old content
                    if new_without_brace.startswith(old_without_brace):
                        diff = new_without_brace[len(old_without_brace) :]
                    else:
                        # Parameters changed in unexpected way - shouldn't happen in normal streaming
                        diff = ""

                if diff:
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            parameters=diff,
                        )
                    )
                    self.streamed_args_for_tool[self.current_tool_id] += diff

                # Update current state
                self._current_parameters = new_params
                self.prev_tool_call_arr[self.current_tool_id]["arguments"] = new_params

            # Check if tool call is complete
            if self.tool_call_end in self._buffer:
                # Send closing brace if we've sent any parameters
                if self.streamed_args_for_tool[self.current_tool_id]:
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            parameters="}",
                        )
                    )
                    self.streamed_args_for_tool[self.current_tool_id] += "}"
```
**EN:** This block continues `_parse_partial_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `startswith`, `append`, `ToolCallItem`.
**CN:** 这一段延续了 `_parse_partial_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`append`、`ToolCallItem`。

### Lines 383-393: Method `_parse_partial_tool_call` logic (part 5)
```python

                # Find the end position
                end_idx = self._buffer.find(self.tool_call_end)
                # Remove the processed tool call from buffer
                self._buffer = self._buffer[end_idx + len(self.tool_call_end) :]

                # Reset state for next tool call
                self._reset_streaming_state()
                self.current_tool_id += 1

        return StreamingParseResult(calls=calls)
```
**EN:** This block continues `_parse_partial_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding step 3 style tool-call detection and normalization workflow. Notable operations include `find`, `_reset_streaming_state`, `StreamingParseResult`.
**CN:** 这一段延续了 `_parse_partial_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的步骤3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`_reset_streaming_state`、`StreamingParseResult`。

### Lines 395-400: Method `_reset_streaming_state`
```python
    def _reset_streaming_state(self):
        """Reset streaming state for the next tool call"""
        self._in_tool_call = False
        self._function_name_sent = False
        self._current_function_name = ""
        self._current_parameters = {}
```
**EN:** This block defines the method `_reset_streaming_state` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `_reset_streaming_state`.
**CN:** 这一段定义了method `_reset_streaming_state`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_reset_streaming_state`。

### Lines 402-404: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        """Return True if this detector supports structural tag format."""
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 406-407: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError()
```
**EN:** This block defines the method `structure_info` on `Step3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of step 3 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `structure_info`（属于 `Step3Detector`），介绍了参数、初始化步骤，以及这部分步骤3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `NotImplementedError`。

## Key Concepts / 关键概念
- `get_argument_type`: Function that performs get argument type for the surrounding workflow. / `get_argument_type`：在周边工作流中执行“getargument类型”相关任务的函数。
- `parse_arguments`: Function that performs parse arguments for the surrounding workflow. / `parse_arguments`：在周边工作流中执行“解析arguments”相关任务的函数。
- `Step3Detector`: Class that encapsulates step 3 detector behavior in this module. / `Step3Detector`：封装与“步骤3检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
