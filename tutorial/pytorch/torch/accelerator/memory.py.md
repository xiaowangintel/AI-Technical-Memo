# memory.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/accelerator/memory.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides accelerator-facing Python APIs for graph capture, memory management, and runtime coordination.
- **Purpose (CN)**: 提供面向 accelerator 的 Python API，用于图捕获、内存管理以及运行时协同。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
from collections import OrderedDict
from typing import Any

import torch

from ._utils import _device_t, _get_device_index


__all__ = [
    "empty_cache",
    "empty_host_cache",
    "get_memory_info",
    "max_memory_allocated",
    "max_memory_reserved",
    "memory_allocated",
    "memory_reserved",
    "memory_stats",
    "reset_accumulated_memory_stats",
    "reset_peak_memory_stats",
]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, ._utils; standard-library helpers such as collections, typing. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、._utils；标准库辅助模块，如 collections、typing。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 23-37 / 第 23-37 行
````python
def empty_cache() -> None:
    r"""Release all unoccupied cached memory currently held by the caching
    allocator so that those can be used in other application.

    .. note:: This function is a no-op if the memory allocator for the current
        :ref:`accelerator <accelerators>` has not been initialized.
    """
    if not torch._C._accelerator_isAllocatorInitialized():
        return
    torch._C._accelerator_emptyCache()


def empty_host_cache() -> None:
    r"""Release all unoccupied cached host (pinned) memory currently held by the host caching
    allocator so that it can be used by other applications.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `empty_host_cache`, which drops cached state so a later execution phase can rebuild it cleanly. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `empty_host_cache`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 39-51 / 第 39-51 行
````python
    .. note:: This function is a no-op if the memory allocator for the current
        :ref:`accelerator <accelerators>` has not been initialized.
    """
    torch._C._accelerator_emptyHostCache()


def memory_stats(device_index: _device_t = None, /) -> OrderedDict[str, Any]:
    r"""Return a dictionary of accelerator device memory allocator statistics for a given device index.

    The return value of this function is a dictionary of statistics, each of
    which is a non-negative integer.

    Core statistics:
````
- **EN**: This chunk defines `memory_stats`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `memory_stats`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 53-70 / 第 53-70 行
````python
    - ``"allocated.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      number of allocation requests received by the memory allocator.
    - ``"allocated_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of allocated memory.
    - ``"segment.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      number of reserved segments from device memory allocation.
    - ``"reserved_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of reserved memory.
    - ``"active.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      number of active memory blocks.
    - ``"active_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of active memory.
    - ``"inactive_split.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      number of inactive, non-releasable memory blocks.
    - ``"inactive_split_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of inactive, non-releasable memory.

    For these core statistics, values are broken down as follows.
````
- **EN**: This chunk continues `memory_stats` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `memory_stats`，进一步展开其内部控制流或状态更新。

### Lines 72-88 / 第 72-88 行
````python
    Pool type:

    - ``all``: combined statistics across all memory pools.
    - ``large_pool``: statistics for the large allocation pool
      (as of June 2025, for size >= 1MB allocations).
    - ``small_pool``: statistics for the small allocation pool
      (as of June 2025, for size < 1MB allocations).

    Metric type:

    - ``current``: current value of this metric.
    - ``peak``: maximum value of this metric.
    - ``allocated``: historical total increase in this metric.
    - ``freed``: historical total decrease in this metric.

    In addition to the core statistics, we also provide some simple event
    counters:
````
- **EN**: This chunk continues `memory_stats` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `memory_stats`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 90-109 / 第 90-109 行
````python
    - ``"num_alloc_retries"``: number of failed device memory allocation calls that
      result in a cache flush and retry.
    - ``"num_ooms"``: number of out-of-memory errors thrown.
    - ``"num_sync_all_streams"``: number of ``synchronize_and_free_events`` calls.
    - ``"num_device_alloc"``: number of device memory allocation calls.
    - ``"num_device_free"``: number of device memory free calls.

    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        OrderedDict[str, Any]: an ordered dictionary mapping statistic names to their values.
    """
    if not torch._C._accelerator_isAllocatorInitialized():
        return OrderedDict()
    device_index = _get_device_index(device_index, optional=True)
    stats = torch._C._accelerator_getDeviceStats(device_index)
````
- **EN**: This chunk continues `memory_stats` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `memory_stats`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 110-128 / 第 110-128 行
````python
    flat_stats = []

    def flatten(prefix: str, value: Any) -> None:
        if isinstance(value, dict):
            for k, v in value.items():
                nested_prefix = f"{prefix}.{k}" if prefix else k
                flatten(nested_prefix, v)
        else:
            flat_stats.append((prefix, value))

    flatten("", stats)
    flat_stats.sort()
    # pyrefly: ignore [no-matching-overload]
    return OrderedDict(flat_stats)


