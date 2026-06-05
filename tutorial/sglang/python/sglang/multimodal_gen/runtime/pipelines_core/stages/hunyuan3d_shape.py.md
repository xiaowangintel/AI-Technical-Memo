# hunyuan3d_shape.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/hunyuan3d_shape.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for hunyuan3d shape, connecting stages, tensors, and runtime metadata. Key symbols include `retrieve_timesteps`, `_prepare_shape_image`, `_move_to_device`. / 该模块负责 hunyuan3d shape 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `retrieve_timesteps`, `_prepare_shape_image`, `_move_to_device`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
Hunyuan3D shape generation stages.

Four-stage pipeline: BeforeDenoising -> Denoising -> Export -> Save.
"""

from __future__ import annotations

import os
from typing import Any

import numpy as np
import torch
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.runtime.utils.mesh3d_utils import export_to_trimesh

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 42-84: Function `retrieve_timesteps` / 函数 `retrieve_timesteps`
```python
def retrieve_timesteps(
    scheduler,
    num_inference_steps=None,
    device=None,
    timesteps=None,
    sigmas=None,
    **kwargs,
):
    """Retrieve timesteps from scheduler."""
    import inspect

    if timesteps is not None and sigmas is not None:
        raise ValueError("Only one of timesteps or sigmas can be passed.")

# ...
        scheduler.set_timesteps(num_inference_steps, device=device, **kwargs)
        timesteps = scheduler.timesteps

    return timesteps, num_inference_steps
```
**EN:** This function drives `retrieve_timesteps` with inputs such as `scheduler`, `num_inference_steps`, `device`, `timesteps`. Retrieve timesteps from scheduler.
**CN:** 这个函数负责 `retrieve_timesteps`，主要处理 `scheduler`, `num_inference_steps`, `device`, `timesteps` 等输入。 文档字符串说明：Retrieve timesteps from scheduler.

### Lines 87-106: Function `_prepare_shape_image` / 函数 `_prepare_shape_image`
```python
def _prepare_shape_image(image_processor, image, mask=None) -> dict:
    """Prepare shape image for conditioning."""
    if isinstance(image, torch.Tensor) and isinstance(mask, torch.Tensor):
        return {"image": image, "mask": mask}

    if isinstance(image, str) and not os.path.exists(image):
        raise FileNotFoundError(f"Couldn't find image at path {image}")

    if not isinstance(image, list):
        image = [image]

    outputs = [image_processor(img) for img in image]
    cond_input = {k: [] for k in outputs[0].keys()}
    for output in outputs:
# ...
    for key, value in cond_input.items():
        if isinstance(value[0], torch.Tensor):
            cond_input[key] = torch.cat(value, dim=0)
    return cond_input
```
**EN:** This function drives `_prepare_shape_image` with inputs such as `image_processor`, `image`, `mask`. Prepare shape image for conditioning.
**CN:** 这个函数负责 `_prepare_shape_image`，主要处理 `image_processor`, `image`, `mask` 等输入。 文档字符串说明：Prepare shape image for conditioning.

### Lines 109-117: Function `_move_to_device` / 函数 `_move_to_device`
```python
def _move_to_device(payload, device, dtype):
    """Recursively move tensors in payload to specified device and dtype."""
    if isinstance(payload, torch.Tensor):
        return payload.to(device=device, dtype=dtype)
    if isinstance(payload, dict):
        return {k: _move_to_device(v, device, dtype) for k, v in payload.items()}
    if isinstance(payload, list):
        return [_move_to_device(v, device, dtype) for v in payload]
    return payload
```
**EN:** This function drives `_move_to_device` with inputs such as `payload`, `device`, `dtype`. Recursively move tensors in payload to specified device and dtype.
**CN:** 这个函数负责 `_move_to_device`，主要处理 `payload`, `device`, `dtype` 等输入。 文档字符串说明：Recursively move tensors in payload to specified device and dtype.

### Lines 120-247: Class `Hunyuan3DShapeBeforeDenoisingStage` / 类 `Hunyuan3DShapeBeforeDenoisingStage`
```python
class Hunyuan3DShapeBeforeDenoisingStage(PipelineStage):
    """Monolithic pre-processing stage for Hunyuan3D shape generation.

    Consolidates input validation, image preprocessing, conditioning, and
    latent/timestep preparation into a single stage.
    """

    def __init__(
        self,
        image_processor: Any,
        conditioner: Any,
        vae: Any,
        model: Any,
        scheduler: Any,
# ...
        result.add_check("timesteps", batch.timesteps, [V.is_tensor, V.min_dims(1)])
        result.add_check("latents", batch.latents, V.is_tensor)
        result.add_check("prompt_embeds", batch.prompt_embeds, V.list_not_empty)
        return result
```
**EN:** This class models `Hunyuan3DShapeBeforeDenoisingStage` as a specialization of `PipelineStage`. Monolithic pre-processing stage for Hunyuan3D shape generation. Important methods include `__init__`, `_validate_input`, `_prepare_latents`, `forward`.
**CN:** 该类实现 `Hunyuan3DShapeBeforeDenoisingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Monolithic pre-processing stage for Hunyuan3D shape generation. 其中较重要的方法包括 `__init__`, `_validate_input`, `_prepare_latents`, `forward`。

### Lines 250-408: Class `Hunyuan3DShapeDenoisingStage` / 类 `Hunyuan3DShapeDenoisingStage`
```python
class Hunyuan3DShapeDenoisingStage(DenoisingStage):
    """Denoising stage for Hunyuan3D shape generation."""

    def __init__(self, transformer: Any, scheduler: Any, **kwargs) -> None:
        super().__init__(transformer=transformer, scheduler=scheduler, **kwargs)

    def _prepare_denoising_loop(self, batch: Req, server_args: ServerArgs):
        """Prepare Hunyuan3D-specific variables for the base denoising loop."""
        assert self.transformer is not None
        pipeline = self.pipeline() if self.pipeline else None
        scheduler = batch.scheduler
        assert scheduler is not None
        cache_dit_num_inference_steps = batch.extra.get(
            "cache_dit_num_inference_steps", batch.num_inference_steps
# ...
    def verify_output(self, batch: Req, server_args: ServerArgs) -> VerificationResult:
        result = VerificationResult()
        result.add_check("latents", batch.latents, V.is_tensor)
        return result
```
**EN:** This class models `Hunyuan3DShapeDenoisingStage` as a specialization of `DenoisingStage`. Denoising stage for Hunyuan3D shape generation. Important methods include `__init__`, `_prepare_denoising_loop`, `_predict_noise`, `_predict_noise_with_cfg`.
**CN:** 该类实现 `Hunyuan3DShapeDenoisingStage`，并继承/扩展 `DenoisingStage`。 文档字符串指出：Denoising stage for Hunyuan3D shape generation. 其中较重要的方法包括 `__init__`, `_prepare_denoising_loop`, `_predict_noise`, `_predict_noise_with_cfg`。

### Lines 411-466: Class `Hunyuan3DShapeExportStage` / 类 `Hunyuan3DShapeExportStage`
```python
class Hunyuan3DShapeExportStage(PipelineStage):
    """VAE decoding and mesh extraction stage."""

    def __init__(self, vae: Any, config: Hunyuan3D2PipelineConfig) -> None:
        super().__init__()
        self.vae = vae
        self.config = config

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        if self.config.shape_mc_algo is not None:
            try:
                from sglang.multimodal_gen.runtime.models.vaes.hunyuan3d_vae import (
                    SurfaceExtractors,
                )
# ...
    def verify_output(self, batch: Req, server_args: ServerArgs) -> VerificationResult:
        result = VerificationResult()
        result.add_check("shape_meshes", batch.extra.get("shape_meshes"), V.not_none)
        return result
```
**EN:** This class models `Hunyuan3DShapeExportStage` as a specialization of `PipelineStage`. VAE decoding and mesh extraction stage. Important methods include `__init__`, `forward`, `verify_input`, `verify_output`.
**CN:** 该类实现 `Hunyuan3DShapeExportStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：VAE decoding and mesh extraction stage. 其中较重要的方法包括 `__init__`, `forward`, `verify_input`, `verify_output`。

### Lines 469-528: Class `Hunyuan3DShapeSaveStage` / 类 `Hunyuan3DShapeSaveStage`
```python
class Hunyuan3DShapeSaveStage(PipelineStage):
    """Mesh file export and output decision stage."""

    def __init__(self, config: Hunyuan3D2PipelineConfig) -> None:
        super().__init__()
        self.config = config

    def _get_output_paths(self, batch: Req) -> tuple[str, str]:
        output_path = batch.output_file_path() or os.path.join(
            batch.output_path, "output.obj"
        )
        if output_path.endswith(".glb"):
            obj_path = output_path[:-4] + ".obj"
            return obj_path, output_path
# ...
    def verify_input(self, batch: Req, server_args: ServerArgs) -> VerificationResult:
        result = VerificationResult()
        result.add_check("shape_meshes", batch.extra.get("shape_meshes"), V.not_none)
        return result
```
**EN:** This class models `Hunyuan3DShapeSaveStage` as a specialization of `PipelineStage`. Mesh file export and output decision stage. Important methods include `__init__`, `_get_output_paths`, `forward`, `verify_input`.
**CN:** 该类实现 `Hunyuan3DShapeSaveStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Mesh file export and output decision stage. 其中较重要的方法包括 `__init__`, `_get_output_paths`, `forward`, `verify_input`。

### Lines 529-537: Registration and exports / 注册与导出
```python


__all__ = [
    "retrieve_timesteps",
    "Hunyuan3DShapeBeforeDenoisingStage",
    "Hunyuan3DShapeDenoisingStage",
    "Hunyuan3DShapeExportStage",
    "Hunyuan3DShapeSaveStage",
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.hunyuan3d`, `sglang.multimodal_gen.runtime.loader.component_loaders.transformer_loader`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.denoising`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `__future__`, `numpy`, `torch`, `diffusers.utils.torch_utils`
- **Stdlib / 标准库**: `os`, `typing`, `inspect`
