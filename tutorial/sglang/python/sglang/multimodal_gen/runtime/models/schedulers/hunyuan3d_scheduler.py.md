# hunyuan3d_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/schedulers/hunyuan3d_scheduler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for hunyuan3d scheduler within the multimodal runtime. Key symbols include `Hunyuan3DFlowMatchSchedulerOutput`, `Hunyuan3DFlowMatchEulerDiscreteScheduler`, `Hunyuan3DConsistencyFlowMatchSchedulerOutput`. / 该模块实现多模态运行时中与 hunyuan3d scheduler 相关的模型构件。 关键符号包括 `Hunyuan3DFlowMatchSchedulerOutput`, `Hunyuan3DFlowMatchEulerDiscreteScheduler`, `Hunyuan3DConsistencyFlowMatchSchedulerOutput`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/Tencent-Hunyuan/Hunyuan3D-2
from __future__ import annotations

import math
from dataclasses import dataclass
from typing import List, Optional, Tuple, Union

import numpy as np
import torch
from diffusers.configuration_utils import ConfigMixin, register_to_config
from diffusers.schedulers.scheduling_utils import SchedulerMixin
from diffusers.utils import BaseOutput
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 15-19: Class `Hunyuan3DFlowMatchSchedulerOutput` / 类 `Hunyuan3DFlowMatchSchedulerOutput`
```python
@dataclass
class Hunyuan3DFlowMatchSchedulerOutput(BaseOutput):
    """Output class for the scheduler's step function."""

    prev_sample: torch.FloatTensor
```
**EN:** This class models `Hunyuan3DFlowMatchSchedulerOutput` as a specialization of `BaseOutput`. Output class for the scheduler's step function.
**CN:** 该类实现 `Hunyuan3DFlowMatchSchedulerOutput`，并继承/扩展 `BaseOutput`。 文档字符串指出：Output class for the scheduler's step function.

### Lines 22-219: Class `Hunyuan3DFlowMatchEulerDiscreteScheduler` / 类 `Hunyuan3DFlowMatchEulerDiscreteScheduler`
```python
class Hunyuan3DFlowMatchEulerDiscreteScheduler(SchedulerMixin, ConfigMixin):
    """Euler discrete scheduler for flow matching."""

    # External module path aliases for compatibility with Hunyuan3D configs
    _aliases = [
        "hy3dgen.shapegen.schedulers.FlowMatchEulerDiscreteScheduler",
        "hy3dshape.schedulers.FlowMatchEulerDiscreteScheduler",
    ]

    _compatibles = []
    order = 1

    @register_to_config
    def __init__(
# ...
        return Hunyuan3DFlowMatchSchedulerOutput(prev_sample=prev_sample)

    def __len__(self) -> int:
        return self.config.num_train_timesteps
```
**EN:** This class models `Hunyuan3DFlowMatchEulerDiscreteScheduler` as a specialization of `SchedulerMixin`, `ConfigMixin`. Euler discrete scheduler for flow matching. Important methods include `__init__`, `step_index`, `begin_index`, `set_begin_index`.
**CN:** 该类实现 `Hunyuan3DFlowMatchEulerDiscreteScheduler`，并继承/扩展 `SchedulerMixin`, `ConfigMixin`。 文档字符串指出：Euler discrete scheduler for flow matching. 其中较重要的方法包括 `__init__`, `step_index`, `begin_index`, `set_begin_index`。

### Lines 222-227: Class `Hunyuan3DConsistencyFlowMatchSchedulerOutput` / 类 `Hunyuan3DConsistencyFlowMatchSchedulerOutput`
```python
@dataclass
class Hunyuan3DConsistencyFlowMatchSchedulerOutput(BaseOutput):
    """Output for consistency flow matching scheduler."""

    prev_sample: torch.FloatTensor
    pred_original_sample: torch.FloatTensor
```
**EN:** This class models `Hunyuan3DConsistencyFlowMatchSchedulerOutput` as a specialization of `BaseOutput`. Output for consistency flow matching scheduler.
**CN:** 该类实现 `Hunyuan3DConsistencyFlowMatchSchedulerOutput`，并继承/扩展 `BaseOutput`。 文档字符串指出：Output for consistency flow matching scheduler.

### Lines 230-364: Class `Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler` / 类 `Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler`
```python
class Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler(SchedulerMixin, ConfigMixin):
    """Consistency Flow Matching Euler Discrete Scheduler."""

    # External module path aliases for compatibility with Hunyuan3D configs
    _aliases = [
        "hy3dshape.schedulers.Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler",
    ]

    _compatibles = []
    order = 1

    @register_to_config
    def __init__(
        self,
# ...
        )

    def __len__(self) -> int:
        return self.config.num_train_timesteps
```
**EN:** This class models `Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler` as a specialization of `SchedulerMixin`, `ConfigMixin`. Consistency Flow Matching Euler Discrete Scheduler. Important methods include `__init__`, `step_index`, `begin_index`, `set_begin_index`.
**CN:** 该类实现 `Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler`，并继承/扩展 `SchedulerMixin`, `ConfigMixin`。 文档字符串指出：Consistency Flow Matching Euler Discrete Scheduler. 其中较重要的方法包括 `__init__`, `step_index`, `begin_index`, `set_begin_index`。

### Lines 365-371: Top-level configuration / 顶层配置
```python


# Entry class for model registry
EntryClass = [
    Hunyuan3DFlowMatchEulerDiscreteScheduler,
    Hunyuan3DConsistencyFlowMatchEulerDiscreteScheduler,
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Scheduling and batching / 调度与批处理
- Symbol `Hunyuan3DFlowMatchSchedulerOutput` anchors the module API / 符号 `Hunyuan3DFlowMatchSchedulerOutput` 构成该模块的核心 API
- Symbol `Hunyuan3DFlowMatchEulerDiscreteScheduler` anchors the module API / 符号 `Hunyuan3DFlowMatchEulerDiscreteScheduler` 构成该模块的核心 API
- Symbol `Hunyuan3DConsistencyFlowMatchSchedulerOutput` anchors the module API / 符号 `Hunyuan3DConsistencyFlowMatchSchedulerOutput` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `numpy`, `torch`, `diffusers.configuration_utils`, `diffusers.schedulers.scheduling_utils`, `diffusers.utils`
- **Stdlib / 标准库**: `math`, `dataclasses`, `typing`
