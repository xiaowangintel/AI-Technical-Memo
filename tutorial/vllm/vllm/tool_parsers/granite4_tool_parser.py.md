# granite4_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/granite4_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the granite4 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 granite4 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-28)
```python
import json
from collections.abc import Sequence
from typing import Any, Protocol, TypeVar

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
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 30-30)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Function `dump_args` (lines 33-37)
```python
def dump_args(args: None | dict[str, Any] | str) -> str | None:
    if args is None or isinstance(args, str):
        return args
    else:
        return json.dumps(args, ensure_ascii=False)
```
**EN:** Provides the `dump_args` helper used by this module to keep parsing logic modular and reusable.
**CN:** `dump_args` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `_FunctionCallCtor` (lines 40-41)
```python
class _FunctionCallCtor(Protocol):
    def __init__(self, *, name: str, arguments: str | None): ...
```
**EN:** Declares `_FunctionCallCtor`, the main parser class in this module. It extends Protocol. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `_FunctionCallCtor` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `_FunctionCallCtor.__init__` (lines 41-41)
```python
    def __init__(self, *, name: str, arguments: str | None): ...
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `_FunctionCallCtor.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Module constants/configuration 2 (lines 44-44)
```python
FuncT = TypeVar("FuncT", bound=_FunctionCallCtor)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Granite4ToolParser` (lines 47-256)
```python
class Granite4ToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id: int = -1
        self.streamed_args_for_tool = list[str]()

        self.look_ahead = ""
        self.in_tc = False

        self.tc_start = "<tool_call>"
        self.tc_end = "</tool_call>"
        self.start_regex = re.compile(self.tc_start)
        self.end_regex = re.compile(self.tc_end)
