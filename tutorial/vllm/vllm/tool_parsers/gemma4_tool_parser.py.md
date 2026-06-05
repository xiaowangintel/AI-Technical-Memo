# gemma4_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/gemma4_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the gemma4 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 gemma4 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 3-17)
```python
"""
Tool call parser for Google Gemma4 models.

Gemma4 uses a custom serialization format (not JSON) for tool calls::

    <|tool_call>call:func_name{key:<|"|>value<|"|>,num:42}<tool_call|>

Strings are delimited by ``<|"|>`` (token 52), keys are unquoted, and
multiple tool calls are concatenated without separators.

Used when ``--enable-auto-tool-choice --tool-call-parser gemma4`` are set.

For offline inference tool call parsing (direct ``tokenizer.decode()`` output),
see ``vllm.tool_parsers.gemma4_utils.parse_tool_calls``.
"""
```
**EN:** Tool call parser for Google Gemma4 models.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Imports and setup (lines 19-42)
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
from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
)
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import Tool, ToolParser
from vllm.tool_parsers.utils import find_common_prefix
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 44-44)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Module constants/configuration 2 (lines 47-49)
```python
TOOL_CALL_START = "<|tool_call>"
TOOL_CALL_END = "<tool_call|>"
STRING_DELIM = '<|"|>'
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `_parse_gemma4_value` (lines 57-82)
```python
def _parse_gemma4_value(value_str: str) -> object:
    """Parse a single Gemma4 value (after key:) into a Python object."""
    value_str = value_str.strip()
    if not value_str:
        return value_str

    # Boolean
    if value_str == "true":
        return True
    if value_str == "false":
        return False

    # Null
    if value_str.lower() in ("null", "none", "nil"):
        return None

    # Number (int or float)
    try:
        if "." in value_str:
            return float(value_str)
        return int(value_str)
    except ValueError:
        pass

    # Bare string (no <|"|> delimiters — shouldn't happen but be safe)
    return value_str
```
**EN:** Parse a single Gemma4 value (after key:) into a Python object.
**CN:** `_parse_gemma4_value` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `_parse_gemma4_args` (lines 85-224)
```python
def _parse_gemma4_args(args_str: str, *, partial: bool = False) -> dict:
    """Parse Gemma4's custom key:value format into a Python dict.

    Format examples::

        location:<|"|>Tokyo<|"|>
        location:<|"|>San Francisco<|"|>,unit:<|"|>celsius<|"|>
        count:42,flag:true
        nested:{inner_key:<|"|>val<|"|>}
        items:[<|"|>a<|"|>,<|"|>b<|"|>]

    Args:
        args_str: The raw Gemma4 argument string.
        partial: When True (streaming), bare values at end of string are
            omitted because they may be incomplete and type-unstable
            (e.g. partial boolean parsed as bare string).

    Returns a dict ready for ``json.dumps()``.
    """
    if not args_str or not args_str.strip():
        return {}

    result: dict = {}
    i = 0
    n = len(args_str)

    while i < n:
        # Skip whitespace and commas
        while i < n and args_str[i] in (" ", ",", "\n", "\t"):
            i += 1
# ... omitted for brevity / 为简洁省略 ...
                break
            if partial:
                raw_val = args_str[val_start:i].strip()
                if raw_val.endswith("."):
                    # Trailing dot means decimal digits may still arrive
                    # (e.g. "108." may become "108.2"). Parsing now would
                    # yield float("108.") == 108.0, whose json repr "108.0"
                    # corrupts the streaming diff when the true digit lands.
                    break
            result[key] = _parse_gemma4_value(args_str[val_start:i])

    return result
```
**EN:** Parse Gemma4's custom key:value format into a Python dict.
**CN:** `_parse_gemma4_args` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `_parse_gemma4_array` (lines 227-311)
```python
def _parse_gemma4_array(arr_str: str, *, partial: bool = False) -> list:
    """Parse a Gemma4 array content string into a Python list."""
    items: list = []
    i = 0
    n = len(arr_str)

    while i < n:
        while i < n and arr_str[i] in (" ", ",", "\n", "\t"):
            i += 1
        if i >= n:
            break

        # String element
        if arr_str[i:].startswith(STRING_DELIM):
            i += len(STRING_DELIM)
            end_pos = arr_str.find(STRING_DELIM, i)
            if end_pos == -1:
                items.append(arr_str[i:])
                break
            items.append(arr_str[i:end_pos])
            i = end_pos + len(STRING_DELIM)

        # Nested object
        elif arr_str[i] == "{":
            depth = 1
            obj_start = i + 1
            i += 1
            while i < n and depth > 0:
                if arr_str[i:].startswith(STRING_DELIM):
                    i += len(STRING_DELIM)
