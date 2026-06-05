# deepseekv32_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/deepseekv32_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a deepseekv 32 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 deepseekv32 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Imports and module setup
```python
import json
import logging
import re
from typing import List, Literal, Optional, Union

from partial_json_parser.core.options import Allow

from sglang.srt.entrypoints.openai.protocol import Tool, ToolChoice
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)
from sglang.srt.function_call.utils import _find_common_prefix, _partial_json_loads

try:
    from xgrammar import StructuralTag
    from xgrammar.structural_tag import (
        AnyTextFormat,
        ConstStringFormat,
        JSONSchemaFormat,
        SequenceFormat,
        TagFormat,
        TagsWithSeparatorFormat,
        TriggeredTagsFormat,
    )
except ImportError:
    StructuralTag = None  # type: ignore

logger = logging.getLogger(__name__)

# Names mirror the DeepSeek-V3.2 official chat template tokens
# (see encoding_dsv32.TOOLS_SYSTEM_TEMPLATE).
_INVOKE_BEGIN_PREFIX = '<｜DSML｜invoke name="'
_INVOKE_BEGIN_SUFFIX = '">\n'
_THINK_TAG_END = "</think>"
_THINK_EXCLUDE_TOKENS = ["<think>", "</think>"]
_XML_STYLE = "deepseek_xml"
```
**EN:** This block gathers the imports and module-level setup for deepseekv 32 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与deepseekv32风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 43-43: Class `DeepSeekV32Detector` declaration
```python
class DeepSeekV32Detector(BaseFormatDetector):
```
**EN:** This block declares the class `DeepSeekV32Detector` and establishes its responsibility inside deepseekv 32 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DeepSeekV32Detector`.
**CN:** 这一段声明了类 `DeepSeekV32Detector`，并说明它在deepseekv32风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DeepSeekV32Detector`。

### Lines 44-84: Supporting state inside `DeepSeekV32Detector`
```python
    """
    Detector for DeepSeek V3.2 model function call format.

    The DeepSeek V3.2 format uses XML-like DSML tags to delimit function calls.
    Supports two parameter formats:

    Format 1 - XML Parameter Tags:
    ```
    <｜DSML｜function_calls>
        <｜DSML｜invoke name="function_name">
        <｜DSML｜parameter name="param_name" string="true">value</｜DSML｜parameter>
        ...
    </｜DSML｜invoke>
    </｜DSML｜function_calls>
    ```

    Format 2 - Direct JSON:
    ```
    <｜DSML｜function_calls>
        <｜DSML｜invoke name="function_name">
        {
            "param_name": "value"
        }
    </｜DSML｜invoke>
    </｜DSML｜function_calls>
    ```

    Examples:
    ```
    <｜DSML｜function_calls>
        <｜DSML｜invoke name="get_favorite_tourist_spot">
        <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>
    </｜DSML｜invoke>
    </｜DSML｜function_calls>

    <｜DSML｜function_calls>
        <｜DSML｜invoke name="get_favorite_tourist_spot">
        { "city": "San Francisco" }
    </｜DSML｜invoke>
    </｜DSML｜function_calls>
    ```
```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV32Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV32Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 85-94: Supporting state inside `DeepSeekV32Detector`
```python

    Key Components:
    - Tool Calls Section: Wrapped between `<｜DSML｜function_calls>` and `</｜DSML｜function_calls>`
    - Individual Tool Call: Wrapped between `<｜DSML｜invoke name="...">` and `</｜DSML｜invoke>`
    - Parameters: Either XML tags or direct JSON format
    - Supports multiple tool calls

    Reference: DeepSeek V3.2 format specification
    """

```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV32Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV32Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 95-117: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<｜DSML｜function_calls>"
        self.eot_token = "</｜DSML｜function_calls>"
        self.invoke_end_token = "</｜DSML｜invoke>"
        self.parameter_regex = r'<｜DSML｜parameter\s+name="([^"]+)"\s+string="([^"]+)"\s*>(.*?)</｜DSML｜parameter>'
        self.partial_parameter_regex = (
            r'<｜DSML｜parameter\s+name="([^"]+)"\s+string="([^"]+)"\s*>(.*)$'
        )
        self.function_calls_regex = (
            r"<｜DSML｜function_calls>(.*?)</｜DSML｜function_calls>"
        )
        # Long-form `<｜DSML｜invoke name="x">...</｜DSML｜invoke>` and the
        # self-closing `<｜DSML｜invoke name="x"/>` shape V4 emits for zero-arg
        # tools. The `end` group is empty when the closer hasn't streamed in.
        self.invoke_regex = (
            r'<｜DSML｜invoke\s+name="(?P<name>[^"]+)"\s*'
            r"(?:(?P<self_close>/>)"
            r"|>(?P<body>.*?)(?P<end>(?:</｜DSML｜invoke>|$)))"
        )
        self.prefix_parameter_end_call = ["</", "｜DSML｜", "parameter"]
        self.prefix_invoke_end_call = ["</", "｜DSML｜", "inv", "oke"]
        self.current_tool_id = -1
