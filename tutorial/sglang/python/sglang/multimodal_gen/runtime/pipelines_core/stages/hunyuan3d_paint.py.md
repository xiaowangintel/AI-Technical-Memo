# hunyuan3d_paint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/hunyuan3d_paint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for hunyuan3d paint, connecting stages, tensors, and runtime metadata. Key symbols include `guidance_scale_embedding`, `extract_into_tensor`, `get_predicted_original_sample`. / 该模块负责 hunyuan3d paint 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `guidance_scale_embedding`, `extract_into_tensor`, `get_predicted_original_sample`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Imports and module setup / 导入与模块初始化
```python
"""
Hunyuan3D paint/texture generation stages.

Three-stage pipeline: Preprocess -> TexGen -> Postprocess.
"""

from __future__ import annotations

import os
from typing import Any

import numpy as np
import torch
from diffusers.pipelines.stable_diffusion.pipeline_stable_diffusion import (
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 41-56: Function `guidance_scale_embedding` / 函数 `guidance_scale_embedding`
```python
def guidance_scale_embedding(
    w: torch.Tensor, embedding_dim: int = 512, dtype: torch.dtype = torch.float32
) -> torch.Tensor:
    """Generate guidance scale embeddings."""
    assert len(w.shape) == 1
    w = w * 1000.0

    half_dim = embedding_dim // 2
    emb = torch.log(torch.tensor(10000.0)) / (half_dim - 1)
    emb = torch.exp(torch.arange(half_dim, dtype=dtype) * -emb)
    emb = w.to(dtype)[:, None] * emb[None, :]
    emb = torch.cat([torch.sin(emb), torch.cos(emb)], dim=1)
    if embedding_dim % 2 == 1:
        emb = torch.nn.functional.pad(emb, (0, 1))
    assert emb.shape == (w.shape[0], embedding_dim)
    return emb
```
**EN:** This function drives `guidance_scale_embedding` with inputs such as `w`, `embedding_dim`, `dtype`. Generate guidance scale embeddings.
**CN:** 这个函数负责 `guidance_scale_embedding`，主要处理 `w`, `embedding_dim`, `dtype` 等输入。 文档字符串说明：Generate guidance scale embeddings.

### Lines 59-67: Function `extract_into_tensor` / 函数 `extract_into_tensor`
```python
def extract_into_tensor(
    a: torch.Tensor, t: torch.Tensor, x_shape: tuple, n_gen: int
) -> torch.Tensor:
    """Extract values from tensor and reshape for multi-view generation."""
    out = a.gather(-1, t)
    out = out.repeat(n_gen)
    out = rearrange(out, "(b n) -> b n", n=n_gen)
    b, c, *_ = out.shape
    return out.reshape(b, c, *((1,) * (len(x_shape) - 2)))
```
**EN:** This function drives `extract_into_tensor` with inputs such as `a`, `t`, `x_shape`, `n_gen`. Extract values from tensor and reshape for multi-view generation.
**CN:** 这个函数负责 `extract_into_tensor`，主要处理 `a`, `t`, `x_shape`, `n_gen` 等输入。 文档字符串说明：Extract values from tensor and reshape for multi-view generation.

### Lines 70-96: Function `get_predicted_original_sample` / 函数 `get_predicted_original_sample`
```python
def get_predicted_original_sample(
    model_output: torch.Tensor,
    timesteps: torch.Tensor,
    sample: torch.Tensor,
    prediction_type: str,
    alphas: torch.Tensor,
    sigmas: torch.Tensor,
    n_gen: int,
) -> torch.Tensor:
    """Get predicted original sample from model output."""
    alphas = extract_into_tensor(alphas, timesteps, sample.shape, n_gen)
    sigmas = extract_into_tensor(sigmas, timesteps, sample.shape, n_gen)
    model_output = rearrange(model_output, "(b n) c h w -> b n c h w", n=n_gen)

# ...
            "currently, `epsilon`, `sample`, and `v_prediction` are supported."
        )

    return pred_x_0
```
**EN:** This function drives `get_predicted_original_sample` with inputs such as `model_output`, `timesteps`, `sample`, `prediction_type`. Get predicted original sample from model output.
**CN:** 这个函数负责 `get_predicted_original_sample`，主要处理 `model_output`, `timesteps`, `sample`, `prediction_type` 等输入。 文档字符串说明：Get predicted original sample from model output.

### Lines 99-125: Function `get_predicted_noise` / 函数 `get_predicted_noise`
```python
def get_predicted_noise(
    model_output: torch.Tensor,
    timesteps: torch.Tensor,
    sample: torch.Tensor,
    prediction_type: str,
    alphas: torch.Tensor,
    sigmas: torch.Tensor,
    n_gen: int,
) -> torch.Tensor:
    """Get predicted noise from model output."""
    alphas = extract_into_tensor(alphas, timesteps, sample.shape, n_gen)
    sigmas = extract_into_tensor(sigmas, timesteps, sample.shape, n_gen)
    model_output = rearrange(model_output, "(b n) c h w -> b n c h w", n=n_gen)

