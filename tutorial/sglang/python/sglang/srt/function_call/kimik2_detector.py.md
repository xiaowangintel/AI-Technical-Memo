# kimik2_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/kimik2_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a kimik 2 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 kimik2 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup
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

_KIMI_K2_SPECIAL_TOKENS = [
    "<|tool_calls_section_begin|>",
    "<|tool_calls_section_end|>",
    "<|tool_call_begin|>",
    "<|tool_call_end|>",
    "<|tool_call_argument_begin|>",
]


```
**EN:** This block gathers the imports and module-level setup for kimik 2 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与kimik2风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 27-31: Function `_strip_special_tokens`
```python
def _strip_special_tokens(text: str) -> str:
    """Remove all Kimi-K2 tool-call special tokens from text."""
    for token in _KIMI_K2_SPECIAL_TOKENS:
        text = text.replace(token, "")
    return text
```
**EN:** This block defines the function `_strip_special_tokens`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `_strip_special_tokens`. Notable operations include `replace`.
**CN:** 这一段定义了function `_strip_special_tokens`，介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_strip_special_tokens`。 值得注意的操作包括 `replace`。

### Lines 34-34: Class `KimiK2Detector` declaration
```python
class KimiK2Detector(BaseFormatDetector):
```
**EN:** This block declares the class `KimiK2Detector` and establishes its responsibility inside kimik 2 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KimiK2Detector`.
**CN:** 这一段声明了类 `KimiK2Detector`，并说明它在kimik2风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KimiK2Detector`。

### Lines 35-52: Supporting state inside `KimiK2Detector`
```python
    """
    Detector for Kimi K2 / K2.5 model function call format.

    Format Structure (standard):
    ```
    <|tool_calls_section_begin|>
    <|tool_call_begin|>functions.{func_name}:{index}<|tool_call_argument_begin|>{json_args}<|tool_call_end|>
    <|tool_calls_section_end|>
    ```

    Format Structure (bare counter — model omits function name):
    ```
    <|tool_call_begin|>{counter}<|tool_call_argument_begin|>{json_args}<|tool_call_end|>
    ```

    Reference: https://huggingface.co/moonshotai/Kimi-K2-Instruct/blob/main/docs/tool_call_guidance.md
    """

```
**EN:** This block adds supporting state or helper logic inside `KimiK2Detector`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `Structure`.
**CN:** 这一段为 `KimiK2Detector` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `Structure`。

### Lines 53-55: Method `__init__` signature and setup
```python
    def __init__(self):
        super().__init__()

```
**EN:** This block defines the method `__init__` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 56-76: Method `__init__` logic (part 1)
```python
        self.bot_token: str = "<|tool_calls_section_begin|>"
        self.eot_token: str = "<|tool_calls_section_end|>"

        self.tool_call_start_token: str = "<|tool_call_begin|>"
        self.tool_call_end_token: str = "<|tool_call_end|>"
        self.tool_call_argument_begin_token: str = "<|tool_call_argument_begin|>"

        # Capture tool_call_id broadly: the model may emit standard IDs
        # like "functions.ReadFile:0" or bare call counters like "3".
        self.tool_call_regex = re.compile(
            r"<\|tool_call_begin\|>\s*(?P<tool_call_id>[^\s<|]+)\s*<\|tool_call_argument_begin\|>\s*(?P<function_arguments>\{.*?\})\s*<\|tool_call_end\|>",
            re.DOTALL,
        )

        self.stream_tool_call_portion_regex = re.compile(
            r"<\|tool_call_begin\|>\s*(?P<tool_call_id>[^\s<|]+)\s*<\|tool_call_argument_begin\|>\s*(?P<function_arguments>\{.*)",
            re.DOTALL,
        )

        self._last_arguments = ""
        self._current_stream_function_name: str | None = None
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `compile`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `compile`。

### Lines 77-83: Method `__init__` logic (part 2)
```python

        # Standard ID: "functions.search:0", "search:0"
        self.tool_call_id_regex = re.compile(
            r"^(?:functions\.)?(?P<name>[\w.\-]+):(?P<index>\d+)$"
        )
        # Bare call counter: "0", "3" (model uses auto-incrementing counter)
        self.tool_call_id_counter_regex = re.compile(r"^\d+$")
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `compile`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `compile`。

