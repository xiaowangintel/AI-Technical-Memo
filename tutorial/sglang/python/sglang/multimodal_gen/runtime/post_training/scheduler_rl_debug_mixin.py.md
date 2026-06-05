# scheduler_rl_debug_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/post_training/scheduler_rl_debug_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for scheduler rl debug mixin in the multimodal generation stack. Key symbols include `SchedulerRLDebugMixin`. / 该模块包含多模态生成体系中与 scheduler rl debug mixin 相关的运行时支持代码。 关键符号包括 `SchedulerRLDebugMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""Debug tensor helpers for rollout-enabled schedulers."""

import torch

from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.post_training.rl_dataclasses import (
    RolloutDebugTensors,
    RolloutSessionData,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 17-116: Class `SchedulerRLDebugMixin` / 类 `SchedulerRLDebugMixin`
```python
class SchedulerRLDebugMixin:
    @staticmethod
    def _reset_rollout_debug_tensors(rollout_session_data: RolloutSessionData) -> None:
        rollout_session_data.local_variance_noises = []
        rollout_session_data.local_prev_sample_means = []
        rollout_session_data.local_noise_std_devs = []
        rollout_session_data.local_model_outputs = []

    def append_local_rollout_debug_tensors(
        self,
        batch,
        *,
        variance_noise: torch.Tensor,
        prev_sample_mean: torch.Tensor,
# ...
            rollout_prev_sample_means=prev_sample_means.cpu(),
            rollout_noise_std_devs=noise_std_devs.cpu(),
            rollout_model_outputs=model_outputs.cpu(),
        )
```
**EN:** This class models `SchedulerRLDebugMixin`. Important methods include `_reset_rollout_debug_tensors`, `append_local_rollout_debug_tensors`, `consume_local_rollout_debug_tensors`, `collect_rollout_debug_tensors`.
**CN:** 该类实现 `SchedulerRLDebugMixin`。 其中较重要的方法包括 `_reset_rollout_debug_tensors`, `append_local_rollout_debug_tensors`, `consume_local_rollout_debug_tensors`, `collect_rollout_debug_tensors`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Distributed execution / 分布式执行
- Symbol `SchedulerRLDebugMixin` anchors the module API / 符号 `SchedulerRLDebugMixin` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`
- **External / 外部**: `torch`
