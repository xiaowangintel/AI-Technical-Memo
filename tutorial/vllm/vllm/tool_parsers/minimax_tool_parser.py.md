# minimax_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/minimax_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the minimax tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 minimax 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-28)
```python
import json
from collections.abc import Sequence
from typing import Any

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
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import extract_intermediate_diff
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 30-30)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `MinimaxToolParser` (lines 33-852)
```python
class MinimaxToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # Initialize streaming state for tracking tool call progress
        self.streaming_state: dict[str, Any] = {
            "current_tool_index": -1,  # Index of current tool being processed
            "tool_ids": [],  # List of tool call IDs
            "sent_tools": [],  # List of tools that have been sent
        }

        # Define tool call tokens and patterns
        self.tool_call_start_token = "<tool_calls>"
        self.tool_call_end_token = "</tool_calls>"
        self.tool_call_regex = re.compile(
```
**EN:** Declares `MinimaxToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `MinimaxToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `MinimaxToolParser.__init__` (lines 34-72)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        # Initialize streaming state for tracking tool call progress
        self.streaming_state: dict[str, Any] = {
            "current_tool_index": -1,  # Index of current tool being processed
            "tool_ids": [],  # List of tool call IDs
            "sent_tools": [],  # List of tools that have been sent
        }

        # Define tool call tokens and patterns
        self.tool_call_start_token = "<tool_calls>"
        self.tool_call_end_token = "</tool_calls>"
        self.tool_call_regex = re.compile(
            r"<tool_calls>(.*?)</tool_calls>|<tool_calls>(.*)", re.DOTALL
        )
        self.thinking_tag_pattern = r"<think>(.*?)</think>"
        self.tool_name_pattern = re.compile(r'"name":\s*"([^"]+)"')
        self.tool_args_pattern = re.compile(r'"arguments":\s*')

        # Buffer for handling partial tool calls during streaming
        self.pending_buffer = ""
        self.in_thinking_tag = False

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        # Get token IDs for tool call start/end tokens
        self.tool_call_start_token_id = self.vocab.get(self.tool_call_start_token)
        self.tool_call_end_token_id = self.vocab.get(self.tool_call_end_token)

        if self.tool_call_start_token_id is None or self.tool_call_end_token_id is None:
            logger.warning(
                "Minimax Tool parser could not locate tool call start/end "
                "tokens in the tokenizer. Falling back to string matching."
            )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `MinimaxToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `MinimaxToolParser.extract_tool_calls` (lines 151-237)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        """
        Extract tool calls from model output for non-streaming mode.

        Args:
            model_output: Complete model output
            request: Chat completion request

        Returns:
            ExtractedToolCallInformation containing tool calls and content
        """
        processed_output = self.preprocess_model_output(model_output)

        if self.tool_call_start_token not in processed_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        try:
            function_call_tuples = self.tool_call_regex.findall(processed_output)

            raw_function_calls = []
            for match in function_call_tuples:
                tool_call_content = match[0] if match[0] else match[1]
                if tool_call_content.strip():
                    lines = tool_call_content.strip().split("\n")
# ... omitted for brevity / 为简洁省略 ...
                tools_called=len(tool_calls) > 0,
                tool_calls=tool_calls,
                content=content.strip() if content.strip() else None,
            )

        except Exception:
            logger.exception(
                "An unexpected error occurred during tool call extraction."
            )
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** Extract tool calls from model output for non-streaming mode.
**CN:** `MinimaxToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MinimaxToolParser._reset_streaming_state` (lines 348-354)
```python
    def _reset_streaming_state(self) -> None:
        """Reset the streaming state to initial values."""
        self.streaming_state = {
            "current_tool_index": -1,
            "tool_ids": [],
            "sent_tools": [],
        }
```
**EN:** Reset the streaming state to initial values.
**CN:** `MinimaxToolParser._reset_streaming_state` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `MinimaxToolParser._find_tool_boundaries` (lines 437-479)
```python
    def _find_tool_boundaries(self, text: str) -> list[tuple[int, int]]:
        """
        Find the boundaries of tool calls in text.

        Args:
            text: Text to analyze

        Returns:
            List of (start, end) positions for tool calls
        """
        boundaries = []
        i = 0
        while i < len(text):
            if text[i] == "{":
                start = i
                depth = 0
                has_name = False
                has_arguments = False

                while i < len(text):
                    if text[i] == "{":
                        depth += 1
                    elif text[i] == "}":
                        depth -= 1
                        if depth == 0:
                            end = i + 1
                            segment = text[start:end]
                            if '"name"' in segment and '"arguments"' in segment:
                                boundaries.append((start, end))
                            break

                    if not has_name and '"name"' in text[start : i + 1]:
                        has_name = True
                    if not has_arguments and '"arguments"' in text[start : i + 1]:
                        has_arguments = True

                    i += 1

                if depth > 0 and has_name:
                    boundaries.append((start, i))
            else:
                i += 1
        return boundaries
```
**EN:** Find the boundaries of tool calls in text.
**CN:** `MinimaxToolParser._find_tool_boundaries` 方法实现针对性的辅助逻辑，用于解释模型输出或定位解析相关结构。

