# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for common in the multimodal generation stack. Key symbols include `kill_process_tree`, `add_prefix`, `is_valid_ipv6_address`. / 该模块包含多模态生成体系中与 common 相关的运行时支持代码。 关键符号包括 `kill_process_tree`, `add_prefix`, `is_valid_ipv6_address`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

import ipaddress
import logging
import os
import platform
import signal
import socket
import sys
import threading
from functools import lru_cache

import psutil
import torch
import zmq

# use the native logger to avoid circular import
logger = logging.getLogger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-57: Function `kill_process_tree` / 函数 `kill_process_tree`
```python
def kill_process_tree(parent_pid, include_parent: bool = True, skip_pid: int = None):
    """Kill the process and all its child processes."""
    # Remove sigchld handler to avoid spammy logs.
    if threading.current_thread() is threading.main_thread():
        signal.signal(signal.SIGCHLD, signal.SIG_DFL)

    if parent_pid is None:
        parent_pid = os.getpid()
        include_parent = False

    try:
        itself = psutil.Process(parent_pid)
    except psutil.NoSuchProcess:
        return
# ...
            # so we send an additional signal to kill them.
            itself.send_signal(signal.SIGQUIT)
        except psutil.NoSuchProcess:
            pass
```
**EN:** This function drives `kill_process_tree` with inputs such as `parent_pid`, `include_parent`, `skip_pid`. Kill the process and all its child processes.
**CN:** 这个函数负责 `kill_process_tree`，主要处理 `parent_pid`, `include_parent`, `skip_pid` 等输入。 文档字符串说明：Kill the process and all its child processes.

### Lines 60-70: Function `add_prefix` / 函数 `add_prefix`
```python
def add_prefix(name: str, prefix: str) -> str:
    """Add a weight path prefix to a module name.

    Args:
        name: base module name.
        prefix: weight prefix str to added to the front of `name` concatenated with `.`.

    Returns:
        The string `prefix.name` if prefix is non-empty, otherwise just `name`.
    """
    return name if not prefix else f"{prefix}.{name}"
```
**EN:** This function drives `add_prefix` with inputs such as `name`, `prefix`. Add a weight path prefix to a module name.
**CN:** 这个函数负责 `add_prefix`，主要处理 `name`, `prefix` 等输入。 文档字符串说明：Add a weight path prefix to a module name.

### Lines 73-78: Function `is_valid_ipv6_address` / 函数 `is_valid_ipv6_address`
```python
def is_valid_ipv6_address(address: str) -> bool:
    try:
        ipaddress.IPv6Address(address)
        return True
    except ValueError:
        return False
```
**EN:** This function drives `is_valid_ipv6_address` with inputs such as `address`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_valid_ipv6_address`，主要处理 `address` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 81-110: Function `configure_ipv6` / 函数 `configure_ipv6`
```python
def configure_ipv6(dist_init_addr):
    addr = dist_init_addr
    end = addr.find("]")
    if end == -1:
        raise ValueError("invalid IPv6 address format: missing ']'")

    host = addr[: end + 1]

    # this only validates the address without brackets: we still need the below checks.
    # if it's invalid, immediately raise an error so we know it's not formatting issues.
    if not is_valid_ipv6_address(host[1:end]):
        raise ValueError(f"invalid IPv6 address: {host}")

    port_str = None
# ...
        port = int(port_str)
    except ValueError:
        raise ValueError(f"invalid port in IPv6 address: '{port_str}'")
    return port, host
```
**EN:** This function drives `configure_ipv6` with inputs such as `dist_init_addr`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `configure_ipv6`，主要处理 `dist_init_addr` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 113-124: Function `is_port_available` / 函数 `is_port_available`
```python
def is_port_available(port):
    """Return whether a port is available."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        try:
            s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            s.bind(("", port))
            s.listen(1)
            return True
        except socket.error:
            return False
        except OverflowError:
            return False
```
**EN:** This function drives `is_port_available` with inputs such as `port`. Return whether a port is available.
**CN:** 这个函数负责 `is_port_available`，主要处理 `port` 等输入。 文档字符串说明：Return whether a port is available.

### Lines 127-264: Function `get_zmq_socket` / 函数 `get_zmq_socket`
```python
def get_zmq_socket(
    context: zmq.Context,
    socket_type: zmq.SocketType,
    endpoint: str,
    bind: bool,
    max_bind_retries: int = 10,
    same_port: bool = False,
) -> tuple[zmq.Socket, str]:
    """
    Create and configure a ZMQ socket.

    Args:
        context: ZMQ context
        socket_type: Type of ZMQ socket
