# numa_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/numa_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `numa_utils` and the surrounding SGLang serving stack. / 提供围绕 `numa_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Module setup and shared state / 模块设置与共享状态
```python
import ctypes
import glob
import logging
import math
import multiprocessing
import os
import random
import shutil
import time
from contextlib import contextmanager
from pathlib import Path
from typing import Optional

import psutil
import torch

from sglang.srt.environ import envs
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import is_cuda

_is_cuda = is_cuda()

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `ctypes`, `glob`, `logging`, `math`, `multiprocessing`, `os`. It also defines symbols such as `_is_cuda`, `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `ctypes`, `glob`, `logging`, `math`, `multiprocessing`, `os`。 同时定义了 `_is_cuda`, `logger` 等符号，供后续逻辑使用。

### Lines 26-43: Function `configure_subprocess` / 函数 `configure_subprocess`
```python
@contextmanager
def configure_subprocess(server_args: ServerArgs, gpu_id: int):
    if envs.SGLANG_NUMA_BIND_V2.get():
        numa_node = get_numa_node_if_available(server_args, gpu_id)
        if numa_node is not None:
            numactl_args = f"--cpunodebind={numa_node} --membind={numa_node}"
            executable, debug_str = _create_numactl_executable(
                numactl_args=numactl_args
            )
            debug_str += (
                f", logical_gpu_id={gpu_id}, "
                f"physical_gpu_id={_get_nvml_device_index(gpu_id)}, "
                f"CUDA_VISIBLE_DEVICES={os.environ.get('CUDA_VISIBLE_DEVICES', '')}"
            )
            with _mp_set_executable(executable=executable, debug_str=debug_str):
                yield
                return
    yield
```
**EN:** This function implements `configure_subprocess`. It primarily calls `envs.SGLANG_NUMA_BIND_V2.get`, `get_numa_node_if_available`, `_create_numactl_executable`, `_mp_set_executable`, `_get_nvml_device_index`, `os.environ.get` to complete its work. State updates are written into `numa_node`, `numactl_args`, `debug_str`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `configure_subprocess`。 它主要通过调用 `envs.SGLANG_NUMA_BIND_V2.get`, `get_numa_node_if_available`, `_create_numactl_executable`, `_mp_set_executable`, `_get_nvml_device_index`, `os.environ.get` 来完成任务。 状态更新主要写入 `numa_node`, `numactl_args`, `debug_str`。 实现中使用了条件分支、上下文管理资源。

### Lines 46-55: Function `_create_numactl_executable` / 函数 `_create_numactl_executable`
```python
def _create_numactl_executable(numactl_args: str):
    old_executable = os.fsdecode(multiprocessing.spawn.get_executable())
    script = f'''#!/bin/sh
exec numactl {numactl_args} {old_executable} "$@"'''
    path = Path(
        f"/tmp/sglang_temp_file_{time.time()}_{random.randrange(0, 10000000)}.sh"
    )
    path.write_text(script)
    path.chmod(0o777)
    return str(path), f"{script=}"
```
**EN:** This function implements `_create_numactl_executable`. It primarily calls `os.fsdecode`, `Path`, `path.write_text`, `path.chmod`, `multiprocessing.spawn.get_executable`, `str` to complete its work. State updates are written into `old_executable`, `script`, `path`.
**CN:** 该函数实现了 `_create_numactl_executable`。 它主要通过调用 `os.fsdecode`, `Path`, `path.write_text`, `path.chmod`, `multiprocessing.spawn.get_executable`, `str` 来完成任务。 状态更新主要写入 `old_executable`, `script`, `path`。

