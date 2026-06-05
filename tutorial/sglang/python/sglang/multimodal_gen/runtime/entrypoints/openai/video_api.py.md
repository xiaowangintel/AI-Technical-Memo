# video_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/video_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `_build_video_sampling_params`, `_video_job_from_sampling`, and `_save_first_input_image`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `_build_video_sampling_params`、`_video_job_from_sampling` 和 `_save_first_input_image` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-47: module setup and imports / 模块初始化与导入
```python
import asyncio
import json
import os
import shutil
import tempfile
import time
from typing import Any, Dict, Optional

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
    VideoGenerationsRequest,
    VideoListResponse,
    VideoResponse,
)
from sglang.multimodal_gen.runtime.entrypoints.openai.storage import cloud_storage
from sglang.multimodal_gen.runtime.entrypoints.openai.stores import VIDEO_STORE
from sglang.multimodal_gen.runtime.entrypoints.openai.utils import (
    DEFAULT_FPS,
    DEFAULT_VIDEO_SECONDS,
    add_common_data_to_response,
    build_sampling_params,
    merge_image_input_list,
    process_generation_batch,
    save_image_to_path,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.srt.observability.trace import extract_trace_headers
```
**EN:** This block establishes the module context and imports `asyncio`, `json`, `os`, `shutil`, `tempfile`, and `time`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `asyncio`、`json`、`os`、`shutil`、`tempfile` 和 `time`。这些依赖为后续实现提供所需符号。

### Lines 49-50: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
router = APIRouter(prefix="/v1/videos", tags=["videos"])
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `router`. The code collaborates with `init_logger`, and `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `router` 等名称。 代码会与 `init_logger` 和 `APIRouter` 协同工作。

### Lines 53-92: `_build_video_sampling_params` implementation / `_build_video_sampling_params` 实现
```python
def _build_video_sampling_params(request_id: str, request: VideoGenerationsRequest):
    """Resolve video-specific defaults (fps, seconds → num_frames) then
    delegate to the shared build_sampling_params."""
    seconds = request.seconds if request.seconds is not None else DEFAULT_VIDEO_SECONDS
    fps = request.fps if request.fps is not None else DEFAULT_FPS
    num_frames = request.num_frames if request.num_frames is not None else fps * seconds
    num_outputs = request.num_outputs_per_prompt
    if num_outputs is None:
        num_outputs = request.n or 1

    return build_sampling_params(
        request_id,
        prompt=request.prompt,
        num_outputs_per_prompt=max(1, min(int(num_outputs), 10)),
        size=request.size,
        width=request.width,
        height=request.height,
        num_frames=num_frames,
        fps=fps,
        image_path=request.input_reference,
        output_file_name=request_id,
        seed=request.seed,
        generator_device=request.generator_device,
        num_inference_steps=request.num_inference_steps,
        guidance_scale=request.guidance_scale,
        guidance_scale_2=request.guidance_scale_2,
        negative_prompt=request.negative_prompt,
        enable_teacache=request.enable_teacache,
        enable_frame_interpolation=request.enable_frame_interpolation,
        frame_interpolation_exp=request.frame_interpolation_exp,
        frame_interpolation_scale=request.frame_interpolation_scale,
        frame_interpolation_model_path=request.frame_interpolation_model_path,
        enable_upscaling=request.enable_upscaling,
        upscaling_model_path=request.upscaling_model_path,
        upscaling_scale=request.upscaling_scale,
        output_path=request.output_path,
        output_compression=request.output_compression,
        output_quality=request.output_quality,
        perf_dump_path=request.perf_dump_path,
    )
```
**EN:** This block defines function `_build_video_sampling_params`. Resolve video-specific defaults (fps, seconds → num_frames) then delegate to the shared build_sampling_params. Key calls include `build_sampling_params`, `max`, `min`, and `int`. The implementation branches on conditions. Parameters such as `request_id`, and `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_video_sampling_params`。 它用于构建video sampling params。 关键调用包括 `build_sampling_params`、`max`、`min` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `request_id` 和 `request` 等参数驱动。

### Lines 96-112: `_video_job_from_sampling` implementation / `_video_job_from_sampling` 实现
```python
def _video_job_from_sampling(
    request_id: str, req: VideoGenerationsRequest, sampling: SamplingParams
) -> Dict[str, Any]:
    size_str = f"{sampling.width}x{sampling.height}"
    seconds = int(round((sampling.num_frames or 0) / float(sampling.fps or 24)))
    return {
        "id": request_id,
        "object": "video",
        "model": req.model or "sora-2",
        "status": "queued",
        "progress": 0,
        "created_at": int(time.time()),
        "size": size_str,
        "seconds": str(seconds),
        "quality": "standard",
        "file_path": os.path.abspath(sampling.output_file_path()),
    }
