# memory.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/xpu/memory.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements XPU-facing Python wrappers for streams, memory, tracing, and runtime behavior.
- **Purpose (CN)**: 实现面向 XPU 的 Python 包装层，用于流、内存、跟踪以及运行时行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
````python
import collections
import contextlib
import ctypes
import pickle
import sys
from typing import Any, Literal

import torch
from torch._utils import _augment_memory_snapshot_stack_traces, _dummy_type
from torch.types import Device

from . import _get_device_index, _is_compiled, _lazy_init, is_initialized


if not _is_compiled():
    # Define dummy base classes
    torch._C.__dict__["_xpu_XPUAllocator"] = _dummy_type("_xpu_XPUAllocator")
    torch._C.__dict__["_XPUMemPool"] = _dummy_type("_XPUMemPool")
    torch._C.__dict__["_xpu_beginAllocateCurrentThreadToPool"] = _dummy_type(
        "_xpu_beginAllocateCurrentThreadToPool"
    )
    torch._C.__dict__["_xpu_endAllocateToPool"] = _dummy_type("_xpu_endAllocateToPool")
    torch._C.__dict__["_xpu_releasePool"] = _dummy_type("_xpu_releasePool")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._utils, torch.types, ...; standard-library helpers such as collections, contextlib, ctypes, .... Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._utils、torch.types、...；标准库辅助模块，如 collections、contextlib、ctypes、...。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 26-51 / 第 26-51 行
````python
def empty_cache() -> None:
    r"""Release all unoccupied cached memory currently held by the caching
    allocator so that those can be used in other XPU application.

    .. note::
        :func:`~torch.xpu.empty_cache` doesn't increase the amount of XPU
        memory available for PyTorch. However, it may help reduce fragmentation
        of XPU memory in certain cases.
    """
    if is_initialized():
        torch._C._xpu_emptyCache()


def reset_peak_memory_stats(device: Device = None) -> None:
    r"""Reset the "peak" stats tracked by the XPU memory allocator.

    See :func:`~torch.xpu.memory_stats` for details. Peak stats correspond to the
    `"peak"` key in each individual stat dict.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    device = _get_device_index(device, optional=True)
    return torch._C._xpu_resetPeakMemoryStats(device)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `reset_peak_memory_stats`, which drops cached state so a later execution phase can rebuild it cleanly. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `reset_peak_memory_stats`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 54-78 / 第 54-78 行
````python
def reset_accumulated_memory_stats(device: Device = None) -> None:
    r"""Reset the "accumulated" (historical) stats tracked by the XPU memory allocator.

    See :func:`~torch.xpu.memory_stats` for details. Accumulated stats correspond to
    the `"allocated"` and `"freed"` keys in each individual stat dict.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    device = _get_device_index(device, optional=True)
    return torch._C._xpu_resetAccumulatedMemoryStats(device)


def memory_stats_as_nested_dict(device: Device = None) -> dict[str, Any]:
    r"""Return the result of :func:`~torch.xpu.memory_stats` as a nested dictionary."""
    if not is_initialized():
        return {}
    device = _get_device_index(device, optional=True)
    return torch._C._xpu_memoryStats(device)


