# common_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/openai/common_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `ModelCard`, `DiffusionModelCard`, and `_handle_lora_request`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `ModelCard`、`DiffusionModelCard` 和 `_handle_lora_request` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module setup and imports / 模块初始化与导入
```python
import time
from typing import Any, List, Optional, Union

from fastapi import APIRouter, Body, HTTPException
from pydantic import BaseModel, Field

from sglang.multimodal_gen.registry import get_model_info
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    ListLorasReq,
    MergeLoraWeightsReq,
    SetLoraReq,
    UnmergeLoraWeightsReq,
    format_lora_message,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.srt.utils.json_response import orjson_response
```
**EN:** This block establishes the module context and imports `time`, `typing`, `fastapi`, `pydantic`, `sglang.multimodal_gen.registry`, and `sglang.multimodal_gen.runtime.entrypoints.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `time`、`typing`、`fastapi`、`pydantic`、`sglang.multimodal_gen.registry` 和 `sglang.multimodal_gen.runtime.entrypoints.utils`。这些依赖为后续实现提供所需符号。

### Lines 21-22: supporting statements / 辅助语句
```python
router = APIRouter(prefix="/v1")
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `router`, and `logger`. The code collaborates with `APIRouter`, and `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `router` 和 `logger` 等名称。 代码会与 `APIRouter` 和 `init_logger` 协同工作。

### Lines 25-27: `ModelCard` class overview / `ModelCard` 类概览
```python
class ModelCard(BaseModel):
    """Model cards."""
```
**EN:** This block defines class `ModelCard`. Model cards. It inherits from `BaseModel`.
**CN:** 该代码块定义了类 `ModelCard`。 它用于封装 model card 相关行为。 它继承自 `BaseModel`。

### Lines 28-34: supporting statements / 辅助语句
```python
    id: str
    object: str = "model"
    created: int = Field(default_factory=lambda: int(time.time()))
    owned_by: str = "sglang"
    root: Optional[str] = None
    parent: Optional[str] = None
    max_model_len: Optional[int] = None
```
**EN:** This block gathers supporting statements inside `ModelCard`. It updates names such as `id`, `object`, `created`, `owned_by`, `root`, and `parent`. The code collaborates with `Field`, `int`, and `time.time`.
**CN:** 该代码块汇集了位于 `ModelCard` 内部的辅助语句。 它会更新 `id`、`object`、`created`、`owned_by`、`root` 和 `parent` 等名称。 代码会与 `Field`、`int` 和 `time.time` 协同工作。

### Lines 37-39: `DiffusionModelCard` class overview / `DiffusionModelCard` 类概览
```python
class DiffusionModelCard(ModelCard):
    """Extended ModelCard with diffusion-specific fields."""
```
**EN:** This block defines class `DiffusionModelCard`. Extended ModelCard with diffusion-specific fields. It inherits from `ModelCard`.
**CN:** 该代码块定义了类 `DiffusionModelCard`。 它用于封装 diffusion model card 相关行为。 它继承自 `ModelCard`。

### Lines 40-45: supporting statements / 辅助语句
```python
    num_gpus: Optional[int] = None
    task_type: Optional[str] = None
    dit_precision: Optional[str] = None
    vae_precision: Optional[str] = None
    pipeline_name: Optional[str] = None
    pipeline_class: Optional[str] = None
```
**EN:** This block gathers supporting statements inside `DiffusionModelCard`. It updates names such as `num_gpus`, `task_type`, `dit_precision`, `vae_precision`, `pipeline_name`, and `pipeline_class`.
**CN:** 该代码块汇集了位于 `DiffusionModelCard` 内部的辅助语句。 它会更新 `num_gpus`、`task_type`、`dit_precision`、`vae_precision`、`pipeline_name` 和 `pipeline_class` 等名称。

### Lines 48-60: `_handle_lora_request` implementation / `_handle_lora_request` 实现
```python
async def _handle_lora_request(req: Any, success_msg: str, failure_msg: str):
    try:
        output: OutputBatch = await async_scheduler_client.forward(req)
        if output.error is None:
            return {"status": "ok", "message": success_msg}
        else:
            error_msg = output.error
            raise HTTPException(status_code=500, detail=f"{failure_msg}: {error_msg}")
    except Exception as e:
        if isinstance(e, HTTPException):
            raise
        logger.error(f"Error during '{failure_msg}': {e}", exc_info=True)
        raise HTTPException(status_code=500, detail=str(e))
