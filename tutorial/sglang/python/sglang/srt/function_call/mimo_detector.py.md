# mimo_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/mimo_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a mimo detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 mimo 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Module-level constants and helper logic
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

import ast
import html
import json
import logging
import re
from typing import Any, Dict, List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import StreamingParseResult, _GetInfoFunc

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for mimo style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与mimo风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 30-37: Function `_get_param_type`
```python
def _get_param_type(func_name: str, param_name: str, tools: List[Tool]) -> str:
    """Get parameter type from tool schema."""
    for tool in tools:
        if tool.function.name == func_name:
            props = tool.function.parameters.get("properties", {})
            if param_name in props:
                return props[param_name].get("type", "string")
    return "string"
```
**EN:** This block defines the function `_get_param_type`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `_get_param_type`. Notable operations include `get`.
**CN:** 这一段定义了function `_get_param_type`，介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_param_type`。 值得注意的操作包括 `get`。

### Lines 40-46: Function `_convert_param_value` signature and setup
```python
def _convert_param_value(
    param_value: str, param_name: str, func_name: str, tools: List[Tool]
) -> Any:
    """
    Convert parameter value based on its type in the schema.
    Adapted from vllm-project/vllm (vllm/entrypoints/openai/tool_parsers/qwen3coder_tool_parser.py)
    """
```
**EN:** This block defines the function `_convert_param_value`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `_convert_param_value`. Notable operations include `vllm`.
**CN:** 这一段定义了function `_convert_param_value`，介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_param_value`。 值得注意的操作包括 `vllm`。

### Lines 47-78: Function `_convert_param_value` logic (part 1)
```python
    param_value = html.unescape(param_value)

    # Handle null value for any type
    if param_value.lower() == "null":
        return None

    param_type = _get_param_type(func_name, param_name, tools)

    if param_type in ["string", "str", "text", "varchar", "char", "enum"]:
        return param_value
    elif (
        param_type.startswith("int")
        or param_type.startswith("integer")
        or param_type.startswith("uint")
        or param_type.startswith("long")
        or param_type.startswith("short")
        or param_type.startswith("unsigned")
    ):
        try:
            return int(param_value)
        except (ValueError, TypeError):
            logger.warning(
                "Parsed value '%s' of parameter '%s' is not an "
                "integer in tool '%s', degenerating to string.",
                param_value,
                param_name,
                func_name,
            )
            return param_value
    elif param_type.startswith("num") or param_type.startswith("float"):
        try:
            float_param_value = float(param_value)
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `unescape`, `lower`, `_get_param_type`, `elif`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `unescape`、`lower`、`_get_param_type`、`elif`。

### Lines 79-110: Function `_convert_param_value` logic (part 2)
```python
            return (
                float_param_value
                if float_param_value - int(float_param_value) != 0
                else int(float_param_value)
            )
        except (ValueError, TypeError):
            logger.warning(
                "Parsed value '%s' of parameter '%s' is not a float "
                "in tool '%s', degenerating to string.",
                param_value,
                param_name,
                func_name,
            )
            return param_value
    elif param_type in ["boolean", "bool", "binary"]:
        param_value = param_value.lower()
        if param_value not in ["true", "false"]:
            logger.warning(
                "Parsed value '%s' of parameter '%s' is not a boolean "
                "(`true` or `false`) in tool '%s', degenerating to "
                "false.",
                param_value,
                param_name,
                func_name,
            )
        return param_value == "true"
    else:
        if (
            param_type in ["object", "array", "arr"]
            or param_type.startswith("dict")
            or param_type.startswith("list")
        ):
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `warning`, `lower`, `startswith`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `warning`、`lower`、`startswith`。

### Lines 111-134: Function `_convert_param_value` logic (part 3)
```python
            try:
                param_value = json.loads(param_value)
                return param_value
            except (json.JSONDecodeError, TypeError, ValueError):
                logger.warning(
                    "Parsed value '%s' of parameter '%s' cannot be "
                    "parsed with json.loads in tool '%s', will try "
                    "other methods to parse it.",
                    param_value,
                    param_name,
                    func_name,
                )
        try:
            param_value = ast.literal_eval(param_value)  # safer
        except (ValueError, SyntaxError, TypeError):
            logger.warning(
                "Parsed value '%s' of parameter '%s' cannot be "
                "converted via Python `ast.literal_eval()` in tool "
                "'%s', degenerating to string.",
                param_value,
                param_name,
                func_name,
            )
        return param_value
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `loads`, `warning`, `literal_eval`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`、`warning`、`literal_eval`。

### Lines 137-137: Class `MiMoDetector` declaration
```python
class MiMoDetector(BaseFormatDetector):
```
**EN:** This block declares the class `MiMoDetector` and establishes its responsibility inside mimo style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MiMoDetector`.
**CN:** 这一段声明了类 `MiMoDetector`，并说明它在mimo风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MiMoDetector`。

### Lines 138-148: Supporting state inside `MiMoDetector`
```python
    """
    Detector for MiMo function call format.

    Format:
        <tool_call>
        <function=execute_bash>
        <parameter=command>pwd && ls</parameter>
        </function>
        </tool_call>
    """

```
**EN:** This block adds supporting state or helper logic inside `MiMoDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MiMoDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 149-157: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<tool_call>"
        self.eot_token = "</tool_call>"
        self.tool_call_regex = re.compile(r"<tool_call>(.*?)</tool_call>", re.DOTALL)
        self.func_regex = re.compile(r"<function=([^>]+)>(.*?)</function>", re.DOTALL)
        self.param_regex = re.compile(
            r"<parameter=([^>]+)>(.*?)</parameter>", re.DOTALL
        )
```
**EN:** This block defines the method `__init__` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `compile`.
**CN:** 这一段定义了method `__init__`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`compile`。

