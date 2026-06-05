# deepseekv32_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/deepseekv32_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the deepseekV32 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 deepseekV32 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
```python
import json
import uuid
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
from vllm.tool_parsers.utils import partial_tag_overlap
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `DeepSeekV32ToolParser` (lines 34-350)
```python
class DeepSeekV32ToolParser(ToolParser):
    """
    example tool call content:
    <｜DSML｜function_calls>
    <｜DSML｜invoke name="get_weather">
    <｜DSML｜parameter name="location" string="true">杭州</｜DSML｜parameter>
    <｜DSML｜parameter name="date" string="true">2024-01-16</｜DSML｜parameter>
    </｜DSML｜invoke>
    <｜DSML｜invoke name="get_weather">
    <｜DSML｜parameter name="location" string="true">北京</｜DSML｜parameter>
    <｜DSML｜parameter name="date" string="true">2024-01-16</｜DSML｜parameter>
    </｜DSML｜invoke>
    </｜DSML｜function_calls>
    """
```
**EN:** example tool call content: <｜DSML｜function_calls> <｜DSML｜invoke name="get_weather"> <｜DSML｜parameter name="location" string="true">杭州</｜DSML｜parameter> <｜DSML｜parameter name="date"
**CN:** 定义 `DeepSeekV32ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `DeepSeekV32ToolParser.__init__` (lines 52-84)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.prev_tool_call_arr: list[dict] = []

        # Streaming state
        self.current_tool_index: int = 0
        self._sent_content_idx: int = 0

        # Regex patterns for complete parsing
        self.tool_call_complete_regex = re.compile(
            re.escape(self.tool_call_start_token)
            + r"(.*?)"
            + re.escape(self.tool_call_end_token),
            re.DOTALL,
        )
        self.invoke_complete_regex = re.compile(
            r'<｜DSML｜invoke\s+name="([^"]+)"\s*>(.*?)</｜DSML｜invoke>', re.DOTALL
        )
        self.parameter_complete_regex = re.compile(
            r'<｜DSML｜parameter\s+name="([^"]+)"\s+string="(true|false)"\s*>(.*?)</｜DSML｜parameter>',
            re.DOTALL,
        )

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        logger.debug(
            "vLLM Successfully import tool parser %s !", self.__class__.__name__
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `DeepSeekV32ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `DeepSeekV32ToolParser.adjust_request` (lines 86-98)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # Ensure tool call tokens
            # (e.g. <｜DSML｜function_calls>, </｜DSML｜function_calls>)
            # are not skippedduring decoding.
            # Even though they are not marked as special tokens,
            # setting skip_special_tokens=False ensures proper handling in
            # transformers 5.x where decoding behavior may have changed.
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `DeepSeekV32ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `DeepSeekV32ToolParser._parse_invoke_params` (lines 104-110)
```python
    def _parse_invoke_params(self, invoke_str: str) -> dict[str, tuple[str, str]]:
        param_dict: dict[str, tuple[str, str]] = {}
        for param_name, string_attr, param_val in self.parameter_complete_regex.findall(
            invoke_str
        ):
            param_dict[param_name] = (param_val, string_attr)
        return param_dict
```
**EN:** Implements a focused helper routine for interpreting model output or locating parser-relevant structure.
**CN:** `DeepSeekV32ToolParser._parse_invoke_params` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `DeepSeekV32ToolParser._convert_param_value_checked` (lines 112-133)
```python
    def _convert_param_value_checked(self, value: str, param_type: str) -> Any:
        """Convert parameter value to the correct type."""
        if value.lower() == "null":
            return None

        param_type = param_type.lower()
        if param_type in ["string", "str", "text"]:
            return value
        elif param_type in ["integer", "int"]:
            return int(value)
        elif param_type in ["number", "float"]:
            val = float(value)
            return val if val != int(val) else int(val)
        elif param_type in ["boolean", "bool"]:
            value = value.strip()
            if value.lower() not in ["false", "0", "true", "1"]:
                raise ValueError("Invalid boolean value")
            return value.lower() in ["true", "1"]
        elif param_type in ["object", "array"]:
            return json.loads(value)
        else:
            return json.loads(value)
```
**EN:** Convert parameter value to the correct type.
**CN:** `DeepSeekV32ToolParser._convert_param_value_checked` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `DeepSeekV32ToolParser._repair_param_dict` (lines 148-167)
```python
    def _repair_param_dict(
        param_dict: dict[str, Any],
        param_config: dict[str, dict],
    ) -> dict[str, Any]:
        """Unwrap single 'arguments' / 'input' wrappers when the wrapper
        is not part of the requested tool schema and the wrapped object
        matches the schema fields."""
        allowed = set(param_config.keys())
        for wrapper in ("arguments", "input"):
            if set(param_dict.keys()) != {wrapper} or wrapper in allowed:
                continue
            inner = param_dict[wrapper]
            if isinstance(inner, str):
                try:
                    inner = json.loads(inner)
                except json.JSONDecodeError:
                    return param_dict
            if isinstance(inner, dict) and set(inner.keys()).issubset(allowed):
                return inner
        return param_dict
```
**EN:** Unwrap single 'arguments' / 'input' wrappers when the wrapper is not part of the requested tool schema and the wrapped object matches the schema fields.
**CN:** `DeepSeekV32ToolParser._repair_param_dict` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `DeepSeekV32ToolParser._convert_params_with_schema` (lines 169-198)
```python
    def _convert_params_with_schema(
        self,
        function_name: str,
        param_dict: dict[str, tuple[str, str]],
    ) -> dict[str, Any]:
        """Convert raw string param values using the tool schema types."""
        param_config: dict = {}
        if self.tools:
            for tool in self.tools:
                if (
                    hasattr(tool, "function")
                    and tool.function.name == function_name
                    and hasattr(tool.function, "parameters")
                ):
                    schema = tool.function.parameters
                    if isinstance(schema, dict) and "properties" in schema:
                        param_config = schema["properties"]
                    break

        converted: dict[str, Any] = {}
        for name, (value, string_attr) in param_dict.items():
            if string_attr == "true":
                converted[name] = value
                continue

            param_type = "string"
            if name in param_config and isinstance(param_config[name], dict):
                param_type = param_config[name].get("type", "string")
            converted[name] = self._convert_param_value(value, param_type)
        return self._repair_param_dict(converted, param_config)
```
**EN:** Convert raw string param values using the tool schema types.
**CN:** `DeepSeekV32ToolParser._convert_params_with_schema` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `DeepSeekV32ToolParser.extract_tool_calls` (lines 200-250)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        """Extract tool calls from complete model output (non-streaming)."""
        # Quick check
        if self.tool_call_start_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        try:
            tool_calls = []

            # Find all complete tool_call blocks
            for tool_call_match in self.tool_call_complete_regex.findall(model_output):
                # Find all invokes within this tool_call
                for invoke_name, invoke_content in self.invoke_complete_regex.findall(
                    tool_call_match
                ):
                    param_dict = self._parse_invoke_params(invoke_content)
                    params = self._convert_params_with_schema(invoke_name, param_dict)
                    tool_calls.append(
                        ToolCall(
                            type="function",
                            function=FunctionCall(
                                name=invoke_name,
                                arguments=json.dumps(params, ensure_ascii=False),
                            ),
                        )
                    )

            if not tool_calls:
                return ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=model_output
                )

            # Extract content before first tool call
            first_tool_idx = model_output.find(self.tool_call_start_token)
            content = model_output[:first_tool_idx] if first_tool_idx > 0 else None

            return ExtractedToolCallInformation(
                tools_called=True, tool_calls=tool_calls, content=content
            )

        except Exception:
            logger.exception("Error extracting tool calls")
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** Extract tool calls from complete model output (non-streaming).
**CN:** `DeepSeekV32ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `DeepSeekV32ToolParser._reset_streaming_state` (lines 252-257)
```python
    def _reset_streaming_state(self):
        """Reset all streaming state."""
        self.current_tool_index = 0
        self._sent_content_idx = 0
        self.prev_tool_call_arr.clear()
        self.streamed_args_for_tool.clear()
