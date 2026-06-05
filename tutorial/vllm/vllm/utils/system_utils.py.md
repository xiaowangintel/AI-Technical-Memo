# system_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/system_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-28)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from __future__ import annotations

import contextlib
import multiprocessing
import os
import signal
import sys
from collections.abc import Callable, Iterator
from pathlib import Path
from typing import TextIO

import psutil

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.platforms.interface import in_wsl
from vllm.ray.lazy_utils import is_in_ray_actor

from .platform_utils import cuda_is_initialized, xpu_is_initialized

logger = init_logger(__name__)

CYAN = "\033[0;36m"
RESET = "\033[0;0m"
```
**EN:** Sets up the module with standard-library support such as `__future__`, `contextlib`, `multiprocessing`, external packages such as `psutil`, `ray`, `setproctitle`, vLLM modules such as `vllm.envs`, `vllm.logger`, `vllm.platforms`. It prepares the symbols later used by `update_environment_variables`, `set_env_var`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger`, `vllm.platforms` 等 vLLM 内部依赖。 这些准备工作为后续的 `update_environment_variables`, `set_env_var` 提供上下文。

### update_environment_variables (lines 34-44)
```python
def update_environment_variables(envs_dict: dict[str, str]):
    """Update multiple environment variables with logging."""
    for k, v in envs_dict.items():
        if k in os.environ and os.environ[k] != v:
            logger.warning(
                "Overwriting environment variable %s from '%s' to '%s'",
                k,
                os.environ[k],
                v,
            )
        os.environ[k] = v
```
**EN:** `update_environment_variables`: Update multiple environment variables with logging. It mainly works with `envs_dict`. Inside the body, it relies on `envs_dict.items`, `logger.warning` to complete the main steps.
**CN:** `update_environment_variables` 负责根据新输入更新模块状态。 它主要处理 `envs_dict` 等参数。 实现过程中会调用 `envs_dict.items`, `logger.warning` 等函数完成关键步骤。

### set_env_var (lines 48-58)
```python
def set_env_var(key: str, value: str) -> Iterator[None]:
    """Temporarily set an environment variable."""
    old = os.environ.get(key)
    os.environ[key] = value
    try:
        yield
    finally:
        if old is None:
            os.environ.pop(key, None)
        else:
            os.environ[key] = old
```
**EN:** `set_env_var`: Temporarily set an environment variable. It mainly works with `key`, `value`. Inside the body, it relies on `os.environ.get`, `os.environ.pop` to complete the main steps.
**CN:** `set_env_var` 负责把值写入对象或配置结构。 它主要处理 `key`, `value` 等参数。 实现过程中会调用 `os.environ.get`, `os.environ.pop` 等函数完成关键步骤。

### suppress_stdout (lines 62-91)
```python
def suppress_stdout():
    """
    Suppress stdout from C libraries at the file descriptor level.

    Only suppresses stdout, not stderr, to preserve error messages.
    Suppression is disabled when VLLM_LOGGING_LEVEL is set to DEBUG.

    Example:
        with suppress_stdout():
            # C library calls that would normally print to stdout
            torch.distributed.new_group(ranks, backend="gloo")
    """
    # Don't suppress if logging level is DEBUG
    if envs.VLLM_LOGGING_LEVEL == "DEBUG":
        yield
        return

    stdout_fd = sys.stdout.fileno()
    stdout_dup = os.dup(stdout_fd)
    devnull_fd = os.open(os.devnull, os.O_WRONLY)

    try:
        sys.stdout.flush()
        os.dup2(devnull_fd, stdout_fd)
        yield
    finally:
        sys.stdout.flush()
        os.dup2(stdout_dup, stdout_fd)
        os.close(stdout_dup)
        os.close(devnull_fd)
```
**EN:** `suppress_stdout`: Suppress stdout from C libraries at the file descriptor level. Inside the body, it relies on `sys.stdout.fileno`, `os.dup`, `os.open` to complete the main steps.
**CN:** `suppress_stdout` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `sys.stdout.fileno`, `os.dup`, `os.open` 等函数完成关键步骤。

