# gigachat3_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/function_call/gigachat3_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a gigachat 3 detector for structured tool/function calling. It recognizes model-specific output patterns and converts them into the normalized parser structures used by SGLang. / 该文件实现了面向 GigaChat3 的结构化工具/函数调用检测器。它识别模型特定的输出模式，并将结果转换为 SGLang 使用的统一解析结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports and module setup
```python
import json
import logging
import re
from typing import List

from sglang.srt.entrypoints.openai.protocol import Tool
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import (
    StreamingParseResult,
    ToolCallItem,
    _GetInfoFunc,
)

logger = logging.getLogger(__name__)

REGEX_FUNCTION_CALL = re.compile(
    r"(?:function call<\|role_sep\|>\n|<\|function_call\|>)(.*)",
    re.DOTALL,
)

REGEX_CONTENT_PATTERN = re.compile(
    r"^(.*?)(?:<\|message_sep\|>|<\|function_call\|>)",
    re.DOTALL,
)

NAME_REGEX = re.compile(
    r'"name"\s*:\s*"([^"]*)"',
    re.DOTALL,
)

ARGS_REGEX = re.compile(
    r'"arguments"\s*:\s*(.*)',
    re.DOTALL,
)


```
**EN:** This block gathers the imports and module-level setup for gigachat 3 style tool-call detection and normalization. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`, `compile`.
**CN:** 这一段汇集了与GigaChat3风格的工具调用检测与规范化相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`、`compile`。

### Lines 37-37: Class `GigaChat3Detector` declaration
```python
class GigaChat3Detector(BaseFormatDetector):
```
**EN:** This block declares the class `GigaChat3Detector` and establishes its responsibility inside gigachat 3 style tool-call detection and normalization. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `GigaChat3Detector`.
**CN:** 这一段声明了类 `GigaChat3Detector`，并说明它在GigaChat3风格的工具调用检测与规范化中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `GigaChat3Detector`。

### Lines 38-44: Method `__init__`
```python
    def __init__(self) -> None:
        super().__init__()
        self.tool_started: bool = False
        self.tool_name_sent: bool = False
        self.end_content: bool = False
        self._buffer: str = ""
        self.prev_tool_call_arr: list[dict] = []
```
**EN:** This block defines the method `__init__` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 46-48: Method `has_tool_call`
```python
    def has_tool_call(self, text: str) -> bool:
        """Check if text contains a tool call marker"""
        return "function call<|role_sep|>\n" in text or "<|function_call|>" in text
```
**EN:** This block defines the method `has_tool_call` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `has_tool_call`.
**CN:** 这一段定义了method `has_tool_call`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `has_tool_call`。

### Lines 50-58: Method `detect_and_parse` signature and setup
```python
    def detect_and_parse(
        self,
        text: str,
        tools: List[Tool],
    ) -> StreamingParseResult:
        """
        Non-streaming parsing of complete model output.
        Extracts tool calls and content from the full text.
        """
```
**EN:** This block defines the method `detect_and_parse` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `detect_and_parse`.
**CN:** 这一段定义了method `detect_and_parse`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `detect_and_parse`。

### Lines 59-82: Method `detect_and_parse` logic (part 1)
```python
        logger.debug(f"[GigaChat3] detect_and_parse: {text}")
        model_output = text
        function_call = None
        content = None
        if model_output.rstrip().endswith("</s>"):
            model_output = model_output[: model_output.rfind("</s>")]
        m_func = REGEX_FUNCTION_CALL.search(model_output)
        if m_func:
            try:
                function_call = json.loads(m_func.group(1), strict=False)
                if not (
                    isinstance(function_call, dict)
                    and "name" in function_call
                    and "arguments" in function_call
                ):
                    function_call = None
                elif not isinstance(function_call["arguments"], dict):
                    function_call = None
            except json.JSONDecodeError as e:
                logger.warning(f"[GigaChat3] JSON decode error: {e}")
                return StreamingParseResult(
                    normal_text=model_output,
                    calls=[],
                )
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gigachat 3 style tool-call detection and normalization workflow. Notable operations include `debug`, `rstrip`, `endswith`, `rfind`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的GigaChat3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `debug`、`rstrip`、`endswith`、`rfind`。

### Lines 83-94: Method `detect_and_parse` logic (part 2)
```python
        m_content = REGEX_CONTENT_PATTERN.search(model_output)
        if m_content:
            content = m_content.group(1)
        else:
            content = model_output
        if not function_call:
            return StreamingParseResult(normal_text=content, calls=[])
        name = function_call["name"]
        args = function_call["arguments"]
        match_result = {"name": name, "arguments": args}
        calls = self.parse_base_json(match_result, tools)
        return StreamingParseResult(normal_text=content, calls=calls)
```
**EN:** This block continues `detect_and_parse` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gigachat 3 style tool-call detection and normalization workflow. Notable operations include `search`, `group`, `StreamingParseResult`, `parse_base_json`.
**CN:** 这一段延续了 `detect_and_parse` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的GigaChat3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `search`、`group`、`StreamingParseResult`、`parse_base_json`。

### Lines 96-104: Method `parse_streaming_increment` signature and setup
```python
    def parse_streaming_increment(
        self,
        new_text: str,
        tools: List[Tool],
    ) -> StreamingParseResult:
        """
        Streaming parser for incremental text chunks.
        Maintains state across calls to build complete tool calls.
        """
