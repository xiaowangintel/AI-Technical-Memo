# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/anthropic/protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements protocol logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 protocol 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Provide supporting module logic / 提供辅助模块逻辑
```python
"""Pydantic models for Anthropic Messages API protocol"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 3-4: Import runtime dependencies / 导入运行时依赖
```python
import uuid
from typing import Any, Literal, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-6: Import runtime dependencies / 导入运行时依赖
```python
from pydantic import BaseModel, Field, field_validator
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-13: Define class AnthropicError / 定义类 AnthropicError
```python
class AnthropicError(BaseModel):
    """Error structure for Anthropic API"""

    type: str
    message: str
```
**EN:** This block declares the class `AnthropicError`. It centers on Error structure for Anthropic API.
**CN:** 该代码块声明类 `AnthropicError`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 16-20: Define class AnthropicErrorResponse / 定义类 AnthropicErrorResponse
```python
class AnthropicErrorResponse(BaseModel):
    """Error response structure for Anthropic API"""

    type: Literal["error"] = "error"
    error: AnthropicError
```
**EN:** This block declares the class `AnthropicErrorResponse`. It centers on Error response structure for Anthropic API.
**CN:** 该代码块声明类 `AnthropicErrorResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 23-29: Define class AnthropicUsage / 定义类 AnthropicUsage
```python
class AnthropicUsage(BaseModel):
    """Token usage information"""

    input_tokens: int
    output_tokens: int
    cache_creation_input_tokens: Optional[int] = None
    cache_read_input_tokens: Optional[int] = None
```
**EN:** This block declares the class `AnthropicUsage`. It centers on Token usage information.
**CN:** 该代码块声明类 `AnthropicUsage`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 32-56: Define class AnthropicContentBlock / 定义类 AnthropicContentBlock
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
    text: Optional[str] = None
    # For image content
    source: Optional[dict[str, Any]] = None
    # For tool use/result
    id: Optional[str] = None
    tool_use_id: Optional[str] = None
    name: Optional[str] = None
    input: Optional[dict[str, Any]] = None
    content: Optional[str | list[dict[str, Any]]] = None
    is_error: Optional[bool] = None
    # For thinking content
    thinking: Optional[str] = None
    signature: Optional[str] = None
```
**EN:** This block declares the class `AnthropicContentBlock`. It centers on Content block in message.
**CN:** 该代码块声明类 `AnthropicContentBlock`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 59-63: Define class AnthropicMessage / 定义类 AnthropicMessage
```python
class AnthropicMessage(BaseModel):
    """Message structure"""

    role: Literal["user", "assistant"]
    content: str | list[AnthropicContentBlock]
```
**EN:** This block declares the class `AnthropicMessage`. It centers on Message structure.
**CN:** 该代码块声明类 `AnthropicMessage`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 66-81: Define class AnthropicTool / 定义类 AnthropicTool
```python
class AnthropicTool(BaseModel):
    """Tool definition"""

    name: str
    description: Optional[str] = None
    input_schema: dict[str, Any]
    defer_loading: Optional[bool] = None

    @field_validator("input_schema")
    @classmethod
    def validate_input_schema(cls, v):
        if not isinstance(v, dict):
            raise ValueError("input_schema must be a dictionary")
        if "type" not in v:
            v["type"] = "object"
        return v
```
**EN:** This block declares the class `AnthropicTool`. It centers on Tool definition, with methods such as validate_input_schema.
**CN:** 该代码块声明类 `AnthropicTool`。它负责承载与 protocol 相关的核心状态与行为，并通过 validate_input_schema 等方法组织实现。

### Lines 84-88: Define class AnthropicToolChoice / 定义类 AnthropicToolChoice
```python
class AnthropicToolChoice(BaseModel):
    """Tool Choice definition"""

    type: Literal["auto", "any", "tool", "none"]
    name: Optional[str] = None
```
**EN:** This block declares the class `AnthropicToolChoice`. It centers on Tool Choice definition.
**CN:** 该代码块声明类 `AnthropicToolChoice`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 91-98: Define class AnthropicCountTokensRequest / 定义类 AnthropicCountTokensRequest
```python
class AnthropicCountTokensRequest(BaseModel):
    """Anthropic Count Tokens API request"""

    model: str
    messages: list[AnthropicMessage]
    system: Optional[str | list[AnthropicContentBlock]] = None
    tool_choice: Optional[AnthropicToolChoice] = None
    tools: Optional[list[AnthropicTool]] = None
