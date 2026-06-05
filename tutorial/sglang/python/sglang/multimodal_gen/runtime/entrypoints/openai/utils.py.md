# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `temp_dir_if_disabled`, `_parse_size`, and `choose_output_image_ext`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `temp_dir_if_disabled`、`_parse_size` 和 `choose_output_image_ext` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-36: module setup and imports / 模块初始化与导入
```python
import asyncio
import base64
import os
import re
import shutil
import tempfile
import time
from contextlib import contextmanager
from typing import Any, Generator, List, Optional, Union

import httpx
from fastapi import UploadFile

from sglang.multimodal_gen.configs.sample.sampling_params import (
    DataType,
    SamplingParams,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    ListLorasReq,
    MergeLoraWeightsReq,
    SetLoraReq,
    ShutdownReq,
    UnmergeLoraWeightsReq,
    format_lora_message,
    save_outputs,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.scheduler_client import AsyncSchedulerClient
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import (
    init_logger,
    log_batch_completion,
    log_generation_timer,
)
from sglang.multimodal_gen.runtime.utils.trace_wrapper import trace_req
```
**EN:** This block establishes the module context and imports `asyncio`, `base64`, `os`, `re`, `shutil`, and `tempfile`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `asyncio`、`base64`、`os`、`re`、`shutil` 和 `tempfile`。这些依赖为后续实现提供所需符号。

### Lines 39-52: supporting statements / 辅助语句
```python
__all__ = [
    "SetLoraReq",
    "MergeLoraWeightsReq",
    "UnmergeLoraWeightsReq",
    "ListLorasReq",
    "ShutdownReq",
    "format_lora_message",
]

logger = init_logger(__name__)

OUTPUT_QUALITY_MAPPER = {"maximum": 100, "high": 90, "medium": 55, "low": 35}
DEFAULT_FPS = 24
DEFAULT_VIDEO_SECONDS = 4
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`, `logger`, `OUTPUT_QUALITY_MAPPER`, `DEFAULT_FPS`, and `DEFAULT_VIDEO_SECONDS`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__`、`logger`、`OUTPUT_QUALITY_MAPPER`、`DEFAULT_FPS` 和 `DEFAULT_VIDEO_SECONDS` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 55-69: `temp_dir_if_disabled` implementation / `temp_dir_if_disabled` 实现
```python
@contextmanager
def temp_dir_if_disabled(
    configured_path: str | None,
) -> Generator[str, None, None]:
    """Yield *configured_path* when it is set, otherwise create a temporary
    directory that is automatically removed when the context exits."""
    if configured_path is not None:
        os.makedirs(configured_path, exist_ok=True)
        yield configured_path
    else:
        tmp = tempfile.mkdtemp(prefix="sglang_")
        try:
            yield tmp
        finally:
            shutil.rmtree(tmp, ignore_errors=True)
```
**EN:** This block defines function `temp_dir_if_disabled`. Yield *configured_path* when it is set, otherwise create a temporary directory that is automatically removed when the context exits. Key calls include `os.makedirs`, `tempfile.mkdtemp`, and `shutil.rmtree`. The implementation branches on conditions, handles exceptional paths. Parameters such as `configured_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `temp_dir_if_disabled`。 它用于处理 temp dir if disabled 相关逻辑。 关键调用包括 `os.makedirs`、`tempfile.mkdtemp` 和 `shutil.rmtree`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `configured_path` 等参数驱动。

### Lines 72-80: `_parse_size` implementation / `_parse_size` 实现
```python
def _parse_size(size: str) -> tuple[int, int] | tuple[None, None]:
    try:
        parts = size.lower().replace(" ", "").split("x")
        if len(parts) != 2:
            raise ValueError
        w, h = int(parts[0]), int(parts[1])
        return w, h
    except Exception:
        return None, None
```
**EN:** This block defines function `_parse_size`. It parses size. Key calls include `size.lower.replace.split`, `len`, `int`, `size.lower.replace`, and `size.lower`. The implementation branches on conditions, handles exceptional paths. Parameters such as `size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_parse_size`。 它用于解析size。 关键调用包括 `size.lower.replace.split`、`len`、`int`、`size.lower.replace` 和 `size.lower`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `size` 等参数驱动。

