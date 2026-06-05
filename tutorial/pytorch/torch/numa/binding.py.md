# binding.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/numa/binding.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-19 / 第 1-19 行
````python
"""
In NUMA (Non-Uniform Memory Access) systems, accessing memory on remote NUMA
nodes incurs additional latency. PyTorch provides NUMA binding utilities to
promote memory locality by binding worker processes to CPUs near their assigned GPUs.

In practice, NUMA binding typically results in 1-10% overall performance improvements,
but some workloads may obtain much greater benefits or none at all.

To enable NUMA binding, use the ``--numa-binding`` flag with :ref:`torchrun <launcher-api>`, e.g.:

.. code-block:: bash

    torchrun --numa-binding=node --nproc_per_node=8 train.py

Alternatively, pass :class:`NumaOptions` to ``LaunchConfig``
when using ``elastic_launch``.

See :class:`AffinityMode` for available binding modes.
"""
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 21-41 / 第 21-41 行
````python
import os
import shutil
import traceback
from collections import defaultdict
from collections.abc import Callable, Iterable
from dataclasses import asdict, dataclass
from enum import Enum
from functools import wraps
from logging import getLogger
from typing import ParamSpec, TypeVar

import torch
from torch._utils_internal import signpost_event


__all__ = [
    "AffinityMode",
    "NumaOptions",
]

