# internlm_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/internlm_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a internlm detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 InternLM 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module-level constants and helper logic
```python
# modified from https://github.com/InternLM/lmdeploy/blob/main/lmdeploy/serve/openai/tool_parser/internlm2_parser.py

import json
import logging
import re
from typing import List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.environ import envs
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    StructureInfo,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for internlm style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `import`, `getLogger`.
**CN:** 这一段包含与InternLM风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `import`、`getLogger`。

### Lines 21-21: Class `InternlmDetector` declaration
```python
class InternlmDetector(BaseFormatDetector):
```
**EN:** This block declares the class `InternlmDetector` and establishes its responsibility inside internlm style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `InternlmDetector`.
**CN:** 这一段声明了类 `InternlmDetector`，并说明它在InternLM风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `InternlmDetector`。

### Lines 22-47: Supporting state inside `InternlmDetector`
```python
    """
    Detector for InternLM2/Intern-S1 model function call format.

    The InternLM format uses special tokens to delimit function calls
    with JSON for arguments.

    Format Structure:
    ```
    text<|action_start|> <|plugin|>
    {json}<|action_end|>
    ```

    Examples:
    ```
    What's the weather like?<|action_start|> <|plugin|>
    {"name": "get_weather", "parameters": {"location": "Tokyo"}}<|action_end|>
    ```

    Key Components:
    - Tool Call Start: `<|action_start|> <|plugin|>`
    - Tool Call End: `<|action_end|>`
    - Arguments: JSON object with `name` and `parameters`/`arguments`
    - Supports multiple sequential tool calls in both streaming and non-streaming modes

    """

```
**EN:** This block adds supporting state or helper logic inside `InternlmDetector`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `InternlmDetector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 48-52: Method `__init__`
```python
    def __init__(self):
        super().__init__()
        self.bot_token = "<|action_start|> <|plugin|>"
        self.eot_token = "<|action_end|>"
        self.position = 0
```
**EN:** This block defines the method `__init__` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 54-57: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains an InternLM format tool call."""
        has_call = self.bot_token in text
        return has_call
```
**EN:** This block defines the method `has_tool_call` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 59-65: Method `get_arguments`
```python
    def get_arguments(self, obj):
        """Extract arguments from object, supporting both 'parameters' and 'arguments' keys."""
        if "parameters" in obj:
            return obj.get("parameters")
        elif "arguments" in obj:
            return obj.get("arguments")
        return None
```
**EN:** This block defines the method `get_arguments` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `get_arguments`. Notable operations include `get`.
**CN:** 这一段定义了method `get_arguments`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `get_arguments`。 值得注意的操作包括 `get`。

### Lines 67-76: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.
        Supports multiple tool calls in the format:
        <|action_start|> <|plugin|>\n{JSON}<|action_end|>

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: StreamingParseResult with normal text and parsed tool calls.
        """
```
**EN:** This block defines the method `detect_and_parse` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 77-104: Method `detect_and_parse` logic (part 1)
```python

        # Find the first occurrence of tool call marker to extract normal text
        idx = text.find(self.bot_token)
        normal_text = text[:idx].strip() if idx != -1 else text

        if self.bot_token not in text:
            logger.warning("[InternLM Tool Call] No tool call markers found in text")
            return StreamingParseResult(normal_text=normal_text, calls=[])

        # Use regex to find all tool call blocks
        # Pattern matches: {self.bot_token}{...}{self.eot_token}
        tool_call_pattern = (
            rf"{re.escape(self.bot_token)}\s*(.*?){re.escape(self.eot_token)}"
        )
        matches = re.findall(tool_call_pattern, text, re.DOTALL)

        if not matches:
            logger.warning("[InternLM Tool Call] No complete tool call blocks found")
            return StreamingParseResult(normal_text=text, calls=[])

        logger.info(f"[InternLM Tool Call] Found {len(matches)} tool call(s)")

        calls = []
        tool_indices = self._get_tool_indices(tools)

        try:
            for idx, action_json in enumerate(matches):
                action_json = action_json.strip()
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `find`, `strip`, `warning`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`strip`、`warning`、`StreamingParseResult`。

### Lines 105-135: Method `detect_and_parse` logic (part 2)
```python

                try:
                    # Parse the JSON
                    action_dict = json.loads(action_json)
                    name = action_dict.get("name")
                    parameters = self.get_arguments(action_dict)

                    if not parameters:
                        parameters = {}

                    logger.info(
                        f"[InternLM Tool Call] Parsed tool call #{idx+1}: name={name}, "
                        f"parameters={json.dumps(parameters, ensure_ascii=False)}"
                    )

                    # Validate tool name
                    if not (name and name in tool_indices):
                        logger.warning(
                            f"[InternLM Tool Call] Model attempted to call undefined function: {name}, "
                            f"available_tools={list(tool_indices.keys())}"
                        )
                        if not envs.SGLANG_FORWARD_UNKNOWN_TOOLS.get():
                            continue  # Skip this tool call

                    # Create tool call item and add to list
                    tool_call = ToolCallItem(
                        tool_index=tool_indices[name],
                        name=name,
                        parameters=json.dumps(parameters, ensure_ascii=False),
                    )
                    calls.append(tool_call)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `loads`, `get`, `get_arguments`, `info`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`、`get`、`get_arguments`、`info`。