### Lines 83-91: `choose_output_image_ext` implementation / `choose_output_image_ext` 实现
```python
def choose_output_image_ext(
    output_format: Optional[str], background: Optional[str]
) -> str:
    fmt = (output_format or "").lower()
    if fmt in {"png", "webp", "jpeg", "jpg"}:
        return "jpg" if fmt == "jpeg" else fmt
    if (background or "auto").lower() == "transparent":
        return "png"
    return "jpg"
```
**EN:** This block defines function `choose_output_image_ext`. It handles choose output image ext logic. Key calls include `lower`. The implementation branches on conditions. Parameters such as `output_format`, and `background` drive the behavior in this section.
**CN:** 该代码块定义了函数 `choose_output_image_ext`。 它用于处理 choose output image ext 相关逻辑。 关键调用包括 `lower`。 实现中包含条件分支。 本段逻辑主要由 `output_format` 和 `background` 等参数驱动。

### Lines 94-139: `build_sampling_params` implementation / `build_sampling_params` 实现
```python
def build_sampling_params(request_id: str, **kwargs) -> SamplingParams:
    """Build SamplingParams from request parameters.

    Handles size parsing, output_quality resolution, and None filtering before
    delegating to SamplingParams.from_user_sampling_params_args. Callers pass
    only the parameters they have; None values are stripped automatically so
    that SamplingParams defaults apply.
    """
    server_args = get_global_server_args()

    # pop HTTP-layer params that aren't SamplingParams fields
    output_quality = kwargs.pop("output_quality", None)

    has_explicit_compression = kwargs.get("output_compression") is not None

    # parse "WxH" size string if provided
    size = kwargs.pop("size", None)
    if size:
        w, h = _parse_size(size)
        if w is not None:
            # treat None dimensions as unset so parsed size can fill them
            if kwargs.get("width") is None:
                kwargs["width"] = w
            if kwargs.get("height") is None:
                kwargs["height"] = h

    # filter out None values to let SamplingParams defaults apply
    kwargs = {k: v for k, v in kwargs.items() if v is not None}
    kwargs.setdefault("save_output", True)

    sampling_params = SamplingParams.from_user_sampling_params_args(
        model_path=server_args.model_path,
        server_args=server_args,
        request_id=request_id,
        **kwargs,
    )

    # resolve output_quality → output_compression with the correct data_type.
    # SamplingParams.__post_init__ may have resolved with the wrong data_type
    # (default VIDEO) before _adjust() set the correct one.
    if not has_explicit_compression and output_quality is not None:
        resolved = adjust_output_quality(output_quality, sampling_params.data_type)
        if resolved is not None:
            sampling_params.output_compression = resolved

    return sampling_params
```
**EN:** This block defines function `build_sampling_params`. Build SamplingParams from request parameters. Handles size parsing, output_quality resolution, and None filtering before delegating to SamplingParams.from_user_sampling_params_args. Key calls include `get_global_server_args`, `kwargs.pop`, `kwargs.setdefault`, `SamplingParams.from_user_sampling_params_args`, and `kwargs.get`. The implementation branches on conditions. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了函数 `build_sampling_params`。 它用于构建sampling params。 关键调用包括 `get_global_server_args`、`kwargs.pop`、`kwargs.setdefault`、`SamplingParams.from_user_sampling_params_args` 和 `kwargs.get`。 实现中包含条件分支。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 142-153: `save_image_to_path` implementation / `save_image_to_path` 实现
```python
async def save_image_to_path(
    image: Union[UploadFile, str],
    target_path: str,
    *,
    prefer_remote_source: bool = False,
) -> str:
    input_path = await _maybe_url_image(
        image, target_path, prefer_remote_source=prefer_remote_source
    )
    if input_path is None:
        input_path = await _save_upload_to_path(image, target_path)
    return input_path
```
**EN:** This block defines function `save_image_to_path`. It saves image to path. Key calls include `_maybe_url_image`, and `_save_upload_to_path`. The implementation branches on conditions. Parameters such as `image`, and `target_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `save_image_to_path`。 它用于保存image to path。 关键调用包括 `_maybe_url_image` 和 `_save_upload_to_path`。 实现中包含条件分支。 本段逻辑主要由 `image` 和 `target_path` 等参数驱动。

### Lines 157-162: `_save_upload_to_path` implementation / `_save_upload_to_path` 实现
```python
async def _save_upload_to_path(upload: UploadFile, target_path: str) -> str:
    os.makedirs(os.path.dirname(target_path), exist_ok=True)
    content = await upload.read()
    with open(target_path, "wb") as f:
        f.write(content)
    return target_path
