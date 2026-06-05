# api_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/sleep/api_router.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines FastAPI/APIRouter endpoints for the surrounding feature area. Scope: serving sleep controls. / 为所在功能域定义 FastAPI/APIRouter 端点。 作用域：服务扩展 / 休眠控制。

## Line-by-Line Analysis / 逐行分析
### Lines 5-10 — Imports and shared dependencies
```python
from fastapi import APIRouter, FastAPI, Request
from fastapi.responses import JSONResponse, Response

import vllm.envs as envs
from vllm.engine.protocol import EngineClient
from vllm.logger import init_logger
```
**EN:** This import block uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.envs`, `vllm.engine.protocol`, `vllm.logger`.
**CN:** 该导入块使用 `fastapi` 等第三方库，依赖 `vllm.envs`, `vllm.engine.protocol`, `vllm.logger` 等 vLLM 内部模块。

### Lines 12-12 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 15-16 — Function `engine_client`
```python
def engine_client(request: Request) -> EngineClient:
    return request.app.state.engine_client
```
**EN:** This function `engine_client` interacts with the model engine.
**CN:** 该函数 `engine_client` 与模型引擎交互。

### Lines 19-19 — Shared module state
```python
router = APIRouter()
```
**EN:** This block initializes `router`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `router`，为后续逻辑准备模块级常量或共享状态。

### Lines 23-30 — Function `sleep`
```python
async def sleep(raw_request: Request):
    # get POST params
    level = raw_request.query_params.get("level", "1")
    mode = raw_request.query_params.get("mode", "abort")
    await engine_client(raw_request).sleep(int(level), mode)
    # FIXME: in v0 with frontend multiprocessing, the sleep command
    # is sent but does not finish yet when we return a response.
    return Response(status_code=200)
```
**EN:** This async function `sleep` interacts with the model engine.
**CN:** 该异步函数 `sleep` 与模型引擎交互。

### Lines 34-43 — Function `wake_up`
```python
async def wake_up(raw_request: Request):
    tags = raw_request.query_params.getlist("tags")
    if tags == []:
        # set to None to wake up all tags if no tags are provided
        tags = None
    logger.info("wake up the engine with tags: %s", tags)
    await engine_client(raw_request).wake_up(tags)
    # FIXME: in v0 with frontend multiprocessing, the wake-up command
    # is sent but does not finish yet when we return a response.
    return Response(status_code=200)
```
**EN:** This async function `wake_up` interacts with the model engine.
**CN:** 该异步函数 `wake_up` 与模型引擎交互。

### Lines 47-49 — Function `is_sleeping`
```python
async def is_sleeping(raw_request: Request):
    is_sleeping = await engine_client(raw_request).is_sleeping()
    return JSONResponse(content={"is_sleeping": is_sleeping})
```
**EN:** This async function `is_sleeping` interacts with the model engine, serializes API responses.
**CN:** 该异步函数 `is_sleeping` 与模型引擎交互，序列化 API 响应。

### Lines 52-56 — Function `attach_router`
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
