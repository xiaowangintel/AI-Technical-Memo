# qwen3_coder_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/qwen3_coder_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a qwen 3 coder detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Qwen3coder 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import ast
import json
import logging
import re
from typing import Any, List, Optional

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for qwen 3 coder style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Qwen3coder风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 18-18: Class `Qwen3CoderDetector` declaration
```python
class Qwen3CoderDetector(BaseFormatDetector):
```
**EN:** This block declares the class `Qwen3CoderDetector` and establishes its responsibility inside qwen 3 coder style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Qwen3CoderDetector`.
**CN:** 这一段声明了类 `Qwen3CoderDetector`，并说明它在Qwen3coder风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Qwen3CoderDetector`。

### Lines 19-21: Method `__init__` signature and setup
```python
    def __init__(self):
        super().__init__()

```
**EN:** This block defines the method `__init__` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 22-42: Method `__init__` logic (part 1)
```python
        # Sentinel tokens
        self.tool_call_start_token: str = "<tool_call>"
        self.tool_call_end_token: str = "</tool_call>"
        self.tool_call_prefix: str = "<function="
        self.function_end_token: str = "</function>"
        self.parameter_prefix: str = "<parameter="
        self.parameter_end_token: str = "</parameter>"

        # Regex for non-streaming fallback
        self.tool_call_regex = re.compile(r"<tool_call>(.*?)</tool_call>", re.DOTALL)
        self.tool_call_function_regex = re.compile(
            r"<function=(.*?)</function>|<function=(.*)$", re.DOTALL
        )
        self.tool_call_parameter_regex = re.compile(
            r"<parameter=(.*?)(?:</parameter>|(?=<parameter=)|(?=</function>)|$)",
            re.DOTALL,
        )

        # Streaming State
        # Base class already initializes _buffer, we just use it directly
        # No need to check with hasattr - we control the lifecycle through inheritance
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `compile`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `compile`。

### Lines 43-55: Method `__init__` logic (part 2)
```python

        # Index pointing to the next character to be processed in buffer
        self.parsed_pos: int = 0
        # Parameter count inside the current tool being processed, used to determine whether to add comma
        self.current_tool_param_count: int = 0
        # Flag indicating whether current tool has already sent '{'
        self.json_started: bool = False

        # [FIX] New state flag: mark whether inside tool_call structure block
        self.is_inside_tool_call: bool = False

        # Initialize attributes that were missing in the original PR
        self.current_func_name: Optional[str] = None
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。

### Lines 57-58: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.tool_call_start_token in text
```
**EN:** This block defines the method `has_tool_call` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 60-63: Method `_get_arguments_config` signature and setup
```python
    def _get_arguments_config(
        self, func_name: str, tools: Optional[list[Tool]]
    ) -> dict:
        """Extract argument configuration for a function."""
```
**EN:** This block defines the method `_get_arguments_config` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `_get_arguments_config`.
**CN:** 这一段定义了method `_get_arguments_config`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_get_arguments_config`。

### Lines 64-87: Method `_get_arguments_config` logic (part 1)
```python
        if tools is None:
            return {}
        for config in tools:
            try:
                config_type = config.type
                config_function = config.function
                config_function_name = config_function.name
            except AttributeError:
                continue

            if config_type == "function" and config_function_name == func_name:
                try:
                    params = config_function.parameters
                except AttributeError:
                    return {}

                if isinstance(params, dict) and "properties" in params:
                    return params["properties"]
                elif isinstance(params, dict):
                    return params
                else:
                    return {}
        logger.warning(f"Tool '{func_name}' is not defined in the tools list.")
        return {}
```
**EN:** This block continues `_get_arguments_config` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `warning`.
**CN:** 这一段延续了 `_get_arguments_config` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `warning`。

