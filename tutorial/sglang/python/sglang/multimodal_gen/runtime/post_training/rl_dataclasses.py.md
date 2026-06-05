# rl_dataclasses.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/post_training/rl_dataclasses.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for rl dataclasses in the multimodal generation stack. Key symbols include `RolloutSessionData`, `RolloutDebugTensors`, `RolloutDenoisingEnv`. / 该模块包含多模态生成体系中与 rl dataclasses 相关的运行时支持代码。 关键符号包括 `RolloutSessionData`, `RolloutDebugTensors`, `RolloutDenoisingEnv`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""RL-specific dataclasses used by post-training and rollout paths."""

from dataclasses import dataclass, field
from typing import Any

import torch
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 10-28: Class `RolloutSessionData` / 类 `RolloutSessionData`
```python
@dataclass
class RolloutSessionData:
    """Per-batch rollout state created by prepare_rollout(), lives on the batch object.

    Cleared by setting ``batch._rollout_session_data = None``.
    """

    pipeline_config: Any = None
    sigma_max: float = 0.0
    latents_shape: tuple | None = None
    noise_buffer: torch.Tensor | None = None

    local_log_prob_sum: list[torch.Tensor] = field(default_factory=list)
    local_log_prob_count: list[torch.Tensor] = field(default_factory=list)

    local_variance_noises: list[torch.Tensor] = field(default_factory=list)
    local_prev_sample_means: list[torch.Tensor] = field(default_factory=list)
    local_noise_std_devs: list[torch.Tensor] = field(default_factory=list)
    local_model_outputs: list[torch.Tensor] = field(default_factory=list)
```
**EN:** This class models `RolloutSessionData`. Per-batch rollout state created by prepare_rollout(), lives on the batch object.
**CN:** 该类实现 `RolloutSessionData`。 文档字符串指出：Per-batch rollout state created by prepare_rollout(), lives on the batch object.

### Lines 31-38: Class `RolloutDebugTensors` / 类 `RolloutDebugTensors`
```python
@dataclass
class RolloutDebugTensors:
    """Container for rollout debug tensors collected during denoising."""

    rollout_variance_noises: torch.Tensor | None = None
    rollout_prev_sample_means: torch.Tensor | None = None
    rollout_noise_std_devs: torch.Tensor | None = None
    rollout_model_outputs: torch.Tensor | None = None
```
**EN:** This class models `RolloutDebugTensors`. Container for rollout debug tensors collected during denoising.
**CN:** 该类实现 `RolloutDebugTensors`。 文档字符串指出：Container for rollout debug tensors collected during denoising.

### Lines 41-46: Class `RolloutDenoisingEnv` / 类 `RolloutDenoisingEnv`
```python
@dataclass
class RolloutDenoisingEnv:
    image_kwargs: dict[str, Any] | None = None
    pos_cond_kwargs: dict[str, Any] | None = None
    neg_cond_kwargs: dict[str, Any] | None = None
    guidance: torch.Tensor | None = None
```
**EN:** This class models `RolloutDenoisingEnv`.
**CN:** 该类实现 `RolloutDenoisingEnv`。

### Lines 49-54: Class `RolloutDitTrajectory` / 类 `RolloutDitTrajectory`
```python
@dataclass
class RolloutDitTrajectory:
    # [B, T+1, ...]: per-step noisy latents x_{t_0..t_{T-1}} followed by the
    # final denoised latent x_{t_T} (last scheduler.step output).
    latents: torch.Tensor | None = None
    timesteps: torch.Tensor | None = None  # [T]
```
**EN:** This class models `RolloutDitTrajectory`.
**CN:** 该类实现 `RolloutDitTrajectory`。

### Lines 57-62: Class `RolloutTrajectoryData` / 类 `RolloutTrajectoryData`
```python
@dataclass
class RolloutTrajectoryData:
    rollout_log_probs: torch.Tensor | None = None
    rollout_debug_tensors: RolloutDebugTensors | None = None
    denoising_env: RolloutDenoisingEnv | None = None
    dit_trajectory: RolloutDitTrajectory | None = None
```
**EN:** This class models `RolloutTrajectoryData`.
**CN:** 该类实现 `RolloutTrajectoryData`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- Symbol `RolloutSessionData` anchors the module API / 符号 `RolloutSessionData` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `dataclasses`, `typing`