### Lines 58-73: Function `_mp_set_executable` / 函数 `_mp_set_executable`
```python
@contextmanager
def _mp_set_executable(executable: str, debug_str: str):
    start_method = multiprocessing.get_start_method()
    assert start_method == "spawn", f"{start_method=}"

    old_executable = os.fsdecode(multiprocessing.spawn.get_executable())
    multiprocessing.spawn.set_executable(executable)
    logger.debug(f"mp.set_executable {old_executable} -> {executable} ({debug_str})")
    try:
        yield
    finally:
        assert (
            os.fsdecode(multiprocessing.spawn.get_executable()) == executable
        ), f"{multiprocessing.spawn.get_executable()=}"
        multiprocessing.spawn.set_executable(old_executable)
        logger.debug(f"mp.set_executable revert to {old_executable}")
```
**EN:** This function implements `_mp_set_executable`. It primarily calls `multiprocessing.get_start_method`, `os.fsdecode`, `multiprocessing.spawn.set_executable`, `logger.debug`, `multiprocessing.spawn.get_executable` to complete its work. State updates are written into `start_method`, `old_executable`. The implementation relies on error handling.
**CN:** 该函数实现了 `_mp_set_executable`。 它主要通过调用 `multiprocessing.get_start_method`, `os.fsdecode`, `multiprocessing.spawn.set_executable`, `logger.debug`, `multiprocessing.spawn.get_executable` 来完成任务。 状态更新主要写入 `start_method`, `old_executable`。 实现中使用了错误处理。

### Lines 76-88: Function `_get_nvml_device_index` / 函数 `_get_nvml_device_index`
```python
def _get_nvml_device_index(device_id: int) -> int:
    # _get_nvml_device_index is an internal PyTorch helper, so fall back to
    # device_id directly if the helper is unavailable.
    get_nvml_device_index = getattr(torch.cuda, "_get_nvml_device_index", None)
    if get_nvml_device_index is None:
        logger.warning(
            "torch.cuda._get_nvml_device_index is unavailable; falling back to "
            f"device_id={device_id} as the NVML device index. This may select "
            "the wrong physical GPU when CUDA_VISIBLE_DEVICES reorders devices "
            f"(CUDA_VISIBLE_DEVICES={os.environ.get('CUDA_VISIBLE_DEVICES', '')})."
        )
        return device_id
    return get_nvml_device_index(device_id)
```
**EN:** This function implements `_get_nvml_device_index`. It primarily calls `getattr`, `get_nvml_device_index`, `logger.warning`, `os.environ.get` to complete its work. State updates are written into `get_nvml_device_index`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_get_nvml_device_index`。 它主要通过调用 `getattr`, `get_nvml_device_index`, `logger.warning`, `os.environ.get` 来完成任务。 状态更新主要写入 `get_nvml_device_index`。 实现中使用了条件分支。

### Lines 91-116: Function `get_numa_node_if_available` / 函数 `get_numa_node_if_available`
```python
def get_numa_node_if_available(server_args: ServerArgs, gpu_id: int) -> Optional[int]:
    """
    Returns the NUMA node for the given GPU id. If it is not set in the server_args, it will try to query the NUMA node for the GPU.
    If the NUMA node is not available, has already been configured externally, or the user lacks permission to set NUMA affinity, it will return None.

    Args:
        server_args: The server arguments.
        gpu_id: The GPU id.

    Returns:
        The NUMA node for the given GPU id or None if it is not available.
    """
    if server_args.numa_node is not None:
        return server_args.numa_node[gpu_id]
    if _is_numa_available():
        queried_numa_node = _query_numa_node_for_gpu(gpu_id)
        if len(queried_numa_node) == 0:
            return None
        if len(queried_numa_node) > 1:
            # get_numa_node_for_gpu could return multiple nodes, we use the first one for now.
            # I don't think there any hardware configs that would have more than one.
            logger.warning(
                f"Multiple NUMA nodes found for GPU {gpu_id}: {queried_numa_node}. Using the first one."
            )
        return queried_numa_node[0]
    return None
```
**EN:** This function implements `get_numa_node_if_available`. It primarily calls `_is_numa_available`, `_query_numa_node_for_gpu`, `len`, `logger.warning` to complete its work. State updates are written into `queried_numa_node`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_numa_node_if_available`。 它主要通过调用 `_is_numa_available`, `_query_numa_node_for_gpu`, `len`, `logger.warning` 来完成任务。 状态更新主要写入 `queried_numa_node`。 实现中使用了条件分支。