### unique_filepath (lines 97-109)
```python
def unique_filepath(fn: Callable[[int], Path]) -> Path:
    """Generate a unique file path by trying incrementing integers.

    Note: This function has a TOCTOU race condition.
    Caller should use atomic operations (e.g., open with 'x' mode)
    when creating the file to ensure thread safety.
    """
    i = 0
    while True:
        p = fn(i)
        if not p.exists():
            return p
        i += 1
```
**EN:** `unique_filepath`: Generate a unique file path by trying incrementing integers. It mainly works with `fn`. Inside the body, it relies on `fn`, `p.exists` to complete the main steps.
**CN:** `unique_filepath` 负责实现本模块使用的辅助逻辑。 它主要处理 `fn` 等参数。 实现过程中会调用 `fn`, `p.exists` 等函数完成关键步骤。

### _sync_visible_devices_env_vars (lines 115-123)
```python
def _sync_visible_devices_env_vars():
    """Sync HIP/CUDA visibility env vars before spawning (ROCm only)."""

    if not current_platform.is_rocm():
        return

    from vllm.platforms.rocm import _sync_hip_cuda_env_vars

    _sync_hip_cuda_env_vars()
```
**EN:** `_sync_visible_devices_env_vars`: Sync HIP/CUDA visibility env vars before spawning (ROCm only). Inside the body, it relies on `_sync_hip_cuda_env_vars`, `current_platform.is_rocm` to complete the main steps.
**CN:** `_sync_visible_devices_env_vars` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `_sync_hip_cuda_env_vars`, `current_platform.is_rocm` 等函数完成关键步骤。

### _maybe_force_spawn (lines 126-165)
```python
def _maybe_force_spawn():
    """Check if we need to force the use of the `spawn` multiprocessing start
    method.
    """
    if os.environ.get("VLLM_WORKER_MULTIPROC_METHOD") == "spawn":
        return

    reasons = []
    if is_in_ray_actor():
        # even if we choose to spawn, we need to pass the ray address
        # to the subprocess so that it knows how to connect to the ray cluster.
        # env vars are inherited by subprocesses, even if we use spawn.
        import ray

        os.environ["RAY_ADDRESS"] = ray.get_runtime_context().gcs_address
        reasons.append("In a Ray actor and can only be spawned")

    # Force spawn if NUMA binding is enabled via --numa-bind.
    # NUMA binding uses executable hijacking which requires spawn
    if "--numa-bind" in sys.argv:
        reasons.append("NUMA binding requires spawn method")

    if cuda_is_initialized():
        reasons.append("CUDA is initialized")
    # ...
            "See https://docs.vllm.ai/en/latest/usage/"
            "troubleshooting.html#python-multiprocessing "
            "for more information. Reasons: %s",
            "; ".join(reasons),
        )
        os.environ["VLLM_WORKER_MULTIPROC_METHOD"] = "spawn"
```
**EN:** `_maybe_force_spawn`: Check if we need to force the use of the `spawn` multiprocessing start method. Inside the body, it relies on `is_in_ray_actor`, `cuda_is_initialized`, `in_wsl` to complete the main steps.
**CN:** `_maybe_force_spawn` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `is_in_ray_actor`, `cuda_is_initialized`, `in_wsl` 等函数完成关键步骤。

### get_mp_context (lines 168-181)
```python
def get_mp_context():
    """Get a multiprocessing context with a particular method (spawn or fork).
    By default we follow the value of the VLLM_WORKER_MULTIPROC_METHOD to
    determine the multiprocessing method (default is fork). However, under
    certain conditions, we may enforce spawn and override the value of
    VLLM_WORKER_MULTIPROC_METHOD.
    """
    _maybe_force_spawn()
    # (ROCm): Sync GPU visibility env vars so spawned children inherit
    # consistent values. Must run after _maybe_force_spawn and regardless
    # of whether spawn was already set.
    _sync_visible_devices_env_vars()
    mp_method = envs.VLLM_WORKER_MULTIPROC_METHOD
    return multiprocessing.get_context(mp_method)
```
**EN:** `get_mp_context`: Get a multiprocessing context with a particular method (spawn or fork). Inside the body, it relies on `_maybe_force_spawn`, `_sync_visible_devices_env_vars`, `multiprocessing.get_context` to complete the main steps.
**CN:** `get_mp_context` 负责获取流水线所需的数据或状态。 实现过程中会调用 `_maybe_force_spawn`, `_sync_visible_devices_env_vars`, `multiprocessing.get_context` 等函数完成关键步骤。

