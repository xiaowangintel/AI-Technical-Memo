# mesh_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/mesh_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `_normalize_format`, `_build_sampling_params_from_request`, and `_mesh_job_from_sampling`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `_normalize_format`、`_build_sampling_params_from_request` 和 `_mesh_job_from_sampling` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: module setup and imports / 模块初始化与导入
```python
import asyncio
import os
import time
from typing import Any, Dict, List, Optional

from fastapi import (
    APIRouter,
    File,
    Form,
    HTTPException,
    Path,
    Query,
    Request,
    UploadFile,
)
from fastapi.responses import FileResponse

from sglang.multimodal_gen.configs.sample.sampling_params import (
    SamplingParams,
    generate_request_id,
)
from sglang.multimodal_gen.runtime.entrypoints.openai.protocol import (
    MeshGenerationsRequest,
    MeshListResponse,
    MeshResponse,
)
from sglang.multimodal_gen.runtime.entrypoints.openai.storage import cloud_storage
from sglang.multimodal_gen.runtime.entrypoints.openai.stores import MESH_STORE
from sglang.multimodal_gen.runtime.entrypoints.openai.utils import (
    add_common_data_to_response,
    merge_image_input_list,
    process_generation_batch,
    save_image_to_path,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `asyncio`, `os`, `time`, `typing`, `fastapi`, and `fastapi.responses`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `asyncio`、`os`、`time`、`typing`、`fastapi` 和 `fastapi.responses`。这些依赖为后续实现提供所需符号。

### Lines 40-41: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
router = APIRouter(prefix="/v1/meshes", tags=["meshes"])
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `router`. The code collaborates with `init_logger`, and `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `router` 等名称。 代码会与 `init_logger` 和 `APIRouter` 协同工作。

### Lines 44-46: `_normalize_format` implementation / `_normalize_format` 实现
```python
def _normalize_format(fmt: Optional[str]) -> str:
    fmt = (fmt or "glb").lower()
    return fmt if fmt in ("glb", "obj") else "glb"
```
**EN:** This block defines function `_normalize_format`. It handles normalize format logic. Key calls include `lower`. Parameters such as `fmt` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_normalize_format`。 它用于处理 normalize format 相关逻辑。 关键调用包括 `lower`。 本段逻辑主要由 `fmt` 等参数驱动。

### Lines 49-76: `_build_sampling_params_from_request` implementation / `_build_sampling_params_from_request` 实现
```python
def _build_sampling_params_from_request(
    request_id: str, req: MeshGenerationsRequest, image_path: Optional[str] = None
) -> SamplingParams:
    ext = _normalize_format(req.output_format)

    server_args = get_global_server_args()
    sampling_kwargs: Dict[str, Any] = {
        "request_id": request_id,
        "prompt": req.prompt,
        "num_frames": 1,
        "image_path": [image_path] if image_path else None,
        "save_output": True,
        "output_file_name": f"{request_id}.{ext}",
        "seed": req.seed,
        "generator_device": req.generator_device,
    }
    if req.num_inference_steps is not None:
        sampling_kwargs["num_inference_steps"] = req.num_inference_steps
    if req.guidance_scale is not None:
        sampling_kwargs["guidance_scale"] = req.guidance_scale
    if req.negative_prompt is not None:
        sampling_kwargs["negative_prompt"] = req.negative_prompt

    return SamplingParams.from_user_sampling_params_args(
        model_path=server_args.model_path,
        server_args=server_args,
        **sampling_kwargs,
    )
```
**EN:** This block defines function `_build_sampling_params_from_request`. It builds sampling params from request. Key calls include `_normalize_format`, `get_global_server_args`, and `SamplingParams.from_user_sampling_params_args`. The implementation branches on conditions. Parameters such as `request_id`, `req`, and `image_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_sampling_params_from_request`。 它用于构建sampling params from request。 关键调用包括 `_normalize_format`、`get_global_server_args` 和 `SamplingParams.from_user_sampling_params_args`。 实现中包含条件分支。 本段逻辑主要由 `request_id`、`req` 和 `image_path` 等参数驱动。

### Lines 79-91: `_mesh_job_from_sampling` implementation / `_mesh_job_from_sampling` 实现
```python
def _mesh_job_from_sampling(
    request_id: str, req: MeshGenerationsRequest, sampling: SamplingParams
) -> Dict[str, Any]:
    return {
        "id": request_id,
        "object": "mesh",
        "model": req.model or "",
        "status": "queued",
        "progress": 0,
        "created_at": int(time.time()),
        "format": _normalize_format(req.output_format),
        "file_path": os.path.abspath(sampling.output_file_path()),
    }
