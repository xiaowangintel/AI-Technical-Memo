# rollout_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/post_training/rollout_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `_extract_single_sample_tensor`, `_slice_rollout_trajectory_for_sample`, and `_serialize_rollout_trajectory`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Rollout HTTP API (``POST /rollout/generate``). / 该文件属于服务入口层。它围绕 `_extract_single_sample_tensor`、`_slice_rollout_trajectory_for_sample` 和 `_serialize_rollout_trajectory` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: module setup and imports / 模块初始化与导入
```python
"""Rollout HTTP API (``POST /rollout/generate``)."""

from __future__ import annotations

from typing import Any

import torch
from fastapi import APIRouter, HTTPException
from fastapi.responses import ORJSONResponse

from sglang.multimodal_gen.configs.sample.sampling_params import generate_request_id
from sglang.multimodal_gen.runtime.entrypoints.openai.utils import build_sampling_params
from sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct import (
    RolloutRequest,
    RolloutResponse,
)
from sglang.multimodal_gen.runtime.entrypoints.post_training.utils import (
    _maybe_serialize,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import prepare_request
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.post_training.rl_dataclasses import (
    RolloutDebugTensors,
    RolloutDenoisingEnv,
    RolloutDitTrajectory,
    RolloutTrajectoryData,
)
from sglang.multimodal_gen.runtime.scheduler_client import async_scheduler_client
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `__future__`, `typing`, `torch`, `fastapi`, `fastapi.responses`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`typing`、`torch`、`fastapi`、`fastapi.responses` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 32-33: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
router = APIRouter(prefix="/rollout", tags=["rollout"])
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `router`. The code collaborates with `init_logger`, and `APIRouter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `router` 等名称。 代码会与 `init_logger` 和 `APIRouter` 协同工作。

### Lines 36-52: `_extract_single_sample_tensor` implementation / `_extract_single_sample_tensor` 实现
```python
def _extract_single_sample_tensor(obj: Any, sample_idx: int, batch_size: int) -> Any:
    if isinstance(obj, torch.Tensor):
        if obj.dim() >= 1 and obj.shape[0] == batch_size:
            return obj[sample_idx].contiguous()
        return obj
    if isinstance(obj, dict):
        return {
            k: _extract_single_sample_tensor(v, sample_idx, batch_size)
            for k, v in obj.items()
        }
    if isinstance(obj, list):
        return [_extract_single_sample_tensor(v, sample_idx, batch_size) for v in obj]
    if isinstance(obj, tuple):
        return tuple(
            _extract_single_sample_tensor(v, sample_idx, batch_size) for v in obj
        )
    return obj
```
**EN:** This block defines function `_extract_single_sample_tensor`. It handles extract single sample tensor logic. Key calls include `isinstance`, `tuple`, `obj.contiguous`, `_extract_single_sample_tensor`, and `obj.dim`. The implementation branches on conditions. Parameters such as `obj`, `sample_idx`, and `batch_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_extract_single_sample_tensor`。 它用于处理 extract single sample tensor 相关逻辑。 关键调用包括 `isinstance`、`tuple`、`obj.contiguous`、`_extract_single_sample_tensor` 和 `obj.dim`。 实现中包含条件分支。 本段逻辑主要由 `obj`、`sample_idx` 和 `batch_size` 等参数驱动。

