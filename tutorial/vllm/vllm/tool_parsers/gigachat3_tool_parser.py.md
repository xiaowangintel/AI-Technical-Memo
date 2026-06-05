# gigachat3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/gigachat3_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the gigachat3 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 gigachat3 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

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

### Module constants/configuration 1 (lines 26-46)
```python
logger = init_logger(__name__)

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
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `GigaChat3ToolParser` (lines 49-214)
```python
class GigaChat3ToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        self.tool_started: bool = False
        self.tool_name_sent: bool = False
        self.tool_id: str | None = None
        self.prev_tool_call_arr: list[dict] = []
        self.end_content: bool = False
        self.streamed_args_for_tool: list[str] = []
```
**EN:** Declares `GigaChat3ToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `GigaChat3ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `GigaChat3ToolParser.__init__` (lines 50-57)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        self.tool_started: bool = False
        self.tool_name_sent: bool = False
        self.tool_id: str | None = None
        self.prev_tool_call_arr: list[dict] = []
        self.end_content: bool = False
        self.streamed_args_for_tool: list[str] = []
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `GigaChat3ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `GigaChat3ToolParser.adjust_request` (lines 59-65)
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
**CN:** `GigaChat3ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GigaChat3ToolParser.extract_tool_calls` (lines 67-119)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        function_call = None
        content = None
        if model_output.rstrip().endswith("</s>"):
            model_output = model_output[: model_output.rfind("</s>")]
        m_func = REGEX_FUNCTION_CALL.search(model_output)
        if m_func:
            try:
                function_call = json.loads(m_func.group(1), strict=False)
                if (
                    isinstance(function_call, dict)
                    and "name" in function_call
                    and "arguments" in function_call
                ):
                    if not isinstance(function_call["arguments"], dict):
                        function_call = None
                else:
                    function_call = None
            except json.JSONDecodeError:
                return ExtractedToolCallInformation(
                    tools_called=False,
                    tool_calls=[],
                    content=model_output,
                )
        m_content = REGEX_CONTENT_PATTERN.search(model_output)
        content = m_content.group(1) if m_content else model_output
        if not function_call:
            return ExtractedToolCallInformation(
                tools_called=False,
                tool_calls=[],
                content=content if content else None,
            )
        name = function_call["name"]
        args = function_call["arguments"]
        if not isinstance(args, str):
            args = json.dumps(function_call["arguments"], ensure_ascii=False)
        return ExtractedToolCallInformation(
            tools_called=True,
            tool_calls=[
                ToolCall(
                    type="function",
                    function=FunctionCall(
                        name=name,
                        arguments=args,
                    ),
                )
            ],
            content=content if content else None,
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GigaChat3ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GigaChat3ToolParser.extract_tool_calls_streaming` (lines 121-214)
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
            if content:
                return DeltaMessage(content=content)
        if not m_func:
            return None
        json_tail = m_func.group(1).strip()
        name_match = NAME_REGEX.search(json_tail)
        if name_match:
            func_name = name_match.group(1)
        args_match = ARGS_REGEX.search(json_tail)
        if args_match:
            cur_args = args_match.group(1).strip()
            if cur_args.endswith("</s>"):
                cur_args = cur_args[: -len("</s>")]
            if cur_args.endswith("}"):  # last '}' end of json
                try:
                    candidate = cur_args[:-1].strip()
                    json.loads(candidate, strict=False)
                    cur_args = candidate
                except json.JSONDecodeError:
                    pass
        if not self.prev_tool_call_arr:
            self.prev_tool_call_arr.append({})
        if not self.tool_name_sent:
            if not func_name:
                return None
            self.tool_name_sent = True
# ... omitted for brevity / 为简洁省略 ...
            self.streamed_args_for_tool.append("")
        self.streamed_args_for_tool[0] = cur_args
        return DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=0,
                    function=DeltaFunctionCall(
                        arguments=delta_args,
                    ).model_dump(exclude_none=True),
                )
            ],
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GigaChat3ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
