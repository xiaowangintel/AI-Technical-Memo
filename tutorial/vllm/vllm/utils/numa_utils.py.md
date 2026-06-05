# numa_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/numa_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: NUMA binding utilities for vLLM worker processes / 该模块围绕 `numa_utils` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-28)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""NUMA binding utilities for vLLM worker processes.

Adapted in part from SGLang's NUMA helper implementation:
https://github.com/sgl-project/sglang/blob/ba6d54d0f08f82f42b8224908ae2459a496b31b3/python/sglang/srt/utils/numa_utils.py
"""

import ctypes
import logging
import multiprocessing
import os
import subprocess
from contextlib import contextmanager
from functools import cache
from pathlib import Path
from typing import TYPE_CHECKING

import psutil

from vllm import envs

if TYPE_CHECKING:
    from vllm.config import VllmConfig

logger = logging.getLogger(__name__)
_NUMACTL_ARGS_ENV = "_VLLM_INTERNAL_NUMACTL_ARGS"
_NUMACTL_PYTHON_EXECUTABLE_ENV = "_VLLM_INTERNAL_NUMACTL_PYTHON_EXECUTABLE"
```
**EN:** Sets up the module with standard-library support such as `ctypes`, `logging`, `multiprocessing`, external packages such as `psutil`, vLLM modules such as `vllm`, `vllm.config`, `vllm.platforms`. It prepares the symbols later used by `get_libnuma`, `_can_set_mempolicy`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm`, `vllm.config`, `vllm.platforms` 等 vLLM 内部依赖。 这些准备工作为后续的 `get_libnuma`, `_can_set_mempolicy` 提供上下文。

### get_libnuma (lines 32-41)
```python
def get_libnuma():
    libnuma = None
    for libnuma_so in ["libnuma.so", "libnuma.so.1"]:
        try:
            libnuma = ctypes.CDLL(libnuma_so)
        except OSError:
            libnuma = None
        if libnuma is not None:
            break
    return libnuma
```
**EN:** `get_libnuma` retrieves data or state needed by the pipeline. Inside the body, it relies on `ctypes.CDLL` to complete the main steps.
**CN:** `get_libnuma` 负责获取流水线所需的数据或状态。 实现过程中会调用 `ctypes.CDLL` 等函数完成关键步骤。

### _can_set_mempolicy (lines 44-56)
```python
def _can_set_mempolicy() -> bool:
    """Check whether the current process can use NUMA memory policy syscalls."""
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
**EN:** `_can_set_mempolicy`: Check whether the current process can use NUMA memory policy syscalls. Inside the body, it relies on `get_libnuma`, `ctypes.c_int`, `libnuma.get_mempolicy` to complete the main steps.
**CN:** `_can_set_mempolicy` 负责检查某个操作是否受支持。 实现过程中会调用 `get_libnuma`, `ctypes.c_int`, `libnuma.get_mempolicy` 等函数完成关键步骤。

### _is_auto_numa_available (lines 59-98)
```python
def _is_auto_numa_available() -> bool:
    """Check whether automatic GPU-to-NUMA detection should be attempted."""
    from vllm.platforms import current_platform

    if not current_platform.is_cuda_alike():
        return False

    if not os.path.isdir("/sys/devices/system/node/node1"):
        return False

    try:
        process = psutil.Process(os.getpid())
        cpu_affinity = process.cpu_affinity()
        cpu_count = psutil.cpu_count()
        if cpu_count is not None and cpu_affinity != list(range(cpu_count)):
            logger.warning(
                "CPU affinity is already constrained for this process. "
                "Skipping automatic NUMA binding; pass --numa-bind-nodes "
                "explicitly to override."
            )
            return False
    except (AttributeError, NotImplementedError, psutil.Error):
        pass

    # ...
            "Platform %s does not support automatic NUMA detection",
            type(current_platform).__name__,
        )
        return False

    return True
```
**EN:** `_is_auto_numa_available`: Check whether automatic GPU-to-NUMA detection should be attempted. Inside the body, it relies on `current_platform.is_cuda_alike`, `os.path.isdir`, `psutil.Process` to complete the main steps.
**CN:** `_is_auto_numa_available` 负责检查条件并返回布尔结果。 实现过程中会调用 `current_platform.is_cuda_alike`, `os.path.isdir`, `psutil.Process` 等函数完成关键步骤。

### get_auto_numa_nodes (lines 102-112)
```python
def get_auto_numa_nodes() -> list[int] | None:
    """Auto-detect NUMA nodes for all visible GPUs."""
    from vllm.platforms import current_platform

    if not _is_auto_numa_available():
        return None

    numa_nodes = current_platform.get_all_device_numa_nodes()
    if numa_nodes is not None:
        logger.info("Auto-detected NUMA nodes for GPUs: %s", numa_nodes)
    return numa_nodes
```
**EN:** `get_auto_numa_nodes`: Auto-detect NUMA nodes for all visible GPUs. Inside the body, it relies on `current_platform.get_all_device_numa_nodes`, `_is_auto_numa_available`, `logger.info` to complete the main steps.
**CN:** `get_auto_numa_nodes` 负责获取流水线所需的数据或状态。 实现过程中会调用 `current_platform.get_all_device_numa_nodes`, `_is_auto_numa_available`, `logger.info` 等函数完成关键步骤。

### _get_gpu_index (lines 115-135)
```python
def _get_gpu_index(
    parallel_config, local_rank: int, dp_local_rank: int | None = None
) -> int:
    """Compute the physical GPU index used for NUMA lookup."""
    if (
        parallel_config.distributed_executor_backend not in ("ray", "external_launcher")
        and parallel_config.data_parallel_backend != "ray"
        and parallel_config.nnodes_within_dp == 1
    ):
        if dp_local_rank is None:
            dp_local_rank = parallel_config.data_parallel_rank_local
            if dp_local_rank is None:
                dp_local_rank = parallel_config.data_parallel_index

        tp_pp_world_size = (
            parallel_config.pipeline_parallel_size
            * parallel_config.tensor_parallel_size
        )
        return local_rank + dp_local_rank * tp_pp_world_size

    return local_rank
```
**EN:** `_get_gpu_index`: Compute the physical GPU index used for NUMA lookup. It mainly works with `parallel_config`, `local_rank`, `dp_local_rank`.
**CN:** `_get_gpu_index` 负责获取流水线所需的数据或状态。 它主要处理 `parallel_config`, `local_rank`, `dp_local_rank` 等参数。

### _get_numa_node (lines 138-156)
```python
def _get_numa_node(parallel_config, gpu_index: int) -> int:
    numa_nodes = parallel_config.numa_bind_nodes
    if numa_nodes is None:
        numa_nodes = get_auto_numa_nodes()
        if numa_nodes is None:
            raise RuntimeError(
                "NUMA binding was requested, but vLLM could not detect the "
                "GPU-to-NUMA topology automatically. Pass --numa-bind-nodes "
                "explicitly or disable --numa-bind."
            )
        parallel_config.numa_bind_nodes = numa_nodes

    if gpu_index >= len(numa_nodes):
        raise ValueError(
            f"GPU index {gpu_index} exceeds numa_bind_nodes size {len(numa_nodes)}. "
            "Ensure the binding lists cover every visible GPU."
        )

    return numa_nodes[gpu_index]
```
**EN:** `_get_numa_node` retrieves data or state needed by the pipeline. It mainly works with `parallel_config`, `gpu_index`. Inside the body, it relies on `get_auto_numa_nodes`, `ValueError`, `RuntimeError` to complete the main steps.
**CN:** `_get_numa_node` 负责获取流水线所需的数据或状态。 它主要处理 `parallel_config`, `gpu_index` 等参数。 实现过程中会调用 `get_auto_numa_nodes`, `ValueError`, `RuntimeError` 等函数完成关键步骤。

### _get_cpu_binding (lines 159-170)
```python
def _get_cpu_binding(parallel_config, gpu_index: int) -> str | None:
    cpu_bindings = parallel_config.numa_bind_cpus
    if cpu_bindings is None:
        return None

    if gpu_index >= len(cpu_bindings):
        raise ValueError(
            f"GPU index {gpu_index} exceeds numa_bind_cpus size "
            f"{len(cpu_bindings)}. Ensure the binding lists cover every visible GPU."
        )

    return cpu_bindings[gpu_index]
```
**EN:** `_get_cpu_binding` retrieves data or state needed by the pipeline. It mainly works with `parallel_config`, `gpu_index`. Inside the body, it relies on `ValueError` to complete the main steps.
**CN:** `_get_cpu_binding` 负责获取流水线所需的数据或状态。 它主要处理 `parallel_config`, `gpu_index` 等参数。 实现过程中会调用 `ValueError` 等函数完成关键步骤。

### _get_numactl_args (lines 173-207)
```python
def _get_numactl_args(
    vllm_config: "VllmConfig",
    local_rank: int,
    dp_local_rank: int | None = None,
    process_kind: str = "worker",
) -> str | None:
    parallel_config = vllm_config.parallel_config
    if not parallel_config.numa_bind:
        return None

    gpu_index = _get_gpu_index(parallel_config, local_rank, dp_local_rank)
    numa_node = _get_numa_node(parallel_config, gpu_index)
    cpu_binding = _get_cpu_binding(parallel_config, gpu_index)

    if cpu_binding is not None:
        bind_arg = f"--physcpubind={cpu_binding}"
        logger.info(
            "Binding %s subprocess (local_rank=%s, gpu_index=%s) to CPUs %s and NUMA node %s",  # noqa: E501
            process_kind,
            local_rank,
            gpu_index,
            cpu_binding,
            numa_node,
        )
    # ...
            local_rank,
            gpu_index,
            numa_node,
        )

    return f"{bind_arg} --membind={numa_node}"
```
**EN:** `_get_numactl_args` retrieves data or state needed by the pipeline. It mainly works with `vllm_config`, `local_rank`, `dp_local_rank`, `process_kind`. Inside the body, it relies on `_get_gpu_index`, `_get_numa_node`, `_get_cpu_binding` to complete the main steps.
**CN:** `_get_numactl_args` 负责获取流水线所需的数据或状态。 它主要处理 `vllm_config`, `local_rank`, `dp_local_rank`, `process_kind` 等参数。 实现过程中会调用 `_get_gpu_index`, `_get_numa_node`, `_get_cpu_binding` 等函数完成关键步骤。

### _log_numactl_show (lines 210-229)
```python
def _log_numactl_show(label: str) -> bool:
    try:
        result = subprocess.run(
            ["numactl", "--show"],
            check=True,
            capture_output=True,
            text=True,
        )
    except (FileNotFoundError, subprocess.CalledProcessError) as e:
        logger.warning("Failed to run `numactl --show` for %s: %s", label, e)
        return False

    output = result.stdout.strip()
    if not output:
        logger.warning("`numactl --show` returned no output for %s", label)
        return False

    summary = ", ".join(line.strip() for line in output.splitlines() if line.strip())
    logger.debug("%s affinity: %s", label, summary)
    return True
```
**EN:** `_log_numactl_show` implements helper logic used by this module. It mainly works with `label`. Inside the body, it relies on `result.stdout.strip`, `join`, `logger.debug` to complete the main steps.
**CN:** `_log_numactl_show` 负责实现本模块使用的辅助逻辑。 它主要处理 `label` 等参数。 实现过程中会调用 `result.stdout.strip`, `join`, `logger.debug` 等函数完成关键步骤。

### log_current_affinity_state (lines 232-234)
```python
def log_current_affinity_state(label: str) -> None:
    """Log the process's effective NUMA affinity state."""
    _log_numactl_show(label)