### Lines 159-160: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 162-163: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """Parse complete text for tool calls."""
```
**EN:** This block defines the method `detect_and_parse` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 164-189: Method `detect_and_parse` logic (part 1)
```python
        idx = text.find(self.bot_token)
        if idx == -1:
            return StreamingParseResult(normal_text=text, calls=[])

        normal_text = text[:idx]
        tool_indices = self._get_tool_indices(tools)

        calls = []
        last_end = idx

        for match in self.tool_call_regex.finditer(text):
            tool_call_body = match.group(1)

            parsed = self._parse_tool_call(tool_call_body, tools)

            if parsed:
                func_name = parsed.get("name")
                if func_name not in tool_indices:
                    # Unknown function
                    logger.warning(f"Unknown function: {func_name}")
                    if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                        # Return tool call block as normal text
                        normal_text += text[last_end : match.end()]
                        last_end = match.end()
                        continue
                calls.extend(self.parse_base_json(parsed, tools))
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `_get_tool_indices`, `finditer`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`_get_tool_indices`、`finditer`。

### Lines 190-193: Method `detect_and_parse` logic (part 2)
```python

            last_end = match.end()

        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `end`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `end`、`StreamingParseResult`。

### Lines 195-200: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming parsing: buffer until complete tool call block.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 201-224: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        start = current_text.find(self.bot_token)
        if start == -1:
            if self.current_tool_id > 0:
                # Already processing tool calls, keep buffering
                # (more tool calls might come, don't discard text yet)
                return StreamingParseResult(normal_text="")
            else:
                # No tool calls seen yet, return as normal text
                self._buffer = ""
                return StreamingParseResult(normal_text=current_text)

        # Find end token AFTER the start token
        end = current_text.find(self.eot_token, start)
        if end == -1:
            # Incomplete tool call, return text before start and keep buffering
            normal_text = current_text[:start]
            self._buffer = current_text[start:]
            return StreamingParseResult(normal_text=normal_text)

        # Parse the complete tool call block
        result = self.detect_and_parse(current_text[: end + len(self.eot_token)], tools)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `detect_and_parse`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`detect_and_parse`。

### Lines 225-248: Method `parse_streaming_increment` logic (part 2)
```python

        if result.calls:
            # Valid tool call - initialize tracking if first one
            if self.current_tool_id == -1:
                self.current_tool_id = 0
                self.prev_tool_call_arr = []
                self.streamed_args_for_tool = [""]

            while len(self.prev_tool_call_arr) <= self.current_tool_id:
                self.prev_tool_call_arr.append({})
            while len(self.streamed_args_for_tool) <= self.current_tool_id:
                self.streamed_args_for_tool.append("")

            call = result.calls[0]
            self.prev_tool_call_arr[self.current_tool_id] = {
                "name": call.name,
                "arguments": json.loads(call.parameters) if call.parameters else {},
            }
            self.streamed_args_for_tool[self.current_tool_id] = call.parameters
            call.tool_index = self.current_tool_id
            self.current_tool_id += 1

        self._buffer = current_text[end + len(self.eot_token) :]
        return result
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `append`, `loads`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`loads`。

### Lines 250-258: Method `_parse_tool_call` signature and setup
```python
    def _parse_tool_call(
        self, tool_call_body: str, tools: List[Tool]
    ) -> Dict[str, Any]:
        """
        Parse content inside <tool_call>...</tool_call>.

        Structure:
            tool_call_body contains: <function=name>...params...</function>
        """
```
**EN:** This block defines the method `_parse_tool_call` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `_parse_tool_call`.
**CN:** 这一段定义了method `_parse_tool_call`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_tool_call`。

### Lines 259-275: Method `_parse_tool_call` logic (part 1)
```python
        # Match complete <function=name>body</function> block
        func_match = self.func_regex.search(tool_call_body)
        if not func_match:
            return None

        func_name = func_match.group(1).strip()
        func_body = func_match.group(2)

        params = {}
        for param_match in self.param_regex.finditer(func_body):
            param_name = param_match.group(1).strip()
            param_value = param_match.group(2)
            params[param_name] = _convert_param_value(
                param_value, param_name, func_name, tools
            )

        return {"name": func_name, "parameters": params}
```
**EN:** This block continues `_parse_tool_call` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding mimo style tool-call detection and normalization workflow. Notable operations include `search`, `group`, `strip`, `finditer`.
**CN:** 这一段延续了 `_parse_tool_call` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的mimo风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`group`、`strip`、`finditer`。

### Lines 277-278: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 280-281: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError
```
**EN:** This block defines the method `structure_info` on `MiMoDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of mimo style tool-call detection and normalization. Definitions introduced here include `structure_info`.
**CN:** 这一段定义了method `structure_info`（属于 `MiMoDetector`），介绍了参数、初始化步骤，以及这部分mimo风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。

## Key Concepts / 关键概念
- `_get_param_type`: Function that performs get param type for the surrounding workflow. / `_get_param_type`：在周边工作流中执行“getparam类型”相关任务的函数。
- `_convert_param_value`: Function that performs convert param value for the surrounding workflow. / `_convert_param_value`：在周边工作流中执行“convertparamvalue”相关任务的函数。
- `MiMoDetector`: Class that encapsulates mi mo detector behavior in this module. / `MiMoDetector`：封装与“mimo检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `html`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