### Lines 55-127: `_slice_rollout_trajectory_for_sample` implementation / `_slice_rollout_trajectory_for_sample` 实现
```python
def _slice_rollout_trajectory_for_sample(
    rtd: RolloutTrajectoryData | None,
    sample_idx: int,
    batch_size: int,
) -> RolloutTrajectoryData | None:
    if rtd is None:
        return None
    log_probs = rtd.rollout_log_probs
    if (
        isinstance(log_probs, torch.Tensor)
        and log_probs.dim() >= 1
        and log_probs.shape[0] == batch_size
    ):
        log_probs = log_probs[sample_idx].contiguous()
    debug_tensors = None
    if rtd.rollout_debug_tensors:
        rd = rtd.rollout_debug_tensors
        debug_tensors = RolloutDebugTensors(
            rollout_variance_noises=_extract_single_sample_tensor(
                rd.rollout_variance_noises, sample_idx, batch_size
            ),
            rollout_prev_sample_means=_extract_single_sample_tensor(
                rd.rollout_prev_sample_means, sample_idx, batch_size
            ),
            rollout_noise_std_devs=_extract_single_sample_tensor(
                rd.rollout_noise_std_devs, sample_idx, batch_size
            ),
            rollout_model_outputs=_extract_single_sample_tensor(
                rd.rollout_model_outputs, sample_idx, batch_size
            ),
        )
    denoising_env = None
    if rtd.denoising_env:
        env = rtd.denoising_env
        denoising_env = RolloutDenoisingEnv(
            image_kwargs=(
                _extract_single_sample_tensor(env.image_kwargs, sample_idx, batch_size)
                if env.image_kwargs
                else None
            ),
            pos_cond_kwargs=(
                _extract_single_sample_tensor(
                    env.pos_cond_kwargs, sample_idx, batch_size
                )
                if env.pos_cond_kwargs
                else None
            ),
            neg_cond_kwargs=(
                _extract_single_sample_tensor(
                    env.neg_cond_kwargs, sample_idx, batch_size
                )
                if env.neg_cond_kwargs
                else None
            ),
            guidance=(
                _extract_single_sample_tensor(env.guidance, sample_idx, batch_size)
                if env.guidance is not None
                else None
            ),
        )
    dit_trajectory = None
    if rtd.dit_trajectory:
        dit = rtd.dit_trajectory
        dit_trajectory = RolloutDitTrajectory(
            latents=_extract_single_sample_tensor(dit.latents, sample_idx, batch_size),
            timesteps=dit.timesteps,
        )
    return RolloutTrajectoryData(
        rollout_log_probs=log_probs,
        rollout_debug_tensors=debug_tensors,
        denoising_env=denoising_env,
        dit_trajectory=dit_trajectory,
    )
```
**EN:** This block defines function `_slice_rollout_trajectory_for_sample`. It handles slice rollout trajectory for sample logic. Key calls include `RolloutTrajectoryData`, `isinstance`, `log_probs.contiguous`, `RolloutDebugTensors`, and `RolloutDenoisingEnv`. The implementation branches on conditions. Parameters such as `rtd`, `sample_idx`, and `batch_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_slice_rollout_trajectory_for_sample`。 它用于处理 slice rollout trajectory for sample 相关逻辑。 关键调用包括 `RolloutTrajectoryData`、`isinstance`、`log_probs.contiguous`、`RolloutDebugTensors` 和 `RolloutDenoisingEnv`。 实现中包含条件分支。 本段逻辑主要由 `rtd`、`sample_idx` 和 `batch_size` 等参数驱动。

