# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/accelerator/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
r"""
This package introduces support for the current :ref:`accelerator<accelerators>` in python.
"""

from functools import cache
from typing import Any
from typing_extensions import deprecated

import torch

from ._utils import _device_t, _get_device_index
from .graphs import Graph
from .memory import (
    empty_cache,
    empty_host_cache,
    get_memory_info,
    max_memory_allocated,
    max_memory_reserved,
    memory_allocated,
    memory_reserved,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, ._utils, .graphs, ...; standard-library helpers such as functools, typing; other helper packages such as typing_extensions. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、._utils、.graphs、...；标准库辅助模块，如 functools、typing；其他辅助包，如 typing_extensions。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 21-40 / 第 21-40 行
````python
    memory_stats,
    reset_accumulated_memory_stats,
    reset_peak_memory_stats,
)


__all__ = [
    "Graph",
    "current_accelerator",
    "current_device_idx",  # deprecated
    "current_device_index",
    "get_device_capability",
    "current_stream",
    "device_count",
    "device_index",
    "empty_cache",
    "empty_host_cache",
    "get_memory_info",
    "is_available",
    "max_memory_allocated",
````
- **EN**: `__all__` defines the public symbols that this module chooses to export.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。

### Lines 41-59 / 第 41-59 行
````python
    "max_memory_reserved",
    "memory_allocated",
    "memory_reserved",
    "memory_stats",
    "reset_accumulated_memory_stats",
    "reset_peak_memory_stats",
    "set_device_idx",  # deprecated
    "set_device_index",
    "set_stream",
    "synchronize",
]


def device_count() -> int:
    r"""Return the number of current :ref:`accelerator<accelerators>` available.

    Returns:
        int: the number of the current :ref:`accelerator<accelerators>` available.
            If there is no available accelerators, return 0.
````
- **EN**: This chunk defines `device_count`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `device_count`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 61-79 / 第 61-79 行
````python
    .. note:: This API delegates to the device-specific version of `device_count`.
        On CUDA, this API will NOT poison fork if NVML discovery succeeds.
        Otherwise, it will. For more details, see :ref:`multiprocessing-poison-fork-note`.
    """
    acc = current_accelerator()
    if acc is None:
        return 0

    mod = torch.get_device_module(acc)
    return mod.device_count()


def is_available() -> bool:
    r"""Check if the current accelerator is available at runtime: it was build, all the
    required drivers are available and at least one device is visible.
    See :ref:`accelerator<accelerators>` for details.

    Returns:
        bool: A boolean indicating if there is an available :ref:`accelerator<accelerators>`.
````
- **EN**: This chunk defines `is_available`, which checks a capability or invariant before later code relies on it. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `is_available`，其作用是检查某项能力或不变量，供后续逻辑依赖。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 81-99 / 第 81-99 行
````python
    .. note:: This API delegates to the device-specific version of `is_available`.
        On CUDA, when the environment variable ``PYTORCH_NVML_BASED_CUDA_CHECK=1`` is set,
        this function will NOT poison fork. Otherwise, it will. For more details, see
        :ref:`multiprocessing-poison-fork-note`.

    Example::

        >>> assert torch.accelerator.is_available() "No available accelerators detected."
    """
    # Why not just check "device_count() > 0" like other is_available call?
    # Because device like CUDA have a python implementation of is_available that is
    # non-poisoning and some features like Dataloader rely on it.
    # So we are careful to delegate to the Python version of the accelerator here
    acc = current_accelerator()
    if acc is None:
        return False

    mod = torch.get_device_module(acc)
    return mod.is_available()