```
**EN:** This block defines the method `parse_streaming_increment` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `parse_streaming_increment`.
**CN:** 这一段定义了method `parse_streaming_increment`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `parse_streaming_increment`。

### Lines 105-136: Method `parse_streaming_increment` logic (part 1)
```python
        if not new_text:
            return StreamingParseResult()
        logger.debug(f"[GigaChat3] parse_streaming_increment: '{new_text}'")
        self._buffer += new_text
        current_text = self._buffer
        delta_text = new_text
        content = None
        func_name = None
        cur_args = None
        m_func = REGEX_FUNCTION_CALL.search(current_text)
        if not self.tool_started:
            m_content = REGEX_CONTENT_PATTERN.search(delta_text)
            if m_content:
                content = m_content.group(1)
                self.end_content = True
            else:
                if not self.end_content:
                    content = delta_text
            if m_func:
                self.tool_started = True
                logger.debug("[GigaChat3] Tool call started")
            if content:
                return StreamingParseResult(normal_text=content)
        if not m_func:
            return StreamingParseResult()
        json_tail = m_func.group(1).strip()
        name_match = NAME_REGEX.search(json_tail)
        if name_match:
            func_name = name_match.group(1)
        args_match = ARGS_REGEX.search(json_tail)
        if args_match:
            cur_args = args_match.group(1).strip()
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gigachat 3 style tool-call detection and normalization workflow. Notable operations include `StreamingParseResult`, `debug`, `search`, `group`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的GigaChat3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `StreamingParseResult`、`debug`、`search`、`group`。

### Lines 137-168: Method `parse_streaming_increment` logic (part 2)
```python
            if cur_args.endswith("</s>"):
                cur_args = cur_args[: -len("</s>")]
            if cur_args.endswith("}"):
                try:
                    candidate = cur_args[:-1].strip()
                    json.loads(candidate, strict=False)
                    cur_args = candidate
                except json.JSONDecodeError:
                    pass
        calls: List[ToolCallItem] = []
        if not self.prev_tool_call_arr:
            self.prev_tool_call_arr.append({})
        if not self.tool_name_sent:
            if not func_name:
                return StreamingParseResult()
            self.tool_name_sent = True
            self.prev_tool_call_arr[0]["name"] = func_name
            logger.debug(f"[GigaChat3] Sending tool name: {func_name}")
            calls.append(
                ToolCallItem(
                    tool_index=0,
                    name=func_name,
                    parameters="",
                )
            )
            return StreamingParseResult(calls=calls)
        if cur_args is None:
            return StreamingParseResult()
        prev_args = self.prev_tool_call_arr[0].get("arguments_str", "")
        if not prev_args:
            delta_args = cur_args
        elif cur_args.startswith(prev_args):
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gigachat 3 style tool-call detection and normalization workflow. Notable operations include `endswith`, `strip`, `loads`, `append`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的GigaChat3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `endswith`、`strip`、`loads`、`append`。

### Lines 169-192: Method `parse_streaming_increment` logic (part 3)
```python
            delta_args = cur_args[len(prev_args) :]
        else:
            logger.warning(
                f"[GigaChat3] Arguments overlap mismatch. "
                f"prev='{prev_args[:50]}...' cur='{cur_args[:50]}...'"
            )
            return StreamingParseResult()
        if not delta_args:
            return StreamingParseResult()
        self.prev_tool_call_arr[0]["arguments_str"] = cur_args
        try:
            args_dict = json.loads(cur_args, strict=False)
            self.prev_tool_call_arr[0]["arguments"] = args_dict
        except json.JSONDecodeError:
            self.prev_tool_call_arr[0]["arguments"] = {}
        logger.debug(f"[GigaChat3] Sending args delta: '{delta_args[:100]}...'")
        calls.append(
            ToolCallItem(
                tool_index=0,
                name=None,
                parameters=delta_args,
            )
        )
        return StreamingParseResult(calls=calls)
```
**EN:** This block continues `parse_streaming_increment` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gigachat 3 style tool-call detection and normalization workflow. Notable operations include `warning`, `StreamingParseResult`, `loads`, `debug`.
**CN:** 这一段延续了 `parse_streaming_increment` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的GigaChat3风格的工具调用检测与规范化工作流。 值得注意的操作包括 `warning`、`StreamingParseResult`、`loads`、`debug`。

### Lines 194-196: Method `supports_structural_tag`
```python
    def supports_structural_tag(self) -> bool:
        """GigaChat3 does not use structural tags"""
        return False
```
**EN:** This block defines the method `supports_structural_tag` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `supports_structural_tag`.
**CN:** 这一段定义了method `supports_structural_tag`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `supports_structural_tag`。

### Lines 198-202: Method `structure_info`
```python
    def structure_info(self) -> _GetInfoFunc:
        """Not applicable for GigaChat3"""
        raise NotImplementedError(
            "GigaChat3Detector does not support structural_tag format."
        )
```
**EN:** This block defines the method `structure_info` on `GigaChat3Detector`. It introduces the parameters, setup steps, and the main entry point for this piece of gigachat 3 style tool-call detection and normalization. Definitions introduced here include `structure_info`. Notable operations include `NotImplementedError`.
**CN:** 这一段定义了method `structure_info`（属于 `GigaChat3Detector`），介绍了参数、初始化步骤，以及这部分GigaChat3风格的工具调用检测与规范化逻辑的主要入口。 此处引入的定义包括 `structure_info`。 值得注意的操作包括 `NotImplementedError`。

## Key Concepts / 关键概念
- `GigaChat3Detector`: Class that encapsulates giga chat 3 detector behavior in this module. / `GigaChat3Detector`：封装与“gigachat3检测器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `re`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`
