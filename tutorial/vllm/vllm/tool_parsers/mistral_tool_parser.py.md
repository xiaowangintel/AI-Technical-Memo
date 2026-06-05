# mistral_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/mistral_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the mistral tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 mistral 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-51)
```python
from __future__ import annotations

import json
from collections.abc import Sequence
from dataclasses import dataclass
from enum import Enum, auto
from random import choices
from string import ascii_letters, digits
from typing import TYPE_CHECKING, Any

import ijson
import regex as re
from mistral_common.protocol.instruct.tool_calls import (
    NamedToolChoice as MistralNamedToolChoice,
)
from mistral_common.protocol.instruct.tool_calls import (
    Tool as MistralTool,
)
from mistral_common.protocol.instruct.tool_calls import (
    ToolChoice as MistralToolChoice,
)
from mistral_common.protocol.instruct.tool_calls import (
    ToolChoiceEnum as MistralToolChoiceEnum,
)
from pydantic import Field

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
from vllm.reasoning.mistral_reasoning_parser import MistralReasoningParser
from vllm.sampling_params import StructuredOutputsParams
from vllm.tokenizers import TokenizerLike
from vllm.tokenizers.mistral import MistralTokenizer
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.utils.mistral import is_mistral_tokenizer
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 56-60)
```python
logger = init_logger(__name__)

ALPHANUMERIC = ascii_letters + digits

_DEFAULT_JSON_SCHEMA = {"anyOf": [{"type": "object"}, {"type": "array"}]}
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `StreamingState` (lines 63-76)
```python
class StreamingState(Enum):
    """Enum for tracking the current streaming parsing state."""
```
**EN:** Enum for tracking the current streaming parsing state.
**CN:** 定义 `StreamingState` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Class `MistralToolCall` (lines 79-90)
```python
class MistralToolCall(ToolCall):
    id: str = Field(default_factory=lambda: MistralToolCall.generate_random_id())
```
**EN:** Declares `MistralToolCall`, the main parser class in this module. It extends ToolCall. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `MistralToolCall` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `MistralToolCall.generate_random_id` (lines 83-86)
```python
    def generate_random_id():
        # Mistral Tool Call Ids must be alphanumeric with a length of 9.
        # https://github.com/mistralai/mistral-common/blob/21ee9f6cee3441e9bb1e6ed2d10173f90bd9b94b/src/mistral_common/protocol/instruct/validator.py#L299
        return "".join(choices(ALPHANUMERIC, k=9))
```
**EN:** Provides the `generate_random_id` helper used by this module to keep parsing logic modular and reusable.
**CN:** `MistralToolCall.generate_random_id` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `MistralToolCall.is_valid_id` (lines 89-90)
```python
    def is_valid_id(id: str) -> bool:
        return id.isalnum() and len(id) == 9
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `MistralToolCall.is_valid_id` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Function `_is_pre_v11_tokeniser` (lines 93-99)
```python
def _is_pre_v11_tokeniser(model_tokenizer: TokenizerLike) -> bool:
    if is_mistral_tokenizer(model_tokenizer):
        return model_tokenizer.version < 11
    # For HF tokenizers, check if [ARGS] token exists in vocab
    # which indicates a v11+ equivalent tokenizer
    vocab: dict[str, int] = getattr(model_tokenizer, "get_vocab", lambda: {})()
    return "[ARGS]" not in vocab
```
**EN:** Provides the `_is_pre_v11_tokeniser` helper used by this module to keep parsing logic modular and reusable.
**CN:** `_is_pre_v11_tokeniser` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `MistralStreamingResult` (lines 103-112)
```python
class MistralStreamingResult:
    r"""Encapsulates the mutable state returned from
    `MistralToolParser.extract_maybe_reasoning_and_tool_streaming`.
    """