```
**EN:** This block defines function `_mesh_job_from_sampling`. It handles mesh job from sampling logic. Key calls include `int`, `_normalize_format`, `os.path.abspath`, `time.time`, and `sampling.output_file_path`. Parameters such as `request_id`, `req`, and `sampling` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_mesh_job_from_sampling`。 它用于处理 mesh job from sampling 相关逻辑。 关键调用包括 `int`、`_normalize_format`、`os.path.abspath`、`time.time` 和 `sampling.output_file_path`。 本段逻辑主要由 `request_id`、`req` 和 `sampling` 等参数驱动。

### Lines 94-125: `_dispatch_job_async` implementation / `_dispatch_job_async` 实现
```python
async def _dispatch_job_async(job_id: str, batch: Req) -> None:
    from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client

    try:
        save_file_path_list, result = await process_generation_batch(
            async_scheduler_client, batch
        )
        save_file_path = save_file_path_list[0]

        file_size = None
        if os.path.exists(save_file_path):
            file_size = os.path.getsize(save_file_path)

        cloud_url = await cloud_storage.upload_and_cleanup(save_file_path)

        update_fields: Dict[str, Any] = {
            "status": "completed",
            "progress": 100,
            "completed_at": int(time.time()),
            "url": cloud_url,
            "file_path": save_file_path if not cloud_url else None,
            "file_size_bytes": file_size,
        }
        update_fields = add_common_data_to_response(
            update_fields, request_id=job_id, result=result
        )
        await MESH_STORE.update_fields(job_id, update_fields)
    except Exception as e:
        logger.error(f"{e}")
        await MESH_STORE.update_fields(
            job_id, {"status": "failed", "error": {"message": str(e)}}
        )
```
**EN:** This block defines function `_dispatch_job_async`. It handles dispatch job async logic. Key calls include `os.path.exists`, `add_common_data_to_response`, `process_generation_batch`, `os.path.getsize`, and `cloud_storage.upload_and_cleanup`. The implementation branches on conditions, handles exceptional paths. Parameters such as `job_id`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_dispatch_job_async`。 它用于处理 dispatch job async 相关逻辑。 关键调用包括 `os.path.exists`、`add_common_data_to_response`、`process_generation_batch`、`os.path.getsize` 和 `cloud_storage.upload_and_cleanup`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `job_id` 和 `batch` 等参数驱动。

### Lines 128-223: `create_mesh` implementation / `create_mesh` 实现
```python
@router.post("", response_model=MeshResponse)
async def create_mesh(
    request: Request,
    image: Optional[List[UploadFile]] = File(None),
    image_array: Optional[List[UploadFile]] = File(None, alias="image[]"),
    url: Optional[List[str]] = Form(None),
    url_array: Optional[List[str]] = Form(None, alias="url[]"),
    prompt: Optional[str] = Form("generate 3d mesh"),
    model: Optional[str] = Form(None),
    seed: Optional[int] = Form(None),
    generator_device: Optional[str] = Form("cuda"),
    guidance_scale: Optional[float] = Form(None),
    num_inference_steps: Optional[int] = Form(None),
    negative_prompt: Optional[str] = Form(None),
    output_format: Optional[str] = Form("glb"),
):
    content_type = request.headers.get("content-type", "").lower()
    request_id = generate_request_id()
    server_args = get_global_server_args()

    input_path = None

    if "multipart/form-data" in content_type:
        images = image or image_array
        urls = url or url_array
        image_list = merge_image_input_list(images, urls)

        if not image_list:
            raise HTTPException(
                status_code=422,
                detail="Field 'image' or 'url' is required for mesh generation",
            )

        uploads_dir = os.path.join("outputs", "uploads")
        os.makedirs(uploads_dir, exist_ok=True)
        img = image_list[0]
        filename = img.filename if hasattr(img, "filename") else "input_image"
        try:
            input_path = await save_image_to_path(
                img, os.path.join(uploads_dir, f"{request_id}_{filename}")
            )
        except Exception as e:
            raise HTTPException(
                status_code=400, detail=f"Failed to process image source: {str(e)}"
            )

        req = MeshGenerationsRequest(
            prompt=prompt or "generate 3d mesh",
            model=model,
            seed=seed,
            generator_device=generator_device,
            num_inference_steps=num_inference_steps,
            negative_prompt=negative_prompt,
            output_format=output_format,
            **(
                {"guidance_scale": guidance_scale} if guidance_scale is not None else {}
            ),
        )
    else:
        try:
            body = await request.json()
        except Exception:
            body = {}
        try:
            payload: Dict[str, Any] = dict(body or {})

            if payload.get("input_image"):
                img_src = payload.pop("input_image")
                uploads_dir = os.path.join("outputs", "uploads")
                os.makedirs(uploads_dir, exist_ok=True)
                input_path = await save_image_to_path(
                    img_src,
                    os.path.join(uploads_dir, f"{request_id}_input_image"),
                )

            req = MeshGenerationsRequest(**payload)
        except Exception as e:
            raise HTTPException(status_code=400, detail=f"Invalid request body: {e}")

    if not input_path:
        raise HTTPException(
            status_code=422,
            detail="An input image is required for mesh generation",
        )

    sampling_params = _build_sampling_params_from_request(request_id, req, input_path)
    job = _mesh_job_from_sampling(request_id, req, sampling_params)
    await MESH_STORE.upsert(request_id, job)

    batch = prepare_request(
        server_args=server_args,
        sampling_params=sampling_params,
    )

    asyncio.create_task(_dispatch_job_async(request_id, batch))
    return MeshResponse(**job)
