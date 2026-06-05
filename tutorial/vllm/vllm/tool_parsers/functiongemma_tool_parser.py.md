# functiongemma_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/functiongemma_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the functiongemma tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 functiongemma 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-24)
```python
import json
from collections.abc import Sequence

import regex as re

from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
    ExtractedToolCallInformation,
    FunctionCall,
    ToolCall,
)
from vllm.entrypoints.openai.responses.protocol import ResponsesRequest
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import Tool, ToolParser
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 26-26)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `FunctionGemmaToolParser` (lines 29-314)
```python
class FunctionGemmaToolParser(ToolParser):
    """
    Tool parser for Google's FunctionGemma model (google/functiongemma-270m-it).

    Handles the FunctionGemma function call format:
    <start_function_call>call:func_name{param:<escape>value<escape>}<end_function_call>
    """
```
**EN:** Tool parser for Google's FunctionGemma model (google/functiongemma-270m-it).
**CN:** 定义 `FunctionGemmaToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `FunctionGemmaToolParser.__init__` (lines 52-60)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # Streaming state
        self.current_tool_name_sent: bool = False
        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id: int = -1
        self.streamed_args_for_tool: list[str] = []
        self.buffered_delta_text = ""
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `FunctionGemmaToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `FunctionGemmaToolParser._parse_arguments` (lines 62-76)
```python
    def _parse_arguments(self, args_str: str) -> dict:
        """Parse FunctionGemma argument string into a dictionary."""
        arguments: dict = {}
        if not args_str:
            return arguments

        matches = self.arg_regex.findall(args_str)
        for key, value in matches:
            try:
                parsed_value = json.loads(value)
                arguments[key] = parsed_value
            except json.JSONDecodeError:
                arguments[key] = value

        return arguments
```
**EN:** Parse FunctionGemma argument string into a dictionary.
**CN:** `FunctionGemmaToolParser._parse_arguments` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `FunctionGemmaToolParser.adjust_request` (lines 78-84)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `FunctionGemmaToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `FunctionGemmaToolParser.extract_tool_calls` (lines 86-145)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        if self.tool_call_start_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        try:
            matches = self.tool_call_regex.findall(model_output)

            if not matches:
                return ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=model_output
                )

            tool_calls: list[ToolCall] = []

            for match in matches:
                func_name = match[0] if match[0] else match[2]
                args_str = match[1] if match[1] else match[3]

                if not func_name:
                    continue

                arguments = self._parse_arguments(args_str)

                tool_calls.append(
                    ToolCall(
                        type="function",
                        function=FunctionCall(
                            name=func_name,
                            arguments=json.dumps(arguments, ensure_ascii=False),
                        ),
                    )
                )

            if tool_calls:
                content_end = model_output.find(self.tool_call_start_token)
                content = (
                    model_output[:content_end].strip() if content_end > 0 else None
                )

                return ExtractedToolCallInformation(
                    tools_called=True,
                    tool_calls=tool_calls,
                    content=content if content else None,
                )

            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        except Exception:
            logger.exception("Error extracting tool calls from FunctionGemma response")
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `FunctionGemmaToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `FunctionGemmaToolParser._buffer_delta_text` (lines 147-165)
```python
    def _buffer_delta_text(self, delta_text: str) -> str:
        """Buffer incoming delta text to handle multi-token special sequences."""
        potential_start = "<start_function_call>"
        potential_end = "<end_function_call>"

        combined = self.buffered_delta_text + delta_text

        if combined.endswith(potential_start) or combined.endswith(potential_end):
            self.buffered_delta_text = ""
            return combined

        for tag in [potential_start, potential_end]:
            for i in range(1, len(tag)):
                if combined.endswith(tag[:i]):
                    self.buffered_delta_text = combined[-(i):]
                    return combined[:-i]

        self.buffered_delta_text = ""
        return combined
```
**EN:** Buffer incoming delta text to handle multi-token special sequences.
**CN:** `FunctionGemmaToolParser._buffer_delta_text` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `FunctionGemmaToolParser.extract_tool_calls_streaming` (lines 167-314)
```python
    def extract_tool_calls_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
        request: ChatCompletionRequest,
    ) -> DeltaMessage | None:
        delta_text = self._buffer_delta_text(delta_text)
        current_text = previous_text + delta_text

        if self.tool_call_start_token not in current_text:
            if delta_text:
                return DeltaMessage(content=delta_text)
            return None

        try:
            start_count = current_text.count(self.tool_call_start_token)
            end_count = current_text.count(self.tool_call_end_token)
            prev_start_count = previous_text.count(self.tool_call_start_token)
            prev_end_count = previous_text.count(self.tool_call_end_token)

            if self.tool_call_start_token not in current_text:
                return DeltaMessage(content=delta_text)

            # Starting a new function call
            if start_count > prev_start_count and start_count > end_count:
                self.current_tool_id += 1
                self.current_tool_name_sent = False
                self.streamed_args_for_tool.append("")
                self.prev_tool_call_arr.append({})
                logger.debug("Starting new tool call %d", self.current_tool_id)
                return None

            # In the middle of a function call
            if start_count > end_count:
                last_start = current_text.rfind(self.tool_call_start_token)
                partial_call = current_text[
                    last_start + len(self.tool_call_start_token) :
                ]

                if partial_call.startswith("call:"):
                    func_part = partial_call[5:]

                    if "{" in func_part:
                        func_name = func_part.split("{")[0]
                        args_part = (
                            func_part.split("{", 1)[1] if "{" in func_part else ""
# ... omitted for brevity / 为简洁省略 ...
                                    )
                                ]
                            )
                return None

            if delta_text:
                return DeltaMessage(content=delta_text)
            return None

        except Exception:
            logger.exception("Error in streaming tool call extraction")
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `FunctionGemmaToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