```
**EN:** This block defines function `_video_job_from_sampling`. It handles video job from sampling logic. Key calls include `int`, `round`, `str`, `os.path.abspath`, and `time.time`. Parameters such as `request_id`, `req`, and `sampling` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_video_job_from_sampling`。 它用于处理 video job from sampling 相关逻辑。 关键调用包括 `int`、`round`、`str`、`os.path.abspath` 和 `time.time`。 本段逻辑主要由 `request_id`、`req` 和 `sampling` 等参数驱动。

### Lines 115-134: `_save_first_input_image` implementation / `_save_first_input_image` 实现
```python
async def _save_first_input_image(
    image_sources,
    request_id: str,
    uploads_dir: str,
    *,
    prefer_remote_source: bool = False,
) -> str | None:
    """Save the first input image from a list of sources and return its path."""
    image_list = merge_image_input_list(image_sources)
    if not image_list:
        return None
    image = image_list[0]

    os.makedirs(uploads_dir, exist_ok=True)

    filename = image.filename if hasattr(image, "filename") else "url_image"
    target_path = os.path.join(uploads_dir, f"{request_id}_{filename}")
    return await save_image_to_path(
        image, target_path, prefer_remote_source=prefer_remote_source
    )
```
**EN:** This block defines function `_save_first_input_image`. Save the first input image from a list of sources and return its path. Key calls include `merge_image_input_list`, `os.makedirs`, `os.path.join`, `hasattr`, and `save_image_to_path`. The implementation branches on conditions. Parameters such as `image_sources`, `request_id`, and `uploads_dir` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_save_first_input_image`。 它用于保存first input image。 关键调用包括 `merge_image_input_list`、`os.makedirs`、`os.path.join`、`hasattr` 和 `save_image_to_path`。 实现中包含条件分支。 本段逻辑主要由 `image_sources`、`request_id` 和 `uploads_dir` 等参数驱动。

### Lines 137-181: `_dispatch_job_async` implementation / `_dispatch_job_async` 实现
```python
async def _dispatch_job_async(
    job_id: str,
    batch: Req,
    *,
    temp_dirs: list[str] | None = None,
    output_persistent: bool = True,
) -> None:
    from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client

    try:
        save_file_path_list, result = await process_generation_batch(
            async_scheduler_client, batch
        )
        save_file_path = save_file_path_list[0]

        cloud_url = await cloud_storage.upload_and_cleanup(save_file_path)

        persistent_path = (
            save_file_path if not cloud_url and output_persistent else None
        )
        update_fields = {
            "status": "completed",
            "progress": 100,
            "completed_at": int(time.time()),
            "url": cloud_url,
            "file_path": persistent_path,
            "file_paths": (
                [os.path.abspath(path) for path in save_file_path_list]
                if output_persistent
                else None
            ),
            "num_outputs": len(save_file_path_list),
        }
        update_fields = add_common_data_to_response(
            update_fields, request_id=job_id, result=result
        )
        await VIDEO_STORE.update_fields(job_id, update_fields)
    except Exception as e:
        logger.error(f"{e}")
        await VIDEO_STORE.update_fields(
            job_id, {"status": "failed", "error": {"message": str(e)}}
        )
    finally:
        for td in temp_dirs or []:
            shutil.rmtree(td, ignore_errors=True)
```
**EN:** This block defines function `_dispatch_job_async`. It handles dispatch job async logic. Key calls include `add_common_data_to_response`, `process_generation_batch`, `cloud_storage.upload_and_cleanup`, `int`, and `len`. The implementation iterates over collections or steps, handles exceptional paths. Parameters such as `job_id`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_dispatch_job_async`。 它用于处理 dispatch job async 相关逻辑。 关键调用包括 `add_common_data_to_response`、`process_generation_batch`、`cloud_storage.upload_and_cleanup`、`int` 和 `len`。 实现中会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `job_id` 和 `batch` 等参数驱动。