logger = getLogger(__name__)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._utils_internal; standard-library helpers such as os, shutil, traceback, .... `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._utils_internal；标准库辅助模块，如 os、shutil、traceback、...。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 44-64 / 第 44-64 行
````python
class AffinityMode(str, Enum):
    NODE = "node"
    """
    Each worker process and its threads will be bound to all the CPUs
    on the NUMA node containing the GPU whose local index equals the worker's local rank.
    If in doubt, use this option rather than the others.

    **Ex.:** If GPU 3 (i.e. ``torch.device("cuda:3")``) lives on NUMA node 1, then the worker
    whose local rank is 3 will only be able to run on the CPUs of NUMA node 1.
    """

    SOCKET = "socket"
    """
    Each worker process and its threads will be bound to all the CPUs on all the NUMA nodes of the
    socket containing the GPU whose local index equals the worker's local rank.

    **Ex.:** If socket 0 contains GPU 3 and NUMA nodes 0-1, then the worker whose
    local rank is 3 will be bound to the CPUs of NUMA nodes 0-1.

    For cases where there is only one NUMA node per socket anyway, this is equivalent to NODE.
    """
````
- **EN**: It introduces or extends `AffinityMode`, which hold the main object-oriented state for this portion of the file.
- **CN**: 它引入或扩展了 `AffinityMode`，这些类承载了本段涉及的主要面向对象状态。

### Lines 66-92 / 第 66-92 行
````python
    EXCLUSIVE = "exclusive"
    """
    Each worker process and its threads will be bound to an exclusive subset of CPUs
    on the NUMA node containing the GPU whose local index equals the worker's local rank.
    The CPUs on the NUMA node are divided evenly among all GPUs on that node, so no two
    workers share the same CPU cores.

    **Ex.:** If NUMA node 1 has 16 physical cores and GPUs 2 and 3, then the worker whose
    local rank is 2 will be bound to cores 0-7, and the worker whose local rank is 3 will
    be bound to cores 8-15.
    """

    CORE_COMPLEX = "core-complex"
    """
    Each worker process and its threads will be bound to a single core complex (a group of cores
    sharing the same L3 cache) on the NUMA node containing the GPU whose local index equals
    the worker's local rank. Each worker is bound to a different core complex when possible.

    **Ex.:** If NUMA node 1 has two core complexes (cores 0-7 sharing one L3 cache, cores 8-15
    sharing another) and GPUs 2 and 3, then the worker whose local rank is 2 will be bound to
    cores 0-7, and the worker whose local rank is 3 will be bound to cores 8-15.
    """


@dataclass(frozen=True)
class NumaOptions:
    affinity_mode: AffinityMode
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `NumaOptions`, which hold the main object-oriented state for this portion of the file. This chunk continues `NumaOptions` and expands its internal control flow or state updates. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `NumaOptions`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `NumaOptions`，进一步展开其内部控制流或状态更新。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 94-115 / 第 94-115 行
````python
    should_fall_back_if_binding_fails: bool = False
    """
    If ``True``, we will silence any exceptions that occur during NUMA binding itself
    rather than raising them.

    There are no expected exceptions, so avoid using this option. Its purpose is simply
    to mitigate crash risk while conducting mass rollouts of NUMA binding.
    """


def _maybe_wrap_command_args_with_numa_binding(
    command_args: tuple[str, ...],
    *,
    gpu_index: int,
    numa_options: NumaOptions | None,
) -> tuple[str, ...]:
    """
    Wraps command arguments with numactl to apply NUMA CPU binding.

    This function prepends numactl with appropriate CPU affinity flags to the
    provided command arguments, binding the process to CPUs associated with
    the specified GPU's NUMA node.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_maybe_wrap_command_args_with_numa_binding`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_maybe_wrap_command_args_with_numa_binding`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 117-141 / 第 117-141 行
````python
    Args:
        command_args: The original command arguments to wrap.
        gpu_index: The index of the GPU that will be used by the subprocess.
        numa_options: Configuration for NUMA binding behavior. If None, returns
            the original command_args unchanged.

    Returns:
        Tuple of command arguments, potentially wrapped with numactl for NUMA binding.
        Returns the original command_args if numa_options is None or if binding fails
        and fallback is enabled.
    """
    if numa_options is None:
        return command_args

    kwargs = {
        "command_args": command_args,
        "gpu_index": gpu_index,
        "numa_options": asdict(numa_options),
    }

    try:
        logical_cpu_indices = _get_validated_logical_cpus_to_bind_to(
            gpu_index=gpu_index,
            numa_options=numa_options,
        )
````
- **EN**: This chunk continues `_maybe_wrap_command_args_with_numa_binding` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_maybe_wrap_command_args_with_numa_binding`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 143-163 / 第 143-163 行
````python
        wrapped_command_args = _assemble_numactl_command_args(
            original_command_args=command_args,
            logical_cpu_indices=logical_cpu_indices,
        )
        signpost_event(
            category="numa_binding",
            name="apply_success",
            parameters={
                **kwargs,
                "wrapped_command": wrapped_command_args,
            },
        )
        return wrapped_command_args
    except Exception:
        # pyrefly: ignore [bad-argument-type]
        _handle_exception(numa_options=numa_options, logger_kwargs=kwargs)
        return command_args


_TParams = ParamSpec("_TParams")
_TReturn = TypeVar("_TReturn")
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk continues `_maybe_wrap_command_args_with_numa_binding` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段延续了 `_maybe_wrap_command_args_with_numa_binding`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 166-190 / 第 166-190 行
````python
def _maybe_wrap_with_numa_binding(
    func: Callable[_TParams, _TReturn],
    *,
    gpu_index: int,
    numa_options: NumaOptions | None,
) -> Callable[_TParams, _TReturn]:
    """
    Wraps a function to apply NUMA CPU binding before execution.

    This decorator applies NUMA CPU affinity to all threads in the current process
    before calling the wrapped function, binding them to CPUs associated with the
    specified GPU's NUMA node.

    Args:
        func: The function to wrap with NUMA binding.
        gpu_index: The index of the GPU that will be used.
        numa_options: Configuration for NUMA binding behavior. If None, returns
            the original function unchanged.

    Returns:
        A wrapped function that applies NUMA binding before execution, or the
        original function if numa_options is None.
    """
    if numa_options is None:
        return func