# ...
            "currently, `epsilon`, `sample`, and `v_prediction` are supported."
        )

    return pred_epsilon
```
**EN:** This function drives `get_predicted_noise` with inputs such as `model_output`, `timesteps`, `sample`, `prediction_type`. Get predicted noise from model output.
**CN:** 这个函数负责 `get_predicted_noise`，主要处理 `model_output`, `timesteps`, `sample`, `prediction_type` 等输入。 文档字符串说明：Get predicted noise from model output.

### Lines 128-142: Function `to_rgb_image` / 函数 `to_rgb_image`
```python
def to_rgb_image(maybe_rgba):
    """Convert RGBA image to RGB."""
    from PIL import Image

    if maybe_rgba.mode == "RGB":
        return maybe_rgba
    if maybe_rgba.mode == "RGBA":
        rgba = maybe_rgba
        img = np.random.randint(
            127, 128, size=[rgba.size[1], rgba.size[0], 3], dtype=np.uint8
        )
        img = Image.fromarray(img, "RGB")
        img.paste(rgba, mask=rgba.getchannel("A"))
        return img
    raise ValueError(f"Unsupported image type: {maybe_rgba.mode}")
```
**EN:** This function drives `to_rgb_image` with inputs such as `maybe_rgba`. Convert RGBA image to RGB.
**CN:** 这个函数负责 `to_rgb_image`，主要处理 `maybe_rgba` 等输入。 文档字符串说明：Convert RGBA image to RGB.

### Lines 145-184: Class `DDIMSolver` / 类 `DDIMSolver`
```python
class DDIMSolver:
    """DDIM solver for fast sampling."""

    def __init__(
        self,
        alpha_cumprods: np.ndarray,
        timesteps: int = 1000,
        ddim_timesteps: int = 50,
    ):
        step_ratio = timesteps // ddim_timesteps
        self.ddim_timesteps = (
            np.arange(1, ddim_timesteps + 1) * step_ratio
        ).round().astype(np.int64) - 1
        self.ddim_alpha_cumprods = alpha_cumprods[self.ddim_timesteps]
# ...
        )
        dir_xt = (1.0 - alpha_cumprod_prev).sqrt() * pred_noise
        x_prev = alpha_cumprod_prev.sqrt() * pred_x0 + dir_xt
        return x_prev
```
**EN:** This class models `DDIMSolver`. DDIM solver for fast sampling. Important methods include `__init__`, `to`, `ddim_step`.
**CN:** 该类实现 `DDIMSolver`。 文档字符串指出：DDIM solver for fast sampling. 其中较重要的方法包括 `__init__`, `to`, `ddim_step`。

### Lines 187-225: Function `_recorrect_rgb` / 函数 `_recorrect_rgb`
```python
def _recorrect_rgb(
    src_image: torch.Tensor,
    target_image: torch.Tensor,
    alpha_channel: torch.Tensor,
    scale: float = 0.95,
) -> torch.Tensor:
    """Correct RGB values to match target color distribution."""

    def flat_and_mask(bgr, a):
        mask = torch.where(a > 0.5, True, False)
        bgr_flat = bgr.reshape(-1, bgr.shape[-1])
        mask_flat = mask.reshape(-1)
        bgr_flat_masked = bgr_flat[mask_flat, :]
        return bgr_flat_masked
# ...
    else:
        corrected_bgr = torch.cat([corrected_bgr, alpha_channel], dim=-1)

    return corrected_bgr
```
**EN:** This function drives `_recorrect_rgb` with inputs such as `src_image`, `target_image`, `alpha_channel`, `scale`. Correct RGB values to match target color distribution.
**CN:** 这个函数负责 `_recorrect_rgb`，主要处理 `src_image`, `target_image`, `alpha_channel`, `scale` 等输入。 文档字符串说明：Correct RGB values to match target color distribution.

### Lines 229-483: Class `Hunyuan3DPaintPreprocessStage` / 类 `Hunyuan3DPaintPreprocessStage`
```python
class Hunyuan3DPaintPreprocessStage(PipelineStage):
    """Preprocessing: UV unwrap + delight in parallel, then multi-view rendering."""

    CAMERA_AZIMS = [0, 90, 180, 270, 0, 180]
    CAMERA_ELEVS = [0, 0, 0, 0, 90, -90]
    VIEW_WEIGHTS = [1, 0.1, 0.5, 0.1, 0.05, 0.05]

    @property
    def parallelism_type(self) -> StageParallelismType:
        return StageParallelismType.MAIN_RANK_ONLY

    def __init__(self, config: Hunyuan3D2PipelineConfig) -> None:
        super().__init__()
        self.config = config
