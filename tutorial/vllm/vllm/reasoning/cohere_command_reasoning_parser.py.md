# cohere_command_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/cohere_command_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the cohere command reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 cohere command 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-36)
```python
from __future__ import annotations

import json
from collections.abc import Mapping, Sequence
from typing import Any, NamedTuple, TypedDict, TypeGuard

import regex as re
import xgrammar as xgr

try:
    from cohere_melody import PyFilter, PyFilterOptions
except ImportError as e:
    raise ImportError(
        "The Cohere reasoning parser requires the `cohere_melody` "
        "package, which is not installed. Install it with:\n"
        "    pip install cohere_melody"
    ) from e


from vllm.entrypoints.mcp.tool_server import ToolServer
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import (
    AnyResponseFormat,
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
)
from vllm.entrypoints.openai.responses.protocol import ResponsesRequest
from vllm.reasoning import ReasoningParser
from vllm.sampling_params import StructuredOutputsParams
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 38-38)
```python
REPLACEMENT_CHAR = "\ufffd"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `CohereTagRegistry` (lines 41-45)
```python
class CohereTagRegistry(NamedTuple):
    """A single ``structural_tag`` trigger / end pair (``begin`` uses ``trigger``)."""
```
**EN:** A single ``structural_tag`` trigger / end pair (``begin`` uses ``trigger``).
**CN:** 定义 `CohereTagRegistry` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Class `CohereTagStyle` (lines 48-56)
```python
class CohereTagStyle(NamedTuple):
    """The structural tags style for a given model architecture.

    ``json_tags`` lists every JSON-schema wrapper the model may emit (MOE uses
    both response and text delimiters). ``tools`` is the tool-call wrapper.
    """
```
**EN:** The structural tags style for a given model architecture.
**CN:** 定义 `CohereTagStyle` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Class `CohereNormalizedTool` (lines 59-67)
```python
class CohereNormalizedTool(TypedDict):
    """A tool definition normalized to the shape ``collect_tool_schema`` expects.

    ``parameters`` is a JSON Schema object (possibly empty) describing the tool's
    call signature.
    """
```
**EN:** A tool definition normalized to the shape ``collect_tool_schema`` expects.
**CN:** 定义 `CohereNormalizedTool` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Module constants/configuration 2 (lines 70-96)
```python
COMMAND_A_TOOLS_TAG = CohereTagRegistry(
    trigger="<|START_ACTION|>",
    end="<|END_ACTION|>",
)
COMMAND_A_JSON_TAG = CohereTagRegistry(
    trigger="<|START_RESPONSE|>",
    end="<|END_RESPONSE|>",
)
COMMAND_A_PLUS_JSON_TAG = CohereTagRegistry(
    trigger="<|START_TEXT|>",
    end="<|END_TEXT|>",
)

MODEL_TO_TAG_STYLE: dict[str, CohereTagStyle] = {
    "Cohere2ForCausalLM": CohereTagStyle(
        json_tags=(COMMAND_A_JSON_TAG,),
        tools=COMMAND_A_TOOLS_TAG,
    ),
    "Cohere2VisionForConditionalGeneration": CohereTagStyle(
        json_tags=(COMMAND_A_JSON_TAG, COMMAND_A_PLUS_JSON_TAG),
        tools=COMMAND_A_TOOLS_TAG,
    ),
    "Cohere2MoeForCausalLM": CohereTagStyle(
        json_tags=(COMMAND_A_JSON_TAG,),
        tools=COMMAND_A_TOOLS_TAG,
    ),
}
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `collect_tool_schema` (lines 99-144)
```python
def collect_tool_schema(tool_schema: list[CohereNormalizedTool]) -> str:
    """Build an xgrammar EBNF grammar that matches a JSON array of tool calls.

    The grammar shape is architecture-independent; callers are responsible for
    wrapping it in the correct structural tag (see ``CohereTagStyle.tools``).
    """
    tool_dictionary: dict[str, str] = {}
    for tool in tool_schema:
        tool_name = tool["name"]
        tool_parameters = json.dumps(tool["parameters"])
        json_schema = f"""{{
                        "type": "object",
                        "properties": {{
                            "tool_call_id": {{
                                "type": "string",
                                "pattern": "^[0-9]+$"
                            }},
                            "tool_name": {{
                                "type": "string",
                                "const": "{tool_name}"
                            }},
                            "parameters": {tool_parameters}
                            }}
                            }}"""
        tool_grammar = str(xgr.Grammar.from_json_schema(json_schema))
        for match in re.findall(r"\b(\w+)\s*::=", tool_grammar):
            tool_grammar = re.sub(
                rf"\b{re.escape(match)}\b", tool_name + match, tool_grammar
            )
        tool_dictionary[tool_name] = f"{tool_name} ::= {tool_name}root\n{tool_grammar}"
    # Emitted grammar shape:
    #   root  ::= tools
    #   tools ::= ws "[" ws tool ws ("," ws tool)* ws "]" ws
    #   ws    ::= (" " | "\t" | "\n")*
    #   tool  ::= <tool_a> | <tool_b> | ...         (one alternative per input)
    #   <tool_x>     ::= <tool_x>root               (per-tool xgrammar rules)
    #   <tool_x>root ::= ...                        (from xgr.Grammar.from_json_schema)
    tool_alternatives = "tool ::= " + " | ".join(tool_dictionary.keys())
    tool_rules = "\n    ".join(tool_dictionary.values())
    grammar = f"""root ::= tools
    tools ::= ws "[" ws tool ws ("," ws tool)*  ws "]" ws
    ws    ::= (" " | "\\t" | "\\n")*
    {tool_alternatives}
    {tool_rules}
    """
    return grammar