### Lines 185-385: `create_video` implementation / `create_video` 实现
```python
@router.post("", response_model=VideoResponse)
async def create_video(
    request: Request,
    # multipart/form-data fields (optional; used only when content-type is multipart)
    prompt: Optional[str] = Form(None),
    input_reference: Optional[UploadFile] = File(None),
    reference_url: Optional[str] = Form(None),
    model: Optional[str] = Form(None),
    n: Optional[int] = Form(1),
    num_outputs_per_prompt: Optional[int] = Form(None),
    seconds: Optional[int] = Form(None),
    size: Optional[str] = Form(None),
    fps: Optional[int] = Form(None),
    num_frames: Optional[int] = Form(None),
    seed: Optional[int] = Form(None),
    generator_device: Optional[str] = Form("cuda"),
    negative_prompt: Optional[str] = Form(None),
    guidance_scale: Optional[float] = Form(None),
    num_inference_steps: Optional[int] = Form(None),
    enable_teacache: Optional[bool] = Form(False),
    enable_frame_interpolation: Optional[bool] = Form(False),
    frame_interpolation_exp: Optional[int] = Form(1),
    frame_interpolation_scale: Optional[float] = Form(1.0),
    frame_interpolation_model_path: Optional[str] = Form(None),
    enable_upscaling: Optional[bool] = Form(False),
    upscaling_model_path: Optional[str] = Form(None),
    upscaling_scale: Optional[int] = Form(4),
    output_quality: Optional[str] = Form("default"),
    output_compression: Optional[int] = Form(None),
    extra_body: Optional[str] = Form(None),
):
    content_type = request.headers.get("content-type", "").lower()
    request_id = generate_request_id()

    server_args = get_global_server_args()
    task_type = server_args.pipeline_config.task_type

    # Resolve input upload directory (may be a temp dir when saving is disabled)
    temp_dirs: list[str] = []
    if server_args.input_save_path is not None:
        uploads_dir = server_args.input_save_path
        os.makedirs(uploads_dir, exist_ok=True)
    else:
        uploads_dir = tempfile.mkdtemp(prefix="sglang_input_")
        temp_dirs.append(uploads_dir)

    # Resolve output directory
    effective_output_path = server_args.output_path
    output_persistent = True
    if "multipart/form-data" not in content_type:
        # JSON body may carry a per-request output_path; checked after parsing below
        pass

    if "multipart/form-data" in content_type:
        if not prompt:
            raise HTTPException(status_code=400, detail="prompt is required")
        # Validate image input based on model task type
        image_sources = merge_image_input_list(input_reference, reference_url)
        if task_type.requires_image_input() and not image_sources:
            raise HTTPException(
                status_code=400,
                detail="input_reference or reference_url is required for image-to-video generation",
            )
        try:
            input_path = await _save_first_input_image(
                image_sources,
                request_id,
                uploads_dir,
                prefer_remote_source=server_args.input_save_path is None,
            )
        except Exception as e:
            raise HTTPException(
                status_code=400, detail=f"Failed to process image source: {str(e)}"
            )

        # Parse extra_body JSON (if provided in multipart form) to get fps/num_frames overrides
        extra_from_form: Dict[str, Any] = {}
        if extra_body:
            try:
                extra_from_form = json.loads(extra_body)
            except Exception:
                extra_from_form = {}

        fps_val = fps if fps is not None else extra_from_form.get("fps")
        num_frames_val = (
            num_frames if num_frames is not None else extra_from_form.get("num_frames")
        )

        req = VideoGenerationsRequest(
            prompt=prompt,
            input_reference=input_path,
            model=model,
            n=n,
            num_outputs_per_prompt=num_outputs_per_prompt,
            seconds=seconds if seconds is not None else 4,
            size=size,
            fps=fps_val,
            num_frames=num_frames_val,
            seed=seed,
            generator_device=generator_device,
            negative_prompt=negative_prompt,
            num_inference_steps=num_inference_steps,
            enable_teacache=enable_teacache,
            enable_frame_interpolation=enable_frame_interpolation,
            frame_interpolation_exp=frame_interpolation_exp,
            frame_interpolation_scale=frame_interpolation_scale,
            frame_interpolation_model_path=frame_interpolation_model_path,
            enable_upscaling=enable_upscaling,
            upscaling_model_path=upscaling_model_path,
            upscaling_scale=upscaling_scale,
            output_compression=output_compression,
            output_quality=output_quality,
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
            # If client uses extra_body, merge it into the top-level payload
            payload: Dict[str, Any] = dict(body or {})
            extra = payload.pop("extra_body", None)
            if isinstance(extra, dict):
                # Shallow-merge: only keys like fps/num_frames are expected
                payload.update(extra)
            # openai may turn extra_body to extra_json
            extra_json = payload.pop("extra_json", None)
            if isinstance(extra_json, dict):
                payload.update(extra_json)
            # Validate image input based on model task type
            has_image_input = payload.get("reference_url") or payload.get(
                "input_reference"
            )
            if task_type.requires_image_input() and not has_image_input:
                raise HTTPException(
                    status_code=400,
                    detail="input_reference or reference_url is required for image-to-video generation",
                )
            # for non-multipart/form-data type
            if payload.get("reference_url"):
                try:
                    input_path = await _save_first_input_image(
                        payload.get("reference_url"),
                        request_id,
                        uploads_dir,
                        prefer_remote_source=server_args.input_save_path is None,
                    )
                except Exception as e:
                    raise HTTPException(
                        status_code=400,
                        detail=f"Failed to process image source: {str(e)}",
                    )
                payload["input_reference"] = input_path
            req = VideoGenerationsRequest(**payload)
        except Exception as e:
            raise HTTPException(status_code=400, detail=f"Invalid request body: {e}")

    # Resolve per-request output_path override
    effective_output_path = req.output_path or server_args.output_path
    if effective_output_path is None:
        output_tmp = tempfile.mkdtemp(prefix="sglang_output_")
        temp_dirs.append(output_tmp)
        effective_output_path = output_tmp
        output_persistent = False

    # Inject resolved output_path so _build_video_sampling_params picks it up
    req.output_path = effective_output_path

    logger.debug(f"Server received from create_video endpoint: req={req}")

    try:
        sampling_params = _build_video_sampling_params(request_id, req)
    except (ValueError, TypeError) as e:
        raise HTTPException(status_code=400, detail=str(e))

    job = _video_job_from_sampling(request_id, req, sampling_params)
    await VIDEO_STORE.upsert(request_id, job)

    # Build Req for scheduler
    trace_headers = extract_trace_headers(request.headers)
    batch = prepare_request(
        server_args=server_args,
        sampling_params=sampling_params,
        external_trace_header=trace_headers,
    )
    # Add diffusers_kwargs if provided
    if req.diffusers_kwargs:
        batch.extra["diffusers_kwargs"] = req.diffusers_kwargs
    # Enqueue the job asynchronously and return immediately
    asyncio.create_task(
        _dispatch_job_async(
            request_id,
            batch,
            temp_dirs=temp_dirs or None,
            output_persistent=output_persistent,
        )
    )
    return VideoResponse(**job)
```
**EN:** This block defines function `create_video`. It creates video. Key calls include `router.post`, `Form`, `File`, `request.headers.get.lower`, and `generate_request_id`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request`, `prompt`, `input_reference`, `reference_url`, and `model` drive the behavior in this section.
**CN:** 该代码块定义了函数 `create_video`。 它用于创建video。 关键调用包括 `router.post`、`Form`、`File`、`request.headers.get.lower` 和 `generate_request_id`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request`、`prompt`、`input_reference`、`reference_url` 和 `model` 等参数驱动。

