# decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for decoding, connecting stages, tensors, and runtime metadata. Key symbols include `_ensure_tensor_decode_output`, `DecodingStage`. / 该模块负责 decoding 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `_ensure_tensor_decode_output`, `DecodingStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Decoding stage for diffusion pipelines.
"""

import weakref

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.vae_loader import VAELoader
from sglang.multimodal_gen.runtime.managers.memory_managers.component_manager import (
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 34-51: Function `_ensure_tensor_decode_output` / 函数 `_ensure_tensor_decode_output`
```python
def _ensure_tensor_decode_output(decode_output):
    """
    Ensure VAE decode output is a tensor.

    Some VAE implementations return DecoderOutput objects with a .sample attribute,
    tuples, or tensors directly. This function normalizes the output to always be a tensor.

    Args:
        decode_output: Output from VAE.decode(), can be DecoderOutput, tuple, or torch.Tensor

    Returns:
        torch.Tensor: The decoded image tensor
    """
    if isinstance(decode_output, tuple):
        return decode_output[0]
    if hasattr(decode_output, "sample"):
        return decode_output.sample
    return decode_output
```
**EN:** This function drives `_ensure_tensor_decode_output` with inputs such as `decode_output`. Ensure VAE decode output is a tensor.
**CN:** 这个函数负责 `_ensure_tensor_decode_output`，主要处理 `decode_output` 等输入。 文档字符串说明：Ensure VAE decode output is a tensor.

### Lines 54-266: Class `DecodingStage` / 类 `DecodingStage`
```python
class DecodingStage(PipelineStage):
    """
    Stage for decoding latent representations into pixel space.

    This stage handles the decoding of latent representations into the final
    output format (e.g., pixel values).
    """

    @property
    def role_affinity(self):
        from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType

        return RoleType.DECODER

# ...
            noise_pred=None,
        )

        return output_batch
```
**EN:** This class models `DecodingStage` as a specialization of `PipelineStage`. Stage for decoding latent representations into pixel space. Important methods include `role_affinity`, `__init__`, `component_uses`, `parallelism_type`.
**CN:** 该类实现 `DecodingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for decoding latent representations into pixel space. 其中较重要的方法包括 `role_affinity`, `__init__`, `component_uses`, `parallelism_type`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Request disaggregation / 请求解耦

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.vae_loader`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.models.vaes.common`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `weakref`
