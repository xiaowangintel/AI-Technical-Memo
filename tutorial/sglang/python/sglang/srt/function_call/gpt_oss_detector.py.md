# gpt_oss_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/gpt_oss_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a gpt oss detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 gptoss 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import json
import logging
import re
from typing import List, Optional

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)
from sglang.srt.parser.harmony_parser import HarmonyParser

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for gpt oss style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与gptoss风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 19-19: Class `GptOssDetector` declaration
```python
class GptOssDetector(BaseFormatDetector):
```
**EN:** This block declares the class `GptOssDetector` and establishes its responsibility inside gpt oss style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `GptOssDetector`.
**CN:** 这一段声明了类 `GptOssDetector`，并说明它在gptoss风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `GptOssDetector`。

### Lines 20-26: Supporting state inside `GptOssDetector`
```python
    """
    Detector for T4-style function calls using HarmonyParser.

    Handles tool calls in the format:
    <|channel|>commentary to={namespace.function}<|constrain|>json<|message|>{args}<|call|>
    """

```
**EN:** This block adds supporting state or helper logic inside `GptOssDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `GptOssDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 27-37: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.harmony_parser = HarmonyParser()
        self.bot_token = "<|start|>assistant<|channel|>commentary"
        self.eot_token = "<|call|>"

        # Pattern to extract function name and JSON from tool_call event content
        self.tool_extract_pattern = re.compile(
            r"to=([a-zA-Z_][a-zA-Z0-9_.-]*)\s*<\|constrain\|>json<\|message\|>(.*?)(?:<\|call\|>|$)",
            re.DOTALL,
        )
```
**EN:** This block defines the method `__init__` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`, `HarmonyParser`, `compile`.
**CN:** 这一段定义了method `__init__`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`、`HarmonyParser`、`compile`。

### Lines 39-41: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if text contains TypeScript-style function call markers."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 43-44: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """Parse TypeScript-style function calls from complete text."""
```
**EN:** This block defines the method `detect_and_parse` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 45-71: Method `detect_and_parse` logic (part 1)
```python
        if not self.has_tool_call(text):
            return StreamingParseResult(normal_text=text, calls=[])

        # Parse with HarmonyParser
        events = self.harmony_parser.parse(text)
        # Flush buffer for complete parsing
        events += self.harmony_parser.parse("")

        tool_indices = self._get_tool_indices(tools)
        calls = []
        normal_parts = []
        tool_index = 0

        for event in events:
            if event.event_type == "tool_call":
                # Extract tool call from event content
                tool_call = self._extract_tool_call_from_event(
                    event.raw_text if event.raw_text else event.content,
                    tool_indices,
                    tool_index,
                )
                if tool_call:
                    calls.append(tool_call)
                    tool_index += 1
            elif event.event_type == "normal":
                normal_parts.append(event.content)
            # Ignore reasoning events in function call context
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `has_tool_call`, `StreamingParseResult`, `parse`, `_get_tool_indices`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `has_tool_call`、`StreamingParseResult`、`parse`、`_get_tool_indices`。

### Lines 72-74: Method `detect_and_parse` logic (part 2)
```python

        normal_text = " ".join(normal_parts).strip()
        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `join`, `strip`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `join`、`strip`、`StreamingParseResult`。

### Lines 76-79: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """Parse incremental streaming text for TypeScript-style function calls."""
```
**EN:** This block defines the method `parse_streaming_increment` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 80-106: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text

        # Always use HarmonyParser for parsing to ensure proper filtering
        events = self.harmony_parser.parse(new_text)

        # If there are no parsed events and the chunk contains no Harmony structural
        # markers, treat it as plain text and pass it through. This fixes a bug where
        # normal content was held in the buffer when tools were provided but not used.
        if not events:
            has_harmony_markers = any(
                marker in self._buffer
                for marker in (
                    "<|start|>",
                    "<|channel|>",
                    "<|message|>",
                    "<|constrain|>",
                    "<|end|>",
                    "<|call|>",
                    "<|return|>",
                    "assistantfinal",
                )
            )
            if not has_harmony_markers:
                # Plain text with no tool markers — emit as normal content
                out = self._buffer
                self._buffer = ""
                return StreamingParseResult(normal_text=out, calls=[])
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `parse`, `in`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `parse`、`in`、`StreamingParseResult`。

### Lines 107-135: Method `parse_streaming_increment` logic (part 2)
```python

        # Quick check if we might have tool calls
        if (
            "<|channel|>commentary to=" not in self._buffer
            and not self.current_tool_name_sent
        ):
            # No tool calls detected, check for final content
            if (
                "<|channel|>final" in self._buffer
                or "assistantfinal" in self._buffer.lower()
            ):
                # Extract normal text from events
                normal_text = "".join(
                    [e.content for e in events if e.event_type == "normal"]
                )
                if normal_text:
                    self._buffer = ""
                    return StreamingParseResult(normal_text=normal_text, calls=[])

            # For other content, extract normal text from events (with filtering applied)
            normal_text = "".join(
                [e.content for e in events if e.event_type == "normal"]
            )
            if normal_text or events:
                self._buffer = ""
                return StreamingParseResult(normal_text=normal_text, calls=[])
            else:
                # No events processed, continue buffering
                return StreamingParseResult(normal_text="", calls=[])
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `lower`, `join`, `StreamingParseResult`, `events`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `lower`、`join`、`StreamingParseResult`、`events`。

