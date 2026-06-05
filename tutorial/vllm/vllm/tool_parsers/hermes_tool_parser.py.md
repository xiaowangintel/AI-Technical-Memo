# hermes_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/hermes_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the hermes tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 hermes 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
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
from vllm.entrypoints.openai.responses.protocol import ResponsesRequest
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import is_complete_json, partial_tag_overlap
from vllm.utils.mistral import is_mistral_tokenizer
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Hermes2ProToolParser` (lines 34-273)
```python
class Hermes2ProToolParser(ToolParser):
    tool_call_start_token: str = "<tool_call>"
```
**EN:** Declares `Hermes2ProToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Hermes2ProToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Hermes2ProToolParser.__init__` (lines 42-56)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        if is_mistral_tokenizer(tokenizer):
            logger.error("Detected Mistral tokenizer when using a Hermes model")
            self.model_tokenizer = tokenizer.tokenizer

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        # Streaming state: what has been sent to the client.
        self._sent_content_idx: int = 0
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Hermes2ProToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Hermes2ProToolParser.adjust_request` (lines 58-67)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # do not skip special tokens because the tool_call tokens are
            # marked "special" in some models. Since they are skipped
            # prior to the call to the tool parser, it breaks tool calling.
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Hermes2ProToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser.extract_tool_calls` (lines 69-119)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        # sanity check; avoid unnecessary processing
        if self.tool_call_start_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        else:
            try:
                # there are two possible captures - between tags, or between a
                # tag and end-of-string so the result of
                # findall is an array of tuples where one is a function call and
                # the other is None
                function_call_tuples = self.tool_call_regex.findall(model_output)

                # load the JSON, and then use it to build the Function and
                # Tool Call
                raw_function_calls = [
                    json.loads(match[0] if match[0] else match[1])
                    for match in function_call_tuples
                ]
                tool_calls = [
                    ToolCall(
                        type="function",
                        function=FunctionCall(
                            name=function_call["name"],
                            # function call args are JSON but as a string
                            arguments=json.dumps(
                                function_call["arguments"], ensure_ascii=False
                            ),
                        ),
                    )
                    for function_call in raw_function_calls
                ]

                content = model_output[: model_output.find(self.tool_call_start_token)]
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
**CN:** `Hermes2ProToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Hermes2ProToolParser._extract_content` (lines 121-138)
```python
    def _extract_content(self, current_text: str) -> str | None:
        """Return unsent non-tool-call text, or None.

        Holds back any suffix that could be a partial <tool_call> tag.
        """
        if self.tool_call_start_token not in current_text:
            overlap_length = partial_tag_overlap(
                current_text, self.tool_call_start_token
            )
            sendable_idx = len(current_text) - overlap_length
        else:
            sendable_idx = current_text.index(self.tool_call_start_token)

        if sendable_idx > self._sent_content_idx:
            content = current_text[self._sent_content_idx : sendable_idx]
            self._sent_content_idx = sendable_idx
            return content
        return None
```
**EN:** Return unsent non-tool-call text, or None.
**CN:** `Hermes2ProToolParser._extract_content` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser._extract_tool_call_jsons` (lines 140-165)
```python
    def _extract_tool_call_jsons(self, text: str) -> list[tuple[str, bool]]:
        """Extract (json_text, is_complete) for each <tool_call> region."""
        results: list[tuple[str, bool]] = []
        pos = 0
        while True:
            start = text.find(self.tool_call_start_token, pos)
            if start == -1:
                break
            json_start = start + len(self.tool_call_start_token)
            json_end = text.find(self.tool_call_end_token, json_start)
            if json_end != -1:
                results.append((text[json_start:json_end].strip(), True))
                pos = json_end + len(self.tool_call_end_token)
            else:
                raw = text[json_start:]
                # Strip partial </tool_call> suffix if present.
                overlap = partial_tag_overlap(raw, self.tool_call_end_token)
                if overlap:
                    raw = raw[:-overlap]
                tc_json = raw.strip()
                # Valid JSON without closing tag = complete body,
                # tag tokens just haven't arrived yet.
                is_complete = is_complete_json(tc_json) if tc_json else False
                results.append((tc_json, is_complete))
                break
        return results
```
**EN:** Extract (json_text, is_complete) for each <tool_call> region.
**CN:** `Hermes2ProToolParser._extract_tool_call_jsons` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser._extract_tool_name` (lines 168-171)
```python
    def _extract_tool_name(tc_json: str) -> str | None:
        """Extract tool name, or None if the name isn't complete yet."""
        match = re.search(r'"name"\s*:\s*"([^"]+)"', tc_json)
        return match.group(1) if match else None
