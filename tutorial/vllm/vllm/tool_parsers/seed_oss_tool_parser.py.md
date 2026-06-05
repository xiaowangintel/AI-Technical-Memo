# seed_oss_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/seed_oss_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the seed oss tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 seed oss 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 6-30)
```python
import ast
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
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 32-32)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `SeedOssToolParser` (lines 35-746)
```python
class SeedOssToolParser(ToolParser):
    TOOL_CALL_START = "<seed:tool_call>"
```
**EN:** Declares `SeedOssToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `SeedOssToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `SeedOssToolParser.__init__` (lines 39-89)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # --- streaming state ---
        self._reset_streaming_state()
        self.prev_tool_call_arr: list[dict] = []

        self.tool_call_start_token: str = self.TOOL_CALL_START
        self.tool_call_end_token: str = self.TOOL_CALL_END
        # Sentinel tokens for streaming mode
        self.tool_call_prefix: str = "<function="
        self.function_end_token: str = "</function>"
        self.parameter_prefix: str = "<parameter="
        self.parameter_end_token: str = "</parameter>"
        self.think_start_token: str = "<seed:think>"
        self.think_end_token: str = "</seed:think>"
        self.is_tool_call_started: bool = False
        self.is_thinking_end: bool = False
        self.failed_count: int = 0
        self._reset_streaming_state()

        self.tool_call_start_token_id = self.vocab.get(self.tool_call_start_token)
        self.tool_call_end_token_id = self.vocab.get(self.tool_call_end_token)
        self.think_end_token_id = self.vocab.get(self.think_end_token)

        if self.tool_call_start_token_id is None or self.tool_call_end_token_id is None:
            raise RuntimeError(
                "Seed_Oss XML parser: tokenizer did not include "
                "<seed:tool_call> or its closing tag."
            )
# ... omitted for brevity / 为简洁省略 ...
        )

        self.tool_call_function_regex = re.compile(
            r"<function=(.*?)</function>|<function=(.*)$", re.DOTALL
        )
        self.tool_call_parameter_regex = re.compile(
            r"<parameter=(.*?)</parameter>|<parameter=(.*?)$", re.DOTALL
        )

        logger.info(
            "vLLM Seed-Oss XML tool parser loaded (%s).", self.__class__.__name__
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `SeedOssToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `SeedOssToolParser._generate_tool_call_id` (lines 91-93)
```python
    def _generate_tool_call_id(self) -> str:
        """Generate a unique tool call ID."""
        return f"call_{uuid.uuid4().hex[:24]}"
```
**EN:** Generate a unique tool call ID.
**CN:** `SeedOssToolParser._generate_tool_call_id` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `SeedOssToolParser._reset_streaming_state` (lines 95-109)
```python
    def _reset_streaming_state(self):
        """Reset all streaming state."""
        self.current_tool_index = 0
        self.is_tool_call_started = False
        self.header_sent = False
        self.current_tool_id = -1
        self.current_function_name = None
        self.current_param_name = None
        self.current_param_value = ""
        self.param_count = 0
        self.in_param = False
        self.in_function = False
        self.accumulated_text = ""
        self.json_started = False
        self.json_closed = False
```
**EN:** Reset all streaming state.
**CN:** `SeedOssToolParser._reset_streaming_state` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `SeedOssToolParser._parse_xml_function_call` (lines 111-258)
```python
    def _parse_xml_function_call(
        self, function_call_str: str, tools: list[Tool] | None
    ) -> ToolCall | None:
        def get_arguments_config(func_name: str) -> dict:
            if tools is None:
                return {}
            for config in tools:
                if not hasattr(config, "type") or not (
                    hasattr(config, "function") and hasattr(config.function, "name")
                ):
                    continue
                if config.type == "function" and config.function.name == func_name:
                    if not hasattr(config.function, "parameters"):
                        return {}
                    params = config.function.parameters
                    if isinstance(params, dict) and "properties" in params:
                        return params["properties"]
                    elif isinstance(params, dict):
                        return params
                    else:
                        return {}
            logger.warning("Tool '%s' is not defined in the tools list.", func_name)
            return {}

        def convert_param_value(
            param_value: str, param_name: str, param_config: dict, func_name: str
        ) -> Any:
            # Handle null value for any type
            if param_value.lower() == "null":
                return None
# ... omitted for brevity / 为简洁省略 ...
            if param_value.endswith("\n"):
                param_value = param_value[:-1]

            param_dict[param_name] = convert_param_value(
                param_value, param_name, param_config, function_name
            )
        return ToolCall(
            type="function",
            function=FunctionCall(
                name=function_name, arguments=json.dumps(param_dict, ensure_ascii=False)
            ),
        )
```
**EN:** Implements a focused helper routine for interpreting model output or locating parser-relevant structure.
**CN:** `SeedOssToolParser._parse_xml_function_call` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `SeedOssToolParser._get_function_calls` (lines 260-278)
```python
    def _get_function_calls(self, model_output: str) -> list[str]:
        # Find all tool calls
        matched_ranges = self.tool_call_regex.findall(model_output)
        raw_tool_calls = [
            match[0] if match[0] else match[1] for match in matched_ranges
        ]

        # Back-off strategy if no tool_call tags found
        if len(raw_tool_calls) == 0:
            raw_tool_calls = [model_output]

        raw_function_calls = []
        for tool_call in raw_tool_calls:
            raw_function_calls.extend(self.tool_call_function_regex.findall(tool_call))

        function_calls = [
            match[0] if match[0] else match[1] for match in raw_function_calls
        ]
        return function_calls
```
**EN:** Provides the `_get_function_calls` helper used by this module to keep parsing logic modular and reusable.
**CN:** `SeedOssToolParser._get_function_calls` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `SeedOssToolParser.extract_tool_calls` (lines 280-349)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        # Quick check to avoid unnecessary processing
        if self.tool_call_prefix not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        # Check if both think start and end tokens are present
        if (
            self.think_start_token in model_output
            and self.think_end_token in model_output
        ):
            # Find the position of think end token
            think_end_index = model_output.find(self.think_end_token) + len(
                self.think_end_token
            )
            # Extract content after think end token
            result_content = model_output[think_end_index:]
            thinking_content = model_output[:think_end_index]
        else:
            thinking_content = ""
            result_content = model_output

        try:
            function_calls = self._get_function_calls(result_content)
            if len(function_calls) == 0:
