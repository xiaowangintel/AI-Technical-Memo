# network_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/network_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-24)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import contextlib
import ipaddress
import os
import socket
import sys
import warnings
from collections.abc import (
    Iterator,
    Sequence,
)
from typing import Any
from uuid import uuid4

import psutil
import zmq
import zmq.asyncio
from urllib3.util import parse_url

import vllm.envs as envs
from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `contextlib`, `ipaddress`, `os`, external packages such as `psutil`, `zmq`, `zmq.asyncio`, vLLM modules such as `vllm.envs`, `vllm.logger`. It prepares the symbols later used by `close_sockets`, `get_ip`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `close_sockets`, `get_ip` 提供上下文。

### close_sockets (lines 27-30)
```python
def close_sockets(sockets: Sequence[zmq.Socket | zmq.asyncio.Socket]):
    for sock in sockets:
        if sock is not None:
            sock.close(linger=0)
```
**EN:** `close_sockets` implements helper logic used by this module. It mainly works with `sockets`. Inside the body, it relies on `sock.close` to complete the main steps.
**CN:** `close_sockets` 负责实现本模块使用的辅助逻辑。 它主要处理 `sockets` 等参数。 实现过程中会调用 `sock.close` 等函数完成关键步骤。

### get_ip (lines 33-72)
```python
def get_ip() -> str:
    host_ip = envs.VLLM_HOST_IP
    if "HOST_IP" in os.environ and "VLLM_HOST_IP" not in os.environ:
        logger.warning(
            "The environment variable HOST_IP is deprecated and ignored, as"
            " it is often used by Docker and other software to"
            " interact with the container's network stack. Please "
            "use VLLM_HOST_IP instead to set the IP address for vLLM processes"
            " to communicate with each other."
        )
    if host_ip:
        return host_ip

    # IP is not set, try to get it from the network interface

    # try ipv4
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
            s.connect(("8.8.8.8", 80))  # Doesn't need to be reachable
            return s.getsockname()[0]
    except Exception:
        pass

    # try ipv6
    # ...
        "Failed to get the IP address, using 0.0.0.0 by default. "
        "The value can be set by the environment variable"
        " VLLM_HOST_IP or HOST_IP.",
        stacklevel=2,
    )
    return "0.0.0.0"
```
**EN:** `get_ip` retrieves data or state needed by the pipeline. Inside the body, it relies on `warnings.warn`, `logger.warning`, `socket.socket` to complete the main steps.
**CN:** `get_ip` 负责获取流水线所需的数据或状态。 实现过程中会调用 `warnings.warn`, `logger.warning`, `socket.socket` 等函数完成关键步骤。

### test_loopback_bind (lines 75-82)
```python
def test_loopback_bind(address: str, family: int) -> bool:
    try:
        s = socket.socket(family, socket.SOCK_DGRAM)
        s.bind((address, 0))  # Port 0 = auto assign
        s.close()
        return True
    except OSError:
        return False
```
**EN:** `test_loopback_bind` implements helper logic used by this module. It mainly works with `address`, `family`. Inside the body, it relies on `socket.socket`, `s.bind`, `s.close` to complete the main steps.
**CN:** `test_loopback_bind` 负责实现本模块使用的辅助逻辑。 它主要处理 `address`, `family` 等参数。 实现过程中会调用 `socket.socket`, `s.bind`, `s.close` 等函数完成关键步骤。

### get_loopback_ip (lines 85-100)
```python
def get_loopback_ip() -> str:
    loopback_ip = envs.VLLM_LOOPBACK_IP
    if loopback_ip:
        return loopback_ip

    # VLLM_LOOPBACK_IP is not set, try to get it based on network interface

    if test_loopback_bind("127.0.0.1", socket.AF_INET):
        return "127.0.0.1"
    elif test_loopback_bind("::1", socket.AF_INET6):
        return "::1"
    else:
        raise RuntimeError(
            "Neither 127.0.0.1 nor ::1 are bound to a local interface. "
            "Set the VLLM_LOOPBACK_IP environment variable explicitly."
        )
```
**EN:** `get_loopback_ip` retrieves data or state needed by the pipeline. Inside the body, it relies on `test_loopback_bind`, `RuntimeError` to complete the main steps.
**CN:** `get_loopback_ip` 负责获取流水线所需的数据或状态。 实现过程中会调用 `test_loopback_bind`, `RuntimeError` 等函数完成关键步骤。

