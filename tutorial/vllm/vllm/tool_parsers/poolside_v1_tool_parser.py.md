# poolside_v1_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/poolside_v1_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the poolside v1 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 poolside v1 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Docstring (lines 3-12)
```python
"""
GLM-4 Tool Call Parser with incremental string streaming support.

This parser fixes the streaming issue reported in Issue #32829 where long string
parameters (e.g., file content with 4000+ characters of code) are buffered until
complete, causing multi-second delays before the user sees any content.

The fix streams string values incrementally as they arrive, providing a true
streaming experience for long content.
"""
```
**EN:** GLM-4 Tool Call Parser with incremental string streaming support.
**CN:** 模块文档字符串直接说明了该文件的用途。

### Imports and setup (lines 14-43)
```python
import ast
import json
from collections.abc import Sequence
from typing import Any

import partial_json_parser.core.complete
import regex as re
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
from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
)
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 45-45)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `PoolsideV1ToolParser` (lines 48-583)
```python
class PoolsideV1ToolParser(ToolParser):
    """Tool parser for GLM-4 models with incremental string streaming.

    This parser emits tool-call deltas incrementally as arguments arrive.
    For string-type parameters, content is streamed character-by-character
    rather than waiting for the complete </arg_value> tag.
    """
```
**EN:** Tool parser for GLM-4 models with incremental string streaming.
**CN:** 定义 `PoolsideV1ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `PoolsideV1ToolParser.__init__` (lines 56-99)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)
        # Stateful streaming fields
        self.current_tool_name_sent: bool = False
        self.prev_tool_call_arr: list[dict[str, Any]] = []
        self.current_tool_id: int = -1
        self.streamed_args_for_tool: list[str] = []

        self.tool_call_start_token: str = "<tool_call>"
        self.tool_call_end_token: str = "</tool_call>"
        self.arg_key_start: str = "<arg_key>"
        self.arg_key_end: str = "</arg_key>"
        self.arg_val_start: str = "<arg_value>"
        self.arg_val_end: str = "</arg_value>"

        self.tool_calls_start_token = self.tool_call_start_token

        self.func_call_regex = re.compile(r"<tool_call>.*?</tool_call>", re.DOTALL)
        self.func_detail_regex = re.compile(
            r"<tool_call>([^\n]*)\n(.*)</tool_call>", re.DOTALL
        )
        self.func_arg_regex = re.compile(
            r"<arg_key>(.*?)</arg_key>\s*<arg_value>(.*?)</arg_value>", re.DOTALL
        )

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        self.tool_call_start_token_id = self.vocab.get(self.tool_call_start_token)
        self.tool_call_end_token_id = self.vocab.get(self.tool_call_end_token)
        self._buffer: str = ""

        # Streaming state for incremental tool-call streaming
        self._in_tool_call: bool = False
        self._current_tool_name: str | None = None
        self._pending_key: str | None = None
        self._streaming_string_value: bool = False
        self._tool_call_ids: list[str] = []
        self._args_started: list[bool] = []
        self._args_closed: list[bool] = []
        self._seen_keys: list[set[str]] = []
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `PoolsideV1ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `PoolsideV1ToolParser._is_string_type` (lines 127-146)
```python
    def _is_string_type(
        tool_name: str,
        arg_name: str,
        tools: list[Tool] | None,
    ) -> bool:
        if tools is None:
            return False
        for tool in tools:
            if tool.function.name != tool_name:
                continue
            if tool.function.parameters is None:
                return False
            arg_type = (
                tool.function.parameters.get("properties", {})
                .get(arg_name, {})
                .get("type", None)
            )
            return arg_type == "string"
        logger.debug("No tool named '%s'.", tool_name)
        return False
