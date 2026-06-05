# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/chat_completion/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: OpenAI-compatible chat completion. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：OpenAI 兼容 / 对话补全。

## Line-by-Line Analysis / 逐行分析
### Lines 6-45 — Imports and shared dependencies
```python
import json
import time
from typing import Annotated, Any, ClassVar, Literal

from openai.types.chat.chat_completion_audio import (
    ChatCompletionAudio as OpenAIChatCompletionAudio,
)
from openai.types.chat.chat_completion_message import Annotation as OpenAIAnnotation
from pydantic import Field, PrivateAttr, model_serializer, model_validator

from vllm.config import ModelConfig
from vllm.config.utils import replace
from vllm.entrypoints.chat_utils import (
    ChatCompletionMessageParam,
    ChatTemplateContentFormatOption,
)
from vllm.entrypoints.openai.engine.protocol import (
    AnyResponseFormat,
    DeltaMessage,
    FunctionCall,
    FunctionDefinition,
    LegacyStructuralTagResponseFormat,
    OpenAIBaseModel,
    StreamOptions,
    StructuralTagResponseFormat,
    ToolCall,
...
from vllm.sampling_params import (
    BeamSearchParams,
    RepetitionDetectionParams,
    RequestOutputKind,
    SamplingParams,
    StructuredOutputsParams,
)
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `json`, `time`, `typing`, uses third-party packages like `openai`, `pydantic`, depends on internal helpers such as `vllm.config`, `vllm.config.utils`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`.
**CN:** 该导入块引入 `json`, `time`, `typing` 等标准库模块，使用 `openai`, `pydantic` 等第三方库，依赖 `vllm.config`, `vllm.config.utils`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger` 等 vLLM 内部模块。

### Lines 47-51 — Module constants
```python
logger = init_logger(__name__)


_INT64_MIN = -(2**63)
_INT64_MAX = 2**63 - 1
```
**EN:** This block initializes `logger`, `_INT64_MIN`, `_INT64_MAX`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `_INT64_MIN`, `_INT64_MAX`，为后续逻辑准备模块级常量或共享状态。

### Lines 54-64 — Class `ChatMessage`
```python
class ChatMessage(OpenAIBaseModel):
    role: str
    content: str | None = None
    refusal: str | None = None
    annotations: OpenAIAnnotation | None = None
    audio: OpenAIChatCompletionAudio | None = None
    function_call: FunctionCall | None = None
    tool_calls: list[ToolCall] = Field(default_factory=list)

    # vLLM-specific fields that are not in OpenAI spec
    reasoning: str | None = None
```
**EN:** Class `ChatMessage` acts as a protocol/data model with about 8 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ChatMessage` 充当协议/数据模型，包含约 8 个字段声明，并提供 继承接口 等辅助方法。

### Lines 67-70 — Class `ChatCompletionLogProb`
```python
class ChatCompletionLogProb(OpenAIBaseModel):
    token: str
    logprob: float = -9999.0
    bytes: list[int] | None = None
```
**EN:** Class `ChatCompletionLogProb` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionLogProb`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 73-77 — Class `ChatCompletionLogProbsContent`
```python
class ChatCompletionLogProbsContent(ChatCompletionLogProb):
    # Workaround: redefine fields name cache so that it's not
    # shared with the super class.
    field_names: ClassVar[set[str] | None] = None
    top_logprobs: list[ChatCompletionLogProb] = Field(default_factory=list)
```
**EN:** Class `ChatCompletionLogProbsContent` is defined here, extending `ChatCompletionLogProb`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionLogProbsContent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 80-81 — Class `ChatCompletionLogProbs`
```python
class ChatCompletionLogProbs(OpenAIBaseModel):
    content: list[ChatCompletionLogProbsContent] | None = None
