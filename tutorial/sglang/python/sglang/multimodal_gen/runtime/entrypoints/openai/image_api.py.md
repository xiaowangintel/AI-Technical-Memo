# image_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/image_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `_get_extra_field`, `_read_b64_for_paths`, and `_build_image_response_kwargs`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `_get_extra_field`、`_read_b64_for_paths` 和 `_build_image_response_kwargs` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-42: module setup and imports / 模块初始化与导入
```python
import base64
import contextlib
import os
import time
from typing import List, Optional

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

from sglang.multimodal_gen.configs.sample.sampling_params import generate_request_id
from sglang.multimodal_gen.runtime.entrypoints.openai.protocol import (
    ImageGenerationsRequest,
    ImageResponse,
    ImageResponseData,
)
from sglang.multimodal_gen.runtime.entrypoints.openai.storage import cloud_storage
from sglang.multimodal_gen.runtime.entrypoints.openai.stores import IMAGE_STORE
from sglang.multimodal_gen.runtime.entrypoints.openai.utils import (
    add_common_data_to_response,
    build_sampling_params,
    choose_output_image_ext,
    merge_image_input_list,
    process_generation_batch,
    save_image_to_path,
    temp_dir_if_disabled,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.srt.observability.trace import extract_trace_headers
```
**EN:** This block establishes the module context and imports `base64`, `contextlib`, `os`, `time`, `typing`, and `fastapi`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `base64`、`contextlib`、`os`、`time`、`typing` 和 `fastapi`。这些依赖为后续实现提供所需符号。

### Lines 44-44: supporting statements / 辅助语句
```python
router = APIRouter(prefix="/v1/images", tags=["images"])
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `router`. The code collaborates with `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `router` 等名称。 代码会与 `APIRouter` 协同工作。

### Lines 47-53: `_get_extra_field` implementation / `_get_extra_field` 实现
```python
def _get_extra_field(request, field_name):
    """Get a field from model_extra, with fallback to nested extra_body dict."""
    extra = request.model_extra or {}
    value = extra.get(field_name)
    if value is None and isinstance(extra.get("extra_body"), dict):
        value = extra["extra_body"].get(field_name)
    return value
```
**EN:** This block defines function `_get_extra_field`. Get a field from model_extra, with fallback to nested extra_body dict. Key calls include `extra.get`, and `isinstance`. The implementation branches on conditions. Parameters such as `request`, and `field_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_extra_field`。 它用于获取extra field。 关键调用包括 `extra.get` 和 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `request` 和 `field_name` 等参数驱动。

### Lines 56-62: `_read_b64_for_paths` implementation / `_read_b64_for_paths` 实现
```python
def _read_b64_for_paths(paths: list[str]) -> list[str]:
    """Read and base64-encode each file. Must be called before cloud upload deletes them."""
    result = []
    for path in paths:
        with open(path, "rb") as f:
            result.append(base64.b64encode(f.read()).decode("utf-8"))
    return result
```
**EN:** This block defines function `_read_b64_for_paths`. Read and base64-encode each file. Must be called before cloud upload deletes them. Key calls include `open`, `result.append`, `base64.b64encode.decode`, `base64.b64encode`, and `f.read`. The implementation iterates over collections or steps, uses context-managed resources. Parameters such as `paths` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_read_b64_for_paths`。 它用于处理 read b64 for paths 相关逻辑。 关键调用包括 `open`、`result.append`、`base64.b64encode.decode`、`base64.b64encode` 和 `f.read`。 实现中会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `paths` 等参数驱动。

### Lines 65-123: `_build_image_response_kwargs` implementation / `_build_image_response_kwargs` 实现
```python
def _build_image_response_kwargs(
    save_file_path_list: list[str],
    resp_format: str,
    prompt: str,
    request_id: str,
    result: OutputBatch,
    *,
    b64_list: list[str] | None = None,
    cloud_url: str | None = None,
    fallback_url: str | None = None,
    is_persistent: bool = True,
) -> dict:
    """Build ImageResponse data list.

    For b64_json: uses pre-read b64_list (call _read_b64_for_paths first).
    For url: uses cloud_url or fallback_url.
    file_path is omitted when is_persistent=False to avoid exposing stale temp paths.
    """
    ret = None
    if resp_format == "b64_json":
        if not b64_list:
            raise ValueError("b64_list required for b64_json response_format")
        data = [
            ImageResponseData(
                b64_json=b64,
                revised_prompt=prompt,
                file_path=os.path.abspath(path) if is_persistent else None,
            )
            for b64, path in zip(b64_list, save_file_path_list)
        ]
        ret = {"data": data}
    elif resp_format == "url":
        url = cloud_url or fallback_url
        if not url:
            raise HTTPException(
                status_code=400,
                detail="response_format='url' requires cloud storage to be configured.",
            )
        ret = {
            "data": [
                ImageResponseData(
                    url=url,
                    revised_prompt=prompt,
                    file_path=(
                        os.path.abspath(save_file_path_list[0])
                        if is_persistent
                        else None
                    ),
                )
            ],
        }
    else:
        raise HTTPException(
            status_code=400, detail=f"response_format={resp_format} is not supported"
        )

    ret = add_common_data_to_response(ret, request_id=request_id, result=result)

    return ret
