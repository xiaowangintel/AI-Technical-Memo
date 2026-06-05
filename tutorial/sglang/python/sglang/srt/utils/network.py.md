# network.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/network.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for networking and distributed coordination helpers. / 为 SGLang 运行时提供面向网络与分布式协调辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import ipaddress
import logging
import os
import socket
import time
from dataclasses import dataclass
from typing import Optional, Tuple, Union

import psutil
import zmq

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `ipaddress`, `logging`, `os`, `socket`, `time`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `ipaddress`, `logging`, `os`, `socket`, `time`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 17-29: Function `get_open_port` / 函数 `get_open_port`
```python
def get_open_port() -> int:
    port = os.getenv("SGLANG_PORT")
    if port is not None:
        port = int(port)
        while True:
            if is_port_available(port):
                return port
            logger.info("Port %d is already in use, trying port %d", port, port + 1)
            port += 1
    sock = try_bind_socket()
    port = sock.getsockname()[1]
    sock.close()
    return port
```
**EN:** This function implements `get_open_port`. It primarily calls `os.getenv`, `try_bind_socket`, `sock.close`, `int`, `sock.getsockname`, `is_port_available` to complete its work. State updates are written into `port`, `sock`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `get_open_port`。 它主要通过调用 `os.getenv`, `try_bind_socket`, `sock.close`, `int`, `sock.getsockname`, `is_port_available` 来完成任务。 状态更新主要写入 `port`, `sock`。 实现中使用了条件分支、迭代逻辑。

### Lines 32-37: Function `is_valid_ipv6_address` / 函数 `is_valid_ipv6_address`
```python
def is_valid_ipv6_address(address: str) -> bool:
    try:
        ipaddress.IPv6Address(address)
        return True
    except ValueError:
        return False
```
**EN:** This function implements `is_valid_ipv6_address`. It primarily calls `ipaddress.IPv6Address` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `is_valid_ipv6_address`。 它主要通过调用 `ipaddress.IPv6Address` 来完成任务。 实现中使用了错误处理。

### Lines 40-49: Function `find_process_using_port` / 函数 `find_process_using_port`
```python
def find_process_using_port(port: int) -> Optional[psutil.Process]:
    for conn in psutil.net_connections(kind="inet"):
        if conn.laddr.port == port:
            try:
                return psutil.Process(conn.pid)
            except psutil.NoSuchProcess:
                # It could happen by race condition (the proc dies when psutil.Process is called).
                pass

    return None
```
**EN:** This function implements `find_process_using_port`. It primarily calls `psutil.net_connections`, `psutil.Process` to complete its work. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `find_process_using_port`。 它主要通过调用 `psutil.net_connections`, `psutil.Process` 来完成任务。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 52-77: Function `wait_port_available` / 函数 `wait_port_available`
```python
def wait_port_available(
    port: int, port_name: str, timeout_s: int = 30, raise_exception: bool = True
) -> bool:
    for i in range(timeout_s):
        if is_port_available(port):
            return True

        if i > 10 and i % 5 == 0:
            process = find_process_using_port(port)
            if process is None:
                logger.warning(
                    f"The port {port} is in use, but we could not find the process that uses it."
                )

            pid = process.pid
            error_message = f"{port_name} is used by a process already. {process.name()=}' {process.cmdline()=} {process.status()=} {pid=}"
            logger.info(
                f"port {port} is in use. Waiting for {i} seconds for {port_name} to be available. {error_message}"
            )
        time.sleep(0.1)

    if raise_exception:
        raise ValueError(
            f"{port_name} at {port} is not available in {timeout_s} seconds. {error_message}"
        )
    return False
```
**EN:** This function implements `wait_port_available`. It primarily calls `range`, `is_port_available`, `time.sleep`, `ValueError`, `find_process_using_port`, `logger.info` to complete its work. State updates are written into `process`, `pid`, `error_message`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `wait_port_available`。 它主要通过调用 `range`, `is_port_available`, `time.sleep`, `ValueError`, `find_process_using_port`, `logger.info` 来完成任务。 状态更新主要写入 `process`, `pid`, `error_message`。 实现中使用了条件分支、迭代逻辑。

