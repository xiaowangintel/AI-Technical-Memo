# deepseekv3_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/deepseekv3_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a deepseekv 3 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 deepseekv3 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import json
import logging
import re
from typing import List

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
**EN:** This block gathers the imports and module-level setup for deepseekv 3 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与deepseekv3风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 19-19: Class `DeepSeekV3Detector` declaration
```python
class DeepSeekV3Detector(BaseFormatDetector):
```
**EN:** This block declares the class `DeepSeekV3Detector` and establishes its responsibility inside deepseekv 3 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DeepSeekV3Detector`.
**CN:** 这一段声明了类 `DeepSeekV3Detector`，并说明它在deepseekv3风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DeepSeekV3Detector`。

### Lines 20-44: Supporting state inside `DeepSeekV3Detector`
```python
    """
    Detector for DeepSeek V3 model function call format.

    The DeepSeek V3 format uses special Unicode tokens to delimit function calls
    with JSON code blocks for arguments.

    Format Structure:
    ```
    <｜tool▁calls▁begin｜><｜tool▁call▁begin｜>function<｜tool▁sep｜>{function_name}\n```json\n{json_arguments}\n```<｜tool▁calls▁end｜><｜end▁of▁sentence｜>
    ```
    Examples:
    ```
    <｜tool▁calls▁begin｜><｜tool▁call▁begin｜>function<｜tool▁sep｜>get_current_weather\n```json\n{"location": "Tokyo"}\n```<｜tool▁call▁end｜>\n<｜tool▁call▁begin｜>function<｜tool▁sep｜>get_current_weather\n```json\n{"location": "Paris"}\n```<｜tool▁call▁end｜><｜tool▁calls▁end｜><｜end▁of▁sentence｜>
    ```

    Key Components:
    - Tool Calls Section: Wrapped between `<｜tool▁calls▁begin｜>` and `<｜tool▁calls▁end｜>`
    - Individual Tool Call: Wrapped between `<｜tool▁call▁begin｜>` and `<｜tool▁call▁end｜>`
    - Function Declaration: `function<｜tool▁sep｜>{function_name}`
    - Arguments: JSON code block between ````json` and ````
    - Supports multiple tool calls

    Reference: https://huggingface.co/deepseek-ai/DeepSeek-V3-0324?chat_template=default
    """

```
**EN:** This block adds supporting state or helper logic inside `DeepSeekV3Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DeepSeekV3Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 45-52: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<｜tool▁calls▁begin｜>"
        self.eot_token = "<｜tool▁calls▁end｜>"
        self.func_call_regex = r"<｜tool▁call▁begin｜>.*?<｜tool▁call▁end｜>"
        self.func_detail_regex = r"<｜tool▁call▁begin｜>(.*)<｜tool▁sep｜>(.*)\n```json\n(.*)\n```<｜tool▁call▁end｜>"
        self._last_arguments = ""
        self.current_tool_id = -1
```
**EN:** This block defines the method `__init__` on `DeepSeekV3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 3 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `n`.
**CN:** 这一段定义了method `__init__`（属于 `DeepSeekV3Detector`），介绍了参数、初始化步骤，以及这部分deepseekv3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`n`。

### Lines 54-56: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a deepseek format tool call."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `DeepSeekV3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 3 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `DeepSeekV3Detector`），介绍了参数、初始化步骤，以及这部分deepseekv3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 58-65: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: ParseResult indicating success or failure, consumed text, leftover text, and parsed calls.
        """
```
**EN:** This block defines the method `detect_and_parse` on `DeepSeekV3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 3 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `DeepSeekV3Detector`），介绍了参数、初始化步骤，以及这部分deepseekv3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 66-86: Method `detect_and_parse` logic (part 1)
```python
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=normal_text, calls=[])
        match_result_list = re.findall(self.func_call_regex, text, re.DOTALL)
        calls = []
        try:
            for match_result in match_result_list:
                # Get function name
                func_detail = re.search(self.func_detail_regex, match_result, re.DOTALL)
                func_name = func_detail.group(2)
                func_args = func_detail.group(3)
                func_args = json.loads(func_args)
                # construct match_result for parse_base_json
                match_result = {"name": func_name, "parameters": func_args}
                calls.extend(self.parse_base_json(match_result, tools))
            return StreamingParseResult(normal_text=normal_text, calls=calls)
        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}")
            # return the normal text if parsing fails
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 3 style tool-call detection and normalization workflow. Notable operations include `find`, `strip`, `StreamingParseResult`, `findall`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`、`StreamingParseResult`、`findall`。