````
- **EN**: This chunk defines `_maybe_wrap_with_numa_binding`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_maybe_wrap_with_numa_binding`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 192-216 / 第 192-216 行
````python
    @wraps(func)
    def wrapped(*args: _TParams.args, **kwargs: _TParams.kwargs) -> _TReturn:
        _maybe_apply_numa_binding_to_current_process(
            gpu_index=gpu_index,
            # pyrefly: ignore [bad-argument-type]
            numa_options=numa_options,
        )
        return func(*args, **kwargs)

    return wrapped


def _maybe_apply_numa_binding_to_current_process(
    *, gpu_index: int, numa_options: NumaOptions
) -> None:
    kwargs = {
        "gpu_index": gpu_index,
        "numa_options": asdict(numa_options),
    }

    try:
        logical_cpu_indices = _get_validated_logical_cpus_to_bind_to(
            gpu_index=gpu_index,
            numa_options=numa_options,
        )
````
- **EN**: This chunk defines `_maybe_apply_numa_binding_to_current_process`, which implements a focused helper used by the surrounding module. Decorators such as `wraps` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_maybe_apply_numa_binding_to_current_process`，其作用是实现周边模块使用的关键辅助逻辑。 像 `wraps` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 218-242 / 第 218-242 行
````python
        _bind_all_threads_in_current_process_to_logical_cpus(
            logical_cpu_indices=logical_cpu_indices
        )

        signpost_event(
            category="numa_binding",
            name="apply_success",
            parameters={
                **kwargs,
                "logical_cpu_indices": _get_ranges_str_from_ints(logical_cpu_indices),
            },
        )
    except Exception:
        # pyrefly: ignore [bad-argument-type]
        _handle_exception(numa_options=numa_options, logger_kwargs=kwargs)


def _assemble_numactl_command_args(
    *, original_command_args: tuple[str, ...], logical_cpu_indices: set[int]
) -> tuple[str, ...]:
    return (
        "numactl",
        f"--physcpubind={_get_ranges_str_from_ints(logical_cpu_indices)}",
        *original_command_args,
    )
````
- **EN**: This chunk defines `_assemble_numactl_command_args`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_assemble_numactl_command_args`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 245-265 / 第 245-265 行
````python
def _handle_exception(
    *, numa_options: NumaOptions, logger_kwargs: dict[str, object]
) -> None:
    signpost_event(
        category="numa_binding",
        name="apply_exception",
        parameters={
            **logger_kwargs,
            "traceback": traceback.format_exc(),
        },
    )
    logger.exception("Failed to apply NUMA binding for input=%r", logger_kwargs)
    if numa_options.should_fall_back_if_binding_fails:
        logger.warning(
            "Continuing executing without applying NUMA binding, despite exception %s",
            traceback.format_exc(),
        )
        return
    # This function is called within an except block, so silence the warning
    # about raise without an exception.
    raise  # noqa: PLE0704
````
- **EN**: This chunk defines `_handle_exception`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_handle_exception`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 268-288 / 第 268-288 行
````python
def _get_validated_logical_cpus_to_bind_to(
    *,
    gpu_index: int,
    numa_options: NumaOptions,
) -> set[int]:
    logical_cpu_indices = _get_logical_cpus_to_bind_to(
        gpu_index=gpu_index, numa_options=numa_options
    )
    _raise_if_binding_invalid(logical_cpu_indices=logical_cpu_indices)

    return logical_cpu_indices


def _raise_if_binding_invalid(*, logical_cpu_indices: set[int]) -> None:
    # NOTE: numactl CLI is only actually necessary for the str entrypoint path,
    # but for simplicity we will just check it no matter what.
    if shutil.which("numactl") is None:
        raise RuntimeError("numactl CLI is required for NUMA binding")

    if not logical_cpu_indices:
        raise RuntimeError("Must bind to a non-empty set of CPU indices")