### Lines 80-115: Function `_get_addrinfos_for_bind` / 函数 `_get_addrinfos_for_bind`
```python
def _get_addrinfos_for_bind(host=None, port=0):
    """Return deduplicated addrinfo tuples for binding (one per address family).

    Args:
        host: Bind address. None (with AI_PASSIVE) resolves to wildcard
              addresses (0.0.0.0 / ::) suitable for accepting on all interfaces.
        port: Port number. 0 lets the OS assign an available ephemeral port.

    Flags:
        AI_ADDRCONFIG — only return families actually configured on this host.
        AI_PASSIVE    — return wildcard addresses suitable for bind().

    Falls back to AF_INET if getaddrinfo fails (e.g. DNS misconfiguration).
    """
    try:
        infos = socket.getaddrinfo(
            host,
            port,
            socket.AF_UNSPEC,
            socket.SOCK_STREAM,
            0,
            socket.AI_ADDRCONFIG | socket.AI_PASSIVE,
        )
        deduped = []
        seen_families = set()
        for info in infos:
            if info[0] not in seen_families:
                seen_families.add(info[0])
                deduped.append(info)
        # Prefer IPv4 so that callers without an explicit host get consistent
        # behaviour across platforms (some OSes list IPv6 first).
        deduped.sort(key=lambda x: (x[0] != socket.AF_INET,))
        return deduped
    except socket.gaierror:
        fallback_host = "0.0.0.0" if host is None else host
        return [(socket.AF_INET, socket.SOCK_STREAM, 0, "", (fallback_host, port))]
```
**EN:** This function implements `_get_addrinfos_for_bind`. It primarily calls `socket.getaddrinfo`, `set`, `deduped.sort`, `seen_families.add`, `deduped.append` to complete its work. State updates are written into `infos`, `deduped`, `seen_families`, `fallback_host`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_get_addrinfos_for_bind`。 它主要通过调用 `socket.getaddrinfo`, `set`, `deduped.sort`, `seen_families.add`, `deduped.append` 来完成任务。 状态更新主要写入 `infos`, `deduped`, `seen_families`, `fallback_host`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 118-150: Function `try_bind_socket` / 函数 `try_bind_socket`
```python
def try_bind_socket(host=None, port=0, *, reuse_addr=True, listen=False):
    """Bind a TCP socket on the first available address family (IPv4/IPv6).

    Iterates over address families returned by _get_addrinfos_for_bind and
    returns the first socket that successfully binds.

    Args:
        host: Bind address. None binds to all interfaces (0.0.0.0 / ::).
        port: Port number. 0 lets the OS assign an available ephemeral port;
              use sock.getsockname()[1] to retrieve the assigned port.
        reuse_addr: Set SO_REUSEADDR to allow quick port reuse after close.
        listen: Call listen(1) after bind, making the socket ready to accept.

    Returns:
        The bound socket. Caller is responsible for closing it.

    Raises:
        OSError: If bind fails on all configured address families.
    """
    for family, socktype, proto, _, sockaddr in _get_addrinfos_for_bind(host, port):
        sock = socket.socket(family, socktype, proto)
        try:
            if family == socket.AF_INET6:
                sock.setsockopt(socket.IPPROTO_IPV6, socket.IPV6_V6ONLY, 1)
            if reuse_addr:
                sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            sock.bind(sockaddr)
            if listen:
                sock.listen(1)
            return sock
        except (OSError, OverflowError):
            sock.close()
    raise OSError(f"Could not bind port {port} on any configured address family")
