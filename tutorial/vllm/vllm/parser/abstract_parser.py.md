# abstract_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/parser/abstract_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `abstract_parser`-related logic centered around `StreamState`, `Parser`. / 实现与 `abstract_parser` 相关的逻辑，核心符号包括 `StreamState`, `Parser`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-48)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import contextlib
import json
from abc import abstractmethod
from collections.abc import Sequence
from dataclasses import dataclass, field
from functools import cached_property

from openai.types.responses import (
    ResponseFunctionToolCall,
    ResponseOutputItem,
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseReasoningItem,
    ToolChoiceFunction,
)
from openai.types.responses.response_output_text import Logprob
from openai.types.responses.response_reasoning_item import (
    Content as ResponseReasoningTextContent,
)
from pydantic import TypeAdapter, ValidationError

    # ...
    extract_required_tool_call_streaming,
)
from vllm.tool_parsers.utils import Tool
from vllm.utils import random_uuid

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `contextlib`, `json`, `abc`, external packages such as `openai.types.responses`, `openai.types.responses.response_output_text`, `openai.types.responses.response_reasoning_item`, vLLM modules such as `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`. It prepares the symbols later used by `StreamState`, `Parser`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol` 等 vLLM 内部依赖。 这些准备工作为后续的 `StreamState`, `Parser` 提供上下文。

### StreamState (lines 52-64)
```python
class StreamState:
    """Mutable state for ``Parser.parse_delta()``. One per stream."""

    reasoning_ended: bool = False
    tool_call_text_started: bool = False
    prompt_reasoning_checked: bool = False
    previous_text: str = ""
    previous_token_ids: list[int] = field(default_factory=list)
    history_tool_call_cnt: int = 0
    tool_call_id_type: str = "random"
    # only used for "required" and "named tool" choices,
    # tracks whether function name has been fully returned in the stream yet
    function_name_returned: bool = False
```
**EN:** `StreamState`: Mutable state for ``Parser.parse_delta()``.
**CN:** `StreamState` 是该文件中的核心类，用于封装与 `StreamState` 相关的状态和行为。

### Parser overview (lines 67-326)
```python
class Parser:
    """
    Abstract Parser class that unifies ReasoningParser and ToolParser into
    a single interface for parsing model output.

    This class provides a unified way to handle both reasoning extraction
    (e.g., chain-of-thought content in <think> tags) and tool call extraction
    (e.g., function calls in XML/JSON format) from model outputs.

    Subclasses can either:
    1. Override the abstract methods directly for custom parsing logic
    2. Set `reasoning_parser` and `tool_parser` properties to delegate to
       existing parser implementations

    Class Attributes:
        reasoning_parser_cls: The ReasoningParser class to use (for compatibility
            with code that needs the class, not instance).
        tool_parser_cls: The ToolParser class to use (for compatibility with
            code that needs the class, not instance).
    """

    # Class-level parser classes for compatibility with existing patterns
    # Subclasses should override these if they use specific parser classes
    reasoning_parser_cls: type[ReasoningParser] | None = None
    tool_parser_cls: type[ToolParser] | None = None

    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
    # ...
```
**EN:** `Parser`: Abstract Parser class that unifies ReasoningParser and ToolParser into a single interface for parsing model output. Key methods include `__init__`, `vocab`, `reasoning_parser`, `reasoning_parser`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `Parser` 是该文件中的核心类，用于封装与 `Parser` 相关的状态和行为。 关键方法包括 `__init__`, `vocab`, `reasoning_parser`, `reasoning_parser`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### Parser.__init__ (lines 93-104)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        """
        Initialize the Parser.

        Args:
            tokenizer: The tokenizer used by the model. This is required for
                token-based parsing operations.
        """
        self.model_tokenizer = tokenizer
        self._reasoning_parser: ReasoningParser | None = None
        self._tool_parser: ToolParser | None = None
        self._stream_state = StreamState()
```
**EN:** `__init__`: Initialize the Parser. It mainly works with `tokenizer`, `*args`, `**kwargs`. Inside the body, it relies on `StreamState` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `tokenizer`, `*args`, `**kwargs` 等参数。 实现过程中会调用 `StreamState` 等函数完成关键步骤。