### Lines 89-92: Method `_convert_param_value` signature and setup
```python
    def _convert_param_value(
        self, param_value: str, param_name: str, param_config: dict, func_name: str
    ) -> Any:
        """Convert parameter value based on its type in the schema."""
```
**EN:** This block defines the method `_convert_param_value` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `_convert_param_value`.
**CN:** 这一段定义了method `_convert_param_value`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_param_value`。

### Lines 93-124: Method `_convert_param_value` logic (part 1)
```python
        # Handle null value for any type
        if param_value.lower() == "null":
            return None

        if param_name not in param_config:
            if param_config != {}:
                logger.warning(
                    f"Parsed parameter '{param_name}' is not defined in the tool "
                    f"parameters for tool '{func_name}', directly returning the string value."
                )
            return param_value

        if (
            isinstance(param_config[param_name], dict)
            and "type" in param_config[param_name]
        ):
            param_type = str(param_config[param_name]["type"]).strip().lower()
        else:
            param_type = "string"
        if param_type in ["string", "str", "text", "varchar", "char", "enum"]:
            return param_value
        elif (
            param_type.startswith("int")
            or param_type.startswith("uint")
            or param_type.startswith("long")
            or param_type.startswith("short")
            or param_type.startswith("unsigned")
        ):
            try:
                param_value = int(param_value)
            except Exception:
                logger.warning(
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `lower`, `warning`, `strip`, `elif`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `lower`、`warning`、`strip`、`elif`。

### Lines 125-156: Method `_convert_param_value` logic (part 2)
```python
                    f"Parsed value '{param_value}' of parameter '{param_name}' is not an integer in tool "
                    f"'{func_name}', degenerating to string."
                )
            return param_value
        elif param_type.startswith("num") or param_type.startswith("float"):
            try:
                maybe_convert = (
                    False if "." in param_value or "e" in param_value.lower() else True
                )
                param_value: float = float(param_value)
                if maybe_convert and param_value.is_integer():
                    param_value = int(param_value)
            except Exception:
                logger.warning(
                    f"Parsed value '{param_value}' of parameter '{param_name}' is not a float in tool "
                    f"'{func_name}', degenerating to string."
                )
            return param_value
        elif param_type in ["boolean", "bool", "binary"]:
            param_value = param_value.lower()
            if param_value not in ["true", "false"]:
                logger.warning(
                    f"Parsed value '{param_value}' of parameter '{param_name}' is not a boolean (`true` of `false`) in tool '{func_name}', degenerating to false."
                )
            return param_value == "true"
        else:
            if (
                param_type in ["object", "array", "arr"]
                or param_type.startswith("dict")
                or param_type.startswith("list")
            ):
                try:
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `startswith`, `lower`, `is_integer`, `warning`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`lower`、`is_integer`、`warning`。

### Lines 157-170: Method `_convert_param_value` logic (part 3)
```python
                    param_value = json.loads(param_value)
                    return param_value
                except Exception:
                    logger.warning(
                        f"Parsed value '{param_value}' of parameter '{param_name}' cannot be parsed with json.loads in tool "
                        f"'{func_name}', will try other methods to parse it."
                    )
            try:
                param_value = ast.literal_eval(param_value)  # safer
            except Exception:
                logger.warning(
                    f"Parsed value '{param_value}' of parameter '{param_name}' cannot be converted via Python `ast.literal_eval()` in tool '{func_name}', degenerating to string."
                )
            return param_value
```
**EN:** This block continues `_convert_param_value` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `loads`, `warning`, `literal_eval`.
**CN:** 这一段延续了 `_convert_param_value` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`、`warning`、`literal_eval`。

### Lines 172-173: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """One-shot parsing for non-streaming scenarios."""
```
**EN:** This block defines the method `detect_and_parse` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 174-194: Method `detect_and_parse` logic (part 1)
```python
        if self.tool_call_start_token not in text:
            return StreamingParseResult(normal_text=text)

        calls = []
        try:
            # Simple cleanup of the text to find tool calls
            # Note: This is a simplified regex approach consistent with vLLM
            raw_tool_calls = self.tool_call_regex.findall(text)
            if not raw_tool_calls:
                # Fallback: maybe the whole text is inside the tag or tags are stripped
                if self.tool_call_prefix in text:
                    raw_tool_calls = [text]

            tool_idx = 0
            for tool_content in raw_tool_calls:
                # Find function calls
                funcs = self.tool_call_function_regex.findall(tool_content)
                for func_match in funcs:
                    func_body = func_match[0] or func_match[1]
                    if ">" not in func_body:
                        continue
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `findall`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`findall`。

### Lines 195-217: Method `detect_and_parse` logic (part 2)
```python

                    name_end = func_body.index(">")
                    func_name = func_body[:name_end]
                    params_str = func_body[name_end + 1 :]

                    param_config = self._get_arguments_config(func_name, tools)
                    parsed_params = {}

                    for p_match in self.tool_call_parameter_regex.findall(params_str):
                        if ">" not in p_match:
                            continue
                        p_idx = p_match.index(">")
                        p_name = p_match[:p_idx]
                        p_val = p_match[p_idx + 1 :]
                        # Remove prefixing and trailing \n
                        if p_val.startswith("\n"):
                            p_val = p_val[1:]
                        if p_val.endswith("\n"):
                            p_val = p_val[:-1]

                        parsed_params[p_name] = self._convert_param_value(
                            p_val, p_name, param_config, func_name
                        )
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `index`, `_get_arguments_config`, `findall`, `startswith`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `index`、`_get_arguments_config`、`findall`、`startswith`。

### Lines 218-238: Method `detect_and_parse` logic (part 3)
```python

                    calls.append(
                        ToolCallItem(
                            tool_index=tool_idx,
                            name=func_name,
                            parameters=json.dumps(parsed_params, ensure_ascii=False),
                        )
                    )
                    tool_idx += 1

            # Determine normal text (text before the first tool call)
            start_idx = text.find(self.tool_call_start_token)
            if start_idx == -1:
                start_idx = text.find(self.tool_call_prefix)
            normal_text = text[:start_idx] if start_idx > 0 else ""

            return StreamingParseResult(normal_text=normal_text, calls=calls)

        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}")
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `dumps`, `text`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`dumps`、`text`。