# ... omitted for brevity / 为简洁省略 ...

            return ExtractedToolCallInformation(
                tools_called=(len(tool_calls) > 0),
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
**CN:** `SeedOssToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `SeedOssToolParser.extract_tool_calls_streaming` (lines 351-746)
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
        # If no delta text, return None unless
        # it's an EOS token after tool calls
        if not delta_text:
            # Check if this is an EOS token after all tool calls are complete
            # We check for tool calls in the text even if is_tool_call_started
            # is False because it might have been reset after processing all tools
            if delta_token_ids and self.tool_call_end_token_id not in delta_token_ids:
                # Count complete tool calls
                complete_calls = len(
                    self.tool_call_complete_regex.findall(current_text)
                )

                # If we have completed tool calls and populated prev_tool_call_arr
                if complete_calls > 0 and len(self.prev_tool_call_arr) > 0:
                    # Check if all tool calls are closed
                    open_calls = current_text.count(
                        self.tool_call_start_token
                    ) - current_text.count(self.tool_call_end_token)
                    if open_calls == 0:
                        # Return empty delta message to allow finish_reason processing
# ... omitted for brevity / 为简洁省略 ...
                            return DeltaMessage(
                                tool_calls=[
                                    DeltaToolCall(
                                        index=self.current_tool_index,
                                        function=DeltaFunctionCall(
                                            arguments=delta_escaped
                                        ),
                                    )
                                ]
                            )

        return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `SeedOssToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `ast`, `json`, `uuid`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