### is_valid_ipv6_address (lines 103-108)
```python
def is_valid_ipv6_address(address: str) -> bool:
    try:
        ipaddress.IPv6Address(address)
        return True
    except ValueError:
        return False
```
**EN:** `is_valid_ipv6_address` checks a condition and returns a boolean-style result. It mainly works with `address`. Inside the body, it relies on `ipaddress.IPv6Address` to complete the main steps.
**CN:** `is_valid_ipv6_address` 负责检查条件并返回布尔结果。 它主要处理 `address` 等参数。 实现过程中会调用 `ipaddress.IPv6Address` 等函数完成关键步骤。

### split_host_port (lines 111-120)
```python
def split_host_port(host_port: str) -> tuple[str, int]:
    # ipv6
    if host_port.startswith("["):
        host, port = host_port.rsplit("]", 1)
        host = host[1:]
        port = port.split(":")[1]
        return host, int(port)
    else:
        host, port = host_port.split(":")
        return host, int(port)
```
**EN:** `split_host_port` implements helper logic used by this module. It mainly works with `host_port`. Inside the body, it relies on `host_port.startswith`, `host_port.rsplit`, `host_port.split` to complete the main steps.
**CN:** `split_host_port` 负责实现本模块使用的辅助逻辑。 它主要处理 `host_port` 等参数。 实现过程中会调用 `host_port.startswith`, `host_port.rsplit`, `host_port.split` 等函数完成关键步骤。

### join_host_port (lines 123-127)
```python
def join_host_port(host: str, port: int) -> str:
    if is_valid_ipv6_address(host):
        return f"[{host}]:{port}"
    else:
        return f"{host}:{port}"
```
**EN:** `join_host_port` implements helper logic used by this module. It mainly works with `host`, `port`. Inside the body, it relies on `is_valid_ipv6_address` to complete the main steps.
**CN:** `join_host_port` 负责实现本模块使用的辅助逻辑。 它主要处理 `host`, `port` 等参数。 实现过程中会调用 `is_valid_ipv6_address` 等函数完成关键步骤。

### get_distributed_init_method (lines 130-131)
```python
def get_distributed_init_method(ip: str, port: int) -> str:
    return get_tcp_uri(ip, port)
```
**EN:** `get_distributed_init_method` retrieves data or state needed by the pipeline. It mainly works with `ip`, `port`. Inside the body, it relies on `get_tcp_uri` to complete the main steps.
**CN:** `get_distributed_init_method` 负责获取流水线所需的数据或状态。 它主要处理 `ip`, `port` 等参数。 实现过程中会调用 `get_tcp_uri` 等函数完成关键步骤。

### get_tcp_uri (lines 134-138)
```python
def get_tcp_uri(ip: str, port: int) -> str:
    if is_valid_ipv6_address(ip):
        return f"tcp://[{ip}]:{port}"
    else:
        return f"tcp://{ip}:{port}"
```
**EN:** `get_tcp_uri` retrieves data or state needed by the pipeline. It mainly works with `ip`, `port`. Inside the body, it relies on `is_valid_ipv6_address` to complete the main steps.
**CN:** `get_tcp_uri` 负责获取流水线所需的数据或状态。 它主要处理 `ip`, `port` 等参数。 实现过程中会调用 `is_valid_ipv6_address` 等函数完成关键步骤。

### get_open_zmq_ipc_path (lines 141-143)
```python
def get_open_zmq_ipc_path() -> str:
    base_rpc_path = envs.VLLM_RPC_BASE_PATH
    return f"ipc://{base_rpc_path}/{uuid4()}"
```
**EN:** `get_open_zmq_ipc_path` retrieves data or state needed by the pipeline. Inside the body, it relies on `uuid4` to complete the main steps.
**CN:** `get_open_zmq_ipc_path` 负责获取流水线所需的数据或状态。 实现过程中会调用 `uuid4` 等函数完成关键步骤。

### get_open_zmq_inproc_path (lines 146-147)
```python
def get_open_zmq_inproc_path() -> str:
    return f"inproc://{uuid4()}"
```
**EN:** `get_open_zmq_inproc_path` retrieves data or state needed by the pipeline. Inside the body, it relies on `uuid4` to complete the main steps.
**CN:** `get_open_zmq_inproc_path` 负责获取流水线所需的数据或状态。 实现过程中会调用 `uuid4` 等函数完成关键步骤。

### get_open_port (lines 150-166)
```python
def get_open_port() -> int:
    """
    Get an open port for the vLLM process to listen on.
    An edge case to handle, is when we run data parallel,
    we need to avoid ports that are potentially used by
    the data parallel master process.
    Right now we reserve 10 ports for the data parallel master
    process. Currently it uses 2 ports.
    """
    if "VLLM_DP_MASTER_PORT" in os.environ:
        dp_master_port = envs.VLLM_DP_MASTER_PORT
        reserved_port_range = range(dp_master_port, dp_master_port + 10)
        while True:
            candidate_port = _get_open_port()
            if candidate_port not in reserved_port_range:
                return candidate_port
    return _get_open_port()
```
**EN:** `get_open_port`: Get an open port for the vLLM process to listen on. Inside the body, it relies on `_get_open_port` to complete the main steps.
**CN:** `get_open_port` 负责获取流水线所需的数据或状态。 实现过程中会调用 `_get_open_port` 等函数完成关键步骤。

