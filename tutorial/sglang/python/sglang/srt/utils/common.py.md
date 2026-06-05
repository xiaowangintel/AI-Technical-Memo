# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/common.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for common runtime utilities. / 为 SGLang 运行时提供面向通用运行时工具的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 14-101: Module setup and shared state / 模块设置与共享状态
```python
"""Common utilities."""

from __future__ import annotations

import argparse
import asyncio
import builtins
import ctypes
import functools
import gc
import importlib
import inspect
import io
import itertools
import json
import logging
import math
import os
import pickle
import platform
import random
import re
import resource
import shutil
import signal
import subprocess
import sys
import tempfile
# ... omitted for brevity ...

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)
torch_release = pkg_version.parse(torch.__version__).release
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `argparse`, `asyncio`, `builtins`, `ctypes`, `functools`. It also defines symbols such as `logger`, `torch_release` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `argparse`, `asyncio`, `builtins`, `ctypes`, `functools`。 同时定义了 `logger`, `torch_release` 等符号，供后续逻辑使用。

### Lines 105-107: Function `is_hip` / 函数 `is_hip`
```python
@lru_cache(maxsize=1)
def is_hip() -> bool:
    return torch.version.hip is not None
```
**EN:** This function implements `is_hip`. It primarily calls `lru_cache` to complete its work.
**CN:** 该函数实现了 `is_hip`。 它主要通过调用 `lru_cache` 来完成任务。

### Lines 110-125: Conditional and fallback logic / 条件与回退逻辑
```python
if is_hip():
    HIP_FP8_E4M3_FNUZ_MAX = 224.0
    FP8_E4M3_MAX = HIP_FP8_E4M3_FNUZ_MAX
else:
    FP8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

FP8_E4M3_MIN = -FP8_E4M3_MAX

builtins.FP8_E4M3_MAX = FP8_E4M3_MAX
builtins.FP8_E4M3_MIN = FP8_E4M3_MIN

# explicitly use pure text format, with a newline at the end
# this makes it impossible to see the animation in the progress bar
# but will avoid messing up with ray or multiprocessing, which wraps
# each line of output with some prefix.
BAR_FORMAT = "{desc}: {percentage:3.0f}% Completed | {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}]"
```
**EN:** This block contains conditional, fallback, or guarded runtime logic that adapts behavior to the current environment. It also updates symbols such as `HIP_FP8_E4M3_FNUZ_MAX`, `FP8_E4M3_MAX`, `FP8_E4M3_MIN`, `builtins.FP8_E4M3_MAX`, `builtins.FP8_E4M3_MIN`, `BAR_FORMAT`.
**CN:** 该代码块包含条件分支、回退路径或受保护的运行时逻辑，用于根据当前环境调整行为。 它还会更新 `HIP_FP8_E4M3_FNUZ_MAX`, `FP8_E4M3_MAX`, `FP8_E4M3_MIN`, `builtins.FP8_E4M3_MAX`, `builtins.FP8_E4M3_MIN`, `BAR_FORMAT` 等符号。

### Lines 128-130: Function `is_cuda` / 函数 `is_cuda`
```python
@lru_cache(maxsize=1)
def is_cuda():
    return torch.cuda.is_available() and torch.version.cuda is not None
```
**EN:** This function implements `is_cuda`. It primarily calls `lru_cache`, `torch.cuda.is_available` to complete its work.
**CN:** 该函数实现了 `is_cuda`。 它主要通过调用 `lru_cache`, `torch.cuda.is_available` 来完成任务。

### Lines 133-135: Function `is_cuda_alike` / 函数 `is_cuda_alike`
```python
@lru_cache(maxsize=1)
def is_cuda_alike():
    return is_cuda() or is_hip()
```
**EN:** This function implements `is_cuda_alike`. It primarily calls `lru_cache`, `is_cuda`, `is_hip` to complete its work.
**CN:** 该函数实现了 `is_cuda_alike`。 它主要通过调用 `lru_cache`, `is_cuda`, `is_hip` 来完成任务。

### Lines 138-140: Function `is_hpu` / 函数 `is_hpu`
```python
@lru_cache(maxsize=1)
def is_hpu() -> bool:
    return hasattr(torch, "hpu") and torch.hpu.is_available()
```
**EN:** This function implements `is_hpu`. It primarily calls `lru_cache`, `hasattr`, `torch.hpu.is_available` to complete its work.
**CN:** 该函数实现了 `is_hpu`。 它主要通过调用 `lru_cache`, `hasattr`, `torch.hpu.is_available` 来完成任务。

### Lines 143-145: Function `is_xpu` / 函数 `is_xpu`
```python
@lru_cache(maxsize=1)
def is_xpu() -> bool:
    return hasattr(torch, "xpu") and torch.xpu.is_available()
```
**EN:** This function implements `is_xpu`. It primarily calls `lru_cache`, `hasattr`, `torch.xpu.is_available` to complete its work.
**CN:** 该函数实现了 `is_xpu`。 它主要通过调用 `lru_cache`, `hasattr`, `torch.xpu.is_available` 来完成任务。

### Lines 148-158: Function `is_npu` / 函数 `is_npu`
```python
@lru_cache(maxsize=1)
def is_npu() -> bool:
    if not hasattr(torch, "npu"):
        return False

    if not torch.npu.is_available():
        raise RuntimeError(
            "torch_npu detected, but NPU device is not available or visible."
        )

    return True
```
**EN:** This function implements `is_npu`. It primarily calls `lru_cache`, `hasattr`, `torch.npu.is_available`, `RuntimeError` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_npu`。 它主要通过调用 `lru_cache`, `hasattr`, `torch.npu.is_available`, `RuntimeError` 来完成任务。 实现中使用了条件分支。

### Lines 161-168: Function `is_host_cpu_x86` / 函数 `is_host_cpu_x86`
```python
@lru_cache(maxsize=1)
def is_host_cpu_x86() -> bool:
    machine = platform.machine().lower()
    return (
        machine in ("x86_64", "amd64", "i386", "i686")
        and hasattr(torch, "cpu")
        and torch.cpu.is_available()
    )
```
**EN:** This function implements `is_host_cpu_x86`. It primarily calls `lru_cache`, `platform.machine.lower`, `hasattr`, `torch.cpu.is_available`, `platform.machine` to complete its work. State updates are written into `machine`.
**CN:** 该函数实现了 `is_host_cpu_x86`。 它主要通过调用 `lru_cache`, `platform.machine.lower`, `hasattr`, `torch.cpu.is_available`, `platform.machine` 来完成任务。 状态更新主要写入 `machine`。

### Lines 171-177: Function `is_host_cpu_arm64` / 函数 `is_host_cpu_arm64`
```python
def is_host_cpu_arm64() -> bool:
    machine = platform.machine().lower()
    return (
        machine in ("aarch64", "arm64")
        and hasattr(torch, "cpu")
        and torch.cpu.is_available()
    )
```
**EN:** This function implements `is_host_cpu_arm64`. It primarily calls `platform.machine.lower`, `hasattr`, `torch.cpu.is_available`, `platform.machine` to complete its work. State updates are written into `machine`.
**CN:** 该函数实现了 `is_host_cpu_arm64`。 它主要通过调用 `platform.machine.lower`, `hasattr`, `torch.cpu.is_available`, `platform.machine` 来完成任务。 状态更新主要写入 `machine`。

### Lines 180-183: Function `is_cpu` / 函数 `is_cpu`
```python
@lru_cache(maxsize=1)
def is_cpu() -> bool:
    is_host_cpu_supported = is_host_cpu_x86() or is_host_cpu_arm64()
    return os.getenv("SGLANG_USE_CPU_ENGINE", "0") == "1" and is_host_cpu_supported
```
**EN:** This function implements `is_cpu`. It primarily calls `lru_cache`, `is_host_cpu_x86`, `is_host_cpu_arm64`, `os.getenv` to complete its work. State updates are written into `is_host_cpu_supported`.
**CN:** 该函数实现了 `is_cpu`。 它主要通过调用 `lru_cache`, `is_host_cpu_x86`, `is_host_cpu_arm64`, `os.getenv` 来完成任务。 状态更新主要写入 `is_host_cpu_supported`。

### Lines 186-192: Function `is_musa` / 函数 `is_musa`
```python
@lru_cache(maxsize=1)
def is_musa() -> bool:
    try:
        import torchada  # noqa: F401
    except ImportError:
        return False
    return hasattr(torch.version, "musa") and torch.version.musa is not None
```
**EN:** This function implements `is_musa`. It primarily calls `lru_cache`, `hasattr` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `is_musa`。 它主要通过调用 `lru_cache`, `hasattr` 来完成任务。 实现中使用了错误处理。

### Lines 195-197: Function `is_mps` / 函数 `is_mps`
```python
@lru_cache(maxsize=1)
def is_mps() -> bool:
    return torch.backends.mps.is_available()
```
**EN:** This function implements `is_mps`. It primarily calls `lru_cache`, `torch.backends.mps.is_available` to complete its work.
**CN:** 该函数实现了 `is_mps`。 它主要通过调用 `lru_cache`, `torch.backends.mps.is_available` 来完成任务。

### Lines 200-203: Function `is_float4_e2m1fn_x2` / 函数 `is_float4_e2m1fn_x2`
```python
def is_float4_e2m1fn_x2(dtype) -> bool:
    """Check if dtype is float4_e2m1fn_x2 and CUDA is available."""
    target_dtype = getattr(torch, "float4_e2m1fn_x2", None)
    return is_cuda() and dtype == target_dtype
```
**EN:** This function implements `is_float4_e2m1fn_x2`. It primarily calls `getattr`, `is_cuda` to complete its work. State updates are written into `target_dtype`.
**CN:** 该函数实现了 `is_float4_e2m1fn_x2`。 它主要通过调用 `getattr`, `is_cuda` 来完成任务。 状态更新主要写入 `target_dtype`。

### Lines 206-209: Function `get_cuda_version` / 函数 `get_cuda_version`
```python
def get_cuda_version():
    if torch.version.cuda:
        return tuple(map(int, torch.version.cuda.split(".")))
    return (0, 0)
```
**EN:** This function implements `get_cuda_version`. It primarily calls `tuple`, `map`, `torch.version.cuda.split` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_cuda_version`。 它主要通过调用 `tuple`, `map`, `torch.version.cuda.split` 来完成任务。 实现中使用了条件分支。

### Lines 212-223: Function `device_context` / 函数 `device_context`
```python
@contextmanager
def device_context(device: torch.device):
    if device.type == "cpu" and is_cpu():
        with torch.device("cpu"):
            yield
    else:
        module = torch.get_device_module(device)
        if module is not None:
            with module.device(device.index):
                yield
        else:
            raise ValueError(f"Unknown device module: {device}")
```
**EN:** This function implements `device_context`. It primarily calls `is_cpu`, `torch.get_device_module`, `torch.device`, `ValueError`, `module.device` to complete its work. State updates are written into `module`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `device_context`。 它主要通过调用 `is_cpu`, `torch.get_device_module`, `torch.device`, `ValueError`, `module.device` 来完成任务。 状态更新主要写入 `module`。 实现中使用了条件分支、上下文管理资源。

### Lines 226-234: Function `_check_cuda_device_version` / 函数 `_check_cuda_device_version`
```python
def _check_cuda_device_version(
    device_capability_majors: List[int], cuda_version: Tuple[int, int]
):
    if not is_cuda():
        return False
    return (
        torch.cuda.get_device_capability()[0] in device_capability_majors
        and tuple(map(int, torch.version.cuda.split(".")[:2])) >= cuda_version
    )
```
**EN:** This function implements `_check_cuda_device_version`. It primarily calls `is_cuda`, `tuple`, `torch.cuda.get_device_capability`, `map`, `torch.version.cuda.split` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_check_cuda_device_version`。 它主要通过调用 `is_cuda`, `tuple`, `torch.cuda.get_device_capability`, `map`, `torch.version.cuda.split` 来完成任务。 实现中使用了条件分支。

### Lines 237-285: Conditional and fallback logic / 条件与回退逻辑
```python
is_ampere_with_cuda_12_3 = lru_cache(maxsize=1)(
    partial(
        _check_cuda_device_version, device_capability_majors=[8], cuda_version=(12, 3)
    )
)
is_hopper_with_cuda_12_3 = lru_cache(maxsize=1)(
    partial(
        _check_cuda_device_version, device_capability_majors=[9], cuda_version=(12, 3)
    )
)
is_blackwell_supported = is_blackwell = lru_cache(maxsize=1)(
    partial(
        _check_cuda_device_version,
        device_capability_majors=[10, 11, 12],
        cuda_version=(12, 8),
    )
)
is_sm120_supported = lru_cache(maxsize=1)(
    partial(
        _check_cuda_device_version, device_capability_majors=[12], cuda_version=(12, 8)
    )
)
is_sm100_supported = lru_cache(maxsize=1)(
    partial(
        _check_cuda_device_version, device_capability_majors=[10], cuda_version=(12, 8)
    )
)
is_sm90_supported = lru_cache(maxsize=1)(
# ... omitted for brevity ...
try:
    # move torch._C._cpu._is_amx_tile_supported() from cpu_has_amx_support
    # to support torch compile
    is_amx_tile_supported = torch._C._cpu._is_amx_tile_supported()
except:
    is_amx_tile_supported = False
```
**EN:** This block contains conditional, fallback, or guarded runtime logic that adapts behavior to the current environment. It also updates symbols such as `is_ampere_with_cuda_12_3`, `is_hopper_with_cuda_12_3`, `is_blackwell_supported`, `is_blackwell`, `is_sm120_supported`, `is_sm100_supported`.
**CN:** 该代码块包含条件分支、回退路径或受保护的运行时逻辑，用于根据当前环境调整行为。 它还会更新 `is_ampere_with_cuda_12_3`, `is_hopper_with_cuda_12_3`, `is_blackwell_supported`, `is_blackwell`, `is_sm120_supported`, `is_sm100_supported` 等符号。

### Lines 288-289: Function `cpu_has_amx_support` / 函数 `cpu_has_amx_support`
```python
def cpu_has_amx_support():
    return is_amx_tile_supported and is_intel_amx_backend_available
```
**EN:** This function implements `cpu_has_amx_support`.
**CN:** 该函数实现了 `cpu_has_amx_support`。

### Lines 292-293: Function `use_intel_amx_backend` / 函数 `use_intel_amx_backend`
```python
def use_intel_amx_backend(layer):
    return getattr(layer, "use_intel_amx_backend", False)
```
**EN:** This function implements `use_intel_amx_backend`. It primarily calls `getattr` to complete its work.
**CN:** 该函数实现了 `use_intel_amx_backend`。 它主要通过调用 `getattr` 来完成任务。

### Lines 296-301: Function `xpu_has_xmx_support` / 函数 `xpu_has_xmx_support`
```python
def xpu_has_xmx_support():
    # TODO: update with XPU capability query
    if is_xpu():
        # currently only PVC/LNL/BMG supports F64, so we only support these now
        return torch.xpu.get_device_properties().has_fp64
    return False
```
**EN:** This function implements `xpu_has_xmx_support`. It primarily calls `is_xpu`, `torch.xpu.get_device_properties` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `xpu_has_xmx_support`。 它主要通过调用 `is_xpu`, `torch.xpu.get_device_properties` 来完成任务。 实现中使用了条件分支。

### Lines 304-305: Function `use_intel_xpu_backend` / 函数 `use_intel_xpu_backend`
```python
def use_intel_xpu_backend():
    return get_bool_env_var("SGLANG_USE_SGL_XPU") and is_xpu()
```
**EN:** This function implements `use_intel_xpu_backend`. It primarily calls `get_bool_env_var`, `is_xpu` to complete its work.
**CN:** 该函数实现了 `use_intel_xpu_backend`。 它主要通过调用 `get_bool_env_var`, `is_xpu` 来完成任务。

### Lines 308-316: Function `is_flashinfer_available` / 函数 `is_flashinfer_available`
```python
@lru_cache(maxsize=1)
def is_flashinfer_available():
    """
    Check whether flashinfer is available.
    As of Oct. 6, 2024, it is only available on NVIDIA GPUs.
    """
    if not get_bool_env_var("SGLANG_IS_FLASHINFER_AVAILABLE", default="true"):
        return False
    return importlib.util.find_spec("flashinfer") is not None and is_cuda()
```
**EN:** This function implements `is_flashinfer_available`. It primarily calls `lru_cache`, `get_bool_env_var`, `is_cuda`, `importlib.util.find_spec` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_flashinfer_available`。 它主要通过调用 `lru_cache`, `get_bool_env_var`, `is_cuda`, `importlib.util.find_spec` 来完成任务。 实现中使用了条件分支。

### Lines 319-328: Function `is_tokenspeed_mla_available` / 函数 `is_tokenspeed_mla_available`
```python
@lru_cache(maxsize=1)
def is_tokenspeed_mla_available():
    """
    Check whether the tokenspeed_mla CuTe DSL kernels are available.
    Only available on NVIDIA Blackwell (SM100) at the moment.
    """
    return (
        importlib.util.find_spec("tokenspeed_mla") is not None
        and is_blackwell_supported()
    )
```
**EN:** This function implements `is_tokenspeed_mla_available`. It primarily calls `lru_cache`, `is_blackwell_supported`, `importlib.util.find_spec` to complete its work.
**CN:** 该函数实现了 `is_tokenspeed_mla_available`。 它主要通过调用 `lru_cache`, `is_blackwell_supported`, `importlib.util.find_spec` 来完成任务。

### Lines 331-338: Function `is_nvidia_cublas_version_ge_12_9` / 函数 `is_nvidia_cublas_version_ge_12_9`
```python
def is_nvidia_cublas_version_ge_12_9():
    """
    temporary fix for issue #11272 (cublas 12.9+)
    """
    for pkg in ("nvidia-cublas", "nvidia-cublas-cu12"):
        if check_pkg_version_at_least(pkg, "12.9"):
            return True
    return False
```
**EN:** This function implements `is_nvidia_cublas_version_ge_12_9`. It primarily calls `check_pkg_version_at_least` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `is_nvidia_cublas_version_ge_12_9`。 它主要通过调用 `check_pkg_version_at_least` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 341-342: Function `random_uuid` / 函数 `random_uuid`
```python
def random_uuid() -> str:
    return str(uuid.uuid4().hex)
```
**EN:** This function implements `random_uuid`. It primarily calls `str`, `uuid.uuid4` to complete its work.
**CN:** 该函数实现了 `random_uuid`。 它主要通过调用 `str`, `uuid.uuid4` 来完成任务。

### Lines 345-345: Constants and shared state / 常量与共享状态
```python
_warned_bool_env_var_keys = set()
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_warned_bool_env_var_keys`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_warned_bool_env_var_keys`。

### Lines 348-365: Function `get_bool_env_var` / 函数 `get_bool_env_var`
```python
def get_bool_env_var(name: str, default: str = "false") -> bool:
    # FIXME: move your environment variable to sglang.srt.environ
    value = os.getenv(name, default)
    value = value.lower()

    truthy_values = ("true", "1")
    falsy_values = ("false", "0")

    if (value not in truthy_values) and (value not in falsy_values):
        # Warn once per env var key (not per value), otherwise different keys that share the
        # same invalid value may suppress warnings incorrectly.
        if name not in _warned_bool_env_var_keys:
            logger.warning(
                f"get_bool_env_var({name}) encountered unrecognized value={value} and will treat as false"
            )
        _warned_bool_env_var_keys.add(name)

    return value in truthy_values
```
**EN:** This function implements `get_bool_env_var`. It primarily calls `os.getenv`, `value.lower`, `_warned_bool_env_var_keys.add`, `logger.warning` to complete its work. State updates are written into `value`, `truthy_values`, `falsy_values`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_bool_env_var`。 它主要通过调用 `os.getenv`, `value.lower`, `_warned_bool_env_var_keys.add`, `logger.warning` 来完成任务。 状态更新主要写入 `value`, `truthy_values`, `falsy_values`。 实现中使用了条件分支。

### Lines 368-376: Function `get_int_env_var` / 函数 `get_int_env_var`
```python
def get_int_env_var(name: str, default: int = 0) -> int:
    # FIXME: move your environment variable to sglang.srt.environ
    value = os.getenv(name)
    if value is None or not value.strip():
        return default
    try:
        return int(value)
    except ValueError:
        return default
```
**EN:** This function implements `get_int_env_var`. It primarily calls `os.getenv`, `int`, `value.strip` to complete its work. State updates are written into `value`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_int_env_var`。 它主要通过调用 `os.getenv`, `int`, `value.strip` 来完成任务。 状态更新主要写入 `value`。 实现中使用了条件分支、错误处理。

### Lines 379-380: Function `support_triton` / 函数 `support_triton`
```python
def support_triton(backend: str) -> bool:
    return backend not in ["torch_native", "intel_amx"]
```
**EN:** This function implements `support_triton`.
**CN:** 该函数实现了 `support_triton`。

### Lines 383-385: Constants and shared state / 常量与共享状态
```python
_ENABLE_TORCH_INFERENCE_MODE = get_bool_env_var(
    "SGLANG_ENABLE_TORCH_INFERENCE_MODE", "false"
)
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_ENABLE_TORCH_INFERENCE_MODE`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_ENABLE_TORCH_INFERENCE_MODE`。

### Lines 388-392: Class `DynamicGradMode` declaration / 类 `DynamicGradMode` 声明
```python
class DynamicGradMode(_DecoratorContextManager):
    """
    A combination of torch.no_grad and torch.inference_mode,
    with their behavior controlled by an environment variable. Just refer to them.
    """
```
**EN:** This class establishes `DynamicGradMode` as the main container/coordinator for the surrounding logic. It inherits from `_DecoratorContextManager`. Its core interface includes methods such as `set_inference_mode`, `__init__`, `__new__`, `__enter__`, `__exit__`, `clone`.
**CN:** 该类将 `DynamicGradMode` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_DecoratorContextManager`。 其核心接口包括 `set_inference_mode`, `__init__`, `__new__`, `__enter__`, `__exit__`, `clone` 等方法。

### Lines 394-401: Method `DynamicGradMode.set_inference_mode` / 方法 `DynamicGradMode.set_inference_mode`
```python
    @staticmethod
    def set_inference_mode(mode: bool):
        if isinstance(mode, bool):
            global _ENABLE_TORCH_INFERENCE_MODE

            _ENABLE_TORCH_INFERENCE_MODE = mode
        else:
            logger.warning("mode is not a boolean object")
```
**EN:** This method implements `set_inference_mode` on `DynamicGradMode`. It primarily calls `isinstance`, `logger.warning` to complete its work. State updates are written into `_ENABLE_TORCH_INFERENCE_MODE`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `set_inference_mode`。 它主要通过调用 `isinstance`, `logger.warning` 来完成任务。 状态更新主要写入 `_ENABLE_TORCH_INFERENCE_MODE`。 实现中使用了条件分支。

### Lines 403-409: Method `DynamicGradMode.__init__` / 方法 `DynamicGradMode.__init__`
```python
    def __init__(self, mode=True):
        if not torch._jit_internal.is_scripting():
            super().__init__()
        if _ENABLE_TORCH_INFERENCE_MODE:
            self.mode = mode
        else:
            self.prev = False
```
**EN:** This method implements `__init__` on `DynamicGradMode`. It primarily calls `torch._jit_internal.is_scripting`, `super.__init__`, `super` to complete its work. State updates are written into `self.mode`, `self.prev`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `__init__`。 它主要通过调用 `torch._jit_internal.is_scripting`, `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.mode`, `self.prev`。 实现中使用了条件分支。

### Lines 411-414: Method `DynamicGradMode.__new__` / 方法 `DynamicGradMode.__new__`
```python
    def __new__(cls, mode_or_orig_func=True if _ENABLE_TORCH_INFERENCE_MODE else None):
        if mode_or_orig_func is None or isinstance(mode_or_orig_func, bool):
            return super().__new__(cls)
        return cls()(mode_or_orig_func)
```
**EN:** This method implements `__new__` on `DynamicGradMode`. It primarily calls `cls`, `isinstance`, `super.__new__`, `super` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `__new__`。 它主要通过调用 `cls`, `isinstance`, `super.__new__`, `super` 来完成任务。 实现中使用了条件分支。

### Lines 416-422: Method `DynamicGradMode.__enter__` / 方法 `DynamicGradMode.__enter__`
```python
    def __enter__(self) -> None:
        if _ENABLE_TORCH_INFERENCE_MODE:
            self._inference_mode_context = torch._C._InferenceMode(self.mode)
            self._inference_mode_context.__enter__()
        else:
            self.prev = torch.is_grad_enabled()
            torch.set_grad_enabled(False)
```
**EN:** This method implements `__enter__` on `DynamicGradMode`. It primarily calls `torch._C._InferenceMode`, `self._inference_mode_context.__enter__`, `torch.is_grad_enabled`, `torch.set_grad_enabled` to complete its work. State updates are written into `self._inference_mode_context`, `self.prev`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `__enter__`。 它主要通过调用 `torch._C._InferenceMode`, `self._inference_mode_context.__enter__`, `torch.is_grad_enabled`, `torch.set_grad_enabled` 来完成任务。 状态更新主要写入 `self._inference_mode_context`, `self.prev`。 实现中使用了条件分支。

### Lines 424-428: Method `DynamicGradMode.__exit__` / 方法 `DynamicGradMode.__exit__`
```python
    def __exit__(self, exc_type: Any, exc_value: Any, traceback: Any) -> None:
        if _ENABLE_TORCH_INFERENCE_MODE:
            self._inference_mode_context.__exit__(exc_type, exc_value, traceback)
        else:
            torch.set_grad_enabled(self.prev)
```
**EN:** This method implements `__exit__` on `DynamicGradMode`. It primarily calls `self._inference_mode_context.__exit__`, `torch.set_grad_enabled` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `__exit__`。 它主要通过调用 `self._inference_mode_context.__exit__`, `torch.set_grad_enabled` 来完成任务。 实现中使用了条件分支。

### Lines 430-437: Method `DynamicGradMode.clone` / 方法 `DynamicGradMode.clone`
```python
    def clone(self) -> "DynamicGradMode":
        r"""
        Create a copy of this class
        """
        if _ENABLE_TORCH_INFERENCE_MODE:
            return self.__class__(self.mode)
        else:
            return self.__class__()
```
**EN:** This method implements `clone` on `DynamicGradMode`. It primarily calls `self.__class__` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `DynamicGradMode`）实现了 `clone`。 它主要通过调用 `self.__class__` 来完成任务。 实现中使用了条件分支。

### Lines 440-441: Constants and shared state / 常量与共享状态
```python
show_time_cost = False
time_infos = {}
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `show_time_cost`, `time_infos`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `show_time_cost`, `time_infos`。

### Lines 444-446: Function `enable_show_time_cost` / 函数 `enable_show_time_cost`
```python
def enable_show_time_cost():
    global show_time_cost
    show_time_cost = True
```
**EN:** This function implements `enable_show_time_cost`. State updates are written into `show_time_cost`.
**CN:** 该函数实现了 `enable_show_time_cost`。 状态更新主要写入 `show_time_cost`。

### Lines 449-449: Class `TimeInfo` declaration / 类 `TimeInfo` 声明
```python
class TimeInfo:
```
**EN:** This class establishes `TimeInfo` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `check`, `pretty_print`.
**CN:** 该类将 `TimeInfo` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `check`, `pretty_print` 等方法。

### Lines 450-457: Method `TimeInfo.__init__` / 方法 `TimeInfo.__init__`
```python
    def __init__(self, name, interval=0.1, color=0, indent=0):
        self.name = name
        self.interval = interval
        self.color = color
        self.indent = indent

        self.acc_time = 0
        self.last_acc_time = 0