````
- **EN**: This chunk defines `_raise_if_binding_invalid`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_raise_if_binding_invalid`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 291-316 / 第 291-316 行
````python
def _bind_all_threads_in_current_process_to_logical_cpus(
    *, logical_cpu_indices: set[int]
) -> None:
    # Save the original affinity of the main thread before changing it
    # pyrefly: ignore [missing-attribute]
    original_main_thread_affinity = os.sched_getaffinity(0)  # type: ignore[attr-defined]

    # 0 represents the current thread.
    # This is outside the try/except because the main thread should always bind successfully.
    # pyrefly: ignore [missing-attribute]
    os.sched_setaffinity(0, logical_cpu_indices)  # type: ignore[attr-defined]

    for tid_str in os.listdir("/proc/self/task"):
        try:
            tid = int(tid_str)
            # pyrefly: ignore [missing-attribute]
            tid_affinity = os.sched_getaffinity(tid)  # type: ignore[attr-defined]

            # Defensive check to ensure we do not overwrite affinity on any threads
            # that have already had their affinity set elsewhere.
            if tid_affinity == original_main_thread_affinity:
                # pyrefly: ignore [missing-attribute]
                os.sched_setaffinity(tid, logical_cpu_indices)  # type: ignore[attr-defined]
        except Exception:
            # Thread may have exited or otherwise become invalid
            pass
````
- **EN**: This chunk defines `_bind_all_threads_in_current_process_to_logical_cpus`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `_bind_all_threads_in_current_process_to_logical_cpus`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 319-344 / 第 319-344 行
````python
def _get_logical_cpus_to_bind_to(
    *,
    gpu_index: int,
    numa_options: NumaOptions,
) -> set[int]:
    """
    Args:
        gpu_index: The index of the GPU that will be used by the subprocess.
            Example: 0
        numa_options: See NumaOptions for details.

    Returns:
        Set of logical CPU indices to bind to.
    """
    if numa_options.affinity_mode == AffinityMode.NODE:
        logical_cpus = _node_get_logical_cpus_to_bind_to(gpu_index=gpu_index)
    elif numa_options.affinity_mode == AffinityMode.SOCKET:
        logical_cpus = _socket_get_logical_cpus_to_bind_to(gpu_index=gpu_index)
    elif numa_options.affinity_mode == AffinityMode.EXCLUSIVE:
        logical_cpus = _exclusive_get_logical_cpus_to_bind_to(gpu_index=gpu_index)
    elif numa_options.affinity_mode == AffinityMode.CORE_COMPLEX:
        logical_cpus = _core_complex_get_logical_cpus_to_bind_to(gpu_index=gpu_index)
    else:
        raise ValueError(f"Affinity mode {numa_options.affinity_mode} not supported.")

    return logical_cpus
````
- **EN**: This chunk defines `_get_logical_cpus_to_bind_to`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_logical_cpus_to_bind_to`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 347-368 / 第 347-368 行
````python
def _node_get_logical_cpus_to_bind_to(*, gpu_index: int) -> set[int]:
    """
    Core logic of 'node' numa strategy.
    """
    numa_node_index = _get_numa_node_index_for_gpu_index(gpu_index=gpu_index)

    return _get_allowed_logical_cpu_indices_for_numa_node(
        numa_node_index=numa_node_index
    )


def _socket_get_logical_cpus_to_bind_to(*, gpu_index: int) -> set[int]:
    """
    Core logic of 'socket' numa strategy.
    """
    numa_node_index_of_gpu = _get_numa_node_index_for_gpu_index(gpu_index=gpu_index)
    socket_index = _get_socket_index_for_numa_node(
        numa_node_index=numa_node_index_of_gpu
    )
    numa_node_indices = _get_numa_node_indices_for_socket_index(
        socket_index=socket_index
    )
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_socket_get_logical_cpus_to_bind_to`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_socket_get_logical_cpus_to_bind_to`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 370-393 / 第 370-393 行
````python
    logical_cpus = set()
    for numa_node_index in numa_node_indices:
        logical_cpus.update(
            _get_allowed_logical_cpu_indices_for_numa_node(
                numa_node_index=numa_node_index
            )
        )

    return logical_cpus