```
**EN:** Encapsulates the mutable state returned from `MistralToolParser.extract_maybe_reasoning_and_tool_streaming`.
**CN:** 定义 `MistralStreamingResult` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Class `MistralToolParser` (lines 115-937)
```python
class MistralToolParser(ToolParser):
    r"""Tool call parser for Mistral models, intended for use with either:

    - `mistral_common <https://github.com/mistralai/mistral-common/>`_
      (recommended)
    - the `examples/tool_chat_template_mistral.jinja` template.

    Used when `--enable-auto-tool-choice --tool-call-parser mistral` are all
    set.
    """
```
**EN:** Tool call parser for Mistral models, intended for use with either: - `mistral_common <https://github.com/mistralai/mistral-common/>`_ (recommended) - the `examples/tool_chat_template_mistral.jinja` template.
**CN:** 定义 `MistralToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `MistralToolParser.__init__` (lines 131-161)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        if not is_mistral_tokenizer(self.model_tokenizer):
            logger.info("Non-Mistral tokenizer detected when using a Mistral model...")

        # initialize properties used for state when parsing tool calls in
        # streaming mode
        self.prev_tool_call_arr: list[dict[str, Any]] = []
        self.current_tool_id: int = -1
        self.streaming_state: StreamingState = StreamingState.WAITING_FOR_TOOL_START

        # For streaming pre v11 tokenizer tool calls
        self.current_tool_name: str | None = None
        self.current_tool_mistral_id: str | None = None
        self.starting_new_tool = False
        self._is_pre_v11 = _is_pre_v11_tokeniser(self.model_tokenizer)
        if self._is_pre_v11:
            self.parse_coro = ijson.parse_coro(
                self.update_stream_state_pre_v11_tokenizer()
            )

        self.bot_token = "[TOOL_CALLS]"
        self.bot_token_id = self.vocab.get(self.bot_token)
        self.tool_call_regex = re.compile(r"\[{.*}\]", re.DOTALL)

        if self.bot_token_id is None:
            raise RuntimeError(
                "Mistral Tool Parser could not locate the tool call token in "
                "the tokenizer!"
            )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `MistralToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `MistralToolParser.adjust_request` (lines 163-282)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        so_non_supported_attributes = [
            "regex",
            "choice",
            "grammar",
            # whitespace_pattern is not a constraint type but an option;
            # Mistral grammar factory does not support it.
            "whitespace_pattern",
            "structural_tag",
        ]
        any_so_non_supported_active = request.structured_outputs is not None and any(
            getattr(request.structured_outputs, attribute) is not None
            for attribute in so_non_supported_attributes
        )
        response_format_non_supported_active = (
            isinstance(request, ResponsesRequest)
            or request.response_format is not None
            and request.response_format.type == "structural_tag"
        )

        if (
            not is_mistral_tokenizer(self.model_tokenizer)
            or isinstance(request, ResponsesRequest)
            or not self.model_tokenizer.supports_grammar
            or any_so_non_supported_active
            or response_format_non_supported_active
        ):
            request = super().adjust_request(request)