### Lines 85-96: Method `_parse_tool_call_id` signature and setup
```python
    def _parse_tool_call_id(
        self, function_id: str, tools: List[Tool], function_args: str = None
    ):
        """Parse a tool call ID into (function_name, call_index).

        Standard format: "functions.ReadFile:0" → ("ReadFile", 0)
        Bare counter:    "3" → call_index=3, infer name from arguments.

        The bare counter is a conversation-level auto-increment, NOT an index
        into the tools list. The function name is inferred by matching argument
        keys against tool parameter schemas.
        """
```
**EN:** This block defines the method `_parse_tool_call_id` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `_parse_tool_call_id`. Notable operations include `into`.
**CN:** 这一段定义了method `_parse_tool_call_id`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_parse_tool_call_id`。 值得注意的操作包括 `into`。

### Lines 97-109: Method `_parse_tool_call_id` logic (part 1)
```python
        m = self.tool_call_id_regex.match(function_id)
        if m:
            return m.group("name"), int(m.group("index"))

        if self.tool_call_id_counter_regex.match(function_id):
            call_index = int(function_id)
            name = self._infer_tool_name(tools, function_args)
            if name:
                return name, call_index
            return None, call_index

        logger.warning("Unexpected tool_call_id format: %s", function_id)
        return None, 0
```
**EN:** This block continues `_parse_tool_call_id` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `match`, `group`, `_infer_tool_name`, `warning`.
**CN:** 这一段延续了 `_parse_tool_call_id` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `match`、`group`、`_infer_tool_name`、`warning`。

### Lines 111-116: Method `_infer_tool_name` signature and setup
```python
    def _infer_tool_name(self, tools: List[Tool], function_args: str = None):
        """Infer function name when the model omits it (bare counter ID).

        Matches argument keys against tool parameter schemas, preferring the
        tool whose declared properties best match the actual arguments.
        """
```
**EN:** This block defines the method `_infer_tool_name` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `_infer_tool_name`. Notable operations include `it`.
**CN:** 这一段定义了method `_infer_tool_name`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `_infer_tool_name`。 值得注意的操作包括 `it`。

### Lines 117-135: Method `_infer_tool_name` logic (part 1)
```python
        if not tools:
            return None
        if len(tools) == 1:
            return tools[0].function.name

        if not function_args:
            logger.debug(
                "No function_args for tool name inference with %d tools", len(tools)
            )
            return None

        try:
            arg_keys = set(json.loads(function_args).keys())
        except (json.JSONDecodeError, TypeError):
            logger.debug(
                "Could not parse function_args for tool name inference "
                "(may be partial JSON in streaming)"
            )
            return None
```
**EN:** This block continues `_infer_tool_name` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `debug`, `loads`, `keys`.
**CN:** 这一段延续了 `_infer_tool_name` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `debug`、`loads`、`keys`。

### Lines 136-152: Method `_infer_tool_name` logic (part 2)
```python

        # Pick the tool whose properties best match the argument keys.
        best_name = None
        best_score = -1
        for tool in tools:
            params = tool.function.parameters or {}
            props = set(params.get("properties", {}).keys())
            if not props:
                continue
            overlap = len(arg_keys & props)
            extra = len(arg_keys - props)
            score = overlap - extra
            if score > best_score:
                best_score = score
                best_name = tool.function.name

        return best_name
```
**EN:** This block continues `_infer_tool_name` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `get`, `keys`.
**CN:** 这一段延续了 `_infer_tool_name` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `get`、`keys`。

### Lines 154-156: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if the text contains a KimiK2 format tool call."""
        return self.bot_token in text
```
**EN:** This block defines the method `has_tool_call` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 158-165: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(self, text: str, tools: List[Tool]) -> StreamingParseResult:
        """
        One-time parsing: Detects and parses tool calls in the provided text.

        :param text: The complete text to parse.
        :param tools: List of available tools.
        :return: StreamingParseResult with normal_text (content before tool calls) and calls (parsed items).
        """
