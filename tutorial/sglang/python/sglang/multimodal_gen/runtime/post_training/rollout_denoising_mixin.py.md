# rollout_denoising_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/post_training/rollout_denoising_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for rollout denoising mixin in the multimodal generation stack. Key symbols include `_kwargs_to_cpu`, `RolloutDenoisingMixin`. / 该模块包含多模态生成体系中与 rollout denoising mixin 相关的运行时支持代码。 关键符号包括 `_kwargs_to_cpu`, `RolloutDenoisingMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and module setup / 导入与模块初始化
```python
"""Mixin for rollout-related denoising hooks.

Moved out of DenoisingStage to keep the core stage lean.
"""

from __future__ import annotations

from typing import Any

import torch

from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.post_training.rl_dataclasses import (
    RolloutDenoisingEnv,
# ...
from sglang.multimodal_gen.runtime.post_training.sp_utils import (
    gather_stacked_latents_for_sp,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 27-36: Function `_kwargs_to_cpu` / 函数 `_kwargs_to_cpu`
```python
def _kwargs_to_cpu(d: Any) -> Any:
    if isinstance(d, torch.Tensor):
        return d.detach().cpu()
    if isinstance(d, dict):
        return {k: _kwargs_to_cpu(v) for k, v in d.items()}
    if isinstance(d, list):
        return [_kwargs_to_cpu(v) for v in d]
    if isinstance(d, tuple):
        return tuple(_kwargs_to_cpu(v) for v in d)
    return d
```
**EN:** This function drives `_kwargs_to_cpu` with inputs such as `d`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_kwargs_to_cpu`，主要处理 `d` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 39-206: Class `RolloutDenoisingMixin` / 类 `RolloutDenoisingMixin`
```python
class RolloutDenoisingMixin:

    def _maybe_prepare_rollout(self, batch: Req):
        """Prepare denoising loop for rollout."""
        if not isinstance(self.scheduler, SchedulerRLMixin):
            if batch.rollout:
                raise ValueError(
                    f"Scheduler {type(self.scheduler)} does not support rollout"
                )
            return

        self.scheduler.release_rollout_resources(batch)
        if batch.rollout:
            self.scheduler.prepare_rollout(
# ...

            batch.rollout_trajectory_data.denoising_env = env

        batch._rollout_denoising_env_state = None
```
**EN:** This class models `RolloutDenoisingMixin`. Important methods include `_maybe_prepare_rollout`, `_maybe_collect_rollout_log_probs`, `_postprocess_rollout_outputs`, `_maybe_init_denoising_env_collection`.
**CN:** 该类实现 `RolloutDenoisingMixin`。 其中较重要的方法包括 `_maybe_prepare_rollout`, `_maybe_collect_rollout_log_probs`, `_postprocess_rollout_outputs`, `_maybe_init_denoising_env_collection`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- Symbol `_kwargs_to_cpu` anchors the module API / 符号 `_kwargs_to_cpu` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`, `sglang.multimodal_gen.runtime.post_training.scheduler_rl_mixin`, `sglang.multimodal_gen.runtime.post_training.sp_utils`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `__future__`, `torch`
- **Stdlib / 标准库**: `typing`