```
**EN:** This method implements `__init__` on `TimeInfo`. State updates are written into `self.name`, `self.interval`, `self.color`, `self.indent`, `self.acc_time`, `self.last_acc_time`.
**CN:** 该方法（属于 `TimeInfo`）实现了 `__init__`。 状态更新主要写入 `self.name`, `self.interval`, `self.color`, `self.indent`, `self.acc_time`, `self.last_acc_time`。

### Lines 459-463: Method `TimeInfo.check` / 方法 `TimeInfo.check`
```python
    def check(self):
        if self.acc_time - self.last_acc_time > self.interval:
            self.last_acc_time = self.acc_time
            return True
        return False
```
**EN:** This method implements `check` on `TimeInfo`. State updates are written into `self.last_acc_time`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `TimeInfo`）实现了 `check`。 状态更新主要写入 `self.last_acc_time`。 实现中使用了条件分支。

### Lines 465-468: Method `TimeInfo.pretty_print` / 方法 `TimeInfo.pretty_print`
```python
    def pretty_print(self):
        print(f"\x1b[{self.color}m", end="")
        print("-" * self.indent * 2, end="")
        print(f"{self.name}: {self.acc_time:.3f}s\x1b[0m")
```
**EN:** This method implements `pretty_print` on `TimeInfo`. It primarily calls `print` to complete its work.
**CN:** 该方法（属于 `TimeInfo`）实现了 `pretty_print`。 它主要通过调用 `print` 来完成任务。

### Lines 471-478: Function `mark_start` / 函数 `mark_start`
```python
def mark_start(name, interval=0.1, color=0, indent=0):
    global time_infos, show_time_cost
    if not show_time_cost:
        return
    torch.cuda.synchronize()
    if time_infos.get(name, None) is None:
        time_infos[name] = TimeInfo(name, interval, color, indent)
    time_infos[name].acc_time -= time.perf_counter()
```
**EN:** This function implements `mark_start`. It primarily calls `torch.cuda.synchronize`, `time.perf_counter`, `time_infos.get`, `TimeInfo` to complete its work. State updates are written into `time_infos.acc_time`, `time_infos`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `mark_start`。 它主要通过调用 `torch.cuda.synchronize`, `time.perf_counter`, `time_infos.get`, `TimeInfo` 来完成任务。 状态更新主要写入 `time_infos.acc_time`, `time_infos`。 实现中使用了条件分支。

### Lines 481-488: Function `mark_end` / 函数 `mark_end`
```python
def mark_end(name):
    global time_infos, show_time_cost
    if not show_time_cost:
        return
    torch.cuda.synchronize()
    time_infos[name].acc_time += time.perf_counter()
    if time_infos[name].check():
        time_infos[name].pretty_print()
```
**EN:** This function implements `mark_end`. It primarily calls `torch.cuda.synchronize`, `time.perf_counter`, `time_infos.check`, `time_infos.pretty_print` to complete its work. State updates are written into `time_infos.acc_time`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `mark_end`。 它主要通过调用 `torch.cuda.synchronize`, `time.perf_counter`, `time_infos.check`, `time_infos.pretty_print` 来完成任务。 状态更新主要写入 `time_infos.acc_time`。 实现中使用了条件分支。

### Lines 491-507: Function `calculate_time` / 函数 `calculate_time`
```python
def calculate_time(show=False, min_cost_ms=0.0):
    def wrapper(func):
        def inner_func(*args, **kwargs):
            torch.cuda.synchronize()
            if show:
                start_time = time.perf_counter()
            result = func(*args, **kwargs)
            torch.cuda.synchronize()
            if show:
                cost_time = (time.perf_counter() - start_time) * 1000
                if cost_time > min_cost_ms:
                    print(f"Function {func.__name__} took {cost_time} ms to run.")
            return result

        return inner_func

    return wrapper
```
**EN:** This function implements `calculate_time`. It primarily calls `torch.cuda.synchronize`, `func`, `time.perf_counter`, `print` to complete its work. State updates are written into `result`, `start_time`, `cost_time`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `calculate_time`。 它主要通过调用 `torch.cuda.synchronize`, `func`, `time.perf_counter`, `print` 来完成任务。 状态更新主要写入 `result`, `start_time`, `cost_time`。 实现中使用了条件分支。

### Lines 510-526: Function `empty_device_cache` / 函数 `empty_device_cache`
```python
def empty_device_cache(device_module: Optional[Any] = None) -> bool:
    """Release unused cached blocks from the active device allocator.

    This does not clear SGLang KV/radix/request caches and does not free live
    tensors. It only forwards to the backend allocator's empty_cache hook when
    one is available.
    """

    if device_module is None:
        device_module = torch.get_device_module()

    empty_cache = getattr(device_module, "empty_cache", None)
    if empty_cache is None:
        return False

    empty_cache()
    return True
```
**EN:** This function implements `empty_device_cache`. It primarily calls `getattr`, `empty_cache`, `torch.get_device_module` to complete its work. State updates are written into `empty_cache`, `device_module`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `empty_device_cache`。 它主要通过调用 `getattr`, `empty_cache`, `torch.get_device_module` 来完成任务。 状态更新主要写入 `empty_cache`, `device_module`。 实现中使用了条件分支。

### Lines 529-653: Function `get_available_gpu_memory` / 函数 `get_available_gpu_memory`
```python
def get_available_gpu_memory(
    device, gpu_id, distributed=False, empty_cache=True, cpu_group=None
):
    """
    Get available memory for cuda:gpu_id device.
    When distributed is True, the available memory is the minimum available memory of all GPUs.
    """
    if device == "cuda":
        num_gpus = torch.cuda.device_count()
        assert gpu_id < num_gpus

        if torch.cuda.current_device() != gpu_id:
            print(
                f"WARNING: current device is not {gpu_id}, but {torch.cuda.current_device()}, ",
                "which may cause useless memory allocation for torch CUDA context.",
            )

        if empty_cache:
            empty_device_cache(torch.cuda)
        props = torch.cuda.get_device_properties(gpu_id)
        if props.is_integrated:
            # On these devices, which use sysmem as device mem, torch.cuda.mem_get_info()
            # only reports "free" memory, which can be lower than what is actually
            # available due to not including cache memory. So we use the system available
            # memory metric instead.
            free_gpu_memory = psutil.virtual_memory().available
        else:
            free_gpu_memory, _ = torch.cuda.mem_get_info(gpu_id)
# ... omitted for brevity ...
        torch.distributed.all_reduce(
            tensor, op=torch.distributed.ReduceOp.MIN, group=cpu_group
        )
        free_gpu_memory = tensor.item()

    return free_gpu_memory / (1 << 30)
```
**EN:** This function implements `get_available_gpu_memory`. It primarily calls `torch.cuda.device_count`, `torch.cuda.get_device_properties`, `torch.tensor`, `torch.distributed.all_reduce`, `tensor.item`, `torch.cuda.current_device` to complete its work. State updates are written into `num_gpus`, `props`, `tensor`, `free_gpu_memory`, `used_memory`, `total_gpu_memory`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_available_gpu_memory`。 它主要通过调用 `torch.cuda.device_count`, `torch.cuda.get_device_properties`, `torch.tensor`, `torch.distributed.all_reduce`, `tensor.item`, `torch.cuda.current_device` 来完成任务。 状态更新主要写入 `num_gpus`, `props`, `tensor`, `free_gpu_memory`, `used_memory`, `total_gpu_memory`。 实现中使用了条件分支。

### Lines 656-661: Function `is_pin_memory_available` / 函数 `is_pin_memory_available`
```python
def is_pin_memory_available(device=None) -> bool:
    if not torch.cuda.is_available():
        return False
    if device is not None and str(device) == "cpu":
        return False
    return True
```
**EN:** This function implements `is_pin_memory_available`. It primarily calls `torch.cuda.is_available`, `str` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_pin_memory_available`。 它主要通过调用 `torch.cuda.is_available`, `str` 来完成任务。 实现中使用了条件分支。

### Lines 664-664: Class `LayerFn` declaration / 类 `LayerFn` 声明
```python
class LayerFn(Protocol):
```
**EN:** This class establishes `LayerFn` as the main container/coordinator for the surrounding logic. It inherits from `Protocol`. Its core interface includes methods such as `__call__`.
**CN:** 该类将 `LayerFn` 定义为周边逻辑的主要封装体或协调者。 它继承自 `Protocol`。 其核心接口包括 `__call__` 等方法。

### Lines 666-666: Method `LayerFn.__call__` / 方法 `LayerFn.__call__`
```python
    def __call__(self, idx: int, prefix: str) -> torch.nn.Module: ...
```
**EN:** This method implements `__call__` on `LayerFn`.
**CN:** 该方法（属于 `LayerFn`）实现了 `__call__`。

### Lines 669-710: Function `make_layers` / 函数 `make_layers`
```python
def make_layers(
    num_hidden_layers: int,
    layer_fn: LayerFn,
    pp_rank: Optional[int] = None,
    pp_size: Optional[int] = None,
    prefix: str = "",
    return_tuple: bool = False,
    offloader_kwargs: Optional[Dict[str, Any]] = None,
) -> Tuple[torch.nn.Module, int, int]:
    """Make a list of layers with the given layer function"""
    # circular imports
    from sglang.srt.distributed import get_pp_indices
    from sglang.srt.layers.utils import PPMissingLayer
    from sglang.srt.utils.offloader import get_offloader

    assert not pp_size or num_hidden_layers >= pp_size
    start_layer, end_layer = (
        get_pp_indices(
            num_hidden_layers,
            pp_rank,
            pp_size,
        )
        if pp_rank is not None and pp_size is not None
        else (0, num_hidden_layers)
    )
    modules = torch.nn.ModuleList(
        [PPMissingLayer(return_tuple=return_tuple) for _ in range(start_layer)]
        + get_offloader().wrap_modules(
# ... omitted for brevity ...
            for _ in range(end_layer, num_hidden_layers)
        ]
    )
    if pp_rank is None or pp_size is None:
        return modules
    return modules, start_layer, end_layer
```
**EN:** This function implements `make_layers`. It primarily calls `torch.nn.ModuleList`, `get_pp_indices`, `get_offloader.wrap_modules`, `PPMissingLayer`, `range`, `get_offloader` to complete its work. State updates are written into `modules`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `make_layers`。 它主要通过调用 `torch.nn.ModuleList`, `get_pp_indices`, `get_offloader.wrap_modules`, `PPMissingLayer`, `range`, `get_offloader` 来完成任务。 状态更新主要写入 `modules`。 实现中使用了条件分支。

### Lines 713-728: Function `make_layers_non_pp` / 函数 `make_layers_non_pp`
```python
def make_layers_non_pp(
    num_hidden_layers: int,
    layer_fn: LayerFn,
    prefix: str = "",
) -> torch.nn.ModuleList:
    from sglang.srt.utils.offloader import get_offloader

    layers = torch.nn.ModuleList(
        get_offloader().wrap_modules(
            (
                layer_fn(idx=idx, prefix=add_prefix(idx, prefix))
                for idx in range(num_hidden_layers)
            )
        )
    )
    return layers
```
**EN:** This function implements `make_layers_non_pp`. It primarily calls `torch.nn.ModuleList`, `get_offloader.wrap_modules`, `get_offloader`, `layer_fn`, `range`, `add_prefix` to complete its work. State updates are written into `layers`.
**CN:** 该函数实现了 `make_layers_non_pp`。 它主要通过调用 `torch.nn.ModuleList`, `get_offloader.wrap_modules`, `get_offloader`, `layer_fn`, `range`, `add_prefix` 来完成任务。 状态更新主要写入 `layers`。

### Lines 731-738: Function `get_dispatch_device_backend` / 函数 `get_dispatch_device_backend`
```python
def get_dispatch_device_backend():
    if is_cuda_alike():
        dispatch_key = "CUDA"
    elif is_xpu():
        dispatch_key = "XPU"
    else:
        raise RuntimeError("No supported accelerator (CUDA/XPU) available")
    return dispatch_key
```
**EN:** This function implements `get_dispatch_device_backend`. It primarily calls `is_cuda_alike`, `is_xpu`, `RuntimeError` to complete its work. State updates are written into `dispatch_key`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_dispatch_device_backend`。 它主要通过调用 `is_cuda_alike`, `is_xpu`, `RuntimeError` 来完成任务。 状态更新主要写入 `dispatch_key`。 实现中使用了条件分支。

### Lines 741-743: Function `get_device_module` / 函数 `get_device_module`
```python
@lru_cache(maxsize=1)
def get_device_module():
    return torch.get_device_module()
```
**EN:** This function implements `get_device_module`. It primarily calls `lru_cache`, `torch.get_device_module` to complete its work.
**CN:** 该函数实现了 `get_device_module`。 它主要通过调用 `lru_cache`, `torch.get_device_module` 来完成任务。

### Lines 746-754: Function `set_random_seed` / 函数 `set_random_seed`
```python
def set_random_seed(seed: int) -> None:
    """Set the random seed for all libraries."""
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
    if torch.xpu.is_available():
        torch.xpu.manual_seed_all(seed)
```
**EN:** This function implements `set_random_seed`. It primarily calls `random.seed`, `np.random.seed`, `torch.manual_seed`, `torch.cuda.is_available`, `torch.xpu.is_available`, `torch.cuda.manual_seed_all` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `set_random_seed`。 它主要通过调用 `random.seed`, `np.random.seed`, `torch.manual_seed`, `torch.cuda.is_available`, `torch.xpu.is_available`, `torch.cuda.manual_seed_all` 来完成任务。 实现中使用了条件分支。

### Lines 757-829: Function `load_audio` / 函数 `load_audio`
```python
def load_audio(
    audio_file: str, sr: Optional[int] = None, mono: bool = True
) -> np.ndarray:
    if sr is None:
        sr = 16000

    # Normalize input: resolve URL / base64 / file:// to bytes or path
    if isinstance(audio_file, bytes):
        source = audio_file
    elif isinstance(audio_file, str) and audio_file.startswith("data:"):
        source = pybase64.b64decode(audio_file.split(",")[1], validate=True)
    elif isinstance(audio_file, str) and (
        audio_file.startswith("http://") or audio_file.startswith("https://")
    ):
        timeout = int(os.getenv("REQUEST_TIMEOUT", "5"))
        with requests.get(audio_file, timeout=timeout) as response:
            response.raise_for_status()
            source = response.content
    elif isinstance(audio_file, str) and audio_file.startswith("file://"):
        source = unquote(urlparse(audio_file).path)
    elif isinstance(audio_file, str):
        source = audio_file
    else:
        raise ValueError(f"Invalid audio format: {audio_file}")

    if _BACKEND == "torchcodec":
        from torchcodec.decoders import AudioDecoder

# ... omitted for brevity ...
        if audio_tensor.shape[0] == 1:
            audio = audio_tensor.squeeze(0).numpy()
        else:
            audio = audio_tensor.T.numpy()

    return audio
```
**EN:** This function implements `load_audio`. It primarily calls `isinstance`, `sf.read`, `np.mean`, `torch.from_numpy.float`, `torchaudio.functional.resample`, `audio_file.startswith` to complete its work. State updates are written into `sr`, `source`, `audio`, `audio_tensor`, `decoder`, `samples`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该函数实现了 `load_audio`。 它主要通过调用 `isinstance`, `sf.read`, `np.mean`, `torch.from_numpy.float`, `torchaudio.functional.resample`, `audio_file.startswith` 来完成任务。 状态更新主要写入 `sr`, `source`, `audio`, `audio_tensor`, `decoder`, `samples`。 实现中使用了条件分支、上下文管理资源、错误处理。

### Lines 832-837: Class `ImageData` declaration / 类 `ImageData` 声明
```python
@dataclass
class ImageData:
    url: str
    detail: Optional[Literal["auto", "low", "high"]] = "auto"
    max_dynamic_patch: Optional[int] = None
    preprocess_kwargs: Optional[Dict] = None
```
**EN:** This class establishes `ImageData` as a compact data container for the surrounding logic. The main stored fields include `url`, `detail`, `max_dynamic_patch`, `preprocess_kwargs`.
**CN:** 该类将 `ImageData` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `url`, `detail`, `max_dynamic_patch`, `preprocess_kwargs`。

### Lines 840-843: Class `VideoData` declaration / 类 `VideoData` 声明
```python
@dataclass
class VideoData:
    url: str
    preprocess_kwargs: Optional[Dict] = None
```
**EN:** This class establishes `VideoData` as a compact data container for the surrounding logic. The main stored fields include `url`, `preprocess_kwargs`.
**CN:** 该类将 `VideoData` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `url`, `preprocess_kwargs`。

### Lines 846-846: Constants and shared state / 常量与共享状态
```python
image_extension_names = (".png", ".jpg", ".jpeg", ".webp", ".gif")
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `image_extension_names`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `image_extension_names`。

### Lines 849-860: Function `is_jpeg_with_cuda` / 函数 `is_jpeg_with_cuda`
```python
def is_jpeg_with_cuda(image_bytes: bytes = b"", gpu_image_decode: bool = True) -> bool:
    """
    Check three conditions:
    1. whether CUDA is available.
    2. whether input is recognized as JPEG.
    3. whether GPU image decode is enabled (some models such as CPM forcibly disable this).
    """
    if not is_cuda() or not gpu_image_decode:
        return False
    if image_bytes != b"":
        return image_bytes.startswith(b"\xff\xd8") and image_bytes.endswith(b"\xff\xd9")
    return False
```
**EN:** This function implements `is_jpeg_with_cuda`. It primarily calls `is_cuda`, `image_bytes.startswith`, `image_bytes.endswith` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_jpeg_with_cuda`。 它主要通过调用 `is_cuda`, `image_bytes.startswith`, `image_bytes.endswith` 来完成任务。 实现中使用了条件分支。

### Lines 863-884: Function `_load_image` / 函数 `_load_image`
```python
def _load_image(
    image_bytes: bytes = b"",
    image_file: str = "",
    gpu_image_decode: bool = True,
) -> Union[torch.Tensor, Image.Image]:
    """
    Try to decode JPEG with nvJPEG on GPU and return a torch device tensor,
    otherwise fallback to decode with PIL on CPU and return a PIL Image.
    Keep the fallback path since nvJPEG may fail on some JPEG images that are not strictly compliant with the standard, while PIL is more tolerant.
    """
    if image_file != "":
        image_bytes = get_image_bytes(image_file)
    if is_jpeg_with_cuda(image_bytes, gpu_image_decode):
        try:
            encoded_image = torch.frombuffer(image_bytes, dtype=torch.uint8)
            image_tensor = decode_jpeg(encoded_image, device="cuda")
            return image_tensor
        except Exception as e:
            logger.warning(
                f"Failed to decode JPEG on GPU, falling back to CPU. Error: {e}"
            )
    return Image.open(BytesIO(image_bytes))
```
**EN:** This function implements `_load_image`. It primarily calls `is_jpeg_with_cuda`, `Image.open`, `get_image_bytes`, `BytesIO`, `torch.frombuffer`, `decode_jpeg` to complete its work. State updates are written into `image_bytes`, `encoded_image`, `image_tensor`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_load_image`。 它主要通过调用 `is_jpeg_with_cuda`, `Image.open`, `get_image_bytes`, `BytesIO`, `torch.frombuffer`, `decode_jpeg` 来完成任务。 状态更新主要写入 `image_bytes`, `encoded_image`, `image_tensor`。 实现中使用了条件分支、错误处理。

### Lines 887-924: Function `load_image` / 函数 `load_image`
```python
def load_image(
    image_file: Union[Image.Image, str, ImageData, bytes],
    gpu_image_decode: bool = True,
) -> tuple[Union[torch.Tensor, Image.Image], Optional[tuple[int, int]]]:
    """
    Load image from multiple input formats, including:
    ImageData, PIL Image, bytes, URL, file path, or base64 string.
    """
    if isinstance(image_file, ImageData):
        image_file = image_file.url

    image = None
    image_size: Optional[tuple[int, int]] = None
    if isinstance(image_file, Image.Image):
        image = image_file
        image_size = (image.width, image.height)
    elif isinstance(image_file, bytes):
        image = _load_image(image_bytes=image_file, gpu_image_decode=gpu_image_decode)
    elif isinstance(image_file, str) and image_file.startswith(("http://", "https://")):
        image = _load_image(image_file=image_file, gpu_image_decode=gpu_image_decode)
    elif isinstance(image_file, str) and image_file.startswith("file://"):
        image = _load_image(
            image_file=unquote(urlparse(image_file).path),
            gpu_image_decode=gpu_image_decode,
        )
    elif isinstance(image_file, str) and image_file.lower().endswith(
        image_extension_names
    ):
# ... omitted for brevity ...
        image_file, str
    ):  # Other formats, try to decode as base64 by default
        image = _load_image(image_file=image_file, gpu_image_decode=gpu_image_decode)
    else:
        raise ValueError(f"Invalid image: {image_file}")
    return image, image_size
```
**EN:** This function implements `load_image`. It primarily calls `isinstance`, `_load_image`, `image_file.startswith`, `image_file.lower.endswith`, `unquote`, `image_file.lower` to complete its work. State updates are written into `image`, `image_size`, `image_file`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `load_image`。 它主要通过调用 `isinstance`, `_load_image`, `image_file.startswith`, `image_file.lower.endswith`, `unquote`, `image_file.lower` 来完成任务。 状态更新主要写入 `image`, `image_size`, `image_file`。 实现中使用了条件分支。

### Lines 927-948: Function `get_image_bytes` / 函数 `get_image_bytes`
```python
def get_image_bytes(image_file: Union[str, bytes]) -> bytes:
    """Normalize various image inputs into raw bytes."""
    if isinstance(image_file, bytes):
        return image_file
    if image_file.startswith(("http://", "https://")):
        timeout = int(os.getenv("REQUEST_TIMEOUT", "3"))
        response = requests.get(image_file, timeout=timeout)
        try:
            response.raise_for_status()
            result = response.content
        finally:
            response.close()
        return result
    if image_file.startswith(("file://", "/")):
        with open(image_file, "rb") as f:
            return f.read()
    if isinstance(image_file, str) and image_file.startswith("data:"):
        _, encoded = image_file.split(",", 1)
        return pybase64.b64decode(encoded, validate=True)
    if isinstance(image_file, str):
        return pybase64.b64decode(image_file, validate=True)
    raise NotImplementedError(f"Invalid image: {image_file}")
```
**EN:** This function implements `get_image_bytes`. It primarily calls `isinstance`, `image_file.startswith`, `NotImplementedError`, `int`, `requests.get`, `image_file.split` to complete its work. State updates are written into `timeout`, `response`, `result`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该函数实现了 `get_image_bytes`。 它主要通过调用 `isinstance`, `image_file.startswith`, `NotImplementedError`, `int`, `requests.get`, `image_file.split` 来完成任务。 状态更新主要写入 `timeout`, `response`, `result`。 实现中使用了条件分支、上下文管理资源、错误处理。

### Lines 951-978: Function `_normalize_video_input` / 函数 `_normalize_video_input`
```python
def _normalize_video_input(
    video_file: Union[str, bytes],
) -> Union[str, bytes, None]:
    """Normalize video input (URL, base64, file://, etc.) to a file path or bytes.

    Returns a file path or bytes suitable for a decoder, or None on failure.
    URLs and base64 are returned as bytes (no temp files needed since both
    torchcodec and VideoDecoderWrapper accept bytes natively).
    """
    if isinstance(video_file, bytes):
        return video_file
    elif isinstance(video_file, str):
        if video_file.startswith(("http://", "https://")):
            timeout = int(os.getenv("REQUEST_TIMEOUT", "10"))
            response = requests.get(video_file, stream=True, timeout=timeout)
            response.raise_for_status()
            return response.content
        elif video_file.startswith("data:"):
            _, encoded = video_file.split(",", 1)
            return pybase64.b64decode(encoded, validate=True)
        elif video_file.startswith("file://"):
            return unquote(urlparse(video_file).path)
        elif os.path.isfile(unquote(urlparse(video_file).path)):
            return video_file
        else:
            return pybase64.b64decode(video_file, validate=True)
    else:
        return None
```
**EN:** This function implements `_normalize_video_input`. It primarily calls `isinstance`, `video_file.startswith`, `int`, `requests.get`, `response.raise_for_status`, `os.getenv` to complete its work. State updates are written into `timeout`, `response`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_normalize_video_input`。 它主要通过调用 `isinstance`, `video_file.startswith`, `int`, `requests.get`, `response.raise_for_status`, `os.getenv` 来完成任务。 状态更新主要写入 `timeout`, `response`。 实现中使用了条件分支。

### Lines 981-994: Function `load_video` / 函数 `load_video`
```python
def load_video(video_file: Union[str, bytes, VideoData], use_gpu: bool = True):
    if isinstance(video_file, VideoData):
        # preprocess_kwargs is consumed by the multimodal processor, not here.
        video_file = video_file.url

    if isinstance(video_file, (list, tuple, torch.Tensor, np.ndarray)):
        return video_file

    source = _normalize_video_input(video_file)
    if source is None:
        raise ValueError(f"Unsupported video input type: {type(video_file)}")

    device = "cuda" if use_gpu else "cpu"
    return VideoDecoderWrapper(source, device=device)
```
**EN:** This function implements `load_video`. It primarily calls `isinstance`, `_normalize_video_input`, `VideoDecoderWrapper`, `ValueError`, `type` to complete its work. State updates are written into `source`, `device`, `video_file`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `load_video`。 它主要通过调用 `isinstance`, `_normalize_video_input`, `VideoDecoderWrapper`, `ValueError`, `type` 来完成任务。 状态更新主要写入 `source`, `device`, `video_file`。 实现中使用了条件分支。

### Lines 997-1011: Function `sample_video_frames` / 函数 `sample_video_frames`
```python
def sample_video_frames(video, *, desired_fps: int, max_frames: int) -> list[int]:
    total_frames = len(video)
    assert total_frames > 0, "Video must have at least one frame"

    avg_fps = video.avg_fps
    duration = total_frames / avg_fps if avg_fps > 0 else 0
    fps = min(desired_fps, avg_fps)

    num_frames = math.floor(duration * fps)
    num_frames = min(max_frames, num_frames, total_frames)
    num_frames = max(1, num_frames)  # At least one frame
    if num_frames == total_frames:
        return list(range(total_frames))
    else:
        return np.linspace(0, total_frames - 1, num_frames, dtype=int).tolist()
```
**EN:** This function implements `sample_video_frames`. It primarily calls `len`, `min`, `math.floor`, `max`, `list`, `np.linspace.tolist` to complete its work. State updates are written into `total_frames`, `avg_fps`, `duration`, `fps`, `num_frames`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `sample_video_frames`。 它主要通过调用 `len`, `min`, `math.floor`, `max`, `list`, `np.linspace.tolist` 来完成任务。 状态更新主要写入 `total_frames`, `avg_fps`, `duration`, `fps`, `num_frames`。 实现中使用了条件分支。