```
**EN:** This block defines function `create_mesh`. It creates mesh. Key calls include `router.post`, `File`, `Form`, `request.headers.get.lower`, and `generate_request_id`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request`, `image`, `image_array`, `url`, and `url_array` drive the behavior in this section.
**CN:** 该代码块定义了函数 `create_mesh`。 它用于创建mesh。 关键调用包括 `router.post`、`File`、`Form`、`request.headers.get.lower` 和 `generate_request_id`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request`、`image`、`image_array`、`url` 和 `url_array` 等参数驱动。

### Lines 226-250: `list_meshes` implementation / `list_meshes` 实现
```python
@router.get("", response_model=MeshListResponse)
async def list_meshes(
    after: Optional[str] = Query(None),
    limit: Optional[int] = Query(None, ge=1, le=100),
    order: Optional[str] = Query("desc"),
):
    order = (order or "desc").lower()
    if order not in ("asc", "desc"):
        order = "desc"
    jobs = await MESH_STORE.list_values()

    reverse = order != "asc"
    jobs.sort(key=lambda j: j.get("created_at", 0), reverse=reverse)

    if after is not None:
        try:
            idx = next(i for i, j in enumerate(jobs) if j["id"] == after)
            jobs = jobs[idx + 1 :]
        except StopIteration:
            jobs = []

    if limit is not None:
        jobs = jobs[:limit]
    items = [MeshResponse(**j) for j in jobs]
    return MeshListResponse(data=items)
```
**EN:** This block defines function `list_meshes`. It handles list meshes logic. Key calls include `router.get`, `Query`, `lower`, `jobs.sort`, and `MeshListResponse`. The implementation branches on conditions, handles exceptional paths. Parameters such as `after`, `limit`, and `order` drive the behavior in this section.
**CN:** 该代码块定义了函数 `list_meshes`。 它用于处理 list meshes 相关逻辑。 关键调用包括 `router.get`、`Query`、`lower`、`jobs.sort` 和 `MeshListResponse`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `after`、`limit` 和 `order` 等参数驱动。

### Lines 253-258: `retrieve_mesh` implementation / `retrieve_mesh` 实现
```python
@router.get("/{mesh_id}", response_model=MeshResponse)
async def retrieve_mesh(mesh_id: str = Path(...)):
    job = await MESH_STORE.get(mesh_id)
    if not job:
        raise HTTPException(status_code=404, detail="Mesh not found")
    return MeshResponse(**job)
```
**EN:** This block defines function `retrieve_mesh`. It handles retrieve mesh logic. Key calls include `router.get`, `Path`, `MeshResponse`, `MESH_STORE.get`, and `HTTPException`. The implementation branches on conditions. Parameters such as `mesh_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `retrieve_mesh`。 它用于处理 retrieve mesh 相关逻辑。 关键调用包括 `router.get`、`Path`、`MeshResponse`、`MESH_STORE.get` 和 `HTTPException`。 实现中包含条件分支。 本段逻辑主要由 `mesh_id` 等参数驱动。