### Lines 119-130: Function `get_libnuma` / 函数 `get_libnuma`
```python
def get_libnuma():
    libnuma = None

    for libnuma_so in ["libnuma.so", "libnuma.so.1"]:
        try:
            libnuma = ctypes.CDLL(libnuma_so)
        except OSError as e:
            logger.debug(f"{e}")
            libnuma = None
        if libnuma is not None:
            break
    return libnuma
```
**EN:** This function implements `get_libnuma`. It primarily calls `ctypes.CDLL`, `logger.debug` to complete its work. State updates are written into `libnuma`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `get_libnuma`。 它主要通过调用 `ctypes.CDLL`, `logger.debug` 来完成任务。 状态更新主要写入 `libnuma`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 133-140: Function `numa_bind_to_node` / 函数 `numa_bind_to_node`
```python
def numa_bind_to_node(node: int):
    libnuma = get_libnuma()

    if libnuma is None or libnuma.numa_available() < 0:
        logger.warning("numa not available on this system, skip bind action")
    else:
        libnuma.numa_run_on_node(ctypes.c_int(node))
        libnuma.numa_set_preferred(ctypes.c_int(node))
```
**EN:** This function implements `numa_bind_to_node`. It primarily calls `get_libnuma`, `logger.warning`, `libnuma.numa_run_on_node`, `libnuma.numa_set_preferred`, `libnuma.numa_available`, `ctypes.c_int` to complete its work. State updates are written into `libnuma`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `numa_bind_to_node`。 它主要通过调用 `get_libnuma`, `logger.warning`, `libnuma.numa_run_on_node`, `libnuma.numa_set_preferred`, `libnuma.numa_available`, `ctypes.c_int` 来完成任务。 状态更新主要写入 `libnuma`。 实现中使用了条件分支。

### Lines 143-155: Function `_can_set_mempolicy` / 函数 `_can_set_mempolicy`
```python
def _can_set_mempolicy() -> bool:
    """Check if the process has permission to use NUMA memory policy syscalls."""
    try:
        libnuma = get_libnuma()
        if libnuma is None or libnuma.numa_available() < 0:
            return False
        mode = ctypes.c_int()
        ret = libnuma.get_mempolicy(
            ctypes.byref(mode), None, ctypes.c_ulong(0), None, ctypes.c_ulong(0)
        )
        return ret == 0
    except Exception:
        return False
```
**EN:** This function implements `_can_set_mempolicy`. It primarily calls `get_libnuma`, `ctypes.c_int`, `libnuma.get_mempolicy`, `ctypes.byref`, `ctypes.c_ulong`, `libnuma.numa_available` to complete its work. State updates are written into `libnuma`, `mode`, `ret`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_can_set_mempolicy`。 它主要通过调用 `get_libnuma`, `ctypes.c_int`, `libnuma.get_mempolicy`, `ctypes.byref`, `ctypes.c_ulong`, `libnuma.numa_available` 来完成任务。 状态更新主要写入 `libnuma`, `mode`, `ret`。 实现中使用了条件分支、错误处理。

### Lines 158-193: Function `_is_numa_available` / 函数 `_is_numa_available`
```python
def _is_numa_available() -> bool:
    """
    Check if NUMA is available and not already configured externally.
    """
    if not _is_cuda:
        return False

    # Check if this is a numa system.
    if not os.path.isdir("/sys/devices/system/node/node1"):
        return False

    # Check if affinity is already constrained
    pid = os.getpid()
    process = psutil.Process(pid)
    cpu_affinity = process.cpu_affinity()
    all_cpus = list(range(psutil.cpu_count()))
    constrained_affinity = cpu_affinity != all_cpus
    if constrained_affinity:
        logger.warning(
            "NUMA affinity is already constrained for process, skipping NUMA node configuration for GPU. Remove your constraints to allow automatic configuration."
        )
        return False

    if not shutil.which("numactl") and envs.SGLANG_NUMA_BIND_V2.get():
        logger.debug(
            "numactl command not found, skipping NUMA node configuration for GPU. Install numactl (e.g., apt-get install numactl) to enable automatic NUMA binding."
        )
        return False

    if not _can_set_mempolicy():
        logger.warning(
            "User lacks permission to set NUMA affinity, skipping NUMA node configuration for GPU. If using docker, try adding --cap-add SYS_NICE to your docker run command."
        )
        return False

    return True