```
**EN:** This block defines the method `__init__` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 119-121: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a deepseek v32 format tool call."""
        return self.bot_token in text or "<｜DSML｜invoke" in text
```
**EN:** This block defines the method `has_tool_call` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 123-135: Method `_unpack_invoke_match`
```python
    @staticmethod
    def _unpack_invoke_match(m: "re.Match[str]") -> tuple[str, str, bool]:
        """Returns (name, body, is_complete) for an invoke_regex match.

        Self-closing invokes have empty body and are always complete.
        Long-form bodies are always strings (possibly empty); they're
        incomplete when matched against `$` because the closing tag
        hasn't streamed in yet.
        """
        name = m.group("name").strip()
        if m.group("self_close"):
            return name, "", True
        return name, m.group("body"), bool(m.group("end"))
```
**EN:** This block defines the method `_unpack_invoke_match` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `_unpack_invoke_match`. Notable operations include `Returns`, `strings`, `group`, `strip`.
**CN:** 这一段定义了method `_unpack_invoke_match`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_unpack_invoke_match`。 值得注意的操作包括 `Returns`、`strings`、`group`、`strip`。

### Lines 137-146: Method `_parse_parameters_from_xml` signature and setup
```python
    def _parse_parameters_from_xml(
        self, invoke_content: str, allow_partial: bool = False
    ) -> str:
        """
        Parse parameters from either XML-like format or JSON format to str.

        Supports two formats:
        1. XML parameter tags: <｜DSML｜parameter name="..." string="...">value</｜DSML｜parameter>
        2. Direct JSON: { "key": "value" }
        """
```
**EN:** This block defines the method `_parse_parameters_from_xml` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `_parse_parameters_from_xml`.
**CN:** 这一段定义了method `_parse_parameters_from_xml`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_parameters_from_xml`。

### Lines 147-170: Method `_parse_parameters_from_xml` logic (part 1)
```python
        # First, try to parse as direct JSON (new format)
        invoke_content_stripped = invoke_content.strip()
        if invoke_content_stripped.startswith("{"):
            if allow_partial:
                # Remove incomplete invoke end call prefix in case they are captured by param
                for token in reversed(self.prefix_invoke_end_call):
                    invoke_content_stripped = invoke_content_stripped.rstrip(token)
                return invoke_content_stripped
            elif invoke_content_stripped.endswith("}"):
                return invoke_content_stripped

        # Fall back to XML parameter tag parsing (original format)
        parameters = {}
        # Find all complete parameter matches
        param_matches = list(
            re.finditer(self.parameter_regex, invoke_content, re.DOTALL)
        )

        last_match_end = 0
        for match in param_matches:
            param_name = match.group(1)
            param_type = match.group(2)
            param_value = match.group(3)
            last_match_end = match.end()
```
**EN:** This block continues `_parse_parameters_from_xml` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `JSON`, `strip`, `startswith`, `reversed`.
**CN:** 这一段延续了 `_parse_parameters_from_xml` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `JSON`、`strip`、`startswith`、`reversed`。

### Lines 171-194: Method `_parse_parameters_from_xml` logic (part 2)
```python

            # Convert value based on type
            if param_type == "true":  # string type
                parameters[param_name] = param_value.strip()
            else:
                # Try to parse as JSON for other types
                try:
                    parameters[param_name] = json.loads(param_value.strip())
                except (json.JSONDecodeError, ValueError):
                    parameters[param_name] = param_value.strip()

        # If allowed, try to parse a partial parameter at the end
        if allow_partial:
            remaining_content = invoke_content[last_match_end:]

            # Remove incomplete parameter_end_call prefix in case they are captured by param
            for token in reversed(self.prefix_parameter_end_call):
                remaining_content = remaining_content.rstrip(token)

            # Match start of a parameter tag + value (potentially incomplete)
            # Regex: <tag name="..." string="...">VALUE... (no end tag)
            partial_match = re.search(
                self.partial_parameter_regex, remaining_content, re.DOTALL
            )
```
**EN:** This block continues `_parse_parameters_from_xml` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `strip`, `loads`, `reversed`, `rstrip`.
**CN:** 这一段延续了 `_parse_parameters_from_xml` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `strip`、`loads`、`reversed`、`rstrip`。

