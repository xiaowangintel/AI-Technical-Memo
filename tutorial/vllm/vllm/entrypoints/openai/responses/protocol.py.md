# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: OpenAI-compatible responses. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 6-72 — Imports and shared dependencies
```python
import time
from typing import Any, Literal, TypeAlias

from openai.types.responses import (
    ResponseCodeInterpreterCallCodeDeltaEvent,
    ResponseCodeInterpreterCallCodeDoneEvent,
    ResponseCodeInterpreterCallCompletedEvent,
    ResponseCodeInterpreterCallInProgressEvent,
    ResponseCodeInterpreterCallInterpretingEvent,
    ResponseContentPartAddedEvent,
    ResponseContentPartDoneEvent,
    ResponseFunctionToolCall,
    ResponseInputItemParam,
    ResponseMcpCallArgumentsDeltaEvent,
    ResponseMcpCallArgumentsDoneEvent,
    ResponseMcpCallCompletedEvent,
    ResponseMcpCallInProgressEvent,
    ResponseOutputItem,
    ResponseOutputItemAddedEvent,
    ResponseOutputItemDoneEvent,
    ResponseOutputMessage,
    ResponsePrompt,
    ResponseReasoningItem,
    ResponseReasoningTextDeltaEvent,
    ResponseReasoningTextDoneEvent,
    ResponseStatus,
...
from vllm.logger import init_logger
from vllm.renderers import ChatParams, TokenizeParams, merge_kwargs
from vllm.sampling_params import (
    RequestOutputKind,
    SamplingParams,
    StructuredOutputsParams,
)
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `time`, `typing`, uses third-party packages like `openai`, `openai_harmony`, `pydantic`, depends on internal helpers such as `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.renderers`.
**CN:** 该导入块引入 `time`, `typing` 等标准库模块，使用 `openai`, `openai_harmony`, `pydantic` 等第三方库，依赖 `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.renderers` 等 vLLM 内部模块。

### Lines 74-77 — Module constants
```python
logger = init_logger(__name__)

_INT64_MIN = -(2**63)
_INT64_MAX = 2**63 - 1
```
**EN:** This block initializes `logger`, `_INT64_MIN`, `_INT64_MAX`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `_INT64_MIN`, `_INT64_MAX`，为后续逻辑准备模块级常量或共享状态。

### Lines 80-83 — Class `InputTokensDetails`
```python
class InputTokensDetails(OpenAIBaseModel):
    cached_tokens: int
    input_tokens_per_turn: list[int] = Field(default_factory=list)
    cached_tokens_per_turn: list[int] = Field(default_factory=list)
```
**EN:** Class `InputTokensDetails` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `InputTokensDetails`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 86-90 — Class `OutputTokensDetails`
```python
class OutputTokensDetails(OpenAIBaseModel):
    reasoning_tokens: int = 0
    tool_output_tokens: int = 0
    output_tokens_per_turn: list[int] = Field(default_factory=list)
    tool_output_tokens_per_turn: list[int] = Field(default_factory=list)
```
**EN:** Class `OutputTokensDetails` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `OutputTokensDetails`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 93-98 — Class `ResponseUsage`
```python
class ResponseUsage(OpenAIBaseModel):
    input_tokens: int
    input_tokens_details: InputTokensDetails
    output_tokens: int
    output_tokens_details: OutputTokensDetails
    total_tokens: int
```
**EN:** Class `ResponseUsage` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseUsage`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 101-111 — Function `serialize_message`
```python
def serialize_message(msg):
    """
    Serializes a single message
    """
    if isinstance(msg, dict):
        return msg
    elif hasattr(msg, "to_dict"):
        return msg.to_dict()
    else:
        # fallback to pydantic dump
        return msg.model_dump(mode="json", by_alias=True)
```
**EN:** This function `serialize_message` is documented as: Serializes a single message
**CN:** 这里定义函数 `serialize_message`，其文档字符串说明了主要职责与调用约定。

### Lines 114-118 — Function `serialize_messages`
```python
def serialize_messages(msgs):
    """
    Serializes multiple messages
    """
    return [serialize_message(msg) for msg in msgs] if msgs else None
```
**EN:** This function `serialize_messages` is documented as: Serializes multiple messages
**CN:** 这里定义函数 `serialize_messages`，其文档字符串说明了主要职责与调用约定。