def memory_allocated(device_index: _device_t = None, /) -> int:
    r"""Return the current :ref:`accelerator<accelerators>` device memory occupied by tensors
    in bytes for a given device index.
````
- **EN**: This chunk defines `memory_allocated`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `memory_allocated`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 130-148 / 第 130-148 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        int: the current memory occupied by live tensors (in bytes) within the current process.
    """
    return memory_stats(device_index).get("allocated_bytes.all.current", 0)


def max_memory_allocated(device_index: _device_t = None, /) -> int:
    r"""Return the current :ref:`accelerator<accelerators>` maximum device memory occupied by tensors
    in bytes for a given device index.

    By default, this returns the peak allocated memory since the beginning of
    this program. :func:`~torch.accelerator.reset_peak_memory_stats` can be used to
    reset the starting point in tracking this metric.
````
- **EN**: This chunk defines `max_memory_allocated`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `max_memory_allocated`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 150-164 / 第 150-164 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        int: the peak memory occupied by live tensors (in bytes) within the current process.
    """
    return memory_stats(device_index).get("allocated_bytes.all.peak", 0)


def memory_reserved(device_index: _device_t = None, /) -> int:
    r"""Return the current :ref:`accelerator<accelerators>` device memory managed by the caching allocator
    in bytes for a given device index.
````
- **EN**: This chunk defines `memory_reserved`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `memory_reserved`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 166-184 / 第 166-184 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        int: the current memory reserved by PyTorch (in bytes) within the current process.
    """
    return memory_stats(device_index).get("reserved_bytes.all.current", 0)


def max_memory_reserved(device_index: _device_t = None, /) -> int:
    r"""Return the current :ref:`accelerator<accelerators>` maximum device memory managed by the caching allocator
    in bytes for a given device index.

    By default, this returns the peak cached memory since the beginning of this
    program. :func:`~torch.accelerator.reset_peak_memory_stats` can be used to reset
    the starting point in tracking this metric.
````
- **EN**: This chunk defines `max_memory_reserved`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `max_memory_reserved`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 186-200 / 第 186-200 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        int: the peak memory reserved by PyTorch (in bytes) within the current process.
    """
    return memory_stats(device_index).get("reserved_bytes.all.peak", 0)


def reset_accumulated_memory_stats(device_index: _device_t = None, /) -> None:
    r"""Reset the "accumulated" (historical) stats tracked by the current :ref:`accelerator<accelerators>`
    memory allocator for a given device index.
````
- **EN**: This chunk defines `reset_accumulated_memory_stats`, which drops cached state so a later execution phase can rebuild it cleanly. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `reset_accumulated_memory_stats`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 202-217 / 第 202-217 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    .. note:: This function is a no-op if the memory allocator for the current
        :ref:`accelerator <accelerators>` has not been initialized.
    """
    device_index = _get_device_index(device_index, optional=True)
    return torch._C._accelerator_resetAccumulatedStats(device_index)


def reset_peak_memory_stats(device_index: _device_t = None, /) -> None:
    r"""Reset the "peak" stats tracked by the current :ref:`accelerator<accelerators>`
    memory allocator for a given device index.
````
- **EN**: This chunk defines `reset_peak_memory_stats`, which drops cached state so a later execution phase can rebuild it cleanly. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `reset_peak_memory_stats`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 219-233 / 第 219-233 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    .. note:: This function is a no-op if the memory allocator for the current
        :ref:`accelerator <accelerators>` has not been initialized.
    """
    device_index = _get_device_index(device_index, optional=True)
    return torch._C._accelerator_resetPeakStats(device_index)


def get_memory_info(device_index: _device_t = None, /) -> tuple[int, int]:
    r"""Return the current device memory information for a given device index.
````
- **EN**: This chunk defines `get_memory_info`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `get_memory_info`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 235-248 / 第 235-248 行
````python
    Args:
        device_index (:class:`torch.device`, str, int, optional): the index of the device to target.
            If not given, use :func:`torch.accelerator.current_device_index` by default.
            If a :class:`torch.device` or str is provided, its type must match the current
            :ref:`accelerator<accelerators>` device type.

    Returns:
        tuple[int, int]: a tuple of two integers (free_memory, total_memory) in bytes.
            The first value is the free memory on the device (available across all processes and applications),
            The second value is the device's total hardware memory capacity.
    """
    device_index = _get_device_index(device_index, optional=True)
    # pyrefly: ignore [missing-attribute]
    return torch._C._accelerator_getMemoryInfo(device_index)
````
- **EN**: This chunk continues `get_memory_info` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_memory_info`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Accelerator runtime**
  - EN: Coordinates device-specific graph capture, cache management, and runtime synchronization.
  - CN: 协调设备相关的图捕获、缓存管理与运行时同步。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **empty_cache**
  - EN: `empty_cache` is one of the main symbols declared or implemented in this file.
  - CN: `empty_cache` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `._utils`
- **Standard library / 标准库**: `collections`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `empty_cache`, `empty_host_cache`, `memory_stats`, `memory_allocated`, `max_memory_allocated`, `memory_reserved`, `max_memory_reserved`, `reset_accumulated_memory_stats`, `reset_peak_memory_stats`