```
**EN:** This block defines function `_build_image_response_kwargs`. Build ImageResponse data list. For b64_json: uses pre-read b64_list (call _read_b64_for_paths first). Key calls include `add_common_data_to_response`, `ValueError`, `ImageResponseData`, `HTTPException`, and `zip`. The implementation branches on conditions. Parameters such as `save_file_path_list`, `resp_format`, `prompt`, `request_id`, and `result` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_image_response_kwargs`。 它用于构建image response kwargs。 关键调用包括 `add_common_data_to_response`、`ValueError`、`ImageResponseData`、`HTTPException` 和 `zip`。 实现中包含条件分支。 本段逻辑主要由 `save_file_path_list`、`resp_format`、`prompt`、`request_id` 和 `result` 等参数驱动。

### Lines 126-209: `generations` implementation / `generations` 实现
```python
@router.post("/generations", response_model=ImageResponse)
async def generations(
    request: ImageGenerationsRequest,
    raw_request: Request,
):
    request_id = generate_request_id()
    server_args = get_global_server_args()
    ext = choose_output_image_ext(request.output_format, request.background)

    with temp_dir_if_disabled(server_args.output_path) as output_dir:
        sampling = build_sampling_params(
            request_id,
            prompt=request.prompt,
            size=request.size,
            width=request.width,
            height=request.height,
            num_outputs_per_prompt=max(1, min(int(request.n or 1), 10)),
            output_file_name=f"{request_id}.{ext}",
            output_path=output_dir,
            seed=request.seed,
            generator_device=request.generator_device,
            num_inference_steps=request.num_inference_steps,
            guidance_scale=request.guidance_scale,
            true_cfg_scale=request.true_cfg_scale,
            negative_prompt=request.negative_prompt,
            enable_teacache=request.enable_teacache,
            output_compression=request.output_compression,
            output_quality=request.output_quality,
            diffusers_kwargs=request.diffusers_kwargs,
            enable_upscaling=request.enable_upscaling,
            upscaling_model_path=request.upscaling_model_path,
            upscaling_scale=request.upscaling_scale,
            perf_dump_path=request.perf_dump_path,
            use_pe=_get_extra_field(request, "use_pe"),
        )
        trace_headers = extract_trace_headers(raw_request.headers)
        batch = prepare_request(
            server_args=server_args,
            sampling_params=sampling,
            external_trace_header=trace_headers,
        )
        # Add diffusers_kwargs if provided
        if request.diffusers_kwargs:
            batch.extra["diffusers_kwargs"] = request.diffusers_kwargs

        save_file_path_list, result = await process_generation_batch(
            async_scheduler_client, batch
        )
        save_file_path = save_file_path_list[0]
        resp_format = (request.response_format or "b64_json").lower()

        # read b64 before cloud upload may delete the local file
        b64_list = (
            _read_b64_for_paths(save_file_path_list)
            if resp_format == "b64_json"
            else None
        )

        cloud_url = await cloud_storage.upload_and_cleanup(save_file_path)

        is_persistent = server_args.output_path is not None
        await IMAGE_STORE.upsert(
            request_id,
            {
                "id": request_id,
                "created_at": int(time.time()),
                "file_path": None if cloud_url or not is_persistent else save_file_path,
                "url": cloud_url,
            },
        )

        response_kwargs = _build_image_response_kwargs(
            save_file_path_list,
            resp_format,
            request.prompt,
            request_id,
            result,
            b64_list=b64_list,
            cloud_url=cloud_url,
            fallback_url=f"/v1/images/{request_id}/content" if is_persistent else None,
            is_persistent=is_persistent,
        )

    return ImageResponse(**response_kwargs)
```
**EN:** This block defines function `generations`. It handles generations logic. Key calls include `router.post`, `generate_request_id`, `get_global_server_args`, `choose_output_image_ext`, and `ImageResponse`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request`, and `raw_request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `generations`。 它用于处理 generations 相关逻辑。 关键调用包括 `router.post`、`generate_request_id`、`get_global_server_args`、`choose_output_image_ext` 和 `ImageResponse`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request` 和 `raw_request` 等参数驱动。

### Lines 212-347: `edits` implementation / `edits` 实现
```python
@router.post("/edits", response_model=ImageResponse)
async def edits(
    raw_request: Request,
    image: Optional[List[UploadFile]] = File(None),
    image_array: Optional[List[UploadFile]] = File(None, alias="image[]"),
    url: Optional[List[str]] = Form(None),
    url_array: Optional[List[str]] = Form(None, alias="url[]"),
    prompt: str = Form(...),
    mask: Optional[UploadFile] = File(None),
    model: Optional[str] = Form(None),
    n: Optional[int] = Form(1),
    response_format: Optional[str] = Form(None),
    size: Optional[str] = Form(None),
    output_format: Optional[str] = Form(None),
    background: Optional[str] = Form("auto"),
    seed: Optional[int] = Form(None),
    generator_device: Optional[str] = Form("cuda"),
    user: Optional[str] = Form(None),
    negative_prompt: Optional[str] = Form(None),
    guidance_scale: Optional[float] = Form(None),
    true_cfg_scale: Optional[float] = Form(None),
    num_inference_steps: Optional[int] = Form(None),
    output_quality: Optional[str] = Form("default"),
    output_compression: Optional[int] = Form(None),
    enable_teacache: Optional[bool] = Form(False),
    enable_upscaling: Optional[bool] = Form(False),
    upscaling_model_path: Optional[str] = Form(None),
    upscaling_scale: Optional[int] = Form(4),
    num_frames: int = Form(1),
):
    request_id = generate_request_id()
    server_args = get_global_server_args()
    # Resolve images from either `image` or `image[]` (OpenAI SDK sends `image[]` when list is provided)
    images = image or image_array
    urls = url or url_array

    if (not images or len(images) == 0) and (not urls or len(urls) == 0):
        raise HTTPException(
            status_code=422, detail="Field 'image' or 'url' is required"
        )

    image_list = merge_image_input_list(images, urls)

    with contextlib.ExitStack() as stack:
        uploads_dir = stack.enter_context(
            temp_dir_if_disabled(server_args.input_save_path)
        )
        output_dir = stack.enter_context(temp_dir_if_disabled(server_args.output_path))

        input_paths = []
        try:
            for idx, img in enumerate(image_list):
                filename = img.filename if hasattr(img, "filename") else f"image_{idx}"
                input_path = await save_image_to_path(
                    img,
                    os.path.join(uploads_dir, f"{request_id}_{idx}_{filename}"),
                    prefer_remote_source=server_args.input_save_path is None,
                )
                input_paths.append(input_path)
        except Exception as e:
            raise HTTPException(
                status_code=400,
                detail=f"Failed to process image source: {str(e)}",
            )

        ext = choose_output_image_ext(output_format, background)
        sampling = build_sampling_params(
            request_id,
            prompt=prompt,
            size=size,
            num_outputs_per_prompt=max(1, min(int(n or 1), 10)),
            output_file_name=f"{request_id}.{ext}",
            output_path=output_dir,
            image_path=input_paths,
            seed=seed,
            generator_device=generator_device,
            negative_prompt=negative_prompt,
            guidance_scale=guidance_scale,
            true_cfg_scale=true_cfg_scale,
            num_inference_steps=num_inference_steps,
            enable_teacache=enable_teacache,
            num_frames=num_frames,
            output_compression=output_compression,
            output_quality=output_quality,
            enable_upscaling=enable_upscaling,
            upscaling_model_path=upscaling_model_path,
            upscaling_scale=upscaling_scale,
        )
        trace_headers = extract_trace_headers(raw_request.headers)
        batch = prepare_request(
            server_args=server_args,
            sampling_params=sampling,
            external_trace_header=trace_headers,
        )
        save_file_path_list, result = await process_generation_batch(
            async_scheduler_client, batch
        )
        save_file_path = save_file_path_list[0]
        resp_format = (response_format or "b64_json").lower()

        # read b64 before cloud upload may delete the local file
        b64_list = (
            _read_b64_for_paths(save_file_path_list)
            if resp_format == "b64_json"
            else None
        )

        cloud_url = await cloud_storage.upload_and_cleanup(save_file_path)

        is_persistent = server_args.output_path is not None
        is_input_persistent = server_args.input_save_path is not None
        await IMAGE_STORE.upsert(
            request_id,
            {
                "id": request_id,
                "created_at": int(time.time()),
                "file_path": None if cloud_url or not is_persistent else save_file_path,
                "url": cloud_url,
                "input_image_paths": input_paths if is_input_persistent else None,
                "num_input_images": len(input_paths),
            },
        )

        response_kwargs = _build_image_response_kwargs(
            save_file_path_list,
            resp_format,
            prompt,
            request_id,
            result,
            b64_list=b64_list,
            cloud_url=cloud_url,
            fallback_url=f"/v1/images/{request_id}/content" if is_persistent else None,
            is_persistent=is_persistent,
        )

    return ImageResponse(**response_kwargs)