### Lines 261-267: `delete_mesh` implementation / `delete_mesh` 实现
```python
@router.delete("/{mesh_id}", response_model=MeshResponse)
async def delete_mesh(mesh_id: str = Path(...)):
    job = await MESH_STORE.pop(mesh_id)
    if not job:
        raise HTTPException(status_code=404, detail="Mesh not found")
    job["status"] = "deleted"
    return MeshResponse(**job)
```
**EN:** This block defines function `delete_mesh`. It handles delete mesh logic. Key calls include `router.delete`, `Path`, `MeshResponse`, `MESH_STORE.pop`, and `HTTPException`. The implementation branches on conditions. Parameters such as `mesh_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `delete_mesh`。 它用于处理 delete mesh 相关逻辑。 关键调用包括 `router.delete`、`Path`、`MeshResponse`、`MESH_STORE.pop` 和 `HTTPException`。 实现中包含条件分支。 本段逻辑主要由 `mesh_id` 等参数驱动。

### Lines 270-296: `download_mesh_content` implementation / `download_mesh_content` 实现
```python
@router.get("/{mesh_id}/content")
async def download_mesh_content(
    mesh_id: str = Path(...), variant: Optional[str] = Query(None)
):
    job = await MESH_STORE.get(mesh_id)
    if not job:
        raise HTTPException(status_code=404, detail="Mesh not found")

    if job.get("url"):
        raise HTTPException(
            status_code=400,
            detail=f"Mesh has been uploaded to cloud storage. Please use the cloud URL: {job.get('url')}",
        )

    file_path = job.get("file_path")
    if not file_path or not os.path.exists(file_path):
        raise HTTPException(status_code=404, detail="Generation is still in-progress")

    ext = os.path.splitext(file_path)[1].lower()
    media_type = {
        ".glb": "model/gltf-binary",
        ".obj": "text/plain",
    }.get(ext, "application/octet-stream")

    return FileResponse(
        path=file_path, media_type=media_type, filename=os.path.basename(file_path)
    )
```
**EN:** This block defines function `download_mesh_content`. It handles download mesh content logic. Key calls include `router.get`, `Path`, `Query`, `job.get`, and `os.path.splitext.lower`. The implementation branches on conditions. Parameters such as `mesh_id`, and `variant` drive the behavior in this section.
**CN:** 该代码块定义了函数 `download_mesh_content`。 它用于处理 download mesh content 相关逻辑。 关键调用包括 `router.get`、`Path`、`Query`、`job.get` 和 `os.path.splitext.lower`。 实现中包含条件分支。 本段逻辑主要由 `mesh_id` 和 `variant` 等参数驱动。

## Key Concepts / 关键概念
- `_normalize_format`: Top-level function that handles normalize format logic. / 顶层函数，用于处理 normalize format 相关逻辑。
- `_build_sampling_params_from_request`: Top-level function that builds sampling params from request. / 顶层函数，用于构建sampling params from request。
- `_mesh_job_from_sampling`: Top-level function that handles mesh job from sampling logic. / 顶层函数，用于处理 mesh job from sampling 相关逻辑。
- `_dispatch_job_async`: Top-level function that handles dispatch job async logic. / 顶层函数，用于处理 dispatch job async 相关逻辑。
- `create_mesh`: Top-level function that creates mesh. / 顶层函数，用于创建mesh。
- `list_meshes`: Top-level function that handles list meshes logic. / 顶层函数，用于处理 list meshes 相关逻辑。
- `retrieve_mesh`: Top-level function that handles retrieve mesh logic. / 顶层函数，用于处理 retrieve mesh 相关逻辑。
- `delete_mesh`: Top-level function that handles delete mesh logic. / 顶层函数，用于处理 delete mesh 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `time`, `typing`
- **Third-party / 第三方依赖**: `fastapi`, `fastapi.responses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.openai.protocol`, `sglang.multimodal_gen.runtime.entrypoints.openai.storage`, `sglang.multimodal_gen.runtime.entrypoints.openai.stores`, `sglang.multimodal_gen.runtime.entrypoints.openai.utils`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.scheduler_client`

- **Total lines / 总行数**: 296
