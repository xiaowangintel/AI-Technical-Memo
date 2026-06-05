# rocm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/platforms/rocm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines platform-specific behavior and capability checks for hardware backends. / 定义硬件后端的平台特定行为与能力检查逻辑。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-74)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import os
from datetime import timedelta
from functools import cache, lru_cache, wraps
from typing import TYPE_CHECKING

import regex as re
import torch
from torch.distributed import PrefixStore, ProcessGroup
from torch.distributed.distributed_c10d import is_nccl_available

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .interface import DeviceCapability, Platform, PlatformEnum

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.config.kernel import IrOpPriorityConfig
    from vllm.v1.attention.selector import AttentionSelectorConfig

    # ...
    "0x150e": "AMD_Radeon_890M",  # gfx1150, Strix Point
    "0x1586": "AMD_Radeon_8060S",  # gfx1151, Strix Halo
    # RDNA 4 discrete (Navi 48)
    "0x7550": "AMD_Radeon_RX9070XT",  # gfx1201
    "0x7551": "AMD_Radeon_R9700",  # gfx1201
}
```
**EN:** Sets up the module with standard-library support such as `os`, `datetime`, `functools`, external packages such as `regex`, `torch`, `torch.distributed`, vLLM modules such as `vllm.envs`, `vllm.logger`, `vllm.v1.attention.backends.registry`. It prepares the symbols later used by `RocmPlatform`, `_rocm_device_count_stateless`, `_sync_hip_cuda_env_vars`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger`, `vllm.v1.attention.backends.registry` 等 vLLM 内部依赖。 这些准备工作为后续的 `RocmPlatform`, `_rocm_device_count_stateless`, `_sync_hip_cuda_env_vars` 提供上下文。

### _rocm_device_count_stateless (lines 78-106)
```python
def _rocm_device_count_stateless(cuda_visible_devices: str | None = None) -> int:
    """Get number of ROCm devices, caching based on the value of CUDA_VISIBLE_DEVICES
    at the time of call.

    This should be used instead of torch.accelerator.device_count() unless
    CUDA_VISIBLE_DEVICES has already been set to the desired value.

    # This can be removed and simply replaced with torch.cuda.get_device_count
    # after https://github.com/pytorch/pytorch/pull/122815 is released."""
    # Note: cuda_visible_devices is not used, but we keep it as an argument for
    # LRU Cache purposes.

    # Code below is based on
    # https://github.com/pytorch/pytorch/blob/
    # c1cd946818442aca8c7f812b16d187ce1586c3bc/
    # torch/cuda/__init__.py#L831C1-L831C17
    import torch.cuda

    if not torch.cuda._is_compiled():
        return 0
    # ROCm uses amdsmi instead of nvml for stateless device count
    # This requires a sufficiently modern version of Torch 2.4.0
    raw_count = (
        torch.cuda._device_count_amdsmi()
        if (hasattr(torch.cuda, "_device_count_amdsmi"))
        else -1
    )
    r = torch._C._cuda_getDeviceCount() if raw_count < 0 else raw_count
    return r
```
**EN:** `_rocm_device_count_stateless`: Get number of ROCm devices, caching based on the value of CUDA_VISIBLE_DEVICES at the time of call. It mainly works with `cuda_visible_devices`. Inside the body, it relies on `lru_cache`, `torch.cuda._is_compiled`, `torch.cuda._device_count_amdsmi` to complete the main steps.
**CN:** `_rocm_device_count_stateless` 负责实现本模块使用的辅助逻辑。 它主要处理 `cuda_visible_devices` 等参数。 实现过程中会调用 `lru_cache`, `torch.cuda._is_compiled`, `torch.cuda._device_count_amdsmi` 等函数完成关键步骤。