```
**EN:** This block defines function `_save_upload_to_path`. It saves upload to path. Key calls include `os.makedirs`, `os.path.dirname`, `upload.read`, `open`, and `f.write`. The implementation uses context-managed resources. Parameters such as `upload`, and `target_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_save_upload_to_path`。 它用于保存upload to path。 关键调用包括 `os.makedirs`、`os.path.dirname`、`upload.read`、`open` 和 `f.write`。 实现中使用上下文管理资源。 本段逻辑主要由 `upload` 和 `target_path` 等参数驱动。

### Lines 165-189: `_maybe_url_image` implementation / `_maybe_url_image` 实现
```python
async def _maybe_url_image(
    img_url: str,
    target_path: str,
    *,
    prefer_remote_source: bool = False,
) -> str | None:
    if not isinstance(img_url, str):
        return None

    if img_url.lower().startswith(("http://", "https://")):
        # Only bypass persistence when the caller explicitly disables input saves.
        # Otherwise keep the prefetch outside the measured server stages.
        if prefer_remote_source:
            return img_url
        # download image from URL and persist on disk
        input_path = await _save_url_image_to_path(img_url, target_path)
        return input_path
    elif img_url.startswith("data:image"):
        if prefer_remote_source:
            return img_url
        # encode image base64 url and persist on disk
        input_path = await _save_base64_image_to_path(img_url, target_path)
        return input_path
    else:
        raise ValueError("Unsupported image url format")
```
**EN:** This block defines function `_maybe_url_image`. It handles maybe url image logic. Key calls include `img_url.lower.startswith`, `isinstance`, `img_url.startswith`, `img_url.lower`, and `_save_url_image_to_path`. The implementation branches on conditions. Parameters such as `img_url`, and `target_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_maybe_url_image`。 它用于处理 maybe url image 相关逻辑。 关键调用包括 `img_url.lower.startswith`、`isinstance`、`img_url.startswith`、`img_url.lower` 和 `_save_url_image_to_path`。 实现中包含条件分支。 本段逻辑主要由 `img_url` 和 `target_path` 等参数驱动。

### Lines 192-280: `_save_url_image_to_path` implementation / `_save_url_image_to_path` 实现
```python
async def _save_url_image_to_path(image_url: str, target_path: str) -> str:
    """Download image from URL and save to target path."""

    def _is_retryable_download_error(error: Exception) -> bool:
        if isinstance(error, httpx.HTTPStatusError):
            status_code = error.response.status_code
            # Retry on rate limit and transient server-side failures.
            return status_code == 429 or 500 <= status_code < 600
        # Retry on transient network/protocol issues.
        return isinstance(
            error,
            (
                httpx.TimeoutException,
                httpx.NetworkError,
                httpx.RemoteProtocolError,
            ),
        )

    os.makedirs(os.path.dirname(target_path), exist_ok=True)

    max_attempts = 3
    backoff_seconds = 0.2
    last_error: Exception | None = None

    try:
        async with httpx.AsyncClient(follow_redirects=True) as client:
            for attempt in range(1, max_attempts + 1):
                try:
                    response = await client.get(image_url, timeout=10.0)
                    response.raise_for_status()

                    # Determine file extension from content type or URL after downloading
                    if not os.path.splitext(target_path)[1]:
                        content_type = response.headers.get("content-type", "").lower()

                        url_path = image_url.split("?")[0]
                        _, url_ext = os.path.splitext(url_path)
                        url_ext = url_ext.lower()

                        if url_ext in {
                            ".jpg",
                            ".jpeg",
                            ".png",
                            ".webp",
                            ".gif",
                            ".bmp",
                        }:
                            ext = ".jpg" if url_ext == ".jpeg" else url_ext
                        elif content_type.startswith("image/"):
                            if "jpeg" in content_type or "jpg" in content_type:
                                ext = ".jpg"
                            elif "png" in content_type:
                                ext = ".png"
                            elif "webp" in content_type:
                                ext = ".webp"
                            else:
                                ext = ".jpg"  # Default to jpg
                        elif content_type == "application/octet-stream":
                            # for octet-stream, if we couldn't get it from URL, default to jpg
                            ext = ".jpg"
                        else:
                            raise ValueError(
                                f"URL does not point to an image. Content-Type: {content_type}"
                            )
                        target_path = f"{target_path}{ext}"

                    with open(target_path, "wb") as f:
                        f.write(response.content)

                    return target_path
                except Exception as e:
                    last_error = e
                    if attempt == max_attempts or not _is_retryable_download_error(e):
                        raise
                    wait_s = backoff_seconds * (2 ** (attempt - 1))
                    logger.warning(
                        "Retrying image download (%s/%s) for %s after %.1fs due to: %s",
                        attempt,
                        max_attempts,
                        image_url,
                        wait_s,
                        e,
                    )
                    await asyncio.sleep(wait_s)
    except Exception as e:
        final_error = last_error or e
        raise Exception(
            f"Failed to download image from URL {image_url}: {str(final_error)}"
        )