```
**EN:** Build an xgrammar EBNF grammar that matches a JSON array of tool calls.
**CN:** `collect_tool_schema` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_tool_definitions_to_schema_list` (lines 147-189)
```python
def _tool_definitions_to_schema_list(
    tools: str | list[Any],
) -> list[CohereNormalizedTool]:
    """
    Build the list of ``CohereNormalizedTool`` dicts expected by
    ``collect_tool_schema``.

    Accepts:
    - JSON string
    - list of dicts with top-level ``name`` / ``parameters``
    - list of Chat Completions-style ``{"type": "function", "function": {...}}``
    - list of Pydantic models with ``model_dump()``
    """
    if isinstance(tools, str):
        try:
            parsed = json.loads(tools)
        except json.JSONDecodeError:
            return []
        if not isinstance(parsed, list):
            return []
    else:
        parsed = list(tools)

    out: list[CohereNormalizedTool] = []
    for raw in parsed:
        t = raw.model_dump() if hasattr(raw, "model_dump") else raw
        if not isinstance(t, dict):
            continue
        # Unwrap Chat Completions' ``{"type": "function", "function": {...}}``
        # shape; otherwise take the dict as-is.
        if t.get("type") == "function" and isinstance(t.get("function"), dict):
            t = t["function"]
        name = t.get("name")
        if not isinstance(name, str):
            continue
        params = t.get("parameters")
        out.append(
            CohereNormalizedTool(
                name=name,
                parameters=params if isinstance(params, dict) else {},
            )
        )
    return out
```
**EN:** Build the list of ``CohereNormalizedTool`` dicts expected by ``collect_tool_schema``.
**CN:** `_tool_definitions_to_schema_list` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_has_effective_tools` (lines 192-206)
```python
def _has_effective_tools(
    tools: str | list[Any] | None,
) -> TypeGuard[str | list[Any]]:
    """
    True when ``tools`` contains at least one tool definition to convert.

    ``ResponsesRequest`` defaults ``tools`` to ``[]``; ``ChatCompletionRequest``
    uses ``None``. Both mean "no tools" here. Strings (e.g. a JSON blob) are
    treated as effective only when non-blank.
    """
    if tools is None:
        return False
    if isinstance(tools, str):
        return bool(tools.strip())
    return len(tools) > 0
```
**EN:** True when ``tools`` contains at least one tool definition to convert.
**CN:** `_has_effective_tools` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `convert_schema_to_structural_tags` (lines 212-266)
```python
def convert_schema_to_structural_tags(
    schema: dict | None = None,
    tools: str | list[Any] | None = None,
    model_architecture: str | None = None,
) -> str | None:
    """
    Returns a response_format string accepted by xgrammar's structural tag format.
    Uses the canonical shape: {"type": "structural_tag", "format": {...}} with
    format.type "triggered_tags" and tag content type "json_schema" or "grammar".

    Callers that are not on an engine path (e.g. the reasoning parser) must pass
    ``model_architecture`` explicitly.
    """
    if model_architecture is None or model_architecture not in MODEL_TO_TAG_STYLE:
        return None
    style = MODEL_TO_TAG_STYLE[model_architecture]

    tags: list[dict] = []
    triggers: list[str] = []

    def _add_tag(tag: CohereTagRegistry, content: dict) -> None:
        tags.append({"begin": tag.trigger, "content": content, "end": tag.end})
        triggers.append(tag.trigger)

    if schema is not None:
        # One structural tag per JSON wrapper (e.g. MOE: response + text).
        # Same for schema-only and "tools plus JSON mode" (North: schema when
        # the model does not call tools).
        for jt in style.json_tags:
            _add_tag(jt, {"type": "json_schema", "json_schema": schema})

    if _has_effective_tools(tools):
        # ``tools`` may be a JSON string (poseidon / RESPONSE_FORMAT_TOOL_DEFINITIONS)
        # or a list (Chat Completions ``request.tools`` as Pydantic models or dicts).
        tool_schema_list = _tool_definitions_to_schema_list(tools)
        if not tool_schema_list:
            raise ValueError(
                "No valid tool definitions could be parsed from the request for "
                "structural tag conversion."
            )
        tool_grammar = collect_tool_schema(tool_schema_list)
        _add_tag(style.tools, {"type": "grammar", "grammar": tool_grammar})

    if not tags:
        return None
    return json.dumps(
        {
            "type": "structural_tag",
            "format": {
                "type": "triggered_tags",
                "triggers": triggers,
                "tags": tags,
            },
        }
    )
