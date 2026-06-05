# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/translation/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: speech-to-text translation. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：语音转文本 / 翻译。

## Line-by-Line Analysis / 逐行分析
### Lines 5-19 — Imports and shared dependencies
```python
from http import HTTPStatus
from typing import Annotated

from fastapi import APIRouter, Form, Request
from fastapi.responses import JSONResponse, StreamingResponse

from vllm.entrypoints.openai.engine.protocol import ErrorResponse
from vllm.entrypoints.utils import (
    load_aware_call,
    with_cancellation,
)
from vllm.logger import init_logger

from .protocol import TranslationRequest, TranslationResponseVariant
from .serving import OpenAIServingTranslation
```
**EN:** This import block pulls in standard-library modules such as `http`, `typing`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.utils`, `vllm.logger`, `.protocol`, `.serving`.
**CN:** 该导入块引入 `http`, `typing` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.utils`, `vllm.logger`, `.protocol`, `.serving` 等 vLLM 内部模块。

### Lines 21-23 — Shared module state
```python
logger = init_logger(__name__)

router = APIRouter()
```
**EN:** This block initializes `logger`, `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 26-27 — Function `translation`
```python
def translation(request: Request) -> OpenAIServingTranslation:
    return request.app.state.openai_serving_translation
```
**EN:** This function `translation` implements the `translation` step within the module flow.
**CN:** 该函数 `translation` 实现了模块流程中的“翻译”步骤。

### Lines 41-60 — Function `create_translations`
```python
async def create_translations(
    request: Annotated[TranslationRequest, Form()], raw_request: Request
):
    handler = translation(raw_request)
    if handler is None:
        raise NotImplementedError("The model does not support Translations API")

    audio_data = await request.file.read()

    generator = await handler.create_translation(audio_data, request, raw_request)

    if isinstance(generator, ErrorResponse):
        return JSONResponse(
            content=generator.model_dump(), status_code=generator.error.code
        )

    elif isinstance(generator, TranslationResponseVariant):
        return JSONResponse(content=generator.model_dump())

    return StreamingResponse(content=generator, media_type="text/event-stream")
```
**EN:** This async function `create_translations` serializes API responses.
**CN:** 该异步函数 `create_translations` 序列化 API 响应。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Router registration / 路由注册
- Async request handling / 异步请求处理
- Streaming responses / 流式响应
- Speech translation flow / 语音翻译流程
- Speech-to-text serving stack / 语音转文本服务栈

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `http`, `typing`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.utils`, `vllm.logger`, `.protocol`, `.serving`