```
**EN:** This block defines the method `detect_and_parse` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`. Notable operations include `normal_text`, `calls`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。 值得注意的操作包括 `normal_text`、`calls`。

### Lines 166-190: Method `detect_and_parse` logic (part 1)
```python
        if self.bot_token not in text:
            return StreamingParseResult(normal_text=text, calls=[])
        try:
            function_call_tuples = self.tool_call_regex.findall(text)

            logger.debug("function_call_tuples: %s", function_call_tuples)

            tool_calls = []
            for match in function_call_tuples:
                function_id, function_args = match
                function_name, function_idx = self._parse_tool_call_id(
                    function_id, tools, function_args
                )
                if function_name is None:
                    continue

                logger.debug(f"function_name {function_name}")

                tool_calls.append(
                    ToolCallItem(
                        tool_index=function_idx,
                        name=function_name,
                        parameters=function_args,
                    )
                )
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `findall`, `debug`, `_parse_tool_call_id`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`findall`、`debug`、`_parse_tool_call_id`。

### Lines 191-197: Method `detect_and_parse` logic (part 2)
```python

            content = text[: text.find(self.bot_token)]
            return StreamingParseResult(normal_text=content, calls=tool_calls)

        except Exception as e:
            logger.error("Error in detect_and_parse: %s", e, exc_info=True)
            return StreamingParseResult(normal_text=text)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `find`, `StreamingParseResult`, `error`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `find`、`StreamingParseResult`、`error`。

