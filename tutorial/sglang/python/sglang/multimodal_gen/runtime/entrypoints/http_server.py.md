# http_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/http_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `lifespan`, `health`, and `get_models`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `lifespan`、`health` 和 `get_models` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-31: module setup and imports / 模块初始化与导入
```python
import asyncio
import base64
import os
import uuid
from contextlib import asynccontextmanager
from typing import TYPE_CHECKING

import torch
from fastapi import APIRouter, FastAPI, Request

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.runtime.entrypoints.openai import image_api, video_api
from sglang.multimodal_gen.runtime.entrypoints.openai.protocol import (
    VertexGenerateReqInput,
)
from sglang.multimodal_gen.runtime.entrypoints.openai.utils import build_sampling_params
from sglang.multimodal_gen.runtime.entrypoints.post_training import (
    rollout_api,
    weights_api,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    prepare_request,
    save_outputs,
)
from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client
from sglang.multimodal_gen.runtime.server_args import ServerArgs, get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.srt.utils.json_response import orjson_response
from sglang.version import __version__
```
**EN:** This block establishes the module context and imports `asyncio`, `base64`, `os`, `uuid`, `contextlib`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `asyncio`、`base64`、`os`、`uuid`、`contextlib` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 33-38: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req

logger = init_logger(__name__)

VERTEX_ROUTE = os.environ.get("AIP_PREDICT_ROUTE", "/vertex_generate")
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `VERTEX_ROUTE`. The code collaborates with `init_logger`, and `os.environ.get`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `VERTEX_ROUTE` 等名称。 代码会与 `init_logger` 和 `os.environ.get` 协同工作。

### Lines 41-60: `lifespan` implementation / `lifespan` 实现
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    from sglang.multimodal_gen.runtime.scheduler_client import (
        async_scheduler_client,
        run_zeromq_broker,
    )

    # 1. Initialize the singleton client that connects to the backend Scheduler
    server_args = app.state.server_args
    async_scheduler_client.initialize(server_args)

    # 2. Start the ZMQ Broker in the background to handle offline requests
    broker_task = asyncio.create_task(run_zeromq_broker(server_args))

    yield

    # On shutdown
    logger.info("FastAPI app is shutting down...")
    broker_task.cancel()
    async_scheduler_client.close()
```
**EN:** This block defines function `lifespan`. It handles lifespan logic. Key calls include `async_scheduler_client.initialize`, `asyncio.create_task`, `logger.info`, `broker_task.cancel`, and `async_scheduler_client.close`. Parameters such as `app` drive the behavior in this section.
**CN:** 该代码块定义了函数 `lifespan`。 它用于处理 lifespan 相关逻辑。 关键调用包括 `async_scheduler_client.initialize`、`asyncio.create_task`、`logger.info`、`broker_task.cancel` 和 `async_scheduler_client.close`。 本段逻辑主要由 `app` 等参数驱动。

### Lines 64-64: supporting statements / 辅助语句
```python
health_router = APIRouter()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `health_router`. The code collaborates with `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `health_router` 等名称。 代码会与 `APIRouter` 协同工作。

### Lines 67-69: `health` implementation / `health` 实现
```python
@health_router.get("/health")
async def health():
    return {"status": "ok"}
```
**EN:** This block defines function `health`. It handles health logic. Key calls include `health_router.get`.
**CN:** 该代码块定义了函数 `health`。 它用于处理 health 相关逻辑。 关键调用包括 `health_router.get`。

### Lines 72-98: `get_models` implementation / `get_models` 实现
```python
@health_router.get("/models", deprecated=True)
async def get_models(request: Request):
    """
    Get information about the model served by this server.

    .. deprecated::
        Use /v1/models instead for OpenAI-compatible model discovery.
        This endpoint will be removed in a future version.
    """
    from sglang.multimodal_gen.registry import get_model_info

    server_args: ServerArgs = request.app.state.server_args
    model_info = get_model_info(server_args.model_path, model_id=server_args.model_id)

    response = {
        "model_path": server_args.model_path,
        "num_gpus": server_args.num_gpus,
        "task_type": server_args.pipeline_config.task_type.name,
        "dit_precision": server_args.pipeline_config.dit_precision,
        "vae_precision": server_args.pipeline_config.vae_precision,
    }

    if model_info:
        response["pipeline_name"] = model_info.pipeline_cls.pipeline_name
        response["pipeline_class"] = model_info.pipeline_cls.__name__

    return response
