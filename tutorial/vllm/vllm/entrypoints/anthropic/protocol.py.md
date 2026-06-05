# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/anthropic/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: Anthropic-compatible. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：Anthropic 兼容。

## Line-by-Line Analysis / 逐行分析
### Lines 3-3 — Module overview
```python
"""Pydantic models for Anthropic API protocol"""
```
**EN:** The module header documents the main contract: Pydantic models for Anthropic API protocol
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 5-8 — Imports and shared dependencies
```python
import time
from typing import Any, Literal

from pydantic import BaseModel, Field, field_validator, model_validator
```
**EN:** This import block pulls in standard-library modules such as `time`, `typing`, uses third-party packages like `pydantic`.
**CN:** 该导入块引入 `time`, `typing` 等标准库模块，使用 `pydantic` 等第三方库。

### Lines 11-15 — Class `AnthropicError`
```python
class AnthropicError(BaseModel):
    """Error structure for Anthropic API"""

    type: str
    message: str
```
**EN:** Class `AnthropicError` is introduced here. Its docstring describes the intent as: Error structure for Anthropic API
**CN:** 这里定义类 `AnthropicError`。其文档字符串说明了该类的职责与使用方式。

### Lines 18-22 — Class `AnthropicErrorResponse`
```python
class AnthropicErrorResponse(BaseModel):
    """Error response structure for Anthropic API"""

    type: Literal["error"] = "error"
    error: AnthropicError
```
**EN:** Class `AnthropicErrorResponse` is introduced here. Its docstring describes the intent as: Error response structure for Anthropic API
**CN:** 这里定义类 `AnthropicErrorResponse`。其文档字符串说明了该类的职责与使用方式。

### Lines 25-31 — Class `AnthropicUsage`
```python
class AnthropicUsage(BaseModel):
    """Token usage information"""

    input_tokens: int
    output_tokens: int
    cache_creation_input_tokens: int | None = None
    cache_read_input_tokens: int | None = None
```
**EN:** Class `AnthropicUsage` is introduced here. Its docstring describes the intent as: Token usage information
**CN:** 这里定义类 `AnthropicUsage`。其文档字符串说明了该类的职责与使用方式。

### Lines 34-62 — Class `AnthropicContentBlock`
```python
class AnthropicContentBlock(BaseModel):
    """Content block in message"""

    type: Literal[
        "text",
        "image",
        "tool_use",
        "tool_result",
        "tool_reference",
        "thinking",
        "redacted_thinking",
    ]
    text: str | None = None
    # For image content
    source: dict[str, Any] | None = None
    # For tool use/result
    id: str | None = None
    tool_use_id: str | None = None
    name: str | None = None
    input: dict[str, Any] | None = None
    content: str | list[dict[str, Any]] | None = None
    is_error: bool | None = None
    # For tool_reference content
    tool_name: str | None = None
    # For thinking content
    thinking: str | None = None
    signature: str | None = None
    # For redacted thinking content (safety-filtered by the API)
    data: str | None = None
```
**EN:** Class `AnthropicContentBlock` is introduced here. Its docstring describes the intent as: Content block in message
**CN:** 这里定义类 `AnthropicContentBlock`。其文档字符串说明了该类的职责与使用方式。

### Lines 65-69 — Class `AnthropicMessage`
```python
class AnthropicMessage(BaseModel):
    """Message structure"""

    role: Literal["user", "assistant"]
    content: str | list[AnthropicContentBlock]
```
**EN:** Class `AnthropicMessage` is introduced here. Its docstring describes the intent as: Message structure
**CN:** 这里定义类 `AnthropicMessage`。其文档字符串说明了该类的职责与使用方式。

### Lines 72-87 — Class `AnthropicTool`
```python
class AnthropicTool(BaseModel):
    """Tool definition"""

    name: str
    description: str | None = None
    input_schema: dict[str, Any]
    defer_loading: bool | None = None

    @field_validator("input_schema")
    @classmethod
    def validate_input_schema(cls, v):
        if not isinstance(v, dict):
            raise ValueError("input_schema must be a dictionary")
        if "type" not in v:
            v["type"] = "object"  # Default to object type
        return v
```
**EN:** Class `AnthropicTool` is introduced here. Its docstring describes the intent as: Tool definition
**CN:** 这里定义类 `AnthropicTool`。其文档字符串说明了该类的职责与使用方式。

### Lines 90-100 — Class `AnthropicToolChoice`
```python
class AnthropicToolChoice(BaseModel):
    """Tool Choice definition"""

    type: Literal["auto", "any", "tool", "none"]
    name: str | None = None

    @model_validator(mode="after")
    def validate_name_required_for_tool(self) -> "AnthropicToolChoice":
        if self.type == "tool" and not self.name:
            raise ValueError("tool_choice.name is required when type is 'tool'")
        return self
```
**EN:** Class `AnthropicToolChoice` is introduced here. Its docstring describes the intent as: Tool Choice definition
**CN:** 这里定义类 `AnthropicToolChoice`。其文档字符串说明了该类的职责与使用方式。