```
**EN:** `log_current_affinity_state`: Log the process's effective NUMA affinity state. It mainly works with `label`. Inside the body, it relies on `_log_numactl_show` to complete the main steps.
**CN:** `log_current_affinity_state` 负责实现本模块使用的辅助逻辑。 它主要处理 `label` 等参数。 实现过程中会调用 `_log_numactl_show` 等函数完成关键步骤。

### configure_subprocess (lines 238-258)
```python
def configure_subprocess(
    vllm_config: "VllmConfig",
    local_rank: int,
    dp_local_rank: int | None = None,
    process_kind: str = "worker",
):
    """Temporarily replace the multiprocessing executable with a numactl wrapper."""
    numactl_args = _get_numactl_args(
        vllm_config, local_rank, dp_local_rank, process_kind
    )
    if numactl_args is None:
        yield
        return

    executable, debug_str = _get_numactl_executable()
    python_executable = os.fsdecode(multiprocessing.spawn.get_executable())
    with (
        _set_numa_wrapper_env(numactl_args, python_executable),
        _mp_set_executable(executable, debug_str),
    ):
        yield
```
**EN:** `configure_subprocess`: Temporarily replace the multiprocessing executable with a numactl wrapper. It mainly works with `vllm_config`, `local_rank`, `dp_local_rank`, `process_kind`. Inside the body, it relies on `_get_numactl_args`, `_get_numactl_executable`, `os.fsdecode` to complete the main steps.
**CN:** `configure_subprocess` 负责实现本模块使用的辅助逻辑。 它主要处理 `vllm_config`, `local_rank`, `dp_local_rank`, `process_kind` 等参数。 实现过程中会调用 `_get_numactl_args`, `_get_numactl_executable`, `os.fsdecode` 等函数完成关键步骤。

### _get_numactl_executable (lines 261-272)
```python
def _get_numactl_executable() -> tuple[str, str]:
    """Return the fixed wrapper executable used to launch numactl."""
    from shutil import which

    if which("numactl") is None:
        raise RuntimeError(
            "numactl is required for NUMA binding but is not installed or "
            "not available on PATH."
        )

    script_path = Path(__file__).with_name("numa_wrapper.sh")
    return str(script_path), f"{script_path} via {_NUMACTL_ARGS_ENV}"