```
**EN:** This block defines function `_save_url_image_to_path`. Download image from URL and save to target path. Key calls include `os.makedirs`, `isinstance`, `os.path.dirname`, `httpx.AsyncClient`, and `range`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `image_url`, and `target_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_save_url_image_to_path`。 它用于保存url image to path。 关键调用包括 `os.makedirs`、`isinstance`、`os.path.dirname`、`httpx.AsyncClient` 和 `range`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `image_url` 和 `target_path` 等参数驱动。

### Lines 283-320: `_save_base64_image_to_path` implementation / `_save_base64_image_to_path` 实现
```python
async def _save_base64_image_to_path(base64_data: str, target_path: str) -> str:
    """Decode base64 image data and save to target path."""

    _B64_FMT_HINT = (
        "Failed to decode base64 image. "
        "Expected format: `data:[<media-type>];base64,<data>`"
    )

    # split `data:[<media-type>][;base64],<data>` to media-type base64 data
    pattern = r"data:(.*?)(;base64)?,(.*)"
    match = re.match(pattern, base64_data)
    if not match:
        raise ValueError(_B64_FMT_HINT)
    media_type = match.group(1)
    is_base64 = match.group(2)
    if not is_base64:
        raise ValueError(f"{_B64_FMT_HINT} (missing ;base64 marker)")
    data = match.group(3)
    if not data:
        raise ValueError(f"{_B64_FMT_HINT} (empty data payload)")
    # get ext from url
    if media_type.startswith("image/"):
        ext = media_type.split("/")[-1].lower()
        if ext == "jpeg":
            ext = "jpg"
    else:
        ext = "jpg"
    target_path = f"{target_path}.{ext}"
    os.makedirs(os.path.dirname(target_path), exist_ok=True)

    try:
        image_data = base64.b64decode(data)
        with open(target_path, "wb") as f:
            f.write(image_data)

        return target_path
    except Exception as e:
        raise Exception(f"Failed to decode base64 image: {str(e)}")
```
**EN:** This block defines function `_save_base64_image_to_path`. Decode base64 image data and save to target path. Key calls include `re.match`, `match.group`, `media_type.startswith`, `os.makedirs`, and `ValueError`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `base64_data`, and `target_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_save_base64_image_to_path`。 它用于保存base64 image to path。 关键调用包括 `re.match`、`match.group`、`media_type.startswith`、`os.makedirs` 和 `ValueError`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `base64_data` 和 `target_path` 等参数驱动。

### Lines 323-370: `process_generation_batch` implementation / `process_generation_batch` 实现
```python
async def process_generation_batch(
    scheduler_client: AsyncSchedulerClient,
    batch,
) -> tuple[list[str], OutputBatch]:
    total_start_time = time.perf_counter()
    with trace_req(batch.trace_ctx), log_generation_timer(logger, batch.prompt):
        result = await scheduler_client.forward([batch])

        if result.output is None and result.output_file_paths is None:
            error_msg = result.error or "Unknown error"
            raise RuntimeError(
                f"Model generation returned no output. Error from scheduler: {error_msg}"
            )

        if result.output_file_paths:
            save_file_path_list = result.output_file_paths
        else:
            num_outputs = len(result.output)
            save_file_path_list = save_outputs(
                result.output,
                batch.data_type,
                batch.fps,
                batch.save_output,
                lambda idx: str(batch.output_file_path(num_outputs, idx)),
                audio=result.audio,
                audio_sample_rate=result.audio_sample_rate,
                output_compression=batch.output_compression,
                enable_frame_interpolation=batch.enable_frame_interpolation,
                frame_interpolation_exp=batch.frame_interpolation_exp,
                frame_interpolation_scale=batch.frame_interpolation_scale,
                frame_interpolation_model_path=batch.frame_interpolation_model_path,
                enable_upscaling=batch.enable_upscaling,
                upscaling_model_path=batch.upscaling_model_path,
                upscaling_scale=batch.upscaling_scale,
            )

    total_time = time.perf_counter() - total_start_time
    if get_global_server_args().batching_max_size > 1:
        log_batch_completion(
            logger,
            len(save_file_path_list),
            total_time,
        )

    if result.peak_memory_mb and result.peak_memory_mb > 0:
        logger.info(f"Peak memory usage: {result.peak_memory_mb:.2f} MB")

    return save_file_path_list, result
