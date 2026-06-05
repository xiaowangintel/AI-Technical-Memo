# npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/npu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for npu in the multimodal generation stack. Key symbols include `device_id_to_physical_device_id`, `NPUPlatformBase`. / 该模块包含多模态生成体系中与 npu 相关的运行时支持代码。 关键符号包括 `device_id_to_physical_device_id`, `NPUPlatformBase`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm-ascend: https://github.com/vllm-project/vllm-ascend/blob/main/vllm_ascend/platform.py

import os
from typing import Any

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

### Lines 21-33: Function `device_id_to_physical_device_id` / 函数 `device_id_to_physical_device_id`
```python
def device_id_to_physical_device_id(device_id: int) -> int:
    if "ASCEND_RT_VISIBLE_DEVICES" in os.environ:
        device_ids = os.environ["ASCEND_RT_VISIBLE_DEVICES"].split(",")
        if device_ids == [""]:
            msg = (
                "ASCEND_RT_VISIBLE_DEVICES is set to empty string, which means"
                " NPU support is disabled"
            )
            raise RuntimeError(msg)
        physical_device_id = device_ids[device_id]
        return int(physical_device_id)
    else:
        return device_id
```
**EN:** This function drives `device_id_to_physical_device_id` with inputs such as `device_id`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `device_id_to_physical_device_id`，主要处理 `device_id` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 36-135: Class `NPUPlatformBase` / 类 `NPUPlatformBase`
```python
class NPUPlatformBase(Platform):
    _enum = PlatformEnum.NPU
    device_name: str = "npu"
    device_type: str = "npu"
    dispatch_key: str = "NPU"
    device_control_env_var: str = "ASCEND_RT_VISIBLE_DEVICES"

    @classmethod
    def get_local_torch_device(cls) -> torch.device:
        return torch.device(f"npu:{envs.LOCAL_RANK}")

    @classmethod
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability:
        return None
# ...
    @classmethod
    def enable_dit_layerwise_offload_for_wan_by_default(cls) -> bool:
        """The performance of the layerwise_offload feature depends on the device's memory size and the memory size occupied by the model. Use --dit-layerwise-offload True if it suitable for your case."""
        return False
```
**EN:** This class models `NPUPlatformBase` as a specialization of `Platform`. Important methods include `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`.
**CN:** 该类实现 `NPUPlatformBase`，并继承/扩展 `Platform`。 其中较重要的方法包括 `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Symbol `device_id_to_physical_device_id` anchors the module API / 符号 `device_id_to_physical_device_id` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `torch.distributed`
- **Stdlib / 标准库**: `os`, `typing`
