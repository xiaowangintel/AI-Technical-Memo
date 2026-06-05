# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/elastic_ep/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: serving elastic endpoint. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：服务扩展 / 弹性端点。

## Line-by-Line Analysis / 逐行分析
### Lines 5-20 — Imports and shared dependencies
```python
import json
from http import HTTPStatus

from fastapi import APIRouter, Depends, FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse

from vllm.engine.protocol import EngineClient
from vllm.entrypoints.openai.engine.protocol import (
    ErrorResponse,
)
from vllm.entrypoints.openai.utils import validate_json_request
from vllm.entrypoints.serve.elastic_ep.middleware import (
    get_scaling_elastic_ep,
    set_scaling_elastic_ep,
)
from vllm.logger import init_logger
```
**EN:** This import block pulls in standard-library modules such as `json`, `http`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.elastic_ep.middleware`, `vllm.logger`.
**CN:** 该导入块引入 `json`, `http` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.elastic_ep.middleware`, `vllm.logger` 等 vLLM 内部模块。

### Lines 22-22 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 25-26 — Function `engine_client`
```python
def engine_client(request: Request) -> EngineClient:
    return request.app.state.engine_client
```
**EN:** This function `engine_client` interacts with the model engine.
**CN:** 该函数 `engine_client` 与模型引擎交互。

### Lines 29-29 — Shared module state
```python
router = APIRouter()
```
**EN:** This block initializes `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 42-87 — Function `scale_elastic_ep`
```python
async def scale_elastic_ep(raw_request: Request):
    try:
        body = await raw_request.json()
    except json.JSONDecodeError as e:
        raise HTTPException(status_code=400, detail="Invalid JSON format") from e

    new_data_parallel_size = body.get("new_data_parallel_size")
    drain_timeout = body.get("drain_timeout", 120)  # Default 2 minutes

    if new_data_parallel_size is None:
        raise HTTPException(
            status_code=400, detail="new_data_parallel_size is required"
        )

    if not isinstance(new_data_parallel_size, int) or new_data_parallel_size <= 0:
        raise HTTPException(
            status_code=400,
            detail="new_data_parallel_size must be a positive integer",
        )

    if not isinstance(drain_timeout, int) or drain_timeout <= 0:
        raise HTTPException(
            status_code=400, detail="drain_timeout must be a positive integer"
        )

    # Set scaling flag to prevent new requests
    set_scaling_elastic_ep(True)
    client = engine_client(raw_request)
...
            detail="Scale failed due to request drain timeout "
            f"after {drain_timeout} seconds",
        ) from e
    except Exception as e:
        logger.error("Scale failed: %s", e)
        raise HTTPException(status_code=500, detail="Scale failed") from e
    finally:
        set_scaling_elastic_ep(False)
```
**EN:** This async function `scale_elastic_ep` interacts with the model engine, serializes API responses.
**CN:** 该异步函数 `scale_elastic_ep` 与模型引擎交互，序列化 API 响应。

### Lines 91-92 — Function `is_scaling_elastic_ep`
```python
async def is_scaling_elastic_ep(raw_request: Request):
    return JSONResponse({"is_scaling_elastic_ep": get_scaling_elastic_ep()})
```
**EN:** This async function `is_scaling_elastic_ep` serializes API responses.
**CN:** 该异步函数 `is_scaling_elastic_ep` 序列化 API 响应。

### Lines 95-96 — Function `attach_router`
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
- Engine-backed serving orchestration / 基于引擎的服务编排
- Middleware pipeline / 中间件链路
- Auxiliary serving endpoints / 辅助服务端点

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `http`
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.utils`, `vllm.entrypoints.serve.elastic_ep.middleware`, `vllm.logger`