### Lines 121-127 — Class `ResponseRawMessageAndToken`
```python
class ResponseRawMessageAndToken(OpenAIBaseModel):
    """Class to show the raw message.
    If message / tokens diverge, tokens is the source of truth"""

    message: str
    tokens: list[int]
    type: Literal["raw_message_tokens"] = "raw_message_tokens"
```
**EN:** Class `ResponseRawMessageAndToken` is introduced here. Its docstring describes the intent as: Class to show the raw message.
**CN:** 这里定义类 `ResponseRawMessageAndToken`。其文档字符串说明了该类的职责与使用方式。

### Lines 130-133 — Module constants and state
```python
ResponseInputOutputMessage: TypeAlias = (
    list[ChatCompletionMessageParam] | list[ResponseRawMessageAndToken]
)
ResponseInputOutputItem: TypeAlias = ResponseInputItemParam | ResponseOutputItem
```
**EN:** This block initializes `ResponseInputOutputMessage`, `ResponseInputOutputItem`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ResponseInputOutputMessage`, `ResponseInputOutputItem`，为后续逻辑准备模块级常量或共享状态。

### Lines 136-590 — Class `ResponsesRequest`
```python
class ResponsesRequest(OpenAIBaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/responses/create
    background: bool | None = False
    include: (
        list[
            Literal[
                "code_interpreter_call.outputs",
                "computer_call_output.output.image_url",
                "file_search_call.results",
                "message.input_image.image_url",
                "message.output_text.logprobs",
                "reasoning.encrypted_content",
            ],
        ]
        | None
    ) = None
    input: str | list[ResponseInputOutputItem]
    instructions: str | None = None
    max_output_tokens: int | None = None
    max_tool_calls: int | None = None
    metadata: Metadata | None = None
    model: str | None = None
    logit_bias: dict[str, float] | None = None
    parallel_tool_calls: bool | None = True
    previous_response_id: str | None = None
    prompt: ResponsePrompt | None = None
    reasoning: Reasoning | None = None
...
            }
            if not tool_name or tool_name not in tool_names:
                raise VLLMValidationError(
                    "Tool choice 'function' not found in 'tools' parameter.",
                    parameter="tool_choice",
                )

        return data
```
**EN:** Class `ResponsesRequest` acts as a protocol/data model with about 46 field declarations and helper methods such as `build_chat_params`, `build_tok_params`, `to_sampling_params`, `is_include_output_logprobs`.
**CN:** 类 `ResponsesRequest` 充当协议/数据模型，包含约 46 个字段声明，并提供 `build_chat_params`、`build_tok_params`、`to_sampling_params`、`is_include_output_logprobs` 等辅助方法。

### Lines 593-728 — Class `ResponsesResponse`
```python
class ResponsesResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"resp_{random_uuid()}")
    created_at: int = Field(default_factory=lambda: int(time.time()))
    # error: Optional[ResponseError] = None
    incomplete_details: IncompleteDetails | None = None
    instructions: str | None = None
    metadata: Metadata | None = None
    model: str
    object: Literal["response"] = "response"
    output: list[ResponseOutputItem]
    parallel_tool_calls: bool
    temperature: float
    tool_choice: ToolChoice
    tools: list[Tool]
    top_p: float
    background: bool
    max_output_tokens: int
    max_tool_calls: int | None = None
    previous_response_id: str | None = None
    prompt: ResponsePrompt | None = None
    reasoning: Reasoning | None = None
    service_tier: Literal["auto", "default", "flex", "scale", "priority"]
    status: ResponseStatus
    text: ResponseTextConfig | None = None
    top_logprobs: int | None = None
    truncation: Literal["auto", "disabled"]
    usage: ResponseUsage | None = None
    user: str | None = None
...
            status=status,
            text=request.text,
            top_logprobs=sampling_params.logprobs,
            truncation=request.truncation,
            user=request.user,
            usage=usage,
            kv_transfer_params=kv_transfer_params,
        )