### Lines 1014-1044: Function `encode_video` / 函数 `encode_video`
```python
def encode_video(video_path, frame_count_limit=None):
    if not os.path.exists(video_path):
        logger.error(f"Video {video_path} does not exist")
        return []

    if frame_count_limit == 0:
        return []

    def uniform_sample(l, n):
        gap = len(l) / n
        idxs = [int(i * gap + gap / 2) for i in range(n)]
        return [l[i] for i in idxs]

    decoder = VideoDecoderWrapper(video_path)
    avg_fps = decoder.avg_fps
    total_frames = len(decoder)

    sample_fps = round(avg_fps / 1)
    if sample_fps == 0:
        sample_fps = 1
    frame_indices = [i for i in range(0, total_frames, sample_fps)]
    if frame_count_limit is not None and len(frame_indices) > frame_count_limit:
        frame_indices = uniform_sample(frame_indices, frame_count_limit)

    if not frame_indices:
        return []

    frames_data = decoder.get_frames_at(frame_indices)
    frames = [Image.fromarray(v.astype("uint8")) for v in frames_data]

    return frames
```
**EN:** This function implements `encode_video`. It primarily calls `VideoDecoderWrapper`, `len`, `round`, `decoder.get_frames_at`, `os.path.exists`, `logger.error` to complete its work. State updates are written into `decoder`, `avg_fps`, `total_frames`, `sample_fps`, `frame_indices`, `frames_data`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `encode_video`。 它主要通过调用 `VideoDecoderWrapper`, `len`, `round`, `decoder.get_frames_at`, `os.path.exists`, `logger.error` 来完成任务。 状态更新主要写入 `decoder`, `avg_fps`, `total_frames`, `sample_fps`, `frame_indices`, `frames_data`。 实现中使用了条件分支。

### Lines 1047-1066: Function `suppress_noisy_warnings` / 函数 `suppress_noisy_warnings`
```python
def suppress_noisy_warnings():
    """Suppress known noisy warnings from third-party libraries."""
    warnings.filterwarnings(
        "ignore", category=UserWarning, message="The given NumPy array is not writable"
    )
    warnings.filterwarnings(
        "ignore",
        message="The cuda.cudart module is deprecated",
        category=FutureWarning,
    )
    warnings.filterwarnings(
        "ignore",
        message="The cuda.nvrtc module is deprecated",
        category=FutureWarning,
    )

    # Suppress noisy third-party HTTP loggers.
    # huggingface_hub uses httpx which logs every HTTP request at INFO level.
    for name in ("httpx", "httpcore"):
        logging.getLogger(name).setLevel(logging.WARNING)
```
**EN:** This function implements `suppress_noisy_warnings`. It primarily calls `warnings.filterwarnings`, `logging.getLogger.setLevel`, `logging.getLogger` to complete its work. The implementation relies on iteration.
**CN:** 该函数实现了 `suppress_noisy_warnings`。 它主要通过调用 `warnings.filterwarnings`, `logging.getLogger.setLevel`, `logging.getLogger` 来完成任务。 实现中使用了迭代逻辑。

### Lines 1069-1084: Function `suppress_other_loggers` / 函数 `suppress_other_loggers`
```python
def suppress_other_loggers():
    suppress_noisy_warnings()

    try:
        from vllm.logger import logger as vllm_default_logger
    except ImportError:
        return

    vllm_default_logger.setLevel(logging.WARN)
    logging.getLogger("vllm.distributed.device_communicators.pynccl").setLevel(
        logging.WARN
    )
    logging.getLogger("vllm.distributed.device_communicators.shm_broadcast").setLevel(
        logging.WARN
    )
    logging.getLogger("vllm.config").setLevel(logging.ERROR)
```
**EN:** This function implements `suppress_other_loggers`. It primarily calls `suppress_noisy_warnings`, `vllm_default_logger.setLevel`, `logging.getLogger.setLevel`, `logging.getLogger` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `suppress_other_loggers`。 它主要通过调用 `suppress_noisy_warnings`, `vllm_default_logger.setLevel`, `logging.getLogger.setLevel`, `logging.getLogger` 来完成任务。 实现中使用了错误处理。

### Lines 1087-1099: Function `assert_pkg_version` / 函数 `assert_pkg_version`
```python
def assert_pkg_version(pkg: str, min_version: str, message: str):
    try:
        installed_version = version(pkg)
        if pkg_version.parse(installed_version) < pkg_version.parse(min_version):
            raise Exception(
                f"{pkg} is installed with version {installed_version}, which "
                f"is less than the minimum required version {min_version}. " + message
            )
    except PackageNotFoundError:
        raise Exception(
            f"{pkg} with minimum required version {min_version} is not installed. "
            + message
        )
```
**EN:** This function implements `assert_pkg_version`. It primarily calls `version`, `pkg_version.parse`, `Exception` to complete its work. State updates are written into `installed_version`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `assert_pkg_version`。 它主要通过调用 `version`, `pkg_version.parse`, `Exception` 来完成任务。 状态更新主要写入 `installed_version`。 实现中使用了条件分支、错误处理。

### Lines 1102-1117: Function `check_pkg_version_at_least` / 函数 `check_pkg_version_at_least`
```python
def check_pkg_version_at_least(pkg: str, min_version: str) -> bool:
    """
    Check if a package is installed and meets the minimum version requirement.

    Args:
        pkg: Package name (distribution name, e.g., "flashinfer-python")
        min_version: Minimum version required (e.g., "0.6.11.post1")

    Returns:
        True if package is installed and version >= min_version, False otherwise
    """
    try:
        installed_version = version(pkg)
        return pkg_version.parse(installed_version) >= pkg_version.parse(min_version)
    except PackageNotFoundError:
        return False
```
**EN:** This function implements `check_pkg_version_at_least`. It primarily calls `version`, `pkg_version.parse` to complete its work. State updates are written into `installed_version`. The implementation relies on error handling.
**CN:** 该函数实现了 `check_pkg_version_at_least`。 它主要通过调用 `version`, `pkg_version.parse` 来完成任务。 状态更新主要写入 `installed_version`。 实现中使用了错误处理。

### Lines 1120-1146: Function `_wait_for_reap_or_raise` / 函数 `_wait_for_reap_or_raise`
```python
def _wait_for_reap_or_raise(procs, wait_timeout: float) -> None:
    """Wait for `procs` to exit; warn at ~10s, raise on `wait_timeout`.

    SIGKILL is asynchronous -- children hold GPU context, pinned memory and
    fds until the kernel reaps them. Raise on timeout so a stuck process
    surfaces instead of leaving a latent race.
    """
    warn_at = min(10.0, wait_timeout / 2)
    gone, alive = psutil.wait_procs(procs, timeout=warn_at)
    if not alive:
        return
    logger.warning(
        "kill_process_tree: %d process(es) still alive after %.1fs SIGKILL; "
        "continuing to wait up to %.1fs total. pids=%s",
        len(alive),
        warn_at,
        wait_timeout,
        [p.pid for p in alive],
    )
    remaining = wait_timeout - warn_at
    if remaining > 0:
        _, alive = psutil.wait_procs(alive, timeout=remaining)
    if alive:
        raise RuntimeError(
            f"kill_process_tree: {len(alive)} process(es) not reaped within "
            f"{wait_timeout}s after SIGKILL; pids={[p.pid for p in alive]}"
        )
```
**EN:** This function implements `_wait_for_reap_or_raise`. It primarily calls `min`, `psutil.wait_procs`, `logger.warning`, `len`, `RuntimeError` to complete its work. State updates are written into `warn_at`, `remaining`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_wait_for_reap_or_raise`。 它主要通过调用 `min`, `psutil.wait_procs`, `logger.warning`, `len`, `RuntimeError` 来完成任务。 状态更新主要写入 `warn_at`, `remaining`。 实现中使用了条件分支。

### Lines 1149-1198: Function `kill_process_tree` / 函数 `kill_process_tree`
```python
def kill_process_tree(
    parent_pid,
    include_parent: bool = True,
    skip_pid: int = None,
    wait_timeout: Optional[float] = None,
):
    """Kill the process and all its child processes.

    `wait_timeout` (seconds) blocks until every killed process is reaped and
    raises `RuntimeError` on timeout; `None` is fire-and-forget. The
    `parent_pid == os.getpid()` branch calls `sys.exit(0)` and cannot wait
    for itself -- use `include_parent=False` if child reap must finish first.
    """
    if parent_pid is None:
        parent_pid = os.getpid()
        include_parent = False

    try:
        itself = psutil.Process(parent_pid)
    except psutil.NoSuchProcess:
        return

    children = itself.children(recursive=True)
    killed = []
    for child in children:
        if child.pid == skip_pid:
            continue
        try:
# ... omitted for brevity ...
            killed.append(itself)
        except psutil.NoSuchProcess:
            pass

    if wait_timeout is not None and killed:
        _wait_for_reap_or_raise(killed, wait_timeout)
```
**EN:** This function implements `kill_process_tree`. It primarily calls `itself.children`, `os.getpid`, `psutil.Process`, `_wait_for_reap_or_raise`, `child.kill`, `killed.append` to complete its work. State updates are written into `children`, `killed`, `parent_pid`, `include_parent`, `itself`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `kill_process_tree`。 它主要通过调用 `itself.children`, `os.getpid`, `psutil.Process`, `_wait_for_reap_or_raise`, `child.kill`, `killed.append` 来完成任务。 状态更新主要写入 `children`, `killed`, `parent_pid`, `include_parent`, `itself`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 1201-1216: Function `monkey_patch_p2p_access_check` / 函数 `monkey_patch_p2p_access_check`
```python
def monkey_patch_p2p_access_check():
    """
    Monkey patch the slow p2p access check.
    NOTE: We assume the p2p access is always allowed, which can be wrong for some setups.
    """

    import sglang.srt.distributed.device_communicators.custom_all_reduce_utils as tgt

    setattr(tgt, "gpu_p2p_access_check", lambda *arg, **kwargs: True)

    # Suppress the warnings from this delete function when using sglang.bench_one_batch
    from sglang.srt.distributed.device_communicators.custom_all_reduce import (
        CustomAllreduce,
    )

    setattr(CustomAllreduce, "__del__", lambda *args, **kwargs: None)
```
**EN:** This function implements `monkey_patch_p2p_access_check`. It primarily calls `setattr` to complete its work.
**CN:** 该函数实现了 `monkey_patch_p2p_access_check`。 它主要通过调用 `setattr` 来完成任务。

### Lines 1219-1240: Function `set_ulimit` / 函数 `set_ulimit`
```python
def set_ulimit(target_soft_limit=65535):
    # number of open files
    resource_type = resource.RLIMIT_NOFILE
    current_soft, current_hard = resource.getrlimit(resource_type)

    if current_soft < target_soft_limit:
        try:
            resource.setrlimit(resource_type, (target_soft_limit, current_hard))
        except ValueError as e:
            logger.warning(f"Fail to set RLIMIT_NOFILE: {e}")

    # stack size
    resource_type = resource.RLIMIT_STACK
    current_soft, current_hard = resource.getrlimit(resource_type)
    target_soft_limit_stack_size = 1024 * target_soft_limit
    if current_soft < target_soft_limit_stack_size:
        try:
            resource.setrlimit(
                resource_type, (target_soft_limit_stack_size, current_hard)
            )
        except ValueError as e:
            logger.warning(f"Fail to set RLIMIT_STACK: {e}")
```
**EN:** This function implements `set_ulimit`. It primarily calls `resource.getrlimit`, `resource.setrlimit`, `logger.warning` to complete its work. State updates are written into `resource_type`, `target_soft_limit_stack_size`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `set_ulimit`。 它主要通过调用 `resource.getrlimit`, `resource.setrlimit`, `logger.warning` 来完成任务。 状态更新主要写入 `resource_type`, `target_soft_limit_stack_size`。 实现中使用了条件分支、错误处理。

### Lines 1243-1250: Function `rank0_log` / 函数 `rank0_log`
```python
def rank0_log(msg: str):
    from sglang.srt.distributed import (
        get_tensor_model_parallel_rank,
        model_parallel_is_initialized,
    )

    if not model_parallel_is_initialized() or get_tensor_model_parallel_rank() == 0:
        logger.info(msg)
```
**EN:** This function implements `rank0_log`. It primarily calls `logger.info`, `model_parallel_is_initialized`, `get_tensor_model_parallel_rank` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `rank0_log`。 它主要通过调用 `logger.info`, `model_parallel_is_initialized`, `get_tensor_model_parallel_rank` 来完成任务。 实现中使用了条件分支。

### Lines 1253-1282: Function `configure_logger` / 函数 `configure_logger`
```python
def configure_logger(server_args, prefix: str = ""):
    if SGLANG_LOGGING_CONFIG_PATH := os.getenv("SGLANG_LOGGING_CONFIG_PATH"):
        if not os.path.exists(SGLANG_LOGGING_CONFIG_PATH):
            raise Exception(
                "Setting SGLANG_LOGGING_CONFIG_PATH from env with "
                f"{SGLANG_LOGGING_CONFIG_PATH} but it does not exist!"
            )
        with open(SGLANG_LOGGING_CONFIG_PATH, encoding="utf-8") as file:
            custom_config = orjson.loads(file.read())
        logging.config.dictConfig(custom_config)
        return
    maybe_ms = ".%(msecs)03d" if envs.SGLANG_LOG_MS.get() else ""
    format = f"[%(asctime)s{maybe_ms}{prefix}] %(message)s"
    logging.basicConfig(
        level=getattr(logging, server_args.log_level.upper()),
        format=format,
        datefmt="%Y-%m-%d %H:%M:%S",
        force=True,
    )

    # Suppress noisy httpx/httpcore loggers in every process that calls
    # configure_logger (main, scheduler, detokenizer). Spawned subprocesses
    # don't inherit the parent's logger state, so this must run here too.
    for name in ("httpx", "httpcore"):
        logging.getLogger(name).setLevel(logging.WARNING)

    if is_flashinfer_available():
        from flashinfer.jit.core import logger as flashinfer_logger

        flashinfer_logger.setLevel(logging.ERROR)
```
**EN:** This function implements `configure_logger`. It primarily calls `logging.basicConfig`, `is_flashinfer_available`, `os.getenv`, `logging.config.dictConfig`, `envs.SGLANG_LOG_MS.get`, `logging.getLogger.setLevel` to complete its work. State updates are written into `maybe_ms`, `format`, `custom_config`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `configure_logger`。 它主要通过调用 `logging.basicConfig`, `is_flashinfer_available`, `os.getenv`, `logging.config.dictConfig`, `envs.SGLANG_LOG_MS.get`, `logging.getLogger.setLevel` 来完成任务。 状态更新主要写入 `maybe_ms`, `format`, `custom_config`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 1286-1293: Function `replace_submodule` / 函数 `replace_submodule`
```python
def replace_submodule(
    model: nn.Module, module_name: str, new_module: nn.Module
) -> nn.Module:
    """Replace a submodule in a model with a new module."""
    parent = model.get_submodule(".".join(module_name.split(".")[:-1]))
    target_name = module_name.split(".")[-1]
    setattr(parent, target_name, new_module)
    return new_module
```
**EN:** This function implements `replace_submodule`. It primarily calls `model.get_submodule`, `setattr`, `join`, `module_name.split` to complete its work. State updates are written into `parent`, `target_name`.
**CN:** 该函数实现了 `replace_submodule`。 它主要通过调用 `model.get_submodule`, `setattr`, `join`, `module_name.split` 来完成任务。 状态更新主要写入 `parent`, `target_name`。

### Lines 1296-1313: Function `set_weight_attrs` / 函数 `set_weight_attrs`
```python
def set_weight_attrs(
    weight: torch.Tensor,
    weight_attrs: Optional[Dict[str, Any]],
):
    """Set attributes on a weight tensor.

    This method is used to set attributes on a weight tensor. This method
    will not overwrite existing attributes.

    Args:
        weight: The weight tensor.
        weight_attrs: A dictionary of attributes to set on the weight tensor.
    """
    if weight_attrs is None:
        return
    for key, value in weight_attrs.items():
        assert not hasattr(weight, key), f"Overwriting existing tensor attribute: {key}"
        setattr(weight, key, value)
```
**EN:** This function implements `set_weight_attrs`. It primarily calls `weight_attrs.items`, `setattr`, `hasattr` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `set_weight_attrs`。 它主要通过调用 `weight_attrs.items`, `setattr`, `hasattr` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 1316-1362: Function `broadcast_pyobj` / 函数 `broadcast_pyobj`
```python
def broadcast_pyobj(
    data: List[Any],
    rank: int,
    dist_group: Optional[torch.distributed.ProcessGroup] = None,
    src: int = 0,
    force_cpu_device: bool = True,
):
    """Broadcast inputs from src rank to all other ranks with torch.dist backend.
    The `rank` here refer to the source rank on global process group (regardless
    of dist_group argument).
    """
    device = torch.device(
        "cuda"
        if torch.cuda.is_available() and not force_cpu_device
        else "musa" if is_musa() and not force_cpu_device else "cpu"
    )

    if rank == src:
        if len(data) == 0:
            tensor_size = torch.tensor([0], dtype=torch.long, device=device)
            dist.broadcast(tensor_size, src=src, group=dist_group)
        else:
            serialized_data = pickle.dumps(data)
            size = len(serialized_data)

            tensor_data = torch.ByteTensor(
                np.frombuffer(serialized_data, dtype=np.uint8)
            ).to(device)
# ... omitted for brevity ...
        tensor_data = torch.empty(size, dtype=torch.uint8, device=device)
        dist.broadcast(tensor_data, src=src, group=dist_group)

        serialized_data = bytes(tensor_data.cpu().numpy())
        data = pickle.loads(serialized_data)
        return data
```
**EN:** This function implements `broadcast_pyobj`. It primarily calls `torch.device`, `torch.tensor`, `dist.broadcast`, `tensor_size.item`, `torch.empty`, `bytes` to complete its work. State updates are written into `device`, `tensor_size`, `size`, `tensor_data`, `serialized_data`, `data`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `broadcast_pyobj`。 它主要通过调用 `torch.device`, `torch.tensor`, `dist.broadcast`, `tensor_size.item`, `torch.empty`, `bytes` 来完成任务。 状态更新主要写入 `device`, `tensor_size`, `size`, `tensor_data`, `serialized_data`, `data`。 实现中使用了条件分支。

### Lines 1365-1430: Function `point_to_point_pyobj` / 函数 `point_to_point_pyobj`
```python
def point_to_point_pyobj(
    data: List[Any],
    rank: int,
    group: Optional[torch.distributed.ProcessGroup] = None,
    src: int = 0,
    dst: int = 1,
    async_send: bool = False,
):
    """Send data from src to dst in group."""
    from sglang.srt.distributed.parallel_state import P2PWork

    if async_send:
        send_func = dist.isend
    else:
        send_func = dist.send
    if rank == src:
        p2p_works = []
        if len(data) == 0:
            tensor_size = torch.tensor(
                [0],
                dtype=torch.long,
            )
            work = send_func(tensor_size, dst, group=group)
            if async_send:
                p2p_works.append(P2PWork(work, tensor_size))
        else:
            serialized_data = pickle.dumps(data)
            size = len(serialized_data)
# ... omitted for brevity ...
        serialized_data = bytes(tensor_data.cpu().numpy())
        data = pickle.loads(serialized_data)
        return data

    # Other ranks in pp_group do nothing
    return []
```
**EN:** This function implements `point_to_point_pyobj`. It primarily calls `len`, `torch.tensor`, `send_func`, `pickle.dumps`, `torch.ByteTensor`, `dist.irecv` to complete its work. State updates are written into `send_func`, `p2p_works`, `tensor_size`, `work`, `serialized_data`, `size`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `point_to_point_pyobj`。 它主要通过调用 `len`, `torch.tensor`, `send_func`, `pickle.dumps`, `torch.ByteTensor`, `dist.irecv` 来完成任务。 状态更新主要写入 `send_func`, `p2p_works`, `tensor_size`, `work`, `serialized_data`, `size`。 实现中使用了条件分支。

### Lines 1433-1438: Function `delete_directory` / 函数 `delete_directory`
```python
def delete_directory(dirpath):
    try:
        # This will remove the directory and all its contents
        shutil.rmtree(dirpath)
    except OSError as e:
        print(f"Warning: {dirpath} : {e.strerror}")
```
**EN:** This function implements `delete_directory`. It primarily calls `shutil.rmtree`, `print` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `delete_directory`。 它主要通过调用 `shutil.rmtree`, `print` 来完成任务。 实现中使用了错误处理。

### Lines 1443-1443: Constants and shared state / 常量与共享状态
```python
prometheus_multiproc_dir: tempfile.TemporaryDirectory
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `prometheus_multiproc_dir`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `prometheus_multiproc_dir`。

### Lines 1446-1461: Function `set_prometheus_multiproc_dir` / 函数 `set_prometheus_multiproc_dir`
```python
def set_prometheus_multiproc_dir():
    # Set prometheus multiprocess directory
    # sglang uses prometheus multiprocess mode
    # we need to set this before importing prometheus_client
    # https://prometheus.github.io/client_python/multiprocess/
    global prometheus_multiproc_dir

    if "PROMETHEUS_MULTIPROC_DIR" in os.environ:
        logger.debug("User set PROMETHEUS_MULTIPROC_DIR detected.")
        prometheus_multiproc_dir = tempfile.TemporaryDirectory(
            dir=os.environ["PROMETHEUS_MULTIPROC_DIR"]
        )
    else:
        prometheus_multiproc_dir = tempfile.TemporaryDirectory()
        os.environ["PROMETHEUS_MULTIPROC_DIR"] = prometheus_multiproc_dir.name
    logger.debug(f"PROMETHEUS_MULTIPROC_DIR: {os.environ['PROMETHEUS_MULTIPROC_DIR']}")
```
**EN:** This function implements `set_prometheus_multiproc_dir`. It primarily calls `logger.debug`, `tempfile.TemporaryDirectory` to complete its work. State updates are written into `prometheus_multiproc_dir`, `os.environ`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `set_prometheus_multiproc_dir`。 它主要通过调用 `logger.debug`, `tempfile.TemporaryDirectory` 来完成任务。 状态更新主要写入 `prometheus_multiproc_dir`, `os.environ`。 实现中使用了条件分支。

### Lines 1464-1474: Function `add_prometheus_middleware` / 函数 `add_prometheus_middleware`
```python
def add_prometheus_middleware(app):
    # We need to import prometheus_client after setting the env variable `PROMETHEUS_MULTIPROC_DIR`
    from prometheus_client import CollectorRegistry, make_asgi_app, multiprocess

    registry = CollectorRegistry()
    multiprocess.MultiProcessCollector(registry)
    metrics_route = Mount("/metrics", make_asgi_app(registry=registry))

    # Workaround for 307 Redirect for /metrics
    metrics_route.path_regex = re.compile("^/metrics(?P<path>.*)$")
    app.routes.append(metrics_route)
```
**EN:** This function implements `add_prometheus_middleware`. It primarily calls `CollectorRegistry`, `multiprocess.MultiProcessCollector`, `Mount`, `re.compile`, `app.routes.append`, `make_asgi_app` to complete its work. State updates are written into `registry`, `metrics_route`, `metrics_route.path_regex`.
**CN:** 该函数实现了 `add_prometheus_middleware`。 它主要通过调用 `CollectorRegistry`, `multiprocess.MultiProcessCollector`, `Mount`, `re.compile`, `app.routes.append`, `make_asgi_app` 来完成任务。 状态更新主要写入 `registry`, `metrics_route`, `metrics_route.path_regex`。

### Lines 1477-1477: Class `RefCountedGauge` declaration / 类 `RefCountedGauge` 声明
```python
class RefCountedGauge:
```
**EN:** This class establishes `RefCountedGauge` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `inc`, `dec`.
**CN:** 该类将 `RefCountedGauge` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `inc`, `dec` 等方法。

### Lines 1478-1480: Method `RefCountedGauge.__init__` / 方法 `RefCountedGauge.__init__`
```python
    def __init__(self, gauge):
        self._gauge = gauge
        self._refcount: Dict[str, int] = {}
```
**EN:** This method implements `__init__` on `RefCountedGauge`. State updates are written into `self._gauge`, `self._refcount`.
**CN:** 该方法（属于 `RefCountedGauge`）实现了 `__init__`。 状态更新主要写入 `self._gauge`, `self._refcount`。

### Lines 1482-1487: Method `RefCountedGauge.inc` / 方法 `RefCountedGauge.inc`
```python
    def inc(self, key: str):
        if key in self._refcount:
            self._refcount[key] += 1
        else:
            self._refcount[key] = 1
            self._gauge.inc()
```
**EN:** This method implements `inc` on `RefCountedGauge`. It primarily calls `self._gauge.inc` to complete its work. State updates are written into `self._refcount`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RefCountedGauge`）实现了 `inc`。 它主要通过调用 `self._gauge.inc` 来完成任务。 状态更新主要写入 `self._refcount`。 实现中使用了条件分支。

### Lines 1489-1494: Method `RefCountedGauge.dec` / 方法 `RefCountedGauge.dec`
```python
    def dec(self, key: str):
        if key in self._refcount:
            self._refcount[key] -= 1
            if self._refcount[key] == 0:
                del self._refcount[key]
                self._gauge.dec()
