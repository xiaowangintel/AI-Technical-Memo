# xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/xpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for xpu in the multimodal generation stack. Key symbols include `XpuPlatform`. / 该模块包含多模态生成体系中与 xpu 相关的运行时支持代码。 关键符号包括 `XpuPlatform`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
# Intel XPU Platform support for SGLang Diffusion

import torch

from sglang.multimodal_gen import envs
from sglang.multimodal_gen.runtime.platforms.interface import (
    AttentionBackendEnum,
    DeviceCapability,
    Platform,
    PlatformEnum,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 18-196: Class `XpuPlatform` / 类 `XpuPlatform`
```python
class XpuPlatform(Platform):
    """Platform implementation for Intel XPU (Data Center GPU Max, Arc, etc.)."""

    _enum = PlatformEnum.XPU
    device_name: str = "xpu"
    device_type: str = "xpu"
    dispatch_key: str = "XPU"
    device_control_env_var: str = "ZE_AFFINITY_MASK"

    @classmethod
    def get_local_torch_device(cls) -> torch.device:
        return torch.device(f"xpu:{envs.LOCAL_RANK}")

    @classmethod
# ...
    def get_device_communicator_cls(cls) -> str:
        """Get device communicator class for Intel XPU distributed communication."""
        # Use base communicator for now; can be updated to use oneCCL-based communicator
        return "sglang.multimodal_gen.runtime.distributed.device_communicators.base_device_communicator.DeviceCommunicatorBase"
```
**EN:** This class models `XpuPlatform` as a specialization of `Platform`. Platform implementation for Intel XPU (Data Center GPU Max, Arc, etc.). Important methods include `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_uuid`.
**CN:** 该类实现 `XpuPlatform`，并继承/扩展 `Platform`。 文档字符串指出：Platform implementation for Intel XPU (Data Center GPU Max, Arc, etc.). 其中较重要的方法包括 `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_uuid`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.layers.attention.backends.xpu_backend`
- **External / 外部**: `torch`, `torch.distributed`