```
**EN:** Reset all streaming state.
**CN:** `DeepSeekV32ToolParser._reset_streaming_state` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `DeepSeekV32ToolParser._extract_delta_tool_calls` (lines 259-298)
```python
    def _extract_delta_tool_calls(
        self,
        current_text: str,
        request: ChatCompletionRequest | None,
    ) -> list[DeltaToolCall]:
        """Extract DeltaToolCalls from newly completed <invoke> blocks.

        Tracks progress via ``current_tool_index`` so each block is
        extracted exactly once across successive streaming calls.
        """
        complete_invokes = self.invoke_complete_regex.findall(current_text)
        delta_tool_calls: list[DeltaToolCall] = []

        while len(complete_invokes) > self.current_tool_index:
            invoke_name, invoke_body = complete_invokes[self.current_tool_index]
            param_dict = self._parse_invoke_params(invoke_body)

            converted = self._convert_params_with_schema(invoke_name, param_dict)
            args_json = json.dumps(converted, ensure_ascii=False)
            idx = self.current_tool_index
            self.current_tool_index += 1

            self.prev_tool_call_arr.append(
                {"name": invoke_name, "arguments": converted}
            )
            self.streamed_args_for_tool.append(args_json)

            delta_tool_calls.append(
                DeltaToolCall(
                    index=idx,
                    id=self._generate_tool_call_id(),
                    function=DeltaFunctionCall(
                        name=invoke_name,
                        arguments=args_json,
                    ),
                    type="function",
                )
            )

        return delta_tool_calls
```
**EN:** Extract DeltaToolCalls from newly completed <invoke> blocks.
**CN:** `DeepSeekV32ToolParser._extract_delta_tool_calls` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `DeepSeekV32ToolParser.extract_tool_calls_streaming` (lines 318-350)
```python
    def extract_tool_calls_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],  # pylint: disable=unused-argument
        current_token_ids: Sequence[int],  # pylint: disable=unused-argument
        delta_token_ids: Sequence[int],
        request: ChatCompletionRequest,
    ) -> DeltaMessage | None:
        """Extract tool calls from streaming model output.

        Uses a buffer-until-complete-invoke strategy: tokens are buffered
        until a complete invoke block is available, then parsed and emitted
        in one shot.
        """

        # First chunk of a new stream — reset state from prior request.
        if not previous_text:
            self._reset_streaming_state()

        content = self._extract_content(current_text)
        delta_tool_calls = self._extract_delta_tool_calls(current_text, request)

        if delta_tool_calls or content:
            return DeltaMessage(content=content, tool_calls=delta_tool_calls)

        # Empty delta with token ids means EOS or closing tag; return
        # non-None so the serving framework can finalize finish_reason.
        if not delta_text and delta_token_ids and self.prev_tool_call_arr:
            return DeltaMessage(content="")

        return None
```
**EN:** Extract tool calls from streaming model output.
**CN:** `DeepSeekV32ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `uuid`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
