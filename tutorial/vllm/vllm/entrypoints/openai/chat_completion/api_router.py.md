# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/chat_completion/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: OpenAI-compatible chat completion. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：OpenAI 兼容 / 对话补全。

## Line-by-Line Analysis / 逐行分析
### Lines 5-24 — Imports and shared dependencies
```python
from http import HTTPStatus

from fastapi import APIRouter, Depends, FastAPI, Request
from fastapi.responses import JSONResponse, StreamingResponse

from vllm.entrypoints.openai.chat_completion.batch_serving import OpenAIServingChatBatch
from vllm.entrypoints.openai.chat_completion.protocol import (
    BatchChatCompletionRequest,
    ChatCompletionRequest,
    ChatCompletionResponse,
)
from vllm.entrypoints.openai.chat_completion.serving import OpenAIServingChat
from vllm.entrypoints.openai.engine.protocol import ErrorResponse
from vllm.entrypoints.openai.orca_metrics import metrics_header
from vllm.entrypoints.openai.utils import validate_json_request
from vllm.entrypoints.utils import (
    load_aware_call,
    with_cancellation,
)
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `http`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.entrypoints.openai.chat_completion.batch_serving`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.chat_completion.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.orca_metrics`, `vllm.entrypoints.openai.utils`.
**CN:** 该导入块引入 `http` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.entrypoints.openai.chat_completion.batch_serving`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.chat_completion.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.orca_metrics`, `vllm.entrypoints.openai.utils` 等 vLLM 内部模块。

### Lines 26-29 — Module constants
```python
logger = init_logger(__name__)

router = APIRouter()
ENDPOINT_LOAD_METRICS_FORMAT_HEADER_LABEL = "endpoint-load-metrics-format"
```
**EN:** This block initializes `logger`, `router`, `ENDPOINT_LOAD_METRICS_FORMAT_HEADER_LABEL`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `router`, `ENDPOINT_LOAD_METRICS_FORMAT_HEADER_LABEL`，为后续逻辑准备模块级常量或共享状态。

### Lines 32-33 — Function `chat`
```python
def chat(request: Request) -> OpenAIServingChat | None:
    return request.app.state.openai_serving_chat
```
**EN:** This function `chat` implements the `chat` step within the module flow.
**CN:** 该函数 `chat` 实现了模块流程中的“对话”步骤。

### Lines 36-37 — Function `batch_chat`
```python
def batch_chat(request: Request) -> OpenAIServingChatBatch | None:
    return request.app.state.openai_serving_chat_batch
```
**EN:** This function `batch_chat` implements the `batch chat` step within the module flow.
**CN:** 该函数 `batch_chat` 实现了模块流程中的“批处理对话”步骤。

### Lines 53-74 — Function `create_chat_completion`
```python
async def create_chat_completion(request: ChatCompletionRequest, raw_request: Request):
    metrics_header_format = raw_request.headers.get(
        ENDPOINT_LOAD_METRICS_FORMAT_HEADER_LABEL, ""
    )
    handler = chat(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Chat Completions API")

    generator = await handler.create_chat_completion(request, raw_request)

    if isinstance(generator, ErrorResponse):
        return JSONResponse(
            content=generator.model_dump(), status_code=generator.error.code
        )

    elif isinstance(generator, ChatCompletionResponse):
        return JSONResponse(
            content=generator.model_dump(),
            headers=metrics_header(metrics_header_format),
        )

    return StreamingResponse(content=generator, media_type="text/event-stream")
```
**EN:** This async function `create_chat_completion` serializes API responses.
**CN:** 该异步函数 `create_chat_completion` 序列化 API 响应。

### Lines 90-102 — Function `create_batch_chat_completion`
```python
async def create_batch_chat_completion(
    request: BatchChatCompletionRequest, raw_request: Request
):
    handler = batch_chat(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Chat Completions API")

    result = await handler.create_batch_chat_completion(request, raw_request)

    if isinstance(result, ErrorResponse):
        return JSONResponse(content=result.model_dump(), status_code=result.error.code)

    return JSONResponse(content=result.model_dump())
```
**EN:** This async function `create_batch_chat_completion` serializes API responses.
**CN:** 该异步函数 `create_batch_chat_completion` 序列化 API 响应。

### Lines 105-106 — Function `attach_router`
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
- Metrics and observability / 指标与可观测性
- Batch workflow handling / 批处理工作流
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `http`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.chat_completion.batch_serving`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.chat_completion.serving`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.orca_metrics`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.utils`, `vllm.logger`