```
**EN:** Declares `Granite4ToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Granite4ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Granite4ToolParser.__init__` (lines 48-61)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        super().__init__(tokenizer, tools)

        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id: int = -1
        self.streamed_args_for_tool = list[str]()

        self.look_ahead = ""
        self.in_tc = False

        self.tc_start = "<tool_call>"
        self.tc_end = "</tool_call>"
        self.start_regex = re.compile(self.tc_start)
        self.end_regex = re.compile(self.tc_end)
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Granite4ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Granite4ToolParser.adjust_request` (lines 63-72)
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
**CN:** `Granite4ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Granite4ToolParser._collect_results` (lines 74-90)
```python
    def _collect_results(
        self, text_segments: list[str], tc_segments: list[str], cls: type[FuncT]
    ) -> tuple[str, list[FuncT]]:
        tool_calls_json: list[dict[str, Any]] = [
            json.loads(tc_text) for tc_text in tc_segments
        ]
        tool_calls = []
        for tc in tool_calls_json:
            assert isinstance(tc, dict)
            self.prev_tool_call_arr.append(tc)
            tool_calls.append(
                cls(
                    name=tc["name"],
                    arguments=dump_args(tc["arguments"]),
                )
            )
        return "".join(text_segments), tool_calls
```
**EN:** Provides the `_collect_results` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Granite4ToolParser._collect_results` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Granite4ToolParser.extract_tool_calls` (lines 92-148)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        msg = ExtractedToolCallInformation(
            tools_called=False, tool_calls=[], content=model_output
        )
        try:
            delimiters = [("TC_START", self.tc_start), ("TC_END", self.tc_end)]
            pattern = "|".join(f"(?P<{name}>{pattern})" for name, pattern in delimiters)
            regex = re.compile(pattern)

            text_segments = list[str]()
            tc_segments = list[str]()
            last_cut_loc = 0

            for match in regex.finditer(model_output):
                match_type = match.lastgroup
                if match_type == "TC_START":
                    assert not self.in_tc, "Two tool call start tokens found in a row"
                    if preceding_text := model_output[last_cut_loc : match.start()]:
                        text_segments.append(preceding_text)
                    self.in_tc = True
                elif match_type == "TC_END":
                    assert self.in_tc, (
                        "Tool call end token found without corresponding start token"
                    )
                    tool_text = model_output[last_cut_loc : match.start()]
                    assert tool_text, (
                        "Expected the model to generate text between tool call tokens"
                    )
                    tc_segments.append(tool_text)
                    self.in_tc = False
                else:
                    raise ValueError("Unexpected match")
                last_cut_loc = match.end()
            assert not self.in_tc, "The model generated an incomplete tool call"
            if final_text := model_output[last_cut_loc:]:
                text_segments.append(final_text)

            content, tool_call_funcs = self._collect_results(
                text_segments, tc_segments, FunctionCall
            )
            tool_calls = [
                ToolCall(
                    type="function",
                    function=func,
                )
                for func in tool_call_funcs
            ]
            msg.tools_called = bool(tool_calls)
            msg.tool_calls = tool_calls
            msg.content = content or None
        except Exception:
            logger.exception("Error in extracting tool call from response.")
        return msg
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Granite4ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Granite4ToolParser._tool_extraction_step` (lines 150-203)
```python
    def _tool_extraction_step(
        self,
        delta_text: str,
    ) -> tuple[bool, str, str]:
        start_token_pos = start_token_end = end_token_pos = end_token_end = -1

        if start_match := self.start_regex.search(delta_text, partial=True):
            if not start_match.partial:
                start_token_pos, start_token_end = start_match.span()
            elif start_match.end() > start_match.start():
                start_token_pos = -2

        if end_match := self.end_regex.search(delta_text):
            end_token_pos, end_token_end = end_match.span()

        # Done means that we've exhausted the current buffer
        # and need more output from the model
        done = True
        content = tc_text = ""

        if start_token_pos < 0:
            # just streaming text so far
            if start_token_pos == -2:
                # There is a partial match
                content = delta_text[: start_match.start()]
                self.look_ahead = delta_text[start_match.start() :]
            else:
                content = delta_text

        elif not self.in_tc:
            # we're entering a new tool call
            self.in_tc = True

            content = delta_text[:start_token_pos]
            if end_token_pos > 0:
                self.start_in_tc = False
                tc_text = delta_text[start_token_end:end_token_pos]
                self.look_ahead = delta_text[end_token_end:]
                done = False  # There could be more content already buffered
            else:
                self.look_ahead = delta_text[start_token_pos:]

        elif end_token_pos < 0:
            # we're in between the start and the end token
            assert self.in_tc
            self.look_ahead = delta_text
        else:
            # We have found the end
            assert self.in_tc
            tc_text = delta_text[start_token_end:end_token_pos]
            self.in_tc = False
            self.look_ahead = delta_text[end_token_end:]
            done = False  # There could be more content already buffered
        return done, content, tc_text
```
**EN:** Provides the `_tool_extraction_step` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Granite4ToolParser._tool_extraction_step` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Granite4ToolParser.extract_tool_calls_streaming` (lines 205-256)
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
        try:
            done = False
            text_segments = list[str]()
            tc_segments = list[str]()

            while not done:
                delta_text = self.look_ahead + delta_text
                self.look_ahead = ""
                done, content, tc_text = self._tool_extraction_step(delta_text)
                if content:
                    text_segments.append(content)
                if tc_text:
                    tc_segments.append(tc_text)
                delta_text = ""

            content, tool_call_funcs = self._collect_results(
                text_segments, tc_segments, DeltaFunctionCall
            )

            delta_tool_calls = list[DeltaToolCall]()
            for function in tool_call_funcs:
                self.current_tool_id += 1
                delta_tool_calls.append(
                    DeltaToolCall(
                        id=make_tool_call_id(),
                        type="function",
                        index=self.current_tool_id,
                        function=function.model_dump(exclude_none=True),
                    )
                )
                self.streamed_args_for_tool.append(function.arguments or "")

            assert self.current_tool_id + 1 == len(self.prev_tool_call_arr)
            assert self.current_tool_id + 1 == len(self.streamed_args_for_tool)

            msg = DeltaMessage(content=content or None, tool_calls=delta_tool_calls)
            if msg.content or msg.tool_calls:
                return msg

        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
        return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Granite4ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
