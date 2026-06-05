# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/mooncake/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides shared enums, constants, and helper routines used across the disaggregation stack. It centralizes configuration-sensitive logic so multiple components can reuse it. / 该文件提供解耦栈共用的枚举、常量与辅助函数，把依赖配置的逻辑集中起来供多个组件复用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Module-level constants and helper logic
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Mooncake-specific utilities for custom memory pool management."""

import logging
from typing import Any, Optional, Tuple

import torch

from sglang.srt.environ import envs

logger = logging.getLogger(__name__)

# Global constants for custom memory pool types
SUPPORTED_MOONCAKE_CUSTOM_MEM_POOL_TYPES = ["NVLINK", "BAREX", "INTRA_NODE_NVLINK"]


```
**EN:** This block contains module-level constants, helpers, or documentation for shared disaggregation helpers and enums. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与共享解耦部署辅助函数与枚举相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 29-40: Function `init_mooncake_custom_mem_pool` signature and setup
```python
def init_mooncake_custom_mem_pool(
    device: str,
) -> Tuple[bool, Optional[Any], Optional[str]]:
    """
    Initialize custom memory pool based on environment variable.

    Args:
        device: The device to allocate memory on

    Returns:
        Tuple of (enable_custom_mem_pool, custom_mem_pool, custom_mem_pool_type)
    """
```
**EN:** This block defines the function `init_mooncake_custom_mem_pool`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `init_mooncake_custom_mem_pool`. Notable operations include `of`.
**CN:** 这一段定义了function `init_mooncake_custom_mem_pool`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `init_mooncake_custom_mem_pool`。 值得注意的操作包括 `of`。

### Lines 41-64: Function `init_mooncake_custom_mem_pool` logic (part 1)
```python
    enable_custom_mem_pool, custom_mem_pool_type = (
        check_mooncake_custom_mem_pool_enabled()
    )

    custom_mem_pool = None

    if enable_custom_mem_pool:
        try:
            # TODO(shangming): abstract custom allocator class for more backends
            if custom_mem_pool_type == "NVLINK":
                from mooncake.allocator import NVLinkAllocator

                allocator = NVLinkAllocator.get_allocator(device)
            elif custom_mem_pool_type == "BAREX":
                from mooncake.allocator import BarexAllocator

                allocator = BarexAllocator.get_allocator(device)
            elif custom_mem_pool_type == "INTRA_NODE_NVLINK":
                return False, None, None
            else:
                # This should not happen due to the enable_custom_mem_pool check above
                raise ValueError(
                    f"Unsupported custom mem pool type: {custom_mem_pool_type}"
                )
```
**EN:** This block continues `init_mooncake_custom_mem_pool` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `check_mooncake_custom_mem_pool_enabled`, `TODO`, `get_allocator`, `ValueError`.
**CN:** 这一段延续了 `init_mooncake_custom_mem_pool` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `check_mooncake_custom_mem_pool_enabled`、`TODO`、`get_allocator`、`ValueError`。

### Lines 65-87: Function `init_mooncake_custom_mem_pool` logic (part 2)
```python

            custom_mem_pool = torch.cuda.MemPool(allocator.allocator())
            logger.debug(
                f"Initialized custom memory pool: {custom_mem_pool_type} on device {device}"
            )
        except ImportError as e:
            logger.warning(
                f"Failed to import mooncake allocator for {custom_mem_pool_type}: {e}. "
                f"Falling back to default memory pool."
            )
            enable_custom_mem_pool = False
            custom_mem_pool = None
            custom_mem_pool_type = None
        except Exception as e:
            logger.error(
                f"Failed to initialize custom memory pool {custom_mem_pool_type}: {e}. "
                f"Falling back to default memory pool."
            )
            enable_custom_mem_pool = False
            custom_mem_pool = None
            custom_mem_pool_type = None
    else:
        return False, None, None
```
**EN:** This block continues `init_mooncake_custom_mem_pool` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow. Notable operations include `MemPool`, `allocator`, `debug`, `warning`.
**CN:** 这一段延续了 `init_mooncake_custom_mem_pool` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。 值得注意的操作包括 `MemPool`、`allocator`、`debug`、`warning`。

### Lines 88-89: Function `init_mooncake_custom_mem_pool` logic (part 3)
```python

    return enable_custom_mem_pool, custom_mem_pool, custom_mem_pool_type
```
**EN:** This block continues `init_mooncake_custom_mem_pool` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding shared disaggregation helpers and enums workflow.
**CN:** 这一段延续了 `init_mooncake_custom_mem_pool` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的共享解耦部署辅助函数与枚举工作流。

### Lines 92-112: Function `check_mooncake_custom_mem_pool_enabled`
```python
def check_mooncake_custom_mem_pool_enabled() -> Tuple[bool, Optional[str]]:
    """
    Check if custom memory pool is enabled without importing allocators.

    Returns:
        Tuple of (enable_custom_mem_pool, custom_mem_pool_type)
    """
    custom_mem_pool_type = envs.SGLANG_MOONCAKE_CUSTOM_MEM_POOL.get()

    if custom_mem_pool_type is not None:
        # Handle boolean True as NVLINK
        if custom_mem_pool_type.lower() == "true":
            custom_mem_pool_type = "NVLINK"
        enable_custom_mem_pool = (
            custom_mem_pool_type in SUPPORTED_MOONCAKE_CUSTOM_MEM_POOL_TYPES
        )
    else:
        enable_custom_mem_pool = False
        custom_mem_pool_type = None

    return enable_custom_mem_pool, custom_mem_pool_type
```
**EN:** This block defines the function `check_mooncake_custom_mem_pool_enabled`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `check_mooncake_custom_mem_pool_enabled`. Notable operations include `of`, `get`, `lower`.
**CN:** 这一段定义了function `check_mooncake_custom_mem_pool_enabled`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `check_mooncake_custom_mem_pool_enabled`。 值得注意的操作包括 `of`、`get`、`lower`。

## Key Concepts / 关键概念
- `init_mooncake_custom_mem_pool`: Function that performs init mooncake custom mem pool for the surrounding workflow. / `init_mooncake_custom_mem_pool`：在周边工作流中执行“初始化Mooncakecustommem池”相关任务的函数。
- `check_mooncake_custom_mem_pool_enabled`: Function that performs check mooncake custom mem pool enabled for the surrounding workflow. / `check_mooncake_custom_mem_pool_enabled`：在周边工作流中执行“checkMooncakecustommem池enabled”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **External packages / 外部依赖**: `torch`, `mooncake`
- **Internal modules / 内部模块**: `sglang.srt.environ`
