# platform_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/platform_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import multiprocessing
from collections.abc import Sequence
from concurrent.futures.process import ProcessPoolExecutor
from functools import cache
from typing import Any

import torch
```
**EN:** Sets up the module with standard-library support such as `multiprocessing`, `collections.abc`, `concurrent.futures.process`, external packages such as `torch`, vLLM modules such as `vllm.platforms`. It prepares the symbols later used by `cuda_is_initialized`, `xpu_is_initialized`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.platforms` 等 vLLM 内部依赖。 这些准备工作为后续的 `cuda_is_initialized`, `xpu_is_initialized` 提供上下文。

### cuda_is_initialized (lines 13-17)
```python
def cuda_is_initialized() -> bool:
    """Check if CUDA is initialized."""
    if not torch.cuda._is_compiled():
        return False
    return torch.cuda.is_initialized()
```
**EN:** `cuda_is_initialized`: Check if CUDA is initialized. Inside the body, it relies on `torch.cuda.is_initialized`, `torch.cuda._is_compiled` to complete the main steps.
**CN:** `cuda_is_initialized` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `torch.cuda.is_initialized`, `torch.cuda._is_compiled` 等函数完成关键步骤。

### xpu_is_initialized (lines 20-24)
```python
def xpu_is_initialized() -> bool:
    """Check if XPU is initialized."""
    if not torch.xpu._is_compiled():
        return False
    return torch.xpu.is_initialized()
```
**EN:** `xpu_is_initialized`: Check if XPU is initialized. Inside the body, it relies on `torch.xpu.is_initialized`, `torch.xpu._is_compiled` to complete the main steps.
**CN:** `xpu_is_initialized` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `torch.xpu.is_initialized`, `torch.xpu._is_compiled` 等函数完成关键步骤。

### cuda_get_device_properties (lines 27-39)
```python
def cuda_get_device_properties(
    device, names: Sequence[str], init_cuda=False
) -> tuple[Any, ...]:
    """Get specified CUDA device property values without initializing CUDA in
    the current process."""
    if init_cuda or cuda_is_initialized():
        props = torch.cuda.get_device_properties(device)
        return tuple(getattr(props, name) for name in names)

    # Run in subprocess to avoid initializing CUDA as a side effect.
    mp_ctx = multiprocessing.get_context("fork")
    with ProcessPoolExecutor(max_workers=1, mp_context=mp_ctx) as executor:
        return executor.submit(cuda_get_device_properties, device, names, True).result()
```
**EN:** `cuda_get_device_properties`: Get specified CUDA device property values without initializing CUDA in the current process. It mainly works with `device`, `names`, `init_cuda`. Inside the body, it relies on `multiprocessing.get_context`, `cuda_is_initialized`, `torch.cuda.get_device_properties` to complete the main steps.
**CN:** `cuda_get_device_properties` 负责实现本模块使用的辅助逻辑。 它主要处理 `device`, `names`, `init_cuda` 等参数。 实现过程中会调用 `multiprocessing.get_context`, `cuda_is_initialized`, `torch.cuda.get_device_properties` 等函数完成关键步骤。

### is_pin_memory_available (lines 43-46)
```python
def is_pin_memory_available() -> bool:
    from vllm.platforms import current_platform

    return current_platform.is_pin_memory_available()
```
**EN:** `is_pin_memory_available` checks a condition and returns a boolean-style result. Inside the body, it relies on `current_platform.is_pin_memory_available` to complete the main steps.
**CN:** `is_pin_memory_available` 负责检查条件并返回布尔结果。 实现过程中会调用 `current_platform.is_pin_memory_available` 等函数完成关键步骤。

### is_uva_available (lines 50-54)
```python
def is_uva_available() -> bool:
    """Check if Unified Virtual Addressing (UVA) is available."""
    # UVA requires pinned memory.
    # TODO: Add more requirements for UVA if needed.
    return is_pin_memory_available()
```
**EN:** `is_uva_available`: Check if Unified Virtual Addressing (UVA) is available. Inside the body, it relies on `is_pin_memory_available` to complete the main steps.
**CN:** `is_uva_available` 负责检查条件并返回布尔结果。 实现过程中会调用 `is_pin_memory_available` 等函数完成关键步骤。

### num_compute_units (lines 58-62)
```python
def num_compute_units(device_id: int = 0) -> int:
    """Get the number of compute units of the current device."""
    from vllm.platforms import current_platform

    return current_platform.num_compute_units(device_id)
```
**EN:** `num_compute_units`: Get the number of compute units of the current device. It mainly works with `device_id`. Inside the body, it relies on `current_platform.num_compute_units` to complete the main steps.
**CN:** `num_compute_units` 负责实现本模块使用的辅助逻辑。 它主要处理 `device_id` 等参数。 实现过程中会调用 `current_platform.num_compute_units` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`cuda_is_initialized`**: Key helper or entry point in this file. / **`cuda_is_initialized`**：本文件中的关键辅助函数或入口。
- **`xpu_is_initialized`**: Key helper or entry point in this file. / **`xpu_is_initialized`**：本文件中的关键辅助函数或入口。
- **`cuda_get_device_properties`**: Key helper or entry point in this file. / **`cuda_get_device_properties`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: multiprocessing, collections.abc, concurrent.futures.process, functools, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.platforms
