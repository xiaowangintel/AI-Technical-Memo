# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `ImageResponseData`, `ImageResponse`, and `ImageGenerationsRequest`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `ImageResponseData`、`ImageResponse` 和 `ImageGenerationsRequest` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module setup and imports / 模块初始化与导入
```python
import time
import uuid
from abc import ABC
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional, Union

from pydantic import BaseModel, ConfigDict, Field
```
**EN:** This block establishes the module context and imports `time`, `uuid`, `abc`, `dataclasses`, `typing`, and `pydantic`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `time`、`uuid`、`abc`、`dataclasses`、`typing` 和 `pydantic`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `ImageResponseData` class overview / `ImageResponseData` 类概览
```python
class ImageResponseData(BaseModel):
```
**EN:** This block defines class `ImageResponseData`. It encapsulates image response data behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `ImageResponseData`。 它用于封装 image response data 相关行为。 它继承自 `BaseModel`。

### Lines 12-15: supporting statements / 辅助语句
```python
    b64_json: Optional[str] = None
    url: Optional[str] = None
    revised_prompt: Optional[str] = None
    file_path: Optional[str] = None
```
**EN:** This block gathers supporting statements inside `ImageResponseData`. It updates names such as `b64_json`, `url`, `revised_prompt`, and `file_path`.
**CN:** 该代码块汇集了位于 `ImageResponseData` 内部的辅助语句。 它会更新 `b64_json`、`url`、`revised_prompt` 和 `file_path` 等名称。

### Lines 18-18: `ImageResponse` class overview / `ImageResponse` 类概览
```python
class ImageResponse(BaseModel):
```
**EN:** This block defines class `ImageResponse`. It encapsulates image response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `ImageResponse`。 它用于封装 image response 相关行为。 它继承自 `BaseModel`。

### Lines 19-23: supporting statements / 辅助语句
```python
    id: str
    created: int = Field(default_factory=lambda: int(time.time()))
    data: List[ImageResponseData]
    peak_memory_mb: Optional[float] = None
    inference_time_s: Optional[float] = None
```
**EN:** This block gathers supporting statements inside `ImageResponse`. It updates names such as `id`, `created`, `data`, `peak_memory_mb`, and `inference_time_s`. The code collaborates with `Field`, `int`, and `time.time`.
**CN:** 该代码块汇集了位于 `ImageResponse` 内部的辅助语句。 它会更新 `id`、`created`、`data`、`peak_memory_mb` 和 `inference_time_s` 等名称。 代码会与 `Field`、`int` 和 `time.time` 协同工作。

### Lines 26-26: `ImageGenerationsRequest` class overview / `ImageGenerationsRequest` 类概览
```python
class ImageGenerationsRequest(BaseModel):
```
**EN:** This block defines class `ImageGenerationsRequest`. It encapsulates image generations request behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `ImageGenerationsRequest`。 它用于封装 image generations request 相关行为。 它继承自 `BaseModel`。

### Lines 27-59: supporting statements / 辅助语句
```python
    model_config = ConfigDict(extra="allow")

    prompt: str
    model: Optional[str] = None
    n: Optional[int] = 1
    quality: Optional[str] = "auto"
    response_format: Optional[str] = "url"  # url | b64_json
    size: Optional[str] = "1024x1024"  # e.g., 1024x1024
    style: Optional[str] = "vivid"
    background: Optional[str] = "auto"  # transparent | opaque | auto
    output_format: Optional[str] = None  # png | jpeg | webp
    user: Optional[str] = None
    # SGLang extensions
    width: Optional[int] = None
    height: Optional[int] = None
    num_inference_steps: Optional[int] = None
    guidance_scale: Optional[float] = None
    true_cfg_scale: Optional[float] = (
        None  # for CFG vs guidance distillation (e.g., QwenImage)
    )
    seed: Optional[Union[int, List[int]]] = None
    generator_device: Optional[str] = "cuda"
    negative_prompt: Optional[str] = None
    output_quality: Optional[str] = "default"
    output_compression: Optional[int] = None
    enable_teacache: Optional[bool] = False
    # Upscaling
    enable_upscaling: Optional[bool] = False
    upscaling_model_path: Optional[str] = None
    upscaling_scale: Optional[int] = 4
    diffusers_kwargs: Optional[Dict[str, Any]] = None  # kwargs for diffusers backend
    # Performance profiling
    perf_dump_path: Optional[str] = None
```
**EN:** This block gathers supporting statements inside `ImageGenerationsRequest`. It updates names such as `model_config`, `prompt`, `model`, `n`, `quality`, and `response_format`. The code collaborates with `ConfigDict`.
**CN:** 该代码块汇集了位于 `ImageGenerationsRequest` 内部的辅助语句。 它会更新 `model_config`、`prompt`、`model`、`n`、`quality` 和 `response_format` 等名称。 代码会与 `ConfigDict` 协同工作。