### Lines 136-168: Method `parse_streaming_increment` logic (part 3)
```python

        if not events:
            # No complete events yet
            return StreamingParseResult(normal_text="", calls=[])

        # Initialize state if needed
        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        calls = []
        normal_text = ""

        for event in events:
            if event.event_type == "tool_call":
                # We got a complete tool call from HarmonyParser
                tool_call_info = self._extract_tool_call_from_event(
                    event.raw_text if event.raw_text else event.content,
                    self._tool_indices,
                    self.current_tool_id if self.current_tool_id >= 0 else 0,
                )

                if tool_call_info:
                    # Initialize state if first tool
                    if self.current_tool_id == -1:
                        self.current_tool_id = 0
                        self.prev_tool_call_arr = []
                        self.streamed_args_for_tool = [""]

                    # Ensure arrays are large enough
                    while len(self.prev_tool_call_arr) <= self.current_tool_id:
                        self.prev_tool_call_arr.append({})
                    while len(self.streamed_args_for_tool) <= self.current_tool_id:
                        self.streamed_args_for_tool.append("")
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `_get_tool_indices`, `_extract_tool_call_from_event`, `append`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`_get_tool_indices`、`_extract_tool_call_from_event`、`append`。

### Lines 169-195: Method `parse_streaming_increment` logic (part 4)
```python

                    # Store tool call info
                    self.prev_tool_call_arr[self.current_tool_id] = {
                        "name": tool_call_info.name,
                        "arguments": json.loads(tool_call_info.parameters),
                    }

                    # Emit the complete tool call at once
                    # (Could be modified to emit name first, then args, if needed)
                    calls.append(tool_call_info)

                    # Mark as streamed
                    self.streamed_args_for_tool[self.current_tool_id] = (
                        tool_call_info.parameters
                    )

                    # Move to next tool
                    self.current_tool_id += 1
                    self.current_tool_name_sent = False

            elif event.event_type == "normal":
                normal_text += event.content

        # Clear buffer since HarmonyParser handles buffering
        self._buffer = ""

        return StreamingParseResult(normal_text=normal_text, calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `loads`, `append`, `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`、`append`、`StreamingParseResult`。

### Lines 197-204: Method `_extract_tool_call_from_event` signature and setup
```python
    def _extract_tool_call_from_event(
        self, content: str, tool_indices: dict, tool_index: int
    ) -> Optional[ToolCallItem]:
        """
        Extract tool call information from HarmonyParser event content.

        Content format: "commentary to=functions.get_weather<|constrain|>json<|message|>{...}"
        """
```
**EN:** This block defines the method `_extract_tool_call_from_event` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `_extract_tool_call_from_event`.
**CN:** 这一段定义了method `_extract_tool_call_from_event`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_extract_tool_call_from_event`。

### Lines 205-225: Method `_extract_tool_call_from_event` logic (part 1)
```python
        match = self.tool_extract_pattern.search(content)

        if not match:
            logger.debug(f"Could not extract tool call from: {content[:100]}")
            return None

        full_function_name = match.group(1)
        json_content = match.group(2)

        # Extract function name (last part after .)
        function_name = (
            full_function_name.split(".")[-1]
            if "." in full_function_name
            else full_function_name
        )

        # Check if tool exists
        if function_name not in tool_indices:
            logger.debug(f"Function {function_name} not in available tools")
            if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                return None  # Skip unknown tools (default legacy behavior)
```
**EN:** This block continues `_extract_tool_call_from_event` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `search`, `debug`, `group`, `name`.
**CN:** 这一段延续了 `_extract_tool_call_from_event` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`debug`、`group`、`name`。

### Lines 226-238: Method `_extract_tool_call_from_event` logic (part 2)
```python

        # Parse JSON arguments
        try:
            arguments = json.loads(json_content) if json_content.strip() else {}
        except json.JSONDecodeError as e:
            logger.debug(f"Failed to parse JSON arguments: {e}")
            return None

        return ToolCallItem(
            tool_index=tool_index,
            name=function_name,
            parameters=json.dumps(arguments, ensure_ascii=False),
        )
```
**EN:** This block continues `_extract_tool_call_from_event` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gpt oss style tool-call detection and normalization workflow. Notable operations include `loads`, `strip`, `debug`, `ToolCallItem`.
**CN:** 这一段延续了 `_extract_tool_call_from_event` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的gptoss风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`、`strip`、`debug`、`ToolCallItem`。

### Lines 240-241: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        raise NotImplementedError("structure_info not used with HarmonyParser")
```
**EN:** This block defines the method `structure_info` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `structure_info`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `NotImplementedError`。

### Lines 243-244: Method `get_structural_tag_name`
```python
    def get_structural_tag_name(self) -> str:
        return "harmony"
```
**EN:** This block defines the method `get_structural_tag_name` on `GptOssDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of gpt oss style tool-call detection and normalization. Definitions introduced here include `get_structural_tag_name`.
**CN:** 这一段定义了method `get_structural_tag_name`（属于 `GptOssDetector`），介绍了参数、初始化步骤，以及这部分gptoss风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_structural_tag_name`。

## Key Concepts / 关键概念
- `GptOssDetector`: Class that encapsulates gpt oss detector behavior in this module. / `GptOssDetector`：封装与“gptoss检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.parser.harmony_parser`