```
**EN:** This method implements `dec` on `RefCountedGauge`. It primarily calls `self._gauge.dec` to complete its work. State updates are written into `self._refcount`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `RefCountedGauge`）实现了 `dec`。 它主要通过调用 `self._gauge.dec` 来完成任务。 状态更新主要写入 `self._refcount`。 实现中使用了条件分支。

### Lines 1497-1559: Function `add_prometheus_track_response_middleware` / 函数 `add_prometheus_track_response_middleware`
```python
def add_prometheus_track_response_middleware(app):
    from prometheus_client import Counter, Gauge

    http_request_counter = Counter(
        name="sglang:http_requests_total",
        documentation="Total number of HTTP requests by endpoint and method",
        labelnames=["endpoint", "method"],
    )

    http_response_counter = Counter(
        name="sglang:http_responses_total",
        documentation="Total number of HTTP responses by endpoint and status code",
        labelnames=["endpoint", "status_code", "method"],
    )

    http_requests_active = Gauge(
        name="sglang:http_requests_active",
        documentation="Number of currently active HTTP requests",
        labelnames=["endpoint", "method"],
        multiprocess_mode="livesum",
    )

    routing_keys_active = RefCountedGauge(
        Gauge(
            name="sglang:routing_keys_active",
            documentation="Number of unique routing keys with active requests",
            multiprocess_mode="livesum",
        )
# ... omitted for brevity ...

            return response
        finally:
            http_requests_active.labels(endpoint=path, method=method).dec()
            if routing_key:
                routing_keys_active.dec(routing_key)
```
**EN:** This function implements `add_prometheus_track_response_middleware`. It primarily calls `Counter`, `Gauge`, `RefCountedGauge`, `patch_app_http_middleware`, `app.middleware`, `_get_fastapi_request_path` to complete its work. State updates are written into `http_request_counter`, `http_response_counter`, `http_requests_active`, `routing_keys_active`, `method`, `routing_key`. The implementation relies on conditional branches, error handling, async awaits.
**CN:** 该函数实现了 `add_prometheus_track_response_middleware`。 它主要通过调用 `Counter`, `Gauge`, `RefCountedGauge`, `patch_app_http_middleware`, `app.middleware`, `_get_fastapi_request_path` 来完成任务。 状态更新主要写入 `http_request_counter`, `http_response_counter`, `http_requests_active`, `routing_keys_active`, `method`, `routing_key`。 实现中使用了条件分支、错误处理、异步等待。

### Lines 1563-1571: Function `_get_fastapi_request_path` / 函数 `_get_fastapi_request_path`
```python
def _get_fastapi_request_path(request) -> Tuple[str, bool]:
    from starlette.routing import Match

    for route in request.app.routes:
        match, child_scope = route.matches(request.scope)
        if match == Match.FULL:
            return route.path, True

    return request.url.path, False
```
**EN:** This function implements `_get_fastapi_request_path`. It primarily calls `route.matches` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_get_fastapi_request_path`。 它主要通过调用 `route.matches` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 1574-1604: Function `get_amdgpu_memory_capacity` / 函数 `get_amdgpu_memory_capacity`
```python
def get_amdgpu_memory_capacity():
    try:
        # Run rocm-smi and capture the output
        result = subprocess.run(
            [
                "rocminfo | grep 'gfx' -A 100 | grep 'Pool 1' -A 5 | grep 'Size:' | awk '{print $2}'"
            ],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            shell=True,
            text=True,
        )
        if result.returncode != 0:
            raise RuntimeError(f"rocm-smi error: {result.stderr.strip()}")

        # Parse the output to extract memory values in MiB
        memory_values = [
            float(mem.split("(")[0].strip()) / 1024
            for mem in result.stdout.strip().split("\n")
        ]

        if not memory_values:
            raise ValueError("No GPU memory values found.")

        # Return the minimum memory value
        return min(memory_values)

    except FileNotFoundError:
        raise RuntimeError(
            "rocm-smi not found. Ensure AMD ROCm drivers are installed and accessible."
        )
```
**EN:** This function implements `get_amdgpu_memory_capacity`. It primarily calls `subprocess.run`, `min`, `RuntimeError`, `ValueError`, `float`, `result.stdout.strip.split` to complete its work. State updates are written into `result`, `memory_values`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_amdgpu_memory_capacity`。 它主要通过调用 `subprocess.run`, `min`, `RuntimeError`, `ValueError`, `float`, `result.stdout.strip.split` 来完成任务。 状态更新主要写入 `result`, `memory_values`。 实现中使用了条件分支、错误处理。

### Lines 1607-1611: Function `get_device_sm` / 函数 `get_device_sm`
```python
def get_device_sm():
    if torch.cuda.is_available() or is_musa():
        major, minor = torch.cuda.get_device_capability()
        return major * 10 + minor
    return 0
```
**EN:** This function implements `get_device_sm`. It primarily calls `torch.cuda.is_available`, `is_musa`, `torch.cuda.get_device_capability` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_device_sm`。 它主要通过调用 `torch.cuda.is_available`, `is_musa`, `torch.cuda.get_device_capability` 来完成任务。 实现中使用了条件分支。

### Lines 1614-1644: Function `_cuda_mem_fallback` / 函数 `_cuda_mem_fallback`
```python
def _cuda_mem_fallback(reason: str) -> int:
    """Fallback to torch.cuda.mem_get_info() and return total GPU memory in MiB.

    Queries all visible CUDA devices and returns the minimum total memory,
    consistent with the nvidia-smi path that takes min(memory_values).

    Returns the total memory in MiB, or raises RuntimeError if CUDA is
    unavailable or mem_get_info() fails.
    """
    if not torch.cuda.is_available():
        raise RuntimeError(reason)
    try:
        device_count = torch.cuda.device_count()
        if device_count == 0:
            # Include the original failure reason for diagnostics
            raise RuntimeError(f"{reason} No CUDA devices found via torch.cuda.")
        memory_values = []
        for i in range(device_count):
            total = torch.cuda.mem_get_info(i)[1] // 1024 // 1024  # unit: MiB
            memory_values.append(total)
        result = min(memory_values)
        logger.warning(
            f"{reason} Falling back to torch.cuda.mem_get_info(). "
            f"Reported total GPU memory per device (MiB): {memory_values}, "
            f"using min: {result} MiB."
        )
        return result
    except (RuntimeError, ValueError, OSError) as e:
        raise RuntimeError(
            f"{reason} torch.cuda.mem_get_info() fallback also failed: {e}"
        ) from e
```
**EN:** This function implements `_cuda_mem_fallback`. It primarily calls `torch.cuda.is_available`, `RuntimeError`, `torch.cuda.device_count`, `range`, `min`, `logger.warning` to complete its work. State updates are written into `device_count`, `memory_values`, `result`, `total`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_cuda_mem_fallback`。 它主要通过调用 `torch.cuda.is_available`, `RuntimeError`, `torch.cuda.device_count`, `range`, `min`, `logger.warning` 来完成任务。 状态更新主要写入 `device_count`, `memory_values`, `result`, `total`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 1647-1682: Function `get_nvgpu_memory_capacity` / 函数 `get_nvgpu_memory_capacity`
```python
def get_nvgpu_memory_capacity():
    try:
        # Run nvidia-smi and capture the output
        result = subprocess.run(
            ["nvidia-smi", "--query-gpu=memory.total", "--format=csv,noheader,nounits"],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
        )

        if result.returncode != 0:
            return _cuda_mem_fallback(
                f"nvidia-smi failed (exit code {result.returncode}: {result.stderr.strip()})."
            )

        # Parse the output to extract memory values
        memory_values = [
            float(mem)
            for mem in result.stdout.strip().split("\n")
            if re.match(r"^\d+(\.\d+)?$", mem.strip())
        ]

        if not memory_values:
            # Fallback when nvidia-smi returns no parseable values,
            # typically in NVIDIA MIG mode.
            return _cuda_mem_fallback(
                "Failed to get GPU memory capacity from nvidia-smi."
            )

        # Return the minimum memory value
        return min(memory_values)

    except FileNotFoundError:
        return _cuda_mem_fallback(
            "nvidia-smi not found. Ensure NVIDIA drivers are installed and accessible."
        )
```
**EN:** This function implements `get_nvgpu_memory_capacity`. It primarily calls `subprocess.run`, `min`, `_cuda_mem_fallback`, `float`, `result.stdout.strip.split`, `re.match` to complete its work. State updates are written into `result`, `memory_values`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_nvgpu_memory_capacity`。 它主要通过调用 `subprocess.run`, `min`, `_cuda_mem_fallback`, `float`, `result.stdout.strip.split`, `re.match` 来完成任务。 状态更新主要写入 `result`, `memory_values`。 实现中使用了条件分支、错误处理。

### Lines 1685-1713: Function `get_hpu_memory_capacity` / 函数 `get_hpu_memory_capacity`
```python
def get_hpu_memory_capacity():
    try:
        # Run hl-smi and capture the output
        result = subprocess.run(
            ["hl-smi --query | grep 'Total'"],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            shell=True,
            text=True,
        )

        if result.returncode != 0:
            raise RuntimeError(f"hl-smi error: {result.stderr.strip()}")

        # Parse the output to extract memory values in MiB
        memory_values = [
            float(mem.split(" ")[-2]) for mem in result.stdout.strip().split("\n")
        ]

        if not memory_values:
            raise ValueError("No GPU memory values found.")

        # Return the minimum memory value
        return min(memory_values)

    except FileNotFoundError:
        raise RuntimeError(
            "hl-smi not found. Ensure Habana drivers are installed and accessible."
        )
```
**EN:** This function implements `get_hpu_memory_capacity`. It primarily calls `subprocess.run`, `min`, `RuntimeError`, `float`, `ValueError`, `result.stdout.strip.split` to complete its work. State updates are written into `result`, `memory_values`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_hpu_memory_capacity`。 它主要通过调用 `subprocess.run`, `min`, `RuntimeError`, `float`, `ValueError`, `result.stdout.strip.split` 来完成任务。 状态更新主要写入 `result`, `memory_values`。 实现中使用了条件分支、错误处理。

### Lines 1716-1725: Function `get_npu_memory_capacity` / 函数 `get_npu_memory_capacity`
```python
def get_npu_memory_capacity():
    try:
        import torch_npu  # noqa: F401

        if envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get() > 0:
            return envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get()  # unit: MB
        else:
            return torch.npu.mem_get_info()[1] // 1024 // 1024  # unit: MB
    except ImportError as e:
        raise ImportError("torch_npu is required when run on npu device.")
```
**EN:** This function implements `get_npu_memory_capacity`. It primarily calls `envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get`, `ImportError`, `torch.npu.mem_get_info` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_npu_memory_capacity`。 它主要通过调用 `envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get`, `ImportError`, `torch.npu.mem_get_info` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 1728-1756: Function `get_cpu_memory_capacity` / 函数 `get_cpu_memory_capacity`
```python
def get_cpu_memory_capacity():
    # Per-rank memory capacity cannot be determined for customized core settings
    if os.environ.get("SGLANG_CPU_OMP_THREADS_BIND", ""):
        return None
    n_numa_node: int = len(get_cpu_ids_by_node())
    if n_numa_node == 0:
        # Cannot determine NUMA config, fallback to total memory and avoid ZeroDivisionError.
        return float(psutil.virtual_memory().total // (1 << 20))
    try:
        numa_mem_list = list()
        file_prefix = "/sys/devices/system/node/"
        for numa_id in range(n_numa_node):
            file_meminfo = f"node{numa_id}/meminfo"
            with open(os.path.join(file_prefix, file_meminfo), "r") as f:
                # MemTotal info is at the 1st line
                line = f.readline()
                # Expected format: "Node 0 MemTotal:       100000000 kB"
                parts = line.split()
                if len(parts) >= 4 and parts[2] == "MemTotal:":
                    numa_mem_list.append(int(parts[3]))
                else:
                    raise ValueError(f"Unexpected format in {file_meminfo}: {line}")
        # Retrieved value in KB, need MB
        numa_mem = float(min(numa_mem_list) // 1024)
        return numa_mem
    except (FileNotFoundError, ValueError, IndexError):
        numa_mem = psutil.virtual_memory().total / n_numa_node
        # Retrieved value in Byte, need MB
        return float(numa_mem // (1 << 20))
```
**EN:** This function implements `get_cpu_memory_capacity`. It primarily calls `os.environ.get`, `len`, `get_cpu_ids_by_node`, `float`, `list`, `range` to complete its work. State updates are written into `n_numa_node`, `numa_mem_list`, `file_prefix`, `numa_mem`, `file_meminfo`, `line`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `get_cpu_memory_capacity`。 它主要通过调用 `os.environ.get`, `len`, `get_cpu_ids_by_node`, `float`, `list`, `range` 来完成任务。 状态更新主要写入 `n_numa_node`, `numa_mem_list`, `file_prefix`, `numa_mem`, `file_meminfo`, `line`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 1759-1765: Function `get_xpu_memory_capacity` / 函数 `get_xpu_memory_capacity`
```python
def get_xpu_memory_capacity():
    try:
        if torch.xpu.is_available():
            return torch.xpu.mem_get_info()[1] // 1024 // 1024  # unit: MB
        raise ValueError("No GPU memory values found.")
    except AttributeError:
        raise RuntimeError("torch.xpu is not available.")
```
**EN:** This function implements `get_xpu_memory_capacity`. It primarily calls `torch.xpu.is_available`, `ValueError`, `RuntimeError`, `torch.xpu.mem_get_info` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_xpu_memory_capacity`。 它主要通过调用 `torch.xpu.is_available`, `ValueError`, `RuntimeError`, `torch.xpu.mem_get_info` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 1768-1806: Function `get_mtgpu_memory_capacity` / 函数 `get_mtgpu_memory_capacity`
```python
def get_mtgpu_memory_capacity():
    try:
        # Run mthreads-gmi and capture the output
        result = subprocess.run(
            [
                "mthreads-gmi --query | grep 'FB Memory Usage' -A 2 | grep 'Total' | awk -F':' '{print $2}' | awk '{print $1}' | sed 's/MiB//'"
            ],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            shell=True,
            text=True,
        )

        if result.returncode != 0:
            raise RuntimeError(f"mthreads-gmi error: {result.stderr.strip()}")

        # Parse the output to extract memory values
        memory_values = [
            float(mem)
            for mem in result.stdout.strip().split("\n")
            if re.match(r"^\d+(\.\d+)?$", mem.strip())
        ]

        if not memory_values:
            # Fallback to torch.musa.mem_get_info() when failed to get memory capacity from mthreads-gmi.
            if hasattr(torch, "musa") and torch.musa.is_available():
                logger.warning(
                    "Failed to get GPU memory capacity from mthreads-gmi, falling back to torch.musa.mem_get_info()."
# ... omitted for brevity ...
        return min(memory_values)

    except FileNotFoundError:
        raise RuntimeError(
            "mthreads-gmi not found. Ensure Moore Threads drivers are installed and accessible."
        )
```
**EN:** This function implements `get_mtgpu_memory_capacity`. It primarily calls `subprocess.run`, `min`, `RuntimeError`, `float`, `ValueError`, `result.stdout.strip.split` to complete its work. State updates are written into `result`, `memory_values`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_mtgpu_memory_capacity`。 它主要通过调用 `subprocess.run`, `min`, `RuntimeError`, `float`, `ValueError`, `result.stdout.strip.split` 来完成任务。 状态更新主要写入 `result`, `memory_values`。 实现中使用了条件分支、错误处理。

### Lines 1809-1836: Function `get_device_memory_capacity` / 函数 `get_device_memory_capacity`
```python
def get_device_memory_capacity(device: str = None):
    # OOT platforms provide their own memory query via the platform class.
    from sglang.srt.platforms import current_platform

    if current_platform.is_out_of_tree():
        mem_bytes = current_platform.get_device_total_memory()
        if mem_bytes:
            return mem_bytes / (1 << 20)  # bytes -> MiB
        return None
    if is_cuda():
        gpu_mem = get_nvgpu_memory_capacity()
    elif is_hip():
        gpu_mem = get_amdgpu_memory_capacity()
    elif device == "hpu":
        gpu_mem = get_hpu_memory_capacity()
    elif device == "npu":
        gpu_mem = get_npu_memory_capacity()
    elif device == "cpu":
        gpu_mem = get_cpu_memory_capacity()
    elif device == "xpu":
        gpu_mem = get_xpu_memory_capacity()
    elif device == "musa":
        gpu_mem = get_mtgpu_memory_capacity()
    else:
        # GPU memory is not known yet or no GPU is available.
        gpu_mem = None

    return gpu_mem
```
**EN:** This function implements `get_device_memory_capacity`. It primarily calls `current_platform.is_out_of_tree`, `is_cuda`, `current_platform.get_device_total_memory`, `get_nvgpu_memory_capacity`, `is_hip`, `get_amdgpu_memory_capacity` to complete its work. State updates are written into `mem_bytes`, `gpu_mem`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_device_memory_capacity`。 它主要通过调用 `current_platform.is_out_of_tree`, `is_cuda`, `current_platform.get_device_total_memory`, `get_nvgpu_memory_capacity`, `is_hip`, `get_amdgpu_memory_capacity` 来完成任务。 状态更新主要写入 `mem_bytes`, `gpu_mem`。 实现中使用了条件分支。

### Lines 1842-1910: Function `init_custom_process_group` / 函数 `init_custom_process_group`
```python
def init_custom_process_group(
    backend=None,
    init_method=None,
    timeout=None,
    world_size=-1,
    rank=-1,
    store=None,
    group_name=None,
    pg_options=None,
    device_id=None,
):
    from torch.distributed.distributed_c10d import (
        Backend,
        PrefixStore,
        _new_process_group_helper,
        _world,
        default_pg_timeout,
        rendezvous,
    )

    assert (store is None) or (
        init_method is None
    ), "Cannot specify both init_method and store."

    if store is not None:
        assert world_size > 0, "world_size must be positive if using store"
        assert rank >= 0, "rank must be non-negative if using store"
    elif init_method is None:
# ... omitted for brevity ...
        device_id=device_id,
    )

    _world.pg_group_ranks[pg] = {i: i for i in range(world_size)}

    return pg
```
**EN:** This function implements `init_custom_process_group`. It primarily calls `_new_process_group_helper`, `Backend`, `rendezvous`, `next`, `store.set_timeout`, `PrefixStore` to complete its work. State updates are written into `pg_options_param_name`, `_world.pg_group_ranks`, `backend`, `timeout`, `rendezvous_iterator`, `store`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `init_custom_process_group`。 它主要通过调用 `_new_process_group_helper`, `Backend`, `rendezvous`, `next`, `store.set_timeout`, `PrefixStore` 来完成任务。 状态更新主要写入 `pg_options_param_name`, `_world.pg_group_ranks`, `backend`, `timeout`, `rendezvous_iterator`, `store`。 实现中使用了条件分支。

### Lines 1913-1915: Function `crash_on_warnings` / 函数 `crash_on_warnings`
```python
def crash_on_warnings():
    # Crash on warning if we are running CI tests
    return get_bool_env_var("SGLANG_IS_IN_CI")
```
**EN:** This function implements `crash_on_warnings`. It primarily calls `get_bool_env_var` to complete its work.
**CN:** 该函数实现了 `crash_on_warnings`。 它主要通过调用 `get_bool_env_var` 来完成任务。

### Lines 1918-1921: Function `print_warning_once` / 函数 `print_warning_once`
```python
@functools.lru_cache(None)
def print_warning_once(msg: str) -> None:
    # Set the stacklevel to 2 to print the caller's line info
    logger.warning(msg)
```
**EN:** This function implements `print_warning_once`. It primarily calls `functools.lru_cache`, `logger.warning` to complete its work.
**CN:** 该函数实现了 `print_warning_once`。 它主要通过调用 `functools.lru_cache`, `logger.warning` 来完成任务。

### Lines 1924-1926: Function `print_info_once` / 函数 `print_info_once`
```python
@functools.lru_cache(None)
def print_info_once(msg: str) -> None:
    logger.info(msg)
```
**EN:** This function implements `print_info_once`. It primarily calls `functools.lru_cache`, `logger.info` to complete its work.
**CN:** 该函数实现了 `print_info_once`。 它主要通过调用 `functools.lru_cache`, `logger.info` 来完成任务。

### Lines 1929-1940: Function `get_device_name` / 函数 `get_device_name`
```python
def get_device_name(device_id: int = 0) -> str:
    if (hasattr(torch, "cuda") and torch.cuda.is_available()) or is_musa():
        return torch.cuda.get_device_name(device_id)

    if hasattr(torch, "xpu") and torch.xpu.is_available():
        return torch.xpu.get_device_name(device_id)

    if hasattr(torch, "hpu") and torch.hpu.is_available():
        return torch.hpu.get_device_name(device_id)

    if hasattr(torch, "npu") and torch.npu.is_available():
        return torch.npu.get_device_name(device_id)
```
**EN:** This function implements `get_device_name`. It primarily calls `is_musa`, `torch.cuda.get_device_name`, `hasattr`, `torch.xpu.is_available`, `torch.xpu.get_device_name`, `torch.hpu.is_available` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_device_name`。 它主要通过调用 `is_musa`, `torch.cuda.get_device_name`, `hasattr`, `torch.xpu.is_available`, `torch.xpu.get_device_name`, `torch.hpu.is_available` 来完成任务。 实现中使用了条件分支。

### Lines 1943-1945: Function `is_habana_available` / 函数 `is_habana_available`
```python
@lru_cache(maxsize=1)
def is_habana_available() -> bool:
    return find_spec("habana_frameworks") is not None
```
**EN:** This function implements `is_habana_available`. It primarily calls `lru_cache`, `find_spec` to complete its work.
**CN:** 该函数实现了 `is_habana_available`。 它主要通过调用 `lru_cache`, `find_spec` 来完成任务。

### Lines 1948-1997: Function `get_device` / 函数 `get_device`
```python
@lru_cache(maxsize=8)
def get_device(device_id: Optional[int] = None) -> str:
    if is_cpu():
        if cpu_has_amx_support():
            logger.info("Intel AMX is detected, using CPU with Intel AMX support.")
        else:
            logger.warning(
                "CPU device enabled, using torch native backend, low performance expected."
            )
        return "cpu"

    if hasattr(torch, "cuda") and torch.cuda.is_available():
        if device_id is None:
            return "cuda"
        return "cuda:{}".format(device_id)

    if hasattr(torch, "xpu") and torch.xpu.is_available():
        if device_id is None:
            return "xpu"
        return "xpu:{}".format(device_id)

    if is_npu():
        if device_id is None:
            return "npu"
        return "npu:{}".format(device_id)

    if is_habana_available():
        try:
# ... omitted for brevity ...
    if is_mps():
        if device_id is None:
            return "mps"
        return "mps:{}".format(device_id)

    raise RuntimeError("No accelerator (CUDA, XPU, HPU, NPU, MUSA, MPS) is available.")
```
**EN:** This function implements `get_device`. It primarily calls `lru_cache`, `is_cpu`, `is_npu`, `is_habana_available`, `is_musa`, `is_mps` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_device`。 它主要通过调用 `lru_cache`, `is_cpu`, `is_npu`, `is_habana_available`, `is_musa`, `is_mps` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 2000-2023: Function `get_device_count` / 函数 `get_device_count`
```python
@lru_cache(maxsize=1)
def get_device_count() -> int:
    if (hasattr(torch, "cuda") and torch.cuda.is_available()) or is_musa():
        try:
            return torch.cuda.device_count()
        except RuntimeError:
            return 0

    if hasattr(torch, "xpu") and torch.xpu.is_available():
        try:
            return torch.xpu.device_count()
        except RuntimeError:
            return 0

    if is_habana_available():
        try:
            import habana_frameworks.torch.hpu  # noqa: F401

            if torch.hpu.is_available():
                return torch.hpu.device_count()
        except (ImportError, RuntimeError):
            return 0

    return 0  # No accelerators available
```
**EN:** This function implements `get_device_count`. It primarily calls `lru_cache`, `is_habana_available`, `is_musa`, `hasattr`, `torch.xpu.is_available`, `torch.cuda.is_available` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_device_count`。 它主要通过调用 `lru_cache`, `is_habana_available`, `is_musa`, `hasattr`, `torch.xpu.is_available`, `torch.cuda.is_available` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 2026-2032: Function `get_device_core_count` / 函数 `get_device_core_count`
```python
def get_device_core_count(device_id: int = 0) -> int:
    if (hasattr(torch, "cuda") and torch.cuda.is_available()) or is_musa():
        return torch.cuda.get_device_properties(device_id).multi_processor_count
    elif hasattr(torch, "xpu") and torch.xpu.is_available():
        return torch.xpu.get_device_properties(device_id).gpu_eu_count

    return 0
```
**EN:** This function implements `get_device_core_count`. It primarily calls `is_musa`, `hasattr`, `torch.cuda.is_available`, `torch.cuda.get_device_properties`, `torch.xpu.is_available`, `torch.xpu.get_device_properties` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_device_core_count`。 它主要通过调用 `is_musa`, `hasattr`, `torch.cuda.is_available`, `torch.cuda.get_device_properties`, `torch.xpu.is_available`, `torch.xpu.get_device_properties` 来完成任务。 实现中使用了条件分支。

### Lines 2035-2058: Function `get_device_capability` / 函数 `get_device_capability`
```python
def get_device_capability(device_id: int = 0) -> Tuple[int, int]:
    major, minor = None, None
    if (hasattr(torch, "cuda") and torch.cuda.is_available()) or is_musa():
        major, minor = torch.cuda.get_device_capability(device_id)

    if hasattr(torch, "xpu") and torch.xpu.is_available():
        major, minor, *_ = torch.xpu.get_device_capability(device_id)["version"].split(
            "."
        )
        # Currently XPU version does not contain capability information.
        major, minor = None, None

    if hasattr(torch, "hpu") and torch.hpu.is_available():
        try:
            # TODO(HandH1998): `get_device_capability` is not supported by `torch.hpu` for now.
            # Update this once the support is available.
            # major, minor = torch.hpu.get_device_capability(device_id)
            major, minor = None, None
        except Exception as e:
            raise RuntimeError(
                f"An error occurred while getting device capability of hpu: {e}."
            ) from e

    return major, minor
```
**EN:** This function implements `get_device_capability`. It primarily calls `is_musa`, `torch.cuda.get_device_capability`, `hasattr`, `torch.xpu.is_available`, `torch.xpu.get_device_capability.split`, `torch.hpu.is_available` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_device_capability`。 它主要通过调用 `is_musa`, `torch.cuda.get_device_capability`, `hasattr`, `torch.xpu.is_available`, `torch.xpu.get_device_capability.split`, `torch.hpu.is_available` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 2061-2089: Function `get_compiler_backend` / 函数 `get_compiler_backend`
```python
def get_compiler_backend(mode=None) -> str:
    # OOT platforms provide their own compile backend.
    from sglang.srt.platforms import current_platform

    if current_platform.is_out_of_tree():
        return current_platform.get_compile_backend(mode)

    if hasattr(torch, "hpu") and torch.hpu.is_available():
        return "hpu_backend"

    if hasattr(torch, "npu") and torch.npu.is_available():
        try:
            import torchair
            import torchair.ge_concrete_graph.ge_converter.experimental.patch_for_hcom_allreduce
            from torchair.configs.compiler_config import CompilerConfig
        except ImportError as e:
            raise ImportError(
                "NPU detected, but torchair package is not installed. "
                "Please install torchair for torch.compile support on NPU."
            )
        compiler_config = CompilerConfig()
        compiler_config.mode = "max-autotune"
        if mode == "npugraph_ex":
            compiler_config.mode = "reduce-overhead"
            compiler_config.debug.run_eagerly = True
        npu_backend = torchair.get_npu_backend(compiler_config=compiler_config)
        return npu_backend

    return "inductor"
```
**EN:** This function implements `get_compiler_backend`. It primarily calls `current_platform.is_out_of_tree`, `current_platform.get_compile_backend`, `hasattr`, `torch.hpu.is_available`, `torch.npu.is_available`, `CompilerConfig` to complete its work. State updates are written into `compiler_config`, `compiler_config.mode`, `npu_backend`, `compiler_config.debug.run_eagerly`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_compiler_backend`。 它主要通过调用 `current_platform.is_out_of_tree`, `current_platform.get_compile_backend`, `hasattr`, `torch.hpu.is_available`, `torch.npu.is_available`, `CompilerConfig` 来完成任务。 状态更新主要写入 `compiler_config`, `compiler_config.mode`, `npu_backend`, `compiler_config.debug.run_eagerly`。 实现中使用了条件分支、错误处理。

### Lines 2092-2092: Constants and shared state / 常量与共享状态
```python
sglang_lib = Library("sglang", "FRAGMENT")  # noqa
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `sglang_lib`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `sglang_lib`。

### Lines 2095-2175: Function `direct_register_custom_op` / 函数 `direct_register_custom_op`
```python
def direct_register_custom_op(
    op_name: str,
    op_func: Callable,
    mutates_args: List[str],
    fake_impl: Optional[Callable] = None,
    target_lib: Optional[Library] = None,
) -> None:
    """
    NOTE: Please try to use `register_custom_op` instead of this function.
    See `python/sglang/srt/utils/custom_op.py` for details.

    `torch.library.custom_op` can have significant overhead because it
    needs to consider complicated dispatching logic. This function
    directly registers a custom op and dispatches it to the CUDA backend.
    See https://gist.github.com/youkaichao/ecbea9ec9fc79a45d2adce1784d7a9a5
    for more details.

    By default, the custom op is registered to the vLLM library. If you
    want to register it to a different library, you can pass the library
    object to the `target_lib` argument.

    IMPORTANT: the lifetime of the operator is tied to the lifetime of the
    library object. If you want to bind the operator to a different library,
    make sure the library object is alive when the operator is used.

    Note: This function will silently skip registration if the operator
    with the same name is already registered to avoid RuntimeError in
    multi-engine scenarios (e.g., VERL framework).
# ... omitted for brevity ...
        else:
            # Re-raise other RuntimeErrors
            raise error
    except AttributeError as error:
        # Always re-raise AttributeError as it indicates missing dependencies
        raise error
```
**EN:** This function implements `direct_register_custom_op`. It primarily calls `hasattr`, `torch.library.infer_schema`, `torch._custom_op.impl.infer_schema`, `my_lib.define`, `is_npu`, `my_lib.impl` to complete its work. State updates are written into `my_lib`, `lib_name`, `schema_str`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `direct_register_custom_op`。 它主要通过调用 `hasattr`, `torch.library.infer_schema`, `torch._custom_op.impl.infer_schema`, `my_lib.define`, `is_npu`, `my_lib.impl` 来完成任务。 状态更新主要写入 `my_lib`, `lib_name`, `schema_str`。 实现中使用了条件分支、错误处理。

### Lines 2178-2211: Function `set_gpu_proc_affinity` / 函数 `set_gpu_proc_affinity`
```python
def set_gpu_proc_affinity(
    pp_size: int,
    tp_size: int,
    nnodes: int,
    gpu_id: int,
):
    # current process
    pid = os.getpid()
    p = psutil.Process(pid)

    nnodes_per_tp_group = max(nnodes // pp_size, 1)
    tp_size_per_node = tp_size // nnodes_per_tp_group

    # total physical cores
    total_pcores = psutil.cpu_count(logical=False)
    # physical cores per TP (N.B. more Cores than GPUs on node)
    num_cores_bind = total_pcores // tp_size_per_node

    # able to handle multiple DP per node
    start_cpu_id = (gpu_id * num_cores_bind) % total_pcores
    end_cpu_id = start_cpu_id + num_cores_bind

    if psutil.cpu_count() != psutil.cpu_count(logical=False):
        # HT on
        lower_cpu_ids = [id for id in range(start_cpu_id, end_cpu_id)]
        upper_cpu_ids = [id + total_pcores for id in range(start_cpu_id, end_cpu_id)]
        bind_cpu_ids = list(itertools.chain(lower_cpu_ids, upper_cpu_ids))
    else:
        # HT off
        bind_cpu_ids = [id for id in range(start_cpu_id, end_cpu_id)]

    # set cpu_affinity to current process
    p.cpu_affinity(bind_cpu_ids)
    logger.info(f"Process {pid} gpu_id {gpu_id} is running on CPUs: {p.cpu_affinity()}")
```
**EN:** This function implements `set_gpu_proc_affinity`. It primarily calls `os.getpid`, `psutil.Process`, `max`, `psutil.cpu_count`, `p.cpu_affinity`, `logger.info` to complete its work. State updates are written into `pid`, `p`, `nnodes_per_tp_group`, `tp_size_per_node`, `total_pcores`, `num_cores_bind`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `set_gpu_proc_affinity`。 它主要通过调用 `os.getpid`, `psutil.Process`, `max`, `psutil.cpu_count`, `p.cpu_affinity`, `logger.info` 来完成任务。 状态更新主要写入 `pid`, `p`, `nnodes_per_tp_group`, `tp_size_per_node`, `total_pcores`, `num_cores_bind`。 实现中使用了条件分支。

### Lines 2214-2231: Function `permute_weight` / 函数 `permute_weight`
```python
def permute_weight(x: torch.Tensor) -> torch.Tensor:
    b_ = x.shape[0]
    n_ = x.shape[1]
    k_ = x.shape[2]

    x_ = x
    if x.dtype == torch.bfloat16 or x.dtype == torch.float16:
        x_ = x_.view(int(b_), int(n_ / 16), 16, int(k_ / 32), 4, 8)
    elif x.dtype == torch.float8_e4m3fnuz or x.dtype == torch.int8:
        x_ = x_.view(int(b_), int(n_ / 16), 16, int(k_ / 64), 4, 16)
    else:
        # return x_
        x_ = x_.view(int(b_), int(n_ / 16), 16, int(k_ / 8), 2, 4)

    x_ = x_.permute(0, 1, 3, 4, 2, 5)
    x_ = x_.contiguous()
    x_ = x_.view(*x.shape)
    return x_
```
**EN:** This function implements `permute_weight`. It primarily calls `x_.permute`, `x_.contiguous`, `x_.view`, `int` to complete its work. State updates are written into `b_`, `n_`, `k_`, `x_`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `permute_weight`。 它主要通过调用 `x_.permute`, `x_.contiguous`, `x_.view`, `int` 来完成任务。 状态更新主要写入 `b_`, `n_`, `k_`, `x_`。 实现中使用了条件分支。

### Lines 2234-2234: Class `MultiprocessingSerializer` declaration / 类 `MultiprocessingSerializer` 声明
```python
class MultiprocessingSerializer:
```
**EN:** This class establishes `MultiprocessingSerializer` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `serialize`, `deserialize`.
**CN:** 该类将 `MultiprocessingSerializer` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `serialize`, `deserialize` 等方法。

### Lines 2235-2256: Method `MultiprocessingSerializer.serialize` / 方法 `MultiprocessingSerializer.serialize`
```python
    @staticmethod
    def serialize(obj, output_str: bool = False):
        """
        Serialize a Python object using ForkingPickler.

        Args:
            obj: The object to serialize.
            output_str (bool): If True, return a base64-encoded string instead of raw bytes.

        Returns:
            bytes or str: The serialized object.
        """
        buf = io.BytesIO()
        ForkingPickler(buf).dump(obj)
        buf.seek(0)
        output = buf.read()

        if output_str:
            # Convert bytes to base64-encoded string
            output = pybase64.b64encode(output).decode("utf-8")

        return output
```
**EN:** This method implements `serialize` on `MultiprocessingSerializer`. It primarily calls `io.BytesIO`, `ForkingPickler.dump`, `buf.seek`, `buf.read`, `pybase64.b64encode.decode`, `ForkingPickler` to complete its work. State updates are written into `buf`, `output`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MultiprocessingSerializer`）实现了 `serialize`。 它主要通过调用 `io.BytesIO`, `ForkingPickler.dump`, `buf.seek`, `buf.read`, `pybase64.b64encode.decode`, `ForkingPickler` 来完成任务。 状态更新主要写入 `buf`, `output`。 实现中使用了条件分支。

### Lines 2258-2273: Method `MultiprocessingSerializer.deserialize` / 方法 `MultiprocessingSerializer.deserialize`
```python
    @staticmethod
    def deserialize(data):
        """
        Deserialize a previously serialized object.

        Args:
            data (bytes or str): The serialized data, optionally base64-encoded.

        Returns:
            The deserialized Python object.
        """
        if isinstance(data, str):
            # Decode base64 string to bytes
            data = pybase64.b64decode(data, validate=True)

        return SafeUnpickler(io.BytesIO(data)).load()
```
**EN:** This method implements `deserialize` on `MultiprocessingSerializer`. It primarily calls `isinstance`, `SafeUnpickler.load`, `pybase64.b64decode`, `SafeUnpickler`, `io.BytesIO` to complete its work. State updates are written into `data`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MultiprocessingSerializer`）实现了 `deserialize`。 它主要通过调用 `isinstance`, `SafeUnpickler.load`, `pybase64.b64decode`, `SafeUnpickler`, `io.BytesIO` 来完成任务。 状态更新主要写入 `data`。 实现中使用了条件分支。

### Lines 2276-2326: Class `SafeUnpickler` declaration / 类 `SafeUnpickler` 声明
```python
class SafeUnpickler(pickle.Unpickler):
    ALLOWED_MODULE_PREFIXES = {
        # --- Python types ---
        "builtins.",
        "collections.",
        "copyreg.",
        "functools.",
        "itertools.",
        "operator.",
        "types.",
        "weakref.",
        # --- PyTorch types ---
        "torch.",
        "torch._tensor.",
        "torch.storage.",
        "torch.nn.parameter.",
        "torch.autograd.function.",
        # --- torch distributed ---
        "torch.distributed.",
        "torch.distributed._shard.",
        "torch.distributed._composable.",
        "torch._C._distributed_c10d.",
        "torch._C._distributed_fsdp.",
        "torch.distributed.optim.",
        # --- multiprocessing ---
        "multiprocessing.resource_sharer.",
        "multiprocessing.reduction.",
        "pickletools.",
# ... omitted for brevity ...
        ("subprocess", "Popen"),
        ("subprocess", "run"),
        ("codecs", "decode"),
        ("types", "CodeType"),
        ("types", "FunctionType"),
    }
```
**EN:** This class establishes `SafeUnpickler` as the main container/coordinator for the surrounding logic. It inherits from `pickle.Unpickler`. Its core interface includes methods such as `find_class`.
**CN:** 该类将 `SafeUnpickler` 定义为周边逻辑的主要封装体或协调者。 它继承自 `pickle.Unpickler`。 其核心接口包括 `find_class` 等方法。

### Lines 2328-2345: Method `SafeUnpickler.find_class` / 方法 `SafeUnpickler.find_class`
```python
    def find_class(self, module, name):
        # Block deterministic attacks
        if (module, name) in self.DENY_CLASSES:
            raise RuntimeError(
                f"Blocked unsafe class loading ({module}.{name}), "
                f"to prevent exploitation of CVE-2025-10164"
            )
        # Allowlist of safe-to-load modules.
        if any(
            (module + ".").startswith(prefix) for prefix in self.ALLOWED_MODULE_PREFIXES
        ):
            return super().find_class(module, name)

        # Block everything else. (Potential attack surface)
        raise RuntimeError(
            f"Blocked unsafe class loading ({module}.{name}), "
            f"to prevent exploitation of CVE-2025-10164"
        )
```
**EN:** This method implements `find_class` on `SafeUnpickler`. It primarily calls `any`, `RuntimeError`, `super.find_class`, `startswith`, `super` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `SafeUnpickler`）实现了 `find_class`。 它主要通过调用 `any`, `RuntimeError`, `super.find_class`, `startswith`, `super` 来完成任务。 实现中使用了条件分支。

### Lines 2348-2350: Function `safe_pickle_load` / 函数 `safe_pickle_load`
```python
def safe_pickle_load(fp):
    """Drop-in replacement for pickle.load() that blocks unsafe class loading."""
    return SafeUnpickler(fp).load()
```
**EN:** This function implements `safe_pickle_load`. It primarily calls `SafeUnpickler.load`, `SafeUnpickler` to complete its work.
**CN:** 该函数实现了 `safe_pickle_load`。 它主要通过调用 `SafeUnpickler.load`, `SafeUnpickler` 来完成任务。

### Lines 2353-2374: Function `debug_timing` / 函数 `debug_timing`
```python
def debug_timing(func):
    # todo: replace with a more organized instrumentation
    def wrapper(*args, **kwargs):
        if logger.isEnabledFor(logging.DEBUG):
            tic = torch.cuda.Event(enable_timing=True)
            toc = torch.cuda.Event(enable_timing=True)
            tic.record()
            result = func(*args, **kwargs)
            toc.record()
            toc.synchronize()  # Wait for the function to complete without synchronizing all ops on the GPU
            elapsed = tic.elapsed_time(toc)
            indices = kwargs.get("indices", args[1] if len(args) > 1 else None)
            num_tokens = len(indices) if indices is not None else 0
            throughput = num_tokens / elapsed * 1000 if elapsed > 0 else 0
            logger.debug(
                f"Transfer time: {elapsed} ms, throughput: {throughput} tokens/s"
            )
            return result
        else:
            return func(*args, **kwargs)

    return wrapper
```
**EN:** This function implements `debug_timing`. It primarily calls `logger.isEnabledFor`, `torch.cuda.Event`, `tic.record`, `func`, `toc.record`, `toc.synchronize` to complete its work. State updates are written into `tic`, `toc`, `result`, `elapsed`, `indices`, `num_tokens`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `debug_timing`。 它主要通过调用 `logger.isEnabledFor`, `torch.cuda.Event`, `tic.record`, `func`, `toc.record`, `toc.synchronize` 来完成任务。 状态更新主要写入 `tic`, `toc`, `result`, `elapsed`, `indices`, `num_tokens`。 实现中使用了条件分支。

### Lines 2377-2380: Function `nullable_str` / 函数 `nullable_str`
```python
def nullable_str(val: str):
    if not val or val == "None":
        return None
    return val
```
**EN:** This function implements `nullable_str`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `nullable_str`。 实现中使用了条件分支。

### Lines 2383-2418: Function `human_readable_int` / 函数 `human_readable_int`
```python
def human_readable_int(value: str) -> int:
    """Supports standard SI suffixes (k, M, G, T) and IEC suffixes
    (Ki, Mi, Gi, Ti). Suffixes are case-sensitive.

    Decimals are allowed for SI suffixes only.

    Examples:
        '1k' -> 1000      '1M' -> 1000000    '25.6k' -> 25600
        '1Ki' -> 1024     '1Mi' -> 1048576
    """
    value = value.strip()

    si_multiplier = {"k": 10**3, "M": 10**6, "G": 10**9, "T": 10**12}
    iec_multiplier = {"Ki": 2**10, "Mi": 2**20, "Gi": 2**30, "Ti": 2**40}

    match = re.fullmatch(r"(\d+(?:\.\d+)?)(Ki|Mi|Gi|Ti|k|M|G|T)", value)
    if match:
        number, suffix = match.groups()
        if suffix in iec_multiplier:
            if "." in number:
                raise argparse.ArgumentTypeError(
                    f"Decimals are not allowed with IEC suffixes like '{suffix}'. "
                    f"Use an integer IEC value such as '{int(Decimal(number))}{suffix}', "
                    f"or an SI value such as '{number}{suffix[0]}'."
                )
            return int(number) * iec_multiplier[suffix]
        return int(Decimal(number) * si_multiplier[suffix])

    try:
        return int(value)
    except ValueError:
        raise argparse.ArgumentTypeError(
            f"Invalid integer value: '{value}'. "
            "Use a plain integer, SI suffixes (1k, 1M), or IEC suffixes (1Ki, 1Mi). "
            "Suffixes are case-sensitive."
        )
```
**EN:** This function implements `human_readable_int`. It primarily calls `value.strip`, `re.fullmatch`, `match.groups`, `int`, `argparse.ArgumentTypeError`, `Decimal` to complete its work. State updates are written into `value`, `si_multiplier`, `iec_multiplier`, `match`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `human_readable_int`。 它主要通过调用 `value.strip`, `re.fullmatch`, `match.groups`, `int`, `argparse.ArgumentTypeError`, `Decimal` 来完成任务。 状态更新主要写入 `value`, `si_multiplier`, `iec_multiplier`, `match`。 实现中使用了条件分支、错误处理。

### Lines 2421-2434: Function `pyspy_dump_schedulers` / 函数 `pyspy_dump_schedulers`
```python
def pyspy_dump_schedulers():
    """py-spy dump on all scheduler in a local node."""
    pid = psutil.Process().pid
    for attempt, native_flag in enumerate(["--native", ""]):
        try:
            cmd = f"py-spy dump {native_flag} --pid {pid}".strip()
            result = subprocess.run(
                cmd, shell=True, capture_output=True, text=True, check=True
            )
            logger.error(f"Pyspy dump for PID {pid} ({cmd}):\n{result.stdout}")
            return
        except subprocess.CalledProcessError as e:
            logger.error(f"Pyspy failed ({cmd}). Error: {e.stderr}")
    logger.error(f"All pyspy dump attempts failed for PID {pid}.")
```
**EN:** This function implements `pyspy_dump_schedulers`. It primarily calls `enumerate`, `logger.error`, `psutil.Process`, `strip`, `subprocess.run` to complete its work. State updates are written into `pid`, `cmd`, `result`. The implementation relies on iteration, error handling.
**CN:** 该函数实现了 `pyspy_dump_schedulers`。 它主要通过调用 `enumerate`, `logger.error`, `psutil.Process`, `strip`, `subprocess.run` 来完成任务。 状态更新主要写入 `pid`, `cmd`, `result`。 实现中使用了迭代逻辑、错误处理。

### Lines 2437-2444: Function `kill_itself_when_parent_died` / 函数 `kill_itself_when_parent_died`
```python
def kill_itself_when_parent_died():
    if sys.platform == "linux":
        # sigkill this process when parent worker manager dies
        PR_SET_PDEATHSIG = 1
        libc = ctypes.CDLL("libc.so.6")
        libc.prctl(PR_SET_PDEATHSIG, signal.SIGKILL)
    else:
        logger.warning("kill_itself_when_parent_died is only supported in linux.")
```
**EN:** This function implements `kill_itself_when_parent_died`. It primarily calls `ctypes.CDLL`, `libc.prctl`, `logger.warning` to complete its work. State updates are written into `PR_SET_PDEATHSIG`, `libc`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `kill_itself_when_parent_died`。 它主要通过调用 `ctypes.CDLL`, `libc.prctl`, `logger.warning` 来完成任务。 状态更新主要写入 `PR_SET_PDEATHSIG`, `libc`。 实现中使用了条件分支。

### Lines 2447-2453: Class `UvicornAccessLogFilter` declaration / 类 `UvicornAccessLogFilter` 声明
```python
class UvicornAccessLogFilter(logging.Filter):
    """Filter uvicorn access logs by request path.

    Notes:
    - Uvicorn access records usually provide `request_line` like: "GET /metrics HTTP/1.1".
    - We defensively fall back to parsing `record.getMessage()` if needed.
    """
```
**EN:** This class establishes `UvicornAccessLogFilter` as the main container/coordinator for the surrounding logic. It inherits from `logging.Filter`. Its core interface includes methods such as `__init__`, `filter`.
**CN:** 该类将 `UvicornAccessLogFilter` 定义为周边逻辑的主要封装体或协调者。 它继承自 `logging.Filter`。 其核心接口包括 `__init__`, `filter` 等方法。

### Lines 2455-2459: Method `UvicornAccessLogFilter.__init__` / 方法 `UvicornAccessLogFilter.__init__`
```python
    def __init__(self, excluded_path_prefixes=None):
        super().__init__()
        excluded_path_prefixes = excluded_path_prefixes or []
        # Normalize once: drop empty prefixes, stringify, keep as tuple (fast iteration, immutable).
        self.excluded_path_prefixes = tuple(str(p) for p in excluded_path_prefixes if p)
```
**EN:** This method implements `__init__` on `UvicornAccessLogFilter`. It primarily calls `super.__init__`, `tuple`, `super`, `str` to complete its work. State updates are written into `excluded_path_prefixes`, `self.excluded_path_prefixes`.
**CN:** 该方法（属于 `UvicornAccessLogFilter`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `tuple`, `super`, `str` 来完成任务。 状态更新主要写入 `excluded_path_prefixes`, `self.excluded_path_prefixes`。

### Lines 2461-2501: Method `UvicornAccessLogFilter.filter` / 方法 `UvicornAccessLogFilter.filter`
```python
    def filter(self, record: logging.LogRecord) -> bool:
        path = None

        request_line = getattr(record, "request_line", None)
        if request_line:
            parts = str(request_line).split()
            if len(parts) >= 2:
                path = parts[1]

        if not path:
            # Fallback for non-standard formatters/records
            try:
                msg = record.getMessage()
            except Exception:
                msg = None
            if msg:
                q1 = msg.find('"')
                q2 = msg.find('"', q1 + 1) if q1 != -1 else -1
                if q1 != -1 and q2 != -1:
                    rl = msg[q1 + 1 : q2]
                    parts = rl.split()
                    if len(parts) >= 2:
                        path = parts[1]

        if not path:
            return True

        # Strip query string for matching
# ... omitted for brevity ...
                # If parsing fails, fall back to the raw value.
                pass
        path = path.split("?", 1)[0]
        return not any(
            path.startswith(prefix) for prefix in self.excluded_path_prefixes
        )
```
**EN:** This method implements `filter` on `UvicornAccessLogFilter`. It primarily calls `getattr`, `str`, `str.split`, `path.split`, `any`, `len` to complete its work. State updates are written into `path`, `request_line`, `parts`, `msg`, `q1`, `q2`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `UvicornAccessLogFilter`）实现了 `filter`。 它主要通过调用 `getattr`, `str`, `str.split`, `path.split`, `any`, `len` 来完成任务。 状态更新主要写入 `path`, `request_line`, `parts`, `msg`, `q1`, `q2`。 实现中使用了条件分支、错误处理。

### Lines 2504-2516: Function `set_uvicorn_logging_configs` / 函数 `set_uvicorn_logging_configs`
```python
def set_uvicorn_logging_configs(server_args=None):
    from uvicorn.config import LOGGING_CONFIG

    LOGGING_CONFIG["formatters"]["default"][
        "fmt"
    ] = "[%(asctime)s] %(levelprefix)s %(message)s"
    LOGGING_CONFIG["formatters"]["default"]["datefmt"] = "%Y-%m-%d %H:%M:%S"
    LOGGING_CONFIG["formatters"]["access"][
        "fmt"
    ] = '[%(asctime)s] %(levelprefix)s %(client_addr)s - "%(request_line)s" %(status_code)s'
    LOGGING_CONFIG["formatters"]["access"]["datefmt"] = "%Y-%m-%d %H:%M:%S"

    _configure_uvicorn_access_log_filter(LOGGING_CONFIG, server_args)
```
**EN:** This function implements `set_uvicorn_logging_configs`. It primarily calls `_configure_uvicorn_access_log_filter` to complete its work. State updates are written into `LOGGING_CONFIG`.
**CN:** 该函数实现了 `set_uvicorn_logging_configs`。 它主要通过调用 `_configure_uvicorn_access_log_filter` 来完成任务。 状态更新主要写入 `LOGGING_CONFIG`。

### Lines 2519-2576: Function `_configure_uvicorn_access_log_filter` / 函数 `_configure_uvicorn_access_log_filter`
```python
def _configure_uvicorn_access_log_filter(
    uvicorn_logging_config: dict, server_args=None
):
    """Configure uvicorn access log path filter into uvicorn LOGGING_CONFIG.

    This optionally filters uvicorn access logs (e.g., suppress noisy /metrics polling).

    Args:
        uvicorn_logging_config: The dict-like LOGGING_CONFIG from uvicorn.
        server_args: Parsed server args object that may contain:
            - uvicorn_access_log_exclude_prefixes (list[str] | tuple[str] | None)
    """
    # Optionally filter uvicorn access logs (e.g., suppress noisy /metrics polling).
    if server_args is None:
        return

    filter_name = "sglang_uvicorn_access_path_filter"

    excluded_prefixes = getattr(
        server_args, "uvicorn_access_log_exclude_prefixes", None
    )
    if not excluded_prefixes:
        return

    # Normalize: accept list/tuple; treat a single string as one prefix (not an iterable of chars).
    if isinstance(excluded_prefixes, str):
        excluded_prefixes = [excluded_prefixes]

# ... omitted for brevity ...
        filters_list = loggers_cfg["uvicorn.access"].setdefault("filters", [])
        if not isinstance(filters_list, list):
            filters_list = list(filters_list)
            loggers_cfg["uvicorn.access"]["filters"] = filters_list
        if filter_name not in filters_list:
            filters_list.append(filter_name)
```
**EN:** This function implements `_configure_uvicorn_access_log_filter`. It primarily calls `getattr`, `isinstance`, `list`, `uvicorn_logging_config.setdefault`, `uvicorn_logging_config.get`, `dict.fromkeys` to complete its work. State updates are written into `filter_name`, `excluded_prefixes`, `uvicorn_logging_config`, `handlers`, `loggers_cfg`, `filters_list`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_configure_uvicorn_access_log_filter`。 它主要通过调用 `getattr`, `isinstance`, `list`, `uvicorn_logging_config.setdefault`, `uvicorn_logging_config.get`, `dict.fromkeys` 来完成任务。 状态更新主要写入 `filter_name`, `excluded_prefixes`, `uvicorn_logging_config`, `handlers`, `loggers_cfg`, `filters_list`。 实现中使用了条件分支。

### Lines 2579-2639: Function `launch_dummy_health_check_server` / 函数 `launch_dummy_health_check_server`
```python
def launch_dummy_health_check_server(host, port, enable_metrics):
    import asyncio

    import uvicorn
    from fastapi import FastAPI, Response

    from sglang.srt.utils.network import NetworkAddress

    app = FastAPI()

    @app.get("/ping")
    async def ping():
        """Could be used by the checkpoint-engine update script to confirm the server is up."""
        return Response(status_code=200)

    @app.get("/health")
    async def health():
        """Check the health of the http server."""
        return Response(status_code=200)

    @app.get("/health_generate")
    async def health_generate():
        """Check the health of the http server."""
        return Response(status_code=200)

    # Add prometheus middleware
    if enable_metrics:
        add_prometheus_middleware(app)
# ... omitted for brevity ...
        target=run_server, daemon=True, name="health-check-server"
    )
    thread.start()
    logger.info(
        f"Dummy health check server started in background thread at {NetworkAddress(host, port).to_host_port_str()}"
    )