```
**EN:** This function implements `try_bind_socket`. It primarily calls `_get_addrinfos_for_bind`, `OSError`, `socket.socket`, `sock.bind`, `sock.setsockopt`, `sock.listen` to complete its work. State updates are written into `sock`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `try_bind_socket`。 它主要通过调用 `_get_addrinfos_for_bind`, `OSError`, `socket.socket`, `sock.bind`, `sock.setsockopt`, `sock.listen` 来完成任务。 状态更新主要写入 `sock`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 153-167: Function `is_port_available` / 函数 `is_port_available`
```python
def is_port_available(port):
    """Return whether a port is available on all configured address families."""
    try:
        for family, socktype, proto, _, sockaddr in _get_addrinfos_for_bind(port=port):
            sock = socket.socket(family, socktype, proto)
            try:
                if family == socket.AF_INET6:
                    sock.setsockopt(socket.IPPROTO_IPV6, socket.IPV6_V6ONLY, 1)
                sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
                sock.bind(sockaddr)
            finally:
                sock.close()
        return True
    except (OSError, OverflowError):
        return False
```
**EN:** This function implements `is_port_available`. It primarily calls `_get_addrinfos_for_bind`, `socket.socket`, `sock.setsockopt`, `sock.bind`, `sock.close` to complete its work. State updates are written into `sock`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `is_port_available`。 它主要通过调用 `_get_addrinfos_for_bind`, `socket.socket`, `sock.setsockopt`, `sock.bind`, `sock.close` 来完成任务。 状态更新主要写入 `sock`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 170-174: Function `get_free_port` / 函数 `get_free_port`
```python
def get_free_port():
    sock = try_bind_socket()
    port = sock.getsockname()[1]
    sock.close()
    return port
```
**EN:** This function implements `get_free_port`. It primarily calls `try_bind_socket`, `sock.close`, `sock.getsockname` to complete its work. State updates are written into `sock`, `port`.
**CN:** 该函数实现了 `get_free_port`。 它主要通过调用 `try_bind_socket`, `sock.close`, `sock.getsockname` 来完成任务。 状态更新主要写入 `sock`, `port`。

### Lines 177-179: Function `bind_port` / 函数 `bind_port`
```python
def bind_port(port):
    """Bind to a specific port, assuming it's available."""
    return try_bind_socket(port=port, listen=True)
```
**EN:** This function implements `bind_port`. It primarily calls `try_bind_socket` to complete its work.
**CN:** 该函数实现了 `bind_port`。 它主要通过调用 `try_bind_socket` 来完成任务。

### Lines 182-210: Function `get_zmq_socket_on_host` / 函数 `get_zmq_socket_on_host`
```python
def get_zmq_socket_on_host(
    context: zmq.Context,
    socket_type: zmq.SocketType,
    host: Optional[str] = None,
) -> Tuple[int, zmq.Socket]:
    """Create and configure a ZeroMQ socket.

    Args:
        context: ZeroMQ context to create the socket from.
        socket_type: Type of ZeroMQ socket to create.
        host: Host to bind to, without "tcp://" prefix. Defaults to
            "127.0.0.1" (localhost-only) to avoid exposing unauthenticated
            sockets to the network (CVE-2026-3060). Callers that need
            cross-machine reachability must pass an explicit host.

    Returns:
        Tuple of (port, socket) where port is the randomly assigned TCP port.
    """
    socket = context.socket(socket_type)
    config_socket(socket, socket_type)
    if host is None:
        host = "127.0.0.1"
    if is_valid_ipv6_address(host):
        socket.setsockopt(zmq.IPV6, 1)
        bind_host = f"tcp://[{host}]"
    else:
        bind_host = f"tcp://{host}"
    port = socket.bind_to_random_port(bind_host)
    return port, socket
```
**EN:** This function implements `get_zmq_socket_on_host`. It primarily calls `context.socket`, `config_socket`, `is_valid_ipv6_address`, `socket.bind_to_random_port`, `socket.setsockopt` to complete its work. State updates are written into `socket`, `port`, `host`, `bind_host`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_zmq_socket_on_host`。 它主要通过调用 `context.socket`, `config_socket`, `is_valid_ipv6_address`, `socket.bind_to_random_port`, `socket.setsockopt` 来完成任务。 状态更新主要写入 `socket`, `port`, `host`, `bind_host`。 实现中使用了条件分支。