### Lines 240-245: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Robust cursor-based streaming parser.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 246-283: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        # Guard against empty buffer
        if not self._buffer:
            return StreamingParseResult()

        calls = []
        normal_text_chunks = []

        while True:
            # Working text slice
            current_slice = self._buffer[self.parsed_pos :]

            # Optimization: If almost empty, wait for more
            if not current_slice:
                break

            # -------------------------------------------------------
            # 1. Priority detection: check if it's the start of Tool Call
            # -------------------------------------------------------
            if current_slice.startswith(self.tool_call_start_token):
                self.parsed_pos += len(self.tool_call_start_token)
                self.is_inside_tool_call = True
                continue

            # -------------------------------------------------------
            # 2. Function Name: <function=name>
            # -------------------------------------------------------
            if current_slice.startswith(self.tool_call_prefix):
                end_angle = current_slice.find(">")
                if end_angle != -1:
                    func_name = current_slice[len(self.tool_call_prefix) : end_angle]

                    self.current_tool_id += 1
                    self.current_tool_name_sent = True
                    self.current_tool_param_count = 0
                    self.json_started = False
                    self.current_func_name = func_name
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `startswith`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`startswith`、`find`。

### Lines 284-325: Method `parse_streaming_increment` logic (part 2)
```python

                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=func_name,
                            parameters="",
                        )
                    )

                    self.parsed_pos += end_angle + 1
                    continue
                else:
                    # Incomplete tag
                    break

            # -------------------------------------------------------
            # 3. Parameter: <parameter=name>value...
            # -------------------------------------------------------
            if current_slice.startswith(self.parameter_prefix):
                name_end = current_slice.find(">")
                if name_end != -1:
                    value_start_idx = name_end + 1
                    rest_of_slice = current_slice[value_start_idx:]

                    # A parameter can end in multiple ways:
                    # 1. [Normal] Encounter </parameter>
                    # 2. [Abnormal] Encounter next <parameter=
                    # 3. [Abnormal] Encounter </function>
                    # So we need to find the smallest one as the parameter end position.
                    cand_end_param = rest_of_slice.find(self.parameter_end_token)
                    cand_next_param = rest_of_slice.find(self.parameter_prefix)
                    cand_end_func = rest_of_slice.find(self.function_end_token)

                    candidates = []
                    if cand_end_param != -1:
                        candidates.append(
                            (cand_end_param, len(self.parameter_end_token))
                        )
                    if cand_next_param != -1:
                        candidates.append((cand_next_param, 0))
                    if cand_end_func != -1:
                        candidates.append((cand_end_func, 0))
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `startswith`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`startswith`、`find`。

### Lines 326-366: Method `parse_streaming_increment` logic (part 3)
```python

                    if candidates:
                        best_cand = min(candidates, key=lambda x: x[0])
                        end_pos = best_cand[0]
                        end_token_len = best_cand[1]

                        param_name = current_slice[
                            len(self.parameter_prefix) : name_end
                        ]
                        raw_value = rest_of_slice[:end_pos]

                        # Cleanup value
                        if raw_value.startswith("\n"):
                            raw_value = raw_value[1:]
                        if raw_value.endswith("\n"):
                            raw_value = raw_value[:-1]

                        # JSON Construction
                        if not self.json_started:
                            calls.append(
                                ToolCallItem(
                                    tool_index=self.current_tool_id, parameters="{"
                                )
                            )
                            self.json_started = True

                        param_config = self._get_arguments_config(
                            self.current_func_name, tools
                        )
                        converted_val = self._convert_param_value(
                            raw_value, param_name, param_config, self.current_func_name
                        )

                        # Construct JSON fragment: "key": value
                        # Note: We must be careful with json.dumps to ensure valid JSON streaming
                        json_key_val = f"{json.dumps(param_name)}: {json.dumps(converted_val, ensure_ascii=False)}"

                        if self.current_tool_param_count > 0:
                            fragment = f", {json_key_val}"
                        else:
                            fragment = json_key_val
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `startswith`, `endswith`, `append`, `ToolCallItem`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`endswith`、`append`、`ToolCallItem`。