def _exclusive_get_logical_cpus_to_bind_to(*, gpu_index: int) -> set[int]:
    """
    Core logic of 'exclusive' numa strategy.
    """
    numa_node_index = _get_numa_node_index_for_gpu_index(gpu_index=gpu_index)

    gpu_indices = _get_gpu_indices_for_numa_node(numa_node_index=numa_node_index)
    gpu_indices = sorted(gpu_indices)
    original_gpu_relative_index = gpu_indices.index(gpu_index)

    allowed_logical_cpu_indices = _get_allowed_logical_cpu_indices_for_numa_node(
        numa_node_index=numa_node_index
    )
````
- **EN**: This chunk defines `_exclusive_get_logical_cpus_to_bind_to`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_exclusive_get_logical_cpus_to_bind_to`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 395-417 / 第 395-417 行
````python
    # Arbitrarily use the min logical cpu index on the physical core to
    # represent the physical core.
    physical_core_to_allowed_logical_cpu_indices = _group_by(
        allowed_logical_cpu_indices,
        lambda logical_cpu_index: min(
            _get_logical_cpu_indices_sharing_same_physical_core_as(
                logical_cpu_index=logical_cpu_index
            )
        ),
    )
    # Sort the dict for consistency (dicts maintain order in Python)
    physical_core_to_allowed_logical_cpu_indices = dict(
        sorted(physical_core_to_allowed_logical_cpu_indices.items())
    )

    num_physical_cores_per_gpu = len(
        physical_core_to_allowed_logical_cpu_indices
    ) // len(gpu_indices)
    # Often, the number of physical cores will not be perfectly divisible by the number
    # of GPUs. In those cases, give the lowest GPU indices an extra core
    num_gpus_to_give_one_extra_physical_core = len(
        physical_core_to_allowed_logical_cpu_indices
    ) % len(gpu_indices)
````
- **EN**: This chunk continues `_exclusive_get_logical_cpus_to_bind_to` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `_exclusive_get_logical_cpus_to_bind_to`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 419-437 / 第 419-437 行
````python
    if num_physical_cores_per_gpu < 1:
        raise RuntimeError(
            f"There are only {len(physical_core_to_allowed_logical_cpu_indices)} physical cores on {numa_node_index=},"
            + f" but there are {len(gpu_indices)} GPUs associated with this NUMA node."
        )

    # Compute slice indices for this GPU
    start = original_gpu_relative_index * num_physical_cores_per_gpu + min(
        original_gpu_relative_index, num_gpus_to_give_one_extra_physical_core
    )
    end = (
        start
        + num_physical_cores_per_gpu
        + (
            1
            if original_gpu_relative_index < num_gpus_to_give_one_extra_physical_core
            else 0
        )
    )
````
- **EN**: This chunk continues `_exclusive_get_logical_cpus_to_bind_to` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `_exclusive_get_logical_cpus_to_bind_to`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 439-462 / 第 439-462 行
````python
    # Slice and flatten the logical CPUs from the selected physical cores
    logical_cpu_indices_for_original_gpu = {
        logical_cpu_index
        for logical_cpu_indices in list(
            physical_core_to_allowed_logical_cpu_indices.values()
        )[start:end]
        for logical_cpu_index in logical_cpu_indices
    }

    return logical_cpu_indices_for_original_gpu


def _core_complex_get_logical_cpus_to_bind_to(*, gpu_index: int) -> set[int]:
    """
    Core logic of 'core-complex' numa strategy.

    Each GPU is assigned a full core complex (group of cores sharing L3 cache)
    within its affined NUMA node.
    """
    numa_node_index = _get_numa_node_index_for_gpu_index(gpu_index=gpu_index)

    gpu_indices = _get_gpu_indices_for_numa_node(numa_node_index=numa_node_index)
    gpu_indices = sorted(gpu_indices)
    original_gpu_relative_index = gpu_indices.index(gpu_index)
````
- **EN**: This chunk defines `_core_complex_get_logical_cpus_to_bind_to`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_core_complex_get_logical_cpus_to_bind_to`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 464-486 / 第 464-486 行
````python
    allowed_logical_cpu_indices = _get_allowed_logical_cpu_indices_for_numa_node(
        numa_node_index=numa_node_index
    )

    # Arbitrarily use the min logical cpu index on the max level cache
    # to represent the max level cache.
    max_level_cache_to_allowed_logical_cpu_indices = _group_by(
        allowed_logical_cpu_indices,
        lambda logical_cpu_index: min(
            _get_logical_cpus_sharing_same_max_level_cache_as(
                logical_cpu_index=logical_cpu_index
            )
        ),
    )

    max_level_cache_to_allowed_logical_cpu_indices = dict(
        sorted(
            max_level_cache_to_allowed_logical_cpu_indices.items(),
            # First, prioritize caches with more available cpus
            # Second, prioritize lower index cpus (just for clarity/consistency)
            key=lambda item: (-len(item[1]), item[0]),
        )
    )
