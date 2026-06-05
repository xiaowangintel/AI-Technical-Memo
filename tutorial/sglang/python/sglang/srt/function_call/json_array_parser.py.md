# json_array_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/json_array_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides a lightweight incremental parser for streamed JSON array fragments. Detectors use it when tool calls arrive in partially complete chunks. / 该文件提供用于流式 JSON 数组片段的轻量级增量解析器，供检测器在工具调用分块到达且尚未完整时使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
```python
from typing import List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import StreamingParseResult


```
**EN:** This block gathers the imports and module-level setup for incremental JSON array parsing for tool calls. The imported modules show which runtime services, schemas, or backend components this file depends on.
**CN:** 这一段汇集了与面向工具调用的增量 JSON 数组解析相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。

### Lines 8-8: Class `JsonArrayParser` declaration
```python
class JsonArrayParser(BaseFormatDetector):
```
**EN:** This block declares the class `JsonArrayParser` and establishes its responsibility inside incremental JSON array parsing for tool calls. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `JsonArrayParser`.
**CN:** 这一段声明了类 `JsonArrayParser`，并说明它在面向工具调用的增量 JSON 数组解析中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `JsonArrayParser`。

### Lines 9-15: Supporting state inside `JsonArrayParser`
```python
    """
    Parser for JSON array tool calls when JSON schema constraints are active.

    This parser is used when tool_choice="required" or a specific tool is named,
    bypassing model-specific parsers in favor of direct JSON array parsing.
    """

```
**EN:** This block adds supporting state or helper logic inside `JsonArrayParser`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `JsonArrayParser` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 16-21: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        # Configure for JSON array parsing
        self.bot_token = "["
        self.eot_token = "]"
        self.tool_call_separator = ","
```
**EN:** This block defines the method `__init__` on `JsonArrayParser`. It introduces the parameters, setup steps, and the main entry point for this piece of incremental JSON array parsing for tool calls. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `JsonArrayParser`），介绍了参数、初始化步骤，以及这部分面向工具调用的增量 JSON 数组解析逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 23-27: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """
        Check if the given text contains a JSON tool call (array or single object).
        """
        return "[" in text or "{" in text
```
**EN:** This block defines the method `has_tool_call` on `JsonArrayParser`. It introduces the parameters, setup steps, and the main entry point for this piece of incremental JSON array parsing for tool calls. Definitions introduced here include `has_tool_call`. Notable operations include `call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `JsonArrayParser`），介绍了参数、初始化步骤，以及这部分面向工具调用的增量 JSON 数组解析逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `call`。

### Lines 29-35: Method `detect_and_parse`
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        Parse JSON tool calls using the base class implementation.
        """
        raise NotImplementedError(
            "Detect and parse not supported for JSON schema constraints."
        )
```
**EN:** This block defines the method `detect_and_parse` on `JsonArrayParser`. It introduces the parameters, setup steps, and the main entry point for this piece of incremental JSON array parsing for tool calls. Definitions introduced here include `detect_and_parse`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `JsonArrayParser`），介绍了参数、初始化步骤，以及这部分面向工具调用的增量 JSON 数组解析逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `NotImplementedError`。

### Lines 37-43: Method `parse_streaming_increment`
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing with tool validation.
        """
        return super().parse_streaming_increment(new_text, tools)
```
**EN:** This block defines the method `parse_streaming_increment` on `JsonArrayParser`. It introduces the parameters, setup steps, and the main entry point for this piece of incremental JSON array parsing for tool calls. Definitions introduced here include `parse_streaming_increment`. Notable operations include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `JsonArrayParser`），介绍了参数、初始化步骤，以及这部分面向工具调用的增量 JSON 数组解析逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `parse_streaming_increment`。

### Lines 45-51: Method `structure_info`
```python
    def structure_info(self) -> callable:
        """
        Return a function that creates StructureInfo for constrained generation.
        This is not used for JSON schema constraints as they are handled
        by the constraint backends directly.
        """
        raise NotImplementedError("structure_info not used for JSON schema constraints")
```
**EN:** This block defines the method `structure_info` on `JsonArrayParser`. It introduces the parameters, setup steps, and the main entry point for this piece of incremental JSON array parsing for tool calls. Definitions introduced here include `structure_info`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `structure_info`（属于 `JsonArrayParser`），介绍了参数、初始化步骤，以及这部分面向工具调用的增量 JSON 数组解析逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `NotImplementedError`。

## Key Concepts / 关键概念
- `JsonArrayParser`: Class that encapsulates json array parser behavior in this module. / `JsonArrayParser`：封装与“JSON数组解析器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