```
**EN:** This function implements `launch_dummy_health_check_server`. It primarily calls `FastAPI`, `app.get`, `uvicorn.Config`, `uvicorn.Server`, `threading.Thread`, `thread.start` to complete its work. State updates are written into `app`, `config`, `server`, `thread`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `launch_dummy_health_check_server`。 它主要通过调用 `FastAPI`, `app.get`, `uvicorn.Config`, `uvicorn.Server`, `threading.Thread`, `thread.start` 来完成任务。 状态更新主要写入 `app`, `config`, `server`, `thread`。 实现中使用了条件分支、错误处理。

### Lines 2642-2648: Function `set_cuda_arch` / 函数 `set_cuda_arch`
```python
def set_cuda_arch():
    if is_flashinfer_available():
        capability = torch.cuda.get_device_capability()
        arch = f"{capability[0]}.{capability[1]}"
        os.environ["FLASHINFER_CUDA_ARCH_LIST"] = (
            f"{arch}{'a' if capability[0] >= 9 else ''}"
        )
```
**EN:** This function implements `set_cuda_arch`. It primarily calls `is_flashinfer_available`, `torch.cuda.get_device_capability` to complete its work. State updates are written into `capability`, `arch`, `os.environ`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `set_cuda_arch`。 它主要通过调用 `is_flashinfer_available`, `torch.cuda.get_device_capability` 来完成任务。 状态更新主要写入 `capability`, `arch`, `os.environ`。 实现中使用了条件分支。

### Lines 2651-2653: Function `cdiv` / 函数 `cdiv`
```python
def cdiv(a: int, b: int) -> int:
    """Ceiling division."""
    return -(a // -b)
```
**EN:** This function implements `cdiv`.
**CN:** 该函数实现了 `cdiv`。

### Lines 2656-2657: Function `next_power_of_2` / 函数 `next_power_of_2`
```python
def next_power_of_2(n: int):
    return 1 << (n - 1).bit_length() if n > 0 else 1
```
**EN:** This function implements `next_power_of_2`. It primarily calls `bit_length` to complete its work.
**CN:** 该函数实现了 `next_power_of_2`。 它主要通过调用 `bit_length` 来完成任务。

### Lines 2660-2661: Function `round_up` / 函数 `round_up`
```python
def round_up(x: int, y: int) -> int:
    return ((x - 1) // y + 1) * y
```
**EN:** This function implements `round_up`.
**CN:** 该函数实现了 `round_up`。

### Lines 2664-2664: Constants and shared state / 常量与共享状态
```python
setattr(triton, "next_power_of_2", next_power_of_2)
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。

### Lines 2667-2667: Class `EmptyContextManager` declaration / 类 `EmptyContextManager` 声明
```python
class EmptyContextManager:
```
**EN:** This class establishes `EmptyContextManager` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__enter__`, `__exit__`.
**CN:** 该类将 `EmptyContextManager` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__enter__`, `__exit__` 等方法。

### Lines 2668-2669: Method `EmptyContextManager.__enter__` / 方法 `EmptyContextManager.__enter__`
```python
    def __enter__(self):
        return self
```
**EN:** This method implements `__enter__` on `EmptyContextManager`.
**CN:** 该方法（属于 `EmptyContextManager`）实现了 `__enter__`。

### Lines 2671-2672: Method `EmptyContextManager.__exit__` / 方法 `EmptyContextManager.__exit__`
```python
    def __exit__(self, exc_type, exc_value, traceback):
        pass
```
**EN:** This method implements `__exit__` on `EmptyContextManager`.
**CN:** 该方法（属于 `EmptyContextManager`）实现了 `__exit__`。

### Lines 2675-2676: Function `empty_context` / 函数 `empty_context`
```python
def empty_context(*args, **kwargs):
    return EmptyContextManager()
```
**EN:** This function implements `empty_context`. It primarily calls `EmptyContextManager` to complete its work.
**CN:** 该函数实现了 `empty_context`。 它主要通过调用 `EmptyContextManager` 来完成任务。

### Lines 2679-2689: Function `add_prefix` / 函数 `add_prefix`
```python
def add_prefix(name: str, prefix: str) -> str:
    """Add a weight path prefix to a module name.

    Args:
        name: base module name.
        prefix: weight prefix str to added to the front of `name` concatenated with `.`.

    Returns:
        The string `prefix.name` if prefix is non-empty, otherwise just `name`.
    """
    return name if not prefix else f"{prefix}.{name}"
```
**EN:** This function implements `add_prefix`.
**CN:** 该函数实现了 `add_prefix`。

### Lines 2692-2702: Function `is_remote_url` / 函数 `is_remote_url`
```python
def is_remote_url(url: Union[str, Path]) -> bool:
    """
    Check if the URL is a remote URL of the format:
    <connector_type>://<host>:<port>/<model_name>
    """
    if isinstance(url, Path):
        return False

    pattern = r"(.+)://(.*)"
    m = re.match(pattern, url)
    return m is not None
```
**EN:** This function implements `is_remote_url`. It primarily calls `isinstance`, `re.match` to complete its work. State updates are written into `pattern`, `m`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_remote_url`。 它主要通过调用 `isinstance`, `re.match` 来完成任务。 状态更新主要写入 `pattern`, `m`。 实现中使用了条件分支。

### Lines 2705-2715: Function `parse_connector_type` / 函数 `parse_connector_type`
```python
def parse_connector_type(url: str) -> str:
    """
    Parse the connector type from the URL of the format:
    <connector_type>://<path>
    """
    pattern = r"(.+)://(.*)"
    m = re.match(pattern, url)
    if m is None:
        return ""

    return m.group(1)
```
**EN:** This function implements `parse_connector_type`. It primarily calls `re.match`, `m.group` to complete its work. State updates are written into `pattern`, `m`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `parse_connector_type`。 它主要通过调用 `re.match`, `m.group` 来完成任务。 状态更新主要写入 `pattern`, `m`。 实现中使用了条件分支。

### Lines 2718-2748: Function `retry` / 函数 `retry`
```python
def retry(
    fn,
    max_retry: int,
    initial_delay: float = 2.0,
    max_delay: float = 60.0,
    should_retry: Callable[[Any], bool] = lambda e: True,
):
    for try_index in itertools.count():
        try:
            return fn()
        except SkipTest:
            # Do NOT retry skipped tests - used in CI and unittest
            raise
        except Exception as e:
            traceback.print_exc()

            if try_index >= max_retry:
                raise Exception(f"retry() exceed maximum number of retries.")

            if not should_retry(e):
                raise Exception(f"retry() observe errors that should not be retried.")

            delay = min(initial_delay * (2**try_index), max_delay) * (
                0.75 + 0.25 * random.random()
            )

            logger.warning(
                f"retry() failed once ({try_index}th try, maximum {max_retry} retries). Will delay {delay:.2f}s and retry. Error: {e}"
            )

            time.sleep(delay)
```
**EN:** This function implements `retry`. It primarily calls `itertools.count`, `fn`, `traceback.print_exc`, `logger.warning`, `time.sleep`, `Exception` to complete its work. State updates are written into `delay`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `retry`。 它主要通过调用 `itertools.count`, `fn`, `traceback.print_exc`, `logger.warning`, `time.sleep`, `Exception` 来完成任务。 状态更新主要写入 `delay`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 2751-2778: Function `has_hf_quant_config` / 函数 `has_hf_quant_config`
```python
def has_hf_quant_config(model_path: str) -> bool:
    """Check if the model path contains hf_quant_config.json file.

    Args:
        model_path: Path to the model, can be local path or remote URL.

    Returns:
        True if hf_quant_config.json exists, False otherwise.
    """
    # Check if the model_path is a local path
    if os.path.exists(os.path.join(model_path, "hf_quant_config.json")):
        return True

    from huggingface_hub import try_to_load_from_cache

    # Check if the model_path is a HuggingFace model ID and exists locally
    result = try_to_load_from_cache(model_path, "hf_quant_config.json")
    if isinstance(result, str):
        return True

    # Check if the model_path is a remote URL and exists on the HuggingFace Hub
    try:
        from huggingface_hub import HfApi

        hf_api = HfApi()
        return hf_api.file_exists(model_path, "hf_quant_config.json")
    except Exception:
        return False
```
**EN:** This function implements `has_hf_quant_config`. It primarily calls `os.path.exists`, `try_to_load_from_cache`, `isinstance`, `os.path.join`, `HfApi`, `hf_api.file_exists` to complete its work. State updates are written into `result`, `hf_api`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `has_hf_quant_config`。 它主要通过调用 `os.path.exists`, `try_to_load_from_cache`, `isinstance`, `os.path.join`, `HfApi`, `hf_api.file_exists` 来完成任务。 状态更新主要写入 `result`, `hf_api`。 实现中使用了条件分支、错误处理。

### Lines 2781-2786: Function `get_quantization_config` / 函数 `get_quantization_config`
```python
def get_quantization_config(hf_config) -> str | None:
    """Extract quantization method from HuggingFace config."""
    quantization_config = getattr(hf_config, "quantization_config", None)
    if quantization_config is not None:
        return quantization_config.get("quant_method")
    return None
```
**EN:** This function implements `get_quantization_config`. It primarily calls `getattr`, `quantization_config.get` to complete its work. State updates are written into `quantization_config`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_quantization_config`。 它主要通过调用 `getattr`, `quantization_config.get` 来完成任务。 状态更新主要写入 `quantization_config`。 实现中使用了条件分支。

### Lines 2789-2853: Function `has_fp8_weights_in_checkpoint` / 函数 `has_fp8_weights_in_checkpoint`
```python
def has_fp8_weights_in_checkpoint(model_path: str) -> bool:
    """Check if a model checkpoint actually contains FP8 (float8_e4m3fn) expert
    weight tensors by reading safetensors metadata headers.

    This is needed because some models (e.g. DeepSeek V3/R1) use native FP8 MoE
    experts without declaring it in quantization_config, while other models
    sharing the same architecture (e.g. Moonlight) are purely BF16.

    Accepts a local directory or a HuggingFace repo ID. For remote repos, only
    safetensors headers (a few KB) are fetched via byte-range reads; full
    shards are never downloaded.
    """
    import json
    import struct

    try:
        if os.path.isdir(model_path):

            def _open(name):
                return open(os.path.join(model_path, name), "rb")

            def _exists(name):
                return os.path.exists(os.path.join(model_path, name))

        else:
            from huggingface_hub import HfFileSystem

            fs = HfFileSystem()
# ... omitted for brevity ...
                continue
            if "experts" in key and "weight" in key:
                return meta.get("dtype") == "F8_E4M3"
        return False
    except Exception:
        return False
```
**EN:** This function implements `has_fp8_weights_in_checkpoint`. It primarily calls `os.path.isdir`, `_exists`, `header.items`, `HfFileSystem`, `sorted`, `_open` to complete its work. State updates are written into `fs`, `expert_files`, `shard_file`, `header_len`, `header`, `weight_map`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `has_fp8_weights_in_checkpoint`。 它主要通过调用 `os.path.isdir`, `_exists`, `header.items`, `HfFileSystem`, `sorted`, `_open` 来完成任务。 状态更新主要写入 `fs`, `expert_files`, `shard_file`, `header_len`, `header`, `weight_map`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 2856-2862: Function `flatten_nested_list` / 函数 `flatten_nested_list`
```python
def flatten_nested_list(nested_list):
    if isinstance(nested_list, list):
        return [
            item for sublist in nested_list for item in flatten_nested_list(sublist)
        ]
    else:
        return [nested_list]
```
**EN:** This function implements `flatten_nested_list`. It primarily calls `isinstance`, `flatten_nested_list` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `flatten_nested_list`。 它主要通过调用 `isinstance`, `flatten_nested_list` 来完成任务。 实现中使用了条件分支。

### Lines 2865-2870: Function `is_non_idle_and_non_empty` / 函数 `is_non_idle_and_non_empty`
```python
def is_non_idle_and_non_empty(forward_mode, hidden_states):
    return (
        (forward_mode is not None)
        and not forward_mode.is_idle()
        and hidden_states.shape[0] > 0
    )
```
**EN:** This function implements `is_non_idle_and_non_empty`. It primarily calls `forward_mode.is_idle` to complete its work.
**CN:** 该函数实现了 `is_non_idle_and_non_empty`。 它主要通过调用 `forward_mode.is_idle` 来完成任务。

### Lines 2873-2879: Function `fast_topk` / 函数 `fast_topk`
```python
def fast_topk(values, topk, dim):
    if topk == 1:
        # Use max along the specified dimension to get both value and index
        return torch.max(values, dim=dim, keepdim=True)
    else:
        # Use topk for efficiency with larger k values
        return torch.topk(values, topk, dim=dim)
```
**EN:** This function implements `fast_topk`. It primarily calls `torch.max`, `torch.topk` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `fast_topk`。 它主要通过调用 `torch.max`, `torch.topk` 来完成任务。 实现中使用了条件分支。

### Lines 2882-2887: Function `bind_or_assign` / 函数 `bind_or_assign`
```python
def bind_or_assign(target, source):
    if target is not None:
        target.copy_(source)
        return target
    else:
        return source
```
**EN:** This function implements `bind_or_assign`. It primarily calls `target.copy_` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `bind_or_assign`。 它主要通过调用 `target.copy_` 来完成任务。 实现中使用了条件分支。

### Lines 2892-2896: Function `is_no_spec_infer_or_topk_one` / 函数 `is_no_spec_infer_or_topk_one`
```python
def is_no_spec_infer_or_topk_one(server_args):
    return server_args.speculative_eagle_topk is None or (
        server_args.speculative_eagle_topk == 1
        and (server_args.page_size == 1 or server_args.page_size is None)
    )
```
**EN:** This function implements `is_no_spec_infer_or_topk_one`.
**CN:** 该函数实现了 `is_no_spec_infer_or_topk_one`。

### Lines 2899-2924: Function `is_fa3_default_architecture` / 函数 `is_fa3_default_architecture`
```python
def is_fa3_default_architecture(hf_config):
    architectures = getattr(hf_config, "architectures", None)
    if not isinstance(architectures, list) or not architectures:
        return False
    default_archs = {
        "Llama4ForConditionalGeneration",
        "LlamaForCausalLM",
        "Olmo2ForCausalLM",
        "Gemma2ForCausalLM",
        "Gemma3ForConditionalGeneration",
        "MixtralForCausalLM",
        "Qwen2ForCausalLM",
        "Qwen3ForCausalLM",
        "Qwen3MoeForCausalLM",
        "Qwen3VLForConditionalGeneration",
        "Qwen3VLMoeForConditionalGeneration",
        "Glm4MoeForCausalLM",
        "Glm4vForConditionalGeneration",
        "Glm4vMoeForConditionalGeneration",
        "GlmOcrForConditionalGeneration",
        "Step3VLForConditionalGeneration",
        "StepVLForConditionalGeneration",
        "MiMoV2ForCausalLM",
        "MiMoV2FlashForCausalLM",
    }
    return architectures[0] in default_archs
```
**EN:** This function implements `is_fa3_default_architecture`. It primarily calls `getattr`, `isinstance` to complete its work. State updates are written into `architectures`, `default_archs`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_fa3_default_architecture`。 它主要通过调用 `getattr`, `isinstance` 来完成任务。 状态更新主要写入 `architectures`, `default_archs`。 实现中使用了条件分支。

### Lines 2928-2928: Class `BumpAllocator` declaration / 类 `BumpAllocator` 声明
```python
class BumpAllocator:
```
**EN:** This class establishes `BumpAllocator` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `allocate`.
**CN:** 该类将 `BumpAllocator` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `allocate` 等方法。

### Lines 2929-2931: Method `BumpAllocator.__init__` / 方法 `BumpAllocator.__init__`
```python
    def __init__(self, buffer_size: int, dtype, device):
        self._buffer = torch.zeros((buffer_size,), dtype=dtype, device=device)
        self._pointer = 0
```
**EN:** This method implements `__init__` on `BumpAllocator`. It primarily calls `torch.zeros` to complete its work. State updates are written into `self._buffer`, `self._pointer`.
**CN:** 该方法（属于 `BumpAllocator`）实现了 `__init__`。 它主要通过调用 `torch.zeros` 来完成任务。 状态更新主要写入 `self._buffer`, `self._pointer`。

### Lines 2933-2937: Method `BumpAllocator.allocate` / 方法 `BumpAllocator.allocate`
```python
    def allocate(self, size: int):
        assert self._pointer + size <= len(self._buffer)
        output = self._buffer[self._pointer : self._pointer + size]
        self._pointer += size
        return output
```
**EN:** This method implements `allocate` on `BumpAllocator`. It primarily calls `len` to complete its work. State updates are written into `output`, `self._pointer`.
**CN:** 该方法（属于 `BumpAllocator`）实现了 `allocate`。 它主要通过调用 `len` 来完成任务。 状态更新主要写入 `output`, `self._pointer`。

### Lines 2940-2951: Function `log_info_on_rank0` / 函数 `log_info_on_rank0`
```python
def log_info_on_rank0(logger, msg):
    from sglang.srt.distributed import get_tensor_model_parallel_rank

    try:
        if torch.distributed.is_initialized() and get_tensor_model_parallel_rank() == 0:
            logger.info(msg)
    except Exception as e:
        if torch.distributed.is_initialized():
            if torch.distributed.get_rank() == 0:
                logger.info(f"{msg} (rank-check failed: {e})")
        else:
            logger.info(f"{msg} (rank-check failed: {e})")
```
**EN:** This function implements `log_info_on_rank0`. It primarily calls `torch.distributed.is_initialized`, `logger.info`, `get_tensor_model_parallel_rank`, `torch.distributed.get_rank` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `log_info_on_rank0`。 它主要通过调用 `torch.distributed.is_initialized`, `logger.info`, `get_tensor_model_parallel_rank`, `torch.distributed.get_rank` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 2954-2969: Function `log_debug_on_rank0` / 函数 `log_debug_on_rank0`
```python
def log_debug_on_rank0(logger, msg):
    """
    Log a debug message only on tensor model parallel rank 0.
    Falls back to logging if distributed is not initialized or error occurs.
    """
    from sglang.srt.distributed import get_tensor_model_parallel_rank

    try:
        if torch.distributed.is_initialized() and get_tensor_model_parallel_rank() == 0:
            logger.debug(msg)
    except Exception as e:
        if torch.distributed.is_initialized():
            if torch.distributed.get_rank() == 0:
                logger.debug(f"{msg} (rank-check failed: {e})")
        else:
            logger.debug(f"{msg} (rank-check failed: {e})")
```
**EN:** This function implements `log_debug_on_rank0`. It primarily calls `torch.distributed.is_initialized`, `logger.debug`, `get_tensor_model_parallel_rank`, `torch.distributed.get_rank` to complete its work. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `log_debug_on_rank0`。 它主要通过调用 `torch.distributed.is_initialized`, `logger.debug`, `get_tensor_model_parallel_rank`, `torch.distributed.get_rank` 来完成任务。 实现中使用了条件分支、错误处理。

### Lines 2972-2976: Function `load_json_config` / 函数 `load_json_config`
```python
def load_json_config(data: str):
    try:
        return orjson.loads(data)
    except JSONDecodeError:
        return orjson.loads(Path(data).read_text())
```
**EN:** This function implements `load_json_config`. It primarily calls `orjson.loads`, `Path.read_text`, `Path` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `load_json_config`。 它主要通过调用 `orjson.loads`, `Path.read_text`, `Path` 来完成任务。 实现中使用了错误处理。

### Lines 2979-2995: Function `dispose_tensor` / 函数 `dispose_tensor`
```python
def dispose_tensor(x: torch.Tensor):
    """
    Dispose a tensor by freeing its memory.
    During piecewise CUDA graph capture/replay, we skip disposal to avoid
    interfering with torch.compile's memory tracking and graph recording.
    """

    # Skip disposal during piecewise CUDA graph to avoid torch.compile issues
    # we do local import to avoid circular import
    from sglang.srt.compilation.piecewise_context_manager import (
        is_in_piecewise_cuda_graph,
    )

    if is_in_piecewise_cuda_graph():
        return

    x.set_(torch.empty((0,), device=x.device, dtype=x.dtype))
```
**EN:** This function implements `dispose_tensor`. It primarily calls `is_in_piecewise_cuda_graph`, `x.set_`, `torch.empty` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `dispose_tensor`。 它主要通过调用 `is_in_piecewise_cuda_graph`, `x.set_`, `torch.empty` 来完成任务。 实现中使用了条件分支。

### Lines 2998-2998: Constants and shared state / 常量与共享状态
```python
T = TypeVar("T")
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `T`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `T`。

### Lines 3001-3001: Class `Withable` declaration / 类 `Withable` 声明
```python
class Withable(Generic[T]):
```
**EN:** This class establishes `Withable` as the main container/coordinator for the surrounding logic. It inherits from `Generic`. Its core interface includes methods such as `__init__`, `value`, `with_value`.
**CN:** 该类将 `Withable` 定义为周边逻辑的主要封装体或协调者。 它继承自 `Generic`。 其核心接口包括 `__init__`, `value`, `with_value` 等方法。

### Lines 3002-3003: Method `Withable.__init__` / 方法 `Withable.__init__`
```python
    def __init__(self):
        self._value: Optional[T] = None
```
**EN:** This method implements `__init__` on `Withable`. State updates are written into `self._value`.
**CN:** 该方法（属于 `Withable`）实现了 `__init__`。 状态更新主要写入 `self._value`。

### Lines 3005-3007: Method `Withable.value` / 方法 `Withable.value`
```python
    @property
    def value(self) -> T:
        return self._value
```
**EN:** This method implements `value` on `Withable`.
**CN:** 该方法（属于 `Withable`）实现了 `value`。

### Lines 3009-3017: Method `Withable.with_value` / 方法 `Withable.with_value`
```python
    @contextmanager
    def with_value(self, new_value: T):
        assert self._value is None
        self._value = new_value
        try:
            yield
        finally:
            assert self._value is new_value
            self._value = None
```
**EN:** This method implements `with_value` on `Withable`. State updates are written into `self._value`. The implementation relies on error handling.
**CN:** 该方法（属于 `Withable`）实现了 `with_value`。 状态更新主要写入 `self._value`。 实现中使用了错误处理。

### Lines 3020-3042: Function `require_mlp_tp_gather` / 函数 `require_mlp_tp_gather`
```python
def require_mlp_tp_gather(server_args: ServerArgs):
    """
    Check if the input of MLP is obtained by all-gather rather than all-reduce. This only happens when each MLP TP group contains multiple attention DP groups.
    """
    from sglang.srt.layers.moe.utils import get_moe_a2a_backend

    if server_args.enable_dp_attention:
        assert server_args.dp_size > 1, "dp_size must be greater than 1"
        if (
            server_args.moe_dense_tp_size is None
        ):  # TODO(ch-wan): some MoE models do not have dense layers
            return True
        elif not server_args.enable_dp_lm_head:
            return True
        elif get_moe_a2a_backend().is_none():
            return True
        else:
            return (
                server_args.moe_dense_tp_size
                > server_args.tp_size // server_args.dp_size
            )
    else:
        return False
```
**EN:** This function implements `require_mlp_tp_gather`. It primarily calls `get_moe_a2a_backend.is_none`, `get_moe_a2a_backend` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `require_mlp_tp_gather`。 它主要通过调用 `get_moe_a2a_backend.is_none`, `get_moe_a2a_backend` 来完成任务。 实现中使用了条件分支。

### Lines 3045-3058: Function `require_attn_tp_gather` / 函数 `require_attn_tp_gather`
```python
def require_attn_tp_gather(server_args: ServerArgs):
    """
    Check if the input of attention is scattered.
    """
    from sglang.srt.layers.moe.utils import get_moe_a2a_backend

    assert server_args.moe_dense_tp_size in [1, None]
    if not get_moe_a2a_backend().is_none() or server_args.moe_dense_tp_size == 1:
        if server_args.enable_dp_attention:
            return server_args.dp_size < server_args.tp_size
        else:
            return True
    else:
        return False
```
**EN:** This function implements `require_attn_tp_gather`. It primarily calls `get_moe_a2a_backend.is_none`, `get_moe_a2a_backend` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `require_attn_tp_gather`。 它主要通过调用 `get_moe_a2a_backend.is_none`, `get_moe_a2a_backend` 来完成任务。 实现中使用了条件分支。

### Lines 3061-3062: Function `require_gathered_buffer` / 函数 `require_gathered_buffer`
```python
def require_gathered_buffer(server_args: ServerArgs):
    return require_mlp_tp_gather(server_args) or require_attn_tp_gather(server_args)
```
**EN:** This function implements `require_gathered_buffer`. It primarily calls `require_mlp_tp_gather`, `require_attn_tp_gather` to complete its work.
**CN:** 该函数实现了 `require_gathered_buffer`。 它主要通过调用 `require_mlp_tp_gather`, `require_attn_tp_gather` 来完成任务。

### Lines 3065-3066: Function `require_mlp_sync` / 函数 `require_mlp_sync`
```python
def require_mlp_sync(server_args: ServerArgs):
    return server_args.enable_dp_attention or require_gathered_buffer(server_args)
```
**EN:** This function implements `require_mlp_sync`. It primarily calls `require_gathered_buffer` to complete its work.
**CN:** 该函数实现了 `require_mlp_sync`。 它主要通过调用 `require_gathered_buffer` 来完成任务。

### Lines 3069-3091: Function `find_local_repo_dir` / 函数 `find_local_repo_dir`
```python
def find_local_repo_dir(repo_id: str, revision: Optional[str] = None) -> Optional[str]:
    import huggingface_hub as hf

    # Build cache path
    cache_path = os.path.join(
        hf.constants.HF_HUB_CACHE,
        hf.constants.REPO_ID_SEPARATOR.join(["models", *repo_id.split("/")]),
    )

    # Get revision from main ref if not specified
    if not revision:
        ref_path = os.path.join(cache_path, "refs", "main")
        if os.path.isfile(ref_path):
            with open(ref_path) as f:
                revision = f.read().strip()

    # List files from revision directory
    if revision:
        rev_dir = os.path.join(cache_path, "snapshots", revision)
        if os.path.isdir(rev_dir):
            return rev_dir

    return None
```
**EN:** This function implements `find_local_repo_dir`. It primarily calls `os.path.join`, `hf.constants.REPO_ID_SEPARATOR.join`, `os.path.isfile`, `os.path.isdir`, `open`, `f.read.strip` to complete its work. State updates are written into `cache_path`, `ref_path`, `rev_dir`, `revision`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `find_local_repo_dir`。 它主要通过调用 `os.path.join`, `hf.constants.REPO_ID_SEPARATOR.join`, `os.path.isfile`, `os.path.isdir`, `open`, `f.read.strip` 来完成任务。 状态更新主要写入 `cache_path`, `ref_path`, `rev_dir`, `revision`。 实现中使用了条件分支、上下文管理资源。

### Lines 3094-3114: Function `read_system_prompt_from_file` / 函数 `read_system_prompt_from_file`
```python
def read_system_prompt_from_file(model_name: str) -> str:
    """Read system prompt from a file in the HuggingFace cache directory.

    Args:
        model_name: The model name to construct the file path

    Returns:
        The system prompt content from the file, or empty string if file not found
    """
    try:
        local_repo_dir = find_local_repo_dir(model_name)
        if local_repo_dir:
            system_prompt_file = os.path.join(local_repo_dir, "SYSTEM_PROMPT.txt")
            if os.path.exists(system_prompt_file):
                with open(system_prompt_file, "r", encoding="utf-8") as f:
                    return f.read()

        return ""
    except Exception:
        # If anything fails, return empty string
        return ""
```
**EN:** This function implements `read_system_prompt_from_file`. It primarily calls `find_local_repo_dir`, `os.path.join`, `os.path.exists`, `open`, `f.read` to complete its work. State updates are written into `local_repo_dir`, `system_prompt_file`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该函数实现了 `read_system_prompt_from_file`。 它主要通过调用 `find_local_repo_dir`, `os.path.join`, `os.path.exists`, `open`, `f.read` 来完成任务。 状态更新主要写入 `local_repo_dir`, `system_prompt_file`。 实现中使用了条件分支、上下文管理资源、错误处理。

### Lines 3117-3123: Function `prepack_weight_if_needed` / 函数 `prepack_weight_if_needed`
```python
def prepack_weight_if_needed(weight):
    if weight.device != torch.device("cpu"):
        return weight
    if not cpu_has_amx_support():
        return weight

    return torch.ops.sgl_kernel.convert_weight_packed(weight)
```
**EN:** This function implements `prepack_weight_if_needed`. It primarily calls `torch.ops.sgl_kernel.convert_weight_packed`, `torch.device`, `cpu_has_amx_support` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `prepack_weight_if_needed`。 它主要通过调用 `torch.ops.sgl_kernel.convert_weight_packed`, `torch.device`, `cpu_has_amx_support` 来完成任务。 实现中使用了条件分支。

### Lines 3129-3130: Function `dim_is_supported` / 函数 `dim_is_supported`
```python
def dim_is_supported(weight):
    return weight.size(0) % 16 == 0 and weight.size(1) % 32 == 0
```
**EN:** This function implements `dim_is_supported`. It primarily calls `weight.size` to complete its work.
**CN:** 该函数实现了 `dim_is_supported`。 它主要通过调用 `weight.size` 来完成任务。

### Lines 3133-3176: Function `_process_weight_after_loading` / 函数 `_process_weight_after_loading`
```python
def _process_weight_after_loading(module, weight_names, transpose_dims=None) -> None:
    # Pack weight for get better performance on CPU
    devices = {getattr(module, weight_name).device for weight_name in weight_names}
    assert len(devices) == 1, f"Expects all weights to be on the same device"
    device = devices.pop()

    if transpose_dims:
        assert len(weight_names) == len(
            transpose_dims
        ), "len(weight_names) should be equal to len(transpose_dims)"

    for i, weight_name in enumerate(weight_names):
        weight_tensor = getattr(module, weight_name)

        # We don't pack weight or use intel amx backend if any weight of this module has unsupported dim.
        if not dim_is_supported(weight_tensor):
            logger.warning(
                f"Expects weight.size(0) % 16 == 0 and weight.size(1) % 32 == 0 "
                f"but {weight_tensor.size(0)=} and {weight_tensor.size(1)=} in {module}. "
                f"{module} won't use intel amx backend."
            )
            module.use_intel_amx_backend = False
            return

        if transpose_dims and transpose_dims[i]:
            weight_tensor = weight_tensor.transpose(*transpose_dims[i])

        packed_weight = torch.nn.Parameter(
# ... omitted for brevity ...
    if (
        module.use_intel_amx_backend
        and hasattr(module, "bias")
        and module.bias is not None
    ):
        module.bias = torch.nn.Parameter(module.bias.data.float(), requires_grad=False)
```
**EN:** This function implements `_process_weight_after_loading`. It primarily calls `devices.pop`, `enumerate`, `len`, `getattr`, `torch.nn.Parameter`, `setattr` to complete its work. State updates are written into `devices`, `device`, `module.use_intel_amx_backend`, `weight_tensor`, `packed_weight`, `packed_weight.__dict__`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_process_weight_after_loading`。 它主要通过调用 `devices.pop`, `enumerate`, `len`, `getattr`, `torch.nn.Parameter`, `setattr` 来完成任务。 状态更新主要写入 `devices`, `device`, `module.use_intel_amx_backend`, `weight_tensor`, `packed_weight`, `packed_weight.__dict__`。 实现中使用了条件分支、迭代逻辑。

### Lines 3179-3179: Class `PackWeightMethod` declaration / 类 `PackWeightMethod` 声明
```python
class PackWeightMethod:
```
**EN:** This class establishes `PackWeightMethod` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `process_weights_after_loading`.
**CN:** 该类将 `PackWeightMethod` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `process_weights_after_loading` 等方法。

### Lines 3180-3182: Method `PackWeightMethod.__init__` / 方法 `PackWeightMethod.__init__`
```python
    def __init__(self, weight_names, transpose_dims=None):
        self.weight_names = weight_names
        self.transpose_dims = transpose_dims
```
**EN:** This method implements `__init__` on `PackWeightMethod`. State updates are written into `self.weight_names`, `self.transpose_dims`.
**CN:** 该方法（属于 `PackWeightMethod`）实现了 `__init__`。 状态更新主要写入 `self.weight_names`, `self.transpose_dims`。

### Lines 3184-3185: Method `PackWeightMethod.process_weights_after_loading` / 方法 `PackWeightMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, module) -> None:
        _process_weight_after_loading(module, self.weight_names, self.transpose_dims)
```
**EN:** This method implements `process_weights_after_loading` on `PackWeightMethod`. It primarily calls `_process_weight_after_loading` to complete its work.
**CN:** 该方法（属于 `PackWeightMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `_process_weight_after_loading` 来完成任务。

### Lines 3188-3188: Class `LazyValue` declaration / 类 `LazyValue` 声明
```python
class LazyValue:
```
**EN:** This class establishes `LazyValue` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__getattr__`, `__getitem__`, `__setitem__`, `value`.
**CN:** 该类将 `LazyValue` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__getattr__`, `__getitem__`, `__setitem__`, `value` 等方法。

### Lines 3189-3191: Method `LazyValue.__init__` / 方法 `LazyValue.__init__`
```python
    def __init__(self, creator: Callable):
        self._creator = creator
        self._value = None
```
**EN:** This method implements `__init__` on `LazyValue`. State updates are written into `self._creator`, `self._value`.
**CN:** 该方法（属于 `LazyValue`）实现了 `__init__`。 状态更新主要写入 `self._creator`, `self._value`。

### Lines 3193-3194: Method `LazyValue.__getattr__` / 方法 `LazyValue.__getattr__`
```python
    def __getattr__(self, name):
        return getattr(self.value, name)
```
**EN:** This method implements `__getattr__` on `LazyValue`. It primarily calls `getattr` to complete its work.
**CN:** 该方法（属于 `LazyValue`）实现了 `__getattr__`。 它主要通过调用 `getattr` 来完成任务。

### Lines 3196-3197: Method `LazyValue.__getitem__` / 方法 `LazyValue.__getitem__`
```python
    def __getitem__(self, key):
        return self.value[key]
```
**EN:** This method implements `__getitem__` on `LazyValue`.
**CN:** 该方法（属于 `LazyValue`）实现了 `__getitem__`。

### Lines 3199-3200: Method `LazyValue.__setitem__` / 方法 `LazyValue.__setitem__`
```python
    def __setitem__(self, key, value):
        self.value[key] = value
```
**EN:** This method implements `__setitem__` on `LazyValue`. State updates are written into `self.value`.
**CN:** 该方法（属于 `LazyValue`）实现了 `__setitem__`。 状态更新主要写入 `self.value`。

### Lines 3202-3207: Method `LazyValue.value` / 方法 `LazyValue.value`
```python
    @property
    def value(self):
        if self._creator is not None:
            self._value = self._creator()
            self._creator = None
        return self._value
```
**EN:** This method implements `value` on `LazyValue`. It primarily calls `self._creator` to complete its work. State updates are written into `self._value`, `self._creator`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `LazyValue`）实现了 `value`。 它主要通过调用 `self._creator` 来完成任务。 状态更新主要写入 `self._value`, `self._creator`。 实现中使用了条件分支。

### Lines 3210-3220: Function `dynamic_import` / 函数 `dynamic_import`
```python
def dynamic_import(func_path: str):
    parts = func_path.split(".")
    if len(parts) < 2:
        raise ValueError(
            "func_path should contain both module name and func name (such as 'module.func')"
        )
    module_path = ".".join(parts[:-1])
    func_name = parts[-1]
    module = importlib.import_module(module_path)
    func = getattr(module, func_name)
    return func
```
**EN:** This function implements `dynamic_import`. It primarily calls `func_path.split`, `join`, `importlib.import_module`, `getattr`, `len`, `ValueError` to complete its work. State updates are written into `parts`, `module_path`, `func_name`, `module`, `func`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `dynamic_import`。 它主要通过调用 `func_path.split`, `join`, `importlib.import_module`, `getattr`, `len`, `ValueError` 来完成任务。 状态更新主要写入 `parts`, `module_path`, `func_name`, `module`, `func`。 实现中使用了条件分支。

### Lines 3223-3229: Function `gc_object_counts` / 函数 `gc_object_counts`
```python
def gc_object_counts():
    import gc

    g0 = len(gc.get_objects(0))
    g1 = len(gc.get_objects(1))
    g2 = len(gc.get_objects(2))
    return g0, g1, g2
```
**EN:** This function implements `gc_object_counts`. It primarily calls `len`, `gc.get_objects` to complete its work. State updates are written into `g0`, `g1`, `g2`.
**CN:** 该函数实现了 `gc_object_counts`。 它主要通过调用 `len`, `gc.get_objects` 来完成任务。 状态更新主要写入 `g0`, `g1`, `g2`。

### Lines 3232-3250: Function `configure_gc_warning` / 函数 `configure_gc_warning`
```python
def configure_gc_warning(warn_threshold_secs):
    import gc

    gc_start_time = {}

    def gc_callback(phase, info):
        gen = info.get("generation", "?")
        if phase == "start":
            gc_start_time[gen] = time.time()
        elif phase == "stop":
            duration = time.time() - gc_start_time.get(gen, time.time())
            if duration > warn_threshold_secs:
                g0, g1, g2 = gc_object_counts()
                logger.warn(
                    f"LONG GARBAGE COLLECTION DETECTED | Generation {gen} | Duration: {duration:.4f}s | # Objects: gen0={g0}, gen1={g1}, gen2={g2} | "
                    f"This may cause latency jitter. Consider calling the freeze_gc API after sending a few warmup requests."
                )

    gc.callbacks.append(gc_callback)
```
**EN:** This function implements `configure_gc_warning`. It primarily calls `gc.callbacks.append`, `info.get`, `time.time`, `gc_start_time.get`, `gc_object_counts`, `logger.warn` to complete its work. State updates are written into `gc_start_time`, `gen`, `duration`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `configure_gc_warning`。 它主要通过调用 `gc.callbacks.append`, `info.get`, `time.time`, `gc_start_time.get`, `gc_object_counts`, `logger.warn` 来完成任务。 状态更新主要写入 `gc_start_time`, `gen`, `duration`。 实现中使用了条件分支。

### Lines 3253-3262: Function `freeze_gc` / 函数 `freeze_gc`
```python
def freeze_gc(context: str):
    g0_before, g1_before, g2_before = gc_object_counts()
    gc.freeze()
    g0_after, g1_after, g2_after = gc_object_counts()
    logger.info(
        f"Freezing GC in {context} process. "
        f"gen0: {g0_before}->{g0_after}, "
        f"gen1: {g1_before}->{g1_after}, "
        f"gen2: {g2_before}->{g2_after}"
    )
```
**EN:** This function implements `freeze_gc`. It primarily calls `gc_object_counts`, `gc.freeze`, `logger.info` to complete its work.
**CN:** 该函数实现了 `freeze_gc`。 它主要通过调用 `gc_object_counts`, `gc.freeze`, `logger.info` 来完成任务。

### Lines 3265-3285: Function `configure_gc_logger` / 函数 `configure_gc_logger`
```python
def configure_gc_logger():
    logger.info("Enable GC Logger")

    gc_start_time = {}

    def gc_callback(phase, info):
        gen = info.get("generation", "?")
        if phase == "start":
            gc_start_time[gen] = time.time()
            logger.info(f"GC start: Time {time.time()} | Generation {gen}")
        elif phase == "stop":
            duration = time.time() - gc_start_time.get(gen, time.time())
            collected = info.get("collected", "?")
            uncollectable = info.get("uncollectable", "?")
            logger.info(
                f"GC end: Time {time.time()} | Generation {gen} | "
                f"Duration: {duration:.4f}s | Collected: {collected} | Uncollectable: {uncollectable} "
                f'{"(LONG GC)" if duration > 0.1 else ""}'
            )

    gc.callbacks.append(gc_callback)
```
**EN:** This function implements `configure_gc_logger`. It primarily calls `logger.info`, `gc.callbacks.append`, `info.get`, `time.time`, `gc_start_time.get` to complete its work. State updates are written into `gc_start_time`, `gen`, `duration`, `collected`, `uncollectable`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `configure_gc_logger`。 它主要通过调用 `logger.info`, `gc.callbacks.append`, `info.get`, `time.time`, `gc_start_time.get` 来完成任务。 状态更新主要写入 `gc_start_time`, `gen`, `duration`, `collected`, `uncollectable`。 实现中使用了条件分支。

### Lines 3289-3290: Function `ceil_align` / 函数 `ceil_align`
```python
def ceil_align(x: int, y: int) -> int:
    return ceil_div(x, y) * y
```
**EN:** This function implements `ceil_align`. It primarily calls `ceil_div` to complete its work.
**CN:** 该函数实现了 `ceil_align`。 它主要通过调用 `ceil_div` 来完成任务。

### Lines 3294-3295: Function `ceil_div` / 函数 `ceil_div`
```python
def ceil_div(x: int, y: int) -> int:
    return (x + y - 1) // y
```
**EN:** This function implements `ceil_div`.
**CN:** 该函数实现了 `ceil_div`。

### Lines 3298-3320: Function `parse_lscpu_topology` / 函数 `parse_lscpu_topology`
```python
def parse_lscpu_topology():
    try:
        # Get CPU topology: CPU,Core,Socket,Node
        output = subprocess.check_output(
            ["lscpu", "-p=CPU,Core,Socket,Node"], text=True
        )
    except Exception as e:
        raise RuntimeError(f"Unexpected error running 'lscpu': {e}")

    # Parse only data lines (skip comments)
    cpu_info = []
    for line in output.splitlines():
        if not line.startswith("#"):
            parts = line.strip().split(",")
            if len(parts) != 4:
                logger.warning("Skipping malformed lscpu line: %s", line.strip())
                continue
            cpu = int(parts[0])  # CPU id must always be present
            core, socket, node = [int(p) if p else 0 for p in parts[1:]]
            cpu_info.append((cpu, core, socket, node))

    # [(0,0,0,0),(1,1,0,0),...,(43,43,0,1),...,(256,0,0,0),...]
    return cpu_info
```
**EN:** This function implements `parse_lscpu_topology`. It primarily calls `output.splitlines`, `subprocess.check_output`, `RuntimeError`, `line.startswith`, `line.strip.split`, `int` to complete its work. State updates are written into `cpu_info`, `output`, `parts`, `cpu`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `parse_lscpu_topology`。 它主要通过调用 `output.splitlines`, `subprocess.check_output`, `RuntimeError`, `line.startswith`, `line.strip.split`, `int` 来完成任务。 状态更新主要写入 `cpu_info`, `output`, `parts`, `cpu`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 3323-3354: Function `get_physical_cpus_by_numa` / 函数 `get_physical_cpus_by_numa`
```python
def get_physical_cpus_by_numa():
    cpu_info = parse_lscpu_topology()

    # Map NUMA node -> set of (core_id, socket) to avoid duplicates
    # 0: {(0,0): 0, (1, 0): 1,...}
    # ...
    # 5: {(214,1): 214, (215,1): 215}
    physical_by_node = defaultdict(dict)  # node -> core_id -> cpu_id

    for cpu, core, socket, node in cpu_info:
        key = (core, socket)
        if key not in physical_by_node[node]:
            physical_by_node[node][
                key
            ] = cpu  # pick first CPU seen for that physical core

    # Retrieves CPUs that the current process is allowed to run on
    cpus_allowed_list = psutil.Process().cpu_affinity()

    # Convert to list of physical CPUs per node
    # 0: [0,1,2,...,42]
    # ...
    # 2: [86,87,...,127]
    # ...
    # 5: [214,215,...,255]
    node_to_cpus = {}
    for node, core_to_cpu in physical_by_node.items():
        cpus = sorted(core_to_cpu.values())
        allowed_cpus = set(cpus).intersection(cpus_allowed_list)
        node_to_cpus[node] = allowed_cpus

    return node_to_cpus
```
**EN:** This function implements `get_physical_cpus_by_numa`. It primarily calls `parse_lscpu_topology`, `defaultdict`, `psutil.Process.cpu_affinity`, `physical_by_node.items`, `sorted`, `set.intersection` to complete its work. State updates are written into `cpu_info`, `physical_by_node`, `cpus_allowed_list`, `node_to_cpus`, `key`, `cpus`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `get_physical_cpus_by_numa`。 它主要通过调用 `parse_lscpu_topology`, `defaultdict`, `psutil.Process.cpu_affinity`, `physical_by_node.items`, `sorted`, `set.intersection` 来完成任务。 状态更新主要写入 `cpu_info`, `physical_by_node`, `cpus_allowed_list`, `node_to_cpus`, `key`, `cpus`。 实现中使用了条件分支、迭代逻辑。

### Lines 3358-3366: Function `get_cpu_ids_by_node` / 函数 `get_cpu_ids_by_node`
```python
def get_cpu_ids_by_node():
    node_to_cpus = get_physical_cpus_by_numa()
    # Sort by NUMA node index
    cpu_ids = [
        ",".join(map(str, sorted(node_to_cpus[node]))) for node in sorted(node_to_cpus)
    ]

    # ['0,1,2,3', '4,5,6,7', '8,9,10,11', '12,13,14,15', '16,17,18,19', '20,21,22,23']
    return cpu_ids
```
**EN:** This function implements `get_cpu_ids_by_node`. It primarily calls `get_physical_cpus_by_numa`, `join`, `map`, `sorted` to complete its work. State updates are written into `node_to_cpus`, `cpu_ids`.
**CN:** 该函数实现了 `get_cpu_ids_by_node`。 它主要通过调用 `get_physical_cpus_by_numa`, `join`, `map`, `sorted` 来完成任务。 状态更新主要写入 `node_to_cpus`, `cpu_ids`。

### Lines 3369-3375: Function `is_shm_available` / 函数 `is_shm_available`
```python
def is_shm_available(dtype, world_size, local_size):
    return (
        (cpu_has_amx_support() or is_host_cpu_arm64())
        and dtype in [torch.bfloat16, torch.float16, torch.float]
        and world_size >= 1
        and world_size == local_size
    )
```
**EN:** This function implements `is_shm_available`. It primarily calls `cpu_has_amx_support`, `is_host_cpu_arm64` to complete its work.
**CN:** 该函数实现了 `is_shm_available`。 它主要通过调用 `cpu_has_amx_support`, `is_host_cpu_arm64` 来完成任务。

### Lines 3378-3420: Function `lru_cache_frozenset` / 函数 `lru_cache_frozenset`
```python
def lru_cache_frozenset(maxsize=128):
    def _to_hashable(o):
        try:
            hash(o)
            return o
        except TypeError:
            # Not hashable; convert based on type
            if isinstance(o, (dict)):
                return frozenset(
                    (_to_hashable(k), _to_hashable(v)) for k, v in o.items()
                )
            elif isinstance(o, set):
                return frozenset(_to_hashable(v) for v in o)
            elif isinstance(o, (list, tuple)) or (
                isinstance(o, Sequence) and not isinstance(o, (str, bytes))
            ):
                return tuple(_to_hashable(v) for v in o)
            else:
                raise TypeError(f"Cannot make hashable: {type(o)}")

    def decorator(func):
        cache = OrderedDict()

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            h_args = tuple(_to_hashable(a) for a in args)
            h_kwargs = frozenset(
                (_to_hashable(k), _to_hashable(v)) for k, v in kwargs.items()
# ... omitted for brevity ...
            return result

        wrapper.cache_clear = cache.clear  # For manual cache clearing
        return wrapper

    return decorator
```
**EN:** This function implements `lru_cache_frozenset`. It primarily calls `OrderedDict`, `functools.wraps`, `hash`, `tuple`, `frozenset`, `func` to complete its work. State updates are written into `cache`, `wrapper.cache_clear`, `h_args`, `h_kwargs`, `key`, `result`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `lru_cache_frozenset`。 它主要通过调用 `OrderedDict`, `functools.wraps`, `hash`, `tuple`, `frozenset`, `func` 来完成任务。 状态更新主要写入 `cache`, `wrapper.cache_clear`, `h_args`, `h_kwargs`, `key`, `result`。 实现中使用了条件分支、错误处理。

### Lines 3423-3446: Function `apply_module_patch` / 函数 `apply_module_patch`
```python
def apply_module_patch(target_module, target_function, wrappers):
    original_module, original_function = parse_module_path(
        target_module, target_function, False
    )

    original_function_id = id(original_function)

    candidate = original_function
    for wrapper in wrappers:
        candidate = wrapper(candidate)
    if target_function is not None:
        setattr(original_module, target_function, candidate)

    for key, value in sys.modules.copy().items():
        try:
            if (
                target_function is not None
                and hasattr(value, target_function)
                and id(getattr(value, target_function)) == original_function_id
            ):
                setattr(value, target_function, candidate)
        except ImportError as e:
            # Ignore some modules reporting ImportError when calling hasattr
            logger.warning(f"Ignore {value} reports ImportError with:\n{str(e)}")
```
**EN:** This function implements `apply_module_patch`. It primarily calls `parse_module_path`, `id`, `sys.modules.copy.items`, `wrapper`, `setattr`, `sys.modules.copy` to complete its work. State updates are written into `original_function_id`, `candidate`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `apply_module_patch`。 它主要通过调用 `parse_module_path`, `id`, `sys.modules.copy.items`, `wrapper`, `setattr`, `sys.modules.copy` 来完成任务。 状态更新主要写入 `original_function_id`, `candidate`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 3449-3522: Function `parse_module_path` / 函数 `parse_module_path`
```python
def parse_module_path(module_path, function_name, create_dummy):
    from importlib.machinery import ModuleSpec

    def create_dummy_module(full_path, parent=None):
        """Create and register a placeholder module"""
        dummy = types.ModuleType(full_path)
        dummy.__file__ = "vllm_ascend.dummy_module.py"
        dummy.__spec__ = ModuleSpec(full_path, None)
        sys.modules[full_path] = dummy
        if parent:
            setattr(parent, full_path.split(".")[-1], dummy)
        return dummy

    def create_placeholder_function(func_name):
        """Create dummy function that raises when called"""

        def placeholder(*args, **kwargs):
            raise NotImplementedError(f"Function {func_name} is a placeholder")

        placeholder.__name__ = func_name
        return placeholder

    modules = module_path.split(".")
    current_module = None
    processed_path = []

    for idx, part in enumerate(modules):
        current_path = ".".join(modules[: idx + 1])
# ... omitted for brevity ...
                setattr(final_module, function_name, ph_func)
            else:
                setattr(final_module, function_name, None)
        return final_module, getattr(final_module, function_name)

    return final_module, None
```
**EN:** This function implements `parse_module_path`. It primarily calls `module_path.split`, `enumerate`, `types.ModuleType`, `ModuleSpec`, `join`, `processed_path.append` to complete its work. State updates are written into `modules`, `current_module`, `processed_path`, `final_module`, `dummy`, `dummy.__file__`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `parse_module_path`。 它主要通过调用 `module_path.split`, `enumerate`, `types.ModuleType`, `ModuleSpec`, `join`, `processed_path.append` 来完成任务。 状态更新主要写入 `modules`, `current_module`, `processed_path`, `final_module`, `dummy`, `dummy.__file__`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 3525-3533: Function `mxfp_supported` / 函数 `mxfp_supported`
```python
def mxfp_supported():
    """
    Returns whether the current platform supports MX types.
    """
    if torch.version.hip:
        gcn_arch = torch.cuda.get_device_properties(0).gcnArchName
        return any(gfx in gcn_arch for gfx in ["gfx95"])
    else:
        return False
```
**EN:** This function implements `mxfp_supported`. It primarily calls `any`, `torch.cuda.get_device_properties` to complete its work. State updates are written into `gcn_arch`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `mxfp_supported`。 它主要通过调用 `any`, `torch.cuda.get_device_properties` 来完成任务。 状态更新主要写入 `gcn_arch`。 实现中使用了条件分支。

### Lines 3536-3545: Function `is_gfx95_supported` / 函数 `is_gfx95_supported`
```python
@lru_cache(maxsize=1)
def is_gfx95_supported():
    """
    Returns whether the current platform supports MX types.
    """
    if torch.version.hip:
        gcn_arch = torch.cuda.get_device_properties(0).gcnArchName
        return any(gfx in gcn_arch for gfx in ["gfx95"])
    else:
        return False
```
**EN:** This function implements `is_gfx95_supported`. It primarily calls `lru_cache`, `any`, `torch.cuda.get_device_properties` to complete its work. State updates are written into `gcn_arch`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_gfx95_supported`。 它主要通过调用 `lru_cache`, `any`, `torch.cuda.get_device_properties` 来完成任务。 状态更新主要写入 `gcn_arch`。 实现中使用了条件分支。

### Lines 3548-3551: Function `get_hip_version` / 函数 `get_hip_version`
```python
def get_hip_version():
    if torch.version.hip:
        return tuple(map(int, torch.version.hip.split("-")[0].split(".")))
    return (0, 0, 0)
```
**EN:** This function implements `get_hip_version`. It primarily calls `tuple`, `map`, `torch.version.hip.split.split`, `torch.version.hip.split` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_hip_version`。 它主要通过调用 `tuple`, `map`, `torch.version.hip.split.split`, `torch.version.hip.split` 来完成任务。 实现中使用了条件分支。

### Lines 3555-3573: Constants and shared state / 常量与共享状态
```python
SUPPORTED_LORA_TARGET_MODULES = [
    "q_proj",
    "k_proj",
    "v_proj",
    "o_proj",
    "q_a_proj",
    "kv_a_proj_with_mqa",
    "q_b_proj",
    "kv_b_proj",
    "gate_proj",
    "up_proj",
    "down_proj",
    "qkv_proj",
    "gate_up_proj",
    "embed_tokens",
    "lm_head",
]

LORA_TARGET_ALL_MODULES = "all"
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `SUPPORTED_LORA_TARGET_MODULES`, `LORA_TARGET_ALL_MODULES`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `SUPPORTED_LORA_TARGET_MODULES`, `LORA_TARGET_ALL_MODULES`。

### Lines 3576-3583: Class `ConcurrentCounter` declaration / 类 `ConcurrentCounter` 声明
```python
class ConcurrentCounter:
    """
    An asynchronous counter for managing concurrent tasks that need
    coordinated increments, decrements, and waiting until the count reaches zero.

    This class is useful for scenarios like tracking the number of in-flight tasks
    and waiting for them to complete.
    """
```
**EN:** This class establishes `ConcurrentCounter` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `value`, `__repr__`, `increment`, `decrement`, `wait_for`.
**CN:** 该类将 `ConcurrentCounter` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `value`, `__repr__`, `increment`, `decrement`, `wait_for` 等方法。

### Lines 3585-3593: Method `ConcurrentCounter.__init__` / 方法 `ConcurrentCounter.__init__`
```python
    def __init__(self, initial: int = 0):
        """
        Initialize the counter with an optional initial value.

        Args:
            initial (int): The initial value of the counter. Default is 0.
        """
        self._count = initial
        self._condition = asyncio.Condition()
```
**EN:** This method implements `__init__` on `ConcurrentCounter`. It primarily calls `asyncio.Condition` to complete its work. State updates are written into `self._count`, `self._condition`.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `__init__`。 它主要通过调用 `asyncio.Condition` 来完成任务。 状态更新主要写入 `self._count`, `self._condition`。

### Lines 3595-3606: Method `ConcurrentCounter.value` / 方法 `ConcurrentCounter.value`
```python
    def value(self) -> int:
        """
        Return the current value of the counter.

        Note:
            This method is not synchronized. It may return a stale value
            if other coroutines are concurrently modifying the counter.

        Returns:
            int: The current counter value.
        """
        return self._count
```
**EN:** This method implements `value` on `ConcurrentCounter`.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `value`。

### Lines 3608-3610: Method `ConcurrentCounter.__repr__` / 方法 `ConcurrentCounter.__repr__`
```python
    def __repr__(self) -> str:
        """Return an informative string representation of the counter."""
        return f"<ConcurrentCounter value={self.value()}>"
```
**EN:** This method implements `__repr__` on `ConcurrentCounter`. It primarily calls `self.value` to complete its work.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `__repr__`。 它主要通过调用 `self.value` 来完成任务。

### Lines 3612-3623: Method `ConcurrentCounter.increment` / 方法 `ConcurrentCounter.increment`
```python
    async def increment(self, n: int = 1, notify_all: bool = True):
        """
        Atomically increment the counter by a given amount and notify all waiters.

        Args:
            n (int): The amount to increment the counter by. Default is 1.
            notify_all (bool): Whether to notify all waiters after incrementing. Default is True.
        """
        async with self._condition:
            self._count += n
            if notify_all:
                self._condition.notify_all()
```
**EN:** This async method implements `increment` on `ConcurrentCounter`. It primarily calls `self._condition.notify_all` to complete its work. State updates are written into `self._count`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `increment`。 它主要通过调用 `self._condition.notify_all` 来完成任务。 状态更新主要写入 `self._count`。 实现中使用了条件分支、上下文管理资源。

### Lines 3625-3636: Method `ConcurrentCounter.decrement` / 方法 `ConcurrentCounter.decrement`
```python
    async def decrement(self, n: int = 1, notify_all: bool = True):
        """
        Atomically decrement the counter by a given amount and notify all waiters.

        Args:
            n (int): The amount to decrement the counter by. Default is 1.
            notify_all (bool): Whether to notify all waiters after decrementing. Default is True.
        """
        async with self._condition:
            self._count -= n
            if notify_all:
                self._condition.notify_all()
```
**EN:** This async method implements `decrement` on `ConcurrentCounter`. It primarily calls `self._condition.notify_all` to complete its work. State updates are written into `self._count`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `decrement`。 它主要通过调用 `self._condition.notify_all` 来完成任务。 状态更新主要写入 `self._count`。 实现中使用了条件分支、上下文管理资源。

### Lines 3638-3650: Method `ConcurrentCounter.wait_for` / 方法 `ConcurrentCounter.wait_for`
```python
    async def wait_for(self, condition: Callable[[int], bool]):
        """
        Asynchronously wait until the counter satisfies a given condition.

        This suspends the calling coroutine without blocking the thread, allowing
        other tasks to run while waiting. When the condition is met, the coroutine resumes.

        Args:
            condition (Callable[[int], bool]): A function that takes the current counter value
                and returns True when the condition is satisfied.
        """
        async with self._condition:
            await self._condition.wait_for(lambda: condition(self._count))
```
**EN:** This async method implements `wait_for` on `ConcurrentCounter`. It primarily calls `self._condition.wait_for`, `condition` to complete its work. The implementation relies on context-managed resources, async awaits.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `wait_for`。 它主要通过调用 `self._condition.wait_for`, `condition` 来完成任务。 实现中使用了上下文管理资源、异步等待。

### Lines 3652-3659: Method `ConcurrentCounter.wait_for_zero` / 方法 `ConcurrentCounter.wait_for_zero`
```python
    async def wait_for_zero(self):
        """
        Asynchronously wait until the counter reaches zero.

        This suspends the calling coroutine without blocking the thread, allowing
        other tasks to run while waiting. When the counter becomes zero, the coroutine resumes.
        """
        await self.wait_for(lambda count: count == 0)
```
**EN:** This async method implements `wait_for_zero` on `ConcurrentCounter`. It primarily calls `self.wait_for` to complete its work. The implementation relies on async awaits.
**CN:** 该方法（属于 `ConcurrentCounter`）实现了 `wait_for_zero`。 它主要通过调用 `self.wait_for` 来完成任务。 实现中使用了异步等待。

### Lines 3662-3672: Function `is_triton_kernels_available` / 函数 `is_triton_kernels_available`
```python
@lru_cache(maxsize=1)
def is_triton_kernels_available() -> bool:
    if importlib.util.find_spec("triton_kernels") is None:
        return False
    try:
        ragged_metadata_spec = importlib.util.find_spec(
            "triton_kernels.tensor_details.ragged_tensor"
        )
    except ModuleNotFoundError:
        return False
    return ragged_metadata_spec is not None
```
**EN:** This function implements `is_triton_kernels_available`. It primarily calls `lru_cache`, `importlib.util.find_spec` to complete its work. State updates are written into `ragged_metadata_spec`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `is_triton_kernels_available`。 它主要通过调用 `lru_cache`, `importlib.util.find_spec` 来完成任务。 状态更新主要写入 `ragged_metadata_spec`。 实现中使用了条件分支、错误处理。

### Lines 3675-3685: Function `get_nvidia_driver_version` / 函数 `get_nvidia_driver_version`
```python
@lru_cache(maxsize=1)
def get_nvidia_driver_version() -> tuple:
    """Return the NVIDIA driver version as a tuple of ints, e.g. (595, 58, 3).
    Returns (0,) on failure."""
    version_str = get_nvidia_driver_version_str()
    if version_str is None:
        return (0,)
    try:
        return tuple(int(x) for x in version_str.split("."))
    except ValueError:
        return (0,)
```
**EN:** This function implements `get_nvidia_driver_version`. It primarily calls `lru_cache`, `get_nvidia_driver_version_str`, `tuple`, `int`, `version_str.split` to complete its work. State updates are written into `version_str`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_nvidia_driver_version`。 它主要通过调用 `lru_cache`, `get_nvidia_driver_version_str`, `tuple`, `int`, `version_str.split` 来完成任务。 状态更新主要写入 `version_str`。 实现中使用了条件分支、错误处理。

### Lines 3688-3707: Function `get_nvidia_driver_version_str` / 函数 `get_nvidia_driver_version_str`
```python
@lru_cache(maxsize=1)
def get_nvidia_driver_version_str() -> str:
    """Return the NVIDIA driver version string, e.g. '595.58.03'.
    Returns None on failure."""
    try:
        result = subprocess.run(
            [
                "nvidia-smi",
                "--query-gpu=driver_version",
                "--format=csv,noheader,nounits",
            ],
            capture_output=True,
            text=True,
            check=True,
            timeout=10,
        )
        version_str = result.stdout.strip().split("\n")[0].strip()
        return version_str if version_str else None
    except (subprocess.CalledProcessError, FileNotFoundError, ValueError):
        return None
```
**EN:** This function implements `get_nvidia_driver_version_str`. It primarily calls `lru_cache`, `subprocess.run`, `result.stdout.strip.split.strip`, `result.stdout.strip.split`, `result.stdout.strip` to complete its work. State updates are written into `result`, `version_str`. The implementation relies on error handling.
**CN:** 该函数实现了 `get_nvidia_driver_version_str`。 它主要通过调用 `lru_cache`, `subprocess.run`, `result.stdout.strip.split.strip`, `result.stdout.strip.split`, `result.stdout.strip` 来完成任务。 状态更新主要写入 `result`, `version_str`。 实现中使用了错误处理。

### Lines 3710-3717: Function `check_cuda_result` / 函数 `check_cuda_result`
```python
def check_cuda_result(raw_output):
    import cuda.bindings.runtime as cuda_rt

    err, *results = raw_output
    if err != cuda_rt.cudaError_t.cudaSuccess:
        raise Exception(f"CUDA error: {err}")

    return results
```
**EN:** This function implements `check_cuda_result`. It primarily calls `Exception` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `check_cuda_result`。 它主要通过调用 `Exception` 来完成任务。 实现中使用了条件分支。

### Lines 3720-3726: Function `get_cuda_driver_bindings` / 函数 `get_cuda_driver_bindings`
```python
def get_cuda_driver_bindings():
    try:
        from cuda.bindings import driver as cuda_driver
    except ImportError:
        from cuda import cuda as cuda_driver

    return cuda_driver
```
**EN:** This function implements `get_cuda_driver_bindings`. The implementation relies on error handling.
**CN:** 该函数实现了 `get_cuda_driver_bindings`。 实现中使用了错误处理。

### Lines 3729-3749: Function `get_physical_device_id` / 函数 `get_physical_device_id`
```python
def get_physical_device_id(pytorch_device_id: int) -> int:
    """
    Convert PyTorch logical device ID to physical device ID.

    When CUDA_VISIBLE_DEVICES is set, maps the logical device ID (as seen by PyTorch)
    to the actual physical device ID. If CUDA_VISIBLE_DEVICES is not set, returns
    the device ID unchanged.

    Args:
        pytorch_device_id: The logical device ID from PyTorch (e.g., torch.cuda.current_device())

    Returns:
        The physical device ID
    """
    device_idx = int(pytorch_device_id)
    cuda_visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES", None)
    if cuda_visible_devices:
        device_list = cuda_visible_devices.split(",")
        return int(device_list[device_idx])
    else:
        return device_idx
```
**EN:** This function implements `get_physical_device_id`. It primarily calls `int`, `os.environ.get`, `cuda_visible_devices.split` to complete its work. State updates are written into `device_idx`, `cuda_visible_devices`, `device_list`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_physical_device_id`。 它主要通过调用 `int`, `os.environ.get`, `cuda_visible_devices.split` 来完成任务。 状态更新主要写入 `device_idx`, `cuda_visible_devices`, `device_list`。 实现中使用了条件分支。

### Lines 3752-3772: Function `get_device_sm_nvidia_smi` / 函数 `get_device_sm_nvidia_smi`
```python
def get_device_sm_nvidia_smi():
    try:
        # Run nvidia-smi command and capture output
        result = subprocess.run(
            ["nvidia-smi", "--query-gpu=compute_cap", "--format=csv,noheader"],
            capture_output=True,
            text=True,
            check=True,
        )

        # Get the first line of output (assuming at least one GPU exists)
        compute_cap_str = result.stdout.strip().split("\n")[0]

        # Convert string (e.g., "9.0") to tuple of integers (9, 0)
        major, minor = map(int, compute_cap_str.split("."))
        return (major, minor)

    except (subprocess.CalledProcessError, FileNotFoundError, ValueError) as e:
        # Handle cases where nvidia-smi isn't available or output is unexpected
        print(f"Error getting compute capability: {e}")
        return (0, 0)  # Default/fallback value
```
**EN:** This function implements `get_device_sm_nvidia_smi`. It primarily calls `subprocess.run`, `map`, `result.stdout.strip.split`, `compute_cap_str.split`, `print`, `result.stdout.strip` to complete its work. State updates are written into `result`, `compute_cap_str`. The implementation relies on error handling.
**CN:** 该函数实现了 `get_device_sm_nvidia_smi`。 它主要通过调用 `subprocess.run`, `map`, `result.stdout.strip.split`, `compute_cap_str.split`, `print`, `result.stdout.strip` 来完成任务。 状态更新主要写入 `result`, `compute_cap_str`。 实现中使用了错误处理。

### Lines 3775-3781: Function `json_list_type` / 函数 `json_list_type`
```python
def json_list_type(value):
    try:
        return orjson.loads(value)
    except json.JSONDecodeError:
        raise argparse.ArgumentTypeError(
            f"Invalid JSON list: {value}. Please provide a valid JSON list."
        )
```
**EN:** This function implements `json_list_type`. It primarily calls `orjson.loads`, `argparse.ArgumentTypeError` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `json_list_type`。 它主要通过调用 `orjson.loads`, `argparse.ArgumentTypeError` 来完成任务。 实现中使用了错误处理。

### Lines 3784-3807: Function `maybe_reindex_device_id` / 函数 `maybe_reindex_device_id`
```python
@contextmanager
def maybe_reindex_device_id(gpu_id: int):

    if envs.SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS.get() is False or not is_cuda_alike():
        yield gpu_id
        return

    original_cuda_visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES")
    if original_cuda_visible_devices:
        cuda_visible_devices = original_cuda_visible_devices.split(",")
    else:
        cuda_visible_devices = []

    str_gpu_id = cuda_visible_devices[gpu_id] if cuda_visible_devices else str(gpu_id)
    os.environ["CUDA_VISIBLE_DEVICES"] = str_gpu_id

    logger.debug(f"Set CUDA_VISIBLE_DEVICES to {str_gpu_id}")

    yield 0

    if original_cuda_visible_devices:
        os.environ["CUDA_VISIBLE_DEVICES"] = original_cuda_visible_devices
    else:
        del os.environ["CUDA_VISIBLE_DEVICES"]
```
**EN:** This function implements `maybe_reindex_device_id`. It primarily calls `os.environ.get`, `logger.debug`, `original_cuda_visible_devices.split`, `str`, `envs.SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS.get`, `is_cuda_alike` to complete its work. State updates are written into `original_cuda_visible_devices`, `str_gpu_id`, `os.environ`, `cuda_visible_devices`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `maybe_reindex_device_id`。 它主要通过调用 `os.environ.get`, `logger.debug`, `original_cuda_visible_devices.split`, `str`, `envs.SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS.get`, `is_cuda_alike` 来完成任务。 状态更新主要写入 `original_cuda_visible_devices`, `str_gpu_id`, `os.environ`, `cuda_visible_devices`。 实现中使用了条件分支。

### Lines 3810-3820: Function `get_extend_input_len_swa_limit` / 函数 `get_extend_input_len_swa_limit`
```python
def get_extend_input_len_swa_limit(
    sliding_window_size: int, chunked_prefill_size: int, page_size: int
) -> int:
    # 1. a factor of 2x is because each prefill contains chunked_prefill_size tokens,
    #    and between prefills, we run swa_radix_cache.cache_unfinished_req(),
    #    so we unlock the previously locked nodes.
    # 2. max is to handle the case that chunked_prefill_size is larger than sliding_window_size.
    #    in that case, each prefill contains chunked_prefill_size tokens,
    #    and we can only free out-of-sliding-window kv indices after each prefill.
    # 3. page_size is because we want to have 1 token extra for generated tokens.
    return page_size + 2 * max(sliding_window_size, chunked_prefill_size)
```
**EN:** This function implements `get_extend_input_len_swa_limit`. It primarily calls `max` to complete its work.
**CN:** 该函数实现了 `get_extend_input_len_swa_limit`。 它主要通过调用 `max` 来完成任务。

### Lines 3823-3846: Function `get_num_new_pages` / 函数 `get_num_new_pages`
```python
def get_num_new_pages(
    seq_lens: torch.Tensor,
    page_size: int,
    prefix_lens: Optional[torch.Tensor] = None,
    decode: bool = False,
) -> torch.Tensor:
    """
    Get the number of new pages for the given prefix and sequence lengths.
    We use cpu tensors to avoid blocking kernel launch.
    """
    cpu_device = torch.device("cpu")
    assert seq_lens.device == cpu_device

    if prefix_lens is None or decode:
        # NOTE: Special case for handling decode, which prefix lens is `seq_lens - 1`.
        assert decode
        return (seq_lens % page_size == 1).int().sum().item()

    assert prefix_lens.device == cpu_device
    num_pages_after = (seq_lens + page_size - 1) // page_size
    num_pages_before = (prefix_lens + page_size - 1) // page_size
    num_new_pages = num_pages_after - num_pages_before
    sum_num_new_pages = torch.sum(num_new_pages).to(torch.int64)
    return sum_num_new_pages.item()
```
**EN:** This function implements `get_num_new_pages`. It primarily calls `torch.device`, `torch.sum.to`, `sum_num_new_pages.item`, `int.sum.item`, `torch.sum`, `int.sum` to complete its work. State updates are written into `cpu_device`, `num_pages_after`, `num_pages_before`, `num_new_pages`, `sum_num_new_pages`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_num_new_pages`。 它主要通过调用 `torch.device`, `torch.sum.to`, `sum_num_new_pages.item`, `int.sum.item`, `torch.sum`, `int.sum` 来完成任务。 状态更新主要写入 `cpu_device`, `num_pages_after`, `num_pages_before`, `num_new_pages`, `sum_num_new_pages`。 实现中使用了条件分支。

### Lines 3849-3855: Class `CachedKernel` declaration / 类 `CachedKernel` 声明
```python
class CachedKernel:
    """
    Wrapper that allows kernel[grid](...) syntax with caching based on a key function.

    This wrapper caches compiled Triton kernels based on keys extracted by a
    user-provided key function to avoid redundant compilations.
    """
```
**EN:** This class establishes `CachedKernel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__getitem__`, `_build_args`, `_clear_cache`.
**CN:** 该类将 `CachedKernel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__getitem__`, `_build_args`, `_clear_cache` 等方法。

### Lines 3857-3876: Method `CachedKernel.__init__` / 方法 `CachedKernel.__init__`
```python
    def __init__(self, fn, key_fn=None):
        self.fn = fn
        assert isinstance(fn, triton.runtime.jit.JITFunction)

        original_fn = fn.fn
        self.signature = inspect.signature(original_fn)
        self.param_names = tuple(self.signature.parameters.keys())
        self.num_args = len(self.param_names)

        # Check that no parameters have default values
        for name, param in self.signature.parameters.items():
            assert (
                param.default is inspect.Parameter.empty
            ), f"Parameter '{name}' has a default value. Default parameters are not supported in cached kernels."

        functools.update_wrapper(self, original_fn)
        self.kernel_cache = {}

        # Store the key function
        self.key_fn = key_fn
```
**EN:** This method implements `__init__` on `CachedKernel`. It primarily calls `isinstance`, `inspect.signature`, `tuple`, `len`, `self.signature.parameters.items`, `functools.update_wrapper` to complete its work. State updates are written into `self.fn`, `original_fn`, `self.signature`, `self.param_names`, `self.num_args`, `self.kernel_cache`. The implementation relies on iteration.
**CN:** 该方法（属于 `CachedKernel`）实现了 `__init__`。 它主要通过调用 `isinstance`, `inspect.signature`, `tuple`, `len`, `self.signature.parameters.items`, `functools.update_wrapper` 来完成任务。 状态更新主要写入 `self.fn`, `original_fn`, `self.signature`, `self.param_names`, `self.num_args`, `self.kernel_cache`。 实现中使用了迭代逻辑。

### Lines 3878-3907: Method `CachedKernel.__getitem__` / 方法 `CachedKernel.__getitem__`
```python
    def __getitem__(self, grid):
        """
        Index with grid to get a launcher function.
        Returns a launcher that will handle caching based on the key function.
        """
        assert (
            isinstance(grid, tuple) and len(grid) <= 3
        ), "Grid must be a tuple with at most 3 dimensions."

        # Normalize grid once
        if len(grid) < 3:
            grid = grid + (1,) * (3 - len(grid))

        def launcher(*args, **kwargs):
            cache_key = self.key_fn(args, kwargs)

            cached_kernel = self.kernel_cache.get(cache_key)

            if cached_kernel is None:
                # First time: compile and cache the kernel
                cached_kernel = self.fn[grid](*args, **kwargs)
                self.kernel_cache[cache_key] = cached_kernel
                return cached_kernel
            else:
                # Use cached kernel
                all_args = self._build_args(args, kwargs)
                cached_kernel[grid](*all_args)
                return cached_kernel

        return launcher
```
**EN:** This method implements `__getitem__` on `CachedKernel`. It primarily calls `isinstance`, `len`, `self.key_fn`, `self.kernel_cache.get`, `self.fn`, `self._build_args` to complete its work. State updates are written into `grid`, `cache_key`, `cached_kernel`, `self.kernel_cache`, `all_args`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `CachedKernel`）实现了 `__getitem__`。 它主要通过调用 `isinstance`, `len`, `self.key_fn`, `self.kernel_cache.get`, `self.fn`, `self._build_args` 来完成任务。 状态更新主要写入 `grid`, `cache_key`, `cached_kernel`, `self.kernel_cache`, `all_args`。 实现中使用了条件分支。

### Lines 3909-3923: Method `CachedKernel._build_args` / 方法 `CachedKernel._build_args`
```python
    def _build_args(self, args, kwargs):
        """
        Build the complete argument list for kernel invocation.
        """
        complete_args = list(args)

        for i in range(len(args), self.num_args):
            name = self.param_names[i]
            value = kwargs.get(name, inspect.Parameter.empty)
            if value is not inspect.Parameter.empty:
                complete_args.append(value)
            else:
                raise ValueError(f"Missing argument: {name}")

        return complete_args
```
**EN:** This method implements `_build_args` on `CachedKernel`. It primarily calls `list`, `range`, `len`, `kwargs.get`, `complete_args.append`, `ValueError` to complete its work. State updates are written into `complete_args`, `name`, `value`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `CachedKernel`）实现了 `_build_args`。 它主要通过调用 `list`, `range`, `len`, `kwargs.get`, `complete_args.append`, `ValueError` 来完成任务。 状态更新主要写入 `complete_args`, `name`, `value`。 实现中使用了条件分支、迭代逻辑。

### Lines 3925-3929: Method `CachedKernel._clear_cache` / 方法 `CachedKernel._clear_cache`
```python
    def _clear_cache(self):
        """
        Clear the kernel cache for testing purposes.
        """
        self.kernel_cache.clear()
```
**EN:** This method implements `_clear_cache` on `CachedKernel`. It primarily calls `self.kernel_cache.clear` to complete its work.
**CN:** 该方法（属于 `CachedKernel`）实现了 `_clear_cache`。 它主要通过调用 `self.kernel_cache.clear` 来完成任务。

### Lines 3932-3979: Function `cached_triton_kernel` / 函数 `cached_triton_kernel`
```python
def cached_triton_kernel(key_fn=None):
    """
    Decorator that enables key-based caching for Triton kernels using a key function.

    It essentially bypasses Triton's built-in caching mechanism, allowing users to
    define their own caching strategy based on kernel parameters. This helps reduce
    the heavy overheads of Triton kernel launch when the kernel specialization dispatch
    is simple.

    Usage:
        @cached_triton_kernel(key_fn=lambda args, kwargs: kwargs.get('BLOCK_SIZE', 1024))
        @triton.jit
        def my_kernel(x_ptr, y_ptr, BLOCK_SIZE: tl.constexpr):
            ...

        # Invoke normally
        my_kernel[grid](x, y, BLOCK_SIZE=1024)

    Args:
        key_fn: A function that takes (args, kwargs) and returns the cache key(s).
                The key can be a single value or a tuple of values.

    Returns:
        A decorator that wraps the kernel with caching functionality.

    Note: Kernels with default parameter values are not supported and will raise an assertion error.
    """

# ... omitted for brevity ...
            logger.debug(
                f"{envs.SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE.name} = False. Using native triton kernel cache."
            )
            return fn

    return decorator
```
**EN:** This function implements `cached_triton_kernel`. It primarily calls `envs.SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE.get`, `is_cuda`, `logger.debug`, `CachedKernel`, `envs.SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE.is_set` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `cached_triton_kernel`。 它主要通过调用 `envs.SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE.get`, `is_cuda`, `logger.debug`, `CachedKernel`, `envs.SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE.is_set` 来完成任务。 实现中使用了条件分支。

### Lines 3982-4019: Function `reserve_rope_cache_for_long_sequences` / 函数 `reserve_rope_cache_for_long_sequences`
```python
def reserve_rope_cache_for_long_sequences(
    model, server_args, model_config, logger=None
):
    """Pre-expand RoPE cache for long sequences and speculative decoding."""
    from sglang.srt.environ import envs

    SAFETY_FACTOR = envs.SGLANG_SPEC_EXPANSION_SAFETY_FACTOR.get()
    MARGIN = envs.SGLANG_ROPE_CACHE_SAFETY_MARGIN.get()
    ALIGN = envs.SGLANG_ROPE_CACHE_ALIGN.get()

    # 1) Estimate base context upper bound
    base_ctx = (
        getattr(server_args, "context_length", None)
        or getattr(model_config, "context_len", None)
        or getattr(model_config, "max_model_len", None)
        or getattr(model_config.hf_text_config, "max_position_embeddings", None)
        or 2048
    )

    # 2) Speculative decoding expansion
    steps = int(getattr(server_args, "speculative_num_steps", 0) or 0)
    draft = int(getattr(server_args, "speculative_num_draft_tokens", 0) or 0)
    reserve = base_ctx + steps * draft * SAFETY_FACTOR + MARGIN

    # 3) Align to reduce reallocation frequency
    reserve = (reserve + ALIGN - 1) // ALIGN * ALIGN

    # Recursively expand all RoPE layers
# ... omitted for brevity ...
            ):
                child._ensure_cos_sin_cache_length(reserve - 1)
            else:
                reserve_rope_cache_recursive(child)

    reserve_rope_cache_recursive(model)
```
**EN:** This function implements `reserve_rope_cache_for_long_sequences`. It primarily calls `envs.SGLANG_SPEC_EXPANSION_SAFETY_FACTOR.get`, `envs.SGLANG_ROPE_CACHE_SAFETY_MARGIN.get`, `envs.SGLANG_ROPE_CACHE_ALIGN.get`, `int`, `reserve_rope_cache_recursive`, `getattr` to complete its work. State updates are written into `SAFETY_FACTOR`, `MARGIN`, `ALIGN`, `base_ctx`, `steps`, `draft`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `reserve_rope_cache_for_long_sequences`。 它主要通过调用 `envs.SGLANG_SPEC_EXPANSION_SAFETY_FACTOR.get`, `envs.SGLANG_ROPE_CACHE_SAFETY_MARGIN.get`, `envs.SGLANG_ROPE_CACHE_ALIGN.get`, `int`, `reserve_rope_cache_recursive`, `getattr` 来完成任务。 状态更新主要写入 `SAFETY_FACTOR`, `MARGIN`, `ALIGN`, `base_ctx`, `steps`, `draft`。 实现中使用了条件分支、迭代逻辑。

### Lines 4023-4027: Function `calc_diff` / 函数 `calc_diff`
```python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** This function implements `calc_diff`. It primarily calls `sum`, `x.double`, `y.double` to complete its work. State updates are written into `denominator`, `sim`.
**CN:** 该函数实现了 `calc_diff`。 它主要通过调用 `sum`, `x.double`, `y.double` 来完成任务。 状态更新主要写入 `denominator`, `sim`。

### Lines 4030-4041: Function `temp_attr_context` / 函数 `temp_attr_context`
```python
@contextmanager
def temp_attr_context(obj, attr, value):
    if obj is None:
        yield
        return

    original_value = getattr(obj, attr)
    setattr(obj, attr, value)
    try:
        yield
    finally:
        setattr(obj, attr, original_value)
```
**EN:** This function implements `temp_attr_context`. It primarily calls `getattr`, `setattr` to complete its work. State updates are written into `original_value`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `temp_attr_context`。 它主要通过调用 `getattr`, `setattr` 来完成任务。 状态更新主要写入 `original_value`。 实现中使用了条件分支、错误处理。

### Lines 4044-4044: Constants and shared state / 常量与共享状态
```python
cached_device_index = -1
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `cached_device_index`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `cached_device_index`。

### Lines 4047-4051: Function `get_current_device_stream_fast` / 函数 `get_current_device_stream_fast`
```python
def get_current_device_stream_fast():
    global cached_device_index
    if cached_device_index == -1:
        cached_device_index = torch.get_device_module().current_device()
    return torch.get_device_module().current_stream(cached_device_index)
```
**EN:** This function implements `get_current_device_stream_fast`. It primarily calls `torch.get_device_module.current_stream`, `torch.get_device_module.current_device`, `torch.get_device_module` to complete its work. State updates are written into `cached_device_index`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_current_device_stream_fast`。 它主要通过调用 `torch.get_device_module.current_stream`, `torch.get_device_module.current_device`, `torch.get_device_module` 来完成任务。 状态更新主要写入 `cached_device_index`。 实现中使用了条件分支。

### Lines 4054-4061: Function `raise_error_or_warn` / 函数 `raise_error_or_warn`
```python
def raise_error_or_warn(obj, strict, counter_name, message, log_interval=1000):
    if strict:
        raise ValueError(message)
    else:
        count = getattr(obj, counter_name, 0)
        if count % log_interval == 0:
            logger.warning(message)
        setattr(obj, counter_name, count + 1)
```
**EN:** This function implements `raise_error_or_warn`. It primarily calls `ValueError`, `getattr`, `setattr`, `logger.warning` to complete its work. State updates are written into `count`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `raise_error_or_warn`。 它主要通过调用 `ValueError`, `getattr`, `setattr`, `logger.warning` 来完成任务。 状态更新主要写入 `count`。 实现中使用了条件分支。

### Lines 4064-4071: Function `get_or_create_event_loop` / 函数 `get_or_create_event_loop`
```python
def get_or_create_event_loop():
    """Gets the running event loop or creates a new one if it doesn't exist."""
    try:
        return asyncio.get_running_loop()
    except RuntimeError:
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        return loop
```
**EN:** This function implements `get_or_create_event_loop`. It primarily calls `asyncio.get_running_loop`, `asyncio.new_event_loop`, `asyncio.set_event_loop` to complete its work. State updates are written into `loop`. The implementation relies on error handling.
**CN:** 该函数实现了 `get_or_create_event_loop`。 它主要通过调用 `asyncio.get_running_loop`, `asyncio.new_event_loop`, `asyncio.set_event_loop` 来完成任务。 状态更新主要写入 `loop`。 实现中使用了错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `DynamicGradMode`, `TimeInfo`, `LayerFn`, `ImageData`, `VideoData`, `RefCountedGauge`, `MultiprocessingSerializer`, `SafeUnpickler`
- **Functions / 函数**: `is_hip`, `is_cuda`, `is_cuda_alike`, `is_hpu`, `is_xpu`, `is_npu`, `is_host_cpu_x86`, `is_host_cpu_arm64`
- **Async functions / 异步函数**: `track_http_status_code`, `ping`, `health`, `health_generate`, `increment`, `decrement`, `wait_for`, `wait_for_zero`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.observability.func_timer`, `sglang.srt.utils.video_decoder`, `sglang.srt.server_args`, `sglang.srt.distributed`, `sglang.srt.layers.utils`, `sglang.srt.utils.offloader`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`, `sglang.srt.distributed.device_communicators.custom_all_reduce`, `sglang.srt.distributed.parallel_state`, `sglang.srt.utils.http_middleware_patch`, `sglang.srt.platforms`
- **External / 外部依赖**: `numpy`, `orjson`, `psutil`, `pybase64`, `requests`, `torch`, `torch.distributed`, `triton`, `packaging`, `PIL`, `starlette.routing`, `torch.library`
- **Standard library / 标准库**: `__future__`, `argparse`, `asyncio`, `builtins`, `ctypes`, `functools`, `gc`, `importlib`, `inspect`, `io`, `itertools`, `json`