### Lines 388-413: `list_videos` implementation / `list_videos` 实现
```python
@router.get("", response_model=VideoListResponse)
async def list_videos(
    after: Optional[str] = Query(None),
    limit: Optional[int] = Query(None, ge=1, le=100),
    order: Optional[str] = Query("desc"),
):
    # Normalize order
    order = (order or "desc").lower()
    if order not in ("asc", "desc"):
        order = "desc"
    jobs = await VIDEO_STORE.list_values()

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
    items = [VideoResponse(**j) for j in jobs]
    return VideoListResponse(data=items)
```
**EN:** This block defines function `list_videos`. It handles list videos logic. Key calls include `router.get`, `Query`, `lower`, `jobs.sort`, and `VideoListResponse`. The implementation branches on conditions, handles exceptional paths. Parameters such as `after`, `limit`, and `order` drive the behavior in this section.
**CN:** 该代码块定义了函数 `list_videos`。 它用于处理 list videos 相关逻辑。 关键调用包括 `router.get`、`Query`、`lower`、`jobs.sort` 和 `VideoListResponse`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `after`、`limit` 和 `order` 等参数驱动。

### Lines 416-421: `retrieve_video` implementation / `retrieve_video` 实现
```python
@router.get("/{video_id}", response_model=VideoResponse)
async def retrieve_video(video_id: str = Path(...)):
    job = await VIDEO_STORE.get(video_id)
    if not job:
        raise HTTPException(status_code=404, detail="Video not found")
    return VideoResponse(**job)
```
**EN:** This block defines function `retrieve_video`. It handles retrieve video logic. Key calls include `router.get`, `Path`, `VideoResponse`, `VIDEO_STORE.get`, and `HTTPException`. The implementation branches on conditions. Parameters such as `video_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `retrieve_video`。 它用于处理 retrieve video 相关逻辑。 关键调用包括 `router.get`、`Path`、`VideoResponse`、`VIDEO_STORE.get` 和 `HTTPException`。 实现中包含条件分支。 本段逻辑主要由 `video_id` 等参数驱动。

### Lines 425-432: `delete_video` implementation / `delete_video` 实现
```python
@router.delete("/{video_id}", response_model=VideoResponse)
async def delete_video(video_id: str = Path(...)):
    job = await VIDEO_STORE.pop(video_id)
    if not job:
        raise HTTPException(status_code=404, detail="Video not found")
    # Mark as deleted in response semantics
    job["status"] = "deleted"
    return VideoResponse(**job)