```
**EN:** Class `ResponsesResponse` acts as a protocol/data model with about 31 field declarations and helper methods such as `serialize_output_messages`, `serialize_input_messages`, `from_request`.
**CN:** 类 `ResponsesResponse` 充当协议/数据模型，包含约 31 个字段声明，并提供 `serialize_output_messages`、`serialize_input_messages`、`from_request` 等辅助方法。

### Lines 733-750 — Class `ResponseReasoningPartDoneEvent`
```python
class ResponseReasoningPartDoneEvent(OpenAIBaseModel):
    content_index: int
    """The index of the content part that is done."""

    item_id: str
    """The ID of the output item that the content part was added to."""

    output_index: int
    """The index of the output item that the content part was added to."""

    part: ResponseReasoningTextContent
    """The content part that is done."""

    sequence_number: int
    """The sequence number of this event."""

    type: Literal["response.reasoning_part.done"]
    """The type of the event. Always `response.reasoning_part.done`."""
```
**EN:** Class `ResponseReasoningPartDoneEvent` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseReasoningPartDoneEvent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 755-772 — Class `ResponseReasoningPartAddedEvent`
```python
class ResponseReasoningPartAddedEvent(OpenAIBaseModel):
    content_index: int
    """The index of the content part that is done."""

    item_id: str
    """The ID of the output item that the content part was added to."""

    output_index: int
    """The index of the output item that the content part was added to."""

    part: ResponseReasoningTextContent
    """The content part that is done."""

    sequence_number: int
    """The sequence number of this event."""

    type: Literal["response.reasoning_part.added"]
    """The type of the event. Always `response.reasoning_part.added`."""
```
**EN:** Class `ResponseReasoningPartAddedEvent` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseReasoningPartAddedEvent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 777-778 — Class `ResponseCompletedEvent`
```python
class ResponseCompletedEvent(OpenAIResponseCompletedEvent):
    response: ResponsesResponse  # type: ignore[override]
```
**EN:** Class `ResponseCompletedEvent` is defined here, extending `OpenAIResponseCompletedEvent`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseCompletedEvent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 781-782 — Class `ResponseCreatedEvent`
```python
class ResponseCreatedEvent(OpenAIResponseCreatedEvent):
    response: ResponsesResponse  # type: ignore[override]
```
**EN:** Class `ResponseCreatedEvent` is defined here, extending `OpenAIResponseCreatedEvent`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseCreatedEvent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 785-786 — Class `ResponseInProgressEvent`
```python
class ResponseInProgressEvent(OpenAIResponseInProgressEvent):
    response: ResponsesResponse  # type: ignore[override]
```
**EN:** Class `ResponseInProgressEvent` is defined here, extending `OpenAIResponseInProgressEvent`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ResponseInProgressEvent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 789-813 — Module constants and state
```python
StreamingResponsesResponse: TypeAlias = (
    ResponseCreatedEvent
    | ResponseInProgressEvent
    | ResponseCompletedEvent
    | ResponseOutputItemAddedEvent
    | ResponseOutputItemDoneEvent
    | ResponseContentPartAddedEvent
    | ResponseContentPartDoneEvent
    | ResponseReasoningTextDeltaEvent
    | ResponseReasoningTextDoneEvent
    | ResponseReasoningPartAddedEvent
    | ResponseReasoningPartDoneEvent
    | ResponseCodeInterpreterCallInProgressEvent
    | ResponseCodeInterpreterCallCodeDeltaEvent
    | ResponseWebSearchCallInProgressEvent
...
    | ResponseCodeInterpreterCallCodeDoneEvent
    | ResponseCodeInterpreterCallInterpretingEvent
    | ResponseCodeInterpreterCallCompletedEvent
    | ResponseMcpCallArgumentsDeltaEvent
    | ResponseMcpCallArgumentsDoneEvent
    | ResponseMcpCallInProgressEvent
    | ResponseMcpCallCompletedEvent
)
```
**EN:** This block initializes `StreamingResponsesResponse`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `StreamingResponsesResponse`，为后续逻辑准备模块级常量或共享状态。

## Key Concepts / 关键概念
- Streaming responses / 流式响应
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `typing`
- **Third-party / 第三方**: `openai`, `openai_harmony`, `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.renderers`, `vllm.sampling_params`, `vllm.utils`, `.utils`
