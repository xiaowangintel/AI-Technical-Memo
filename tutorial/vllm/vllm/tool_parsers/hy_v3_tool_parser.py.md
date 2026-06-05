# hy_v3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/hy_v3_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the hy V3 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 hy V3 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
```python
import ast
import json
from collections.abc import Sequence
from typing import Any

import regex as re

from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
    ChatCompletionToolsParam,
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
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `HYV3ToolParser` (lines 34-645)
```python
class HYV3ToolParser(ToolParser):
    _TYPE_ALIASES: dict[str, str] = {
        "str": "string",
        "text": "string",
        "varchar": "string",
        "char": "string",
        "enum": "string",
        "bool": "boolean",
        "binary": "boolean",
        "int": "integer",
        "float": "number",
        "double": "number",
        "list": "array",
        "dict": "object",
        "map": "object",
```
**EN:** Declares `HYV3ToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `HYV3ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `HYV3ToolParser._get_schema_options` (lines 101-117)
```python
    def _get_schema_options(arg_schema: dict) -> list[dict]:
        """Normalize any property schema into a list of sub-schemas.
        - has type (single type) → return [arg_schema]
        - anyOf  → return the anyOf list
        - oneOf  → return the oneOf list
        - fallback → [{"type": "string"}]

        Note: single ``type`` has the highest priority.
        """
        if "type" in arg_schema:
            return [arg_schema]
        if "anyOf" in arg_schema:
            return arg_schema["anyOf"]
        if "oneOf" in arg_schema:
            return arg_schema["oneOf"]

        return [{"type": "string"}]
```
**EN:** Normalize any property schema into a list of sub-schemas.
**CN:** `HYV3ToolParser._get_schema_options` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `HYV3ToolParser._parse_value` (lines 198-244)
```python
    def _parse_value(
        value: str,
        function_name: str,
        arg_key: str,
        tools: list[ChatCompletionToolsParam] | None,
    ) -> Any:
        """Unified argument value parser with anyOf/oneOf support.

        Fallthrough chain:
            bool → int → number(wildcard_number)
            → json.loads for array/object
            → string → _deserialize
        """
        arg_schema = HYV3ToolParser._get_arg_schema(function_name, arg_key, tools)
        types = HYV3ToolParser._get_types(arg_schema)

        # 1. Try bool
        if "boolean" in types:
            result_bool = HYV3ToolParser._try_parse_bool(value)
            if result_bool is not None:
                return result_bool

        # 2. Try int
        if "integer" in types:
            result_int = HYV3ToolParser._try_parse_int(value)
            if result_int is not None:
                return result_int

        # 3. Try number (wildcard_number: int if no '.'/e/E, float otherwise)
        if "number" in types:
            result_number = HYV3ToolParser._try_parse_wildcard_number(value)
            if result_number is not None:
                return result_number

        # 4. Try json.loads (covers array/object and other unlisted types)
        if types - {"string", "boolean", "integer", "number"}:
            try:
                return json.loads(value)
            except (json.JSONDecodeError, ValueError):
                pass

        # 5. String fallback
        if "string" in types:
            return value

        # 6. Final fallback
        return HYV3ToolParser._deserialize(value)
```
**EN:** Unified argument value parser with anyOf/oneOf support.
**CN:** `HYV3ToolParser._parse_value` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `HYV3ToolParser.__init__` (lines 246-314)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.current_tool_name_sent: bool = False
        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id: int = -1
        self.streamed_args_for_tool: list[
            str
        ] = []  # map what has been streamed for each tool so far to a list

        # Streaming state: send tool name first, then return arguments at once
        self._streaming_tool_name: str | None = None  # tool name being streamed

        # State fields for incremental argument streaming
        self._completed_args: dict = {}  # closed {key: parsed_value}
        self._current_arg_key: str | None = None  # key being collected
        self._current_arg_is_string: bool = False  # is current arg pure string?
        self._streamed_json_len: int = 0  # bytes of JSON already sent

        self.tool_calls_start_token: str = "<tool_calls>"
        self.tool_calls_end_token: str = "</tool_calls>"

        self.tool_call_start_token: str = "<tool_call>"
        self.tool_call_end_token: str = "</tool_call>"

        self.tool_sep_token: str = "<tool_sep>"

        self.arg_key_start_token: str = "<arg_key>"
        self.arg_key_end_token: str = "</arg_key>"

# ... omitted for brevity / 为简洁省略 ...
        self.tool_call_start_token_id = self.vocab.get(self.tool_call_start_token)
        self.tool_call_end_token_id = self.vocab.get(self.tool_call_end_token)
        self._buffer = ""

        if (
            self.tool_calls_start_token_id is None
            or self.tool_calls_end_token_id is None
        ):
            raise RuntimeError(
                "HYV3 Tool parser could not locate tool call "
                "start/end tokens in the tokenizer!"
            )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `HYV3ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `HYV3ToolParser._extract_tool_calls` (lines 316-359)
```python
    def _extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> list[ToolCall]:
        try:
            function_call_tuples = []
            # start_token{name}sep_token{args}end_token...
            function_calls = self.tool_call_regex.findall(model_output)
            if function_calls:
                function_call_tuples.extend(function_calls)
                remaining = model_output.split(self.tool_call_end_token)[-1]
                function_calls = self.tool_call_portion_regex.findall(remaining)
                function_call_tuples += function_calls
            else:
                function_calls = self.tool_call_portion_regex.findall(model_output)
                if function_calls:
                    function_call_tuples.extend(function_calls)
            tool_calls = []
            for match in function_call_tuples:
                function_name, function_args = match
                function_name = function_name.strip()
                function_args = function_args.strip()

                arg_pairs = self.func_args_regex.findall(function_args)
                arg_dict = {}
                for key, value in arg_pairs:
                    parsed_value = HYV3ToolParser._parse_value(
                        value, function_name, key, request.tools
                    )
                    arg_dict[key] = parsed_value
                tool_calls.append(
                    ToolCall(
                        type="function",
                        function=FunctionCall(
                            name=function_name,
                            arguments=json.dumps(arg_dict, ensure_ascii=False),
                        ),
                    )
                )
            return tool_calls
        except Exception:
            logger.exception("Error in extracting tool call from response.")
            return []
```
**EN:** Provides the `_extract_tool_calls` helper used by this module to keep parsing logic modular and reusable.
**CN:** `HYV3ToolParser._extract_tool_calls` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `HYV3ToolParser.extract_tool_calls` (lines 361-387)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        # sanity check; avoid unnecessary processing
        if self.tool_calls_start_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
        else:
            try:
                tool_calls = self._extract_tool_calls(model_output, request)

                s_index = model_output.find(self.tool_calls_start_token)
                content = model_output[:s_index] if s_index != -1 else model_output
                return ExtractedToolCallInformation(
                    tools_called=True,
                    tool_calls=tool_calls,
                    content=content if content else None,
                )

            except Exception:
                logger.exception("Error in extracting tool call from response.")
                return ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=model_output
                )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HYV3ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HYV3ToolParser._reset_streaming_tool_state` (lines 389-395)
```python
    def _reset_streaming_tool_state(self):
        """Reset the streaming state for a single tool call."""
        self._streaming_tool_name = None
        self._completed_args = {}
        self._current_arg_key = None
        self._current_arg_is_string = False
        self._streamed_json_len = 0