````
- **EN**: This chunk continues `_core_complex_get_logical_cpus_to_bind_to` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `_core_complex_get_logical_cpus_to_bind_to`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 488-510 / 第 488-510 行
````python
    cache_index_for_original_gpu = original_gpu_relative_index % len(
        max_level_cache_to_allowed_logical_cpu_indices
    )
    logical_cpu_indices_for_original_gpu = list(
        max_level_cache_to_allowed_logical_cpu_indices.values()
    )[cache_index_for_original_gpu]

    return logical_cpu_indices_for_original_gpu


K = TypeVar("K")
V = TypeVar("V")


def _group_by(values: Iterable[V], get_key: Callable[[V], K]) -> dict[K, set[V]]:
    """
    Groups elements with same key into sets.
    """
    key_to_values: defaultdict[K, set[V]] = defaultdict(set)
    for value in values:
        key = get_key(value)
        key_to_values[key].add(value)
    return key_to_values
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. This chunk defines `_group_by`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 这一段定义了 `_group_by`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 513-535 / 第 513-535 行
````python
def _get_logical_cpu_indices_sharing_same_physical_core_as(
    *, logical_cpu_index: int
) -> set[int]:
    thread_siblings_list_absolute_path = (
        f"/sys/devices/system/cpu/cpu{logical_cpu_index}/topology/thread_siblings_list"
    )
    with open(thread_siblings_list_absolute_path) as f:
        return _get_set_of_int_from_ranges_str(f.read())


def _get_logical_cpus_sharing_same_max_level_cache_as(
    *, logical_cpu_index: int
) -> set[int]:
    cpu_cache_dir_absolute_path = (
        f"/sys/devices/system/cpu/cpu{logical_cpu_index}/cache"
    )

    max_level = -1
    logical_cpus_sharing_max_level_cache = set()
    for entry in os.listdir(cpu_cache_dir_absolute_path):
        if not entry.startswith("index") or not entry[5:].isdecimal():
            continue
        cache_index_absolute_path = os.path.join(cpu_cache_dir_absolute_path, entry)
````
- **EN**: This chunk defines `_get_logical_cpus_sharing_same_max_level_cache_as`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_logical_cpus_sharing_same_max_level_cache_as`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 537-558 / 第 537-558 行
````python
        # Filter out other cache types like Instruction
        type_absolute_path = os.path.join(cache_index_absolute_path, "type")
        with open(type_absolute_path) as type_file:
            if type_file.read().strip() not in {"Unified", "Data"}:
                continue

        level_absolute_path = os.path.join(cache_index_absolute_path, "level")
        with open(level_absolute_path) as level_file:
            level = int(level_file.read())
        if level <= max_level:
            continue

        max_level = level
        shared_cpu_list_absolute_path = os.path.join(
            cache_index_absolute_path, "shared_cpu_list"
        )
        with open(shared_cpu_list_absolute_path) as share_cpu_list_file:
            logical_cpus_sharing_max_level_cache = _get_set_of_int_from_ranges_str(
                share_cpu_list_file.read()
            )

    return logical_cpus_sharing_max_level_cache
````
- **EN**: This chunk continues `_get_logical_cpus_sharing_same_max_level_cache_as` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_get_logical_cpus_sharing_same_max_level_cache_as`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 561-585 / 第 561-585 行
````python
def _get_allowed_logical_cpu_indices_for_numa_node(*, numa_node_index: int) -> set[int]:
    all_cpu_indices = _get_cpu_indices_for_numa_node_MAYBE_NOT_ALLOWED(
        numa_node_index=numa_node_index
    )
    allowed_cpu_indices = _get_allowed_cpu_indices_for_current_thread()
    return all_cpu_indices & allowed_cpu_indices