```
**EN:** Returns a response_format string accepted by xgrammar's structural tag format.
**CN:** `convert_schema_to_structural_tags` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_maybe_parse_json_dict` (lines 280-290)
```python
def _maybe_parse_json_dict(value: Any) -> dict | None:
    """If value is a JSON string, parse to dict; otherwise require dict."""
    if isinstance(value, dict):
        return value
    if isinstance(value, str):
        try:
            parsed = json.loads(value)
        except (TypeError, json.JSONDecodeError):
            return None
        return parsed if isinstance(parsed, dict) else None
    return None
```
**EN:** If value is a JSON string, parse to dict; otherwise require dict.
**CN:** `_maybe_parse_json_dict` 函数实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Function `_schema_from_json_schema_field` (lines 305-342)
```python
def _schema_from_json_schema_field(js_wr: Any) -> dict | None:
    """
    Extract the JSON Schema object from Chat Completions ``json_schema`` payload.

    Accepts:
    - ``JsonSchemaResponseFormat`` (Pydantic) with ``schema`` / ``json_schema`` field
    - dict in OpenAI shape ``{"name": ..., "schema": {...}}``
    - dict with ``json_schema`` key holding either the schema or a nested wrapper
    - dict that is already a JSON Schema document (some clients omit the wrapper)
    - JSON strings for any of the above
    """
    if js_wr is None:
        return None

    parsed_wr = _maybe_parse_json_dict(js_wr)
    if parsed_wr is not None:
        js_wr = parsed_wr

    if hasattr(js_wr, "model_dump"):
        for by_alias in (True, False):
            try:
                data = js_wr.model_dump(by_alias=by_alias, exclude_none=False)
            except TypeError:
                data = js_wr.model_dump(by_alias=by_alias)
            out = _unwrap_nested_schema(data.get("schema") or data.get("json_schema"))
            if out is not None:
                return out
        inner_attr = getattr(js_wr, "json_schema", None)
        return inner_attr if isinstance(inner_attr, dict) else None

    if isinstance(js_wr, dict):
        for key in ("schema", "json_schema"):
            out = _unwrap_nested_schema(js_wr.get(key))
            if out is not None:
                return out
        return js_wr

    return None
```
**EN:** Extract the JSON Schema object from Chat Completions ``json_schema`` payload.
**CN:** `_schema_from_json_schema_field` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_schema_dict_from_chat_response_format` (lines 345-361)
```python
def _schema_dict_from_chat_response_format(
    rf: AnyResponseFormat | dict | None,
) -> dict | None:
    """JSON schema dict from Chat Completions ``request.response_format`` only."""
    if rf is None:
        return None
    rf_type = _response_format_type(rf)
    if rf_type == "json_object":
        return {"type": "object"}
    if rf_type != "json_schema":
        return None
    js_wr = (
        rf.get("json_schema")
        if isinstance(rf, dict)
        else getattr(rf, "json_schema", None)
    )
    return _schema_from_json_schema_field(js_wr)
