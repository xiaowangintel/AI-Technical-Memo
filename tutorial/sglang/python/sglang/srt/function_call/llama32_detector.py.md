# llama32_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/llama32_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a llama 32 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 Llama32 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import ast
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
**EN:** This block gathers the imports and module-level setup for llama 32 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与Llama32风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 18-18: Class `Llama32Detector` declaration
```python
class Llama32Detector(BaseFormatDetector):
```
**EN:** This block declares the class `Llama32Detector` and establishes its responsibility inside llama 32 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `Llama32Detector`.
**CN:** 这一段声明了类 `Llama32Detector`，并说明它在Llama32风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `Llama32Detector`。

### Lines 19-27: Supporting state inside `Llama32Detector`
```python
    """
    Detector for Llama 3.2 models with json tool call format.

    Format Structure:
    ```
    <python_tag>{"name":"xxx", "arguments":{...}}
    ```
    """

```
**EN:** This block adds supporting state or helper logic inside `Llama32Detector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `Llama32Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 28-35: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<|python_tag|>"
        # NOTE: technically Llama3.2 doesn't support well with parallel tool calls
        # They need specific prompt engineering to support parallel tool calls
        # Here we use ';' as the separator, which might have compatibility issues
        # if users define to use a different separator in their prompt
        self.tool_call_separator = ";"
```
**EN:** This block defines the method `__init__` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 37-45: Method `_convert_python_dict_to_json`
```python
    def _convert_python_dict_to_json(self, text: str) -> str:
        """Convert Python dict strings to JSON format."""
        try:
            parsed = ast.literal_eval(text.strip())
            if isinstance(parsed, dict):
                return json.dumps(parsed, ensure_ascii=False)
        except:
            pass
        return text
```
**EN:** This block defines the method `_convert_python_dict_to_json` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `_convert_python_dict_to_json`. Notable operations include `literal_eval`, `strip`, `dumps`.
**CN:** 这一段定义了method `_convert_python_dict_to_json`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_convert_python_dict_to_json`。 值得注意的操作包括 `literal_eval`、`strip`、`dumps`。

### Lines 47-51: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a Llama 3.2 format tool call."""
        # depending on the prompt format the Llama model may or may not
        # prefix the output with the <|python_tag|> token
        return "<|python_tag|>" in text or text.startswith("{")
```
**EN:** This block defines the method `has_tool_call` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`. Notable operations include `startswith`.
**CN:** 这一段定义了method `has_tool_call`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。 值得注意的操作包括 `startswith`。

### Lines 53-54: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """Parse function calls from text, handling multiple JSON objects."""
```
**EN:** This block defines the method `detect_and_parse` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 55-78: Method `detect_and_parse` logic (part 1)
```python
        if "<|python_tag|>" not in text and not text.startswith("{"):
            return StreamingParseResult(normal_text=text, calls=[])

        if "<|python_tag|>" in text:
            normal_text, action_text = text.split("<|python_tag|>", maxsplit=1)
        else:
            normal_text, action_text = "", text

        decoder = json.JSONDecoder()
        idx = 0
        safe_idx = idx  # the index of the last valid JSON object
        all_actions = []
        action_text_len = len(action_text)
        while idx < action_text_len:
            try:
                obj, end = decoder.raw_decode(action_text[idx:])
                all_actions.append(obj)
                idx += end + len(self.tool_call_separator)
                safe_idx = idx
            except json.JSONDecodeError:
                # Try Python dict conversion as fallback
                try:
                    dict_end = idx
                    brace_count = 0
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding llama 32 style tool-call detection and normalization workflow. Notable operations include `startswith`, `StreamingParseResult`, `split`, `JSONDecoder`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Llama32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `startswith`、`StreamingParseResult`、`split`、`JSONDecoder`。

### Lines 79-103: Method `detect_and_parse` logic (part 2)
```python
                    for i in range(idx, action_text_len):
                        if action_text[i] == "{":
                            brace_count += 1
                        elif action_text[i] == "}":
                            brace_count -= 1
                            if brace_count == 0:
                                dict_end = i + 1
                                break

                    if dict_end > idx:
                        potential_dict = action_text[idx:dict_end]
                        json_version = self._convert_python_dict_to_json(potential_dict)
                        if json_version != potential_dict:
                            obj, _ = decoder.raw_decode(json_version)
                            all_actions.append(obj)
                            idx = dict_end + len(self.tool_call_separator)
                            safe_idx = idx
                            continue
                except:
                    pass

                next_obj_start = action_text.find('{"name":', idx + 1)
                if next_obj_start == -1:
                    break
                idx = next_obj_start
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding llama 32 style tool-call detection and normalization workflow. Notable operations include `_convert_python_dict_to_json`, `raw_decode`, `append`, `find`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Llama32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `_convert_python_dict_to_json`、`raw_decode`、`append`、`find`。

### Lines 104-113: Method `detect_and_parse` logic (part 3)
```python

        # Only process if we found valid JSON objects
        calls = self.parse_base_json(all_actions, tools) if all_actions else []
        # Use safe_idx to avoid idx containing the last part of an invalid JSON object
        trailing_text = (
            action_text[safe_idx:].strip() if safe_idx < action_text_len else ""
        )
        return StreamingParseResult(
            normal_text=normal_text + trailing_text, calls=calls
        )
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding llama 32 style tool-call detection and normalization workflow. Notable operations include `parse_base_json`, `strip`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的Llama32风格的工具调用检测与规范化工作流。 值得注意的操作包括 `parse_base_json`、`strip`、`StreamingParseResult`。

### Lines 115-137: Method `parse_streaming_increment`
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """Override to handle Python dict format in streaming."""
        # First try with converted Python dict
        self._buffer += new_text
        converted_buffer = self._buffer

        # Convert Python dict syntax to JSON
        converted_buffer = re.sub(r"'([^']*)':", r'"\1":', converted_buffer)
        converted_buffer = re.sub(r":\s*'([^']*)'", r': "\1"', converted_buffer)

        # Temporarily replace buffer for parsing
        original_buffer = self._buffer
        self._buffer = converted_buffer

        try:
            result = super().parse_streaming_increment("", tools)
            return result
        except:
            # Fall back to original buffer
            self._buffer = original_buffer
            return super().parse_streaming_increment(new_text, tools)
```
**EN:** This block defines the method `parse_streaming_increment` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`. Notable operations include `sub`, `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。 值得注意的操作包括 `sub`、`parse_streaming_increment`。

### Lines 139-144: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        return lambda name: StructureInfo(
            begin='<|python_tag|>{"name":"' + name + '", "arguments":',
            end="}",
            trigger="<|python_tag|>",
        )
```
**EN:** This block defines the method `structure_info` on `Llama32Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of llama 32 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `Llama32Detector`），介绍了参数、初始化步骤，以及这部分Llama32风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `Llama32Detector`: Class that encapsulates llama 32 detector behavior in this module. / `Llama32Detector`：封装与“Llama32检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