### _sync_hip_cuda_env_vars (lines 109-126)
```python
def _sync_hip_cuda_env_vars():
    """Ensure HIP_VISIBLE_DEVICES and CUDA_VISIBLE_DEVICES are consistent.
    Treats empty string as unset. Raises on genuine conflicts."""
    hip_val = os.environ.get("HIP_VISIBLE_DEVICES") or None
    cuda_val = os.environ.get("CUDA_VISIBLE_DEVICES") or None

    if hip_val is not None and cuda_val is not None:
        if hip_val != cuda_val:
            raise ValueError(
                f"Inconsistent GPU visibility env vars: "
                f"HIP_VISIBLE_DEVICES='{hip_val}' vs "
                f"CUDA_VISIBLE_DEVICES='{cuda_val}'. "
                f"Please set only one, or ensure they match."
            )
    elif hip_val is not None:
        os.environ["CUDA_VISIBLE_DEVICES"] = hip_val
    elif cuda_val is not None:
        os.environ["HIP_VISIBLE_DEVICES"] = cuda_val
```
**EN:** `_sync_hip_cuda_env_vars`: Ensure HIP_VISIBLE_DEVICES and CUDA_VISIBLE_DEVICES are consistent. Inside the body, it relies on `os.environ.get`, `ValueError` to complete the main steps.
**CN:** `_sync_hip_cuda_env_vars` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.environ.get`, `ValueError` 等函数完成关键步骤。

### with_amdsmi_context (lines 138-147)
```python
def with_amdsmi_context(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        amdsmi_init()
        try:
            return fn(*args, **kwargs)
        finally:
            amdsmi_shut_down()

    return wrapper
```
**EN:** `with_amdsmi_context` implements helper logic used by this module. It mainly works with `fn`. Inside the body, it relies on `wraps`, `amdsmi_init`, `fn` to complete the main steps.
**CN:** `with_amdsmi_context` 负责实现本模块使用的辅助逻辑。 它主要处理 `fn` 等参数。 实现过程中会调用 `wraps`, `amdsmi_init`, `fn` 等函数完成关键步骤。

### _query_gcn_arch_from_amdsmi (lines 151-161)
```python
def _query_gcn_arch_from_amdsmi() -> str:
    """Query GCN arch from amdsmi. Raises if not available."""
    handles = amdsmi_get_processor_handles()
    if handles:
        asic_info = amdsmi_get_gpu_asic_info(handles[0])
        # Use target_graphics_version which contains the gfx name
        # e.g., 'gfx942' for MI300X/MI325X
        target_gfx = asic_info.get("target_graphics_version", "")
        if target_gfx:
            return target_gfx
    raise RuntimeError("amdsmi did not return valid GCN arch")
```
**EN:** `_query_gcn_arch_from_amdsmi`: Query GCN arch from amdsmi. Inside the body, it relies on `amdsmi_get_processor_handles`, `RuntimeError`, `amdsmi_get_gpu_asic_info` to complete the main steps.
**CN:** `_query_gcn_arch_from_amdsmi` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `amdsmi_get_processor_handles`, `RuntimeError`, `amdsmi_get_gpu_asic_info` 等函数完成关键步骤。

### _get_gcn_arch (lines 164-179)
```python
def _get_gcn_arch() -> str:
    """
    Get GCN arch via amdsmi (no CUDA init), fallback to torch.cuda.
    Called once at module level; result stored in _GCN_ARCH.
    """
    try:
        return _query_gcn_arch_from_amdsmi()
    except Exception as e:
        logger.debug("Failed to get GCN arch via amdsmi: %s", e)
        logger.warning_once(
            "Failed to get GCN arch via amdsmi, falling back to torch.cuda. "
            "This will initialize CUDA and may cause "
            "issues if CUDA_VISIBLE_DEVICES is not set yet."
        )
    # Ultimate fallback: use torch.cuda (will initialize CUDA)
    return torch.cuda.get_device_properties("cuda").gcnArchName
```
**EN:** `_get_gcn_arch`: Get GCN arch via amdsmi (no CUDA init), fallback to torch.cuda. Inside the body, it relies on `_query_gcn_arch_from_amdsmi`, `torch.cuda.get_device_properties`, `logger.debug` to complete the main steps.
**CN:** `_get_gcn_arch` 负责获取流水线所需的数据或状态。 实现过程中会调用 `_query_gcn_arch_from_amdsmi`, `torch.cuda.get_device_properties`, `logger.debug` 等函数完成关键步骤。

### _capability_from_gcn_arch (lines 196-264)
```python
def _capability_from_gcn_arch(gcn_arch: str) -> tuple[int, int] | None:
    """
    Parse (major, minor) from a GCN arch string, mirroring how
    HIP derives hipDeviceProp_t.major / .minor.

    Format: gfx<MAJOR><MINOR><STEPPING>
      - 1-digit major  (gfx9xx):  "gfx" + M + m + stepping
      - 2-digit major  (gfx1xxx): "gfx" + MM + m + stepping

    Examples:
      gfx90a  -> (9, 0)    gfx942  -> (9, 4)    gfx950 -> (9, 5)
      gfx1100 -> (11, 0)   gfx1101 -> (11, 0)   gfx1200 -> (12, 0)

    Returns None only when the string is not gfx-prefixed at all
    (i.e. not a ROCm arch string). Raises on any string that looks
    like a GCN arch but does not match a known layout.
    """
    m = re.match(r"gfx(\d+)", gcn_arch)
    if not m:
        # Not a gfx string at all — caller should fall back to torch.cuda
        return None

    digits = m.group(1)
    n = len(digits)
    # ...
            f"Major version > 12 is beyond currently known AMD generations. "
            f"Please file a vLLM issue with your GPU model so support "
            f"can be added."
        )

    return (major, minor)
