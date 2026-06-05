# utils.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/fla/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides utils utilities shared by multiple attention implementations in SGLang. / 该模块提供 SGLang 多种注意力实现共享的 utils 工具函数。
## Line-by-Line Analysis / 逐行分析
### Lines 4-18: imports
```python
import contextlib
import functools
import inspect
import logging
import os
import sys
from enum import Enum
from functools import lru_cache
from typing import Any, Callable, Dict, Literal, Optional, Tuple

import torch
import triton
from packaging import version

from sglang.srt.utils.common import torch_release
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 20-33: module constants
```python
logger = logging.getLogger(__name__)

COMPILER_MODE = os.getenv("FLA_COMPILER_MODE") == "1"
FLA_CI_ENV = os.getenv("FLA_CI_ENV") == "1"
FLA_CACHE_RESULTS = os.getenv("FLA_CACHE_RESULTS", "1") == "1"


SUPPORTS_AUTOTUNE_CACHE = (
    "cache_results" in inspect.signature(triton.autotune).parameters
)

autotune_cache_kwargs = (
    {"cache_results": FLA_CACHE_RESULTS} if SUPPORTS_AUTOTUNE_CACHE else {}
)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 36-71: function check_environments
```python
@lru_cache(maxsize=1)
def check_environments():
    """
    Checks the current operating system, Triton version, and Python version,
    issuing warnings if they don't meet recommendations.
    This function's body only runs once due to lru_cache.
    """
    # Check Operating System
    if sys.platform == "win32":
        logger.warning(
            "Detected Windows operating system. Triton does not have an official Windows release, "
            "thus FLA will not be adapted for Windows, and any potential errors will not be fixed. "
            "Please consider using a Linux environment for compatibility."
        )

    triton_version = version.parse(triton.__version__)
    required_triton_version = version.parse("3.2.0")

    if triton_version < required_triton_version:
        logger.warning(
            f"Current Triton version {triton_version} is below the recommended 3.2.0 version. "
            "Errors may occur and these issues will not be fixed. "
            "Please consider upgrading Triton."
        )

    # Check Python version
    py_version = version.parse(f"{sys.version_info.major}.{sys.version_info.minor}")
    required_py_version = version.parse("3.11")

    if py_version < required_py_version:
        logger.warning(
            f"Current Python version {py_version} is below the recommended 3.11 version. "
            "It is recommended to upgrade to Python 3.11 or higher for the best experience."
        )

    return None
```
**EN:** Implements the check environments routine used by this attention module.
**CN:** 实现该注意力模块使用的 check environments 例程。

### Lines 74-75: function get_abs_err
```python
def get_abs_err(x, y):
    return (x.detach() - y.detach()).flatten().abs().max().item()
```
**EN:** Computes and returns get abs err from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get abs err。

### Lines 78-81: function get_err_ratio
```python
def get_err_ratio(x, y):
    err = (x.detach() - y.detach()).flatten().square().mean().sqrt().item()
    base = (x.detach()).flatten().square().mean().sqrt().item()
    return err / (base + 1e-8)
```
**EN:** Computes and returns get err ratio from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get err ratio。

### Lines 84-97: function assert_close
```python
def assert_close(prefix, ref, tri, ratio, warning=False, err_atol=1e-6):
    abs_atol = get_abs_err(ref, tri)
    msg = f"{prefix} diff: {abs_atol:.6f} ratio: {get_err_ratio(ref, tri):.6f}"
    logger.info(msg)
    error_rate = get_err_ratio(ref, tri)
    if abs_atol <= err_atol:
        return
    if warning or (FLA_CI_ENV and (error_rate < 0.01 or abs_atol <= 0.3)):
        if error_rate > ratio:
            import warnings

            warnings.warn(msg)
    else:
        assert error_rate < ratio, msg
```
**EN:** Implements the assert close routine used by this attention module.
**CN:** 实现该注意力模块使用的 assert close 例程。