def _get_cpu_indices_for_numa_node_MAYBE_NOT_ALLOWED(
    *, numa_node_index: int
) -> set[int]:
    """
    Returns:
        Indices of all CPUs associated with numa_node_index. However, the list
        is not filtered based on whether the thread is allowed to use them.
    """
    cpulist_absolute_path = f"/sys/devices/system/node/node{numa_node_index}/cpulist"
    try:
        with open(cpulist_absolute_path) as f:
            cpu_range_str = f.read()
    except FileNotFoundError as e:
        raise RuntimeError(
            f"Could not determine CPUs corresponding to {numa_node_index=}."
        ) from e
    return _get_set_of_int_from_ranges_str(cpu_range_str)
````
- **EN**: This chunk defines `_get_cpu_indices_for_numa_node_MAYBE_NOT_ALLOWED`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_cpu_indices_for_numa_node_MAYBE_NOT_ALLOWED`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 588-607 / 第 588-607 行
````python
def _get_gpu_count() -> int:
    return torch.cuda.device_count()


def _get_numa_node_index_for_gpu_index(*, gpu_index: int) -> int:
    device_properties = torch.cuda.get_device_properties(gpu_index)

    domain = device_properties.pci_domain_id  # type: ignore[attr-defined]
    bus = device_properties.pci_bus_id  # type: ignore[attr-defined]
    device = device_properties.pci_device_id  # type: ignore[attr-defined]

    # Format to sysfs PCI address: "0000:dc:00.0"
    pci_addr = f"{domain:04x}:{bus:02x}:{device:02x}.0"

    pci_numa_node_absolute_path = f"/sys/bus/pci/devices/{pci_addr}/numa_node"
    with open(pci_numa_node_absolute_path) as f:
        # In systems with only one NUMA node, this will
        # often be saved as -1. In those cases, there is obviously
        # at least one numa node, 0, so we use that.
        return max(int(f.read().strip()), 0)
````
- **EN**: This chunk defines `_get_numa_node_index_for_gpu_index`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_numa_node_index_for_gpu_index`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 610-634 / 第 610-634 行
````python
def _get_gpu_indices_for_numa_node(*, numa_node_index: int) -> set[int]:
    return {
        gpu_index
        for gpu_index in range(_get_gpu_count())
        if _get_numa_node_index_for_gpu_index(gpu_index=gpu_index) == numa_node_index
    }


def _get_socket_index_for_numa_node(*, numa_node_index: int) -> int:
    arbitrary_cpu_index = _get_arbitrary_allowed_cpu_index_for_numa_node(
        numa_node_index=numa_node_index
    )

    return _get_socket_index_for_cpu(cpu_index=arbitrary_cpu_index)


def _get_socket_index_for_cpu(*, cpu_index: int) -> int:
    package_id_absolute_path = (
        f"/sys/devices/system/cpu/cpu{cpu_index}/topology/physical_package_id"
    )
    try:
        with open(package_id_absolute_path) as f:
            return int(f.read().strip())
    except FileNotFoundError as e:
        raise RuntimeError(f"Could not determine socket for {cpu_index=}") from e
