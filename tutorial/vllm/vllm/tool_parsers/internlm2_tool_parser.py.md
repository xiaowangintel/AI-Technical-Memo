# internlm2_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/internlm2_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the internlm2 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 internlm2 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
```python
import json
from collections.abc import Sequence

import partial_json_parser
from partial_json_parser.core.options import Allow

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
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import extract_intermediate_diff
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Internlm2ToolParser` (lines 34-233)
```python
class Internlm2ToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        self.position = 0
```
**EN:** Declares `Internlm2ToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Internlm2ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Internlm2ToolParser.__init__` (lines 35-37)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        self.position = 0
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Internlm2ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Internlm2ToolParser.adjust_request` (lines 39-48)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # do not skip special tokens because internlm use the special
            # tokens to indicate the start and end of the tool calls
            # information.
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Internlm2ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Internlm2ToolParser.get_arguments` (lines 50-55)
```python
    def get_arguments(self, obj):
        if "parameters" in obj:
            return obj.get("parameters")
        elif "arguments" in obj:
            return obj.get("arguments")
        return None
```
**EN:** Builds or returns derived metadata used by the parser and by higher-level serving code.
**CN:** `Internlm2ToolParser.get_arguments` 方法构建或返回解析器与上层服务需要的派生元数据。

### Method `Internlm2ToolParser.extract_tool_calls_streaming` (lines 57-195)
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
        if "<|action_start|>" not in current_text:
            self.position = len(current_text)
            return DeltaMessage(content=delta_text)
        # if the tool call is sent, return an empty delta message
        # to make sure the finish_reason will be sent correctly.
        if self.current_tool_id > 0:
            return DeltaMessage(content="")

        last_pos = self.position
        if "<|action_start|><|plugin|>" not in current_text[last_pos:]:
            return None

        new_delta = current_text[last_pos:]
        text, action = new_delta.split("<|action_start|><|plugin|>")

        if len(text) > 0:
            self.position = self.position + len(text)
            return DeltaMessage(content=text)

        action = action.strip()
        action = action.split("<|action_end|>".strip())[0]

        # bit mask flags for partial JSON parsing. If the name hasn't been
        # sent yet, don't allow sending
        # an incomplete string since OpenAI only ever (as far as I have
        # seen) allows sending the entire tool/ function name at once.
        flags = Allow.ALL if self.current_tool_name_sent else Allow.ALL & ~Allow.STR

        try:
            parsable_arr = action

            # tool calls are generated in an object in internlm2
            # it's not support parallel tool calls
            try:
                tool_call_arr: dict = partial_json_parser.loads(parsable_arr, flags)
            except partial_json_parser.core.exceptions.MalformedJSON:
                logger.debug("not enough tokens to parse into JSON yet")
                return None

            # if the current tool name hasn't been sent, send if available
# ... omitted for brevity / 为简洁省略 ...
            # check to see if the name is defined and has been sent. if so,
            # stream the name - otherwise keep waiting
            # finish by setting old and returning None as base case
            tool_call_arr["arguments"] = self.get_arguments(tool_call_arr)
            self.prev_tool_call_arr = [tool_call_arr]
            return delta
        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            logger.debug(
                "Skipping chunk as a result of tool streaming extraction error"
            )
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Internlm2ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Internlm2ToolParser.extract_tool_calls` (lines 197-233)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        text = model_output
        tools = self.tools
        if "<|action_start|><|plugin|>" in text:
            text, action = text.split("<|action_start|><|plugin|>")
            action = action.split("<|action_end|>".strip())[0]
            action = action[action.find("{") :]
            action_dict = json.loads(action)
            name, parameters = (
                action_dict["name"],
                json.dumps(
                    action_dict.get("parameters", action_dict.get("arguments", {})),
                    ensure_ascii=False,
                ),
            )

            if not tools or name not in [t.function.name for t in tools]:
                ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=text
                )

            tool_calls = [
                ToolCall(function=FunctionCall(name=name, arguments=parameters))
            ]
            return ExtractedToolCallInformation(
                tools_called=True,
                tool_calls=tool_calls,
                content=text if len(text) > 0 else None,
            )

        return ExtractedToolCallInformation(
            tools_called=False, tool_calls=[], content=text
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Internlm2ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Partial JSON recovery**: Partial JSON recovery. / 在 JSON 尚未闭合时尽量提取稳定的中间结果。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `partial_json_parser`, `partial_json_parser.core.options`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