### set_process_title (lines 184-198)
```python
def set_process_title(
    name: str,
    suffix: str = "",
    prefix: str = envs.VLLM_PROCESS_NAME_PREFIX,
) -> None:
    """Set the current process title with optional suffix."""
    try:
        import setproctitle
    except ImportError:
        return

    if suffix:
        name = f"{name}_{suffix}"

    setproctitle.setproctitle(f"{prefix}::{name}")
```
**EN:** `set_process_title`: Set the current process title with optional suffix. It mainly works with `name`, `suffix`, `prefix`. Inside the body, it relies on `setproctitle.setproctitle` to complete the main steps.
**CN:** `set_process_title` 负责把值写入对象或配置结构。 它主要处理 `name`, `suffix`, `prefix` 等参数。 实现过程中会调用 `setproctitle.setproctitle` 等函数完成关键步骤。

### _add_prefix (lines 201-234)
```python
def _add_prefix(file: TextIO, worker_name: str, pid: int) -> None:
    """Add colored prefix to file output for log decoration."""
    is_tty = hasattr(file, "isatty") and file.isatty()
    if (
        envs.NO_COLOR
        or envs.VLLM_LOGGING_COLOR == "0"
        or (envs.VLLM_LOGGING_COLOR != "1" and not is_tty)
    ):
        prefix = f"({worker_name} pid={pid}) "
    else:
        prefix = f"{CYAN}({worker_name} pid={pid}){RESET} "
    # Use the original write to avoid nesting prefixes on repeated calls.
    file_write = getattr(file, "_original_write", file.write)

    def write_with_prefix(s: str):
        if not s:
            return
        if file.start_new_line:  # type: ignore[attr-defined]
            file_write(prefix)
        idx = 0
        while (next_idx := s.find("\n", idx)) != -1:
            next_idx += 1
            file_write(s[idx:next_idx])
            if next_idx == len(s):
    # ...
        file_write(s[idx:])
        file.start_new_line = False  # type: ignore[attr-defined]

    file.start_new_line = True  # type: ignore[attr-defined]
    file._original_write = file_write  # type: ignore[attr-defined]
    file.write = write_with_prefix  # type: ignore[method-assign]
```
**EN:** `_add_prefix`: Add colored prefix to file output for log decoration. It mainly works with `file`, `worker_name`, `pid`. Inside the body, it relies on `file.isatty`, `file_write`, `s.find` to complete the main steps.
**CN:** `_add_prefix` 负责实现本模块使用的辅助逻辑。 它主要处理 `file`, `worker_name`, `pid` 等参数。 实现过程中会调用 `file.isatty`, `file_write`, `s.find` 等函数完成关键步骤。

### decorate_logs (lines 237-248)
```python
def decorate_logs(process_name: str | None = None) -> None:
    """Decorate stdout/stderr with process name and PID prefix."""
    # Respect VLLM_CONFIGURE_LOGGING environment variable
    if not envs.VLLM_CONFIGURE_LOGGING:
        return

    if process_name is None:
        process_name = get_mp_context().current_process().name

    pid = os.getpid()
    _add_prefix(sys.stdout, process_name, pid)
    _add_prefix(sys.stderr, process_name, pid)
```
**EN:** `decorate_logs`: Decorate stdout/stderr with process name and PID prefix. It mainly works with `process_name`. Inside the body, it relies on `os.getpid`, `_add_prefix`, `get_mp_context.current_process` to complete the main steps.
**CN:** `decorate_logs` 负责实现本模块使用的辅助逻辑。 它主要处理 `process_name` 等参数。 实现过程中会调用 `os.getpid`, `_add_prefix`, `get_mp_context.current_process` 等函数完成关键步骤。