### Lines 103-144 — Class `AnthropicMessagesRequest`
```python
class AnthropicMessagesRequest(BaseModel):
    """Anthropic Messages API request"""

    model: str
    messages: list[AnthropicMessage]
    max_tokens: int
    metadata: dict[str, Any] | None = None
    stop_sequences: list[str] | None = None
    stream: bool | None = False
    system: str | list[AnthropicContentBlock] | None = None
    temperature: float | None = None
    tool_choice: AnthropicToolChoice | None = None
    tools: list[AnthropicTool] | None = None
    top_k: int | None = None
    top_p: float | None = None

    # vLLM-specific fields that are not in Anthropic spec
    kv_transfer_params: dict[str, Any] | None = Field(
        default=None,
        description="KVTransfer parameters used for disaggregated serving.",
    )
    chat_template_kwargs: dict[str, Any] | None = Field(
        default=None,
        description=(
            "Additional keyword args to pass to the chat template renderer. "
            "Will be accessible by the template."
        ),
    )

    @field_validator("model")
    @classmethod
    def validate_model(cls, v):
        if not v:
            raise ValueError("Model is required")
        return v

    @field_validator("max_tokens")
    @classmethod
    def validate_max_tokens(cls, v):
        if v <= 0:
            raise ValueError("max_tokens must be positive")
        return v
```
**EN:** Class `AnthropicMessagesRequest` is introduced here. Its docstring describes the intent as: Anthropic Messages API request
**CN:** 这里定义类 `AnthropicMessagesRequest`。其文档字符串说明了该类的职责与使用方式。

### Lines 147-163 — Class `AnthropicDelta`
```python
class AnthropicDelta(BaseModel):
    """Delta for streaming responses"""

    type: (
        Literal["text_delta", "input_json_delta", "thinking_delta", "signature_delta"]
        | None
    ) = None
    text: str | None = None
    thinking: str | None = None
    partial_json: str | None = None
    signature: str | None = None

    # Message delta
    stop_reason: (
        Literal["end_turn", "max_tokens", "stop_sequence", "tool_use"] | None
    ) = None
    stop_sequence: str | None = None
```
**EN:** Class `AnthropicDelta` is introduced here. Its docstring describes the intent as: Delta for streaming responses
**CN:** 这里定义类 `AnthropicDelta`。其文档字符串说明了该类的职责与使用方式。

### Lines 166-184 — Class `AnthropicStreamEvent`
```python
class AnthropicStreamEvent(BaseModel):
    """Streaming event"""

    type: Literal[
        "message_start",
        "message_delta",
        "message_stop",
        "content_block_start",
        "content_block_delta",
        "content_block_stop",
        "ping",
        "error",
    ]
    message: "AnthropicMessagesResponse | None" = None
    delta: AnthropicDelta | None = None
    content_block: AnthropicContentBlock | None = None
    index: int | None = None
    error: AnthropicError | None = None
    usage: AnthropicUsage | None = None
```
**EN:** Class `AnthropicStreamEvent` is introduced here. Its docstring describes the intent as: Streaming event
**CN:** 这里定义类 `AnthropicStreamEvent`。其文档字符串说明了该类的职责与使用方式。

### Lines 187-208 — Class `AnthropicMessagesResponse`
```python
class AnthropicMessagesResponse(BaseModel):
    """Anthropic Messages API response"""

    id: str
    type: Literal["message"] = "message"
    role: Literal["assistant"] = "assistant"
    content: list[AnthropicContentBlock]
    model: str
    stop_reason: (
        Literal["end_turn", "max_tokens", "stop_sequence", "tool_use"] | None
    ) = None
    stop_sequence: str | None = None
    usage: AnthropicUsage | None = None

    # vLLM-specific fields that are not in Anthropic spec
    kv_transfer_params: dict[str, Any] | None = Field(
        default=None, description="KVTransfer parameters."
    )

    def model_post_init(self, __context):
        if not self.id:
            self.id = f"msg_{int(time.time() * 1000)}"
```
**EN:** Class `AnthropicMessagesResponse` is introduced here. Its docstring describes the intent as: Anthropic Messages API response
**CN:** 这里定义类 `AnthropicMessagesResponse`。其文档字符串说明了该类的职责与使用方式。

### Lines 211-214 — Class `AnthropicContextManagement`
```python
class AnthropicContextManagement(BaseModel):
    """Context management information for token counting."""

    original_input_tokens: int
```
**EN:** Class `AnthropicContextManagement` is introduced here. Its docstring describes the intent as: Context management information for token counting.
**CN:** 这里定义类 `AnthropicContextManagement`。其文档字符串说明了该类的职责与使用方式。

### Lines 217-240 — Class `AnthropicCountTokensRequest`
```python
class AnthropicCountTokensRequest(BaseModel):
    """Anthropic messages.count_tokens request"""

    model: str
    messages: list[AnthropicMessage]
    system: str | list[AnthropicContentBlock] | None = None
    tool_choice: AnthropicToolChoice | None = None
    tools: list[AnthropicTool] | None = None

    # vLLM-specific fields that are not in Anthropic spec
    chat_template_kwargs: dict[str, Any] | None = Field(
        default=None,
        description=(
            "Additional keyword args to pass to the chat template renderer. "
            "Will be accessible by the template."
        ),
    )

    @field_validator("model")
    @classmethod
    def validate_model(cls, v):
        if not v:
            raise ValueError("Model is required")
        return v
```
**EN:** Class `AnthropicCountTokensRequest` is introduced here. Its docstring describes the intent as: Anthropic messages.count_tokens request
**CN:** 这里定义类 `AnthropicCountTokensRequest`。其文档字符串说明了该类的职责与使用方式。

### Lines 243-247 — Class `AnthropicCountTokensResponse`
```python
class AnthropicCountTokensResponse(BaseModel):
    """Anthropic messages.count_tokens response"""

    input_tokens: int
    context_management: AnthropicContextManagement | None = None
```
**EN:** Class `AnthropicCountTokensResponse` is introduced here. Its docstring describes the intent as: Anthropic messages.count_tokens response
**CN:** 这里定义类 `AnthropicCountTokensResponse`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Anthropic-compatible API surface / Anthropic 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `typing`
- **Third-party / 第三方**: `pydantic`