### Lines 213-238: Function `config_socket` / 函数 `config_socket`
```python
def config_socket(socket, socket_type: zmq.SocketType):
    mem = psutil.virtual_memory()
    total_mem = mem.total / 1024**3
    available_mem = mem.available / 1024**3
    if total_mem > 32 and available_mem > 16:
        buf_size = int(0.5 * 1024**3)
    else:
        buf_size = -1

    def set_send_opt():
        socket.setsockopt(zmq.SNDHWM, 0)
        socket.setsockopt(zmq.SNDBUF, buf_size)

    def set_recv_opt():
        socket.setsockopt(zmq.RCVHWM, 0)
        socket.setsockopt(zmq.RCVBUF, buf_size)

    if socket_type == zmq.PUSH:
        set_send_opt()
    elif socket_type == zmq.PULL:
        set_recv_opt()
    elif socket_type in [zmq.DEALER, zmq.REQ, zmq.REP]:
        set_send_opt()
        set_recv_opt()
    else:
        raise ValueError(f"Unsupported socket type: {socket_type}")
```
**EN:** This function implements `config_socket`. It primarily calls `psutil.virtual_memory`, `int`, `socket.setsockopt`, `set_send_opt`, `set_recv_opt`, `ValueError` to complete its work. State updates are written into `mem`, `total_mem`, `available_mem`, `buf_size`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `config_socket`。 它主要通过调用 `psutil.virtual_memory`, `int`, `socket.setsockopt`, `set_send_opt`, `set_recv_opt`, `ValueError` 来完成任务。 状态更新主要写入 `mem`, `total_mem`, `available_mem`, `buf_size`。 实现中使用了条件分支。