# ... omitted for brevity / 为简洁省略 ...
                lark_grammar = grammar_factory.get_lark_from_jinja(
                    template=template,
                    mode=tool_choice,
                    tools=mistral_tools,
                    json_schema=json_schema,
                    parallel_tool_calls=request.parallel_tool_calls,
                    json_only=False,
                )

        request.structured_outputs = StructuredOutputsParams(grammar=lark_grammar)
        request._grammar_from_tool_parser = True
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `MistralToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `MistralToolParser.extract_maybe_reasoning_and_tool_streaming` (lines 284-409)
```python
    def extract_maybe_reasoning_and_tool_streaming(
        self,
        *,
        reasoning_parser: ReasoningParser | None,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: list[int],
        current_token_ids: list[int],
        output_token_ids: Sequence[int],
        reasoning_ended: bool,
        prompt_is_reasoning_end: bool | None,
        request: ChatCompletionRequest,
    ) -> MistralStreamingResult:
        r"""Streaming extraction with reasoning followed by tool-call parsing.

        This method encapsulates the combined reasoning extraction and
        tool-call streaming logic so that the serving layer only needs a
        thin routing branch.

        The flow is:

        1. If a *reasoning_parser* is present and reasoning has **not** ended,
           extract reasoning tokens.  Pre-v15 models may have pre-filled
           `[THINK]...[/THINK]` in system prompts, so we skip the
           prompt-level reasoning-end check for those.
        2. Once reasoning ends (or if there is no reasoning parser), delegate
           to `extract_tool_calls_streaming` and track whether tools were
           called.

# ... omitted for brevity / 为简洁省略 ...
            request=request,
        )
        if delta_message and delta_message.tool_calls:
            tools_called = True

        return MistralStreamingResult(
            delta_message=delta_message,
            reasoning_ended=reasoning_ended,
            tools_called=tools_called,
            current_text=current_text,
            current_token_ids=current_token_ids,
        )
```
**EN:** Streaming extraction with reasoning followed by tool-call parsing.
**CN:** `MistralToolParser.extract_maybe_reasoning_and_tool_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MistralToolParser.extract_tool_calls` (lines 426-540)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        """
        Extract the tool calls from a complete model response.

        Content and tool calls formatting depends on the Mistral's tokenizer version
        used to train the model:

        - < v11: `content[BOT] [{tool_call1},{tool_call2}]`
        - >= v11: `content[BOT]tool_name1{args_call1}[BOT]tool_name2{args_call2}`

        with [BOT] the tool call token.

        Note:
            For tokenizer versions >= v11, tool calls with arguments wrongly formatted
            are still returned as tool calls. This is to allow the model to know it
            tried to make a tool call. It reduces chance of another failure and
            prevents that the context is filled with tool calls wrongly placed in
            assistant message contents.
        """

        # If the tool call token is not present, return a text response
        if self.bot_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

# ... omitted for brevity / 为简洁省略 ...
                    name=tool_call["name"],
                    arguments=tool_call.get("arguments", "{}"),
                ),
            )
            for tool_call in tool_calls
        ]

        return ExtractedToolCallInformation(
            tools_called=True,
            tool_calls=mistral_tool_calls,
            content=content if len(content) > 0 else None,
        )
```
**EN:** Extract the tool calls from a complete model response.
**CN:** `MistralToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MistralToolParser.extract_tool_calls_streaming` (lines 542-574)
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
        has_bot_token = (
            self.bot_token_id in current_token_ids or self.bot_token in current_text
        )
        if not has_bot_token:
            # if the tool call token is not in the tokens generated so far,
            # append output to contents since it's not a tool
            return DeltaMessage(content=delta_text)

        # if the tool call token IS in the tokens generated so far, that
        # means we're parsing as tool calls now
        try:
            if self._is_pre_v11:
                return self._extract_tool_calls_streaming_pre_v11_tokenizer(
                    delta_text=delta_text,
                    delta_token_ids=delta_token_ids,
                )
            else:
                return self._extract_tool_calls_streaming(
                    delta_text=delta_text, delta_token_ids=delta_token_ids
                )
        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `MistralToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MistralToolParser._extract_tool_calls_streaming` (lines 576-621)
```python
    def _extract_tool_calls_streaming(
        self,
        delta_text: str,
        delta_token_ids: Sequence[int],
    ) -> DeltaMessage | None:
        """
        Extracts tool calls for Mistral models
        doing tool calls of the following format:
        `[TOOL_CALLS]add{"a": 3.5, "b": 4}`
        """
        additional_content: str = ""
        if self.streaming_state == StreamingState.WAITING_FOR_TOOL_START:
            # this is the first tool call
            if self.bot_token not in delta_text:
                return DeltaMessage(content=delta_text)
            if not delta_text.startswith(self.bot_token):
                additional_content += delta_text.split(self.bot_token)[0]
                delta_text = self.bot_token + "".join(
                    delta_text.split(self.bot_token)[1:]
                )

        delta_tool_calls = self._generate_delta_tool_call(delta_text)
        if not additional_content and len(delta_tool_calls) == 0:
            if self.streaming_state in [
                StreamingState.PARSING_ARGUMENTS,
                StreamingState.PARSING_ARGUMENTS_COMPLETED,
                StreamingState.TOOL_COMPLETE,
                StreamingState.ALL_TOOLS_COMPLETE,
            ]:
                # Return an empty DeltaMessage once the tool calls are all done
                # so that finish_reason gets set.
                return DeltaMessage()
            else:
                # return None when the tool is not likely to be finished
                # This can occur when the name is being parsed for example
                # and we wait for the name to be complete
                # before sending the function name
                return None

        delta = DeltaMessage()
        if additional_content:
            delta.content = additional_content
        if len(delta_tool_calls) > 0:
            delta.tool_calls = delta_tool_calls

        return delta
```
**EN:** Extracts tool calls for Mistral models doing tool calls of the following format: `[TOOL_CALLS]add{"a": 3.5, "b": 4}`
**CN:** `MistralToolParser._extract_tool_calls_streaming` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `MistralToolParser._extract_tool_calls_streaming_pre_v11_tokenizer` (lines 713-880)
```python
    def _extract_tool_calls_streaming_pre_v11_tokenizer(
        self,
        delta_text: str,
        delta_token_ids: Sequence[int],
    ) -> DeltaMessage | None:
        """
        Extracts tool calls for Mistral models
        doing tool calls of the following format:
        `[TOOL_CALLS][{"name": "add", "arguments":{"a": 3.5, "b": 4}}`
        """
        assert self.parse_coro is not None
        content = None
        delta_tool_calls: list[DeltaToolCall] = []
        current_tool_call: DeltaToolCall = DeltaToolCall(
            index=self.current_tool_id, type="function"
        )
        current_tool_call_modified = False
        if self.bot_token_id in delta_token_ids or self.bot_token in delta_text:
            # this is the first tool call
            if not delta_text.startswith(self.bot_token):
                content = delta_text.split(self.bot_token)[0]
            delta_text = "".join(delta_text.split(self.bot_token)[1:])

        # Cut smartly the delta text to catch the ijson events
        # as ijson does not give us the index in the text at each event.
        # We need to cut so that we know
        # where in the text the events are emitted from.
        while len(delta_text) > 0:
            streaming_state_before_parse = self.streaming_state

# ... omitted for brevity / 为简洁省略 ...
        if content or len(delta_tool_calls) > 0:
            delta_message = DeltaMessage()
            if content:
                delta_message.content = content
            if len(delta_tool_calls) > 0:
                delta_message.tool_calls = delta_tool_calls
            return delta_message
        else:
            if self.streaming_state == StreamingState.ALL_TOOLS_COMPLETE:
                return DeltaMessage()
            else:
                return None
```
**EN:** Extracts tool calls for Mistral models doing tool calls of the following format: `[TOOL_CALLS][{"name": "add", "arguments":{"a": 3.5, "b": 4}}`
**CN:** `MistralToolParser._extract_tool_calls_streaming_pre_v11_tokenizer` 方法处理增量式流输出，只返回客户端应该看到的新差量。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Tokenizer-aware parsing**: Tokenizer-aware parsing. / 依赖 tokenizer 词表和 token ID 来提高边界检测效率。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `dataclasses`, `enum`, `random`, `string`, `typing`
- **Third-party / 第三方**: `__future__`, `ijson`, `regex`, `mistral_common.protocol.instruct.tool_calls`, `pydantic`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.reasoning.mistral_reasoning_parser`, `vllm.sampling_params`, `vllm.tokenizers`, `vllm.tokenizers.mistral`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.utils.mistral`