````
- **EN**: This chunk defines `_get_socket_index_for_cpu`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_socket_index_for_cpu`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 637-664 / 第 637-664 行
````python
def _get_arbitrary_allowed_cpu_index_for_numa_node(*, numa_node_index: int) -> int:
    return min(
        _get_allowed_logical_cpu_indices_for_numa_node(numa_node_index=numa_node_index)
    )


def _get_set_of_int_from_ranges_str(ranges_str: str) -> set[int]:
    """
    Util for parsing a string of int ranges, as in a sysfs file.

    Args:
        ranges_str: E.g., "0-2,4,6-7"

    Returns:
        E.g., {0, 1, 2, 4, 6, 7}
    """
    ints: set[int] = set()
    for range_str in ranges_str.split(","):
        range_str = range_str.strip()
        if not range_str:
            continue
        if "-" in range_str:
            start_str, end_str = range_str.split("-")
            start, end = int(start_str), int(end_str)
            ints.update(range(start, end + 1))
        else:
            ints.add(int(range_str))
    return ints
````
- **EN**: This chunk defines `_get_set_of_int_from_ranges_str`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_set_of_int_from_ranges_str`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 667-692 / 第 667-692 行
````python
def _get_ranges_str_from_ints(ints: Iterable[int]) -> str:
    """
    Convert a set of integers to a compact string with ranges.

    Args:
        ints: E.g., {0, 1, 2, 4, 6, 7}

    Returns:
        E.g., "0-2,4,6-7"
    """
    if not ints:
        return ""

    sorted_ints = sorted(ints)
    ranges = []
    start = prev = sorted_ints[0]

    for num in sorted_ints[1:]:
        if num == prev + 1:
            prev = num
        else:
            if start == prev:
                ranges.append(f"{start}")
            else:
                ranges.append(f"{start}-{prev}")
            start = prev = num
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_get_ranges_str_from_ints`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_get_ranges_str_from_ints`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 694-719 / 第 694-719 行
````python
    # Append the last range
    if start == prev:
        ranges.append(f"{start}")
    else:
        ranges.append(f"{start}-{prev}")

    return ",".join(ranges)


def _get_systemwide_numa_node_indices() -> set[int]:
    with open("/sys/devices/system/node/possible") as f:
        possible_nodes_str = f.read()

    return _get_set_of_int_from_ranges_str(possible_nodes_str)


def _get_numa_node_indices_for_socket_index(*, socket_index: int) -> set[int]:
    systemwide_numa_node_indices = _get_systemwide_numa_node_indices()

    matching_numa_node_indices = set()
    for numa_node_index in systemwide_numa_node_indices:
        arbitrary_cpu_index = _get_arbitrary_allowed_cpu_index_for_numa_node(
            numa_node_index=numa_node_index
        )
        if socket_index == _get_socket_index_for_cpu(cpu_index=arbitrary_cpu_index):
            matching_numa_node_indices.add(numa_node_index)
````
- **EN**: This chunk defines `_get_numa_node_indices_for_socket_index`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_numa_node_indices_for_socket_index`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 721-727 / 第 721-727 行
````python
    return matching_numa_node_indices


def _get_allowed_cpu_indices_for_current_thread() -> set[int]:
    # 0 denotes current thread
    # pyrefly: ignore [missing-attribute]
    return os.sched_getaffinity(0)  # type:ignore[attr-defined]
````
- **EN**: This chunk defines `_get_allowed_cpu_indices_for_current_thread`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_allowed_cpu_indices_for_current_thread`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **AffinityMode**
  - EN: `AffinityMode` is one of the main symbols declared or implemented in this file.
  - CN: `AffinityMode` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._utils_internal`
- **Standard library / 标准库**: `os`, `shutil`, `traceback`, `collections`, `collections.abc`, `dataclasses`, `enum`, `functools`, `logging`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `AffinityMode`, `NumaOptions`, `_maybe_wrap_command_args_with_numa_binding`, `_maybe_wrap_with_numa_binding`, `_maybe_apply_numa_binding_to_current_process`, `_assemble_numactl_command_args`, `_handle_exception`, `_get_validated_logical_cpus_to_bind_to`, `_raise_if_binding_invalid`
