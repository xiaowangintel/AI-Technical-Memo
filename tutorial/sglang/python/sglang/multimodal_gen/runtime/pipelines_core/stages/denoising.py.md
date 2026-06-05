# denoising.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/denoising.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for denoising, connecting stages, tensors, and runtime metadata. Key symbols include `DenoisingContext`, `DenoisingStepState`, `DenoisingStage`. / 该模块负责 denoising 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `DenoisingContext`, `DenoisingStepState`, `DenoisingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-106: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Denoising stage for diffusion pipelines.
"""

import inspect
import math
import os
import time
import weakref
from collections.abc import Callable, Iterable
from dataclasses import dataclass, field, fields
# ...
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE, dict_to_3d_list
from sglang.srt.utils.common import get_compiler_backend

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 109-143: Class `DenoisingContext` / 类 `DenoisingContext`
```python
@dataclass(slots=True)
class DenoisingContext:
    """Loop-scoped state shared across the denoising skeleton and its hooks."""

    scheduler: Any
    extra_step_kwargs: dict[str, Any]
    target_dtype: torch.dtype
    autocast_enabled: bool
    timesteps: torch.Tensor
    num_inference_steps: int
    num_warmup_steps: int
    image_kwargs: dict[str, Any]
    pos_cond_kwargs: dict[str, Any]
    neg_cond_kwargs: dict[str, Any]
# ...

    def to_kwargs(self) -> dict[str, Any]:
        """Return a shallow field mapping for derived context construction."""
        return {item.name: getattr(self, item.name) for item in fields(self)}
```
**EN:** This class models `DenoisingContext`. Loop-scoped state shared across the denoising skeleton and its hooks. Important methods include `__getitem__`, `get`, `to_kwargs`.
**CN:** 该类实现 `DenoisingContext`。 文档字符串指出：Loop-scoped state shared across the denoising skeleton and its hooks. 其中较重要的方法包括 `__getitem__`, `get`, `to_kwargs`。

### Lines 146-156: Class `DenoisingStepState` / 类 `DenoisingStepState`
```python
@dataclass(slots=True)
class DenoisingStepState:
    """Per-step hot-path state computed once and reused within a denoising step."""

    step_index: int
    t_host: torch.Tensor
    t_device: torch.Tensor
    t_int: int
    current_model: Any
    current_guidance_scale: Any
    attn_metadata: Any | None
```
**EN:** This class models `DenoisingStepState`. Per-step hot-path state computed once and reused within a denoising step.
**CN:** 该类实现 `DenoisingStepState`。 文档字符串指出：Per-step hot-path state computed once and reused within a denoising step.

### Lines 159-1800: Class `DenoisingStage` / 类 `DenoisingStage`
```python
class DenoisingStage(PipelineStage, RolloutDenoisingMixin):
    """
    Stage for running the denoising loop in diffusion pipelines.

    This stage handles the iterative denoising process that transforms
    the initial noise into the final output.
    """

    @property
    def role_affinity(self):
        return RoleType.DENOISER

    def __init__(
        self, transformer, scheduler, pipeline=None, transformer_2=None, vae=None
# ...
        """Verify denoising stage outputs."""
        result = VerificationResult()
        # result.add_check("latents", batch.latents, [V.is_tensor, V.with_dims(5)])
        return result
```
**EN:** This class models `DenoisingStage` as a specialization of `PipelineStage`, `RolloutDenoisingMixin`. Stage for running the denoising loop in diffusion pipelines. Important methods include `role_affinity`, `__init__`, `_infer_transformer_attention_backend`, `component_uses`.
**CN:** 该类实现 `DenoisingStage`，并继承/扩展 `PipelineStage`, `RolloutDenoisingMixin`。 文档字符串指出：Stage for running the denoising loop in diffusion pipelines. 其中较重要的方法包括 `role_affinity`, `__init__`, `_infer_transformer_attention_backend`, `component_uses`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.jit_kernel.nvfp4`, `sglang.multimodal_gen`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.flux`, `sglang.multimodal_gen.configs.pipeline_configs.zimage`, `sglang.multimodal_gen.runtime.cache.cache_dit_integration`, `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.distributed`
- **External / 外部**: `torch`, `torch.nn`, `tqdm.auto`, `torch._inductor.config`
- **Stdlib / 标准库**: `inspect`, `math`, `os`, `time`, `weakref`, `collections.abc`, `dataclasses`, `functools`