### kill_process_tree (lines 251-273)
```python
def kill_process_tree(pid: int):
    """
    Kills all descendant processes of the given pid by sending SIGKILL.

    Args:
        pid (int): Process ID of the parent process
    """
    try:
        parent = psutil.Process(pid)
    except psutil.NoSuchProcess:
        return

    # Get all children recursively
    children = parent.children(recursive=True)

    # Send SIGKILL to all children first
    for child in children:
        with contextlib.suppress(ProcessLookupError):
            os.kill(child.pid, signal.SIGKILL)

    # Finally kill the parent
    with contextlib.suppress(ProcessLookupError):
        os.kill(pid, signal.SIGKILL)
```
**EN:** `kill_process_tree`: Kills all descendant processes of the given pid by sending SIGKILL. It mainly works with `pid`. Inside the body, it relies on `parent.children`, `psutil.Process`, `contextlib.suppress` to complete the main steps.
**CN:** `kill_process_tree` 负责实现本模块使用的辅助逻辑。 它主要处理 `pid` 等参数。 实现过程中会调用 `parent.children`, `psutil.Process`, `contextlib.suppress` 等函数完成关键步骤。

### set_ulimit (lines 280-301)
```python
def set_ulimit(target_soft_limit: int = 65535):
    if sys.platform.startswith("win"):
        logger.info("Windows detected, skipping ulimit adjustment.")
        return

    import resource

    resource_type = resource.RLIMIT_NOFILE
    current_soft, current_hard = resource.getrlimit(resource_type)

    if current_soft < target_soft_limit:
        try:
            resource.setrlimit(resource_type, (target_soft_limit, current_hard))
        except ValueError as e:
            logger.warning(
                "Found ulimit of %s and failed to automatically increase "
                "with error %s. This can cause fd limit errors like "
                "`OSError: [Errno 24] Too many open files`. Consider "
                "increasing with ulimit -n",
                current_soft,
                e,
            )
```
**EN:** `set_ulimit` writes values into an object or configuration structure. It mainly works with `target_soft_limit`. Inside the body, it relies on `sys.platform.startswith`, `resource.getrlimit`, `logger.info` to complete the main steps.
**CN:** `set_ulimit` 负责把值写入对象或配置结构。 它主要处理 `target_soft_limit` 等参数。 实现过程中会调用 `sys.platform.startswith`, `resource.getrlimit`, `logger.info` 等函数完成关键步骤。

### find_loaded_library (lines 304-328)
```python
def find_loaded_library(lib_name: str) -> str | None:
    """
    According to according to https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html,
    the file `/proc/self/maps` contains the memory maps of the process, which includes the
    shared libraries loaded by the process. We can use this file to find the path of the
    loaded library.
    """  # noqa
    found_line = None
    with open("/proc/self/maps") as f:
        for line in f:
            if lib_name in line:
                found_line = line
                break
    if found_line is None:
        # the library is not loaded in the current process
        return None
    # if lib_name is libcudart, we need to match a line with:
    # address /path/to/libcudart-hash.so.11.0
    start = found_line.index("/")
    path = found_line[start:].strip()
    filename = path.split("/")[-1]
    assert filename.rpartition(".so")[0].startswith(lib_name), (
        f"Unexpected filename: {filename} for library {lib_name}"
    )
    return path
```
**EN:** `find_loaded_library`: According to according to https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html, the file `/proc/self/maps` contains the memory maps of the process, which includes the shared libraries loaded by the process. It mainly works with `lib_name`. Inside the body, it relies on `found_line.index`, `found_line.strip`, `filename.rpartition.startswith` to complete the main steps.
**CN:** `find_loaded_library` 负责实现本模块使用的辅助逻辑。 它主要处理 `lib_name` 等参数。 实现过程中会调用 `found_line.index`, `found_line.strip`, `filename.rpartition.startswith` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`update_environment_variables`**: Key helper or entry point in this file. / **`update_environment_variables`**：本文件中的关键辅助函数或入口。
- **`set_env_var`**: Key helper or entry point in this file. / **`set_env_var`**：本文件中的关键辅助函数或入口。
- **`suppress_stdout`**: Key helper or entry point in this file. / **`suppress_stdout`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: __future__, contextlib, multiprocessing, os, signal, sys, collections.abc, pathlib, typing, resource
- **Third-party / 第三方**: psutil, ray, setproctitle
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger, vllm.platforms, vllm.platforms.interface, vllm.ray.lazy_utils, .platform_utils, vllm.platforms.rocm
