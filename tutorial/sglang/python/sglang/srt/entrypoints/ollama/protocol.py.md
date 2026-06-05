# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/ollama/protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements protocol logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 protocol 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6: Provide supporting module logic / 提供辅助模块逻辑
```python
"""
Ollama-compatible API protocol definitions.

These models match the Ollama API format:
https://github.com/ollama/ollama/blob/main/docs/api.md
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 8-8: Import runtime dependencies / 导入运行时依赖
```python
from typing import Any, Dict, List, Literal, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 10-10: Import runtime dependencies / 导入运行时依赖
```python
from pydantic import BaseModel, Field
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 13-18: Define class OllamaMessage / 定义类 OllamaMessage
```python
class OllamaMessage(BaseModel):
    """Ollama message format."""

    role: str
    content: str
    images: Optional[List[str]] = None
```
**EN:** This block declares the class `OllamaMessage`. It centers on Ollama message format..
**CN:** 该代码块声明类 `OllamaMessage`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 21-30: Define class OllamaChatRequest / 定义类 OllamaChatRequest
```python
class OllamaChatRequest(BaseModel):
    """Ollama /api/chat request format."""

    model: str
    messages: List[OllamaMessage]
    stream: bool = True
    format: Optional[Union[Literal["json"], Dict[str, Any]]] = None
    options: Optional[Dict[str, Any]] = None
    keep_alive: Optional[Union[float, str]] = None
    think: Optional[Union[bool, Literal["low", "medium", "high"]]] = None
```
**EN:** This block declares the class `OllamaChatRequest`. It centers on Ollama /api/chat request format..
**CN:** 该代码块声明类 `OllamaChatRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 33-46: Define class OllamaChatResponse / 定义类 OllamaChatResponse
```python
class OllamaChatResponse(BaseModel):
    """Ollama /api/chat response format (non-streaming)."""

    model: str
    created_at: str
    message: OllamaMessage
    done: bool = True
    done_reason: Optional[str] = "stop"
    total_duration: Optional[int] = None
    load_duration: Optional[int] = None
    prompt_eval_count: Optional[int] = None
    prompt_eval_duration: Optional[int] = None
    eval_count: Optional[int] = None
    eval_duration: Optional[int] = None
```
**EN:** This block declares the class `OllamaChatResponse`. It centers on Ollama /api/chat response format (non-streaming)..
**CN:** 该代码块声明类 `OllamaChatResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 49-56: Define class OllamaChatStreamResponse / 定义类 OllamaChatStreamResponse
```python
class OllamaChatStreamResponse(BaseModel):
    """Ollama /api/chat streaming response chunk."""

    model: str
    created_at: str
    message: OllamaMessage
    done: bool = False
    done_reason: Optional[str] = None
```
**EN:** This block declares the class `OllamaChatStreamResponse`. It centers on Ollama /api/chat streaming response chunk..
**CN:** 该代码块声明类 `OllamaChatStreamResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 59-74: Define class OllamaGenerateRequest / 定义类 OllamaGenerateRequest
```python
class OllamaGenerateRequest(BaseModel):
    """Ollama /api/generate request format."""

    model: str
    prompt: str
    suffix: Optional[str] = None
    system: Optional[str] = None
    template: Optional[str] = None
    context: Optional[List[int]] = None
    stream: bool = True
    raw: bool = False
    format: Optional[Union[Literal["json"], Dict[str, Any]]] = None
    options: Optional[Dict[str, Any]] = None
    keep_alive: Optional[Union[float, str]] = None
    images: Optional[List[str]] = None
    think: Optional[bool] = None
```
**EN:** This block declares the class `OllamaGenerateRequest`. It centers on Ollama /api/generate request format..
**CN:** 该代码块声明类 `OllamaGenerateRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 77-91: Define class OllamaGenerateResponse / 定义类 OllamaGenerateResponse
```python
class OllamaGenerateResponse(BaseModel):
    """Ollama /api/generate response format (non-streaming)."""

    model: str
    created_at: str
    response: str
    done: bool = True
    done_reason: Optional[str] = "stop"
    context: Optional[List[int]] = None
    total_duration: Optional[int] = None
    load_duration: Optional[int] = None
    prompt_eval_count: Optional[int] = None
    prompt_eval_duration: Optional[int] = None
    eval_count: Optional[int] = None
    eval_duration: Optional[int] = None
```
**EN:** This block declares the class `OllamaGenerateResponse`. It centers on Ollama /api/generate response format (non-streaming)..
**CN:** 该代码块声明类 `OllamaGenerateResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 94-101: Define class OllamaGenerateStreamResponse / 定义类 OllamaGenerateStreamResponse
```python
class OllamaGenerateStreamResponse(BaseModel):
    """Ollama /api/generate streaming response chunk."""

    model: str
    created_at: str
    response: str
    done: bool = False
    done_reason: Optional[str] = None
```
**EN:** This block declares the class `OllamaGenerateStreamResponse`. It centers on Ollama /api/generate streaming response chunk..
**CN:** 该代码块声明类 `OllamaGenerateStreamResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 104-112: Define class OllamaModelInfo / 定义类 OllamaModelInfo
```python
class OllamaModelInfo(BaseModel):
    """Model information for /api/tags response."""

    name: str
    model: str
    modified_at: str
    size: int
    digest: str
    details: Optional[Dict[str, Any]] = None
```
**EN:** This block declares the class `OllamaModelInfo`. It centers on Model information for /api/tags response..
**CN:** 该代码块声明类 `OllamaModelInfo`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 115-118: Define class OllamaTagsResponse / 定义类 OllamaTagsResponse
```python
class OllamaTagsResponse(BaseModel):
    """Ollama /api/tags response format."""

    models: List[OllamaModelInfo]
```
**EN:** This block declares the class `OllamaTagsResponse`. It centers on Ollama /api/tags response format..
**CN:** 该代码块声明类 `OllamaTagsResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 121-124: Define class OllamaShowRequest / 定义类 OllamaShowRequest
```python
class OllamaShowRequest(BaseModel):
    """Ollama /api/show request format."""

    model: str
```
**EN:** This block declares the class `OllamaShowRequest`. It centers on Ollama /api/show request format..
**CN:** 该代码块声明类 `OllamaShowRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 127-137: Define class OllamaShowResponse / 定义类 OllamaShowResponse
```python
class OllamaShowResponse(BaseModel):
    """Ollama /api/show response format."""

    license: str = ""
    modelfile: str = ""
    parameters: str = ""
    template: str = ""
    modified_at: str = ""
    details: Dict[str, Any] = Field(default_factory=dict)
    model_info: Dict[str, Any] = Field(default_factory=dict)
    capabilities: List[str] = Field(default_factory=list)
```
**EN:** This block declares the class `OllamaShowResponse`. It centers on Ollama /api/show response format..
**CN:** 该代码块声明类 `OllamaShowResponse`。它负责承载与 protocol 相关的核心状态与行为。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OllamaMessage, OllamaChatRequest, OllamaChatResponse, OllamaChatStreamResponse, OllamaGenerateRequest, OllamaGenerateResponse, ...
- **Domain focus / 领域焦点**: protocol / protocol
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing
- **Third-party / 第三方库**: pydantic
- **Local Modules / 本地模块**: None / 无