```
**EN:** Class `ChatCompletionLogProbs` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionLogProbs`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 84-94 — Class `ChatCompletionResponseChoice`
```python
class ChatCompletionResponseChoice(OpenAIBaseModel):
    index: int
    message: ChatMessage
    logprobs: ChatCompletionLogProbs | None = None
    # per OpenAI spec this is the default
    finish_reason: str | None = "stop"
    # not part of the OpenAI spec but included in vLLM for legacy reasons
    stop_reason: int | str | None = None
    # not part of the OpenAI spec but is useful for tracing the tokens
    # in agent scenarios
    token_ids: list[int] | None = None
```
**EN:** Class `ChatCompletionResponseChoice` acts as a protocol/data model with about 6 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ChatCompletionResponseChoice` 充当协议/数据模型，包含约 6 个字段声明，并提供 继承接口 等辅助方法。

### Lines 97-115 — Class `ChatCompletionResponse`
```python
class ChatCompletionResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"chatcmpl-{random_uuid()}")
    object: Literal["chat.completion"] = "chat.completion"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: list[ChatCompletionResponseChoice]
    service_tier: Literal["auto", "default", "flex", "scale", "priority"] | None = None
    system_fingerprint: str | None = None
    usage: UsageInfo

    # vLLM-specific fields that are not in OpenAI spec
    prompt_logprobs: list[dict[int, Logprob] | None] | None = None
    prompt_token_ids: list[int] | None = None
    # Rendered prompt text from chat templating (only set when
    # ``return_prompt_text=True`` on the request).
    prompt_text: str | None = None
    kv_transfer_params: dict[str, Any] | None = Field(
        default=None, description="KVTransfer parameters."
    )
```
**EN:** Class `ChatCompletionResponse` acts as a protocol/data model with about 12 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ChatCompletionResponse` 充当协议/数据模型，包含约 12 个字段声明，并提供 继承接口 等辅助方法。

### Lines 118-125 — Class `ChatCompletionResponseStreamChoice`
```python
class ChatCompletionResponseStreamChoice(OpenAIBaseModel):
    index: int
    delta: DeltaMessage
    logprobs: ChatCompletionLogProbs | None = None
    finish_reason: str | None = None
    stop_reason: int | str | None = None
    # not part of the OpenAI spec but for tracing the tokens
    token_ids: list[int] | None = None
```
**EN:** Class `ChatCompletionResponseStreamChoice` acts as a protocol/data model with about 6 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ChatCompletionResponseStreamChoice` 充当协议/数据模型，包含约 6 个字段声明，并提供 继承接口 等辅助方法。

### Lines 128-142 — Class `ChatCompletionStreamResponse`
```python
class ChatCompletionStreamResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"chatcmpl-{random_uuid()}")
    object: Literal["chat.completion.chunk"] = "chat.completion.chunk"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: list[ChatCompletionResponseStreamChoice]
    usage: UsageInfo | None = Field(default=None)
    # Set only on the final chunk of a stream to mirror non-streaming responses
    # without the per-chunk serialization overhead.
    system_fingerprint: str | None = None
    # not part of the OpenAI spec but for tracing the tokens
    prompt_token_ids: list[int] | None = None
    # Rendered prompt text from chat templating (only set when
    # ``return_prompt_text=True`` on the request); only sent on the first chunk.
    prompt_text: str | None = None
```
**EN:** Class `ChatCompletionStreamResponse` acts as a protocol/data model with about 9 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ChatCompletionStreamResponse` 充当协议/数据模型，包含约 9 个字段声明，并提供 继承接口 等辅助方法。

### Lines 145-161 — Class `ChatCompletionToolsParam`
```python
class ChatCompletionToolsParam(OpenAIBaseModel):
    type: Literal["function"] = "function"
    function: FunctionDefinition
    defer_loading: bool | None = None

    @model_validator(mode="after")
    def _propagate_defer_loading(self) -> "ChatCompletionToolsParam":
        if self.defer_loading is not None and self.function.defer_loading is None:
            self.function.defer_loading = self.defer_loading
        return self

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.defer_loading is None:
            data.pop("defer_loading", None)
        return data
```
**EN:** Class `ChatCompletionToolsParam` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like `_propagate_defer_loading`, `_serialize`.
**CN:** 这里定义类 `ChatCompletionToolsParam`，其职责是封装相关状态与行为，并通过 `_propagate_defer_loading`、`_serialize` 等方法组织逻辑。

### Lines 164-165 — Class `ChatCompletionNamedFunction`
```python
class ChatCompletionNamedFunction(OpenAIBaseModel):
    name: str
```
**EN:** Class `ChatCompletionNamedFunction` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionNamedFunction`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 168-170 — Class `ChatCompletionNamedToolChoiceParam`
```python
class ChatCompletionNamedToolChoiceParam(OpenAIBaseModel):
    function: ChatCompletionNamedFunction
    type: Literal["function"] = "function"