```
**EN:** `_get_numactl_executable`: Return the fixed wrapper executable used to launch numactl. Inside the body, it relies on `Path.with_name`, `which`, `RuntimeError` to complete the main steps.
**CN:** `_get_numactl_executable` 负责获取流水线所需的数据或状态。 实现过程中会调用 `Path.with_name`, `which`, `RuntimeError` 等函数完成关键步骤。

### _set_numa_wrapper_env (lines 276-292)
```python
def _set_numa_wrapper_env(numactl_args: str, python_executable: str):
    old_numactl_args = os.environ.get(_NUMACTL_ARGS_ENV)
    old_python_executable = os.environ.get(_NUMACTL_PYTHON_EXECUTABLE_ENV)
    os.environ[_NUMACTL_ARGS_ENV] = numactl_args
    os.environ[_NUMACTL_PYTHON_EXECUTABLE_ENV] = python_executable
    try:
        yield
    finally:
        if old_numactl_args is None:
            os.environ.pop(_NUMACTL_ARGS_ENV, None)
        else:
            os.environ[_NUMACTL_ARGS_ENV] = old_numactl_args

        if old_python_executable is None:
            os.environ.pop(_NUMACTL_PYTHON_EXECUTABLE_ENV, None)
        else:
            os.environ[_NUMACTL_PYTHON_EXECUTABLE_ENV] = old_python_executable