### Lines 241-267: Function `get_local_ip_by_nic` / 函数 `get_local_ip_by_nic`
```python
def get_local_ip_by_nic(interface: str = None) -> Optional[str]:
    if not (interface := interface or os.environ.get("SGLANG_LOCAL_IP_NIC", None)):
        return None
    try:
        import netifaces
    except ImportError as e:
        raise ImportError(
            "Environment variable SGLANG_LOCAL_IP_NIC requires package netifaces, please install it through 'pip install netifaces'"
        ) from e

    try:
        addresses = netifaces.ifaddresses(interface)
        if netifaces.AF_INET in addresses:
            for addr_info in addresses[netifaces.AF_INET]:
                ip = addr_info.get("addr")
                if ip and ip != "127.0.0.1" and ip != "0.0.0.0":
                    return ip
        if netifaces.AF_INET6 in addresses:
            for addr_info in addresses[netifaces.AF_INET6]:
                ip = addr_info.get("addr")
                if ip and not ip.startswith("fe80::") and ip != "::1":
                    return ip.split("%")[0]
    except (ValueError, OSError) as e:
        logger.warning(
            f"{e} Can not get local ip from NIC. Please verify whether SGLANG_LOCAL_IP_NIC is set correctly."
        )
    return None
```
**EN:** This function implements `get_local_ip_by_nic`. It primarily calls `netifaces.ifaddresses`, `ImportError`, `logger.warning`, `os.environ.get`, `addr_info.get`, `ip.startswith` to complete its work. State updates are written into `addresses`, `ip`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `get_local_ip_by_nic`。 它主要通过调用 `netifaces.ifaddresses`, `ImportError`, `logger.warning`, `os.environ.get`, `addr_info.get`, `ip.startswith` 来完成任务。 状态更新主要写入 `addresses`, `ip`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 270-305: Function `get_local_ip_by_remote` / 函数 `get_local_ip_by_remote`
```python
def get_local_ip_by_remote() -> Optional[str]:
    # Google's public DNS servers, used to discover the local IP.
    # UDP connect doesn't send packets; it just selects the right source address.
    # https://developers.google.com/speed/public-dns/docs/using#addresses
    # Try IPv4 first, then IPv6. getaddrinfo on a literal IP returns exactly
    # one result, so we unpack directly instead of looping.
    for dns_host, dns_port in [("8.8.8.8", 80), ("2001:4860:4860::8888", 80)]:
        try:
            family, socktype, proto, _, sockaddr = socket.getaddrinfo(
                dns_host,
                dns_port,
                socket.AF_UNSPEC,
                socket.SOCK_DGRAM,
                0,
                socket.AI_ADDRCONFIG,
            )[0]
            with socket.socket(family, socktype, proto) as s:
                s.connect(sockaddr)
                return s.getsockname()[0]
        except (socket.gaierror, OSError):
            continue

    # Fallback: resolve the local hostname to an IP address via /etc/hosts or DNS.
    # Unreliable — many machines resolve hostname to 127.0.0.1, so we skip loopback.
    try:
        hostname = socket.gethostname()
        ip = socket.getaddrinfo(
            hostname, None, socket.AF_UNSPEC, 0, 0, socket.AI_ADDRCONFIG
        )[0][4][0]
        if ip and ip not in ("127.0.0.1", "0.0.0.0", "::1"):
            return ip
    except Exception:
        pass

    logger.warning("Can not get local ip by remote")
    return None
```
**EN:** This function implements `get_local_ip_by_remote`. It primarily calls `logger.warning`, `socket.gethostname`, `socket.getaddrinfo`, `socket.socket`, `s.connect`, `s.getsockname` to complete its work. State updates are written into `hostname`, `ip`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `get_local_ip_by_remote`。 它主要通过调用 `logger.warning`, `socket.gethostname`, `socket.getaddrinfo`, `socket.socket`, `s.connect`, `s.getsockname` 来完成任务。 状态更新主要写入 `hostname`, `ip`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 308-348: Function `get_local_ip_auto` / 函数 `get_local_ip_auto`
```python
def get_local_ip_auto(fallback: str = None) -> str:
    """
    Automatically detect the local IP address using multiple fallback strategies.

    This function attempts to obtain the local IP address through several methods.
    If all methods fail, it returns the specified fallback value or raises an exception.

    Args:
        fallback (str, optional): Fallback IP address to return if all detection
            methods fail. For server applications, explicitly set this to
            "0.0.0.0" (IPv4) or "::" (IPv6) to bind to all available interfaces.
            Defaults to None.

    Returns:
        str: The detected local IP address, or the fallback value if detection fails.

    Raises:
        ValueError: If IP detection fails and no fallback value is provided.

    Note:
        The function tries detection methods in the following order:
        1. Direct IP detection via get_ip()
        2. Network interface enumeration via get_local_ip_by_nic()
        3. Remote connection method via get_local_ip_by_remote()
    """
    # Try environment variable
    host_ip = os.getenv("SGLANG_HOST_IP", "") or os.getenv("HOST_IP", "")
    if host_ip:
# ... omitted for brevity ...
    if ip := get_local_ip_by_remote():
        return ip
    logger.debug("get_local_ip_by_remote failed")
    if fallback:
        return fallback
    raise ValueError("Can not get local ip")
```
**EN:** This function implements `get_local_ip_auto`. It primarily calls `logger.debug`, `ValueError`, `os.getenv`, `get_local_ip_by_nic`, `get_local_ip_by_remote` to complete its work. State updates are written into `host_ip`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_local_ip_auto`。 它主要通过调用 `logger.debug`, `ValueError`, `os.getenv`, `get_local_ip_by_nic`, `get_local_ip_by_remote` 来完成任务。 状态更新主要写入 `host_ip`。 实现中使用了条件分支。

### Lines 351-388: Function `get_zmq_socket` / 函数 `get_zmq_socket`
```python
def get_zmq_socket(
    context: zmq.Context,
    socket_type: zmq.SocketType,
    endpoint: Optional[str] = None,
    bind: bool = True,
) -> Union[zmq.Socket, Tuple[int, zmq.Socket]]:
    """Create and configure a ZeroMQ socket.

    Args:
        context: ZeroMQ context to create the socket from.
        socket_type: Type of ZeroMQ socket to create.
        endpoint: Optional endpoint to bind/connect to. If None, binds to a random TCP port.
        bind: Whether to bind (True) or connect (False) to the endpoint. Ignored if endpoint is None.

    Returns:
        If endpoint is None: Tuple of (port, socket) where port is the randomly assigned TCP port.
        If endpoint is provided: The configured ZeroMQ socket.
    """
    socket = context.socket(socket_type)

    if endpoint is None:
        # Bind to random TCP port
        config_socket(socket, socket_type)
        port = socket.bind_to_random_port("tcp://*")
        return port, socket
    else:
        # Handle IPv6 if endpoint contains brackets
        if endpoint.find("[") != -1:
# ... omitted for brevity ...
        if bind:
            socket.bind(endpoint)
        else:
            socket.connect(endpoint)

        return socket
```
**EN:** This function implements `get_zmq_socket`. It primarily calls `context.socket`, `config_socket`, `socket.bind_to_random_port`, `endpoint.find`, `socket.setsockopt`, `socket.bind` to complete its work. State updates are written into `socket`, `port`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `get_zmq_socket`。 它主要通过调用 `context.socket`, `config_socket`, `socket.bind_to_random_port`, `endpoint.find`, `socket.setsockopt`, `socket.bind` 来完成任务。 状态更新主要写入 `socket`, `port`。 实现中使用了条件分支。

### Lines 391-397: Function `_is_ipv6` / 函数 `_is_ipv6`
```python
def _is_ipv6(host: str) -> bool:
    """Check whether *host* is a valid IPv6 address (without brackets)."""
    try:
        ipaddress.IPv6Address(host)
        return True
    except ValueError:
        return False
```
**EN:** This function implements `_is_ipv6`. It primarily calls `ipaddress.IPv6Address` to complete its work. The implementation relies on error handling.
**CN:** 该函数实现了 `_is_ipv6`。 它主要通过调用 `ipaddress.IPv6Address` 来完成任务。 实现中使用了错误处理。

### Lines 400-402: Function `_wrap` / 函数 `_wrap`
```python
def _wrap(host: str) -> str:
    """Wrap an IPv6 address in brackets; pass IPv4/hostname through."""
    return f"[{host}]" if _is_ipv6(host) else host
```
**EN:** This function implements `_wrap`. It primarily calls `_is_ipv6` to complete its work.
**CN:** 该函数实现了 `_wrap`。 它主要通过调用 `_is_ipv6` 来完成任务。

### Lines 405-412: Function `_parse_port` / 函数 `_parse_port`
```python
def _parse_port(s: str) -> int:
    try:
        port = int(s)
    except ValueError:
        raise ValueError(f"Invalid port number: {s!r}")
    if not (0 <= port <= 65535):
        raise ValueError(f"Port out of range (0-65535): {port}")
    return port
```
**EN:** This function implements `_parse_port`. It primarily calls `int`, `ValueError` to complete its work. State updates are written into `port`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_parse_port`。 它主要通过调用 `int`, `ValueError` 来完成任务。 状态更新主要写入 `port`。 实现中使用了条件分支、错误处理。

### Lines 415-418: Class `NetworkAddress` declaration / 类 `NetworkAddress` 声明
```python
@dataclass(frozen=True)
class NetworkAddress:
    host: str
    port: int
```
**EN:** This class establishes `NetworkAddress` as a compact data container for the surrounding logic. Its core interface includes methods such as `__post_init__`, `is_ipv6`, `family`, `to_url`, `to_tcp`, `to_host_port_str`.
**CN:** 该类将 `NetworkAddress` 定义为周边逻辑的紧凑的数据容器。 其核心接口包括 `__post_init__`, `is_ipv6`, `family`, `to_url`, `to_tcp`, `to_host_port_str` 等方法。

### Lines 420-423: Method `NetworkAddress.__post_init__` / 方法 `NetworkAddress.__post_init__`
```python
    def __post_init__(self):
        # Auto-strip IPv6 brackets so callers can pass "[::1]" or "::1"
        if self.host.startswith("[") and self.host.endswith("]"):
            object.__setattr__(self, "host", self.host[1:-1])
```
**EN:** This method implements `__post_init__` on `NetworkAddress`. It primarily calls `self.host.startswith`, `self.host.endswith`, `object.__setattr__` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `__post_init__`。 它主要通过调用 `self.host.startswith`, `self.host.endswith`, `object.__setattr__` 来完成任务。 实现中使用了条件分支。