```
**EN:** This block defines function `get_models`. Get information about the model served by this server. .. Key calls include `health_router.get`, and `get_model_info`. The implementation branches on conditions. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_models`。 它用于获取models。 关键调用包括 `health_router.get` 和 `get_model_info`。 实现中包含条件分支。 本段逻辑主要由 `request` 等参数驱动。

### Lines 101-116: `server_info_endpoint` implementation / `server_info_endpoint` 实现
```python
@health_router.get("/server_info")
async def server_info_endpoint(request: Request):
    """Get server information.

    Returns fields compatible with the LLM engine's /server_info so that
    the model gateway can discover diffusion workers.
    """
    server_args: ServerArgs = request.app.state.server_args

    return {
        "model_path": server_args.model_path,
        "served_model_name": server_args.model_id or server_args.model_path,
        "tp_size": server_args.tp_size,
        "dp_size": server_args.dp_size,
        "version": __version__,
    }
```
**EN:** This block defines function `server_info_endpoint`. Get server information. Returns fields compatible with the LLM engine's /server_info so that the model gateway can discover diffusion workers. Key calls include `health_router.get`. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `server_info_endpoint`。 它用于处理 server info endpoint 相关逻辑。 关键调用包括 `health_router.get`。 本段逻辑主要由 `request` 等参数驱动。

### Lines 119-155: `model_info_endpoint` implementation / `model_info_endpoint` 实现
```python
@health_router.get("/model_info")
async def model_info_endpoint(request: Request):
    """Get model information.

    Returns fields compatible with the LLM engine's /model_info so that
    the model gateway can detect capabilities for diffusion workers.
    """
    from sglang.multimodal_gen.registry import get_model_info

    server_args: ServerArgs = request.app.state.server_args
    task_type = server_args.pipeline_config.task_type

    try:
        registry_info = get_model_info(
            server_args.model_path,
            backend=server_args.backend,
            model_id=server_args.model_id,
        )
    except Exception:
        logger.warning("Failed to resolve model info from registry", exc_info=True)
        registry_info = None

    return {
        # Fields consumed by the model gateway for worker discovery
        "model_path": server_args.model_path,
        "is_generation": True,
        "model_type": "diffusion",
        "architectures": (
            [registry_info.pipeline_cls.__name__] if registry_info else None
        ),
        # Fields matching the LLM engine's /model_info shape
        "has_image_understanding": task_type.accepts_image_input(),
        "has_audio_understanding": False,
        # Diffusion-specific fields
        "task_type": task_type.name,
        "is_image_gen": task_type.is_image_gen(),
    }
```
**EN:** This block defines function `model_info_endpoint`. Get model information. Returns fields compatible with the LLM engine's /model_info so that the model gateway can detect capabilities for diffusion workers. Key calls include `health_router.get`, `get_model_info`, `task_type.accepts_image_input`, `task_type.is_image_gen`, and `logger.warning`. The implementation handles exceptional paths. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `model_info_endpoint`。 它用于处理 model info endpoint 相关逻辑。 关键调用包括 `health_router.get`、`get_model_info`、`task_type.accepts_image_input`、`task_type.is_image_gen` 和 `logger.warning`。 实现中处理异常路径。 本段逻辑主要由 `request` 等参数驱动。

### Lines 158-161: `health_generate` implementation / `health_generate` 实现
```python
@health_router.get("/health_generate")
async def health_generate():
    # TODO : health generate endpoint
    return {"status": "ok"}
```
**EN:** This block defines function `health_generate`. It handles health generate logic. Key calls include `health_router.get`.
**CN:** 该代码块定义了函数 `health_generate`。 它用于处理 health generate 相关逻辑。 关键调用包括 `health_router.get`。

