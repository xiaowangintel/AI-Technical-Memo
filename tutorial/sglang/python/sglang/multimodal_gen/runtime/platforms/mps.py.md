# mps.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/mps.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for mps in the multimodal generation stack. Key symbols include `MpsPlatform`. / 该模块包含多模态生成体系中与 mps 相关的运行时支持代码。 关键符号包括 `MpsPlatform`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo
from functools import lru_cache
from typing import Any

import psutil
import torch

from sglang.multimodal_gen.runtime.platforms import (
    AttentionBackendEnum,
    Platform,
    PlatformEnum,
)
from sglang.multimodal_gen.runtime.platforms.interface import DeviceCapability
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

# SPDX-License-Identifier: Apache-2.0


logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 22-130: Class `MpsPlatform` / 类 `MpsPlatform`
```python
class MpsPlatform(Platform):
    _enum = PlatformEnum.MPS
    device_name: str = "mps"
    device_type: str = "mps"
    dispatch_key: str = "MPS"
    device_control_env_var: str = "MPS_VISIBLE_DEVICES"

    @classmethod
    @lru_cache(maxsize=1)
    def is_amp_supported(cls) -> bool:
        return False

    @classmethod
    @lru_cache(maxsize=1)
# ...

            random.seed(seed)
            np.random.seed(seed)
            torch.manual_seed(seed)
```
**EN:** This class models `MpsPlatform` as a specialization of `Platform`. Important methods include `is_amp_supported`, `is_float64_supported`, `get_local_torch_device`, `get_device_capability`.
**CN:** 该类实现 `MpsPlatform`，并继承/扩展 `Platform`。 其中较重要的方法包括 `is_amp_supported`, `is_float64_supported`, `get_local_torch_device`, `get_device_capability`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `psutil`, `torch`, `torch.distributed`, `numpy`
- **Stdlib / 标准库**: `functools`, `typing`, `random`