```
**EN:** This block declares the class `AnthropicCountTokensRequest`. It centers on Anthropic Count Tokens API request.
**CN:** 该代码块声明类 `AnthropicCountTokensRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 101-104: Define class AnthropicCountTokensResponse / 定义类 AnthropicCountTokensResponse
```python
class AnthropicCountTokensResponse(BaseModel):
    """Anthropic Count Tokens API response"""

    input_tokens: int
```
**EN:** This block declares the class `AnthropicCountTokensResponse`. It centers on Anthropic Count Tokens API response.
**CN:** 该代码块声明类 `AnthropicCountTokensResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 107-135: Define class AnthropicMessagesRequest / 定义类 AnthropicMessagesRequest
```python
class AnthropicMessagesRequest(BaseModel):
    """Anthropic Messages API request"""

    model: str
    messages: list[AnthropicMessage]
    max_tokens: int
    metadata: Optional[dict[str, Any]] = None
    stop_sequences: Optional[list[str]] = None
    stream: Optional[bool] = False
    system: Optional[str | list[AnthropicContentBlock]] = None
    temperature: Optional[float] = None
    tool_choice: Optional[AnthropicToolChoice] = None
    tools: Optional[list[AnthropicTool]] = None
    top_k: Optional[int] = None
    top_p: Optional[float] = None

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
**EN:** This block declares the class `AnthropicMessagesRequest`. It centers on Anthropic Messages API request, with methods such as validate_model, validate_max_tokens.
**CN:** 该代码块声明类 `AnthropicMessagesRequest`。它负责承载与 protocol 相关的核心状态与行为，并通过 validate_model, validate_max_tokens 等方法组织实现。

### Lines 138-149: Define class AnthropicDelta / 定义类 AnthropicDelta
```python
class AnthropicDelta(BaseModel):
    """Delta for streaming responses"""

    type: Optional[Literal["text_delta", "input_json_delta"]] = None
    text: Optional[str] = None
    partial_json: Optional[str] = None

    # Message delta fields
    stop_reason: Optional[
        Literal["end_turn", "max_tokens", "stop_sequence", "tool_use"]
    ] = None
    stop_sequence: Optional[str] = None
```
**EN:** This block declares the class `AnthropicDelta`. It centers on Delta for streaming responses.
**CN:** 该代码块声明类 `AnthropicDelta`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 152-170: Define class AnthropicStreamEvent / 定义类 AnthropicStreamEvent
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
    message: Optional["AnthropicMessagesResponse"] = None
    delta: Optional[AnthropicDelta] = None
    content_block: Optional[AnthropicContentBlock] = None
    index: Optional[int] = None
    error: Optional[AnthropicError] = None
    usage: Optional[AnthropicUsage] = None
```
**EN:** This block declares the class `AnthropicStreamEvent`. It centers on Streaming event.
**CN:** 该代码块声明类 `AnthropicStreamEvent`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 173-185: Define class AnthropicMessagesResponse / 定义类 AnthropicMessagesResponse
```python
class AnthropicMessagesResponse(BaseModel):
    """Anthropic Messages API response"""

    id: str = Field(default_factory=lambda: f"msg_{uuid.uuid4().hex}")
    type: Literal["message"] = "message"
    role: Literal["assistant"] = "assistant"
    content: list[AnthropicContentBlock]
    model: str
    stop_reason: Optional[
        Literal["end_turn", "max_tokens", "stop_sequence", "tool_use"]
    ] = None
    stop_sequence: Optional[str] = None
    usage: Optional[AnthropicUsage] = None
```
**EN:** This block declares the class `AnthropicMessagesResponse`. It centers on Anthropic Messages API response.
**CN:** 该代码块声明类 `AnthropicMessagesResponse`。它负责承载与 protocol 相关的核心状态与行为。

## Key Concepts / 关键概念
- **Core types / 核心类型**: AnthropicError, AnthropicErrorResponse, AnthropicUsage, AnthropicContentBlock, AnthropicMessage, AnthropicTool, ...
- **Domain focus / 领域焦点**: protocol / protocol
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing, uuid
- **Third-party / 第三方库**: pydantic
- **Local Modules / 本地模块**: None / 无
