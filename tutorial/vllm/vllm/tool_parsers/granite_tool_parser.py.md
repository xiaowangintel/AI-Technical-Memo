# granite_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/granite_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the granite tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 granite 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-33)
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
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import (
    consume_space,
    find_common_prefix,
    is_complete_json,
    partial_json_loads,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 35-35)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `GraniteToolParser` (lines 38-256)
```python
class GraniteToolParser(ToolParser):
    """
    Tool call parser for the granite 3.0 models. Intended
    for use with the examples/tool_chat_template_granite.jinja
    template.

    Used when --enable-auto-tool-choice --tool-call-parser granite
    are all set
    """
```
**EN:** Tool call parser for the granite 3.0 models.
**CN:** 定义 `GraniteToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `GraniteToolParser.__init__` (lines 48-53)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        # for granite 3.0, the token `<|tool_call|>`
        self.bot_token = "<|tool_call|>"
        # for granite 3.1, the string `<tool_call>`
        self.bot_string = "<tool_call>"
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `GraniteToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `GraniteToolParser.extract_tool_calls` (lines 55-100)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        stripped = (
            model_output.strip()
            .removeprefix(self.bot_token)
            .removeprefix(self.bot_string)
            .lstrip()
        )
        if not stripped or stripped[0] != "[":
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
        try:
            raw_function_calls = json.loads(stripped)
            if not isinstance(raw_function_calls, list):
                raise Exception(
                    f"Expected dict or list, got {type(raw_function_calls)}"
                )

            logger.debug("Extracted %d tool calls", len(raw_function_calls))
            tool_calls = [
                ToolCall(
                    type="function",
                    function=FunctionCall(
                        name=function_call["name"],
                        # function call args are JSON but as a string
                        arguments=json.dumps(
                            function_call["arguments"], ensure_ascii=False
                        ),
                    ),
                )
                for function_call in raw_function_calls
            ]

            return ExtractedToolCallInformation(
                tools_called=True,
                tool_calls=tool_calls,
                content=None,
            )

        except Exception as e:
            logger.error("Error in extracting tool call from response %s", e)
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GraniteToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GraniteToolParser.extract_tool_calls_streaming` (lines 102-256)
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
        start_idx = consume_space(0, current_text)
        if current_text[start_idx:].startswith(self.bot_token):
            start_idx = consume_space(start_idx + len(self.bot_token), current_text)
        if current_text[start_idx:].startswith(self.bot_string):
            start_idx = consume_space(start_idx + len(self.bot_string), current_text)
        if (
            not current_text
            or start_idx >= len(current_text)
            or current_text[start_idx] != "["
        ):
            return DeltaMessage(content=delta_text)

        # bit mask flags for partial JSON parsing. If the name hasn't been
        # sent yet, don't allow sending
        # an incomplete string since OpenAI only ever (as far as I have
        # seen) allows sending the entire tool/ function name at once.
        flags = Allow.ALL if self.current_tool_name_sent else Allow.ALL & ~Allow.STR
        try:
            tool_call_arr = None
            is_complete = None
            try:
                tool_calls, end_idx = partial_json_loads(
                    current_text[start_idx:], flags
                )
                if type(tool_calls) is list:
                    tool_call_arr = tool_calls
                else:
                    return DeltaMessage(content=delta_text)

                is_complete = [True] * len(tool_calls)
                if not is_complete_json(current_text[start_idx : start_idx + end_idx]):
                    is_complete[-1] = False
            except partial_json_parser.core.exceptions.MalformedJSON:
                logger.debug("not enough tokens to parse into JSON yet")
                return None

            # case -- if no tokens have been streamed for the tool, e.g.
            #   only the array brackets, stream nothing
            if not tool_call_arr:
                return None
# ... omitted for brevity / 为简洁省略 ...
                            argument_diff
                        )

            self.prev_tool_call_arr = tool_call_arr
            return delta

        except Exception as e:
            logger.error("Error trying to handle streaming tool call: %s", e)
            logger.debug(
                "Skipping chunk as a result of tool streaming extraction error"
            )
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `GraniteToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Partial JSON recovery**: Partial JSON recovery. / 在 JSON 尚未闭合时尽量提取稳定的中间结果。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `partial_json_parser`, `partial_json_parser.core.options`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