### Lines 164-187: `stats_endpoint` implementation / `stats_endpoint` 实现
```python
@health_router.get("/stats")
async def stats_endpoint(request: Request):
    """Get runtime statistics including disagg pipeline metrics.

    Returns queue depth, request counts, latency, throughput, etc.
    Sends a GetDisaggStatsReq to the scheduler via ZMQ and returns the result.
    """
    from sglang.multimodal_gen.runtime.entrypoints.utils import GetDisaggStatsReq

    server_args: ServerArgs = request.app.state.server_args
    response: dict = {
        "status": "ok",
        "model_path": server_args.model_path,
    }

    # Query the scheduler for disagg metrics
    try:
        stats_response = await async_scheduler_client.forward(GetDisaggStatsReq())
        if hasattr(stats_response, "output") and stats_response.output is not None:
            response["disagg"] = stats_response.output
    except Exception as e:
        response["disagg"] = {"error": str(e)}

    return response
```
**EN:** This block defines function `stats_endpoint`. Get runtime statistics including disagg pipeline metrics. Returns queue depth, request counts, latency, throughput, etc. Key calls include `health_router.get`, `async_scheduler_client.forward`, `hasattr`, `GetDisaggStatsReq`, and `str`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `stats_endpoint`。 它用于处理 stats endpoint 相关逻辑。 关键调用包括 `health_router.get`、`async_scheduler_client.forward`、`hasattr`、`GetDisaggStatsReq` 和 `str`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request` 等参数驱动。

### Lines 190-198: `make_serializable` implementation / `make_serializable` 实现
```python
def make_serializable(obj):
    """Recursively converts Tensors to None for JSON serialization."""
    if isinstance(obj, torch.Tensor):
        return None
    if isinstance(obj, dict):
        return {k: make_serializable(v) for k, v in obj.items()}
    if isinstance(obj, list):
        return [make_serializable(v) for v in obj]
    return obj
