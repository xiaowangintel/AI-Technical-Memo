# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/mps/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
r"""
This package enables an interface for accessing MPS (Metal Performance Shaders) backend in Python.
Metal is Apple's API for programming metal GPU (graphics processor unit). Using MPS means that increased
performance can be achieved, by running work on the metal GPU(s).
See https://developer.apple.com/documentation/metalperformanceshaders for more details.
"""

import torch
from torch import Tensor


_is_in_bad_fork = getattr(torch._C, "_mps_is_in_bad_fork", lambda: False)
_default_mps_generator: torch._C.Generator = None  # type: ignore[assignment]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 17-32 / 第 17-32 行
````python
# local helper function (not public or exported)
def _get_default_mps_generator() -> torch._C.Generator:
    global _default_mps_generator
    if _default_mps_generator is None:
        _default_mps_generator = torch._C._mps_get_default_generator()
    return _default_mps_generator


def device_count() -> int:
    r"""Returns the number of available MPS devices."""
    return int(torch._C._has_mps and torch._C._mps_is_available())


def synchronize() -> None:
    r"""Waits for all kernels in all streams on a MPS device to complete."""
    return torch._C._mps_deviceSynchronize()
````
- **EN**: This chunk defines `synchronize`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `synchronize`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 35-54 / 第 35-54 行
````python
def get_rng_state(device: int | str | torch.device = "mps") -> Tensor:
    r"""Returns the random number generator state as a ByteTensor.

    Args:
        device (torch.device or int, optional): The device to return the RNG state of.
            Default: ``'mps'`` (i.e., ``torch.device('mps')``, the current MPS device).
    """
    return _get_default_mps_generator().get_state()


def set_rng_state(new_state: Tensor, device: int | str | torch.device = "mps") -> None:
    r"""Sets the random number generator state.

    Args:
        new_state (torch.ByteTensor): The desired state
        device (torch.device or int, optional): The device to set the RNG state.
            Default: ``'mps'`` (i.e., ``torch.device('mps')``, the current MPS device).
    """
    new_state_copy = new_state.clone(memory_format=torch.contiguous_format)
    _get_default_mps_generator().set_state(new_state_copy)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `set_rng_state`, which mutates configuration or backend state that affects later execution. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `set_rng_state`，其作用是修改会影响后续执行的配置或后端状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-75 / 第 57-75 行
````python
def manual_seed(seed: int) -> None:
    r"""Sets the seed for generating random numbers.

    Args:
        seed (int): The desired seed.
    """
    # the torch.mps.manual_seed() can be called from the global
    # torch.manual_seed() in torch/random.py. So we need to make
    # sure mps is available (otherwise we just return without
    # erroring out)
    if not torch._C._has_mps:
        return
    seed = int(seed)
    _get_default_mps_generator().manual_seed(seed)


def seed() -> None:
    r"""Sets the seed for generating random numbers to a random number."""
    _get_default_mps_generator().seed()
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `seed`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `seed`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 78-93 / 第 78-93 行
````python
def empty_cache() -> None:
    r"""Releases all unoccupied cached memory currently held by the caching
    allocator so that those can be used in other GPU applications.
    """
    torch._C._mps_emptyCache()


def set_per_process_memory_fraction(fraction) -> None:
    r"""Set memory fraction for limiting process's memory allocation on MPS device.
    The allowed value equals the fraction multiplied by recommended maximum device memory
    (obtained from Metal API device.recommendedMaxWorkingSetSize).
    If trying to allocate more than the allowed value in a process, it will raise an out of
    memory error in allocator.

    Args:
        fraction(float): Range: 0~2. Allowed memory equals total_memory * fraction.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `set_per_process_memory_fraction`, which mutates configuration or backend state that affects later execution. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `set_per_process_memory_fraction`，其作用是修改会影响后续执行的配置或后端状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 95-111 / 第 95-111 行
````python
    .. note::
       Passing 0 to fraction means unlimited allocations
       (may cause system failure if out of memory).
       Passing fraction greater than 1.0 allows limits beyond the value
       returned from device.recommendedMaxWorkingSetSize.
    """

    if not isinstance(fraction, float):
        raise TypeError("Invalid type for fraction argument, must be `float`")
    if fraction < 0 or fraction > 2:
        raise ValueError(f"Invalid fraction value: {fraction}. Allowed range: 0~2")

    torch._C._mps_setMemoryFraction(fraction)


def current_allocated_memory() -> int:
    r"""Returns the current GPU memory occupied by tensors in bytes.
````
- **EN**: This chunk defines `current_allocated_memory`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `current_allocated_memory`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 113-131 / 第 113-131 行
````python
    .. note::
       The returned size does not include cached allocations in
       memory pools of MPSAllocator.
    """
    return torch._C._mps_currentAllocatedMemory()


def driver_allocated_memory() -> int:
    r"""Returns total GPU memory allocated by Metal driver for the process in bytes.

    .. note::
       The returned size includes cached allocations in MPSAllocator pools
       as well as allocations from MPS/MPSGraph frameworks.
    """
    return torch._C._mps_driverAllocatedMemory()