# ... omitted for brevity / 为简洁省略 ...
                    "Gemma4 array parser made no progress at position %d; "
                    "aborting on malformed input.",
                    i,
                )
                break
            if partial:
                raw_val = arr_str[val_start:i].strip()
                if raw_val.endswith("."):
                    break
            items.append(_parse_gemma4_value(arr_str[val_start:i]))

    return items
```
**EN:** Parse a Gemma4 array content string into a Python list.
**CN:** `_parse_gemma4_array` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Class `Gemma4ToolParser` (lines 319-790)
```python
class Gemma4ToolParser(ToolParser):
    """
    Tool call parser for Google Gemma4 models.

    Handles the Gemma4 function call format::

        <|tool_call>call:func_name{key:<|"|>value<|"|>}<tool_call|>

    Used when ``--enable-auto-tool-choice --tool-call-parser gemma4``
    are set.

    Streaming strategy: **accumulate-then-parse-then-diff**

    Instead of trying to convert Gemma4's custom format to JSON
    token-by-token (which fails because Gemma4 uses bare keys, custom
```
**EN:** Tool call parser for Google Gemma4 models.
**CN:** 定义 `Gemma4ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Gemma4ToolParser.__init__` (lines 346-381)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        # Token strings
        self.tool_call_start_token = TOOL_CALL_START
        self.tool_call_end_token = TOOL_CALL_END

        # Token IDs
        self.tool_call_start_token_id = self.vocab.get(TOOL_CALL_START)
        self.tool_call_end_token_id = self.vocab.get(TOOL_CALL_END)

        if self.tool_call_start_token_id is None:
            raise RuntimeError(
                "Gemma4 ToolParser could not locate the tool call start "
                f"token '{TOOL_CALL_START}' in the tokenizer!"
            )

        # Regex for non-streaming: extract complete tool calls.
        # Supports function names with letters, digits, underscores,
        # hyphens, and dots (e.g. "get-weather", "module.func").
        self.tool_call_regex = re.compile(
            r"<\|tool_call>call:([\w\-\.]+)\{(.*?)\}<tool_call\|>",
            re.DOTALL,
        )

        # Streaming state — reset per-request via _reset_streaming_state()
        self._reset_streaming_state()

        # Delta buffer for handling multi-token special sequences
        self.buffered_delta_text = ""
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Gemma4ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Gemma4ToolParser._reset_streaming_state` (lines 383-388)
```python
    def _reset_streaming_state(self) -> None:
        """Reset all streaming state for a new request."""
        self.current_tool_id = -1
        self.current_tool_name_sent = False
        self.prev_tool_call_arr: list[dict] = []
        self.streamed_args_for_tool: list[str] = []
```
**EN:** Reset all streaming state for a new request.
**CN:** `Gemma4ToolParser._reset_streaming_state` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `Gemma4ToolParser.adjust_request` (lines 390-402)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # Don't skip special tokens — <|tool_call> etc. are needed for
            # the parser to detect tool calls. Apply to BOTH
            # ChatCompletionRequest and ResponsesRequest (the previous
            # isinstance(ChatCompletionRequest) guard caused tool-call
            # delimiters to be stripped on /v1/responses, leaking raw
            # `call:fn{...}` text via output_text.delta).
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Gemma4ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Gemma4ToolParser.extract_tool_calls` (lines 440-484)
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
            for func_name, args_str in matches:
                arguments = _parse_gemma4_args(args_str)
                tool_calls.append(
                    ToolCall(
                        type="function",
                        function=FunctionCall(
                            name=func_name,
                            arguments=json.dumps(arguments, ensure_ascii=False),
                        ),
                    )
                )

            # Content = text before first tool call (if any)
            content_end = model_output.find(self.tool_call_start_token)
            content = model_output[:content_end].strip() if content_end > 0 else None

            return ExtractedToolCallInformation(
                tools_called=True,
                tool_calls=tool_calls,
                content=content if content else None,
            )

        except Exception:
            logger.exception("Error extracting tool calls from Gemma4 response")
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Gemma4ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Gemma4ToolParser.extract_tool_calls_streaming` (lines 490-521)
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
        # Buffer delta text to handle multi-token special sequences
        delta_text = self._buffer_delta_text(delta_text)
        # Keep current_text from the upstream stream state. The buffered delta
        # is only for emission, and must not be stitched back into the
        # accumulated model text or normal content like "<div>" can be
        # duplicated into "<<div>" when a tool call just ended.

        # If no tool call token seen yet, emit as content
        if self.tool_call_start_token not in current_text:
            if delta_text:
                return DeltaMessage(content=delta_text)
            return None

        try:
            return self._extract_streaming(
                previous_text=previous_text,
                current_text=current_text,
                delta_text=delta_text,
            )
        except Exception:
            logger.exception("Error in Gemma4 streaming tool call extraction")
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Gemma4ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Gemma4ToolParser._extract_streaming` (lines 523-579)
```python
    def _extract_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
    ) -> DeltaMessage | None:
        """Tag-counting streaming parser.

        Uses the proven approach from FunctionGemma/Hermes: count start/end
        tags in previous vs current text to determine phase, then
        accumulate-parse-diff for arguments.

        Format: ``<|tool_call>call:name{args}<tool_call|>``
        """
        start_count = current_text.count(self.tool_call_start_token)
        end_count = current_text.count(self.tool_call_end_token)
        prev_start_count = previous_text.count(self.tool_call_start_token)
        prev_end_count = previous_text.count(self.tool_call_end_token)

        # Case 1: Not inside any tool call — emit as content
        if (
            start_count == end_count
            and prev_end_count == end_count
            and self.tool_call_end_token not in delta_text
        ):
            if delta_text:
                return DeltaMessage(content=delta_text)
            return None

        # Case 2: Starting a new tool call
# ... omitted for brevity / 为简洁省略 ...

        # Case 4: In the middle of a tool call — parse partial content
        if start_count > end_count:
            return self._handle_tool_call_middle(current_text)

        # Default: generate text outside tool calls
        if delta_text:
            text = delta_text.replace(self.tool_call_start_token, "")
            text = text.replace(self.tool_call_end_token, "")
            if text:
                return DeltaMessage(content=text)
        return None
```
**EN:** Tag-counting streaming parser.
**CN:** `Gemma4ToolParser._extract_streaming` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `Gemma4ToolParser._handle_tool_call_middle` (lines 616-653)
```python
    def _handle_tool_call_middle(self, current_text: str) -> DeltaMessage | None:
        """Handle streaming when we're inside an active tool call.

        Accumulates the raw Gemma4 arguments, parses them into JSON, and
        diffs against the previously-streamed JSON to emit only the new
        fragment.
        """
        func_name, args_part = self._extract_partial_call(current_text)

        if func_name is None:
            return None

        # Step 1: Send function name (once)
        if not self.current_tool_name_sent and func_name:
            self.current_tool_name_sent = True
            self.prev_tool_call_arr[self.current_tool_id] = {
                "name": func_name,
                "arguments": {},
            }
            return DeltaMessage(
                tool_calls=[
                    DeltaToolCall(
                        index=self.current_tool_id,
                        type="function",
                        id=make_tool_call_id(),
                        function=DeltaFunctionCall(
                            name=func_name,
                            arguments="",
                        ).model_dump(exclude_none=True),
                    )
                ]
            )

        # Step 2: Parse and diff arguments
        if self.current_tool_name_sent and args_part:
            return self._emit_argument_diff(args_part)

        return None
```
**EN:** Handle streaming when we're inside an active tool call.
**CN:** `Gemma4ToolParser._handle_tool_call_middle` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `Gemma4ToolParser._handle_tool_call_end` (lines 655-694)
```python
    def _handle_tool_call_end(self, current_text: str) -> DeltaMessage | None:
        """Handle streaming when a tool call has just completed.

        Performs a final parse of the complete tool call and flushes
        any remaining un-streamed argument fragments.
        """
        if self.current_tool_id < 0 or self.current_tool_id >= len(
            self.prev_tool_call_arr
        ):
            logger.debug(
                "Tool call end detected but no active tool call (current_tool_id=%d)",
                self.current_tool_id,
            )
            return None

        # Parse the complete tool call using regex for accuracy
        all_matches = self.tool_call_regex.findall(current_text)
        if self.current_tool_id < len(all_matches):
            _, args_str = all_matches[self.current_tool_id]
            final_args = _parse_gemma4_args(args_str)
            final_args_json = json.dumps(final_args, ensure_ascii=False)

            prev_streamed = self.streamed_args_for_tool[self.current_tool_id]
            if len(final_args_json) > len(prev_streamed):
                diff = final_args_json[len(prev_streamed) :]
                self.streamed_args_for_tool[self.current_tool_id] = final_args_json
                self.prev_tool_call_arr[self.current_tool_id]["arguments"] = final_args

                return DeltaMessage(
                    tool_calls=[
                        DeltaToolCall(
                            index=self.current_tool_id,
                            function=DeltaFunctionCall(arguments=diff).model_dump(
                                exclude_none=True
                            ),
                        )
                    ]
                )

        return None
```
**EN:** Handle streaming when a tool call has just completed.
**CN:** `Gemma4ToolParser._handle_tool_call_end` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
