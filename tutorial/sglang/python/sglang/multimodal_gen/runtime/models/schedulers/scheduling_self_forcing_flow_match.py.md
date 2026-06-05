# scheduling_self_forcing_flow_match.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/schedulers/scheduling_self_forcing_flow_match.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for scheduling self forcing flow match within the multimodal runtime. Key symbols include `SelfForcingFlowMatchSchedulerOutput`, `SelfForcingFlowMatchScheduler`. / 该模块实现多模态运行时中与 scheduling self forcing flow match 相关的模型构件。 关键符号包括 `SelfForcingFlowMatchSchedulerOutput`, `SelfForcingFlowMatchScheduler`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
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

### Lines 15-25: Class `SelfForcingFlowMatchSchedulerOutput` / 类 `SelfForcingFlowMatchSchedulerOutput`
```python
class SelfForcingFlowMatchSchedulerOutput(BaseOutput):
    """
    Output class for the scheduler's `step` function output.

    Args:
        prev_sample (`torch.FloatTensor` of shape `(batch_size, num_channels, height, width)` for images):
            Computed sample `(x_{t-1})` of previous timestep. `prev_sample` should be used as next model input in the
            denoising loop.
    """

    prev_sample: torch.FloatTensor
```
**EN:** This class models `SelfForcingFlowMatchSchedulerOutput` as a specialization of `BaseOutput`. Output class for the scheduler's `step` function output.
**CN:** 该类实现 `SelfForcingFlowMatchSchedulerOutput`，并继承/扩展 `BaseOutput`。 文档字符串指出：Output class for the scheduler's `step` function output.

### Lines 28-139: Class `SelfForcingFlowMatchScheduler` / 类 `SelfForcingFlowMatchScheduler`
```python
class SelfForcingFlowMatchScheduler(BaseScheduler, ConfigMixin, SchedulerMixin):
    config_name = "scheduler_config.json"
    order = 1

    @register_to_config
    def __init__(
        self,
        num_inference_steps=100,
        num_train_timesteps=1000,
        shift=3.0,
        sigma_max=1.0,
        sigma_min=0.003 / 1.002,
        inverse_timesteps=False,
        extra_one_step=False,
# ...
        return sample

    def set_shift(self, shift: float) -> None:
        self.shift = shift
```
**EN:** This class models `SelfForcingFlowMatchScheduler` as a specialization of `BaseScheduler`, `ConfigMixin`, `SchedulerMixin`. Important methods include `__init__`, `set_timesteps`, `step`, `add_noise`.
**CN:** 该类实现 `SelfForcingFlowMatchScheduler`，并继承/扩展 `BaseScheduler`, `ConfigMixin`, `SchedulerMixin`。 其中较重要的方法包括 `__init__`, `set_timesteps`, `step`, `add_noise`。

### Lines 140-142: Top-level configuration / 顶层配置
```python


EntryClass = SelfForcingFlowMatchScheduler
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Scheduling and batching / 调度与批处理
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `SelfForcingFlowMatchSchedulerOutput` anchors the module API / 符号 `SelfForcingFlowMatchSchedulerOutput` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.models.schedulers.base`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `diffusers.configuration_utils`, `diffusers.schedulers.scheduling_utils`, `diffusers.utils`