def recommended_max_memory() -> int:
    r"""Returns recommended max Working set size for GPU memory in bytes.
````
- **EN**: This chunk defines `recommended_max_memory`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `recommended_max_memory`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 133-152 / 第 133-152 行
````python
    .. note::
       Recommended max working set size for Metal.
       returned from device.recommendedMaxWorkingSetSize.
    """
    return torch._C._mps_recommendedMaxMemory()


def compile_shader(source: str):
    r"""Compiles compute shader from source and allows one to invoke kernels
    defined there from the comfort of Python runtime
    Example::

        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_MPS)
        >>> lib = torch.mps.compile_shader(
        ... "kernel void full(device float* out, constant float& val, uint idx [[thread_position_in_grid]]) { out[idx] = val; }"
        ...  )
        >>> x = torch.zeros(16, device="mps")
        >>> lib.full(x, 3.14)
    """
    from pathlib import Path
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as pathlib. This chunk defines `compile_shader`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 pathlib。 这一段定义了 `compile_shader`，其作用是协调 tracing、捕获或编译所需的图相关状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 154-172 / 第 154-172 行
````python
    from torch.utils._cpp_embed_headers import _embed_headers

    if not hasattr(torch._C, "_mps_compileShader"):
        raise RuntimeError("MPS is not available")
    source = _embed_headers(
        [l + "\n" for l in source.split("\n")],
        [Path(__file__).parent.parent / "include"],
        set(),
    )
    return torch._C._mps_compileShader(source)


def load_metallib(source):
    r"""Loads a precompiled Metal library (.metallib) and returns a shader
    library object that allows invoking kernels defined in it.

    Args:
        source: Either raw metallib bytes (``bytes``/``bytearray``) or a
            filesystem path (``str``/``os.PathLike``) to a ``.metallib`` file.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.utils._cpp_embed_headers. This chunk defines `load_metallib`, which serializes or reconstructs state across a Python-visible boundary. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.utils._cpp_embed_headers。 这一段定义了 `load_metallib`，其作用是在 Python 可见边界上序列化或重建状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 174-193 / 第 174-193 行
````python
    This is useful for loading Metal libraries compiled ahead of time or
    generated by external tools (e.g. Triton, MetalASM).

    Example::

        >>> # xdoctest: +SKIP("requires external .metallib file")
        >>> lib = torch.mps.load_metallib("kernels.metallib")
        >>> x = torch.ones(16, device="mps")
        >>> lib.square(x)
    """
    import os

    if isinstance(source, (bytes, bytearray)):
        if not hasattr(torch._C, "_mps_loadMetalllib"):
            raise RuntimeError("MPS is not available")
        return torch._C._mps_loadMetalllib(bytes(source))
    elif isinstance(source, (str, os.PathLike)):
        if not hasattr(torch._C, "_mps_loadMetallibFromPath"):
            raise RuntimeError("MPS is not available")
        return torch._C._mps_loadMetallibFromPath(str(source))
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as os. This chunk continues `load_metallib` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 os。 这一段延续了 `load_metallib`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 194-213 / 第 194-213 行
````python
    else:
        raise TypeError(f"expected bytes or path, got {type(source).__name__}")


def is_available() -> bool:
    return device_count() > 0


from . import profiler
from .event import Event


__all__ = [
    "compile_shader",
    "load_metallib",
    "device_count",
    "get_rng_state",
    "manual_seed",
    "seed",
    "set_rng_state",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as ., .event. `__all__` defines the public symbols that this module chooses to export. This chunk defines `is_available`, which checks a capability or invariant before later code relies on it. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 .、.event。 `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `is_available`，其作用是检查某项能力或不变量，供后续逻辑依赖。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 214-223 / 第 214-223 行
````python
    "synchronize",
    "empty_cache",
    "set_per_process_memory_fraction",
    "current_allocated_memory",
    "driver_allocated_memory",
    "Event",
    "profiler",
    "recommended_max_memory",
    "is_available",
]
````
- **EN**: This chunk continues `is_available` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `is_available`，进一步展开其内部控制流或状态更新。

## Key Concepts / 关键概念

- **MPS runtime**
  - EN: Wraps Apple Metal Performance Shaders integration points and device-facing helpers.
  - CN: 封装 Apple Metal Performance Shaders 集成点及面向设备的辅助逻辑。
- **_get_default_mps_generator**
  - EN: `_get_default_mps_generator` is one of the main symbols declared or implemented in this file.
  - CN: `_get_default_mps_generator` 是本文件声明或实现的主要符号之一。
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

- **Internal torch modules / torch 内部模块**: `torch`, `torch.utils._cpp_embed_headers`, `.`, `.event`
- **Standard library / 标准库**: `pathlib`, `os`
- **Primary symbols in this file / 本文件核心符号**: `_get_default_mps_generator`, `device_count`, `synchronize`, `get_rng_state`, `set_rng_state`, `manual_seed`, `seed`, `empty_cache`, `set_per_process_memory_fraction`, `current_allocated_memory`
