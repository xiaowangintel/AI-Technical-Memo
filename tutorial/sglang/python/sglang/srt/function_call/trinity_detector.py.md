# trinity_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/trinity_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a trinity detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Trinity 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module-level constants and helper logic
```python
import logging
from typing import List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.core_types import StreamingParseResult
from sglang.srt.function_call.qwen25_detector import Qwen25Detector

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for trinity style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与Trinity风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 11-11: Class `TrinityDetector` declaration
```python
class TrinityDetector(Qwen25Detector):
```
**EN:** This block declares the class `TrinityDetector` and establishes its responsibility inside trinity style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `TrinityDetector`.
**CN:** 这一段声明了类 `TrinityDetector`，并说明它在Trinity风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `TrinityDetector`。

### Lines 12-20: Supporting state inside `TrinityDetector`
```python
    """
    Detector for Trinity models using Qwen-style function call format.

    This detector extends Qwen25Detector to handle tool calls that may appear
    inside <think> sections by stripping the think tags before parsing.

    Reference: https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct?chat_template=default
    """

```
**EN:** This block adds supporting state or helper logic inside `TrinityDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `TrinityDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 21-23: Method `_strip_think_tags`
```python
    def _strip_think_tags(self, text: str) -> str:
        """Remove <think> and </think> tags, keeping the content inside."""
        return text.replace("<think>", "").replace("</think>", "")
```
**EN:** This block defines the method `_strip_think_tags` on `TrinityDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of trinity style tool-call detection and normalization. Definitions introduced here include `_strip_think_tags`. Notable operations include `replace`.
**CN:** 这一段定义了method `_strip_think_tags`（属于 `TrinityDetector`），介绍了参数、初始化步骤，以及这部分Trinity风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_strip_think_tags`。 值得注意的操作包括 `replace`。

### Lines 25-27: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a tool call."""
        return super().has_tool_call(self._strip_think_tags(text))
```
**EN:** This block defines the method `has_tool_call` on `TrinityDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of trinity style tool-call detection and normalization. Definitions introduced here include `has_tool_call`. Notable operations include `has_tool_call`, `_strip_think_tags`.
**CN:** 这一段定义了method `has_tool_call`（属于 `TrinityDetector`），介绍了参数、初始化步骤，以及这部分Trinity风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `has_tool_call`、`_strip_think_tags`。

### Lines 29-33: Method `detect_and_parse`
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.
        """
        return super().detect_and_parse(self._strip_think_tags(text), tools)
```
**EN:** This block defines the method `detect_and_parse` on `TrinityDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of trinity style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `detect_and_parse`, `_strip_think_tags`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `TrinityDetector`），介绍了参数、初始化步骤，以及这部分Trinity风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `detect_and_parse`、`_strip_think_tags`。

### Lines 35-43: Method `parse_streaming_increment`
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for tool calls.
        """
        return super().parse_streaming_increment(
            self._strip_think_tags(new_text), tools
        )
```
**EN:** This block defines the method `parse_streaming_increment` on `TrinityDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of trinity style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `parse_streaming_increment`, `_strip_think_tags`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `TrinityDetector`），介绍了参数、初始化步骤，以及这部分Trinity风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `parse_streaming_increment`、`_strip_think_tags`。

## Key Concepts / 关键概念
- `TrinityDetector`: Class that encapsulates trinity detector behavior in this module. / `TrinityDetector`：封装与“Trinity检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.qwen25_detector`