```
**EN:** Extract tool name, or None if the name isn't complete yet.
**CN:** `Hermes2ProToolParser._extract_tool_name` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser._extract_tool_args` (lines 174-189)
```python
    def _extract_tool_args(tc_json: str, is_complete: bool) -> str | None:
        """Extract tool arguments from the tool call JSON.

        Given {"name": "f", "arguments": {"x": 1}}, returns '{"x": 1}'.
        When is_complete, strips the trailing '}' that closes the outer
        object (not the arguments). For partial JSON, returns as-is.
        """
        match = re.search(r'"arguments"\s*:\s*', tc_json)
        if not match:
            return None
        raw = tc_json[match.end() :]
        if is_complete:
            raw = raw.rstrip()
            if raw.endswith("}"):
                raw = raw[:-1].rstrip()
        return raw
```
**EN:** Extract tool arguments from the tool call JSON.
**CN:** `Hermes2ProToolParser._extract_tool_args` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser._compute_args_diff` (lines 191-201)
```python
    def _compute_args_diff(
        self, index: int, tc_json: str, is_complete: bool
    ) -> str | None:
        """Return new argument text not yet sent for tool `index`, or None."""
        args = self._extract_tool_args(tc_json, is_complete)
        if args is None or len(args) <= len(self.streamed_args_for_tool[index]):
            return None
        diff = args[len(self.streamed_args_for_tool[index]) :]
        self.streamed_args_for_tool[index] = args
        self.prev_tool_call_arr[index]["arguments"] = args
        return diff
```
**EN:** Return new argument text not yet sent for tool `index`, or None.
**CN:** `Hermes2ProToolParser._compute_args_diff` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Hermes2ProToolParser.extract_tool_calls_streaming` (lines 203-273)
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
        """Incrementally stream tool call deltas from accumulated output.

        On each invocation, re-parses the full ``current_text`` to find
        ``<tool_call>`` regions, then diffs against previously sent state
        to emit only new content, tool names, or argument fragments.

        Returns a ``DeltaMessage`` containing either plain content (for
        text preceding any tool call) or one or more ``DeltaToolCall``
        entries, or ``None`` if there is nothing new to send yet."""
        try:
            # Extract any content before tool calls.
            content = self._extract_content(current_text)
            tool_call_jsons = self._extract_tool_call_jsons(current_text)
            tool_call_deltas: list[DeltaToolCall] = []

            for i, (tc_json, is_complete) in enumerate(tool_call_jsons):
                if i >= len(self.prev_tool_call_arr):
                    self.prev_tool_call_arr.append({})
                    self.streamed_args_for_tool.append("")

                # Stream back tool name.
                if "name" not in self.prev_tool_call_arr[i]:
                    name = self._extract_tool_name(tc_json)
                    if not name:
                        # Can't skip to tool i+1 if i isn't ready
                        break
                    self.prev_tool_call_arr[i]["name"] = name
                    tool_call_deltas.append(
                        DeltaToolCall(
                            index=i,
                            type="function",
                            id=make_tool_call_id(),
                            function=DeltaFunctionCall(name=name).model_dump(
                                exclude_none=True
                            ),
                        )
                    )

                # Stream back new tool args by diffing against what was sent.
                args_diff = self._compute_args_diff(i, tc_json, is_complete)
                if args_diff:
                    tool_call_deltas.append(
                        DeltaToolCall(
                            index=i,
                            function=DeltaFunctionCall(arguments=args_diff).model_dump(
                                exclude_none=True
                            ),
                        )
                    )

            if content or tool_call_deltas:
                return DeltaMessage(
                    content=content,
                    tool_calls=tool_call_deltas,
                )

            return None

        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            return None
```
**EN:** Incrementally stream tool call deltas from accumulated output.
**CN:** `Hermes2ProToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`, `vllm.utils.mistral`
