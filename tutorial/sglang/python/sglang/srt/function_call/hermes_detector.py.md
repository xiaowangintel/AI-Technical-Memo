# hermes_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/hermes_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a hermes detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Hermes 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

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
**EN:** This block gathers the imports and module-level setup for hermes style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Hermes风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 17-17: Class `HermesDetector` declaration
```python
class HermesDetector(BaseFormatDetector):
```
**EN:** This block declares the class `HermesDetector` and establishes its responsibility inside hermes style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `HermesDetector`.
**CN:** 这一段声明了类 `HermesDetector`，并说明它在Hermes风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `HermesDetector`。

### Lines 18-24: Supporting state inside `HermesDetector`
```python
    """
    Detector for Hermes tool call format.

    Format:
        <tool_call>{"name": "...", "arguments": {...}}</tool_call>
    """

```
**EN:** This block adds supporting state or helper logic inside `HermesDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `HermesDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 25-32: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<tool_call>"
        self.eot_token = "</tool_call>"
        self.tool_call_regex = re.compile(
            r"<tool_call>(.*?)</tool_call>|<tool_call>(.*)", re.DOTALL
        )
        self._normal_text_buffer = ""
```
**EN:** This block defines the method `__init__` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `compile`.
**CN:** 这一段定义了method `__init__`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`compile`。

### Lines 34-35: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 37-60: Method `detect_and_parse`
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.
        """
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=normal_text, calls=[])

        calls = []
        try:
            for match in self.tool_call_regex.findall(text):
                raw = match[0] or match[1]
                if not raw:
                    continue
                parsed = json.loads(raw.strip())
                if isinstance(parsed, list):
                    calls.extend(self.parse_base_json(parsed, tools))
                else:
                    calls.extend(self.parse_base_json(parsed, tools))
            return StreamingParseResult(normal_text=normal_text, calls=calls)
        except Exception as e:
            logger.error(f"Error in detect_and_parse: {e}")
            return StreamingParseResult(normal_text=text)
```
**EN:** This block defines the method `detect_and_parse` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `find`, `strip`, `StreamingParseResult`, `findall`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `find`、`strip`、`StreamingParseResult`、`findall`。

### Lines 62-83: Method `_clean_normal_text`
```python
    def _clean_normal_text(self, text: str) -> str:
        if not text:
            return text

        self._normal_text_buffer += text

        if self.eot_token in self._normal_text_buffer:
            cleaned = self._normal_text_buffer.replace(self.eot_token, "")
            self._normal_text_buffer = ""
            return cleaned

        partial_len = self._ends_with_partial_token(
            self._normal_text_buffer, self.eot_token
        )
        if partial_len:
            safe_text = self._normal_text_buffer[:-partial_len]
            self._normal_text_buffer = self._normal_text_buffer[-partial_len:]
            return safe_text

        cleaned = self._normal_text_buffer
        self._normal_text_buffer = ""
        return cleaned
```
**EN:** This block defines the method `_clean_normal_text` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `_clean_normal_text`. Notable operations include `replace`, `_ends_with_partial_token`.
**CN:** 这一段定义了method `_clean_normal_text`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_clean_normal_text`。 值得注意的操作包括 `replace`、`_ends_with_partial_token`。

### Lines 85-90: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming parsing: handle normal text, partial tags, and tool calls.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 91-113: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        if self.bot_token not in current_text:
            partial_len = self._ends_with_partial_token(current_text, self.bot_token)
            if partial_len:
                safe_text = current_text[:-partial_len]
                self._buffer = current_text[-partial_len:]
            else:
                safe_text = current_text
                self._buffer = ""
            return StreamingParseResult(normal_text=self._clean_normal_text(safe_text))

        bot_pos = current_text.find(self.bot_token)
        if bot_pos > 0:
            normal_text = current_text[:bot_pos]
            self._buffer = current_text[bot_pos:]
            return StreamingParseResult(normal_text=normal_text)

        result = super().parse_streaming_increment(new_text="", tools=tools)
        if result.normal_text:
            result.normal_text = self._clean_normal_text(result.normal_text)
        return result
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding hermes style tool-call detection and normalization workflow. Notable operations include `_ends_with_partial_token`, `StreamingParseResult`, `_clean_normal_text`, `find`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Hermes风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_ends_with_partial_token`、`StreamingParseResult`、`_clean_normal_text`、`find`。

### Lines 115-120: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin='<tool_call>{"name":"' + name + '", "arguments":',
            end="}</tool_call>",
            trigger="<tool_call>",
        )
```
**EN:** This block defines the method `structure_info` on `HermesDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of hermes style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `HermesDetector`），介绍了参数、初始化步骤，以及这部分Hermes风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `HermesDetector`: Class that encapsulates hermes detector behavior in this module. / `HermesDetector`：封装与“Hermes检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
