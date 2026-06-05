# comfyui_latent_preparation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/comfyui_latent_preparation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for comfyui latent preparation, connecting stages, tensors, and runtime metadata. Key symbols include `ComfyUILatentPreparationStage`. / 该模块负责 comfyui latent preparation 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `ComfyUILatentPreparationStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
"""
ComfyUI latent preparation stage with device mismatch fix.
This stage extends LatentPreparationStage to handle device mismatch issues
that occur when tensors are pickled and unpickled via broadcast_pyobj in
multi-GPU scenarios.
"""

import dataclasses

import torch

from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
# ...
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 26-114: Class `ComfyUILatentPreparationStage` / 类 `ComfyUILatentPreparationStage`
```python
class ComfyUILatentPreparationStage(LatentPreparationStage):
    """
    ComfyUI-specific latent preparation stage with device mismatch fix.

    This stage extends LatentPreparationStage to automatically fix device
    mismatches for tensor fields on non-source ranks in multi-GPU scenarios.
    """

    @staticmethod
    def _fix_tensor_device(value, target_device):
        """Recursively fix tensor device, handling single tensors, lists, and tuples."""
        if isinstance(value, torch.Tensor):
            if value.device != target_device:
                return value.detach().clone().to(target_device)
# ...
            # (e.g., 4D spatial -> 3D sequence) to ensure proper unpadding later.
            result.raw_latent_shape = original_latents_shape

        return result
```
**EN:** This class models `ComfyUILatentPreparationStage` as a specialization of `LatentPreparationStage`. ComfyUI-specific latent preparation stage with device mismatch fix. Important methods include `_fix_tensor_device`, `_has_tensor`, `forward`.
**CN:** 该类实现 `ComfyUILatentPreparationStage`，并继承/扩展 `LatentPreparationStage`。 文档字符串指出：ComfyUI-specific latent preparation stage with device mismatch fix. 其中较重要的方法包括 `_fix_tensor_device`, `_has_tensor`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Symbol `ComfyUILatentPreparationStage` anchors the module API / 符号 `ComfyUILatentPreparationStage` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.pipelines_core.stages.latent_preparation`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `dataclasses`