### Lines 199-204: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self, new_text: str, tools: List[Tool]
    ) -> StreamingParseResult:
        """
        Streaming incremental parsing tool calls for KimiK2 format.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 205-237: Method `parse_streaming_increment` logic (part 1)
```python
        self._buffer += new_text
        current_text = self._buffer

        # Check if we have a tool call (either the start token or individual tool call)
        has_tool_call = (
            self.bot_token in current_text or self.tool_call_start_token in current_text
        )

        if not has_tool_call:
            self._buffer = ""
            normal_text = _strip_special_tokens(new_text)
            return StreamingParseResult(normal_text=normal_text)

        if not hasattr(self, "_tool_indices"):
            self._tool_indices = self._get_tool_indices(tools)

        calls: list[ToolCallItem] = []
        try:
            match = self.stream_tool_call_portion_regex.search(current_text)
            if match:
                function_id = match.group("tool_call_id")
                function_args = match.group("function_arguments")

                # Reuse cached name for current tool call to avoid repeated
                # json.loads on partial JSON in _infer_tool_name.
                if self._current_stream_function_name is not None:
                    function_name = self._current_stream_function_name
                else:
                    function_name, _ = self._parse_tool_call_id(
                        function_id, tools, function_args
                    )
                if function_name is None:
                    return StreamingParseResult(normal_text="", calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `call`, `_strip_special_tokens`, `StreamingParseResult`, `_get_tool_indices`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `call`、`_strip_special_tokens`、`StreamingParseResult`、`_get_tool_indices`。

### Lines 238-270: Method `parse_streaming_increment` logic (part 2)
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
                            name=function_name,
                            parameters="",
                        )
                    )
                    self.current_tool_name_sent = True
                    self._current_stream_function_name = function_name
                    self.prev_tool_call_arr[self.current_tool_id] = {
                        "name": function_name,
                        "arguments": {},
                    }
                else:
                    argument_diff = (
                        function_args[len(self._last_arguments) :]
                        if function_args.startswith(self._last_arguments)
                        else function_args
                    )
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `append`, `ToolCallItem`, `startswith`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `append`、`ToolCallItem`、`startswith`。

### Lines 271-297: Method `parse_streaming_increment` logic (part 3)
```python

                    parsed_args_diff = argument_diff.split(self.tool_call_end_token, 1)[
                        0
                    ]

                    if parsed_args_diff:
                        calls.append(
                            ToolCallItem(
                                tool_index=self.current_tool_id,
                                name=None,
                                parameters=parsed_args_diff,
                            )
                        )
                        self._last_arguments += parsed_args_diff
                        self.streamed_args_for_tool[
                            self.current_tool_id
                        ] += parsed_args_diff

                    parsed_args = function_args.split(self.tool_call_end_token, 1)[0]
                    if _is_complete_json(parsed_args):
                        try:
                            parsed_args = json.loads(parsed_args)
                            self.prev_tool_call_arr[self.current_tool_id][
                                "arguments"
                            ] = parsed_args
                        except json.JSONDecodeError:
                            pass
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `split`, `append`, `ToolCallItem`, `_is_complete_json`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `split`、`append`、`ToolCallItem`、`_is_complete_json`。

### Lines 298-322: Method `parse_streaming_increment` logic (part 4)
```python

                        # Find the end of the current tool call and remove only that part from buffer
                        tool_call_end_pattern = (
                            r"<\|tool_call_begin\|>.*?<\|tool_call_end\|>"
                        )
                        end_match = re.search(
                            tool_call_end_pattern, current_text, re.DOTALL
                        )
                        if end_match:
                            self._buffer = current_text[end_match.end() :]
                        else:
                            self._buffer = ""

                        result = StreamingParseResult(normal_text="", calls=calls)
                        self.current_tool_id += 1
                        self._last_arguments = ""
                        self.current_tool_name_sent = False
                        self._current_stream_function_name = None
                        return result

            return StreamingParseResult(normal_text="", calls=calls)

        except Exception as e:
            logger.error("Error in parse_streaming_increment: %s", e, exc_info=True)
            return StreamingParseResult(normal_text=_strip_special_tokens(current_text))
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding kimik 2 style tool-call detection and normalization workflow. Notable operations include `search`, `end`, `StreamingParseResult`, `error`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的kimik2风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`end`、`StreamingParseResult`、`error`。

### Lines 324-334: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        """Return function that creates StructureInfo for guided generation."""

        def get_info(name: str) -> StructureInfo:
            return StructureInfo(
                begin=f"<|tool_calls_section_begin|><|tool_call_begin|>functions.{name}:0<|tool_call_argument_begin|>",
                end="<|tool_call_end|><|tool_calls_section_end|>",
                trigger="<|tool_calls_section_begin|>",
            )

        return get_info
```
**EN:** This block defines the method `structure_info` on `KimiK2Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of kimik 2 style tool-call detection and normalization. Definitions introduced here include `structure_info`, `get_info`. Notable operations include `StructureInfo`.
**CN:** 这一段定义了method `structure_info`（属于 `KimiK2Detector`），介绍了参数、初始化步骤，以及这部分kimik2风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`、`get_info`。 值得注意的操作包括 `StructureInfo`。

### Lines 335-344: Module-level constants and helper logic
```python

    # Kimi stays on the SGLang legacy structural tag path. xgrammar 0.2.0's
    # get_kimi_structural_tag(tool_choice="auto") emits a bare
    # <|tool_call_begin|>...<|tool_call_end|> grammar without the
    # <|tool_calls_section_begin|>/<|tool_calls_section_end|> wrapper Kimi's
    # chat template uses, and KimiK2Detector.has_tool_call() keys off the
    # section marker — bare tool calls would be silently dropped. Inheriting
    # the base get_structural_tag_name (returns None) keeps FunctionCallParser
    # on the legacy path, whose structure_info bakes the section markers in.
    # TODO: re-enable the builtin once https://github.com/mlc-ai/xgrammar/issues/622 is fixed.
```
**EN:** This block contains module-level constants, helpers, or documentation for kimik 2 style tool-call detection and normalization. It prepares shared state that later classes and functions build on. Notable operations include `get_kimi_structural_tag`, `has_tool_call`, `get_structural_tag_name`.
**CN:** 这一段包含与kimik2风格的工具调用检测与规范化相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `get_kimi_structural_tag`、`has_tool_call`、`get_structural_tag_name`。

## Key Concepts / 关键概念
- `_strip_special_tokens`: Function that performs strip special tokens for the surrounding workflow. / `_strip_special_tokens`：在周边工作流中执行“stripspecialtokens”相关任务的函数。
- `KimiK2Detector`: Class that encapsulates kimi k 2 detector behavior in this module. / `KimiK2Detector`：封装与“Kimik2检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.utils`
