# step3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/step3_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the step3 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 step3 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
```python
import contextlib
import json
from collections.abc import Sequence
from typing import Any

import regex as re

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
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.utils import random_uuid
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Step3ToolParser` (lines 34-307)
```python
class Step3ToolParser(ToolParser):
    """
    Tool parser for a model that uses a specific XML-like format for tool calls.
    This version uses a robust, stateful, cursor-based streaming parser and
    consolidates tool arguments into a single message.
    """
```
**EN:** Tool parser for a model that uses a specific XML-like format for tool calls.
**CN:** 定义 `Step3ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Step3ToolParser.__init__` (lines 48-53)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        self.position = 0
        # Explicit state flags for robust streaming
        self.tool_block_started = False
        self.tool_block_finished = False
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Step3ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Step3ToolParser.adjust_request` (lines 55-61)
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
**CN:** `Step3ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3ToolParser._parse_steptml_invoke` (lines 64-79)
```python
    def _parse_steptml_invoke(
        action_text: str,
    ) -> tuple[str | None, dict[str, str] | None]:
        func_name_match = re.search(r'<steptml:invoke name="([^"]+)">', action_text)
        if not func_name_match:
            return None, None
        func_name = func_name_match.group(1)

        params: dict[str, str] = {}
        param_matches = re.findall(
            r'<steptml:parameter name="([^"]+)">([^<]*)</steptml:parameter>',
            action_text,
        )
        for name, value in param_matches:
            params[name] = value.strip()
        return func_name, params
```
**EN:** Implements a focused helper routine for interpreting model output or locating parser-relevant structure.
**CN:** `Step3ToolParser._parse_steptml_invoke` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `Step3ToolParser._cast_arguments` (lines 81-113)
```python
    def _cast_arguments(
        self,
        func_name: str,
        params: dict[str, Any],
    ) -> dict[str, Any]:
        for tool in self.tools or []:
            if tool.function.name == func_name:
                schema = tool.function.parameters or {}
                properties = schema.get("properties", {})
                for key, value in params.items():
                    if not isinstance(value, str):
                        continue
                    prop = properties.get(key, {})
                    typ = prop.get("type")
                    if typ == "string":
                        params[key] = value.strip()
                    elif typ == "integer":
                        with contextlib.suppress(ValueError):
                            params[key] = int(value)
                    elif typ == "number":
                        with contextlib.suppress(ValueError):
                            params[key] = float(value)
                    elif typ == "boolean":
                        lower_val = value.lower()
                        params[key] = (
                            lower_val == "true"
                            if lower_val in ("true", "false")
                            else value
                        )
                    elif typ == "null":
                        params[key] = None if value.lower() == "null" else value
                break
        return params
```
**EN:** Provides the `_cast_arguments` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3ToolParser._cast_arguments` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3ToolParser.extract_tool_calls_streaming` (lines 115-253)
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
        # The main loop processes the stream from the last known position.
        while True:
            if self.position >= len(current_text):
                return None  # We've processed the entire stream.

            unprocessed_text = current_text[self.position :]

            # STATE: After all tools are done, all subsequent text is content.
            if self.tool_block_finished:
                self.position = len(current_text)
                return DeltaMessage(content=unprocessed_text)

            # STATE: Before the tool block has started.
            if not self.tool_block_started:
                if unprocessed_text.startswith(self.TOOL_CALLS_BEGIN):
                    self.position += len(self.TOOL_CALLS_BEGIN)
                    self.tool_block_started = True
                    continue  # Token consumed, re-loop.

                start_pos = unprocessed_text.find(self.TOOL_CALLS_BEGIN)
                if start_pos == -1:
                    if (
                        self.TOOL_CALLS_BEGIN.startswith(unprocessed_text.strip())
                        and unprocessed_text
                    ):
                        return None  # It's a prefix, wait.
                    self.position = len(current_text)
                    return DeltaMessage(content=unprocessed_text)
                else:
                    content = unprocessed_text[:start_pos]
                    self.position += len(content)
                    return DeltaMessage(content=content)

            # STATE: Inside the main tool block.
            offset = len(unprocessed_text) - len(unprocessed_text.lstrip())
            unprocessed_text = unprocessed_text.lstrip()
            self.position += offset

            if unprocessed_text.startswith(self.TOOL_CALLS_END):
                self.position += len(self.TOOL_CALLS_END)
# ... omitted for brevity / 为简洁省略 ...
                                    index=self.current_tool_id,
                                    function=DeltaFunctionCall(
                                        arguments=final_args_json
                                    ),
                                )
                            ]
                        )

                # If tool is not finished, return None to wait for more tokens.
                return None

            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Step3ToolParser.extract_tool_calls` (lines 255-307)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        if self.TOOL_CALLS_BEGIN not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        pre_text, rest = model_output.split(self.TOOL_CALLS_BEGIN, 1)
        if self.TOOL_CALLS_END not in rest:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        tool_block, post_text = rest.split(self.TOOL_CALLS_END, 1)
        content = (pre_text + post_text).strip()

        tool_calls: list[ToolCall] = []
        call_parts = tool_block.split(self.TOOL_CALL_BEGIN)

        for part in call_parts:
            if not part or self.TOOL_CALL_END not in part:
                continue

            call_content = part.split(self.TOOL_CALL_END, 1)[0]
            if self.TOOL_SEP not in call_content:
                continue

            type_part, invoke_part = call_content.split(self.TOOL_SEP, 1)
            if type_part.strip() != "function":
                continue

            function_name, params_dict = self._parse_steptml_invoke(invoke_part)

            if function_name and params_dict is not None:
                params_dict = self._cast_arguments(function_name, params_dict)
                params_str = json.dumps(params_dict, ensure_ascii=False)
                tool_calls.append(
                    ToolCall(
                        function=FunctionCall(name=function_name, arguments=params_str)
                    )
                )
        if tool_calls:
            return ExtractedToolCallInformation(
                tools_called=True,
                tool_calls=tool_calls,
                content=content if content else None,
            )
        return ExtractedToolCallInformation(
            tools_called=False, tool_calls=[], content=model_output
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `contextlib`, `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.utils`