### Method `MinimaxToolParser._get_current_tool_content` (lines 511-545)
```python
    def _get_current_tool_content(
        self, text: str, tool_index: int
    ) -> tuple[str | None, str | None]:
        """
        Get the content of a specific tool by index.

        Args:
            text: Text containing tool calls
            tool_index: Index of tool to extract

        Returns:
            Tuple of (tool_name, tool_arguments) or (None, None) if not found
        """
        boundaries = self._find_tool_boundaries(text)

        if tool_index >= len(boundaries):
            return None, None

        start, end = boundaries[tool_index]
        tool_content = text[start:end]

        name_match = self.tool_name_pattern.search(tool_content)
        name = name_match.group(1) if name_match else None

        args_match = self.tool_args_pattern.search(tool_content)
        if args_match:
            try:
                args_text = self._extract_tool_args(tool_content, args_match)
                return name, args_text
            except Exception:
                remaining_content = tool_content[args_match.end() :]
                args_text = remaining_content.rstrip("}").strip()
                return name, args_text

        return name, None
```
**EN:** Get the content of a specific tool by index.
**CN:** `MinimaxToolParser._get_current_tool_content` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `MinimaxToolParser._handle_tool_name_streaming` (lines 547-595)
```python
    def _handle_tool_name_streaming(
        self, tool_content: str, tool_count: int
    ) -> DeltaMessage | None:
        """
        Handle streaming of tool names.

        Args:
            tool_content: Content containing tool calls
            tool_count: Total number of tools

        Returns:
            DeltaMessage with tool name or None if no tool to stream
        """
        next_idx = self._get_next_unsent_tool_index(tool_count)

        if next_idx == -1:
            return None

        boundaries = self._find_tool_boundaries(tool_content)
        if next_idx >= len(boundaries):
            return None

        tool_name, _ = self._get_current_tool_content(tool_content, next_idx)
        if not tool_name:
            return None

        self._set_current_tool_index(next_idx)
        sent_tools = list(self.streaming_state["sent_tools"])
        tool_ids = list(self.streaming_state["tool_ids"])

        tool_id = sent_tools[next_idx]["id"]
        tool_ids[next_idx] = tool_id
        sent_tools[next_idx]["sent_name"] = True

        self.streaming_state["sent_tools"] = sent_tools
        self.streaming_state["tool_ids"] = tool_ids

        return DeltaMessage(
            tool_calls=[
                DeltaToolCall(
                    index=next_idx,
                    type="function",
                    id=tool_id,
                    function=DeltaFunctionCall(name=tool_name).model_dump(
                        exclude_none=True
                    ),
                )
            ]
        )
```
**EN:** Handle streaming of tool names.
**CN:** `MinimaxToolParser._handle_tool_name_streaming` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `MinimaxToolParser._handle_tool_args_streaming` (lines 597-667)
```python
    def _handle_tool_args_streaming(
        self, tool_content: str, tool_count: int
    ) -> DeltaMessage | None:
        """
        Handle streaming of tool arguments.

        Args:
            tool_content: Content containing tool calls
            tool_count: Total number of tools

        Returns:
            DeltaMessage with tool arguments or None if no arguments to stream
        """
        current_idx = self._get_current_tool_index()

        if current_idx < 0 or current_idx >= tool_count:
            return None

        tool_name, tool_args = self._get_current_tool_content(tool_content, current_idx)
        if not tool_name or tool_args is None:
            return None

        sent_tools = list(self.streaming_state["sent_tools"])

        if not sent_tools[current_idx]["sent_name"]:
            return None

        clean_args = self._clean_duplicate_braces(tool_args)
        sent_args = sent_tools[current_idx]["sent_arguments"]

# ... omitted for brevity / 为简洁省略 ...
                return DeltaMessage(
                    tool_calls=[
                        DeltaToolCall(
                            index=current_idx,
                            function=DeltaFunctionCall(
                                arguments=clean_args_delta
                            ).model_dump(exclude_none=True),
                        )
                    ]
                )

        return None
```
**EN:** Handle streaming of tool arguments.
**CN:** `MinimaxToolParser._handle_tool_args_streaming` 方法处理增量式流输出，只返回客户端应该看到的新差量。

### Method `MinimaxToolParser.extract_tool_calls_streaming` (lines 697-778)
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
        self._update_thinking_state(current_text)

        if self.in_thinking_tag:
            return DeltaMessage(content=delta_text)

        if self._should_buffer_content(delta_text):
            buffered_output = self._process_buffer(delta_text)
            return DeltaMessage(content=buffered_output) if buffered_output else None

        if self._is_end_tool_calls(current_text):
            return DeltaMessage(content=delta_text)

        safe_content, potential_tag = self._split_content_for_buffering(delta_text)
        if potential_tag:
            self.pending_buffer += potential_tag
            return DeltaMessage(content=safe_content) if safe_content else None

        processed_current_text = self.preprocess_model_output(current_text)

        if self.tool_call_start_token not in processed_current_text:
# ... omitted for brevity / 为简洁省略 ...

            self._ensure_state_arrays(current_tools_count)

            return self._handle_tool_name_streaming(
                tool_content, current_tools_count
            ) or self._handle_tool_args_streaming(tool_content, current_tools_count)

        except Exception:
            logger.exception(
                "An unexpected error occurred ", "during streaming tool call handling."
            )
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `MinimaxToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