```
**EN:** JSON schema dict from Chat Completions ``request.response_format`` only.
**CN:** `_schema_dict_from_chat_response_format` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `_schema_dict_from_structured_outputs` (lines 364-405)
```python
def _schema_dict_from_structured_outputs(
    so: StructuredOutputsParams | None,
) -> dict | None:
    """Schema dict from ``structured_outputs`` (``json`` / ``json_object``).

    Same unwrapping as ``json_schema``. ``json`` is expected to be ``str`` or
    ``dict`` (enforced by ``StructuredOutputsParams`` / request models); other
    types raise ``ValueError`` only if a caller bypasses that validation.
    """
    if so is None:
        return None
    if so.json_object:
        return {"type": "object"}
    raw: Any = so.json
    if raw is None:
        return None

    if hasattr(raw, "model_dump"):
        out = _schema_from_json_schema_field(raw)
        if out is None:
            raise ValueError(
                "structured_outputs.json model has no extractable JSON Schema."
            )
        return out

    if isinstance(raw, str):
        if not raw.strip():
            raise ValueError("structured_outputs.json cannot be empty.")
        try:
            raw = json.loads(raw)
        except json.JSONDecodeError as e:
            raise ValueError("structured_outputs.json must be valid JSON.") from e
        if not isinstance(raw, dict):
            raise ValueError("structured_outputs.json must decode to a JSON object.")

    if isinstance(raw, Mapping):
        body = raw if isinstance(raw, dict) else dict(raw)
        return _schema_from_json_schema_field(body) or body

    raise ValueError(
        f"structured_outputs.json has unsupported type {type(raw).__name__}."
    )
```
**EN:** Schema dict from ``structured_outputs`` (``json`` / ``json_object``).
**CN:** `_schema_dict_from_structured_outputs` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `BaseCohereCommandReasoningParser` (lines 408-543)
```python
class BaseCohereCommandReasoningParser(ReasoningParser):
    def __init__(
        self,
        tokenizer: TokenizerLike,
        *args,
        streaming_opts: PyFilterOptions,
        unary_opts: PyFilterOptions,
        **kwargs,
    ):
        super().__init__(tokenizer, *args, **kwargs)
        self.end_token_id = tokenizer.convert_tokens_to_ids("<|END_THINKING|>")
        self.unary_opts = unary_opts
        self.melody_unary = PyFilter(unary_opts)
        self.melody_streaming = PyFilter(streaming_opts)
```
**EN:** Declares `BaseCohereCommandReasoningParser`, the main parser class in this module. It extends ReasoningParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `BaseCohereCommandReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `BaseCohereCommandReasoningParser.__init__` (lines 409-421)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        *args,
        streaming_opts: PyFilterOptions,
        unary_opts: PyFilterOptions,
        **kwargs,
    ):
        super().__init__(tokenizer, *args, **kwargs)
        self.end_token_id = tokenizer.convert_tokens_to_ids("<|END_THINKING|>")
        self.unary_opts = unary_opts
        self.melody_unary = PyFilter(unary_opts)
        self.melody_streaming = PyFilter(streaming_opts)
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `BaseCohereCommandReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `BaseCohereCommandReasoningParser.reasoning_start_str` (lines 424-425)
```python
    def reasoning_start_str(self) -> str | None:
        return "<|START_THINKING|>"
```
**EN:** Provides the `reasoning_start_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `BaseCohereCommandReasoningParser.reasoning_start_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseCohereCommandReasoningParser.extract_reasoning_streaming` (lines 431-458)
```python
    def extract_reasoning_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
    ) -> DeltaMessage | None:
        r = self.melody_streaming.write_decoded(delta_text)
        if r.content is None and r.reasoning is None and not r.tool_calls:
            return None
        msg = DeltaMessage()
        if r.content is not None:
            msg.content = r.content
        if r.reasoning is not None:
            msg.reasoning = r.reasoning
        if r.tool_calls:
            msg.tool_calls = [
                DeltaToolCall(
                    id=tc.id,
                    index=tc.index,
                    type="function",
                    function=DeltaFunctionCall(name=tc.name, arguments=tc.arguments),
                )
                for tc in r.tool_calls
            ]
        return msg
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `BaseCohereCommandReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseCohereCommandReasoningParser.extract_reasoning` (lines 460-464)
```python
    def extract_reasoning(
        self, model_output: str, request: ChatCompletionRequest | ResponsesRequest
    ) -> tuple[str | None, str | None]:
        result = self.melody_unary.process_full_text(model_output)
        return result.reasoning, result.content
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `BaseCohereCommandReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseCohereCommandReasoningParser.extract_content_ids` (lines 466-479)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        token_buf: list[int] = []
        content_ids: list[int] = []
        content_filter = PyFilter(self.unary_opts)
        for t in input_ids:
            token_buf.append(t)
            s = self.model_tokenizer.decode(token_buf, skip_special_tokens=False)
            if s.endswith(REPLACEMENT_CHAR):
                continue
            r = content_filter.write_decoded(s)
            if r.content is not None:
                content_ids.extend(token_buf)
            token_buf = []
        return content_ids
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `BaseCohereCommandReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseCohereCommandReasoningParser.is_reasoning_end` (lines 481-482)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        return any(tid == self.end_token_id for tid in reversed(input_ids))
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `BaseCohereCommandReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `BaseCohereCommandReasoningParser.prepare_structured_tag` (lines 484-491)
```python
    def prepare_structured_tag(
        self, original_tag: str | None, tool_server: ToolServer | None
    ) -> str | None:
        # Responses API replaces ``structural_tag`` via the reasoning parser.
        # Default ``ReasoningParser.prepare_structured_tag`` returns None, which
        # would clear a Cohere tag produced in ``adjust_request`` and break
        # ``StructuredOutputsParams`` validation. Preserve the existing tag.
        return original_tag