```
**EN:** This block defines function `_handle_lora_request`. It handles handle lora request logic. Key calls include `async_scheduler_client.forward`, `HTTPException`, `isinstance`, `logger.error`, and `str`. The implementation branches on conditions, handles exceptional paths. Parameters such as `req`, `success_msg`, and `failure_msg` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_handle_lora_request`。 它用于处理 handle lora request 相关逻辑。 关键调用包括 `async_scheduler_client.forward`、`HTTPException`、`isinstance`、`logger.error` 和 `str`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `req`、`success_msg` 和 `failure_msg` 等参数驱动。

### Lines 63-105: `set_lora` implementation / `set_lora` 实现
```python
@router.post("/set_lora")
async def set_lora(
    lora_nickname: Union[str, List[str]] = Body(..., embed=True),
    lora_path: Optional[Union[str, List[Optional[str]]]] = Body(None, embed=True),
    target: Union[str, List[str]] = Body("all", embed=True),
    strength: Union[float, List[float]] = Body(1.0, embed=True),
    merge_mode: Optional[str] = Body(None, embed=True),
):
    """
    Set LoRA adapter(s) for the specified transformer(s).
    Supports both single LoRA (backward compatible) and multiple LoRA adapters.

    Args:
        lora_nickname: The nickname(s) of the adapter(s). Can be a string or a list of strings.
        lora_path: Path(s) to the LoRA adapter(s) (local path or HF repo id).
            Can be a string, None, or a list of strings/None. Must match the length of lora_nickname.
        target: Which transformer(s) to apply the LoRA to. Can be a string or a list of strings.
            If a list, must match the length of lora_nickname. Valid values:
            - "all": Apply to all transformers (default)
            - "transformer": Apply only to the primary transformer (high noise for Wan2.2)
            - "transformer_2": Apply only to transformer_2 (low noise for Wan2.2)
            - "critic": Apply only to the critic model
        strength: LoRA strength(s) for merge, default 1.0. Can be a float or a list of floats.
            If a list, must match the length of lora_nickname. Values < 1.0 reduce the effect,
            values > 1.0 amplify the effect.
        merge_mode: Optional LoRA merge mode: "auto", "merge", or "dynamic".
    """
    req = SetLoraReq(
        lora_nickname=lora_nickname,
        lora_path=lora_path,
        target=target,
        strength=strength,
        merge_mode=merge_mode,
    )
    nickname_str, target_str, strength_str = format_lora_message(
        lora_nickname, target, strength
    )

    return await _handle_lora_request(
        req,
        f"Successfully set LoRA adapter(s): {nickname_str} (target: {target_str}, strength: {strength_str})",
        "Failed to set LoRA adapter",
    )
```
**EN:** This block defines function `set_lora`. Set LoRA adapter(s) for the specified transformer(s). Supports both single LoRA (backward compatible) and multiple LoRA adapters. Key calls include `router.post`, `Body`, `SetLoraReq`, `format_lora_message`, and `_handle_lora_request`. Parameters such as `lora_nickname`, `lora_path`, `target`, `strength`, and `merge_mode` drive the behavior in this section.
**CN:** 该代码块定义了函数 `set_lora`。 它用于设置lora。 关键调用包括 `router.post`、`Body`、`SetLoraReq`、`format_lora_message` 和 `_handle_lora_request`。 本段逻辑主要由 `lora_nickname`、`lora_path`、`target`、`strength` 和 `merge_mode` 等参数驱动。

