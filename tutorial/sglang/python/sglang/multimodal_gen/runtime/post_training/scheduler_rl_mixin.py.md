# scheduler_rl_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/post_training/scheduler_rl_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for scheduler rl mixin in the multimodal generation stack. Key symbols include `SchedulerRLMixin`. / 该模块包含多模态生成体系中与 scheduler rl mixin 相关的运行时支持代码。 关键符号包括 `SchedulerRLMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""Flow-matching rollout step utilities for log-prob computation."""

import math
from typing import Any, Union

import torch

from sglang.multimodal_gen.runtime.distributed import (
    get_sp_world_size,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.post_training.rl_dataclasses import (
    RolloutSessionData,
# ...
    SchedulerRLDebugMixin,
)

_LOG_SQRT_2PI = math.log(math.sqrt(2 * math.pi))
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 23-295: Class `SchedulerRLMixin` / 类 `SchedulerRLMixin`
```python
class SchedulerRLMixin(SchedulerRLDebugMixin):
    @staticmethod
    def _get_rollout_session_data(batch) -> RolloutSessionData:
        """Return the RolloutSessionData attached to *batch*, or raise if not prepared."""
        rollout_session_data = getattr(batch, "_rollout_session_data", None)
        if rollout_session_data is None:
            raise RuntimeError("prepare_rollout() not called before rollout")
        return rollout_session_data

    def release_rollout_resources(self, batch) -> None:
        """Release rollout-owned resources. Call when denoising ends or before a new rollout."""
        batch._rollout_session_data = None

    def prepare_rollout(self, batch: Req, pipeline_config: Any = None) -> None:
# ...
            self.consume_local_rollout_log_probs(batch)
        )
        rollout_log_probs_tensor = trajectory_log_prob_sum / trajectory_log_prob_count
        return rollout_log_probs_tensor.cpu()
```
**EN:** This class models `SchedulerRLMixin` as a specialization of `SchedulerRLDebugMixin`. Important methods include `_get_rollout_session_data`, `release_rollout_resources`, `prepare_rollout`, `already_prepared_rollout`.
**CN:** 该类实现 `SchedulerRLMixin`，并继承/扩展 `SchedulerRLDebugMixin`。 其中较重要的方法包括 `_get_rollout_session_data`, `release_rollout_resources`, `prepare_rollout`, `already_prepared_rollout`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Distributed execution / 分布式执行
- Sampling parameter control / 采样参数控制

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`, `sglang.multimodal_gen.runtime.post_training.scheduler_rl_debug_mixin`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `math`, `typing`