### Lines 425-427: Method `NetworkAddress.is_ipv6` / 方法 `NetworkAddress.is_ipv6`
```python
    @property
    def is_ipv6(self) -> bool:
        return _is_ipv6(self.host)
```
**EN:** This method implements `is_ipv6` on `NetworkAddress`. It primarily calls `_is_ipv6` to complete its work.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `is_ipv6`。 它主要通过调用 `_is_ipv6` 来完成任务。

### Lines 429-431: Method `NetworkAddress.family` / 方法 `NetworkAddress.family`
```python
    @property
    def family(self) -> socket.AddressFamily:
        return socket.AF_INET6 if self.is_ipv6 else socket.AF_INET
```
**EN:** This method implements `family` on `NetworkAddress`.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `family`。

### Lines 433-435: Method `NetworkAddress.to_url` / 方法 `NetworkAddress.to_url`
```python
    def to_url(self, scheme: str = "http") -> str:
        """``http://127.0.0.1:30000`` or ``http://[::1]:30000``."""
        return f"{scheme}://{_wrap(self.host)}:{self.port}"
```
**EN:** This method implements `to_url` on `NetworkAddress`. It primarily calls `_wrap` to complete its work.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `to_url`。 它主要通过调用 `_wrap` 来完成任务。

### Lines 437-439: Method `NetworkAddress.to_tcp` / 方法 `NetworkAddress.to_tcp`
```python
    def to_tcp(self) -> str:
        """``tcp://`` endpoint for ZMQ / torch distributed."""
        return self.to_url("tcp")
```
**EN:** This method implements `to_tcp` on `NetworkAddress`. It primarily calls `self.to_url` to complete its work.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `to_tcp`。 它主要通过调用 `self.to_url` 来完成任务。

### Lines 441-443: Method `NetworkAddress.to_host_port_str` / 方法 `NetworkAddress.to_host_port_str`
```python
    def to_host_port_str(self) -> str:
        """``host:port`` string for gRPC listen address, session IDs, logs."""
        return f"{_wrap(self.host)}:{self.port}"
```
**EN:** This method implements `to_host_port_str` on `NetworkAddress`. It primarily calls `_wrap` to complete its work.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `to_host_port_str`。 它主要通过调用 `_wrap` 来完成任务。

### Lines 445-458: Method `NetworkAddress.resolve_host` / 方法 `NetworkAddress.resolve_host`
```python
    @staticmethod
    def resolve_host(host: str) -> str:
        """Return *host* as-is if it's an IP, otherwise DNS-resolve to one."""
        try:
            ipaddress.ip_address(host)
            return host
        except ValueError:
            pass
        try:
            return socket.getaddrinfo(
                host, None, socket.AF_UNSPEC, 0, 0, socket.AI_ADDRCONFIG
            )[0][4][0]
        except socket.gaierror as e:
            raise ValueError(f"Cannot resolve host {host!r}: {e}") from e
```
**EN:** This method implements `resolve_host` on `NetworkAddress`. It primarily calls `ipaddress.ip_address`, `ValueError`, `socket.getaddrinfo` to complete its work. The implementation relies on error handling.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `resolve_host`。 它主要通过调用 `ipaddress.ip_address`, `ValueError`, `socket.getaddrinfo` 来完成任务。 实现中使用了错误处理。

### Lines 460-463: Method `NetworkAddress.resolved` / 方法 `NetworkAddress.resolved`
```python
    def resolved(self) -> NetworkAddress:
        """DNS-resolve hostname to IP; return self if already an IP."""
        ip = self.resolve_host(self.host)
        return self if ip == self.host else NetworkAddress(ip, self.port)
```
**EN:** This method implements `resolved` on `NetworkAddress`. It primarily calls `self.resolve_host`, `NetworkAddress` to complete its work. State updates are written into `ip`.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `resolved`。 它主要通过调用 `self.resolve_host`, `NetworkAddress` 来完成任务。 状态更新主要写入 `ip`。