# ...
        socket.connect(endpoint)
        return socket, endpoint

    return socket, endpoint
```
**EN:** This function drives `get_zmq_socket` with inputs such as `context`, `socket_type`, `endpoint`, `bind`. Create and configure a ZMQ socket.
**CN:** 这个函数负责 `get_zmq_socket`，主要处理 `context`, `socket_type`, `endpoint`, `bind` 等输入。 文档字符串说明：Create and configure a ZMQ socket.

### Lines 270-277: Function `is_host_cpu_x86` / 函数 `is_host_cpu_x86`
```python
@lru_cache(maxsize=1)
def is_host_cpu_x86() -> bool:
    machine = platform.machine().lower()
    return (
        machine in ("x86_64", "amd64", "i386", "i686")
        and hasattr(torch, "cpu")
        and torch.cpu.is_available()
    )
```
**EN:** This function drives `is_host_cpu_x86`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_host_cpu_x86`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 283-288: Function `set_cuda_arch` / 函数 `set_cuda_arch`
```python
def set_cuda_arch():
    """Set CUDA architecture for compilation. Only applies to CUDA devices."""
    if torch.cuda.is_available():
        capability = torch.cuda.get_device_capability()
        arch = f"{capability[0]}.{capability[1]}"
        os.environ["TORCH_CUDA_ARCH_LIST"] = f"{arch}{'+PTX' if arch == '9.0' else ''}"
```
**EN:** This function drives `set_cuda_arch`. Set CUDA architecture for compilation.
**CN:** 这个函数负责 `set_cuda_arch`。 文档字符串说明：Set CUDA architecture for compilation.

### Lines 295-298: Function `set_musa_arch` / 函数 `set_musa_arch`
```python
def set_musa_arch():
    capability = torch.cuda.get_device_capability()
    arch = f"{capability[0]}{capability[1]}"
    os.environ["TORCH_MUSA_ARCH_LIST"] = f"{arch}"
```
**EN:** This function drives `set_musa_arch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `set_musa_arch`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 299-303: Top-level configuration / 顶层配置
```python


# env var managements

_warned_bool_env_var_keys = set()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 306-320: Function `get_bool_env_var` / 函数 `get_bool_env_var`
```python
def get_bool_env_var(name: str, default: str = "false") -> bool:
    value = os.getenv(name, default)
    value = str(value).strip().lower()

    truthy_values = {"1", "true", "yes", "y", "t", "on"}
    falsy_values = {"0", "false", "no", "n", "f", "off", ""}

    if (value not in truthy_values) and (value not in falsy_values):
        if value not in _warned_bool_env_var_keys:
            logger.warning(
                f"get_bool_env_var({name}) see non-understandable value={value} and treat as false"
            )
        _warned_bool_env_var_keys.add(value)

    return value in truthy_values
```
**EN:** This function drives `get_bool_env_var` with inputs such as `name`, `default`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_bool_env_var`，主要处理 `name`, `default` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 321-337: Imports and module setup / 导入与模块初始化
```python


try:
    import sgl_kernel  # noqa: F401

    is_intel_amx_backend_available = hasattr(
        torch.ops.sgl_kernel, "convert_weight_packed"
    )
except:
    is_intel_amx_backend_available = False

try:
    # move torch._C._cpu._is_amx_tile_supported() from cpu_has_amx_support
    # to support torch compile
    is_amx_tile_supported = torch._C._cpu._is_amx_tile_supported()
except:
    is_amx_tile_supported = False
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 340-341: Function `cpu_has_amx_support` / 函数 `cpu_has_amx_support`
```python
def cpu_has_amx_support():
    return is_amx_tile_supported and is_intel_amx_backend_available
```
**EN:** This function drives `cpu_has_amx_support`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `cpu_has_amx_support`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 344-345: Function `use_intel_amx_backend` / 函数 `use_intel_amx_backend`
```python
def use_intel_amx_backend(layer):
    return getattr(layer, "use_intel_amx_backend", False)
```
**EN:** This function drives `use_intel_amx_backend` with inputs such as `layer`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `use_intel_amx_backend`，主要处理 `layer` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- HTTP route definition / HTTP 路由定义
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Video generation flow / 视频生成流程
- Symbol `kill_process_tree` anchors the module API / 符号 `kill_process_tree` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `ipaddress`, `platform`, `psutil`, `torch`, `zmq`, `sgl_kernel`
- **Stdlib / 标准库**: `logging`, `os`, `signal`, `socket`, `sys`, `threading`, `functools`, `re`
