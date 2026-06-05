# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/anthropic/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: Anthropic-compatible. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：Anthropic 兼容。

## Line-by-Line Analysis / 逐行分析
### Lines 5-25 — Imports and shared dependencies
```python
from http import HTTPStatus

from fastapi import APIRouter, Depends, FastAPI, Request
from fastapi.responses import JSONResponse, StreamingResponse

from vllm.entrypoints.anthropic.protocol import (
    AnthropicCountTokensRequest,
    AnthropicCountTokensResponse,
    AnthropicError,
    AnthropicErrorResponse,
    AnthropicMessagesRequest,
    AnthropicMessagesResponse,
)
from vllm.entrypoints.anthropic.serving import AnthropicServingMessages
from vllm.entrypoints.openai.engine.protocol import ErrorResponse
from vllm.entrypoints.openai.utils import validate_json_request
from vllm.entrypoints.utils import (
    load_aware_call,
    with_cancellation,
)
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `http`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.entrypoints.anthropic.protocol`, `vllm.entrypoints.anthropic.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger`.
**CN:** 该导入块引入 `http` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.entrypoints.anthropic.protocol`, `vllm.entrypoints.anthropic.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger` 等 vLLM 内部模块。

### Lines 27-29 — Shared module state
```python
logger = init_logger(__name__)

router = APIRouter()
```
**EN:** This block initializes `logger`, `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 32-33 — Function `messages`
```python
def messages(request: Request) -> AnthropicServingMessages:
    return request.app.state.anthropic_serving_messages
```
**EN:** This function `messages` implements the `messages` step within the module flow.
**CN:** 该函数 `messages` 实现了模块流程中的“messages”步骤。

### Lines 36-45 — Function `translate_error_response`
```python
def translate_error_response(response: ErrorResponse) -> JSONResponse:
    anthropic_error = AnthropicErrorResponse(
        error=AnthropicError(
            type=response.error.type,
            message=response.error.message,
        )
    )
    return JSONResponse(
        status_code=response.error.code, content=anthropic_error.model_dump()
    )
```
**EN:** This function `translate_error_response` serializes API responses.
**CN:** 该函数 `translate_error_response` 序列化 API 响应。

### Lines 60-91 — Function `create_messages`
```python
async def create_messages(request: AnthropicMessagesRequest, raw_request: Request):
    handler = messages(raw_request)
    if handler is None:
        base_server = raw_request.app.state.openai_serving_tokenization
        error = base_server.create_error_response(
            NotImplementedError("The model does not support Messages API")
        )
        return translate_error_response(error)

    try:
        generator = await handler.create_messages(request, raw_request)
    except Exception as e:
        logger.exception("Error in create_messages: %s", e)
        return JSONResponse(
            status_code=HTTPStatus.INTERNAL_SERVER_ERROR.value,
            content=AnthropicErrorResponse(
                error=AnthropicError(
                    type="internal_error",
                    message=str(e),
                )
            ).model_dump(),
        )

    if isinstance(generator, ErrorResponse):
        return translate_error_response(generator)

    elif isinstance(generator, AnthropicMessagesResponse):
        resp = generator.model_dump(exclude_none=True)
        logger.debug("Anthropic Messages Response: %s", resp)
        return JSONResponse(content=resp)

    return StreamingResponse(content=generator, media_type="text/event-stream")
```
**EN:** This async function `create_messages` serializes API responses.
**CN:** 该异步函数 `create_messages` 序列化 API 响应。

### Lines 106-132 — Function `count_tokens`
```python
async def count_tokens(request: AnthropicCountTokensRequest, raw_request: Request):
    handler = messages(raw_request)
    if handler is None:
        base_server = raw_request.app.state.openai_serving_tokenization
        error = base_server.create_error_response(
            NotImplementedError("The model does not support Messages API")
        )
        return translate_error_response(error)

    try:
        response = await handler.count_tokens(request, raw_request)
    except Exception as e:
        logger.exception("Error in count_tokens: %s", e)
        return JSONResponse(
            status_code=HTTPStatus.INTERNAL_SERVER_ERROR.value,
            content=AnthropicErrorResponse(
                error=AnthropicError(
                    type="internal_error",
                    message=str(e),
                )
            ).model_dump(),
        )

    if isinstance(response, ErrorResponse):
        return translate_error_response(response)

    return JSONResponse(content=response.model_dump(exclude_none=True))
```
**EN:** This async function `count_tokens` serializes API responses.
**CN:** 该异步函数 `count_tokens` 序列化 API 响应。

### Lines 135-136 — Function `attach_router`
```python
def attach_router(app: FastAPI):
    app.include_router(router)
```
**EN:** This function `attach_router` implements the `attaches router` step within the module flow.
**CN:** 该函数 `attach_router` 实现了模块流程中的“挂载路由”步骤。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Router registration / 路由注册
- Async request handling / 异步请求处理
- Streaming responses / 流式响应
- Tokenization or token-level processing / 分词或 token 级处理
- Anthropic-compatible API surface / Anthropic 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `http`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.anthropic.protocol`, `vllm.entrypoints.anthropic.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger`