```
**EN:** This block defines function `edits`. It handles edits logic. Key calls include `router.post`, `File`, `Form`, `generate_request_id`, and `get_global_server_args`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `raw_request`, `image`, `image_array`, `url`, and `url_array` drive the behavior in this section.
**CN:** 该代码块定义了函数 `edits`。 它用于处理 edits 相关逻辑。 关键调用包括 `router.post`、`File`、`Form`、`generate_request_id` 和 `get_global_server_args`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `raw_request`、`image`、`image_array`、`url` 和 `url_array` 等参数驱动。

### Lines 350-382: `download_image_content` implementation / `download_image_content` 实现
```python
@router.get("/{image_id}/content")
async def download_image_content(
    image_id: str = Path(...), variant: Optional[str] = Query(None)
):
    item = await IMAGE_STORE.get(image_id)
    if not item:
        raise HTTPException(status_code=404, detail="Image not found")

    if item.get("url"):
        raise HTTPException(
            status_code=400,
            detail=f"Image has been uploaded to cloud storage. Please use the cloud URL: {item.get('url')}",
        )

    file_path = item.get("file_path")
    if not file_path:
        raise HTTPException(
            status_code=404,
            detail="Image was not persisted on disk (output_path is disabled). Use b64_json response_format or configure cloud storage.",
        )
    if not os.path.exists(file_path):
        raise HTTPException(status_code=404, detail="Image is still being generated")

    ext = os.path.splitext(file_path)[1].lower()
    media_type = "image/jpeg"
    if ext == ".png":
        media_type = "image/png"
    elif ext == ".webp":
        media_type = "image/webp"

    return FileResponse(
        path=file_path, media_type=media_type, filename=os.path.basename(file_path)
    )