def memory_stats(device: Device = None) -> dict[str, Any]:
    r"""Return a dictionary of XPU memory allocator statistics for a given device.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `memory_stats`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `memory_stats`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 80-103 / 第 80-103 行
````python
    The return value of this function is a dictionary of statistics, each of
    which is a non-negative integer.

    Core statistics:

    - ``"allocated_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of allocated memory.
    - ``"reserved_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of reserved memory.
    - ``"active_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      amount of active memory.
    - ``"requested_bytes.{all,large_pool,small_pool}.{current,peak,allocated,freed}"``:
      memory requested by client code, compare this with allocated_bytes to check if
      allocation rounding adds too much overhead.

    For these core statistics, values are broken down as follows.

    Pool type:

    - ``all``: combined statistics across all memory pools.
    - ``large_pool``: statistics for the large allocation pool (for size >= 1MB allocations).
    - ``small_pool``: statistics for the small allocation pool (for size < 1MB allocations).

    Metric type:
````
- **EN**: This chunk continues `memory_stats` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `memory_stats`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 105-130 / 第 105-130 行
````python
    - ``current``: current value of this metric.
    - ``peak``: maximum value of this metric.
    - ``allocated``: historical total increase in this metric.
    - ``freed``: historical total decrease in this metric.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistics for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    result = []

    def _recurse_add_to_result(prefix: str, obj: Any) -> None:
        if isinstance(obj, dict):
            if len(prefix) > 0:
                prefix += "."
            for k, v in obj.items():
                _recurse_add_to_result(prefix + k, v)
        else:
            result.append((prefix, obj))

    stats = memory_stats_as_nested_dict(device=device)
    _recurse_add_to_result("", stats)
    result.sort()

    return collections.OrderedDict(result)
````
- **EN**: This chunk defines `_recurse_add_to_result`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_recurse_add_to_result`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 133-156 / 第 133-156 行
````python
def memory_allocated(device: Device = None) -> int:
    r"""Return the current GPU memory occupied by tensors in bytes for a given device.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).

    .. note::
        This is likely less than the amount shown in `xpu-smi` since some
        unused memory can be held by the caching allocator and some context
        needs to be created on GPU.
    """
    return memory_stats(device=device).get("allocated_bytes.all.current", 0)


def max_memory_allocated(device: Device = None) -> int:
    r"""Return the maximum GPU memory occupied by tensors in bytes for a given device.

    By default, this returns the peak allocated memory since the beginning of
    this program. :func:`~torch.xpu.reset_peak_memory_stats` can be used to
    reset the starting point in tracking this metric. For example, these two
    functions can measure the peak allocated memory usage of each iteration in a
    training loop.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `max_memory_allocated`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `max_memory_allocated`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 158-184 / 第 158-184 行
````python
    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    return memory_stats(device=device).get("allocated_bytes.all.peak", 0)


def memory_reserved(device: Device = None) -> int:
    r"""Return the current GPU memory managed by the caching allocator in bytes for a given device.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    return memory_stats(device=device).get("reserved_bytes.all.current", 0)


def max_memory_reserved(device: Device = None) -> int:
    r"""Return the maximum GPU memory managed by the caching allocator in bytes for a given device.

    By default, this returns the peak cached memory since the beginning of this
    program. :func:`~torch.xpu.reset_peak_memory_stats` can be used to reset
    the starting point in tracking this metric. For example, these two functions
    can measure the peak cached memory amount of each iteration in a training
    loop.
