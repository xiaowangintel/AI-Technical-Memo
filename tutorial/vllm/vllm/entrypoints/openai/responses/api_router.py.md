# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: OpenAI-compatible responses. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 5-23 — Imports and shared dependencies
```python
from collections.abc import AsyncGenerator
from http import HTTPStatus

from fastapi import APIRouter, Depends, FastAPI, Request
from fastapi.responses import JSONResponse, StreamingResponse

from vllm.entrypoints.openai.engine.protocol import ErrorResponse
from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
    ResponsesResponse,
    StreamingResponsesResponse,
)
from vllm.entrypoints.openai.responses.serving import OpenAIServingResponses
from vllm.entrypoints.openai.utils import validate_json_request
from vllm.entrypoints.utils import (
    load_aware_call,
    with_cancellation,
)
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `collections`, `http`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.entrypoints.openai.responses.serving`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger`.
**CN:** 该导入块引入 `collections`, `http` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.entrypoints.openai.responses.serving`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger` 等 vLLM 内部模块。

### Lines 25-27 — Shared module state
```python
logger = init_logger(__name__)

router = APIRouter()
```
**EN:** This block initializes `logger`, `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 30-31 — Function `responses`
```python
def responses(request: Request) -> OpenAIServingResponses | None:
    return request.app.state.openai_serving_responses
```
**EN:** This function `responses` implements the `responses` step within the module flow.
**CN:** 该函数 `responses` 实现了模块流程中的“responses”步骤。

### Lines 34-45 — Function `_convert_stream_to_sse_events`
```python
async def _convert_stream_to_sse_events(
    generator: AsyncGenerator[StreamingResponsesResponse, None],
) -> AsyncGenerator[str, None]:
    """Convert the generator to a stream of events in SSE format"""
    async for event in generator:
        event_type = getattr(event, "type", "unknown")
        # https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event_stream_format
        event_data = (
            f"event: {event_type}\ndata: "
            f"{event.model_dump_json(indent=None, by_alias=True)}\n\n"
        )
        yield event_data
```
**EN:** This async function `_convert_stream_to_sse_events` is documented as: Convert the generator to a stream of events in SSE format
**CN:** 这里定义异步函数 `_convert_stream_to_sse_events`，其文档字符串说明了主要职责与调用约定。

### Lines 60-77 — Function `create_responses`
```python
async def create_responses(request: ResponsesRequest, raw_request: Request):
    handler = responses(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Responses API")

    generator = await handler.create_responses(request, raw_request)

    if isinstance(generator, ErrorResponse):
        return JSONResponse(
            content=generator.model_dump(mode="json", by_alias=True),
            status_code=generator.error.code,
        )
    elif isinstance(generator, ResponsesResponse):
        return JSONResponse(content=generator.model_dump(mode="json", by_alias=True))

    return StreamingResponse(
        content=_convert_stream_to_sse_events(generator), media_type="text/event-stream"
    )
```
**EN:** This async function `create_responses` serializes API responses.
**CN:** 该异步函数 `create_responses` 序列化 API 响应。

### Lines 82-107 — Function `retrieve_responses`
```python
async def retrieve_responses(
    response_id: str,
    raw_request: Request,
    starting_after: int | None = None,
    stream: bool | None = False,
):
    handler = responses(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Responses API")

    response = await handler.retrieve_responses(
        response_id,
        starting_after=starting_after,
        stream=stream,
    )

    if isinstance(response, ErrorResponse):
        return JSONResponse(
            content=response.model_dump(mode="json", by_alias=True),
            status_code=response.error.code,
        )
    elif isinstance(response, ResponsesResponse):
        return JSONResponse(content=response.model_dump(mode="json", by_alias=True))
    return StreamingResponse(
        content=_convert_stream_to_sse_events(response), media_type="text/event-stream"
    )
```
**EN:** This async function `retrieve_responses` serializes API responses.
**CN:** 该异步函数 `retrieve_responses` 序列化 API 响应。

### Lines 112-124 — Function `cancel_responses`
```python
async def cancel_responses(response_id: str, raw_request: Request):
    handler = responses(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Responses API")

    response = await handler.cancel_responses(response_id)

    if isinstance(response, ErrorResponse):
        return JSONResponse(
            content=response.model_dump(mode="json", by_alias=True),
            status_code=response.error.code,
        )
    return JSONResponse(content=response.model_dump(mode="json", by_alias=True))
```
**EN:** This async function `cancel_responses` serializes API responses.
**CN:** 该异步函数 `cancel_responses` 序列化 API 响应。

### Lines 127-128 — Function `attach_router`
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
- Incremental result emission / 增量结果输出
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections`, `http`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.entrypoints.openai.responses.serving`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger`
