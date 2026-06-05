# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fla/ops/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fast linear attention custom kernels and utility ops / 快速线性注意力自定义内核与工具算子

## Line-by-Line Analysis / 逐行分析
### Lines 10-31 — imports and setup
```python
import contextlib
import functools
import logging
import os
from collections.abc import Callable
from enum import Enum
from typing import Any, Literal

import torch

from vllm.platforms import current_platform
from vllm.triton_utils import triton

logger = logging.getLogger(__name__)

COMPILER_MODE = os.getenv("FLA_COMPILER_MODE") == "1"
FLA_CI_ENV = os.getenv("FLA_CI_ENV") == "1"

SUPPRESS_LEVEL = int(os.getenv("GDN_RECOMPUTE_SUPPRESS_LEVEL", "0"))

# Default chunk size used across FLA triton kernels (kda, chunk, chunk_o, etc.)
FLA_CHUNK_SIZE = 64
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `logging.getLogger`, `int`, `os.getenv`. It writes or updates `logger`, `COMPILER_MODE`, `FLA_CI_ENV`, `SUPPRESS_LEVEL`, `FLA_CHUNK_SIZE`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `logging.getLogger`, `int`, `os.getenv`。 它会写入或更新 `logger`, `COMPILER_MODE`, `FLA_CI_ENV`, `SUPPRESS_LEVEL`, `FLA_CHUNK_SIZE`。

### Lines 34-80 — function `tensor_cache`
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

    cache_entries: tuple[tuple | None, dict | None, Any] = []
    cache_size = 8

    @functools.wraps(fn)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        nonlocal cache_entries, cache_size
        for i, entry in enumerate(cache_entries):
            last_args, last_kwargs, last_result = entry
            if (
                len(args) == len(last_args)
                and len(kwargs) == len(last_kwargs)
                and all(a is b for a, b in zip(args, last_args))
                and all(
                    k in last_kwargs and v is last_kwargs[k] for k, v in kwargs.items()
                )
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
**EN:** This function defines `tensor_cache`. A decorator that caches the most recent results of a function with tensor inputs. The main inputs are `fn`. Key calls include `functools.wraps`, `enumerate`, `fn`, `cache_entries.append`, `len`, `all`. It writes or updates `cache_entries`, `cache_size`, `result`, `last_args`, `last_kwargs`, `last_result`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `tensor_cache`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `fn`。 关键调用包括 `functools.wraps`, `enumerate`, `fn`, `cache_entries.append`, `len`, `all`。 它会写入或更新 `cache_entries`, `cache_size`, `result`, `last_args`, `last_kwargs`, `last_result`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 83-117 — function `input_guard`
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
            ctx = torch.accelerator.device_index(tensor.device.index)
        else:
            ctx = contextlib.nullcontext()

        with ctx:
            return fn(*contiguous_args, **contiguous_kwargs)

    return wrapper
```
**EN:** This function defines `input_guard`. A decorator to make sure all input tensors are contiguous and set the device based on input tensors. The main inputs are `fn`. Key calls include `functools.wraps`, `isinstance`, `kwargs.values`, `torch.accelerator.device_index`, `contextlib.nullcontext`, `fn`. It writes or updates `contiguous_args`, `contiguous_kwargs`, `tensor`, `ctx`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `input_guard`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `fn`。 关键调用包括 `functools.wraps`, `isinstance`, `kwargs.values`, `torch.accelerator.device_index`, `contextlib.nullcontext`, `fn`。 它会写入或更新 `contiguous_args`, `contiguous_kwargs`, `tensor`, `ctx`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 120-125 — function `get_available_device`
```python
@functools.cache
def get_available_device() -> str:
    try:
        return triton.runtime.driver.active.get_current_target().backend
    except (RuntimeError, AttributeError):
        return "cpu"
```
**EN:** This function defines `get_available_device`. It derives and returns computed metadata or outputs needed by later stages. Key calls include `triton.runtime.driver.active.get_current_target`.
**CN:** 该函数定义 `get_available_device`。 它推导并返回后续阶段所需的元数据或输出。 关键调用包括 `triton.runtime.driver.active.get_current_target`。