### Lines 195-208: Method `_parse_parameters_from_xml` logic (part 3)
```python

            if partial_match and (param_value := partial_match.group(3)):
                param_name = partial_match.group(1)
                if partial_match.group(2) == "true":
                    parameters[param_name] = param_value.strip()
                else:
                    try:
                        parameters[param_name] = _partial_json_loads(
                            param_value, Allow.ALL
                        )[0]
                    except json.JSONDecodeError:
                        parameters[param_name] = param_value.strip()

        return json.dumps(parameters, ensure_ascii=False)
```
**EN:** This block continues `_parse_parameters_from_xml` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `and`, `group`, `strip`, `_partial_json_loads`.
**CN:** 这一段延续了 `_parse_parameters_from_xml` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `and`、`group`、`strip`、`_partial_json_loads`。

### Lines 210-217: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: list[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: ParseResult indicating success or failure, consumed text, leftover text, and parsed calls.
        """
```
**EN:** This block defines the method `detect_and_parse` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 218-244: Method `detect_and_parse` logic (part 1)
```python
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=normal_text, calls=[])

        calls = []
        try:
            # Extract content between function_calls tags
            function_calls_match = re.search(
                self.function_calls_regex,
                text,
                re.DOTALL,
            )
            if not function_calls_match:
                return StreamingParseResult(normal_text=normal_text, calls=[])

            function_calls_content = function_calls_match.group(1)

            # Find all invoke blocks
            for invoke_match in re.finditer(
                self.invoke_regex, function_calls_content, re.DOTALL
            ):
                func_name, invoke_content, _ = self._unpack_invoke_match(invoke_match)
                func_args = self._parse_parameters_from_xml(invoke_content)
                # construct match_result for parse_base_json
                match_result = {"name": func_name, "parameters": json.loads(func_args)}
                calls.extend(self.parse_base_json(match_result, tools))
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `find`, `strip`, `StreamingParseResult`, `search`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`、`StreamingParseResult`、`search`。

### Lines 245-250: Method `detect_and_parse` logic (part 2)
```python

            return StreamingParseResult(normal_text=normal_text, calls=calls)
        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}")
            # return the normal text if parsing fails
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `error`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`error`。

### Lines 252-258: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: list[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing tool calls for DeepSeekV32 format.
        Supports multiple consecutive invoke blocks and argument streaming.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 259-295: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        # Check if buffer contains any DSML markers or ends with potential tag prefix
        # This handles partial/streaming DSML content
        dsml_markers = ["｜DSML｜", "<｜", "</｜"]
        potentially_dsml = any(marker in current_text for marker in dsml_markers)

        # Also check if text ends with start of a tag (to handle "<" arriving separately)
        dsml_prefixes = ["<", "<｜", "</", "</｜"]
        ends_with_prefix = any(
            current_text.rstrip().endswith(prefix) for prefix in dsml_prefixes
        )

        if (
            not self.has_tool_call(current_text)
            and not potentially_dsml
            and not ends_with_prefix
        ):
            self._buffer = ""
            for e_token in [self.eot_token, self.invoke_end_token]:
                if e_token in current_text:
                    current_text = current_text.replace(e_token, "")
            return StreamingParseResult(normal_text=current_text)

        all_calls: list[ToolCallItem] = []
        try:
            # Loop to handle multiple consecutive invoke blocks
            while True:
                # Try to match an invoke block (may be partial)
                invoke_match = re.search(
                    pattern=self.invoke_regex,
                    string=current_text,
                    flags=re.DOTALL,
                )
                if not invoke_match:
                    break
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `tag`, `rstrip`, `endswith`, `has_tool_call`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `tag`、`rstrip`、`endswith`、`has_tool_call`。

### Lines 296-327: Method `parse_streaming_increment` logic (part 2)
```python

                func_name, invoke_content, is_tool_end = self._unpack_invoke_match(
                    invoke_match
                )

                # Initialize state if this is the first tool call
                if self.current_tool_id == -1:
                    self.current_tool_id = 0
                    self.prev_tool_call_arr = []
                    self.streamed_args_for_tool = [""]

                # Ensure arrays are large enough for current tool
                while len(self.prev_tool_call_arr) <= self.current_tool_id:
                    self.prev_tool_call_arr.append({})
                while len(self.streamed_args_for_tool) <= self.current_tool_id:
                    self.streamed_args_for_tool.append("")

                # 1. Send tool name if not sent yet
                if not self.current_tool_name_sent:
                    all_calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=func_name,
                            parameters="",
                        )
                    )
                    self.current_tool_name_sent = True

                # 2. Parse current parameters (partial or complete)
                current_params = self._parse_parameters_from_xml(
                    invoke_content, allow_partial=not is_tool_end
                )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `_unpack_invoke_match`, `append`, `ToolCallItem`, `parameters`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_unpack_invoke_match`、`append`、`ToolCallItem`、`parameters`。

### Lines 328-355: Method `parse_streaming_increment` logic (part 3)
```python

                # 3. Calculate and send incremental arguments
                sent_len = len(self.streamed_args_for_tool[self.current_tool_id])
                prev_params = self.prev_tool_call_arr[self.current_tool_id].get(
                    "arguments"
                )

                argument_diff = None

                if is_tool_end:
                    # If complete, send everything remaining
                    argument_diff = current_params[sent_len:]
                elif prev_params is not None:
                    # If partial, send stable prefix diff
                    if current_params != prev_params:
                        prefix = _find_common_prefix(current_params, prev_params)
                        if len(prefix) > sent_len:
                            argument_diff = prefix[sent_len:]

                if argument_diff:
                    all_calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=None,
                            parameters=argument_diff,
                        )
                    )
                    self.streamed_args_for_tool[self.current_tool_id] += argument_diff
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `get`, `_find_common_prefix`, `append`, `ToolCallItem`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get`、`_find_common_prefix`、`append`、`ToolCallItem`。

### Lines 356-385: Method `parse_streaming_increment` logic (part 4)
```python

                # Update the stored arguments
                self.prev_tool_call_arr[self.current_tool_id] = {
                    "name": func_name,
                    "arguments": current_params,
                }

                # Check if tool call is complete (has closing tag)
                if is_tool_end:
                    # Remove the completed tool call from buffer
                    self._buffer = current_text[invoke_match.end() :]
                    current_text = self._buffer  # Update for next iteration

                    # Move to next tool call
                    self.current_tool_id += 1
                    self.current_tool_name_sent = False

                    # Continue loop to check for more invoke blocks
                    continue
                else:
                    # Tool call not complete yet, don't return anything
                    # Wait for more chunks until we see </｜DSML｜invoke>
                    break

            # No more invoke blocks found
            return StreamingParseResult(normal_text="", calls=all_calls)

        except Exception as e:
            logger.error(f"Error in parse_streaming_increment: {e}")
            return StreamingParseResult(normal_text=current_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `complete`, `end`, `StreamingParseResult`, `error`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `complete`、`end`、`StreamingParseResult`、`error`。

### Lines 387-392: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin=f'<｜DSML｜invoke name="{name}">',
            end="</｜DSML｜invoke>",
            trigger="<｜DSML｜invoke",
        )
```
**EN:** This block defines the method `structure_info` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

### Lines 394-422: Method `get_structural_tag` signature and setup
```python
    def get_structural_tag(
        self,
        tools: Union[List[Tool], None] = None,
        tool_choice: Union[ToolChoice, Literal["auto", "required"]] = "auto",
        thinking_mode: bool = False,
    ) -> Optional["StructuralTag"]:
        """
        Build an xgrammar StructuralTag locally for DeepSeek-V3.2.

        Both layers — the outer `<｜DSML｜function_calls｜>...` wrapper and
        the inner `<｜DSML｜invoke>...</｜DSML｜invoke>` blocks — are encoded
        directly in the grammar with a single-newline join between
        consecutive invokes, matching DeepSeek-V3.2's official chat
        template. This avoids two layered defects that surfaced with the
        prior `xgrammar.get_model_structural_tag("deepseek_v3_2")` path:

        - the xgrammar builtin template (pre mlc-ai/xgrammar#638) forced
          a double-newline join, which deterministically collapsed
          parallel tool calls to one at greedy decoding.
        - falling back to the legacy structural tag (built from
          `structure_info()`) only constrains the inner invoke block;
          the outer wrapper is off-grammar and the model can skip it
          under `at_least_one=True`, leaving `detect_and_parse` with no
          `<｜DSML｜function_calls>` marker to anchor on.

        Returning a fully-formed StructuralTag from the detector keeps
        both fixes local to sglang and decoupled from the xgrammar
        release cadence.
        """
```
**EN:** This block defines the method `get_structural_tag` on `DeepSeekV32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 32 style tool-call detection and normalization. Definitions introduced here include `get_structural_tag`. Notable operations include `get_model_structural_tag`, `template`, `tag`, `structure_info`.
**CN:** 这一段定义了method `get_structural_tag`（属于 `DeepSeekV32Detector`），介绍了参数、初始化步骤，以及这部分deepseekv32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag`。 值得注意的操作包括 `get_model_structural_tag`、`template`、`tag`、`structure_info`。

### Lines 423-453: Method `get_structural_tag` logic (part 1)
```python
        if not tools or StructuralTag is None:
            return None

        # `INVOKE_END` and the empty separator together yield a single `\n`
        # between consecutive invokes — matching DeepSeek-V3.2's chat template
        # `"\n".join(invoke_blocks)`.
        function_calls_begin = self.bot_token + "\n"
        invoke_end = self.invoke_end_token + "\n"

        def _invoke_tag(tool: Tool) -> TagFormat:
            return TagFormat(
                begin=_INVOKE_BEGIN_PREFIX + tool.function.name + _INVOKE_BEGIN_SUFFIX,
                content=JSONSchemaFormat(
                    json_schema=tool.function.parameters or {},
                    style=_XML_STYLE,
                ),
                end=invoke_end,
            )

        if isinstance(tool_choice, ToolChoice):
            target = next(
                (t for t in tools if t.function.name == tool_choice.function.name),
                None,
            )
            if target is None:
                return None
            invoke_tags = [_invoke_tag(target)]
            is_required = True
        else:
            invoke_tags = [_invoke_tag(t) for t in tools]
            is_required = tool_choice == "required"
```
**EN:** This block continues `get_structural_tag` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Definitions introduced here include `_invoke_tag`. Notable operations include `join`, `TagFormat`, `JSONSchemaFormat`, `next`.
**CN:** 这一段延续了 `get_structural_tag` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 此处引入的定义包括 `_invoke_tag`。 值得注意的操作包括 `join`、`TagFormat`、`JSONSchemaFormat`、`next`。

### Lines 454-484: Method `get_structural_tag` logic (part 2)
```python

        inner_tool_calls = TagsWithSeparatorFormat(
            tags=invoke_tags, separator="", at_least_one=True
        )

        if is_required:
            suffix_tag = SequenceFormat(
                elements=[
                    ConstStringFormat(value=function_calls_begin),
                    inner_tool_calls,
                    ConstStringFormat(value=self.eot_token),
                ]
            )
        else:
            suffix_tag = TriggeredTagsFormat(
                triggers=[self.bot_token],
                tags=[
                    TagFormat(
                        begin=function_calls_begin,
                        content=inner_tool_calls,
                        end=self.eot_token,
                    )
                ],
                excludes=_THINK_EXCLUDE_TOKENS,
            )

        if not thinking_mode:
            return StructuralTag(format=suffix_tag)

        prefix_tag = TagFormat(begin="", content=AnyTextFormat(), end=_THINK_TAG_END)
        return StructuralTag(format=SequenceFormat(elements=[prefix_tag, suffix_tag]))
```
**EN:** This block continues `get_structural_tag` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 32 style tool-call detection and normalization workflow. Notable operations include `TagsWithSeparatorFormat`, `SequenceFormat`, `ConstStringFormat`, `TriggeredTagsFormat`.
**CN:** 这一段延续了 `get_structural_tag` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `TagsWithSeparatorFormat`、`SequenceFormat`、`ConstStringFormat`、`TriggeredTagsFormat`。

## Key Concepts / 关键概念
- `DeepSeekV32Detector`: Class that encapsulates deep seek v 32 detector behavior in this module. / `DeepSeekV32Detector`：封装与“deepseekv32检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **External packages / 外部依赖**: `partial_json_parser`, `xgrammar`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.utils`
