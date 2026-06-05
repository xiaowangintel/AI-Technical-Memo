# cohere_command_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/cohere_command_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the cohere command tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 cohere command 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-31)
```python
from collections.abc import Sequence

try:
    from cohere_melody import PyFilter, PyFilterOptions
except ImportError as e:
    raise ImportError(
        "The Cohere tool parser requires the `cohere_melody` "
        "package, which is not installed. Install it with:\n"
        "    pip install cohere_melody"
    ) from e

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
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser
from vllm.tool_parsers.utils import Tool
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Class `BaseCohereCommandToolParser` (lines 34-101)
```python
class BaseCohereCommandToolParser(ToolParser):
    def __init__(
        self,
        tokenizer: TokenizerLike,
        streaming_opts: PyFilterOptions,
        unary_opts: PyFilterOptions,
    ):
        super().__init__(tokenizer)
        self.melody_streaming = PyFilter(streaming_opts)
        self.melody_unary = PyFilter(unary_opts)
```
**EN:** Declares `BaseCohereCommandToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `BaseCohereCommandToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `BaseCohereCommandToolParser.__init__` (lines 35-43)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        streaming_opts: PyFilterOptions,
        unary_opts: PyFilterOptions,
    ):
        super().__init__(tokenizer)
        self.melody_streaming = PyFilter(streaming_opts)
        self.melody_unary = PyFilter(unary_opts)
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `BaseCohereCommandToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `BaseCohereCommandToolParser.adjust_request` (lines 45-50)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `BaseCohereCommandToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseCohereCommandToolParser.extract_tool_calls_streaming` (lines 52-81)
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
        r = self.melody_streaming.write_decoded(delta_text)
        if r.content is not None:
            return DeltaMessage(content=r.content)
        if r.reasoning is not None:
            return DeltaMessage(reasoning=r.reasoning)
        if r.tool_calls:
            return DeltaMessage(
                tool_calls=[
                    DeltaToolCall(
                        id=tc.id,
                        index=tc.index,
                        type="function",
                        function=DeltaFunctionCall(
                            name=tc.name, arguments=tc.arguments
                        ),
                    )
                    for tc in r.tool_calls
                ]
            )
        return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `BaseCohereCommandToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseCohereCommandToolParser.extract_tool_calls` (lines 83-101)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        result = self.melody_unary.process_full_text(model_output)
        tool_calls = [
            ToolCall(
                id=tc.id,
                type="function",
                function=FunctionCall(name=tc.name, arguments=tc.arguments),
            )
            for tc in result.tool_calls
        ]
        return ExtractedToolCallInformation(
            tools_called=len(tool_calls) > 0,
            tool_calls=tool_calls,
            content=result.content,
        )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `BaseCohereCommandToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Class `CohereCommand3ToolParser` (lines 104-114)
```python
class CohereCommand3ToolParser(BaseCohereCommandToolParser):
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        super().__init__(
            tokenizer,
            streaming_opts=PyFilterOptions().cmd3(),
            unary_opts=PyFilterOptions().cmd3(),
        )
```
**EN:** Declares `CohereCommand3ToolParser`, the main parser class in this module. It extends BaseCohereCommandToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `CohereCommand3ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `CohereCommand3ToolParser.__init__` (lines 105-114)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        super().__init__(
            tokenizer,
            streaming_opts=PyFilterOptions().cmd3(),
            unary_opts=PyFilterOptions().cmd3(),
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `CohereCommand3ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Class `CohereCommand4ToolParser` (lines 117-127)
```python
class CohereCommand4ToolParser(BaseCohereCommandToolParser):
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        super().__init__(
            tokenizer,
            streaming_opts=PyFilterOptions().cmd4(),
            unary_opts=PyFilterOptions().cmd4(),
        )
```
**EN:** Declares `CohereCommand4ToolParser`, the main parser class in this module. It extends BaseCohereCommandToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `CohereCommand4ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `CohereCommand4ToolParser.__init__` (lines 118-127)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        super().__init__(
            tokenizer,
            streaming_opts=PyFilterOptions().cmd4(),
            unary_opts=PyFilterOptions().cmd4(),
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `CohereCommand4ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.tokenizers`, `vllm.tool_parsers`, `vllm.tool_parsers.utils`