```
**EN:** This block defines function `delete_video`. It handles delete video logic. Key calls include `router.delete`, `Path`, `VideoResponse`, `VIDEO_STORE.pop`, and `HTTPException`. The implementation branches on conditions. Parameters such as `video_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `delete_video`。 它用于处理 delete video 相关逻辑。 关键调用包括 `router.delete`、`Path`、`VideoResponse`、`VIDEO_STORE.pop` 和 `HTTPException`。 实现中包含条件分支。 本段逻辑主要由 `video_id` 等参数驱动。

### Lines 435-456: `download_video_content` implementation / `download_video_content` 实现
```python
@router.get("/{video_id}/content")
async def download_video_content(
    video_id: str = Path(...), variant: Optional[str] = Query(None)
):
    job = await VIDEO_STORE.get(video_id)
    if not job:
        raise HTTPException(status_code=404, detail="Video not found")

    if job.get("url"):
        raise HTTPException(
            status_code=400,
            detail=f"Video has been uploaded to cloud storage. Please use the cloud URL: {job.get('url')}",
        )

    file_path = job.get("file_path")
    if not file_path or not os.path.exists(file_path):
        raise HTTPException(status_code=404, detail="Generation is still in-progress")

    media_type = "video/mp4"  # default variant
    return FileResponse(
        path=file_path, media_type=media_type, filename=os.path.basename(file_path)
    )
```
**EN:** This block defines function `download_video_content`. It handles download video content logic. Key calls include `router.get`, `Path`, `Query`, `job.get`, and `FileResponse`. The implementation branches on conditions. Parameters such as `video_id`, and `variant` drive the behavior in this section.
**CN:** 该代码块定义了函数 `download_video_content`。 它用于处理 download video content 相关逻辑。 关键调用包括 `router.get`、`Path`、`Query`、`job.get` 和 `FileResponse`。 实现中包含条件分支。 本段逻辑主要由 `video_id` 和 `variant` 等参数驱动。

## Key Concepts / 关键概念
- `_build_video_sampling_params`: Resolve video-specific defaults (fps, seconds → num_frames) then delegate to the shared build_sampling_params. / 顶层函数，用于构建video sampling params。
- `_video_job_from_sampling`: Top-level function that handles video job from sampling logic. / 顶层函数，用于处理 video job from sampling 相关逻辑。
- `_save_first_input_image`: Save the first input image from a list of sources and return its path. / 顶层函数，用于保存first input image。
- `_dispatch_job_async`: Top-level function that handles dispatch job async logic. / 顶层函数，用于处理 dispatch job async 相关逻辑。
- `create_video`: Top-level function that creates video. / 顶层函数，用于创建video。
- `list_videos`: Top-level function that handles list videos logic. / 顶层函数，用于处理 list videos 相关逻辑。
- `retrieve_video`: Top-level function that handles retrieve video logic. / 顶层函数，用于处理 retrieve video 相关逻辑。
- `delete_video`: Top-level function that handles delete video logic. / 顶层函数，用于处理 delete video 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `json`, `os`, `shutil`, `tempfile`, `time`, `typing`
- **Third-party / 第三方依赖**: `fastapi`, `fastapi.responses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.openai.protocol`, `sglang.multimodal_gen.runtime.entrypoints.openai.storage`, `sglang.multimodal_gen.runtime.entrypoints.openai.stores`, `sglang.multimodal_gen.runtime.entrypoints.openai.utils`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.observability.trace`, `sglang.multimodal_gen.runtime.scheduler_client`

- **Total lines / 总行数**: 456
