# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/cache/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: serving cache control. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：服务扩展 / 缓存控制。

## Line-by-Line Analysis / 逐行分析
### Lines 5-10 — Imports and shared dependencies
```python
from fastapi import APIRouter, FastAPI, Query, Request
from fastapi.responses import Response

import vllm.envs as envs
from vllm.engine.protocol import EngineClient
from vllm.logger import init_logger
```
**EN:** This import block uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.envs`, `vllm.engine.protocol`, `vllm.logger`.
**CN:** 该导入块使用 `fastapi` 等第三方库，依赖 `vllm.envs`, `vllm.engine.protocol`, `vllm.logger` 等 vLLM 内部模块。

### Lines 12-14 — Shared module state
```python
logger = init_logger(__name__)

router = APIRouter()
```
**EN:** This block initializes `logger`, `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 17-18 — Function `engine_client`
```python
def engine_client(request: Request) -> EngineClient:
    return request.app.state.engine_client
```
**EN:** This function `engine_client` interacts with the model engine.
**CN:** 该函数 `engine_client` 与模型引擎交互。

### Lines 22-44 — Function `reset_prefix_cache`
```python
async def reset_prefix_cache(
    raw_request: Request,
    reset_running_requests: bool = Query(default=False),
    reset_external: bool = Query(default=False),
):
    """
    Reset the local prefix cache.

    Optionally, if the query parameter `reset_external=true`
    also resets the external (connector-managed) prefix cache.

    Note that we currently do not check if the prefix cache
    is successfully reset in the API server.

    Example:
       POST /reset_prefix_cache?reset_external=true
    """
    logger.info("Resetting prefix cache...")

    await engine_client(raw_request).reset_prefix_cache(
        reset_running_requests, reset_external
    )
    return Response(status_code=200)
```
**EN:** This async function `reset_prefix_cache` is documented as: Reset the local prefix cache.
**CN:** 这里定义异步函数 `reset_prefix_cache`，其文档字符串说明了主要职责与调用约定。

### Lines 48-55 — Function `reset_mm_cache`
```python
async def reset_mm_cache(raw_request: Request):
    """
    Reset the multi-modal cache. Note that we currently do not check if the
    multi-modal cache is successfully reset in the API server.
    """
    logger.info("Resetting multi-modal cache...")
    await engine_client(raw_request).reset_mm_cache()
    return Response(status_code=200)
```
**EN:** This async function `reset_mm_cache` is documented as: Reset the multi-modal cache.
**CN:** 这里定义异步函数 `reset_mm_cache`，其文档字符串说明了主要职责与调用约定。

### Lines 59-66 — Function `reset_encoder_cache`
```python
async def reset_encoder_cache(raw_request: Request):
    """
    Reset the encoder cache. Note that we currently do not check if the
    encoder cache is successfully reset in the API server.
    """
    logger.info("Resetting encoder cache...")
    await engine_client(raw_request).reset_encoder_cache()
    return Response(status_code=200)
```
**EN:** This async function `reset_encoder_cache` is documented as: Reset the encoder cache.
**CN:** 这里定义异步函数 `reset_encoder_cache`，其文档字符串说明了主要职责与调用约定。

### Lines 69-72 — Function `attach_router`
```python
def attach_router(app: FastAPI):
    if not envs.VLLM_SERVER_DEV_MODE:
        return
    app.include_router(router)
```
**EN:** This function `attach_router` implements the `attaches router` step within the module flow.
**CN:** 该函数 `attach_router` 实现了模块流程中的“挂载路由”步骤。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Router registration / 路由注册
- Async request handling / 异步请求处理
- Engine-backed serving orchestration / 基于引擎的服务编排
- Auxiliary serving endpoints / 辅助服务端点

## Dependencies / 依赖关系
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.envs`, `vllm.engine.protocol`, `vllm.logger`