### Lines 100-100: module constants
```python
SUPPRESS_LEVEL = int(os.getenv("GDN_RECOMPUTE_SUPPRESS_LEVEL", "0"))
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 103-142: function tensor_cache
```python
def tensor_cache(fn: Callable[..., torch.Tensor]) -> Callable[..., torch.Tensor]:
    """
    A decorator that caches the most recent results of a function with tensor inputs.
    This decorator will store the output of the decorated function for the most recent set of input tensors.
    The cache is limited to a fixed size (default is 4). When the cache is full, the oldest entry will be removed.
    Args:
        fn (Callable[..., torch.Tensor]):
            The function to be decorated. It should take tensor inputs and return tensor outputs.
    Returns:
        Callable[..., torch.Tensor]:
            A wrapped version of the input function with single-entry caching.
    """

    cache_entries: Tuple[Optional[Tuple], Optional[Dict], Any] = []
    cache_size = 4

    @functools.wraps(fn)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        nonlocal cache_entries, cache_size
        for i, entry in enumerate(cache_entries):
            last_args, last_kwargs, last_result = entry
            if len(args) == len(last_args) and len(kwargs) == len(last_kwargs):
                if all(a is b for a, b in zip(args, last_args)) and all(
                    k in last_kwargs and v is last_kwargs[k] for k, v in kwargs.items()
                ):
                    cache_entries = (
                        cache_entries[:i]
                        + cache_entries[i + 1 :]
                        + [(args, kwargs, last_result)]
                    )
                    return last_result

        result = fn(*args, **kwargs)

        if len(cache_entries) >= cache_size:
            cache_entries = cache_entries[1:]
        cache_entries.append((args, kwargs, result))
        return result

    return wrapper
```
**EN:** Implements the tensor cache routine used by this attention module.
**CN:** 实现该注意力模块使用的 tensor cache 例程。

### Lines 145-179: function input_guard
```python
def input_guard(fn: Callable[..., torch.Tensor]) -> Callable[..., torch.Tensor]:
    """
    A decorator to make sure all input tensors are contiguous and set the device based on input tensors.
    """

    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        contiguous_args = (
            i if not isinstance(i, torch.Tensor) else i.contiguous() for i in args
        )
        contiguous_kwargs = {
            k: (v if not isinstance(v, torch.Tensor) else v.contiguous())
            for k, v in kwargs.items()
        }

        tensor = None
        for arg in args:
            if isinstance(arg, torch.Tensor):
                tensor = arg
                break
        if tensor is None:
            for value in kwargs.values():
                if isinstance(value, torch.Tensor):
                    tensor = value
                    break

        if tensor is not None:
            ctx = custom_device_ctx(tensor.device.index)
        else:
            ctx = contextlib.nullcontext()

        with ctx:
            return fn(*contiguous_args, **contiguous_kwargs)

    return wrapper
```
**EN:** Implements the input guard routine used by this attention module.
**CN:** 实现该注意力模块使用的 input guard 例程。

### Lines 182-182: module constants
```python
contiguous = input_guard
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 185-210: function require_version
```python
def require_version(version, hint):
    """
    Perform a runtime check of the dependency versions, using the exact same syntax used by pip.
    """

    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(ctx, *args, **kwargs):
            from transformers.utils.versions import require_version

            require_version(version, hint)
            return fn(
                ctx,
                *(
                    i if not isinstance(i, torch.Tensor) else i.contiguous()
                    for i in args
                ),
                **{
                    k: (v if not isinstance(v, torch.Tensor) else v.contiguous())
                    for k, v in kwargs.items()
                },
            )

        return wrapper

    return decorator
```
**EN:** Implements the require version routine used by this attention module.
**CN:** 实现该注意力模块使用的 require version 例程。

### Lines 213-217: function checkpoint
```python
def checkpoint(fn):
    def wrapper(*args, **kwargs):
        return torch.utils.checkpoint.checkpoint(fn, *args, **kwargs)

    return wrapper
```
**EN:** Implements the checkpoint routine used by this attention module.
**CN:** 实现该注意力模块使用的 checkpoint 例程。

### Lines 220-225: function _cpu_device_warning
```python
def _cpu_device_warning():
    import warnings

    warnings.warn(
        ("Triton is not supported on current platform, roll back to CPU."), stacklevel=1
    )
```
**EN:** Implements the cpu device warning routine used by this attention module.
**CN:** 实现该注意力模块使用的 cpu device warning 例程。

### Lines 228-236: function get_multiprocessor_count
```python
@lru_cache(maxsize=None)
def get_multiprocessor_count(tensor_idx: int = 0) -> int:
    try:
        return triton.runtime.driver.active.utils.get_device_properties(tensor_idx)[
            "multiprocessor_count"
        ]
    except BaseException:
        _cpu_device_warning()
        return -1
```
**EN:** Computes and returns get multiprocessor count from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get multiprocessor count。