````
- **EN**: This chunk defines `max_memory_reserved`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `max_memory_reserved`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 186-209 / 第 186-209 行
````python
    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
    """
    return memory_stats(device=device).get("reserved_bytes.all.peak", 0)


def mem_get_info(device: Device = None) -> tuple[int, int]:
    r"""Return the global free and total GPU memory for a given device.

    Args:
        device (torch.device or int or str, optional): selected device. Returns
            statistic for the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).

    Returns:
        tuple[int, int]: a tuple of two integers (free_memory, total_memory) in bytes.
            The first value is the free memory on the device (available across all processes and applications),
            The second value is the device's total hardware memory capacity.
    """
    _lazy_init()
    device = _get_device_index(device, optional=True)
    return torch._C._xpu_getMemoryInfo(device)
````
- **EN**: This chunk defines `mem_get_info`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mem_get_info`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 212-238 / 第 212-238 行
````python
def get_per_process_memory_fraction(device: Device = None) -> float:
    r"""
    Retrieve the memory fraction currently set for a process on a given XPU device.
    This fraction represents the portion of the total device memory that
    the caching allocator is allowed to use. The allowed memory is calculated as:

    .. math:: \text{allowed\_memory} = \text{total\_memory} \times \text{fraction}

    Args:
        device (torch.device or int or str, optional): selected device. It uses the current device,
            given by :func:`~torch.xpu.current_device`, if :attr:`device` is ``None`` (default).

    Returns:
        float: The memory fraction in the range 0.0 to 1.0.
    """
    _lazy_init()
    device = _get_device_index(device, optional=True)
    return torch._C._xpu_getMemoryFraction(device)


def set_per_process_memory_fraction(fraction: float, device: Device = None) -> None:
    r"""
    Set the memory fraction for a single process on XPU device.
    This function limits the amount of memory that the caching allocator can allocate
    on the specified XPU device. The allowed memory is computed as:

    .. math:: \text{allowed\_memory} = \text{total\_memory} \times \text{fraction}
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `set_per_process_memory_fraction`, which mutates configuration or backend state that affects later execution. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `set_per_process_memory_fraction`，其作用是修改会影响后续执行的配置或后端状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 240-266 / 第 240-266 行
````python
    If the process attempts to allocate more than this allowed memory,
    an out-of-memory error will be raised by the allocator.

    Arguments:
        fraction (float): Range: 0~1. Allowed memory equals total_memory * fraction.
        device (torch.device or int or str, optional): selected device. It uses the current device,
            given by :func:`~torch.xpu.current_device`, if :attr:`device` is ``None`` (default).

    .. note:: In general, the total available free memory is less than the total capacity.
    """
    _lazy_init()
    device = _get_device_index(device, optional=True)
    if not isinstance(fraction, float):
        raise TypeError("Invalid type for fraction argument, must be `float`")
    torch._C._xpu_setMemoryFraction(fraction, device)


def memory_snapshot(
    mempool_id: tuple[int, int] | None = None,
) -> list[dict[str, Any]]:
    r"""
    Return a snapshot of the XPU memory allocator state across all devices.
    Provides detailed information for each memory segment managed by the allocator
    including its size, owning pool, associated stream, call stack traces, and other relevant attributes.

    Arguments:
        mempool_id (tuple[int, int] or None, optional): The memory pool id. If None, the default memory pool is used.
````
- **EN**: This chunk defines `memory_snapshot`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `memory_snapshot`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 268-286 / 第 268-286 行
````python
    Returns:
        list[dict[str, Any]]: List of memory segments and their attributes.
    """
    if not is_initialized():
        return []
    return torch._C._xpu_memorySnapshot(mempool_id)["segments"]


def _snapshot(device: Device = None, augment_with_fx_traces: bool = False):
    """
    Capture a snapshot of the XPU memory state at the time this function is called.

    The returned snapshot is a dictionary with the following structure.

    .. code-block:: python

        class Snapshot(TypedDict):
            segments: List[Segment]
            device_traces: List[List[TraceEntry]]