```
**EN:** Provides the `prepare_structured_tag` helper used by this module to keep parsing logic modular and reusable.
**CN:** `BaseCohereCommandReasoningParser.prepare_structured_tag` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseCohereCommandReasoningParser.adjust_request` (lines 493-543)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        so = request.structured_outputs
        if so is not None and so.structural_tag:
            return request
        # Schema: prefer ``response_format`` (OpenAI Chat Completions), then
        # ``structured_outputs.json`` / ``json_object`` (vLLM direct). Tools stay
        # on ``request.tools``.
        rf = (
            request.response_format
            if isinstance(request, ChatCompletionRequest)
            else None
        )
        if rf is not None and _response_format_type(rf) == "structural_tag":
            return request
        model_architecture = (
            self._model_config.architecture if self._model_config is not None else None
        )
        tools = request.tools
        # ``response_format`` wins if both it and ``structured_outputs`` supply JSON.
        schema = _schema_dict_from_chat_response_format(rf)
        if schema is None:
            schema = _schema_dict_from_structured_outputs(so)
        if schema is None and not _has_effective_tools(tools):
            return request
        if model_architecture is None:
            return request
        result = convert_schema_to_structural_tags(
            schema=schema,
            tools=tools,
            model_architecture=model_architecture,
        )
        if result is None:
            # Unsupported architectures are not in ``MODEL_TO_TAG_STYLE``.
            raise ValueError(
                "Failed to build structural_tag guided decoding constraints from "
                "this request's JSON schema and/or tools. The configured model "
                f"architecture ({model_architecture!r}) does not support Cohere "
                "command structural tags, or the schema cannot be expressed in "
                "that format."
            )
        request.structured_outputs = StructuredOutputsParams(structural_tag=result)
        # Folded JSON constraints into ``structural_tag``; drop ``response_format``
        # when it was the source so ``to_sampling_params`` does not also set ``json`` /
        # ``json_object`` (mutually exclusive in ``StructuredOutputsParams``).
        if isinstance(request, ChatCompletionRequest) and rf is not None:
            rf_type = _response_format_type(rf)
            if rf_type in ("json_schema", "json_object"):
                request.response_format = None
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `BaseCohereCommandReasoningParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `CohereCommand3ReasoningParser` (lines 546-554)
```python
class CohereCommand3ReasoningParser(BaseCohereCommandReasoningParser):
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(
            tokenizer,
            *args,
            streaming_opts=PyFilterOptions().cmd3(),
            unary_opts=PyFilterOptions().cmd3().no_tools(),
            **kwargs,
        )
```
**EN:** Declares `CohereCommand3ReasoningParser`, the main parser class in this module. It extends BaseCohereCommandReasoningParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `CohereCommand3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `CohereCommand3ReasoningParser.__init__` (lines 547-554)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(
            tokenizer,
            *args,
            streaming_opts=PyFilterOptions().cmd3(),
            unary_opts=PyFilterOptions().cmd3().no_tools(),
            **kwargs,
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `CohereCommand3ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Class `CohereCommand4ReasoningParser` (lines 557-565)
```python
class CohereCommand4ReasoningParser(BaseCohereCommandReasoningParser):
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(
            tokenizer,
            *args,
            streaming_opts=PyFilterOptions().cmd4(),
            unary_opts=PyFilterOptions().cmd4().no_tools(),
            **kwargs,
        )
```
**EN:** Declares `CohereCommand4ReasoningParser`, the main parser class in this module. It extends BaseCohereCommandReasoningParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `CohereCommand4ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `CohereCommand4ReasoningParser.__init__` (lines 558-565)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(
            tokenizer,
            *args,
            streaming_opts=PyFilterOptions().cmd4(),
            unary_opts=PyFilterOptions().cmd4().no_tools(),
            **kwargs,
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `CohereCommand4ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Structured decoding constraints**: Structured decoding constraints. / 用结构标签约束模型输出，减少无效工具调用格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `__future__`, `regex`, `xgrammar`
- **Internal / 内部**: `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.reasoning`, `vllm.sampling_params`, `vllm.tokenizers`