```
**EN:** This block defines function `make_serializable`. Recursively converts Tensors to None for JSON serialization. Key calls include `isinstance`, `make_serializable`, and `obj.items`. The implementation branches on conditions. Parameters such as `obj` drive the behavior in this section.
**CN:** 该代码块定义了函数 `make_serializable`。 它用于构建serializable。 关键调用包括 `isinstance`、`make_serializable` 和 `obj.items`。 实现中包含条件分支。 本段逻辑主要由 `obj` 等参数驱动。

### Lines 201-205: `encode_video_to_base64` implementation / `encode_video_to_base64` 实现
```python
def encode_video_to_base64(file_path: str):
    if not os.path.exists(file_path):
        return None
    with open(file_path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")
```
**EN:** This block defines function `encode_video_to_base64`. It encodes video to base64. Key calls include `os.path.exists`, `open`, `base64.b64encode.decode`, `base64.b64encode`, and `f.read`. The implementation branches on conditions, uses context-managed resources. Parameters such as `file_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `encode_video_to_base64`。 它用于编码video to base64。 关键调用包括 `os.path.exists`、`open`、`base64.b64encode.decode`、`base64.b64encode` 和 `f.read`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `file_path` 等参数驱动。

### Lines 208-257: `forward_to_scheduler` implementation / `forward_to_scheduler` 实现
```python
async def forward_to_scheduler(
    req_obj: "Req",
    sp: SamplingParams,
):
    """Forwards request to scheduler and processes the result."""
    try:
        response = await async_scheduler_client.forward(req_obj)
        if response.output is None and response.output_file_paths is None:
            raise RuntimeError("Model generation returned no output.")

        if response.output_file_paths:
            output_file_path = response.output_file_paths[0]
        else:
            output_file_path = sp.output_file_path()
            save_outputs(
                [response.output[0]],
                sp.data_type,
                sp.fps,
                True,
                lambda _idx: output_file_path,
                audio=response.audio,
                audio_sample_rate=response.audio_sample_rate,
                enable_frame_interpolation=sp.enable_frame_interpolation,
                frame_interpolation_exp=sp.frame_interpolation_exp,
                frame_interpolation_scale=sp.frame_interpolation_scale,
                frame_interpolation_model_path=sp.frame_interpolation_model_path,
                enable_upscaling=sp.enable_upscaling,
                upscaling_model_path=sp.upscaling_model_path,
                upscaling_scale=sp.upscaling_scale,
            )

        if hasattr(response, "model_dump"):
            data = response.model_dump()
        else:
            data = response if isinstance(response, dict) else vars(response)

        if output_file_path:
            logger.info("Processing output file: %s", output_file_path)
            b64_video = encode_video_to_base64(output_file_path)

            if b64_video:
                data["output"] = b64_video
                data.pop("video_data", None)
                data.pop("video_tensor", None)

        return make_serializable(data)

    except Exception as e:
        logger.error("Error during generation: %s", e, exc_info=True)
        return {"error": str(e)}
```
**EN:** This block defines function `forward_to_scheduler`. Forwards request to scheduler and processes the result. Key calls include `hasattr`, `make_serializable`, `async_scheduler_client.forward`, `RuntimeError`, and `sp.output_file_path`. The implementation branches on conditions, handles exceptional paths. Parameters such as `req_obj`, and `sp` drive the behavior in this section.
**CN:** 该代码块定义了函数 `forward_to_scheduler`。 它用于执行前向计算to scheduler。 关键调用包括 `hasattr`、`make_serializable`、`async_scheduler_client.forward`、`RuntimeError` 和 `sp.output_file_path`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `req_obj` 和 `sp` 等参数驱动。

### Lines 260-260: supporting statements / 辅助语句
```python
vertex_router = APIRouter()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `vertex_router`. The code collaborates with `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `vertex_router` 等名称。 代码会与 `APIRouter` 协同工作。

### Lines 263-293: `vertex_generate` implementation / `vertex_generate` 实现
```python
@vertex_router.post(VERTEX_ROUTE)
async def vertex_generate(vertex_req: VertexGenerateReqInput):
    if not vertex_req.instances:
        return orjson_response({"predictions": []})

    server_args = get_global_server_args()
    params = vertex_req.parameters or {}

    futures = []

    for inst in vertex_req.instances:
        rid = f"vertex_{uuid.uuid4()}"

        sp = build_sampling_params(
            rid,
            prompt=inst.get("prompt") or inst.get("text"),
            image_path=inst.get("image") or inst.get("image_url"),
            num_frames=params.get("num_frames"),
            fps=params.get("fps"),
            width=params.get("width"),
            height=params.get("height"),
            guidance_scale=params.get("guidance_scale"),
            save_output=params.get("save_output"),
        )

        backend_req = prepare_request(server_args, sampling_params=sp)
        futures.append(forward_to_scheduler(backend_req, sp))

    results = await asyncio.gather(*futures)

    return orjson_response({"predictions": results})
```
**EN:** This block defines function `vertex_generate`. It handles vertex generate logic. Key calls include `vertex_router.post`, `get_global_server_args`, `orjson_response`, `build_sampling_params`, and `prepare_request`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `vertex_req` drive the behavior in this section.
**CN:** 该代码块定义了函数 `vertex_generate`。 它用于处理 vertex generate 相关逻辑。 关键调用包括 `vertex_router.post`、`get_global_server_args`、`orjson_response`、`build_sampling_params` 和 `prepare_request`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `vertex_req` 等参数驱动。

### Lines 296-315: `create_app` implementation / `create_app` 实现
```python
def create_app(server_args: ServerArgs):
    """
    Create and configure the FastAPI application instance.
    """
    app = FastAPI(lifespan=lifespan)

    app.include_router(health_router)
    app.include_router(vertex_router)

    from sglang.multimodal_gen.runtime.entrypoints.openai import common_api, mesh_api

    app.include_router(common_api.router)
    app.include_router(image_api.router)
    app.include_router(video_api.router)
    app.include_router(mesh_api.router)
    app.include_router(weights_api.router)
    app.include_router(rollout_api.router)

    app.state.server_args = server_args
    return app
```
**EN:** This block defines function `create_app`. Create and configure the FastAPI application instance. Key calls include `FastAPI`, and `app.include_router`. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `create_app`。 它用于创建app。 关键调用包括 `FastAPI` 和 `app.include_router`。 本段逻辑主要由 `server_args` 等参数驱动。

## Key Concepts / 关键概念
- `lifespan`: Top-level function that handles lifespan logic. / 顶层函数，用于处理 lifespan 相关逻辑。
- `health`: Top-level function that handles health logic. / 顶层函数，用于处理 health 相关逻辑。
- `get_models`: Get information about the model served by this server. / 顶层函数，用于获取models。
- `server_info_endpoint`: Get server information. / 顶层函数，用于处理 server info endpoint 相关逻辑。
- `model_info_endpoint`: Get model information. / 顶层函数，用于处理 model info endpoint 相关逻辑。
- `health_generate`: Top-level function that handles health generate logic. / 顶层函数，用于处理 health generate 相关逻辑。
- `stats_endpoint`: Get runtime statistics including disagg pipeline metrics. / 顶层函数，用于处理 stats endpoint 相关逻辑。
- `make_serializable`: Recursively converts Tensors to None for JSON serialization. / 顶层函数，用于构建serializable。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `base64`, `os`, `uuid`, `contextlib`, `typing`
- **Third-party / 第三方依赖**: `torch`, `fastapi`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.openai`, `sglang.multimodal_gen.runtime.entrypoints.openai.protocol`, `sglang.multimodal_gen.runtime.entrypoints.openai.utils`, `sglang.multimodal_gen.runtime.entrypoints.post_training`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils.json_response`, `sglang.version`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`

- **Total lines / 总行数**: 315