```
**EN:** `_capability_from_gcn_arch`: Parse (major, minor) from a GCN arch string, mirroring how HIP derives hipDeviceProp_t.major / .minor. It mainly works with `gcn_arch`. Inside the body, it relies on `re.match`, `m.group`, `ValueError` to complete the main steps.
**CN:** `_capability_from_gcn_arch` 负责实现本模块使用的辅助逻辑。 它主要处理 `gcn_arch` 等参数。 实现过程中会调用 `re.match`, `m.group`, `ValueError` 等函数完成关键步骤。

### on_gfx1x (lines 267-268)
```python
def on_gfx1x() -> bool:
    return _ON_GFX1X
```
**EN:** `on_gfx1x` implements helper logic used by this module.
**CN:** `on_gfx1x` 负责实现本模块使用的辅助逻辑。

### on_gfx12x (lines 271-272)
```python
def on_gfx12x() -> bool:
    return _ON_GFX12X
```
**EN:** `on_gfx12x` implements helper logic used by this module.
**CN:** `on_gfx12x` 负责实现本模块使用的辅助逻辑。

### on_mi3xx (lines 275-276)
```python
def on_mi3xx() -> bool:
    return _ON_MI3XX
```
**EN:** `on_mi3xx` implements helper logic used by this module.
**CN:** `on_mi3xx` 负责实现本模块使用的辅助逻辑。

### on_gfx9 (lines 279-280)
```python
def on_gfx9() -> bool:
    return _ON_GFX9
```
**EN:** `on_gfx9` implements helper logic used by this module.
**CN:** `on_gfx9` 负责实现本模块使用的辅助逻辑。

### on_gfx90a (lines 283-284)
```python
def on_gfx90a() -> bool:
    return _ON_GFX90A
```
**EN:** `on_gfx90a` implements helper logic used by this module.
**CN:** `on_gfx90a` 负责实现本模块使用的辅助逻辑。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`RocmPlatform`**: Core class that organizes module behavior. / **`RocmPlatform`**：组织模块行为的核心类。
- **`_rocm_device_count_stateless`**: Key helper or entry point in this file. / **`_rocm_device_count_stateless`**：本文件中的关键辅助函数或入口。
- **`_sync_hip_cuda_env_vars`**: Key helper or entry point in this file. / **`_sync_hip_cuda_env_vars`**：本文件中的关键辅助函数或入口。
- **`with_amdsmi_context`**: Key helper or entry point in this file. / **`with_amdsmi_context`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: os, datetime, functools, typing, importlib.util, contextlib
- **Third-party / 第三方**: regex, torch, torch.distributed, torch.distributed.distributed_c10d, amdsmi, torch.cuda
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger, vllm.v1.attention.backends.registry, .interface, vllm.config, vllm.config.kernel, vllm.v1.attention.selector, vllm._C, vllm._rocm_C, vllm._aiter_ops, vllm.config.compilation