### Lines 465-471: Method `NetworkAddress.to_bind_tuple` / 方法 `NetworkAddress.to_bind_tuple`
```python
    def to_bind_tuple(self) -> Tuple[str, int]:
        """Raw ``(host, port)`` tuple for ``socket.bind()`` / ``socket.connect()``.

        Returns the *unwrapped* host — sockets need the raw address, not
        the bracketed form.
        """
        return (self.host, self.port)
```
**EN:** This method implements `to_bind_tuple` on `NetworkAddress`.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `to_bind_tuple`。

### Lines 473-519: Method `NetworkAddress.parse` / 方法 `NetworkAddress.parse`
```python
    @staticmethod
    def parse(addr: str) -> NetworkAddress:
        """Parse a ``host:port`` string into a ``NetworkAddress``.

        Accepted formats::

            [::1]:8000          → NetworkAddress("::1", 8000)
            127.0.0.1:8000      → NetworkAddress("127.0.0.1", 8000)
            my-hostname:8000    → NetworkAddress("my-hostname", 8000)

        IPv6 addresses **must** be bracketed.  Bare ``::1:8000`` is
        ambiguous and will raise ``ValueError``.

        Raises:
            ValueError: If the string cannot be unambiguously parsed.
        """
        if not addr:
            raise ValueError("Empty address string")

        # --- Bracketed IPv6: [addr]:port ---
        if addr.startswith("["):
            close = addr.find("]")
            if close == -1:
                raise ValueError(f"Missing closing bracket in IPv6 address: {addr!r}")
            host = addr[1:close]
            if not _is_ipv6(host):
                raise ValueError(f"Invalid IPv6 address inside brackets: {host!r}")
            rest = addr[close + 1 :]
# ... omitted for brevity ...
        if ":" in host and _is_ipv6(host):
            raise ValueError(
                f"Bare IPv6 address without brackets is ambiguous: {addr!r}. "
                f"Use [{host}]:{port_str} instead."
            )
        return NetworkAddress(host, _parse_port(port_str))
```
**EN:** This method implements `parse` on `NetworkAddress`. It primarily calls `addr.startswith`, `addr.rsplit`, `NetworkAddress`, `ValueError`, `addr.find`, `_is_ipv6` to complete its work. State updates are written into `close`, `host`, `rest`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `parse`。 它主要通过调用 `addr.startswith`, `addr.rsplit`, `NetworkAddress`, `ValueError`, `addr.find`, `_is_ipv6` 来完成任务。 状态更新主要写入 `close`, `host`, `rest`。 实现中使用了条件分支。

### Lines 521-522: Method `NetworkAddress.__str__` / 方法 `NetworkAddress.__str__`
```python
    def __str__(self) -> str:
        return self.to_host_port_str()
```
**EN:** This method implements `__str__` on `NetworkAddress`. It primarily calls `self.to_host_port_str` to complete its work.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `__str__`。 它主要通过调用 `self.to_host_port_str` 来完成任务。

### Lines 524-525: Method `NetworkAddress.__repr__` / 方法 `NetworkAddress.__repr__`
```python
    def __repr__(self) -> str:
        return f"NetworkAddress({self.host!r}, {self.port})"
```
**EN:** This method implements `__repr__` on `NetworkAddress`.
**CN:** 该方法（属于 `NetworkAddress`）实现了 `__repr__`。

## Key Concepts / 关键概念
- **Classes / 类**: `NetworkAddress`
- **Functions / 函数**: `get_open_port`, `is_valid_ipv6_address`, `find_process_using_port`, `wait_port_available`, `_get_addrinfos_for_bind`, `try_bind_socket`, `is_port_available`, `get_free_port`
- **Themes / 主题**: `network`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `psutil`, `zmq`, `netifaces`
- **Standard library / 标准库**: `__future__`, `ipaddress`, `logging`, `os`, `socket`, `time`, `dataclasses`, `typing`
