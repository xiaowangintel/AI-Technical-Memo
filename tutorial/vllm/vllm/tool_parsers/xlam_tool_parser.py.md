# xlam_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/xlam_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the xlam tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 xlam 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-28)
```python
import json
from collections.abc import Sequence
from typing import Any, Optional, Union

import regex as re

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.engine.protocol import (
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
    ExtractedToolCallInformation,
    FunctionCall,
    ToolCall,
)
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.utils import random_uuid
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 30-30)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `xLAMToolParser` (lines 33-559)
```python
class xLAMToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # Initialize state for streaming mode
        self.prev_tool_calls: list[dict] = []
        self.current_tool_id = -1
        self.current_tool_name_sent = False
        self.streamed_args: list[str] = []  # Track arguments sent for each tool

        # For backward compatibility with tests
        self.current_tools_sent: list[bool] = []

        # For backward compatibility with serving code
        self.prev_tool_call_arr = []
```
**EN:** Declares `xLAMToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `xLAMToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `xLAMToolParser.__init__` (lines 34-62)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # Initialize state for streaming mode
        self.prev_tool_calls: list[dict] = []
        self.current_tool_id = -1
        self.current_tool_name_sent = False
        self.streamed_args: list[str] = []  # Track arguments sent for each tool

        # For backward compatibility with tests
        self.current_tools_sent: list[bool] = []

        # For backward compatibility with serving code
        self.prev_tool_call_arr = []

        # Regex patterns for preprocessing
        self.json_code_block_patterns = [
            r"```(?:json)?\s*([\s\S]*?)```",
            r"\[TOOL_CALLS\]([\s\S]*?)(?=\n|$)",
            r"<tool_call>([\s\S]*?)</tool_call>",
        ]
        self.thinking_tag_pattern = r"</think>([\s\S]*)"

        # Define streaming state type to be initialized later
        self.streaming_state: dict[str, Any] = {
            "current_tool_index": -1,
            "tool_ids": [],
            "sent_tools": [],
        }
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `xLAMToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `xLAMToolParser.preprocess_model_output` (lines 64-122)
```python
    def preprocess_model_output(
        self, model_output: str
    ) -> tuple[Optional[str], Optional[str]]:
        """
        Preprocess the model output to extract content and potential tool calls.
        Returns:
            Tuple of (content, potential_tool_calls_json)
        """
        # Check for thinking tag
        thinking_match = re.search(self.thinking_tag_pattern, model_output)
        if thinking_match:
            content = model_output[: thinking_match.start() + len("</think>")].strip()
            thinking_content = thinking_match.group(1).strip()

            # Try to parse the thinking content as JSON
            try:
                json.loads(thinking_content)
                return content, thinking_content
            except json.JSONDecodeError:
                # If can't parse as JSON, look for JSON code blocks
                for json_pattern in self.json_code_block_patterns:
                    json_matches = re.findall(json_pattern, thinking_content)
                    if json_matches:
                        for json_str in json_matches:
                            try:
                                json.loads(json_str)
                                return content, json_str
                            except json.JSONDecodeError:
                                continue

        # Check for JSON code blocks in the entire output
        for json_pattern in self.json_code_block_patterns:
            json_matches = re.findall(json_pattern, model_output)
            if json_matches:
                for json_str in json_matches:
                    try:
                        json.loads(json_str)
                        # Extract content by removing the JSON code block
                        content = re.sub(json_pattern, "", model_output).strip()
                        return content, json_str
                    except json.JSONDecodeError:
                        continue

        # If the entire output is a valid JSON array or looks like one, treat it as tool calls
        if model_output.strip().startswith("["):
            try:
                json.loads(model_output)
                return None, model_output
            except json.JSONDecodeError:
                # Even if it's not valid JSON yet, it might be a tool call in progress
                if (
                    "{" in model_output
                    and "name" in model_output
                    and "arguments" in model_output
                ):
                    return None, model_output

        # If no tool calls found, return the original output as content
        return model_output, None
```
**EN:** Preprocess the model output to extract content and potential tool calls.
**CN:** `xLAMToolParser.preprocess_model_output` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `xLAMToolParser.extract_tool_calls` (lines 124-186)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        """
        Extract tool calls from a complete model output.
        """
        try:
            # Preprocess the model output
            content, potential_tool_calls = self.preprocess_model_output(model_output)

            if not potential_tool_calls:
                return ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=content
                )

            # Parse the potential tool calls as JSON
            tool_calls_data = json.loads(potential_tool_calls)

            # Ensure it's an array
            if not isinstance(tool_calls_data, list):
                logger.debug("Tool calls data is not an array")
                return ExtractedToolCallInformation(
                    tools_called=False,
                    tool_calls=[],
                    content=content or model_output,
                )

            tool_calls: list[ToolCall] = []

            for idx, call in enumerate(tool_calls_data):
                if (
                    not isinstance(call, dict)
                    or "name" not in call
                    or "arguments" not in call
                ):
                    logger.debug("Invalid tool call format at index %d", idx)
                    continue

                tool_call = ToolCall(
                    id=f"call_{idx}_{random_uuid()}",
                    type="function",
                    function=FunctionCall(
                        name=call["name"],
                        arguments=(
                            json.dumps(call["arguments"])
                            if isinstance(call["arguments"], dict)
                            else call["arguments"]
                        ),
                    ),
                )
                tool_calls.append(tool_call)

            return ExtractedToolCallInformation(
                tools_called=len(tool_calls) > 0,
                tool_calls=tool_calls,
                content=content,
            )

        except Exception as e:
            logger.exception("Error extracting tool calls: %s", str(e))
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** Extract tool calls from a complete model output.
**CN:** `xLAMToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `xLAMToolParser.extract_tool_calls_streaming` (lines 188-559)
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
    ) -> Union[DeltaMessage, None]:
        """
        Extract tool calls for streaming mode.
        """
        # First, check for a definitive start of a tool call block.
        # This prevents premature parsing of incomplete output.
        stripped_text = current_text.strip()
        preprocessed_content, preprocessed_tool_calls = self.preprocess_model_output(
            current_text
        )

        # For JSON code blocks, we need to detect them earlier, even if incomplete
        has_potential_json_block = (
            "```json" in current_text
            or "```\n[" in current_text
            or "[TOOL_CALLS]" in current_text
            or "<tool_call>" in current_text
        )

        is_tool_call_block = (
            stripped_text.startswith("[")
            or stripped_text.startswith("<tool_call>")
            or stripped_text.startswith("[TOOL_CALLS]")
            or
            # Check if we have thinking tags with JSON-like content following
            ("</think>[" in current_text)
            or
            # Check if the text contains a JSON array after preprocessing
            preprocessed_tool_calls is not None
            or
            # For JSON code blocks, detect early if we see enough structure
            (
                has_potential_json_block
                and '"name"' in current_text
                and '"arguments"' in current_text
            )
        )

        if not is_tool_call_block:
            return DeltaMessage(content=delta_text)

# ... omitted for brevity / 为简洁省略 ...
                            self.streaming_state["current_tool_index"] += 1
                            self.current_tool_id = self.streaming_state[
                                "current_tool_index"
                            ]  # For compatibility

            # If we got here, we couldn't determine what to stream next
            return None

        except Exception as e:
            logger.exception(f"Error in streaming tool calls: {e}")
            # If we encounter an error, just return the delta text as regular content
            return DeltaMessage(content=delta_text)
```
**EN:** Extract tool calls for streaming mode.
**CN:** `xLAMToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.logger`, `vllm.tokenizers`, `vllm.utils`