### Lines 239-245: function get_available_device
```python
@lru_cache(maxsize=None)
def get_available_device() -> str:
    try:
        return triton.runtime.driver.active.get_current_target().backend
    except BaseException:
        _cpu_device_warning()
        return "cpu"
```
**EN:** Computes and returns get available device from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get available device。

### Lines 248-258: function _check_platform
```python
@lru_cache(maxsize=None)
def _check_platform() -> Literal["nvidia", "amd", "intel", "musa"]:
    device = get_available_device()
    if device == "cuda":
        return "nvidia"
    elif device == "hip":
        return "amd"
    elif device == "xpu":
        return "intel"
    else:
        return device
```
**EN:** Implements the check platform routine used by this attention module.
**CN:** 实现该注意力模块使用的 check platform 例程。

### Lines 264-280: module constants
```python
device = get_available_device() if get_available_device() != "hip" else "cuda"
device_torch_lib = getattr(torch, device)
device_platform = _check_platform()

is_amd = device_platform == "amd"
is_intel = device_platform == "intel"
is_nvidia = device_platform == "nvidia"
is_intel_alchemist = is_intel and "Intel(R) Arc(TM) A" in torch.xpu.get_device_name(0)
is_nvidia_hopper = is_nvidia and (
    "NVIDIA H" in torch.cuda.get_device_name(0)
    or torch.cuda.get_device_capability()[0] >= 9
)
use_cuda_graph = is_nvidia and os.environ.get("FLA_USE_CUDA_GRAPH", "0") == "1"

# Nvidia Ampere or newer, haven't check AMD and intel yet.
is_tf32_supported = is_nvidia and torch.cuda.get_device_capability(0)[0] >= 8
is_gather_supported = hasattr(triton.language, "gather")
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 283-293: function get_all_max_shared_mem
```python
def get_all_max_shared_mem():
    try:
        return [
            triton.runtime.driver.active.utils.get_device_properties(i)[
                "max_shared_mem"
            ]
            for i in range(device_torch_lib.device_count())
        ]
    except BaseException:
        _cpu_device_warning()
        return [-1]
```
**EN:** Computes and returns get all max shared mem from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get all max shared mem。

### Lines 296-301: class Backend
```python
class Backend(Enum):
    ADA = 101376  # RTX 4090
    AMPERE = 166912  # A100
    HOPPER = 232448  # H100
    DEFAULT = 102400  # Default
```
**EN:** Enumeration that names execution modes, backend choices, or other symbolic options used by this module.
**CN:** 该枚举为本模块使用的执行模式、后端选择或其他符号化选项命名。

### Lines 302-307: method Backend.get_shared_memory
```python
    @classmethod
    def get_shared_memory(cls, arch: str) -> int:
        try:
            return cls[arch.upper()].value
        except KeyError:
            return cls.DEFAULT.value
```
**EN:** Computes and returns get shared memory from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get shared memory。

### Lines 310-317: function check_shared_mem
```python
@lru_cache(maxsize=None)
def check_shared_mem(arch: str = "none", tensor_idx: int = 0) -> bool:
    try:
        device_shared_mem_list = get_all_max_shared_mem()
        max_shared_memory = device_shared_mem_list[tensor_idx]
        return max_shared_memory >= Backend.get_shared_memory(arch)
    except Exception:
        return False
```
**EN:** Implements the check shared mem routine used by this attention module.
**CN:** 实现该注意力模块使用的 check shared mem 例程。

### Lines 320-336: conditional branch
```python
if torch_release >= (2, 4):
    device = "cuda" if device == "cpu" else device
    autocast_custom_fwd = functools.partial(torch.amp.custom_fwd, device_type=device)
    autocast_custom_bwd = functools.partial(torch.amp.custom_bwd, device_type=device)

    def custom_device_ctx(index: int):
        return device_torch_lib.device(index)

else:
    assert (
        device == "cuda"
    ), "Only cuda device is supported for PyTorch version < 2.4.0."
    autocast_custom_fwd = device_torch_lib.amp.custom_fwd
    autocast_custom_bwd = device_torch_lib.amp.custom_bwd

    def custom_device_ctx(index: int):
        return torch.cuda.device(index)
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 339-339: module constants
```python
device_platform = get_available_device()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `contextlib`
- `functools`
- `inspect`
- `logging`
- `os`
- `sys`
- `enum.Enum`
- `functools.lru_cache`
- `typing.Any`
- `typing.Callable`
- `typing.Dict`
- `typing.Literal`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `triton`
- `packaging.version`
- `sglang.srt.utils.common.torch_release`
- `warnings`
- `transformers.utils.versions.require_version`
