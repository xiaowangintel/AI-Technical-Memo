# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/tokenize/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: serving tokenization. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：服务扩展 / 分词。

## Line-by-Line Analysis / 逐行分析
### Lines 5-26 — Imports and shared dependencies
```python
from http import HTTPStatus

from fastapi import APIRouter, Depends, FastAPI, HTTPException, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from typing_extensions import assert_never

from vllm.entrypoints.openai.engine.protocol import (
    ErrorResponse,
)
from vllm.entrypoints.openai.utils import validate_json_request
from vllm.entrypoints.serve.tokenize.protocol import (
    DetokenizeRequest,
    DetokenizeResponse,
    TokenizeRequest,
    TokenizeResponse,
)
from vllm.entrypoints.serve.tokenize.serving import OpenAIServingTokenization
from vllm.entrypoints.utils import (
    with_cancellation,
)
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `http`, uses third-party packages like `fastapi`, `typing_extensions`, depends on internal helpers such as `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.tokenize.protocol`, `vllm.entrypoints.serve.tokenize.serving`, `vllm.entrypoints.utils`, `vllm.logger`.
**CN:** 该导入块引入 `http` 等标准库模块，使用 `fastapi`, `typing_extensions` 等第三方库，依赖 `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.tokenize.protocol`, `vllm.entrypoints.serve.tokenize.serving`, `vllm.entrypoints.utils`, `vllm.logger` 等 vLLM 内部模块。

### Lines 28-28 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 31-32 — Function `tokenization`
```python
def tokenization(request: Request) -> OpenAIServingTokenization:
    return request.app.state.openai_serving_tokenization
```
**EN:** This function `tokenization` implements the `tokenization` step within the module flow.
**CN:** 该函数 `tokenization` 实现了模块流程中的“分词”步骤。

### Lines 35-35 — Shared module state
```python
router = APIRouter()
```
**EN:** This block initializes `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 49-61 — Function `tokenize`
```python
async def tokenize(request: TokenizeRequest, raw_request: Request):
    handler = tokenization(raw_request)

    generator = await handler.create_tokenize(request, raw_request)

    if isinstance(generator, ErrorResponse):
        return JSONResponse(
            content=generator.model_dump(), status_code=generator.error.code
        )
    elif isinstance(generator, TokenizeResponse):
        return JSONResponse(content=generator.model_dump())

    assert_never(generator)
```
**EN:** This async function `tokenize` serializes API responses.
**CN:** 该异步函数 `tokenize` 序列化 API 响应。

### Lines 74-93 — Function `detokenize`
```python
async def detokenize(request: DetokenizeRequest, raw_request: Request):
    handler = tokenization(raw_request)

    try:
        generator = await handler.create_detokenize(request, raw_request)
    except OverflowError as e:
        raise RequestValidationError(errors=[str(e)]) from e
    except Exception as e:
        raise HTTPException(
            status_code=HTTPStatus.INTERNAL_SERVER_ERROR.value, detail=str(e)
        ) from e

    if isinstance(generator, ErrorResponse):
        return JSONResponse(
            content=generator.model_dump(), status_code=generator.error.code
        )
    elif isinstance(generator, DetokenizeResponse):
        return JSONResponse(content=generator.model_dump())

    assert_never(generator)
```
**EN:** This async function `detokenize` serializes API responses.
**CN:** 该异步函数 `detokenize` 序列化 API 响应。

### Lines 96-111 — Function `attach_router`
```python
def attach_router(app: FastAPI):
    if getattr(app.state.args, "enable_tokenizer_info_endpoint", False):
        """Conditionally register the tokenizer info endpoint if enabled."""

        @router.get("/tokenizer_info")
        async def get_tokenizer_info(raw_request: Request):
            """Get comprehensive tokenizer information."""
            result = await tokenization(raw_request).get_tokenizer_info()
            return JSONResponse(
                content=result.model_dump(),
                status_code=result.error.code
                if isinstance(result, ErrorResponse)
                else 200,
            )

    app.include_router(router)
```
**EN:** This function `attach_router` serializes API responses.
**CN:** 该函数 `attach_router` 序列化 API 响应。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Router registration / 路由注册
- Async request handling / 异步请求处理
- Tokenization or token-level processing / 分词或 token 级处理
- Auxiliary serving endpoints / 辅助服务端点

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `http`
- **Third-party / 第三方**: `fastapi`, `typing_extensions`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.tokenize.protocol`, `vllm.entrypoints.serve.tokenize.serving`, `vllm.entrypoints.utils`, `vllm.logger`