### Lines 108-127: `merge_lora_weights` implementation / `merge_lora_weights` 实现
```python
@router.post("/merge_lora_weights")
async def merge_lora_weights(
    target: str = Body("all", embed=True),
    strength: float = Body(1.0, embed=True),
):
    """
    Merge LoRA weights into the base model.

    Args:
        target: Which transformer(s) to merge. One of "all", "transformer",
                "transformer_2", "critic".
        strength: LoRA strength for merge, default 1.0. Values < 1.0 reduce the effect,
            values > 1.0 amplify the effect.
    """
    req = MergeLoraWeightsReq(target=target, strength=strength)
    return await _handle_lora_request(
        req,
        f"Successfully merged LoRA weights (target: {target}, strength: {strength})",
        "Failed to merge LoRA weights",
    )
```
**EN:** This block defines function `merge_lora_weights`. Merge LoRA weights into the base model. Args: target: Which transformer(s) to merge. Key calls include `router.post`, `Body`, `MergeLoraWeightsReq`, and `_handle_lora_request`. Parameters such as `target`, and `strength` drive the behavior in this section.
**CN:** 该代码块定义了函数 `merge_lora_weights`。 它用于合并lora weights。 关键调用包括 `router.post`、`Body`、`MergeLoraWeightsReq` 和 `_handle_lora_request`。 本段逻辑主要由 `target` 和 `strength` 等参数驱动。

### Lines 130-146: `unmerge_lora_weights` implementation / `unmerge_lora_weights` 实现
```python
@router.post("/unmerge_lora_weights")
async def unmerge_lora_weights(
    target: str = Body("all", embed=True),
):
    """
    Unmerge LoRA weights from the base model.

    Args:
        target: Which transformer(s) to unmerge. One of "all", "transformer",
                "transformer_2", "critic".
    """
    req = UnmergeLoraWeightsReq(target=target)
    return await _handle_lora_request(
        req,
        f"Successfully unmerged LoRA weights (target: {target})",
        "Failed to unmerge LoRA weights",
    )
```
**EN:** This block defines function `unmerge_lora_weights`. Unmerge LoRA weights from the base model. Args: target: Which transformer(s) to unmerge. Key calls include `router.post`, `Body`, `UnmergeLoraWeightsReq`, and `_handle_lora_request`. Parameters such as `target` drive the behavior in this section.
**CN:** 该代码块定义了函数 `unmerge_lora_weights`。 它用于处理 unmerge lora weights 相关逻辑。 关键调用包括 `router.post`、`Body`、`UnmergeLoraWeightsReq` 和 `_handle_lora_request`。 本段逻辑主要由 `target` 等参数驱动。

### Lines 149-159: `model_info` implementation / `model_info` 实现
```python
@router.get("/model_info")
async def model_info():
    """Get the model information."""
    server_args = get_global_server_args()
    if not server_args:
        raise HTTPException(status_code=500, detail="Server args not initialized")

    result = {
        "model_path": server_args.model_path,
    }
    return result
```
**EN:** This block defines function `model_info`. Get the model information. Key calls include `router.get`, `get_global_server_args`, and `HTTPException`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `model_info`。 它用于处理 model info 相关逻辑。 关键调用包括 `router.get`、`get_global_server_args` 和 `HTTPException`。 实现中包含条件分支。

### Lines 162-176: `list_loras` implementation / `list_loras` 实现
```python
@router.get("/list_loras")
async def list_loras():
    """List loaded LoRA adapters and current application status per module."""
    try:
        req = ListLorasReq()
        output: OutputBatch = await async_scheduler_client.forward(req)
        if output.error is None:
            return output.output or {}
        else:
            raise HTTPException(status_code=500, detail=output.error)
    except Exception as e:
        if isinstance(e, HTTPException):
            raise
        logger.error(f"Error during 'list_loras': {e}", exc_info=True)
        raise HTTPException(status_code=500, detail=str(e))
```
**EN:** This block defines function `list_loras`. List loaded LoRA adapters and current application status per module. Key calls include `router.get`, `ListLorasReq`, `async_scheduler_client.forward`, `HTTPException`, and `isinstance`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `list_loras`。 它用于处理 list loras 相关逻辑。 关键调用包括 `router.get`、`ListLorasReq`、`async_scheduler_client.forward`、`HTTPException` 和 `isinstance`。 实现中包含条件分支，处理异常路径。