```
**EN:** Reset the streaming state for a single tool call.
**CN:** `HYV3ToolParser._reset_streaming_tool_state` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `HYV3ToolParser.extract_tool_calls_streaming` (lines 397-476)
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
        # Check whether current tokens contain the tool_calls start token
        if self.tool_calls_start_token_id not in current_token_ids:
            return DeltaMessage(content=delta_text)

        # Encountered tool_calls start tag; extract preceding content and buffer
        if self.tool_calls_start_token in delta_text:
            text_parts = delta_text.split(self.tool_calls_start_token)
            self._buffer += text_parts[-1]
            if text_parts[0]:
                return DeltaMessage(content=text_parts[0])
            # Don't return None; continue processing buffer for complete content
        else:
            self._buffer += delta_text

        # Encountered finish, extract valid arguments
        if (
            current_text.find(self.tool_call_end_token + self.tool_calls_end_token)
            != -1
            and self._buffer.find(self.tool_call_end_token) == -1
        ):
# ... omitted for brevity / 为简洁省略 ...
                        ),
                    )
                ]
            )

            # Check if buffer already has complete arguments (all-in-one-delta)
            if self.tool_call_end_token not in self._buffer:
                return name_delta
            # Buffer already has a complete tool call; continue to phase 2 below

        # === Phase 2: Incremental argument streaming ===
        return self._extract_streaming_incremental(name_delta, request)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HYV3ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HYV3ToolParser._extract_streaming_incremental` (lines 489-645)
```python
    def _extract_streaming_incremental(
        self,
        name_delta: DeltaMessage | None,
        request: ChatCompletionRequest,
    ) -> DeltaMessage | None:
        """Incremental phase-2: scan tags in buffer, emit JSON diffs.

        Strategy:
        - Track completed args and emit each one as a JSON fragment.
        - For string-typed args, stream the value character-by-character.
        - Withhold the closing ``}`` until ``</tool_call>`` is seen.

        We build JSON manually via fragments rather than using json.dumps
        with a cursor, because json.dumps of partial-vs-full string values
        produces incompatible prefixes (e.g. ``""}`` vs ``"Hello"}``).
        """
        buf = self._buffer
        is_complete = self.tool_call_end_token in buf

        if is_complete:
            end_idx = buf.find(self.tool_call_end_token)
            args_text = buf[:end_idx]
            remaining = buf[end_idx + len(self.tool_call_end_token) :]
        else:
            args_text = buf
            remaining = ""

        # --- scan all fully closed kv pairs ---
        arg_pairs = self.func_args_regex.findall(args_text)
        for key, value in arg_pairs:
# ... omitted for brevity / 为简洁省略 ...
                            name=nd_func.name if nd_func else None,
                            arguments=argument_diff,
                        ),
                    )
                ]
            )
        elif name_delta is not None:
            return name_delta
        elif argument_diff:
            return self._make_args_delta(argument_diff)
        else:
            return None
```
**EN:** Incremental phase-2: scan tags in buffer, emit JSON diffs.
**CN:** `HYV3ToolParser._extract_streaming_incremental` 方法处理增量式流输出，只返回客户端应该看到的新差量。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `ast`, `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
