# zen_cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/platforms/zen_cpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines platform-specific behavior and capability checks for hardware backends. / 定义硬件后端的平台特定行为与能力检查逻辑。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-9)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import torch

from vllm.logger import init_logger
from vllm.platforms.cpu import CpuPlatform

logger = init_logger(__name__)
```
**EN:** Sets up the module with external packages such as `torch`, vLLM modules such as `vllm.logger`, `vllm.platforms.cpu`. It prepares the symbols later used by `ZenCpuPlatform`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.platforms.cpu` 等 vLLM 内部依赖。 这些准备工作为后续的 `ZenCpuPlatform` 提供上下文。

### ZenCpuPlatform (lines 12-32)
```python
class ZenCpuPlatform(CpuPlatform):
    """CPU platform with AMD Zen (ZenDNN/zentorch) optimizations.

    Model-load time (dispatch_cpu_unquantized_gemm in layers/utils.py):
      - Routes linear ops to zentorch_linear_unary.
      - When VLLM_ZENTORCH_WEIGHT_PREPACK=1 (default), eagerly prepacks
        weights via zentorch_weight_prepack_for_linear.
    """

    device_name: str = "cpu"
    device_type: str = "cpu"

    def is_zen_cpu(self) -> bool:
        # is_cpu() also returns True for this platform (inherited from CpuPlatform).
        return True

    # Currently, AMD CPUs do not support float16 compute.
    # Hence explicitly return bfloat16 and float32.
    @property
    def supported_dtypes(self) -> list[torch.dtype]:
        return [torch.bfloat16, torch.float32]
```
**EN:** `ZenCpuPlatform`: CPU platform with AMD Zen (ZenDNN/zentorch) optimizations. It extends `CpuPlatform`. Key methods include `is_zen_cpu`, `supported_dtypes`.
**CN:** `ZenCpuPlatform` 是该文件中的核心类，用于封装与 `ZenCpuPlatform` 相关的状态和行为。 它继承自 `CpuPlatform`。 关键方法包括 `is_zen_cpu`, `supported_dtypes`。

## Key Concepts / 关键概念
- **`ZenCpuPlatform`**: Core class that organizes module behavior. / **`ZenCpuPlatform`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.platforms.cpu