### Lines 179-209: `available_models` implementation / `available_models` 实现
```python
@router.get("/models")
async def available_models():
    """Show available models. OpenAI-compatible endpoint with extended diffusion info."""
    server_args = get_global_server_args()
    if not server_args:
        raise HTTPException(status_code=500, detail="Server args not initialized")

    model_info = get_model_info(
        server_args.model_path,
        backend=server_args.backend,
        model_id=server_args.model_id,
    )

    card_kwargs = {
        "id": server_args.model_path,
        "root": server_args.model_path,
        # Extended diffusion-specific fields
        "num_gpus": server_args.num_gpus,
        "task_type": server_args.pipeline_config.task_type.name,
        "dit_precision": server_args.pipeline_config.dit_precision,
        "vae_precision": server_args.pipeline_config.vae_precision,
    }

    if model_info:
        card_kwargs["pipeline_name"] = model_info.pipeline_cls.pipeline_name
        card_kwargs["pipeline_class"] = model_info.pipeline_cls.__name__

    model_card = DiffusionModelCard(**card_kwargs)

    # Return dict directly to preserve extended fields (ModelList strips them)
    return {"object": "list", "data": [model_card.model_dump()]}
```
**EN:** This block defines function `available_models`. Show available models. OpenAI-compatible endpoint with extended diffusion info. Key calls include `router.get`, `get_global_server_args`, `get_model_info`, `DiffusionModelCard`, and `HTTPException`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `available_models`。 它用于处理 available models 相关逻辑。 关键调用包括 `router.get`、`get_global_server_args`、`get_model_info`、`DiffusionModelCard` 和 `HTTPException`。 实现中包含条件分支。

### Lines 212-252: `retrieve_model` implementation / `retrieve_model` 实现
```python
@router.get("/models/{model:path}")
async def retrieve_model(model: str):
    """Retrieve a model instance. OpenAI-compatible endpoint with extended diffusion info."""
    server_args = get_global_server_args()
    if not server_args:
        raise HTTPException(status_code=500, detail="Server args not initialized")

    if model != server_args.model_path:
        return orjson_response(
            {
                "error": {
                    "message": f"The model '{model}' does not exist",
                    "type": "invalid_request_error",
                    "param": "model",
                    "code": "model_not_found",
                }
            },
            status_code=404,
        )

    model_info = get_model_info(
        server_args.model_path,
        backend=server_args.backend,
        model_id=server_args.model_id,
    )

    card_kwargs = {
        "id": model,
        "root": model,
        "num_gpus": server_args.num_gpus,
        "task_type": server_args.pipeline_config.task_type.name,
        "dit_precision": server_args.pipeline_config.dit_precision,
        "vae_precision": server_args.pipeline_config.vae_precision,
    }

    if model_info:
        card_kwargs["pipeline_name"] = model_info.pipeline_cls.pipeline_name
        card_kwargs["pipeline_class"] = model_info.pipeline_cls.__name__

    # Return dict to preserve extended fields
    return DiffusionModelCard(**card_kwargs).model_dump()
```
**EN:** This block defines function `retrieve_model`. Retrieve a model instance. OpenAI-compatible endpoint with extended diffusion info. Key calls include `router.get`, `get_global_server_args`, `get_model_info`, `DiffusionModelCard.model_dump`, and `HTTPException`. The implementation branches on conditions. Parameters such as `model` drive the behavior in this section.
**CN:** 该代码块定义了函数 `retrieve_model`。 它用于处理 retrieve model 相关逻辑。 关键调用包括 `router.get`、`get_global_server_args`、`get_model_info`、`DiffusionModelCard.model_dump` 和 `HTTPException`。 实现中包含条件分支。 本段逻辑主要由 `model` 等参数驱动。

## Key Concepts / 关键概念
- `ModelCard`: Model cards. / 核心类，用于封装 model card 相关行为。
- `DiffusionModelCard`: Extended ModelCard with diffusion-specific fields. / 核心类，用于封装 diffusion model card 相关行为。
- `_handle_lora_request`: Top-level function that handles handle lora request logic. / 顶层函数，用于处理 handle lora request 相关逻辑。
- `set_lora`: Set LoRA adapter(s) for the specified transformer(s). / 顶层函数，用于设置lora。
- `merge_lora_weights`: Merge LoRA weights into the base model. / 顶层函数，用于合并lora weights。
- `unmerge_lora_weights`: Unmerge LoRA weights from the base model. / 顶层函数，用于处理 unmerge lora weights 相关逻辑。
- `model_info`: Get the model information. / 顶层函数，用于处理 model info 相关逻辑。
- `list_loras`: List loaded LoRA adapters and current application status per module. / 顶层函数，用于处理 list loras 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `typing`
- **Third-party / 第三方依赖**: `fastapi`, `pydantic`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.registry`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils.json_response`

- **Total lines / 总行数**: 252