### Lines 63-63: `VideoResponse` class overview / `VideoResponse` 类概览
```python
class VideoResponse(BaseModel):
```
**EN:** This block defines class `VideoResponse`. It encapsulates video response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `VideoResponse`。 它用于封装 video response 相关行为。 它继承自 `BaseModel`。

### Lines 64-82: supporting statements / 辅助语句
```python
    id: str
    object: str = "video"
    model: str = "sora-2"
    status: str = "queued"
    progress: int = 0
    created_at: int = Field(default_factory=lambda: int(time.time()))
    size: str = ""
    seconds: str = "4"
    quality: str = "standard"
    url: Optional[str] = None
    remixed_from_video_id: Optional[str] = None
    completed_at: Optional[int] = None
    expires_at: Optional[int] = None
    error: Optional[Dict[str, Any]] = None
    file_path: Optional[str] = None
    file_paths: Optional[List[str]] = None
    num_outputs: Optional[int] = None
    peak_memory_mb: Optional[float] = None
    inference_time_s: Optional[float] = None
```
**EN:** This block gathers supporting statements inside `VideoResponse`. It updates names such as `id`, `object`, `model`, `status`, `progress`, and `created_at`. The code collaborates with `Field`, `int`, and `time.time`.
**CN:** 该代码块汇集了位于 `VideoResponse` 内部的辅助语句。 它会更新 `id`、`object`、`model`、`status`、`progress` 和 `created_at` 等名称。 代码会与 `Field`、`int` 和 `time.time` 协同工作。

### Lines 85-85: `VideoGenerationsRequest` class overview / `VideoGenerationsRequest` 类概览
```python
class VideoGenerationsRequest(BaseModel):
```
**EN:** This block defines class `VideoGenerationsRequest`. It encapsulates video generations request behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `VideoGenerationsRequest`。 它用于封装 video generations request 相关行为。 它继承自 `BaseModel`。

### Lines 86-123: supporting statements / 辅助语句
```python
    prompt: str
    input_reference: Optional[str] = None
    reference_url: Optional[str] = None
    model: Optional[str] = None
    n: Optional[int] = 1
    num_outputs_per_prompt: Optional[int] = None
    seconds: Optional[int] = 4
    size: Optional[str] = ""
    fps: Optional[int] = None
    num_frames: Optional[int] = None
    seed: Optional[Union[int, List[int]]] = None
    generator_device: Optional[str] = "cuda"
    # SGLang extensions
    width: Optional[int] = None
    height: Optional[int] = None
    num_inference_steps: Optional[int] = None
    guidance_scale: Optional[float] = None
    guidance_scale_2: Optional[float] = None
    true_cfg_scale: Optional[float] = (
        None  # for CFG vs guidance distillation (e.g., QwenImage)
    )
    negative_prompt: Optional[str] = None
    enable_teacache: Optional[bool] = False
    # Frame interpolation
    enable_frame_interpolation: Optional[bool] = False
    frame_interpolation_exp: Optional[int] = 1  # 1=2×, 2=4×
    frame_interpolation_scale: Optional[float] = 1.0
    frame_interpolation_model_path: Optional[str] = None
    # Upscaling
    enable_upscaling: Optional[bool] = False
    upscaling_model_path: Optional[str] = None
    upscaling_scale: Optional[int] = 4
    output_quality: Optional[str] = "default"
    output_compression: Optional[int] = None
    output_path: Optional[str] = None
    diffusers_kwargs: Optional[Dict[str, Any]] = None  # kwargs for diffusers backend
    # Performance profiling
    perf_dump_path: Optional[str] = None
```
**EN:** This block gathers supporting statements inside `VideoGenerationsRequest`. It updates names such as `prompt`, `input_reference`, `reference_url`, `model`, `n`, and `num_outputs_per_prompt`.
**CN:** 该代码块汇集了位于 `VideoGenerationsRequest` 内部的辅助语句。 它会更新 `prompt`、`input_reference`、`reference_url`、`model`、`n` 和 `num_outputs_per_prompt` 等名称。

### Lines 126-126: `VideoListResponse` class overview / `VideoListResponse` 类概览
```python
class VideoListResponse(BaseModel):
```
**EN:** This block defines class `VideoListResponse`. It encapsulates video list response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `VideoListResponse`。 它用于封装 video list response 相关行为。 它继承自 `BaseModel`。

### Lines 127-128: supporting statements / 辅助语句
```python
    data: List[VideoResponse]
    object: str = "list"
```
**EN:** This block gathers supporting statements inside `VideoListResponse`. It updates names such as `data`, and `object`.
**CN:** 该代码块汇集了位于 `VideoListResponse` 内部的辅助语句。 它会更新 `data` 和 `object` 等名称。

### Lines 131-131: `VideoRemixRequest` class overview / `VideoRemixRequest` 类概览
```python
class VideoRemixRequest(BaseModel):
```
**EN:** This block defines class `VideoRemixRequest`. It encapsulates video remix request behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `VideoRemixRequest`。 它用于封装 video remix request 相关行为。 它继承自 `BaseModel`。

### Lines 132-132: supporting statements / 辅助语句
```python
    prompt: str
```
**EN:** This block gathers supporting statements inside `VideoRemixRequest`. It updates names such as `prompt`.
**CN:** 该代码块汇集了位于 `VideoRemixRequest` 内部的辅助语句。 它会更新 `prompt` 等名称。

### Lines 136-136: `MeshResponse` class overview / `MeshResponse` 类概览
```python
class MeshResponse(BaseModel):
```
**EN:** This block defines class `MeshResponse`. It encapsulates mesh response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `MeshResponse`。 它用于封装 mesh response 相关行为。 它继承自 `BaseModel`。

### Lines 137-151: supporting statements / 辅助语句
```python
    id: str
    object: str = "mesh"
    model: str = ""
    status: str = "queued"
    progress: int = 0
    created_at: int = Field(default_factory=lambda: int(time.time()))
    format: str = "glb"
    url: Optional[str] = None
    completed_at: Optional[int] = None
    expires_at: Optional[int] = None
    error: Optional[Dict[str, Any]] = None
    file_path: Optional[str] = None
    file_size_bytes: Optional[int] = None
    peak_memory_mb: Optional[float] = None
    inference_time_s: Optional[float] = None
```
**EN:** This block gathers supporting statements inside `MeshResponse`. It updates names such as `id`, `object`, `model`, `status`, `progress`, and `created_at`. The code collaborates with `Field`, `int`, and `time.time`.
**CN:** 该代码块汇集了位于 `MeshResponse` 内部的辅助语句。 它会更新 `id`、`object`、`model`、`status`、`progress` 和 `created_at` 等名称。 代码会与 `Field`、`int` 和 `time.time` 协同工作。

### Lines 154-154: `MeshGenerationsRequest` class overview / `MeshGenerationsRequest` 类概览
```python
class MeshGenerationsRequest(BaseModel):
```
**EN:** This block defines class `MeshGenerationsRequest`. It encapsulates mesh generations request behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `MeshGenerationsRequest`。 它用于封装 mesh generations request 相关行为。 它继承自 `BaseModel`。

### Lines 155-163: supporting statements / 辅助语句
```python
    prompt: str = "generate 3d mesh"
    input_image: Optional[str] = None
    model: Optional[str] = None
    seed: Optional[Union[int, List[int]]] = None
    generator_device: Optional[str] = "cuda"
    num_inference_steps: Optional[int] = None
    guidance_scale: Optional[float] = None
    negative_prompt: Optional[str] = None
    output_format: Optional[str] = "glb"
```
**EN:** This block gathers supporting statements inside `MeshGenerationsRequest`. It updates names such as `prompt`, `input_image`, `model`, `seed`, `generator_device`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `MeshGenerationsRequest` 内部的辅助语句。 它会更新 `prompt`、`input_image`、`model`、`seed`、`generator_device` 和 `num_inference_steps` 等名称。

### Lines 166-166: `MeshListResponse` class overview / `MeshListResponse` 类概览
```python
class MeshListResponse(BaseModel):
```
**EN:** This block defines class `MeshListResponse`. It encapsulates mesh list response behavior. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `MeshListResponse`。 它用于封装 mesh list response 相关行为。 它继承自 `BaseModel`。

### Lines 167-168: supporting statements / 辅助语句
```python
    data: List[MeshResponse]
    object: str = "list"
```
**EN:** This block gathers supporting statements inside `MeshListResponse`. It updates names such as `data`, and `object`.
**CN:** 该代码块汇集了位于 `MeshListResponse` 内部的辅助语句。 它会更新 `data` 和 `object` 等名称。

### Lines 172-172: `BaseReq` class overview / `BaseReq` 类概览
```python
class BaseReq(ABC):
```
**EN:** This block defines class `BaseReq`. It encapsulates base req behavior. It inherits from `ABC`.
**CN:** 该代码块定义了类 `BaseReq`。 它用于封装 base req 相关行为。 它继承自 `ABC`。

### Lines 173-174: supporting statements / 辅助语句
```python
    rid: Optional[Union[str, List[str]]] = field(default=None, kw_only=True)
    http_worker_ipc: Optional[str] = field(default=None, kw_only=True)
```
**EN:** This block gathers supporting statements inside `BaseReq`. It updates names such as `rid`, and `http_worker_ipc`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `BaseReq` 内部的辅助语句。 它会更新 `rid` 和 `http_worker_ipc` 等名称。 代码会与 `field` 协同工作。

### Lines 176-182: `regenerate_rid` implementation / `regenerate_rid` 实现
```python
    def regenerate_rid(self):
        """Generate a new request ID and return it."""
        if isinstance(self.rid, list):
            self.rid = [uuid.uuid4().hex for _ in range(len(self.rid))]
        else:
            self.rid = uuid.uuid4().hex
        return self.rid
```
**EN:** This block defines method `regenerate_rid` on `BaseReq`. Generate a new request ID and return it. Key calls include `isinstance`, `uuid.uuid4`, `range`, and `len`. The implementation branches on conditions.
**CN:** 该代码块定义了 `BaseReq` 的方法 `regenerate_rid`。 它用于处理 regenerate rid 相关逻辑。 关键调用包括 `isinstance`、`uuid.uuid4`、`range` 和 `len`。 实现中包含条件分支。

### Lines 186-186: `VertexGenerateReqInput` class overview / `VertexGenerateReqInput` 类概览
```python
class VertexGenerateReqInput(BaseReq):
```
**EN:** This block defines class `VertexGenerateReqInput`. It encapsulates vertex generate req input behavior. It inherits from `BaseReq`.
**CN:** 该代码块定义了类 `VertexGenerateReqInput`。 它用于封装 vertex generate req input 相关行为。 它继承自 `BaseReq`。

### Lines 187-188: supporting statements / 辅助语句
```python
    instances: List[dict]
    parameters: Optional[dict] = None
```
**EN:** This block gathers supporting statements inside `VertexGenerateReqInput`. It updates names such as `instances`, and `parameters`.
**CN:** 该代码块汇集了位于 `VertexGenerateReqInput` 内部的辅助语句。 它会更新 `instances` 和 `parameters` 等名称。

## Key Concepts / 关键概念
- `ImageResponseData`: Primary class that encapsulates image response data behavior. / 核心类，用于封装 image response data 相关行为。
- `ImageResponse`: Primary class that encapsulates image response behavior. / 核心类，用于封装 image response 相关行为。
- `ImageGenerationsRequest`: Primary class that encapsulates image generations request behavior. / 核心类，用于封装 image generations request 相关行为。
- `VideoResponse`: Primary class that encapsulates video response behavior. / 核心类，用于封装 video response 相关行为。
- `VideoGenerationsRequest`: Primary class that encapsulates video generations request behavior. / 核心类，用于封装 video generations request 相关行为。
- `VideoListResponse`: Primary class that encapsulates video list response behavior. / 核心类，用于封装 video list response 相关行为。
- `VideoRemixRequest`: Primary class that encapsulates video remix request behavior. / 核心类，用于封装 video remix request 相关行为。
- `MeshResponse`: Primary class that encapsulates mesh response behavior. / 核心类，用于封装 mesh response 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `uuid`, `abc`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `pydantic`

- **Total lines / 总行数**: 188