````
- **EN**: It introduces or extends `Snapshot`, which hold the main object-oriented state for this portion of the file. This chunk defines `_snapshot`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 它引入或扩展了 `Snapshot`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_snapshot`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 289-316 / 第 289-316 行
````python
        class Segment(TypedDict):
            # A Segment represents a contiguous memory region returned by the SYCL runtime.
            #
            # All reserved memory is composed of these segments. Segments are
            # cached and reused by the allocator. When allocations are smaller
            # than the segment, the segment may be split into multiple Blocks.
            #
            # Calling :func:`~torch.xpu.memory.empty_cache` releases segments that are entirely inactive.
            address: int
            total_size: int  #  total size of segment
            stream: int
            segment_type: Literal["small", "large"]  # 'large' (>1MB)
            allocated_size: int  # size of memory in use
            active_size: int  # size of memory in use or in active_awaiting_free state
            blocks: List[Block]


        class Block(TypedDict):
            # A sub-region of a Segment, either currently allocated or cached for reuse.
            size: int
            requested_size: int  # Original requested size (may be smaller than `size`)
            address: int
            state: Literal[
                "active_allocated",  # used by a tensor
                "active_awaiting_free",  # waiting for another stream synchronization, then become free
                "inactive",  # free for reuse
            ]
            frames: List[Frame]  # stack trace from where the allocation occurred
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. It introduces or extends `Segment`, `Block`, which hold the main object-oriented state for this portion of the file. This chunk continues `Block` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 它引入或扩展了 `Segment`、`Block`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `Block`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 319-346 / 第 319-346 行
````python
        class Frame(TypedDict):
            filename: str
            line: int
            name: str
            # Optional fields when `augment_with_fx_traces=True` and the frame
            # corresponds to FX-generated code.
            fx_node_op: str  # FX node operation type (e.g., 'call_function', 'output')
            fx_node_name: str  # FX node name (e.g., 'linear', 'relu_1')
            fx_original_trace: str  # Original model source code stack trace


        class TraceEntry(TypedDict):
            # Trace entries are recorded only when :func:`~torch.xpu.memory._record_memory_history` is enabled.
            action: Literal[
                "alloc"  # memory allocated
                "free_requested",  # received a call to free memory
                "free_completed",  # memory reclaimed and reusable
                "segment_alloc",  # ask SYCL runtime for more memory
                "segment_free",  # called SYCL runtime to return memory to XPU
                "segment_map",  # ask SYCL runtime to map memory
                "segment_unmap",  # called SYCL runtime to unmap memory
                "snapshot",  # snapshot taken
                "oom",  # threw an OOM exception
            ]
            addr: int  # not present for OOM
            frames: List[Frame]
            size: int
            stream: int
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. It introduces or extends `Frame`, `TraceEntry`, which hold the main object-oriented state for this portion of the file. This chunk continues `TraceEntry` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 它引入或扩展了 `Frame`、`TraceEntry`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `TraceEntry`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 347-371 / 第 347-371 行
````python
            device_free: int  # only present for OOM, the amount of free memory reported by the device

    Arguments:
        device (torch.device or int or str, optional): selected device. It uses the current device,
            given by :func:`~torch.xpu.current_device`, if :attr:`device` is ``None`` (default).
        augment_with_fx_traces (bool, optional): If True, augment stack trace frames with FX debug information
            that maps generated FX code back to original model source code. This adds the FX-related
            fields (fx_node_op, fx_node_name, fx_original_trace) to Frame objects. Default is ``False``.

    Returns:
        The Snapshot dictionary object
    """
    s = torch._C._xpu_memorySnapshot(None)
    if augment_with_fx_traces:
        s = _augment_memory_snapshot_stack_traces(s)  # type: ignore[assignment, arg-type]
    return s


def _dump_snapshot(
    filename: str = "dump_snapshot.pickle", augment_with_fx_traces: bool = False
) -> None:
    """
    Save a pickled version of the `torch.memory._snapshot()` dictionary to a file.

    This file can be opened by the interactive snapshot viewer at pytorch.org/memory_viz
````
- **EN**: This chunk defines `_dump_snapshot`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_dump_snapshot`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 373-399 / 第 373-399 行
````python
    Snapshot file sizes scale with `max_entries` and stack trace depth per entry,
    with several KB per entry. These can easily be in the GB range for longer running
    workflows with large `max_entries`.

    Arguments:
        filename (str, optional): Name of the file to create. Defaults to "dump_snapshot.pickle".
        augment_with_fx_traces (bool, optional): If True, augment the snapshot with FX debug information
            before dumping. This maps generated FX code stack traces back to original model
            source code. Defaults to ``False``.
    """
    s = _snapshot(augment_with_fx_traces=augment_with_fx_traces)

    with open(filename, "wb") as f:
        pickle.dump(s, f)


def _record_memory_history(
    enabled: Literal["state", "all"] | None = "all",
    context: Literal["state", "alloc", "all"] | None = "all",
    stacks: Literal["python", "all"] = "all",
    max_entries: int = sys.maxsize,
    clear_history: bool = False,
    skip_actions: list[str] | None = None,
) -> None:
    """
    Enable recording of stack traces associated with memory allocations, so you can
    tell what allocated any piece of memory in :func:`~torch.xpu.memory._snapshot()`.
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `_record_memory_history`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `_record_memory_history`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 401-427 / 第 401-427 行
````python
    In addition to keeping stack traces with each current allocation and free,
    this will also enable recording of a history of all alloc/free events.

    Use :func:`~torch.xpu.memory._snapshot()` to retrieve this information,
    and the tools in `_memory_viz.py` to visualize snapshots.

    Buffer behavior
    ---------------

    This will store up to `max_entries` instances of `TraceEntry` when enabled.
    Python trace collection defaults to `sys.maxsize`, meaning long-running
    or indefinitely running jobs should set a reasonable limit to avoid excessive
    memory use. Expect each entry to be several KB.

    Longer running workflows or those with smaller `max_entries` values will only
    store the last accumulated `max_entries` entries, meaning new entries overwrite
    older entries, reference to ring buffer behavior.

    Latency impact
    --------------

    The Python trace collection is fast (2us per trace), so you may consider
    enabling this on production jobs if you anticipate ever having to debug
    memory issues.

    C++ trace collection is also fast (~50ns/frame), which for many typical programs
    works out to ~2us per trace, but can vary depending on stack depth.