### get_open_ports_list (lines 169-187)
```python
def get_open_ports_list(count: int = 5) -> list[int]:
    """Get a list of unique open ports.

    When VLLM_PORT is set, scans upward from that port, advancing
    the start position after each find so every port is unique.
    """
    ports_set = set[int]()
    if envs.VLLM_PORT is not None:
        next_port = envs.VLLM_PORT
        for _ in range(count):
            port = _get_open_port(start_port=next_port, max_attempts=1000)
            ports_set.add(port)
            next_port = port + 1
        return list(ports_set)
    else:
        while len(ports_set) < count:
            ports_set.add(get_open_port())

    return list(ports_set)
```
**EN:** `get_open_ports_list`: Get a list of unique open ports. It mainly works with `count`. Inside the body, it relies on `_get_open_port`, `ports_set.add`, `get_open_port` to complete the main steps.
**CN:** `get_open_ports_list` 负责获取流水线所需的数据或状态。 它主要处理 `count` 等参数。 实现过程中会调用 `_get_open_port`, `ports_set.add`, `get_open_port` 等函数完成关键步骤。

### _get_open_port (lines 190-221)
```python
def _get_open_port(
    start_port: int | None = None,
    max_attempts: int | None = None,
) -> int:
    start_port = start_port if start_port is not None else envs.VLLM_PORT
    port = start_port
    if port is not None:
        attempts = 0
        while True:
            try:
                with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
                    s.bind(("", port))
                    return port
            except OSError:
                port += 1  # Increment port number if already in use
                logger.info("Port %d is already in use, trying port %d", port - 1, port)
            attempts += 1
            if max_attempts is not None and attempts >= max_attempts:
                raise RuntimeError(
                    f"Could not find open port after {max_attempts} "
                    f"attempts starting from port {start_port}"
                )
    # try ipv4
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.bind(("", 0))
            return s.getsockname()[1]
    except OSError:
        # try ipv6
        with socket.socket(socket.AF_INET6, socket.SOCK_STREAM) as s:
            s.bind(("", 0))
            return s.getsockname()[1]
```
**EN:** `_get_open_port` retrieves data or state needed by the pipeline. It mainly works with `start_port`, `max_attempts`. Inside the body, it relies on `socket.socket`, `s.bind`, `RuntimeError` to complete the main steps.
**CN:** `_get_open_port` 负责获取流水线所需的数据或状态。 它主要处理 `start_port`, `max_attempts` 等参数。 实现过程中会调用 `socket.socket`, `s.bind`, `RuntimeError` 等函数完成关键步骤。

### find_process_using_port (lines 224-239)
```python
def find_process_using_port(port: int) -> psutil.Process | None:
    # TODO: We can not check for running processes with network
    # port on macOS. Therefore, we can not have a full graceful shutdown
    # of vLLM. For now, let's not look for processes in this case.
    # Ref: https://www.florianreinhard.de/accessdenied-in-psutil/
    if sys.platform.startswith("darwin"):
        return None

    our_pid = os.getpid()
    for conn in psutil.net_connections():
        if conn.laddr.port == port and (conn.pid is not None and conn.pid != our_pid):
            try:
                return psutil.Process(conn.pid)
            except psutil.NoSuchProcess:
                return None
    return None
```
**EN:** `find_process_using_port` implements helper logic used by this module. It mainly works with `port`. Inside the body, it relies on `sys.platform.startswith`, `os.getpid`, `psutil.net_connections` to complete the main steps.
**CN:** `find_process_using_port` 负责实现本模块使用的辅助逻辑。 它主要处理 `port` 等参数。 实现过程中会调用 `sys.platform.startswith`, `os.getpid`, `psutil.net_connections` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`close_sockets`**: Key helper or entry point in this file. / **`close_sockets`**：本文件中的关键辅助函数或入口。
- **`get_ip`**: Key helper or entry point in this file. / **`get_ip`**：本文件中的关键辅助函数或入口。
- **`test_loopback_bind`**: Key helper or entry point in this file. / **`test_loopback_bind`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: contextlib, ipaddress, os, socket, sys, warnings, collections.abc, typing, uuid
- **Third-party / 第三方**: psutil, zmq, zmq.asyncio, urllib3.util
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger
