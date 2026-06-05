# weights_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/post_training/weights_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `update_weights_from_disk`, and `get_weights_checksum`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Weight update API for the diffusion engine. / 该文件属于服务入口层。它围绕 `update_weights_from_disk` 和 `get_weights_checksum` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module setup and imports / 模块初始化与导入
```python
"""Weight update API for the diffusion engine."""

from fastapi import APIRouter, Request

from sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct import (
    GetWeightsChecksumReqInput,
    UpdateWeightFromDiskReqInput,
)
from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client
from sglang.srt.utils.json_response import orjson_response
```
**EN:** This block establishes the module context and imports `fastapi`, `sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct`, `sglang.multimodal_gen.runtime.scheduler_client`, and `sglang.srt.utils.json_response`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `fastapi`、`sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct`、`sglang.multimodal_gen.runtime.scheduler_client` 和 `sglang.srt.utils.json_response`。这些依赖为后续实现提供所需符号。

### Lines 12-12: supporting statements / 辅助语句
```python
router = APIRouter()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `router`. The code collaborates with `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `router` 等名称。 代码会与 `APIRouter` 协同工作。

### Lines 15-46: `update_weights_from_disk` implementation / `update_weights_from_disk` 实现
```python
@router.post("/update_weights_from_disk")
async def update_weights_from_disk(request: Request):
    """Update model weights from disk inplace without restarting the server."""
    body = await request.json()
    model_path = body.get("model_path")
    if not model_path:
        return orjson_response(
            {"success": False, "message": "model_path is required"},
            status_code=400,
        )

    req = UpdateWeightFromDiskReqInput(
        model_path=model_path,
        flush_cache=body.get("flush_cache", True),
        target_modules=body.get("target_modules"),
    )

    try:
        response = await async_scheduler_client.forward(req)
    except Exception as e:
        return orjson_response(
            {"success": False, "message": str(e)},
            status_code=500,
        )

    result = response.output
    success = result.get("success", False)
    message = result.get("message", "Unknown status")
    return orjson_response(
        {"success": success, "message": message},
        status_code=200 if success else 400,
    )
```
**EN:** This block defines function `update_weights_from_disk`. Update model weights from disk inplace without restarting the server. Key calls include `router.post`, `body.get`, `UpdateWeightFromDiskReqInput`, `result.get`, and `orjson_response`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `update_weights_from_disk`。 它用于更新weights from disk。 关键调用包括 `router.post`、`body.get`、`UpdateWeightFromDiskReqInput`、`result.get` 和 `orjson_response`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request` 等参数驱动。

### Lines 49-62: `get_weights_checksum` implementation / `get_weights_checksum` 实现
```python
@router.post("/get_weights_checksum")
async def get_weights_checksum(request: Request):
    """Return SHA-256 checksum of each requested module's weights."""
    body = await request.json()
    req = GetWeightsChecksumReqInput(
        module_names=body.get("module_names"),
    )

    try:
        response = await async_scheduler_client.forward(req)
    except Exception as e:
        return orjson_response({"error": str(e)}, status_code=500)

    return orjson_response(response.output, status_code=200)
```
**EN:** This block defines function `get_weights_checksum`. Return SHA-256 checksum of each requested module's weights. Key calls include `router.post`, `GetWeightsChecksumReqInput`, `orjson_response`, `request.json`, and `body.get`. The implementation handles exceptional paths. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_weights_checksum`。 它用于获取weights checksum。 关键调用包括 `router.post`、`GetWeightsChecksumReqInput`、`orjson_response`、`request.json` 和 `body.get`。 实现中处理异常路径。 本段逻辑主要由 `request` 等参数驱动。

## Key Concepts / 关键概念
- `update_weights_from_disk`: Update model weights from disk inplace without restarting the server. / 顶层函数，用于更新weights from disk。
- `get_weights_checksum`: Return SHA-256 checksum of each requested module's weights. / 顶层函数，用于获取weights checksum。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `fastapi`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.srt.utils.json_response`

- **Total lines / 总行数**: 62