### Lines 130-179: `_serialize_rollout_trajectory` implementation / `_serialize_rollout_trajectory` 实现
```python
def _serialize_rollout_trajectory(
    rtd: RolloutTrajectoryData | None,
    *,
    serialized_dit_timesteps: dict | None = None,
) -> tuple[dict | None, dict | None, dict | None, dict | None]:
    """Return order: rollout_log_probs, rollout_debug_tensors, denoising_env, dit_trajectory."""
    if rtd is None:
        return None, None, None, None
    serialized_log_probs = _maybe_serialize(rtd.rollout_log_probs)
    serialized_debug_tensors = None
    if rtd.rollout_debug_tensors:
        rd = rtd.rollout_debug_tensors
        serialized_debug_tensors = {
            "rollout_variance_noises": _maybe_serialize(rd.rollout_variance_noises),
            "rollout_prev_sample_means": _maybe_serialize(rd.rollout_prev_sample_means),
            "rollout_noise_std_devs": _maybe_serialize(rd.rollout_noise_std_devs),
            "rollout_model_outputs": _maybe_serialize(rd.rollout_model_outputs),
        }
    serialized_denoising_env = None
    if rtd.denoising_env:
        env = rtd.denoising_env
        serialized_denoising_env = {
            "image_kwargs": (
                _maybe_serialize(env.image_kwargs) if env.image_kwargs else None
            ),
            "pos_cond_kwargs": (
                _maybe_serialize(env.pos_cond_kwargs) if env.pos_cond_kwargs else None
            ),
            "neg_cond_kwargs": (
                _maybe_serialize(env.neg_cond_kwargs) if env.neg_cond_kwargs else None
            ),
            "guidance": (
                _maybe_serialize(env.guidance) if env.guidance is not None else None
            ),
        }
    serialized_dit_trajectory = None
    if rtd.dit_trajectory:
        dit = rtd.dit_trajectory
        serialized_dit_trajectory = {
            "latents": (
                _maybe_serialize(dit.latents) if dit.latents is not None else None
            ),
            "timesteps": serialized_dit_timesteps,
        }
    return (
        serialized_log_probs,
        serialized_debug_tensors,
        serialized_denoising_env,
        serialized_dit_trajectory,
    )
```
**EN:** This block defines function `_serialize_rollout_trajectory`. Return order: rollout_log_probs, rollout_debug_tensors, denoising_env, dit_trajectory. Key calls include `_maybe_serialize`. The implementation branches on conditions. Parameters such as `rtd` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_serialize_rollout_trajectory`。 它用于处理 serialize rollout trajectory 相关逻辑。 关键调用包括 `_maybe_serialize`。 实现中包含条件分支。 本段逻辑主要由 `rtd` 等参数驱动。

### Lines 182-249: `_build_response` implementation / `_build_response` 实现
```python
def _build_response(
    request_id: str, prompt: str, seed: int, rollout: bool, result: OutputBatch
) -> list[RolloutResponse]:
    """
    rollout: bool - set to False when evaluating the model
    """
    batch_size = result.output.shape[0]
    inference_time_s = (
        result.metrics.total_duration_s
        if result.metrics and result.metrics.total_duration_s > 0
        else None
    )
    peak_memory_mb = result.peak_memory_mb if result.peak_memory_mb > 0 else None
    rollout_trajectory_data = result.rollout_trajectory_data
    if rollout:
        assert (
            rollout_trajectory_data is not None
        ), "rollout_trajectory_data must be present when rollout=True"

    serialized_dit_timesteps = None
    if rollout and rollout_trajectory_data and rollout_trajectory_data.dit_trajectory:
        serialized_dit_timesteps = _maybe_serialize(
            rollout_trajectory_data.dit_trajectory.timesteps
        )

    responses: list[RolloutResponse] = []
    for sample_idx in range(batch_size):
        out_i = result.output[sample_idx].contiguous()
        serialized_generated_output = _maybe_serialize(out_i)
        if not rollout:
            responses.append(
                RolloutResponse(
                    request_id=request_id,
                    prompt=prompt,
                    seed=seed,
                    generated_output=serialized_generated_output,
                    inference_time_s=inference_time_s,
                    peak_memory_mb=peak_memory_mb,
                )
            )
            continue
        per_sample_trajectory = _slice_rollout_trajectory_for_sample(
            result.rollout_trajectory_data, sample_idx, batch_size
        )
        (
            serialized_log_probs,
            serialized_debug_tensors,
            serialized_denoising_env,
            serialized_dit_trajectory,
        ) = _serialize_rollout_trajectory(
            per_sample_trajectory,
            serialized_dit_timesteps=serialized_dit_timesteps,
        )
        responses.append(
            RolloutResponse(
                request_id=request_id,
                prompt=prompt,
                seed=seed,
                generated_output=serialized_generated_output,
                rollout_log_probs=serialized_log_probs,
                rollout_debug_tensors=serialized_debug_tensors,
                denoising_env=serialized_denoising_env,
                dit_trajectory=serialized_dit_trajectory,
                inference_time_s=inference_time_s,
                peak_memory_mb=peak_memory_mb,
            )
        )
    return responses
```
**EN:** This block defines function `_build_response`. rollout: bool - set to False when evaluating the model Key calls include `range`, `_maybe_serialize`, `result.output.contiguous`, `_slice_rollout_trajectory_for_sample`, and `_serialize_rollout_trajectory`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `request_id`, `prompt`, `seed`, `rollout`, and `result` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_response`。 它用于构建response。 关键调用包括 `range`、`_maybe_serialize`、`result.output.contiguous`、`_slice_rollout_trajectory_for_sample` 和 `_serialize_rollout_trajectory`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `request_id`、`prompt`、`seed`、`rollout` 和 `result` 等参数驱动。