### Lines 367-406: Method `parse_streaming_increment` logic (part 4)
```python

                        calls.append(
                            ToolCallItem(
                                tool_index=self.current_tool_id, parameters=fragment
                            )
                        )
                        self.current_tool_param_count += 1

                        # Advance cursor
                        total_len = (name_end + 1) + end_pos + end_token_len
                        self.parsed_pos += total_len
                        continue

                # Incomplete parameter tag or value
                break

            # -------------------------------------------------------
            # 4. Function End: </function>
            # -------------------------------------------------------
            if current_slice.startswith(self.function_end_token):
                if not self.json_started:
                    calls.append(
                        ToolCallItem(tool_index=self.current_tool_id, parameters="{")
                    )
                    self.json_started = True

                calls.append(
                    ToolCallItem(tool_index=self.current_tool_id, parameters="}")
                )
                self.parsed_pos += len(self.function_end_token)
                self.current_func_name = None
                continue

            # -------------------------------------------------------
            # 5. Tool Call End: </tool_call>
            # -------------------------------------------------------
            if current_slice.startswith(self.tool_call_end_token):
                self.parsed_pos += len(self.tool_call_end_token)
                self.is_inside_tool_call = False  # [FIX] Exit tool call region
                continue
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`startswith`。

### Lines 407-441: Method `parse_streaming_increment` logic (part 5)
```python

            # -------------------------------------------------------
            # 6. Handling content / whitespace / normal text
            # -------------------------------------------------------
            # If current position is not the start of a tag (i.e., doesn't start with <), it might be plain text,
            # or a newline between two tags.
            # But we need to be careful not to output truncated tags like "<fun" as text.

            next_open_angle = current_slice.find("<")

            if next_open_angle == -1:
                # This entire segment is plain text
                if not self.is_inside_tool_call:
                    normal_text_chunks.append(current_slice)
                # [FIX] If inside tool call, discard this text (usually \n), don't append
                self.parsed_pos += len(current_slice)
                continue

            elif next_open_angle == 0:
                # Looks like a Tag, but doesn't match any known Tag above

                possible_tags = [
                    self.tool_call_start_token,
                    self.tool_call_end_token,
                    self.tool_call_prefix,
                    self.function_end_token,
                    self.parameter_prefix,
                    self.parameter_end_token,
                ]

                is_potential_tag = False
                for tag in possible_tags:
                    if tag.startswith(current_slice):
                        is_potential_tag = True
                        break
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `tag`, `find`, `append`, `text`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `tag`、`find`、`append`、`text`。

### Lines 442-468: Method `parse_streaming_increment` logic (part 6)
```python

                if is_potential_tag:
                    break  # Wait for more
                else:
                    # Just a plain '<' symbol
                    if not self.is_inside_tool_call:
                        normal_text_chunks.append("<")
                    self.parsed_pos += 1
                    continue

            else:
                # '<' is in the middle
                text_segment = current_slice[:next_open_angle]
                if not self.is_inside_tool_call:
                    normal_text_chunks.append(text_segment)
                # [FIX] If inside tool call, discard whitespace/text before Tag
                self.parsed_pos += next_open_angle
                continue

        # Memory Cleanup: Slice the buffer
        # Keep unparsed part, discard parsed part
        if self.parsed_pos > 0:
            self._buffer = self._buffer[self.parsed_pos :]
            self.parsed_pos = 0

        normal_text = "".join(normal_text_chunks) if normal_text_chunks else ""
        return StreamingParseResult(calls=calls, normal_text=normal_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 3 coder style tool-call detection and normalization workflow. Notable operations include `append`, `join`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen3coder风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`join`、`StreamingParseResult`。

### Lines 470-471: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        return True
```
**EN:** This block defines the method `supports_structural_tag` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 473-474: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError
```
**EN:** This block defines the method `structure_info` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `structure_info`.
**CN:** 这一段定义了method `structure_info`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。

### Lines 476-477: Method `get_structural_tag_name`
```python
    def get_structural_tag_name(self) -> str:
        return "qwen_3_coder"
```
**EN:** This block defines the method `get_structural_tag_name` on `Qwen3CoderDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 3 coder style tool-call detection and normalization. Definitions introduced here include `get_structural_tag_name`.
**CN:** 这一段定义了method `get_structural_tag_name`（属于 `Qwen3CoderDetector`），介绍了参数、初始化步骤，以及这部分Qwen3coder风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag_name`。

## Key Concepts / 关键概念
- `Qwen3CoderDetector`: Class that encapsulates qwen 3 coder detector behavior in this module. / `Qwen3CoderDetector`：封装与“Qwen3coder检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