```
**EN:** Provides the `_is_string_type` helper used by this module to keep parsing logic modular and reusable.
**CN:** `PoolsideV1ToolParser._is_string_type` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `PoolsideV1ToolParser.adjust_request` (lines 159-170)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        """Adjust request parameters for tool call token handling."""
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # Ensure tool call tokens (<tool_call>, </tool_call>) are not skipped
            # during decoding. Even though they are not marked as special tokens,
            # setting skip_special_tokens=False ensures proper handling in
            # transformers 5.x where decoding behavior may have changed.
            request.skip_special_tokens = False
        return request
```
**EN:** Adjust request parameters for tool call token handling.
**CN:** `PoolsideV1ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `PoolsideV1ToolParser.extract_tool_calls` (lines 172-227)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        matched_tool_calls = self.func_call_regex.findall(model_output)
        logger.debug("model_output: %s", model_output)
        try:
            tool_calls: list[ToolCall] = []
            for match in matched_tool_calls:
                tc_detail = self.func_detail_regex.search(match)
                if not tc_detail:
                    logger.warning(
                        "Failed to parse tool call details from: %s",
                        match,
                    )
                    continue
                tc_name = tc_detail.group(1).strip()
                tc_args = tc_detail.group(2)
                pairs = self.func_arg_regex.findall(tc_args) if tc_args else []
                arg_dct: dict[str, Any] = {}
                for key, value in pairs:
                    arg_key = key.strip()
                    arg_val = value.strip()
                    if not self._is_string_type(tc_name, arg_key, request.tools):
                        arg_val = self._deserialize(arg_val)
                    logger.debug("arg_key = %s, arg_val = %s", arg_key, arg_val)
                    arg_dct[arg_key] = arg_val
                tool_calls.append(
                    ToolCall(
                        type="function",
                        function=FunctionCall(
                            name=tc_name,
                            arguments=json.dumps(arg_dct, ensure_ascii=False),
                        ),
                    )
                )
        except Exception:
            logger.exception("Failed to extract tool call spec")
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
        else:
            if len(tool_calls) > 0:
                content: str | None = model_output[
                    : model_output.find(self.tool_calls_start_token)
                ]
                # Normalize empty/whitespace-only content to None
                if not content or not content.strip():
                    content = None
                return ExtractedToolCallInformation(
                    tools_called=True, tool_calls=tool_calls, content=content
                )
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `PoolsideV1ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `PoolsideV1ToolParser.extract_tool_calls_streaming` (lines 229-430)
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
        if not self._tools_enabled(request):
            return DeltaMessage(content=delta_text) if delta_text else None

        self._buffer += delta_text

        pending_deltas: dict[int, DeltaToolCall] = {}
        content: str | None = None

        while True:
            if not self._in_tool_call:
                start_idx = self._buffer.find(self.tool_call_start_token)
                if start_idx == -1:
                    # Check for partial start token at end of buffer
                    for i in range(1, len(self.tool_call_start_token)):
                        if self._buffer.endswith(self.tool_call_start_token[:i]):
                            out = self._buffer[:-i]
                            self._buffer = self._buffer[-i:]
                            if out:
                                content = (content or "") + out
                            break
                    else:
                        out = self._buffer
                        self._buffer = ""
                        if out:
                            content = (content or "") + out
                    break

                if start_idx > 0:
                    content = (content or "") + self._buffer[:start_idx]
                    self._buffer = self._buffer[start_idx:]

                self._buffer = self._buffer[len(self.tool_call_start_token) :]
                self._begin_tool_call()
                continue

            # Parse tool name first
            if not self.current_tool_name_sent:
                nl = self._buffer.find("\n")
                ak = self._buffer.find(self.arg_key_start)
                end = self._buffer.find(self.tool_call_end_token)
# ... omitted for brevity / 为简洁省略 ...
                break
            key = self._buffer[len(self.arg_key_start) : key_end]
            self._buffer = self._buffer[key_end + len(self.arg_key_end) :]
            self._pending_key = key
            continue

        tool_calls = list(pending_deltas.values())
        if content is None and len(tool_calls) == 0:
            if request.logprobs:
                return DeltaMessage(content="")
            return None
        return DeltaMessage(content=content, tool_calls=tool_calls)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `PoolsideV1ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `PoolsideV1ToolParser._get_or_create_delta` (lines 478-487)
```python
    def _get_or_create_delta(self, pending: dict[int, DeltaToolCall]) -> DeltaToolCall:
        idx = self.current_tool_id
        if idx not in pending:
            pending[idx] = DeltaToolCall(
                index=idx,
                function=DeltaFunctionCall(),
            )
        delta = pending[idx]
        assert delta.function is not None
        return delta
```
**EN:** Provides the `_get_or_create_delta` helper used by this module to keep parsing logic modular and reusable.
**CN:** `PoolsideV1ToolParser._get_or_create_delta` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `PoolsideV1ToolParser._complete_json_prefix` (lines 505-527)
```python
    def _complete_json_prefix(
        json_prefix: str,
        allowed_partial_types: Allow,
    ) -> dict | None:
        """Complete a partial JSON prefix into a valid JSON object.

        Returns (formatted_prefix, parsed_dict) or None on failure.

        Note: ``partial_json_parser`` strips trailing whitespace before
        parsing (``complete.py:20``), which means the returned slice is
        shorter than ``json_prefix`` when it has trailing whitespace.
        Since the parser controls the construction of the json_prefix value,
        this code relies on it being a valid prefix and we only use the fix for
        the completion of the JSON object.
        """
        try:
            _, partial_str_completion = partial_json_parser.core.complete.fix(
                json_prefix,
                allowed_partial_types,
            )
            return json.loads(json_prefix + partial_str_completion)
        except Exception:
            return None
```
**EN:** Complete a partial JSON prefix into a valid JSON object.
**CN:** `PoolsideV1ToolParser._complete_json_prefix` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `PoolsideV1ToolParser._append_arg_fragment` (lines 544-571)
```python
    def _append_arg_fragment(
        self,
        *,
        key: str,
        raw_val: str,
    ) -> str | None:
        key = key.strip()
        if not key:
            return None
        if key in self._seen_keys[self.current_tool_id]:
            return None

        # This function is only called for non-string types (already checked
        # by _is_string_type in the caller), so we always deserialize.
        val_obj: Any = self._deserialize(raw_val)

        key_json = json.dumps(key, ensure_ascii=False)
        val_json = json.dumps(val_obj, ensure_ascii=False)

        if not self._args_started[self.current_tool_id]:
            fragment = "{" + key_json + ": " + val_json
            self._args_started[self.current_tool_id] = True
        else:
            fragment = ", " + key_json + ": " + val_json

        self._seen_keys[self.current_tool_id].add(key)
        self.streamed_args_for_tool[self.current_tool_id] += fragment
        return fragment
```
**EN:** Provides the `_append_arg_fragment` helper used by this module to keep parsing logic modular and reusable.
**CN:** `PoolsideV1ToolParser._append_arg_fragment` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Partial JSON recovery**: Partial JSON recovery. / 在 JSON 尚未闭合时尽量提取稳定的中间结果。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `ast`, `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `partial_json_parser.core.complete`, `regex`, `partial_json_parser.core.options`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