# ...
        result.add_check("normal_maps", batch.extra.get("normal_maps"), V.is_list)
        result.add_check("position_maps", batch.extra.get("position_maps"), V.is_list)
        result.add_check("renderer", batch.extra.get("renderer"), V.not_none)
        return result
```
**EN:** This class models `Hunyuan3DPaintPreprocessStage` as a specialization of `PipelineStage`. Preprocessing: UV unwrap + delight in parallel, then multi-view rendering. Important methods include `parallelism_type`, `__init__`, `_do_uv_unwrap`, `_load_delight_model`.
**CN:** 该类实现 `Hunyuan3DPaintPreprocessStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Preprocessing: UV unwrap + delight in parallel, then multi-view rendering. 其中较重要的方法包括 `parallelism_type`, `__init__`, `_do_uv_unwrap`, `_load_delight_model`。

### Lines 487-971: Class `Hunyuan3DPaintTexGenStage` / 类 `Hunyuan3DPaintTexGenStage`
```python
class Hunyuan3DPaintTexGenStage(PipelineStage):
    def __init__(
        self,
        config: Hunyuan3D2PipelineConfig,
        paint_dir: str | None = None,
        transformer: Any = None,
        scheduler: Any = None,
        vae: Any = None,
        vae_scale_factor: int = 8,
        image_processor: Any = None,
        solver: Any = None,
        is_turbo: bool = False,
    ) -> None:
        super().__init__()
# ...
        result.add_check(
            "multiview_textures", batch.extra.get("multiview_textures"), V.is_list
        )
        return result
```
**EN:** This class models `Hunyuan3DPaintTexGenStage` as a specialization of `PipelineStage`. Important methods include `__init__`, `parallelism_type`, `_load_paint_models`, `_do_load_paint`.
**CN:** 该类实现 `Hunyuan3DPaintTexGenStage`，并继承/扩展 `PipelineStage`。 其中较重要的方法包括 `__init__`, `parallelism_type`, `_load_paint_models`, `_do_load_paint`。

### Lines 975-1064: Class `Hunyuan3DPaintPostprocessStage` / 类 `Hunyuan3DPaintPostprocessStage`
```python
class Hunyuan3DPaintPostprocessStage(PipelineStage):
    """Texture baking from multi-view images and final mesh export."""

    @property
    def parallelism_type(self) -> StageParallelismType:
        return StageParallelismType.MAIN_RANK_ONLY

    def __init__(self, config: Hunyuan3D2PipelineConfig) -> None:
        super().__init__()
        self.config = config

    def forward(self, batch: Req, server_args: ServerArgs) -> OutputBatch:
        if batch.extra.get("_mesh_failed"):
            logger.warning("Mesh generation failed, skipping paint postprocess")
# ...
        result.add_check("camera_elevs", batch.extra.get("camera_elevs"), V.is_list)
        result.add_check("camera_azims", batch.extra.get("camera_azims"), V.is_list)
        result.add_check("view_weights", batch.extra.get("view_weights"), V.is_list)
        return result
```
**EN:** This class models `Hunyuan3DPaintPostprocessStage` as a specialization of `PipelineStage`. Texture baking from multi-view images and final mesh export. Important methods include `parallelism_type`, `__init__`, `forward`, `_cleanup_obj_artifacts`.
**CN:** 该类实现 `Hunyuan3DPaintPostprocessStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Texture baking from multi-view images and final mesh export. 其中较重要的方法包括 `parallelism_type`, `__init__`, `forward`, `_cleanup_obj_artifacts`。

### Lines 1065-1071: Registration and exports / 注册与导出
```python


__all__ = [
    "Hunyuan3DPaintPreprocessStage",
    "Hunyuan3DPaintTexGenStage",
    "Hunyuan3DPaintPostprocessStage",
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.hunyuan3d`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.mesh3d_utils`
- **External / 外部**: `__future__`, `numpy`, `torch`, `diffusers.pipelines.stable_diffusion.pipeline_stable_diffusion`, `einops`, `PIL`, `diffusers`, `huggingface_hub`
- **Stdlib / 标准库**: `os`, `typing`, `time`, `concurrent.futures`, `copy`, `json`, `random`, `inspect`