```
**EN:** This block defines function `process_generation_batch`. It processes generation batch. Key calls include `time.perf_counter`, `trace_req`, `log_generation_timer`, `log_batch_completion`, and `logger.info`. The implementation branches on conditions, uses context-managed resources. Parameters such as `scheduler_client`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了函数 `process_generation_batch`。 它用于处理generation batch。 关键调用包括 `time.perf_counter`、`trace_req`、`log_generation_timer`、`log_batch_completion` 和 `logger.info`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `scheduler_client` 和 `batch` 等参数驱动。

### Lines 373-398: `merge_image_input_list` implementation / `merge_image_input_list` 实现
```python
def merge_image_input_list(*inputs: Union[List, Any, None]) -> List:
    """
    Merge multiple image input sources into a single list.

    This function handles both single items and lists of items, merging them
    into a single flattened list. Useful for processing images, URLs, or other
    multimedia inputs that can come as either single items or lists.

    Args:
        *inputs: Variable number of inputs, each can be None, single item, or list

    Returns:
        List: Flattened list of all non-None inputs

    Example:
        >>> merge_image_input_list(["img1", "img2"], "img3", None)
        ["img1", "img2", "img3"]
    """
    result = []
    for input_item in inputs:
        if input_item is not None:
            if isinstance(input_item, list):
                result.extend(input_item)
            else:
                result.append(input_item)
    return result
```
**EN:** This block defines function `merge_image_input_list`. Merge multiple image input sources into a single list. This function handles both single items and lists of items, merging them into a single flattened list. Key calls include `isinstance`, `result.extend`, and `result.append`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了函数 `merge_image_input_list`。 它用于合并image input list。 关键调用包括 `isinstance`、`result.extend` 和 `result.append`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 401-412: `add_common_data_to_response` implementation / `add_common_data_to_response` 实现
```python
def add_common_data_to_response(
    response: dict, request_id: str, result: OutputBatch
) -> dict:
    if result.peak_memory_mb and result.peak_memory_mb > 0:
        response["peak_memory_mb"] = result.peak_memory_mb

    if result.metrics and result.metrics.total_duration_s > 0:
        response["inference_time_s"] = result.metrics.total_duration_s

    response["id"] = request_id

    return response
```
**EN:** This block defines function `add_common_data_to_response`. It handles add common data to response logic. The implementation branches on conditions. Parameters such as `response`, `request_id`, and `result` drive the behavior in this section.
**CN:** 该代码块定义了函数 `add_common_data_to_response`。 它用于处理 add common data to response 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `response`、`request_id` 和 `result` 等参数驱动。

### Lines 415-418: `adjust_output_quality` implementation / `adjust_output_quality` 实现
```python
def adjust_output_quality(output_quality: str, data_type: DataType = None) -> int:
    if output_quality == "default":
        return 50 if data_type == DataType.VIDEO else 75
    return OUTPUT_QUALITY_MAPPER.get(output_quality, None)
```
**EN:** This block defines function `adjust_output_quality`. It handles adjust output quality logic. Key calls include `OUTPUT_QUALITY_MAPPER.get`. The implementation branches on conditions. Parameters such as `output_quality`, and `data_type` drive the behavior in this section.
**CN:** 该代码块定义了函数 `adjust_output_quality`。 它用于处理 adjust output quality 相关逻辑。 关键调用包括 `OUTPUT_QUALITY_MAPPER.get`。 实现中包含条件分支。 本段逻辑主要由 `output_quality` 和 `data_type` 等参数驱动。

## Key Concepts / 关键概念
- `temp_dir_if_disabled`: Yield *configured_path* when it is set, otherwise create a temporary directory that is automatically removed when the context exits. / 顶层函数，用于处理 temp dir if disabled 相关逻辑。
- `_parse_size`: Top-level function that parses size. / 顶层函数，用于解析size。
- `choose_output_image_ext`: Top-level function that handles choose output image ext logic. / 顶层函数，用于处理 choose output image ext 相关逻辑。
- `build_sampling_params`: Build SamplingParams from request parameters. / 顶层函数，用于构建sampling params。
- `save_image_to_path`: Top-level function that saves image to path. / 顶层函数，用于保存image to path。
- `_save_upload_to_path`: Top-level function that saves upload to path. / 顶层函数，用于保存upload to path。
- `_maybe_url_image`: Top-level function that handles maybe url image logic. / 顶层函数，用于处理 maybe url image 相关逻辑。
- `_save_url_image_to_path`: Download image from URL and save to target path. / 顶层函数，用于保存url image to path。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `base64`, `os`, `re`, `shutil`, `tempfile`, `time`, `contextlib`, `typing`
- **Third-party / 第三方依赖**: `httpx`, `fastapi`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.trace_wrapper`

- **Total lines / 总行数**: 418