### Lines 136-153: Method `detect_and_parse` logic (part 3)
```python

                except json.JSONDecodeError as e:
                    logger.error(
                        f"[InternLM Tool Call] Failed to parse JSON for tool call #{idx+1}: {e}"
                    )
                    continue

            logger.info(
                f"[InternLM Tool Call] Successfully parsed {len(calls)} tool call(s), "
                f"normal_text_length={len(normal_text)}"
            )
            return StreamingParseResult(normal_text=normal_text, calls=calls)

        except Exception as e:
            logger.error(
                f"[InternLM Tool Call] Error in detect_and_parse: {e}", exc_info=True
            )
            return StreamingParseResult(normal_text=text, calls=[])
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `error`, `info`, `call`, `StreamingParseResult`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `error`、`info`、`call`、`StreamingParseResult`。

### Lines 155-162: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing for InternLM format.

        Supports a single tool call in streaming mode.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 163-186: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        # Check if we don't have a tool call start marker
        start = current_text.find(self.bot_token)
        if start == -1:
            # No tool call marker found
            # If we've already processed tool calls, don't return text again
            if self.current_tool_id > 0:
                self._buffer = ""
                return StreamingParseResult(normal_text="")

            # Check if buffer could be partial start of bot_token
            if not self._ends_with_partial_token(current_text, self.bot_token):
                # Not a partial match, return as normal text
                normal_text = current_text
                self._buffer = ""
                # Clean up any stray end tokens
                if self.eot_token in normal_text:
                    normal_text = normal_text.replace(self.eot_token, "")
                return StreamingParseResult(normal_text=normal_text)
            else:
                # Might be partial start token, keep buffering
                return StreamingParseResult()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `_ends_with_partial_token`, `replace`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`_ends_with_partial_token`、`replace`。

### Lines 187-206: Method `parse_streaming_increment` logic (part 2)
```python

        # Check if we have a complete tool call (with end marker)
        end = current_text.find(self.eot_token)
        if end != -1:
            # We have a complete tool call
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

            # Use detect_and_parse on the complete tool call
            complete_section = current_text[: end + len(self.eot_token)]
            result = self.detect_and_parse(complete_section, tools=tools)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `call`, `find`, `append`, `detect_and_parse`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `call`、`find`、`append`、`detect_and_parse`。

### Lines 207-224: Method `parse_streaming_increment` logic (part 3)
```python

            if result.calls:
                # Update the tool call index
                result.calls[0].tool_index = self.current_tool_id
                # Store the parsed tool call for reference
                self.prev_tool_call_arr[self.current_tool_id] = {
                    "name": result.calls[0].name,
                    "arguments": json.loads(result.calls[0].parameters),
                }
                self.streamed_args_for_tool[self.current_tool_id] = result.calls[
                    0
                ].parameters
                # Increment tool ID for next tool call
                self.current_tool_id += 1

            # Remove the completed tool call from buffer
            self._buffer = current_text[end + len(self.eot_token) :]
            return result
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `loads`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `loads`。

### Lines 225-231: Method `parse_streaming_increment` logic (part 4)
```python

        # We have bot_token but no eot_token yet - handle partial tool call streaming
        # Extract normal text before the tool call
        normal_text = current_text[:start]
        # Keep the tool call part in buffer
        self._buffer = current_text[start:]
        return StreamingParseResult(normal_text=normal_text)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding internlm style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的InternLM风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`。

### Lines 233-248: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        """
        Return structure information for constrained generation.

        For InternLM format, the structure is:
        - begin: <|action_start|> <|plugin|>\n
        - end: <|action_end|>
        - trigger: the begin token
        """
        return lambda name: StructureInfo(
            begin='<|action_start|> <|plugin|>\n{"name": "'
            + name
            + '", "parameters": ',
            end="}<|action_end|>",
            trigger="<|action_start|> <|plugin|>",
        )
```
**EN:** This block defines the method `structure_info` on `InternlmDetector`. It introduces the parameters, setup steps, and the main entry point for this piece of internlm style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `InternlmDetector`），介绍了参数、初始化步骤，以及这部分InternLM风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `StructureInfo`。

## Key Concepts / 关键概念
- `InternlmDetector`: Class that encapsulates internlm detector behavior in this module. / `InternlmDetector`：封装与“InternLM检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.environ`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
