# wan_ti2v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/wan_ti2v.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for wan ti2v, connecting stages, tensors, and runtime metadata. Key symbols include `should_apply_wan_ti2v`, `prepare_wan_ti2v_latents`, `prepare_wan_ti2v_sp_inputs`. / 该模块负责 wan ti2v 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `should_apply_wan_ti2v`, `prepare_wan_ti2v_latents`, `prepare_wan_ti2v_sp_inputs`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup / 导入与模块初始化
```python
"""WAN TI2V-specific helpers shared by the generic denoising stage."""

import math

import torch
from einops import rearrange

from sglang.multimodal_gen.configs.pipeline_configs.base import ModelTaskType
from sglang.multimodal_gen.configs.pipeline_configs.wan import (
    Wan2_2_TI2V_5B_Config,
)
from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
    get_sp_parallel_rank,
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.utils import masks_like
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 22-29: Function `should_apply_wan_ti2v` / 函数 `should_apply_wan_ti2v`
```python
def should_apply_wan_ti2v(batch: Req, server_args: ServerArgs) -> bool:
    """Return whether the request should use the Wan2.2 TI2V latent path."""

    return bool(
        server_args.pipeline_config.task_type == ModelTaskType.TI2V
        and batch.condition_image is not None
        and type(server_args.pipeline_config) is Wan2_2_TI2V_5B_Config
    )
```
**EN:** This function drives `should_apply_wan_ti2v` with inputs such as `batch`, `server_args`. Return whether the request should use the Wan2.2 TI2V latent path.
**CN:** 这个函数负责 `should_apply_wan_ti2v`，主要处理 `batch`, `server_args` 等输入。 文档字符串说明：Return whether the request should use the Wan2.2 TI2V latent path.

### Lines 32-91: Function `prepare_wan_ti2v_latents` / 函数 `prepare_wan_ti2v_latents`
```python
def prepare_wan_ti2v_latents(
    vae: object,
    latents: torch.Tensor,
    target_dtype: torch.dtype,
    vae_dtype: torch.dtype,
    batch: Req,
    server_args: ServerArgs,
) -> tuple[int, torch.Tensor, list[torch.Tensor]]:
    """Encode the conditioning image and splice it into Wan TI2V latents."""

    # Wan2.2 TI2V directly replaces the first frame of the latent with
    # the image latent instead of appending along the channel dim.
    assert batch.image_latent is None, "TI2V task should not have image latents"
    assert vae is not None, "VAE is not provided for TI2V task"
# ...
        // (patch_size[1] * patch_size[2])
    )
    seq_len = int(math.ceil(seq_len / get_sp_world_size())) * get_sp_world_size()
    return seq_len, z, reserved_frames_masks
```
**EN:** This function drives `prepare_wan_ti2v_latents` with inputs such as `vae`, `latents`, `target_dtype`, `vae_dtype`. Encode the conditioning image and splice it into Wan TI2V latents.
**CN:** 这个函数负责 `prepare_wan_ti2v_latents`，主要处理 `vae`, `latents`, `target_dtype`, `vae_dtype` 等输入。 文档字符串说明：Encode the conditioning image and splice it into Wan TI2V latents.

### Lines 94-132: Function `prepare_wan_ti2v_sp_inputs` / 函数 `prepare_wan_ti2v_sp_inputs`
```python
def prepare_wan_ti2v_sp_inputs(
    z: torch.Tensor | None,
    reserved_frames_masks: list[torch.Tensor] | None,
    batch: Req,
) -> tuple[torch.Tensor | None, torch.Tensor | None]:
    """Shard Wan TI2V image-conditioning state to match SP-sharded video latents."""

    rank_in_sp_group = get_sp_parallel_rank()
    sp_world_size = get_sp_world_size()

    if getattr(batch, "did_sp_shard_latents", False):
        if z is not None and z.shape[2] == 1:
            z_sp = z if rank_in_sp_group == 0 else None
        else:
# ...
            reserved_frames_masks[0] if reserved_frames_masks is not None else None
        )

    return reserved_frames_mask_sp, z_sp
```
**EN:** This function drives `prepare_wan_ti2v_sp_inputs` with inputs such as `z`, `reserved_frames_masks`, `batch`. Shard Wan TI2V image-conditioning state to match SP-sharded video latents.
**CN:** 这个函数负责 `prepare_wan_ti2v_sp_inputs`，主要处理 `z`, `reserved_frames_masks`, `batch` 等输入。 文档字符串说明：Shard Wan TI2V image-conditioning state to match SP-sharded video latents.

### Lines 135-161: Function `expand_wan_ti2v_timestep` / 函数 `expand_wan_ti2v_timestep`
```python
def expand_wan_ti2v_timestep(
    batch: Req,
    t_device: torch.Tensor,
    target_dtype: torch.dtype,
    seq_len: int,
    reserved_frames_mask: torch.Tensor | None,
) -> torch.Tensor:
    """Expand the timestep tensor for Wan TI2V's first-frame masking semantics."""

    batch_size = batch.raw_latent_shape[0]
    t_device_rounded = t_device.to(target_dtype)

    local_seq_len = seq_len
    if get_sp_world_size() > 1 and getattr(batch, "did_sp_shard_latents", False):
# ...
        )
        return temp_ts.unsqueeze(0).repeat(batch_size, 1)

    return t_device.repeat(batch_size, local_seq_len)
```
**EN:** This function drives `expand_wan_ti2v_timestep` with inputs such as `batch`, `t_device`, `target_dtype`, `seq_len`. Expand the timestep tensor for Wan TI2V's first-frame masking semantics.
**CN:** 这个函数负责 `expand_wan_ti2v_timestep`，主要处理 `batch`, `t_device`, `target_dtype`, `seq_len` 等输入。 文档字符串说明：Expand the timestep tensor for Wan TI2V's first-frame masking semantics.

### Lines 164-175: Function `blend_wan_ti2v_latents` / 函数 `blend_wan_ti2v_latents`
```python
def blend_wan_ti2v_latents(
    latents: torch.Tensor,
    reserved_frames_mask: torch.Tensor | None,
    z: torch.Tensor | None,
) -> torch.Tensor:
    """Restore Wan TI2V's conditioned first frame after each denoising step."""

    if z is None or reserved_frames_mask is None:
        return latents
    return (
        1.0 - reserved_frames_mask.unsqueeze(0)
    ) * z + reserved_frames_mask.unsqueeze(0) * latents
```
**EN:** This function drives `blend_wan_ti2v_latents` with inputs such as `latents`, `reserved_frames_mask`, `z`. Restore Wan TI2V's conditioned first frame after each denoising step.
**CN:** 这个函数负责 `blend_wan_ti2v_latents`，主要处理 `latents`, `reserved_frames_mask`, `z` 等输入。 文档字符串说明：Restore Wan TI2V's conditioned first frame after each denoising step.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.wan`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.utils`
- **External / 外部**: `torch`, `einops`
- **Stdlib / 标准库**: `math`
