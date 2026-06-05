# scheduling_comfyui_passthrough.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/schedulers/scheduling_comfyui_passthrough.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for scheduling comfyui passthrough within the multimodal runtime. Key symbols include `ComfyUIPassThroughSchedulerOutput`, `ComfyUIPassThroughScheduler`. / 该模块实现多模态运行时中与 scheduling comfyui passthrough 相关的模型构件。 关键符号包括 `ComfyUIPassThroughSchedulerOutput`, `ComfyUIPassThroughScheduler`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo
# SPDX-License-Identifier: Apache-2.0
"""
Pass-through scheduler for ComfyUI integration.

This scheduler does not modify latents - it simply returns the input sample unchanged.
The actual denoising logic is handled by ComfyUI.
"""

import torch
from diffusers.configuration_utils import ConfigMixin, register_to_config
from diffusers.schedulers.scheduling_utils import SchedulerMixin
from diffusers.utils import BaseOutput

from sglang.multimodal_gen.runtime.models.schedulers.base import BaseScheduler
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-29: Class `ComfyUIPassThroughSchedulerOutput` / 类 `ComfyUIPassThroughSchedulerOutput`
```python
class ComfyUIPassThroughSchedulerOutput(BaseOutput):
    """
    Output class for the scheduler's `step` function output.

    Args:
        prev_sample (`torch.FloatTensor`): The input sample unchanged (pass-through).
    """

    prev_sample: torch.FloatTensor
```
**EN:** This class models `ComfyUIPassThroughSchedulerOutput` as a specialization of `BaseOutput`. Output class for the scheduler's `step` function output.
**CN:** 该类实现 `ComfyUIPassThroughSchedulerOutput`，并继承/扩展 `BaseOutput`。 文档字符串指出：Output class for the scheduler's `step` function output.

### Lines 32-190: Class `ComfyUIPassThroughScheduler` / 类 `ComfyUIPassThroughScheduler`
```python
class ComfyUIPassThroughScheduler(BaseScheduler, ConfigMixin, SchedulerMixin):
    """
    Pass-through scheduler for ComfyUI integration.

    This scheduler does not modify latents. It is used when the denoising logic
    is handled externally by ComfyUI. The scheduler simply returns the input
    sample unchanged, allowing ComfyUI to manage the denoising process.

    Usage:
        - num_inference_steps is always 1 (each step is handled separately)
        - timesteps are provided externally by ComfyUI
        - step() returns the input sample unchanged
    """

# ...
        Returns:
            Original samples unchanged
        """
        return original_samples
```
**EN:** This class models `ComfyUIPassThroughScheduler` as a specialization of `BaseScheduler`, `ConfigMixin`, `SchedulerMixin`. Pass-through scheduler for ComfyUI integration. Important methods include `__init__`, `set_timesteps`, `step`, `scale_model_input`.
**CN:** 该类实现 `ComfyUIPassThroughScheduler`，并继承/扩展 `BaseScheduler`, `ConfigMixin`, `SchedulerMixin`。 文档字符串指出：Pass-through scheduler for ComfyUI integration. 其中较重要的方法包括 `__init__`, `set_timesteps`, `step`, `scale_model_input`。

### Lines 191-193: Top-level configuration / 顶层配置
```python


EntryClass = ComfyUIPassThroughScheduler
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Video generation flow / 视频生成流程
- Symbol `ComfyUIPassThroughSchedulerOutput` anchors the module API / 符号 `ComfyUIPassThroughSchedulerOutput` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.models.schedulers.base`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `diffusers.configuration_utils`, `diffusers.schedulers.scheduling_utils`, `diffusers.utils`