```
**EN:** This block defines function `download_image_content`. It handles download image content logic. Key calls include `router.get`, `Path`, `Query`, `item.get`, and `os.path.splitext.lower`. The implementation branches on conditions. Parameters such as `image_id`, and `variant` drive the behavior in this section.
**CN:** 该代码块定义了函数 `download_image_content`。 它用于处理 download image content 相关逻辑。 关键调用包括 `router.get`、`Path`、`Query`、`item.get` 和 `os.path.splitext.lower`。 实现中包含条件分支。 本段逻辑主要由 `image_id` 和 `variant` 等参数驱动。

## Key Concepts / 关键概念
- `_get_extra_field`: Get a field from model_extra, with fallback to nested extra_body dict. / 顶层函数，用于获取extra field。
- `_read_b64_for_paths`: Read and base64-encode each file. / 顶层函数，用于处理 read b64 for paths 相关逻辑。
- `_build_image_response_kwargs`: Build ImageResponse data list. / 顶层函数，用于构建image response kwargs。
- `generations`: Top-level function that handles generations logic. / 顶层函数，用于处理 generations 相关逻辑。
- `edits`: Top-level function that handles edits logic. / 顶层函数，用于处理 edits 相关逻辑。
- `download_image_content`: Top-level function that handles download image content logic. / 顶层函数，用于处理 download image content 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `base64`, `contextlib`, `os`, `time`, `typing`
- **Third-party / 第三方依赖**: `fastapi`, `fastapi.responses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.openai.protocol`, `sglang.multimodal_gen.runtime.entrypoints.openai.storage`, `sglang.multimodal_gen.runtime.entrypoints.openai.stores`, `sglang.multimodal_gen.runtime.entrypoints.openai.utils`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.srt.observability.trace`

- **Total lines / 总行数**: 382