### Lines 88-93: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing tool calls for DeepSeekV3 format.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `DeepSeekV3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 3 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `DeepSeekV3Detector`），介绍了参数、初始化步骤，以及这部分deepseekv3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 94-121: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        # Check if we have a tool call (either the start token or individual tool call)
        has_tool_call = (
            self.bot_token in current_text or "<｜tool▁call▁begin｜>" in current_text
        )

        if not has_tool_call:
            self._buffer = ""
            for e_token in [self.eot_token, "```", "<｜tool▁call▁end｜>"]:
                if e_token in new_text:
                    new_text = new_text.replace(e_token, "")
            return StreamingParseResult(normal_text=new_text)

        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        calls: list[ToolCallItem] = []
        try:
            partial_match = re.search(
                pattern=r"<｜tool▁call▁begin｜>(.*)<｜tool▁sep｜>(.*)\n```json\n(.*)\n```.*",
                string=current_text,
                flags=re.DOTALL,
            )
            if partial_match:
                func_name = partial_match.group(2).strip()
                func_args_raw = partial_match.group(3).strip()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 3 style tool-call detection and normalization workflow. Notable operations include `call`, `replace`, `StreamingParseResult`, `_get_tool_indices`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `call`、`replace`、`StreamingParseResult`、`_get_tool_indices`。

### Lines 122-154: Method `parse_streaming_increment` logic (part 2)
```python

                # Initialize state if this is the first tool call
                if self.current_tool_id == -1:
                    self.current_tool_id = 0
                    self.prev_tool_call_arr = []
                    self.streamed_args_for_tool = [""]

                # Ensure we have enough entries in our tracking arrays
                while len(self.prev_tool_call_arr) <= self.current_tool_id:
                    self.prev_tool_call_arr.append({})
                while len(self.streamed_args_for_tool) <= self.current_tool_id:
                    self.streamed_args_for_tool.append("")

                if not self.current_tool_name_sent:
                    calls.append(
                        ToolCallItem(
                            tool_index=self.current_tool_id,
                            name=func_name,
                            parameters="",
                        )
                    )
                    self.current_tool_name_sent = True
                    # Store the tool call info for serving layer completions endpoint
                    self.prev_tool_call_arr[self.current_tool_id] = {
                        "name": func_name,
                        "arguments": {},
                    }
                else:
                    argument_diff = (
                        func_args_raw[len(self._last_arguments) :]
                        if func_args_raw.startswith(self._last_arguments)
                        else func_args_raw
                    )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 3 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`startswith`。

### Lines 155-190: Method `parse_streaming_increment` logic (part 3)
```python

                    if argument_diff:
                        calls.append(
                            ToolCallItem(
                                tool_index=self.current_tool_id,
                                name=None,
                                parameters=argument_diff,
                            )
                        )
                        self._last_arguments += argument_diff
                        self.streamed_args_for_tool[
                            self.current_tool_id
                        ] += argument_diff

                    if _is_complete_json(func_args_raw):
                        # Update the stored arguments
                        try:
                            parsed_args = json.loads(func_args_raw)
                            self.prev_tool_call_arr[self.current_tool_id][
                                "arguments"
                            ] = parsed_args
                        except json.JSONDecodeError:
                            pass

                        # Find the end of the current tool call and remove only that part from buffer
                        tool_call_end_pattern = (
                            r"<｜tool▁call▁begin｜>.*?<｜tool▁call▁end｜>"
                        )
                        match = re.search(
                            tool_call_end_pattern, current_text, re.DOTALL
                        )
                        if match:
                            # Remove the completed tool call from buffer, keep any remaining content
                            self._buffer = current_text[match.end() :]
                        else:
                            self._buffer = ""
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 3 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `_is_complete_json`, `loads`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`_is_complete_json`、`loads`。

### Lines 191-202: Method `parse_streaming_increment` logic (part 4)
```python

                        result = StreamingParseResult(normal_text="", calls=calls)
                        self.current_tool_id += 1
                        self._last_arguments = ""
                        self.current_tool_name_sent = False
                        return result

            return StreamingParseResult(normal_text="", calls=calls)

        except Exception as e:
            logger.error(f"Error in parse_streaming_increment: {e}")
            return StreamingParseResult(normal_text=current_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding deepseekv 3 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `error`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的deepseekv3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`error`。

### Lines 204-211: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin="<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>function<｜tool▁sep｜>"
            + name
            + "\n```json\n",
            end="\n```<｜tool▁call▁end｜><｜tool▁calls▁end｜>",
            trigger="<｜tool▁calls▁begin｜>",
        )
```
**EN:** This block defines the method `structure_info` on `DeepSeekV3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of deepseekv 3 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `DeepSeekV3Detector`），介绍了参数、初始化步骤，以及这部分deepseekv3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `DeepSeekV3Detector`: Class that encapsulates deep seek v 3 detector behavior in this module. / `DeepSeekV3Detector`：封装与“deepseekv3检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.utils`