```
**EN:** `_set_numa_wrapper_env` writes values into an object or configuration structure. It mainly works with `numactl_args`, `python_executable`. Inside the body, it relies on `os.environ.get`, `os.environ.pop` to complete the main steps.
**CN:** `_set_numa_wrapper_env` 负责把值写入对象或配置结构。 它主要处理 `numactl_args`, `python_executable` 等参数。 实现过程中会调用 `os.environ.get`, `os.environ.pop` 等函数完成关键步骤。

### _mp_set_executable (lines 296-317)
```python
def _mp_set_executable(executable: str, debug_str: str):
    start_method = envs.VLLM_WORKER_MULTIPROC_METHOD
    if start_method != "spawn":
        logger.warning(
            "NUMA binding requires spawn method but got '%s'. "
            "NUMA binding will be ineffective. "
            "Set VLLM_WORKER_MULTIPROC_METHOD=spawn to enable NUMA binding.",
            start_method,
        )
        yield
        return

    old_executable = os.fsdecode(multiprocessing.spawn.get_executable())
    multiprocessing.spawn.set_executable(executable)
    try:
        yield
    finally:
        assert os.fsdecode(multiprocessing.spawn.get_executable()) == executable, (
            "Executable was changed during NUMA binding context: "
            f"expected {executable}, got {multiprocessing.spawn.get_executable()}"
        )
        multiprocessing.spawn.set_executable(old_executable)
```
**EN:** `_mp_set_executable` implements helper logic used by this module. It mainly works with `executable`, `debug_str`. Inside the body, it relies on `os.fsdecode`, `multiprocessing.spawn.set_executable`, `logger.warning` to complete the main steps.
**CN:** `_mp_set_executable` 负责实现本模块使用的辅助逻辑。 它主要处理 `executable`, `debug_str` 等参数。 实现过程中会调用 `os.fsdecode`, `multiprocessing.spawn.set_executable`, `logger.warning` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`get_libnuma`**: Key helper or entry point in this file. / **`get_libnuma`**：本文件中的关键辅助函数或入口。
- **`_can_set_mempolicy`**: Key helper or entry point in this file. / **`_can_set_mempolicy`**：本文件中的关键辅助函数或入口。
- **`_is_auto_numa_available`**: Key helper or entry point in this file. / **`_is_auto_numa_available`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: ctypes, logging, multiprocessing, os, subprocess, contextlib, functools, pathlib, typing, shutil
- **Third-party / 第三方**: psutil
- **Internal vLLM / vLLM 内部依赖**: vllm, vllm.config, vllm.platforms
