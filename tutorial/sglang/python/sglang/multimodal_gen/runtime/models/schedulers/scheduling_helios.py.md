# scheduling_helios.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/schedulers/scheduling_helios.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for scheduling helios within the multimodal runtime. Key symbols include `HeliosSchedulerOutput`, `HeliosSchedulerConfig`, `HeliosScheduler`. / 该模块实现多模态运行时中与 scheduling helios 相关的模型构件。 关键符号包括 `HeliosSchedulerOutput`, `HeliosSchedulerConfig`, `HeliosScheduler`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
# Adapted from Helios diffusers scheduler:
# https://github.com/BestWishYsh/Helios
"""
Helios scheduler implementing flow-matching with UniPC/Euler solvers.

For Phase 1 T2V (stages=1), this simplifies to standard flow-matching
with dynamic shifting and UniPC multistep solver.
"""

import math
from dataclasses import dataclass

import numpy as np
import torch
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 18-23: Class `HeliosSchedulerOutput` / 类 `HeliosSchedulerOutput`
```python
@dataclass
class HeliosSchedulerOutput:
    prev_sample: torch.FloatTensor
    model_outputs: torch.FloatTensor | None = None
    last_sample: torch.FloatTensor | None = None
    this_order: int | None = None
```
**EN:** This class models `HeliosSchedulerOutput`.
**CN:** 该类实现 `HeliosSchedulerOutput`。

### Lines 26-34: Class `HeliosSchedulerConfig` / 类 `HeliosSchedulerConfig`
```python
class HeliosSchedulerConfig:
    """Mimics diffusers config interface for scheduler parameters."""

    def __init__(self, **kwargs):
        for k, v in kwargs.items():
            setattr(self, k, v)

    def get(self, key, default=None):
        return getattr(self, key, default)
```
**EN:** This class models `HeliosSchedulerConfig`. Mimics diffusers config interface for scheduler parameters. Important methods include `__init__`, `get`.
**CN:** 该类实现 `HeliosSchedulerConfig`。 文档字符串指出：Mimics diffusers config interface for scheduler parameters. 其中较重要的方法包括 `__init__`, `get`。

### Lines 37-731: Class `HeliosScheduler` / 类 `HeliosScheduler`
```python
class HeliosScheduler:
    """
    Helios multi-stage scheduler supporting Euler, UniPC, and DMD solvers.

    For Phase 1 T2V with stages=1, this is a standard flow-matching scheduler
    with optional time shifting and UniPC multistep updates.
    """

    order = 1

    def __init__(
        self,
        num_train_timesteps: int = 1000,
        shift: float = 1.0,
# ...
        self.shift = shift

    def __len__(self):
        return self.config.num_train_timesteps
```
**EN:** This class models `HeliosScheduler`. Helios multi-stage scheduler supporting Euler, UniPC, and DMD solvers. Important methods include `__init__`, `init_sigmas`, `init_sigmas_for_each_stage`, `step_index`.
**CN:** 该类实现 `HeliosScheduler`。 文档字符串指出：Helios multi-stage scheduler supporting Euler, UniPC, and DMD solvers. 其中较重要的方法包括 `__init__`, `init_sigmas`, `init_sigmas_for_each_stage`, `step_index`。

### Lines 732-737: Top-level configuration / 顶层配置
```python


# Alias for Helios-Distilled which uses "HeliosDMDScheduler" in scheduler_config.json
HeliosDMDScheduler = HeliosScheduler

EntryClass = [HeliosScheduler, "HeliosDMDScheduler"]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Scheduling and batching / 调度与批处理
- Sampling parameter control / 采样参数控制
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **External / 外部**: `numpy`, `torch`
- **Stdlib / 标准库**: `math`, `dataclasses`