````
- **EN**: This chunk continues `is_available` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `is_available`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 102-120 / 第 102-120 行
````python
def current_accelerator(check_available: bool = False) -> torch.device | None:
    r"""Return the device of the accelerator available at compilation time.
    If no accelerator were available at compilation time, returns None.
    See :ref:`accelerator<accelerators>` for details.

    Args:
        check_available (bool, optional): if True, will also do a runtime check to see
            if the device :func:`torch.accelerator.is_available` on top of the compile-time
            check.
            Default: ``False``

    Returns:
        torch.device: return the current accelerator as :class:`torch.device`.

    .. note:: The index of the returned :class:`torch.device` will be ``None``, please use
        :func:`torch.accelerator.current_device_index` to know the current index being used.
        This API does NOT poison fork. For more details, see :ref:`multiprocessing-poison-fork-note`.

    Example::
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `current_accelerator`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `current_accelerator`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 122-140 / 第 122-140 行
````python
        >>> # xdoctest:
        >>> # If an accelerator is available, sent the model to it
        >>> model = torch.nn.Linear(2, 2)
        >>> if (current_device := current_accelerator(check_available=True)) is not None:
        >>>     model.to(current_device)
    """
    if (acc := torch._C._accelerator_getAccelerator()) is not None:
        if (not check_available) or (check_available and is_available()):
            return acc
    return None


def current_device_index() -> int:
    r"""Return the index of a currently selected device for the current :ref:`accelerator<accelerators>`.

    Returns:
        int: the index of a currently selected device.
    """
    return torch._C._accelerator_getDeviceIndex()
````
- **EN**: This chunk defines `current_device_index`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `current_device_index`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 143-158 / 第 143-158 行
````python
current_device_idx = deprecated(
    "Use `current_device_index` instead.",
    category=FutureWarning,
)(current_device_index)

current_device_idx.__doc__ = r"""
    (Deprecated) Return the index of a currently selected device for the current :ref:`accelerator<accelerators>`.

    Returns:
        int: the index of a currently selected device.

    .. warning::

        :func:`torch.accelerator.current_device_idx` is deprecated in favor of :func:`torch.accelerator.current_device_index`
        and will be removed in a future PyTorch release.
    """
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 161-175 / 第 161-175 行
````python
@cache
def get_device_capability(device: _device_t = None, /) -> dict[str, Any]:
    r"""Return the capability of the currently selected device.

    Args:
        device (:class:`torch.device`, str, int, optional): The device to query capabilities for
            :ref:`accelerator<accelerators>` device type. If not given,
            use :func:`torch.accelerator.current_device_index` by default.

    Returns:
        dict[str, Any]: A dictionary containing device capability information. The dictionary includes:
            - ``supported_dtypes`` (set(torch.dtype)): Set of PyTorch data types for which
              tensors can be allocated on the accelerator and type conversion across
              supported dtypes are supported. Any operator support outside of that
              is not guaranteed
````
- **EN**: This chunk defines `get_device_capability`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `cache` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `get_device_capability`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `cache` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 177-193 / 第 177-193 行
````python
    Examples:
        >>> # xdoctest: +SKIP("requires cuda")
        >>> # Query capabilities for current device
        >>> capabilities = torch.accelerator.get_device_capability("cuda:0")
        >>> print("Supported dtypes:", capabilities["supported_dtypes"])
    """
    device_index = _get_device_index(device, optional=True)
    # pyrefly: ignore [missing-attribute]
    return torch._C._accelerator_getDeviceCapability(device_index)


def set_device_index(device: _device_t, /) -> None:
    r"""Set the current device index to a given device.

    Args:
        device (:class:`torch.device`, str, int): a given device that must match the current
            :ref:`accelerator<accelerators>` device type.
````
- **EN**: This chunk defines `set_device_index`, which mutates configuration or backend state that affects later execution. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `set_device_index`，其作用是修改会影响后续执行的配置或后端状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 195-213 / 第 195-213 行
````python
    .. note:: This function is a no-op if this device index is negative.
    """
    device_index = _get_device_index(device, optional=False)
    torch._C._accelerator_setDeviceIndex(device_index)


set_device_idx = deprecated(
    "Use `set_device_index` instead.",
    category=FutureWarning,
)(set_device_index)

set_device_idx.__doc__ = r"""
    (Deprecated) Set the current device index to a given device.

    Args:
        device (:class:`torch.device`, str, int): a given device that must match the current
            :ref:`accelerator<accelerators>` device type.

    .. warning::
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk continues `set_device_index` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段延续了 `set_device_index`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 215-232 / 第 215-232 行
````python
        :func:`torch.accelerator.set_device_idx` is deprecated in favor of :func:`torch.accelerator.set_device_index`
        and will be removed in a future PyTorch release.
    """


