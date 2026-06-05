# causal_denoising.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/causal_denoising.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for causal denoising, connecting stages, tensors, and runtime metadata. Key symbols include `CausalDMDDenoisingStage`. / 该模块负责 causal denoising 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `CausalDMDDenoisingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

import torch  # type: ignore

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.forward_context import set_forward_context
from sglang.multimodal_gen.runtime.models.utils import pred_noise_to_pred_video
from sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils import (
    get_or_create_request_scheduler,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.denoising import DenoisingStage
from sglang.multimodal_gen.runtime.pipelines_core.stages.validators import (
    StageValidators as V,
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 29-503: Class `CausalDMDDenoisingStage` / 类 `CausalDMDDenoisingStage`
```python
class CausalDMDDenoisingStage(DenoisingStage):
    """
    Denoising stage for causal diffusion.
    """

    def __init__(self, transformer, scheduler) -> None:
        super().__init__(transformer, scheduler)
        # KV and cross-attention cache state (initialized on first forward)
        self.kv_cache1: list | None = None
        self.crossattn_cache: list | None = None
        # Model-dependent constants (aligned with causal_inference.py assumptions)
        self.num_transformer_blocks = self.transformer.config.arch_config.num_layers
        self.num_frames_per_block = (
            self.transformer.config.arch_config.num_frames_per_block
# ...
            batch.negative_prompt_embeds,
            lambda x: not batch.do_classifier_free_guidance or V.list_not_empty(x),
        )
        return result
```
**EN:** This class models `CausalDMDDenoisingStage` as a specialization of `DenoisingStage`. Denoising stage for causal diffusion. Important methods include `__init__`, `forward`, `_initialize_kv_cache`, `_initialize_crossattn_cache`.
**CN:** 该类实现 `CausalDMDDenoisingStage`，并继承/扩展 `DenoisingStage`。 文档字符串指出：Denoising stage for causal diffusion. 其中较重要的方法包括 `__init__`, `forward`, `_initialize_kv_cache`, `_initialize_crossattn_cache`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.denoising`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`
