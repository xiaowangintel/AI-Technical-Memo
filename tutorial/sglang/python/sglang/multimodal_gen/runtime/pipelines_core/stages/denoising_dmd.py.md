# denoising_dmd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/denoising_dmd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for denoising dmd, connecting stages, tensors, and runtime metadata. Key symbols include `DmdDenoisingStage`. / 该模块负责 denoising dmd 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `DmdDenoisingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

import time

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.forward_context import set_forward_context
from sglang.multimodal_gen.runtime.models.schedulers.scheduling_flow_match_euler_discrete import (
    FlowMatchEulerDiscreteScheduler,
)
from sglang.multimodal_gen.runtime.models.utils import pred_noise_to_pred_video
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages import DenoisingStage
# ...
from sglang.multimodal_gen.runtime.utils.perf_logger import StageProfiler
from sglang.multimodal_gen.utils import dict_to_3d_list

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 24-277: Class `DmdDenoisingStage` / 类 `DmdDenoisingStage`
```python
class DmdDenoisingStage(DenoisingStage):
    """
    Denoising stage for DMD.
    """

    def __init__(self, transformer, scheduler, transformer_2=None) -> None:
        super().__init__(
            transformer=transformer, scheduler=scheduler, transformer_2=transformer_2
        )
        self.scheduler = FlowMatchEulerDiscreteScheduler(shift=8.0)

    def _preprocess_sp_latents(self, batch: Req, server_args: ServerArgs):
        # 1. to shard latents (B, C, T, H, W) along dim 2
        super()._preprocess_sp_latents(batch, server_args)
# ...
        else:
            boundary_timestep = None

        return boundary_timestep
```
**EN:** This class models `DmdDenoisingStage` as a specialization of `DenoisingStage`. Denoising stage for DMD. Important methods include `__init__`, `_preprocess_sp_latents`, `_postprocess_sp_latents`, `forward`.
**CN:** 该类实现 `DmdDenoisingStage`，并继承/扩展 `DenoisingStage`。 文档字符串指出：Denoising stage for DMD. 其中较重要的方法包括 `__init__`, `_preprocess_sp_latents`, `_postprocess_sp_latents`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Distributed execution / 分布式执行
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.models.schedulers.scheduling_flow_match_euler_discrete`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `time`