### Lines 252-284: `_build_sampling_kwargs` implementation / `_build_sampling_kwargs` 实现
```python
def _build_sampling_kwargs(request: RolloutRequest) -> dict:
    sampling_kwargs: dict = dict(
        prompt=request.prompt,
        negative_prompt=request.negative_prompt,
        seed=request.seed,
        generator_device=request.generator_device,
        width=request.width,
        height=request.height,
        num_inference_steps=request.num_inference_steps,
        num_outputs_per_prompt=request.num_outputs_per_prompt,
        guidance_scale=request.guidance_scale,
        true_cfg_scale=request.true_cfg_scale,
        num_frames=request.num_frames,
        fps=request.fps,
        image_path=request.image_path,
        rollout=request.rollout,
        rollout_sde_type=request.rollout_sde_type,
        rollout_noise_level=request.rollout_noise_level,
        rollout_log_prob_no_const=request.rollout_log_prob_no_const,
        rollout_debug_mode=request.rollout_debug_mode,
        rollout_return_denoising_env=request.rollout_return_denoising_env,
        rollout_return_dit_trajectory=request.rollout_return_dit_trajectory,
        rollout_sde_step_indices=request.rollout_sde_step_indices,
        rollout_return_step_indices=request.rollout_return_step_indices,
        suppress_logs=request.suppress_logs,
        save_output=False,
        return_trajectory_latents=False,
        return_trajectory_decoded=False,
    )
    if request.extra_sampling_params:
        sampling_kwargs.update(request.extra_sampling_params)
        sampling_kwargs["rollout"] = request.rollout
    return {k: v for k, v in sampling_kwargs.items() if v is not None}
```
**EN:** This block defines function `_build_sampling_kwargs`. It builds sampling kwargs. Key calls include `dict`, `sampling_kwargs.update`, and `sampling_kwargs.items`. The implementation branches on conditions. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_sampling_kwargs`。 它用于构建sampling kwargs。 关键调用包括 `dict`、`sampling_kwargs.update` 和 `sampling_kwargs.items`。 实现中包含条件分支。 本段逻辑主要由 `request` 等参数驱动。

### Lines 287-315: `rollout_generate` implementation / `rollout_generate` 实现
```python
@router.post("/generate", response_model=list[RolloutResponse])
async def rollout_generate(request: RolloutRequest):
    request_id = generate_request_id()
    server_args = get_global_server_args()
    sampling_kwargs = _build_sampling_kwargs(request)
    try:
        sampling_params = build_sampling_params(request_id, **sampling_kwargs)
    except Exception as exc:
        raise HTTPException(
            status_code=400, detail=f"Invalid sampling params: {exc}"
        ) from exc
    pipeline_request = prepare_request(
        server_args=server_args, sampling_params=sampling_params
    )
    try:
        output_batch: OutputBatch = await async_scheduler_client.forward(
            pipeline_request
        )
    except Exception as exc:
        logger.error("Rollout generation failed: %s", exc, exc_info=True)
        raise HTTPException(
            status_code=500, detail=f"Generation failed: {exc}"
        ) from exc
    if output_batch.error:
        raise HTTPException(status_code=500, detail=output_batch.error)
    rollout_responses = _build_response(
        request_id, request.prompt, request.seed, request.rollout, output_batch
    )
    return ORJSONResponse(content=[r.model_dump() for r in rollout_responses])
```
**EN:** This block defines function `rollout_generate`. It handles rollout generate logic. Key calls include `router.post`, `generate_request_id`, `get_global_server_args`, `_build_sampling_kwargs`, and `prepare_request`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request` drive the behavior in this section.
**CN:** 该代码块定义了函数 `rollout_generate`。 它用于处理 rollout generate 相关逻辑。 关键调用包括 `router.post`、`generate_request_id`、`get_global_server_args`、`_build_sampling_kwargs` 和 `prepare_request`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request` 等参数驱动。

## Key Concepts / 关键概念
- `_extract_single_sample_tensor`: Top-level function that handles extract single sample tensor logic. / 顶层函数，用于处理 extract single sample tensor 相关逻辑。
- `_slice_rollout_trajectory_for_sample`: Top-level function that handles slice rollout trajectory for sample logic. / 顶层函数，用于处理 slice rollout trajectory for sample 相关逻辑。
- `_serialize_rollout_trajectory`: Return order: rollout_log_probs, rollout_debug_tensors, denoising_env, dit_trajectory. / 顶层函数，用于处理 serialize rollout trajectory 相关逻辑。
- `_build_response`: rollout: bool - set to False when evaluating the model / 顶层函数，用于构建response。
- `_build_sampling_kwargs`: Top-level function that builds sampling kwargs. / 顶层函数，用于构建sampling kwargs。
- `rollout_generate`: Top-level function that handles rollout generate logic. / 顶层函数，用于处理 rollout generate 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方依赖**: `torch`, `fastapi`, `fastapi.responses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.openai.utils`, `sglang.multimodal_gen.runtime.entrypoints.post_training.io_struct`, `sglang.multimodal_gen.runtime.entrypoints.post_training.utils`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 315