def current_stream(device: _device_t = None, /) -> torch.Stream:
    r"""Return the currently selected stream for a given device.

    Args:
        device (:class:`torch.device`, str, int, optional): a given device that must match the current
            :ref:`accelerator<accelerators>` device type. If not given,
            use :func:`torch.accelerator.current_device_index` by default.

    Returns:
        torch.Stream: the currently selected stream for a given device.
    """
    device_index = _get_device_index(device, optional=True)
    return torch._C._accelerator_getStream(device_index)
````
- **EN**: This chunk defines `current_stream`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `current_stream`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 235-252 / 第 235-252 行
````python
def set_stream(stream: torch.Stream) -> None:
    r"""Set the current stream to a given stream.

    Args:
        stream (torch.Stream): a given stream that must match the current :ref:`accelerator<accelerators>` device type.

    .. note:: This function will set the current device index to the device index of the given stream.
    """
    torch._C._accelerator_setStream(stream)


def synchronize(device: _device_t = None, /) -> None:
    r"""Wait for all kernels in all streams on the given device to complete.

    Args:
        device (:class:`torch.device`, str, int, optional): device for which to synchronize. It must match
            the current :ref:`accelerator<accelerators>` device type. If not given,
            use :func:`torch.accelerator.current_device_index` by default.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `synchronize`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `synchronize`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 254-270 / 第 254-270 行
````python
    .. note:: This function is a no-op if the current :ref:`accelerator<accelerators>` is not initialized.

    Example::

        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_CUDA)
        >>> assert torch.accelerator.is_available() "No available accelerators detected."
        >>> start_event = torch.Event(enable_timing=True)
        >>> end_event = torch.Event(enable_timing=True)
        >>> start_event.record()
        >>> tensor = torch.randn(100, device=torch.accelerator.current_accelerator())
        >>> sum = torch.sum(tensor)
        >>> end_event.record()
        >>> torch.accelerator.synchronize()
        >>> elapsed_time_ms = start_event.elapsed_time(end_event)
    """
    device_index = _get_device_index(device, optional=True)
    torch._C._accelerator_synchronizeDevice(device_index)
````
- **EN**: This chunk continues `synchronize` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `synchronize`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 273-292 / 第 273-292 行
````python
class device_index:
    r"""Context manager to set the current device index for the current :ref:`accelerator<accelerators>`.
    Temporarily changes the current device index to the specified value for the duration
    of the context, and automatically restores the previous device index when exiting
    the context.

    Args:
        device (Optional[int]): a given device index to temporarily set. If None,
            no device index switching occurs.

    Examples:

        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_CUDA)
        >>> # Set device 0 as the current device temporarily
        >>> with torch.accelerator.device_index(0):
        ...     # Code here runs with device 0 as the current device
        ...     pass
        >>> # Original device is now restored
        >>> # No-op when None is passed
        >>> with torch.accelerator.device_index(None):
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `device_index`, which hold the main object-oriented state for this portion of the file. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `device_index`，这些类承载了本段涉及的主要面向对象状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 293-307 / 第 293-307 行
````python
        ...     # No device switching occurs
        ...     pass
    """

    def __init__(self, device: int | None, /) -> None:
        self.idx = device
        self.prev_idx = -1

    def __enter__(self) -> None:
        if self.idx is not None:
            self.prev_idx = torch._C._accelerator_exchangeDevice(self.idx)

    def __exit__(self, *exc_info: object) -> None:
        if self.idx is not None:
            torch._C._accelerator_maybeExchangeDevice(self.prev_idx)
````
- **EN**: This chunk defines `__exit__`, which implements context-manager boundaries around temporary runtime state. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `__exit__`，其作用是实现围绕临时运行时状态的上下文管理器边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Accelerator runtime**
  - EN: Coordinates device-specific graph capture, cache management, and runtime synchronization.
  - CN: 协调设备相关的图捕获、缓存管理与运行时同步。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **device_count**
  - EN: `device_count` is one of the main symbols declared or implemented in this file.
  - CN: `device_count` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `._utils`, `.graphs`, `.memory`
- **Standard library / 标准库**: `functools`, `typing`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `device_count`, `is_available`, `current_accelerator`, `current_device_index`, `get_device_capability`, `set_device_index`, `current_stream`, `set_stream`, `synchronize`