```
**EN:** Class `ChatCompletionNamedToolChoiceParam` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionNamedToolChoiceParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 173-870 — Class `ChatCompletionRequest`
```python
class ChatCompletionRequest(OpenAIBaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/chat/create
    messages: list[ChatCompletionMessageParam]
    model: str | None = None
    frequency_penalty: float | None = 0.0
    logit_bias: dict[str, float] | None = None
    logprobs: bool | None = False
    top_logprobs: int | None = 0
    max_tokens: int | None = Field(
        default=None,
        deprecated="max_tokens is deprecated in favor of "
        "the max_completion_tokens field",
    )
    max_completion_tokens: int | None = None
    n: int | None = 1
    presence_penalty: float | None = 0.0
    response_format: AnyResponseFormat | None = None
    seed: int | None = Field(None, ge=_INT64_MIN, le=_INT64_MAX)
    stop: str | list[str] | None = []
    stream: bool | None = False
    stream_options: StreamOptions | None = None
    temperature: float | None = None
    top_p: float | None = None
    tools: list[ChatCompletionToolsParam] | None = None
    tool_choice: (
        Literal["none"]
        | Literal["auto"]
...
                                logger.warning_once(
                                    "System messages should only contain text "
                                    "content according to the OpenAI API spec. "
                                    "Found content type: '%s'.",
                                    part_type,
                                )

        return data
```
**EN:** Class `ChatCompletionRequest` acts as a protocol/data model with about 60 field declarations and helper methods such as `_materialize_tool_calls_before`, `_materialize_tool_calls_after`, `build_chat_params`, `build_tok_params`.
**CN:** 类 `ChatCompletionRequest` 充当协议/数据模型，包含约 60 个字段声明，并提供 `_materialize_tool_calls_before`、`_materialize_tool_calls_after`、`build_chat_params`、`build_tok_params` 等辅助方法。

### Lines 873-950 — Class `BatchChatCompletionRequest`
```python
class BatchChatCompletionRequest(OpenAIBaseModel):
    """Request model for the /v1/chat/completions/batch endpoint.

    Accepts the same fields as ChatCompletionRequest except that ``messages``
    is a list of conversations (each conversation is a
    ``list[ChatCompletionMessageParam]``).  Each conversation is processed
    independently and the response contains one choice per conversation,
    indexed 0, 1, ..., N-1.

    Current limitations compared to the single-conversation endpoint:
    - Streaming is not supported (``stream`` must be False or omitted).
    - Tool use is not supported (``tools`` must be omitted).
    - Beam search is not supported (``use_beam_search`` must be False or omitted).
    - The ``n`` parameter must be 1 (or omitted).
    """

    messages: list[list[ChatCompletionMessageParam]] = Field(..., min_length=1)
    model: str | None = None

    # Shared sampling / generation fields — mirror ChatCompletionRequest.
    frequency_penalty: float | None = 0.0
    logit_bias: dict[str, float] | None = None
    logprobs: bool | None = False
    top_logprobs: int | None = 0
    max_tokens: int | None = None
    max_completion_tokens: int | None = None
    n: int | None = 1
    presence_penalty: float | None = 0.0
...

    def to_chat_completion_request(
        self, messages: list[ChatCompletionMessageParam]
    ) -> ChatCompletionRequest:
        """Build a single-conversation ChatCompletionRequest from one conversation."""
        data = self.model_dump(exclude={"messages"}, exclude_none=True)
        data["messages"] = messages
        return ChatCompletionRequest.model_validate(data)
```
**EN:** Class `BatchChatCompletionRequest` is introduced here. Its docstring describes the intent as: Request model for the /v1/chat/completions/batch endpoint.
**CN:** 这里定义类 `BatchChatCompletionRequest`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Batch workflow handling / 批处理工作流
- Embedding generation / 嵌入生成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `time`, `typing`
- **Third-party / 第三方**: `openai`, `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.config.utils`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `vllm.logprobs`, `vllm.renderers`, `vllm.sampling_params`, `vllm.utils`