### Parser.parse_delta (lines 317-326)
```python
    def parse_delta(
        self,
        delta_text: str,
        delta_token_ids: list[int],
        request: ChatCompletionRequest | ResponsesRequest,
        prompt_token_ids: list[int] | None = None,
    ) -> DeltaMessage | None:
        """Parse a single streaming delta, orchestrating reasoning then
        tool call extraction via internal stream state.
        """
```
**EN:** `parse_delta`: Parse a single streaming delta, orchestrating reasoning then tool call extraction via internal stream state. It mainly works with `delta_text`, `delta_token_ids`, `request`, `prompt_token_ids`.
**CN:** `parse_delta` 负责把原始输入解析为结构化对象。 它主要处理 `delta_text`, `delta_token_ids`, `request`, `prompt_token_ids` 等参数。

### Parser.extract_response_outputs (lines 177-204)
```python
    def extract_response_outputs(
        self,
        *,
        model_output: str,
        model_output_token_ids: Sequence[int],
        request: ResponsesRequest,
        enable_auto_tools: bool = False,
        tool_call_id_type: str = "random",
        logprobs: list[Logprob] | None = None,
    ) -> list[ResponseOutputItem]:
        """
        Extract reasoning, content, and tool calls from a complete
        model-generated string and return as ResponseOutputItem objects.

        Used for non-streaming responses where we have the entire model
        response available before sending to the client.

        Args:
            model_output: The complete model-generated string.
            model_output_token_ids: The token IDs of the model output.
            request: The request object used to generate the output.
            enable_auto_tools: Whether to enable automatic tool call parsing.
            tool_call_id_type: Type of tool call ID generation ("random", etc).
            logprobs: Pre-computed logprobs for the output text, if any.

        Returns:
            A list of ResponseOutputItem objects.
        """
```
**EN:** `extract_response_outputs`: Extract reasoning, content, and tool calls from a complete model-generated string and return as ResponseOutputItem objects. It mainly works with `model_output`, `model_output_token_ids`, `request`, `enable_auto_tools`.
**CN:** `extract_response_outputs` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_output`, `model_output_token_ids`, `request`, `enable_auto_tools` 等参数。

### Parser.extract_tool_calls_streaming (lines 290-314)
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
        """
        Extract tool calls from a streaming delta message.

        Args:
            previous_text: Text from all previous tokens.
            current_text: Text including the current delta.
            delta_text: The new text in this delta.
            previous_token_ids: Token IDs from previous generation.
            current_token_ids: All token IDs including current.
            delta_token_ids: The new token IDs in this delta.
            request: The request object.

        Returns:
            A DeltaMessage with tool_calls field, or None.
        """
```
**EN:** `extract_tool_calls_streaming`: Extract tool calls from a streaming delta message. It mainly works with `previous_text`, `current_text`, `delta_text`, `previous_token_ids`.
**CN:** `extract_tool_calls_streaming` 负责实现本模块使用的辅助逻辑。 它主要处理 `previous_text`, `current_text`, `delta_text`, `previous_token_ids` 等参数。