### Lines 128-137 — function `_check_platform`
```python
@functools.cache
def _check_platform() -> Literal["nvidia", "amd", "intel", "musa"]:
    device = get_available_device()
    mapping = {
        "cuda": "nvidia",
        "hip": "amd",
        "xpu": "intel",
    }
    # return the mapped value, or the original if not found
    return mapping.get(device, device)
```
**EN:** This function defines `_check_platform`. It provides one of the file's main runtime building blocks. Key calls include `get_available_device`, `mapping.get`. It writes or updates `device`, `mapping`.
**CN:** 该函数定义 `_check_platform`。 它是该文件中的一个主要运行时构件。 关键调用包括 `get_available_device`, `mapping.get`。 它会写入或更新 `device`, `mapping`。

### Lines 167-176 — function `get_all_max_shared_mem`
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
        return [-1]
```
**EN:** This function defines `get_all_max_shared_mem`. It derives and returns computed metadata or outputs needed by later stages. Key calls include `triton.runtime.driver.active.utils.get_device_properties`, `range`, `device_torch_lib.device_count`.
**CN:** 该函数定义 `get_all_max_shared_mem`。 它推导并返回后续阶段所需的元数据或输出。 关键调用包括 `triton.runtime.driver.active.utils.get_device_properties`, `range`, `device_torch_lib.device_count`。

### Lines 179-190 — class `Backend`
```python
class Backend(Enum):
    ADA = 101376  # RTX 4090
    AMPERE = 166912  # A100
    HOPPER = 232448  # H100
    DEFAULT = 102400  # Default

    @classmethod
    def get_shared_memory(cls, arch: str) -> int:
        try:
            return cls[arch.upper()].value
        except KeyError:
            return cls.DEFAULT.value
```
**EN:** This class defines `Backend`. It inherits from `Enum`. It provides one of the file's main runtime building blocks. Important methods include `get_shared_memory`. Key calls include `arch.upper`. It writes or updates `ADA`, `AMPERE`, `HOPPER`, `DEFAULT`.
**CN:** 该类定义了 `Backend`。 它继承自 `Enum`。 它是该文件中的一个主要运行时构件。 重要方法包括 `get_shared_memory`。 关键调用包括 `arch.upper`。 它会写入或更新 `ADA`, `AMPERE`, `HOPPER`, `DEFAULT`。

### Lines 185-190 — method `Backend.get_shared_memory`
```python
    @classmethod
    def get_shared_memory(cls, arch: str) -> int:
        try:
            return cls[arch.upper()].value
        except KeyError:
            return cls.DEFAULT.value
```
**EN:** This method defines `get_shared_memory`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `arch`. Key calls include `arch.upper`.
**CN:** 该方法定义 `get_shared_memory`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `arch`。 关键调用包括 `arch.upper`。

### Lines 193-200 — function `check_shared_mem`
```python
@functools.cache
def check_shared_mem(arch: str = "none", tensor_idx: int = 0) -> bool:
    try:
        device_shared_mem_list = get_all_max_shared_mem()
        max_shared_memory = device_shared_mem_list[tensor_idx]
        return max_shared_memory >= Backend.get_shared_memory(arch)
    except Exception:
        return False
```
**EN:** This function defines `check_shared_mem`. It provides one of the file's main runtime building blocks. The main inputs are `arch`, `tensor_idx`. Key calls include `get_all_max_shared_mem`, `Backend.get_shared_memory`. It writes or updates `device_shared_mem_list`, `max_shared_memory`.
**CN:** 该函数定义 `check_shared_mem`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `arch`, `tensor_idx`。 关键调用包括 `get_all_max_shared_mem`, `Backend.get_shared_memory`。 它会写入或更新 `device_shared_mem_list`, `max_shared_memory`。

## Key Concepts / 关键概念
- [EN] Fast linear attention custom kernels and utility ops / [CN] 快速线性注意力自定义内核与工具算子
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `tensor_cache`, `input_guard`, `get_available_device`, `_check_platform` / [CN] 核心符号：`tensor_cache`, `input_guard`, `get_available_device`, `_check_platform`

## Dependencies / 依赖关系
- **External**: `contextlib`, `functools`, `logging`, `os`, `collections.abc`, `enum`, `typing`, `torch` / **外部依赖**: `contextlib`, `functools`, `logging`, `os`, `collections.abc`, `enum`, `typing`, `torch`
- **Internal**: `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels / **运行时特征**: platform-aware dispatch, Triton kernels