```
**EN:** This function implements `_is_numa_available`. It primarily calls `os.getpid`, `psutil.Process`, `process.cpu_affinity`, `list`, `os.path.isdir`, `range` to complete its work. State updates are written into `pid`, `process`, `cpu_affinity`, `all_cpus`, `constrained_affinity`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_is_numa_available`。 它主要通过调用 `os.getpid`, `psutil.Process`, `process.cpu_affinity`, `list`, `os.path.isdir`, `range` 来完成任务。 状态更新主要写入 `pid`, `process`, `cpu_affinity`, `all_cpus`, `constrained_affinity`。 实现中使用了条件分支。

### Lines 196-246: Function `_query_numa_node_for_gpu` / 函数 `_query_numa_node_for_gpu`
```python
def _query_numa_node_for_gpu(device_id: int):
    """
    Get the NUMA node affinity list for a GPU device.

    Args:
        device_id: CUDA logical device index (post-CUDA_VISIBLE_DEVICES).
    Returns:
        List of NUMA node IDs that have affinity with the device.
    """
    try:
        import pynvml
    except ModuleNotFoundError:
        logger.warning("pynvml not installed, skipping NUMA node configuration for GPU")
        return []

    try:
        pynvml.nvmlInit()

        # device_id is a CUDA logical index. Convert it to the corresponding
        # NVML index so reordered CUDA_VISIBLE_DEVICES maps to the right GPU.
        # _get_nvml_device_index takes CUDA_VISIBLE_DEVICES into account.
        nvml_device_id = _get_nvml_device_index(device_id)
        handle = pynvml.nvmlDeviceGetHandleByIndex(nvml_device_id)
        numa_node_count = len(glob.glob("/sys/devices/system/node/node[0-9]*"))

        c_ulong_bits = ctypes.sizeof(ctypes.c_ulong) * 8
        node_set_size = max(1, math.ceil(numa_node_count / c_ulong_bits))
        node_set = pynvml.nvmlDeviceGetMemoryAffinity(
# ... omitted for brevity ...
        return []
    finally:
        try:
            pynvml.nvmlShutdown()
        except Exception:
            pass  # Ignore shutdown errors
```
**EN:** This function implements `_query_numa_node_for_gpu`. It primarily calls `pynvml.nvmlInit`, `_get_nvml_device_index`, `pynvml.nvmlDeviceGetHandleByIndex`, `len`, `max`, `pynvml.nvmlDeviceGetMemoryAffinity` to complete its work. State updates are written into `nvml_device_id`, `handle`, `numa_node_count`, `c_ulong_bits`, `node_set_size`, `node_set`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_query_numa_node_for_gpu`。 它主要通过调用 `pynvml.nvmlInit`, `_get_nvml_device_index`, `pynvml.nvmlDeviceGetHandleByIndex`, `len`, `max`, `pynvml.nvmlDeviceGetMemoryAffinity` 来完成任务。 状态更新主要写入 `nvml_device_id`, `handle`, `numa_node_count`, `c_ulong_bits`, `node_set_size`, `node_set`。 实现中使用了条件分支、迭代逻辑、错误处理。

## Key Concepts / 关键概念
- **Functions / 函数**: `configure_subprocess`, `_create_numactl_executable`, `_mp_set_executable`, `_get_nvml_device_index`, `get_numa_node_if_available`, `get_libnuma`, `numa_bind_to_node`, `_can_set_mempolicy`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.server_args`, `sglang.srt.utils`
- **External / 外部依赖**: `psutil`, `torch`, `pynvml`
- **Standard library / 标准库**: `ctypes`, `glob`, `logging`, `math`, `multiprocessing`, `os`, `random`, `shutil`, `time`, `contextlib`, `pathlib`, `typing`