### DelegatingParser overview (lines 329-732)
```python
class DelegatingParser(Parser):
    """
    A Parser implementation that delegates to separate ReasoningParser and
    ToolParser instances.

    This is the recommended base class for creating model-specific parsers
    that combine existing reasoning and tool parser implementations.
    Subclasses should set `self._reasoning_parser` and `self._tool_parser`
    in their `__init__` method.

    If either parser is None, the corresponding methods will return default
    values (no reasoning extraction, no tool calls).
    """

    def extract_reasoning(
        self,
        model_output: str,
        request: ChatCompletionRequest | ResponsesRequest,
    ) -> tuple[str | None, str | None]:
        if self._reasoning_parser is None:
            return None, model_output
        return self._reasoning_parser.extract_reasoning(model_output, request)

    def extract_response_outputs(
        self,
        *,
        model_output: str,
    # ...
```
**EN:** `DelegatingParser`: A Parser implementation that delegates to separate ReasoningParser and ToolParser instances. It extends `Parser`. Key methods include `extract_reasoning`, `extract_response_outputs`, `adjust_request`, `extract_reasoning_streaming`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `DelegatingParser` 是该文件中的核心类，用于封装与 `DelegatingParser` 相关的状态和行为。 它继承自 `Parser`。 关键方法包括 `extract_reasoning`, `extract_response_outputs`, `adjust_request`, `extract_reasoning_streaming`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### DelegatingParser.parse_delta (lines 652-732)
```python
    def parse_delta(
        self,
        delta_text: str,
        delta_token_ids: list[int],
        request: ChatCompletionRequest | ResponsesRequest,
        prompt_token_ids: list[int] | None = None,
    ) -> DeltaMessage | None:
        state = self._stream_state

        if not state.prompt_reasoning_checked and prompt_token_ids is not None:
            state.prompt_reasoning_checked = True
            if self._reasoning_parser is None or self.is_reasoning_end(
                prompt_token_ids
            ):
                state.reasoning_ended = True

        current_text = state.previous_text + delta_text
        current_token_ids = state.previous_token_ids + delta_token_ids
        delta_message: DeltaMessage | None = None

    # ...
        ):
            delta_message = DeltaMessage(content=delta_text)

        state.previous_text = current_text
        state.previous_token_ids = current_token_ids
        return delta_message
```
**EN:** `parse_delta` parses raw inputs into structured objects. It mainly works with `delta_text`, `delta_token_ids`, `request`, `prompt_token_ids`. Inside the body, it relies on `self._in_reasoning_phase`, `self._in_tool_call_phase`, `self.extract_reasoning_streaming` to complete the main steps.
**CN:** `parse_delta` 负责把原始输入解析为结构化对象。 它主要处理 `delta_text`, `delta_token_ids`, `request`, `prompt_token_ids` 等参数。 实现过程中会调用 `self._in_reasoning_phase`, `self._in_tool_call_phase`, `self.extract_reasoning_streaming` 等函数完成关键步骤。

### DelegatingParser._parse_tool_calls (lines 440-511)
```python
    def _parse_tool_calls(
        self,
        request: ResponsesRequest,
        content: str | None,
        enable_auto_tools: bool,
    ) -> tuple[list[FunctionCall], str | None]:
        """
        TODO(qandrew): merge _parse_tool_calls_from_content
        for ChatCompletions into this function
        Parse tool calls from content based on request tool_choice settings.

        Returns:
            A tuple of (function_calls, remaining_content) if tool calls
            were parsed
        """
        function_calls: list[FunctionCall] = []

        if request.tool_choice and isinstance(
            request.tool_choice,
            (ToolChoiceFunction, ChatCompletionNamedToolChoiceParam),
    # ...
                if remaining_content and remaining_content.strip() == "":
                    remaining_content = None
                return function_calls, remaining_content

        # No tool calls
        return [], content
```
**EN:** `_parse_tool_calls`: TODO(qandrew): merge _parse_tool_calls_from_content for ChatCompletions into this function Parse tool calls from content based on request tool_choice settings. It mainly works with `request`, `content`, `enable_auto_tools`. Inside the body, it relies on `function_calls.append`, `self._tool_parser.extract_tool_calls`, `FunctionCall` to complete the main steps.
**CN:** `_parse_tool_calls` 负责把原始输入解析为结构化对象。 它主要处理 `request`, `content`, `enable_auto_tools` 等参数。 实现过程中会调用 `function_calls.append`, `self._tool_parser.extract_tool_calls`, `FunctionCall` 等函数完成关键步骤。

### DelegatingParser.extract_response_outputs (lines 352-427)
```python
    def extract_response_outputs(
        self,
        *,
        model_output: str,
        model_output_token_ids: Sequence[int],
        request: ResponsesRequest,
        enable_auto_tools: bool = False,
        tool_call_id_type: str = "random",
        logprobs: list[Logprob] | None = None,
    ) -> list[ResponseOutputItem]:
        # First extract reasoning
        reasoning, content = self.extract_reasoning(model_output, request)

        # Then parse tool calls from the content
        tool_calls, content = self._parse_tool_calls(
            request=request,
            content=content,
            enable_auto_tools=enable_auto_tools,
        )

    # ...
                    name=tool_call.name,
                    arguments=tool_call.arguments,
                )
                outputs.append(tool_call_item)

        return outputs
```
**EN:** `extract_response_outputs` implements helper logic used by this module. It mainly works with `model_output`, `model_output_token_ids`, `request`, `enable_auto_tools`. Inside the body, it relies on `self.extract_reasoning`, `self._parse_tool_calls`, `ResponseReasoningItem` to complete the main steps.
**CN:** `extract_response_outputs` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_output`, `model_output_token_ids`, `request`, `enable_auto_tools` 等参数。 实现过程中会调用 `self.extract_reasoning`, `self._parse_tool_calls`, `ResponseReasoningItem` 等函数完成关键步骤。

### DelegatingParser._extract_tool_calls_streaming (lines 575-630)
```python
    def _extract_tool_calls_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
        request: ChatCompletionRequest | ResponsesRequest,
        # The following parameters are used for "required" tool choice parsing and are
        # tracked in StreamState for streaming parsing.
        tool_call_idx: int | None = None,
        tool_call_id_type: str = "random",
        function_name_returned: bool = False,
    ) -> tuple[DeltaMessage | None, bool]:
        assert self._tool_parser is not None
        supports_required_and_named = self._tool_parser.supports_required_and_named
        if (
            supports_required_and_named
            and request.tool_choice
    # ...
            delta_text,
            previous_token_ids,
            current_token_ids,
            delta_token_ids,
            request,  # type: ignore[arg-type]
        ), False