````
- **EN**: This chunk continues `_record_memory_history` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_record_memory_history`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 429-450 / 第 429-450 行
````python
    Arguments:
        enabled (Literal["state", "all"], optional):
            `None`, disable recording memory history.
            `"state"`, keep information for currently allocated memory.
            `"all"`, additionally keep a history of all alloc/free calls.
            Defaults to "all".
        context (Literal["state", "alloc", "all"], optional):
            `None`, Do not record any tracebacks.
            `"state"`, Record tracebacks for currently allocated memory.
            `"alloc"`, additionally keep tracebacks for alloc calls.
            `"all"`, additionally keep tracebacks for free calls.
            Defaults to "all".
        stacks (Literal["python", "all"], optional):
            `"python"`, include Python, TorchScript, and inductor frames in tracebacks.
            `"all"`, additionally include C++ frames.
            Defaults to "all".
        max_entries (int, optional): Keep a maximum of `max_entries`
            alloc/free events in the recorded history recorded.
        clear_history (bool, optional): Clear history when enabling, defaults to ``False``.
        skip_actions (list[str], optional): List of action types to skip when recording
            memory history. This can be used to reduce memory overhead by excluding
            certain types of events from being recorded. Valid action types are:
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk continues `_record_memory_history` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段延续了 `_record_memory_history`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 452-478 / 第 452-478 行
````python
            - `"alloc"`: Memory allocation events
            - `"free_requested"`: Free requests (memory marked for freeing)
            - `"free_completed"`: Completed free operations (memory actually freed)
            - `"segment_alloc"`: Segment allocation from SYCL runtime
            - `"segment_free"`: Segment freed back to XPU via SYCL runtime
            - `"segment_map"`: Segment map events
            - `"segment_unmap"`: Segment unmap events
            - `"snapshot"`: Memory snapshot generation events
            - `"oom"`: Out-of-memory exceptions

            For example, to skip recording free_requested events:
            `skip_actions=["free_requested"]`

            Defaults to ``None`` (record all actions).
    """
    torch._C._xpu_recordMemoryHistory(
        enabled,
        context,
        stacks,
        max_entries,
        clear_history,
        skip_actions if skip_actions is not None else [],
    )


class _XPUAllocator:
    r"""Wrapper over internal XPU memory allocators."""
````
- **EN**: It introduces or extends `_XPUAllocator`, which hold the main object-oriented state for this portion of the file. This chunk continues `_XPUAllocator` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `_XPUAllocator`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `_XPUAllocator`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 480-503 / 第 480-503 行
````python
    def __init__(self, allocator: torch._C._xpu_XPUAllocator):
        self._allocator = allocator

    def allocator(self):
        return self._allocator


class XPUPluggableAllocator(_XPUAllocator):
    r"""
    XPU memory allocator loaded dynamically from a shared library.

    This lets users provide custom allocation and free functions implemented
    in a separate shared library. The allocator is registered and could become
    available for use via :func:`~torch.xpu.memory.change_current_allocator`.

    Arguments:
        path_to_lib_file (str):
            Filesystem path to the shared library file containing the allocation
            and free functions.
        alloc_fn_name (str):
            Name of the allocation function exported from the shared library.
            The function must have the signature:

                ``void* alloc_fn(size_t size, int device, sycl::queue* queue);``
````
- **EN**: It introduces or extends `XPUPluggableAllocator`, which hold the main object-oriented state for this portion of the file. This chunk defines `allocator`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `XPUPluggableAllocator`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `allocator`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 505-530 / 第 505-530 行
````python
        free_fn_name (str):
            Name of the free function exported from the shared library.
            The function must have the signature:

                ``void free_fn(void* ptr, size_t size, int device, sycl::queue* queue);``
    """

    def __init__(self, path_to_lib_file: str, alloc_fn_name: str, free_fn_name: str):
        allocator_lib = ctypes.CDLL(path_to_lib_file)

        alloc_fn_ptr = getattr(allocator_lib, alloc_fn_name)
        free_fn_ptr = getattr(allocator_lib, free_fn_name)

        alloc_fn_addr = ctypes.cast(alloc_fn_ptr, ctypes.c_void_p).value
        free_fn_addr = ctypes.cast(free_fn_ptr, ctypes.c_void_p).value

        if alloc_fn_addr is None or free_fn_addr is None:
            raise RuntimeError(
                "Failed to load allocator symbols from the shared library."
            )

        self._allocator = torch._C._xpu_customAllocator(alloc_fn_addr, free_fn_addr)


def change_current_allocator(allocator: _XPUAllocator) -> None:
    r"""Change the currently used memory allocator to be the one provided.
````
- **EN**: This chunk defines `change_current_allocator`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段定义了 `change_current_allocator`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 532-552 / 第 532-552 行
````python
    .. note::
        If the current allocator has already been used/initialized, this function will error.

    Arguments:
        allocator (torch.xpu.memory._XPUAllocator): allocator to be set as the active one.
    """
    torch._C._xpu_changeCurrentAllocator(allocator.allocator())


def _get_current_allocator() -> _XPUAllocator:
    r"""Return the allocator being currently used.

    Returns:
        _XPUAllocator: the allocator being currently used.
    """
    return _XPUAllocator(torch._C._xpu_getAllocator())


class MemPool(torch._C._XPUMemPool):
    r"""MemPool represents a pool of memory in a caching allocator. Currently,
    it's just the ID of the pool object maintained in the XPUCachingAllocator.
````
- **EN**: It introduces or extends `MemPool`, which hold the main object-oriented state for this portion of the file. This chunk defines `_get_current_allocator`, which implements a focused helper used by the surrounding module.
- **CN**: 它引入或扩展了 `MemPool`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_get_current_allocator`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 554-580 / 第 554-580 行
````python
    Args:
        allocator(torch._C._xpu_XPUAllocator, optional): a
            torch._C._xpu_XPUAllocator object that can be used to
            define how memory gets allocated in the pool. If :attr:`allocator`
            is ``None`` (default), memory allocation follows the default/
            current configuration of the XPUCachingAllocator.
        use_on_oom(bool): a bool that indicates if this pool can be used
            as a last resort if a memory allocation outside of the pool fails due
            to Out Of Memory. This is ``False`` by default.
    """

    def __init__(
        self,
        allocator: torch._C._xpu_XPUAllocator | None = None,
        use_on_oom: bool = False,
    ):
        super().__init__(allocator, True, use_on_oom)

    @property
    def id(self) -> tuple[int, int]:
        r"""Returns the ID of this pool as a tuple of two ints."""
        return super().id

    @property
    def allocator(self) -> torch._C._xpu_XPUAllocator | None:
        r"""Returns the allocator this MemPool routes allocations to."""
        return super().allocator
````
- **EN**: This chunk defines `allocator`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `allocator`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 582-606 / 第 582-606 行
````python
    def use_count(self) -> int:
        r"""Returns the reference count of this pool."""
        return super().use_count()

    def snapshot(self):
        r"""Return a snapshot of the XPU memory allocator pool state across all
        devices.

        Interpreting the output of this function requires familiarity with the
        memory allocator internals.
        """
        snapshot = torch.xpu.memory_snapshot(self.id)
        return snapshot


@contextlib.contextmanager
def use_mem_pool(pool: MemPool, device: "Device" = None):
    r"""A context manager that routes allocations to a given pool.

    Args:
        pool(torch.xpu.MemPool): a :class:`MemPool` object to be made active so that
            allocations route to this pool.
        device (torch.device or int, optional): selected device. Uses :class:`MemPool on
            the current device, given by :func:`~torch.xpu.current_device`,
            if :attr:`device` is ``None`` (default).
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `use_mem_pool`, which implements a focused helper used by the surrounding module. Decorators such as `contextlib.contextmanager` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `use_mem_pool`，其作用是实现周边模块使用的关键辅助逻辑。 像 `contextlib.contextmanager` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 608-635 / 第 608-635 行
````python
    .. note::
        This context manager makes only current thread's allocations route to
        the given pool. If a new thread is spawned inside the context manager
        (e.g. by calling backward) the allocations in that thread will not
        route to the given pool.
    """
    device_index = (
        torch.xpu.current_device() if device is None else _get_device_index(device)
    )
    torch._C._xpu_beginAllocateCurrentThreadToPool(device_index, pool.id)
    try:
        yield
    finally:
        torch._C._xpu_endAllocateToPool(device_index, pool.id)
        torch._C._xpu_releasePool(device_index, pool.id)


__all__ = [
    "MemPool",
    "XPUPluggableAllocator",
    "change_current_allocator",
    "empty_cache",
    "get_per_process_memory_fraction",
    "max_memory_allocated",
    "max_memory_reserved",
    "mem_get_info",
    "memory_allocated",
    "memory_reserved",
````
- **EN**: `__all__` defines the public symbols that this module chooses to export. This chunk continues `use_mem_pool` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: `__all__` 定义了本模块选择导出的公共符号。 这一段延续了 `use_mem_pool`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 636-643 / 第 636-643 行
````python
    "memory_snapshot",
    "memory_stats",
    "memory_stats_as_nested_dict",
    "reset_accumulated_memory_stats",
    "reset_peak_memory_stats",
    "set_per_process_memory_fraction",
    "use_mem_pool",
]
````
- **EN**: This chunk continues `use_mem_pool` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `use_mem_pool`，进一步展开其内部控制流或状态更新。

## Key Concepts / 关键概念

- **XPU runtime**
  - EN: Wraps XPU device state, streams, memory, and tracing helpers.
  - CN: 封装 XPU 设备状态、流、内存以及跟踪辅助逻辑。
- **empty_cache**
  - EN: `empty_cache` is one of the main symbols declared or implemented in this file.
  - CN: `empty_cache` 是本文件声明或实现的主要符号之一。
- **reset_peak_memory_stats**
  - EN: `reset_peak_memory_stats` is one of the main symbols declared or implemented in this file.
  - CN: `reset_peak_memory_stats` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Deferred execution**
  - EN: The file records intent or metadata now so execution can be materialized later.
  - CN: 该文件先记录意图或元数据，等待后续阶段再真正执行。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._utils`, `torch.types`, `.`
- **Standard library / 标准库**: `collections`, `contextlib`, `ctypes`, `pickle`, `sys`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `empty_cache`, `reset_peak_memory_stats`, `reset_accumulated_memory_stats`, `memory_stats_as_nested_dict`, `memory_stats`, `memory_allocated`, `max_memory_allocated`, `memory_reserved`, `max_memory_reserved`, `mem_get_info`
