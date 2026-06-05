# rocm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/rocm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for rocm in the multimodal generation stack. Key symbols include `RocmPlatform`. / 该模块包含多模态生成体系中与 rocm 相关的运行时支持代码。 关键符号包括 `RocmPlatform`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from rocm/vllm: https://github.com/ROCm/vllm/blob/v0.7.3%2Brocm/vllm/platforms/rocm.py
"""
This file is a platform abstraction for ROCm GPUs,
adjusted to match the structure and interface of `cuda.py`.
"""

import types
from functools import lru_cache
from typing import Any

import torch
# ...
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-399: Class `RocmPlatform` / 类 `RocmPlatform`
```python
class RocmPlatform(Platform):
    _enum = PlatformEnum.ROCM
    device_name: str = "rocm"
    device_type: str = "cuda"  # torch uses 'cuda' backend string
    dispatch_key: str = "CUDA"
    device_control_env_var: str = "CUDA_VISIBLE_DEVICES"

    @classmethod
    def get_local_torch_device(cls) -> torch.device:
        return torch.device(f"cuda:{envs.LOCAL_RANK}")

    @classmethod
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability:
        major, minor = torch.cuda.get_device_capability(device_id)
# ...
    @classmethod
    def enable_dit_layerwise_offload_for_wan_by_default(cls) -> bool:
        """ROCm performs better without DIT layerwise offload on Wan."""
        return False
```
**EN:** This class models `RocmPlatform` as a specialization of `Platform`. Important methods include `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`.
**CN:** 该类实现 `RocmPlatform`，并继承/扩展 `Platform`。 其中较重要的方法包括 `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.envs`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.jit_kernel.flash_attention_v3`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`
- **External / 外部**: `torch`, `torch.nn`, `torch.nn.functional`, `torch.distributed`, `flash_attn`, `aiter.ops.groupnorm`
- **Stdlib / 标准库**: `types`, `functools`, `typing`