```
**EN:** `_extract_tool_calls_streaming` implements helper logic used by this module. It mainly works with `previous_text`, `current_text`, `delta_text`, `previous_token_ids`. Inside the body, it relies on `extract_named_tool_call_streaming`, `extract_required_tool_call_streaming`, `self.extract_tool_calls_streaming` to complete the main steps.
**CN:** `_extract_tool_calls_streaming` 负责实现本模块使用的辅助逻辑。 它主要处理 `previous_text`, `current_text`, `delta_text`, `previous_token_ids` 等参数。 实现过程中会调用 `extract_named_tool_call_streaming`, `extract_required_tool_call_streaming`, `self.extract_tool_calls_streaming` 等函数完成关键步骤。

### _WrappedParser (lines 735-763)
```python
class _WrappedParser(DelegatingParser):
    """
    A DelegatingParser subclass that instantiates parsers from class attributes.

    This class is used to dynamically create a parser that wraps individual
    ReasoningParser and ToolParser classes. The class attributes
    `reasoning_parser_cls` and `tool_parser_cls` should be set before
    instantiation.

    Usage:
        _WrappedParser.reasoning_parser_cls = MyReasoningParser
        _WrappedParser.tool_parser_cls = MyToolParser
        parser = _WrappedParser(tokenizer)
    """

    reasoning_parser_cls: type[ReasoningParser] | None = None
    tool_parser_cls: type[ToolParser] | None = None

    def __init__(
        self, tokenizer: TokenizerLike, tools: list[Tool] | None = None, **kwargs
    ):
        super().__init__(tokenizer)
        # Instantiate the underlying parsers from class attributes
        if self.__class__.reasoning_parser_cls is not None:
            self._reasoning_parser = self.__class__.reasoning_parser_cls(
                tokenizer, **kwargs
            )
        if self.__class__.tool_parser_cls is not None:
            self._tool_parser = self.__class__.tool_parser_cls(tokenizer, tools)
```
**EN:** `_WrappedParser`: A DelegatingParser subclass that instantiates parsers from class attributes. It extends `DelegatingParser`. Key methods include `__init__`.
**CN:** `_WrappedParser` 是该文件中的核心类，用于封装与 `_WrappedParser` 相关的状态和行为。 它继承自 `DelegatingParser`。 关键方法包括 `__init__`。

## Key Concepts / 关键概念
- **`StreamState`**: Core class that organizes module behavior. / **`StreamState`**：组织模块行为的核心类。
- **`Parser`**: Core class that organizes module behavior. / **`Parser`**：组织模块行为的核心类。
- **`DelegatingParser`**: Core class that organizes module behavior. / **`DelegatingParser`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: contextlib, json, abc, collections.abc, dataclasses, functools
- **Third-party / 第三方**: openai.types.responses, openai.types.responses.response_output_text, openai.types.responses.response_reasoning_item, pydantic
- **Internal vLLM / vLLM 内部依赖**: vllm.entrypoints.chat_utils, vllm.entrypoints.openai.chat_completion.protocol, vllm.entrypoints.openai.engine.protocol, vllm.entrypoints.openai.responses.protocol, vllm.logger, vllm.reasoning.abs_reasoning_parsers, vllm.tokenizers, vllm.tool_parsers.abstract_tool_parser, vllm.tool_parsers.streaming, vllm.tool_parsers.utils, vllm.utils
