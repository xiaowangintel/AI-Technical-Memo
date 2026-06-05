# qwen25_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/qwen25_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a qwen 25 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Qwen25 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup
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
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for qwen 25 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Qwen25风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 17-17: Class `Qwen25Detector` declaration
```python
class Qwen25Detector(BaseFormatDetector):
```
**EN:** This block declares the class `Qwen25Detector` and establishes its responsibility inside qwen 25 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Qwen25Detector`.
**CN:** 这一段声明了类 `Qwen25Detector`，并说明它在Qwen25风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Qwen25Detector`。

### Lines 18-32: Supporting state inside `Qwen25Detector`
```python
    """
    Detector for Qwen 2.5 and Qwen 3 model function call format.

    Format Structure:
    ```
    <tool_call>\n{"name":"func1", "arguments":{...}}\n</tool_call>\n<tool_call>\n{"name":"func2", "arguments":{...}}\n</tool_call>
    ```

    Key Components:
    - Tool Call Tags: `<tool_call>` and `</tool_call>` wrap each individual call
    - Function Call Object: JSON object with "name" and "arguments" fields

    Reference: https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct?chat_template=default
    """

```
**EN:** This block adds supporting state or helper logic inside `Qwen25Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `Qwen25Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 33-41: Method `__init__`
```python
    def __init__(self):
        """
        Initializes the detector with necessary state variables.
        """
        super().__init__()
        self.bot_token = "<tool_call>\n"
        self.eot_token = "\n</tool_call>"
        self.tool_call_separator = "\n"
        self._normal_text_buffer = ""  # Buffer for handling partial end tokens
```
**EN:** This block defines the method `__init__` on `Qwen25Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 25 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `Qwen25Detector`），介绍了参数、初始化步骤，以及这部分Qwen25风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 43-45: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a Qwen 2.5 format tool call."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `Qwen25Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 25 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Qwen25Detector`），介绍了参数、初始化步骤，以及这部分Qwen25风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 47-54: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: ParseResult indicating success or failure, consumed text, leftover text, and parsed calls.
        """
```
**EN:** This block defines the method `detect_and_parse` on `Qwen25Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 25 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Qwen25Detector`），介绍了参数、初始化步骤，以及这部分Qwen25风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 55-73: Method `detect_and_parse` logic (part 1)
```python
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=normal_text, calls=[])

        # Find all <tool_call>\n...\n</tool_call> blocks
        pattern = rf"{re.escape(self.bot_token)}(.*?){re.escape(self.eot_token)}"
        match_result_list = re.findall(pattern, text, re.DOTALL)
        calls = []
        for match_result in match_result_list:
            try:
                parsed_call = json.loads(match_result.strip())
                calls.extend(self.parse_base_json(parsed_call, tools))
            except json.JSONDecodeError as e:
                logger.warning(
                    f"Failed to parse JSON part: {match_result}, JSON parse error: {str(e)}"
                )
                continue
        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 25 style tool-call detection and normalization workflow. Notable operations include `find`, `strip`, `StreamingParseResult`, `escape`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen25风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`、`StreamingParseResult`、`escape`。

### Lines 75-81: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for Qwen 2.5 tool calls.
        Uses base class implementation with buffering to handle partial end tokens.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `Qwen25Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 25 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Qwen25Detector`），介绍了参数、初始化步骤，以及这部分Qwen25风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 82-100: Method `parse_streaming_increment` logic (part 1)
```python
        result = super().parse_streaming_increment(new_text, tools)

        # Handle partial end tokens that are streamed character by character
        if result.normal_text:
            self._normal_text_buffer += result.normal_text

            # Check if buffer contains complete end token (without leading newline)
            end_token_without_newline = self.eot_token[1:]  # "</tool_call>"
            if end_token_without_newline in self._normal_text_buffer:
                cleaned_text = self._normal_text_buffer.replace(
                    end_token_without_newline, ""
                )
                self._normal_text_buffer = ""
                result.normal_text = cleaned_text
            else:
                # Check if buffer might contain partial end token at the end
                partial_match_len = self._ends_with_partial_token(
                    self._normal_text_buffer, end_token_without_newline
                )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 25 style tool-call detection and normalization workflow. Notable operations include `parse_streaming_increment`, `token`, `replace`, `_ends_with_partial_token`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen25风格的工具调用检测与规范化工作流。 值得注意的操作包括 `parse_streaming_increment`、`token`、`replace`、`_ends_with_partial_token`。

### Lines 101-113: Method `parse_streaming_increment` logic (part 2)
```python

                if partial_match_len:
                    # Keep potential partial match in buffer, return the rest
                    result.normal_text = self._normal_text_buffer[:-partial_match_len]
                    self._normal_text_buffer = self._normal_text_buffer[
                        -partial_match_len:
                    ]
                else:
                    # No partial match, return all buffered text
                    result.normal_text = self._normal_text_buffer
                    self._normal_text_buffer = ""

        return result
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding qwen 25 style tool-call detection and normalization workflow.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Qwen25风格的工具调用检测与规范化工作流。

### Lines 115-120: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin='<tool_call>\n{"name":"' + name + '", "arguments":',
            end="}\n</tool_call>",
            trigger="<tool_call>",
        )
```
**EN:** This block defines the method `structure_info` on `Qwen25Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of qwen 25 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `Qwen25Detector`），介绍了参数、初始化步骤，以及这部分Qwen25风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `Qwen25Detector`: Class that encapsulates qwen 25 detector behavior in this module. / `Qwen25Detector`：封装与“Qwen25检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
