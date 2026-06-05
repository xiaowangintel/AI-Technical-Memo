# launch_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/bindings/python/src/sglang_router/launch_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements functions `setup_logger`, `run_server`, `launch_server_process`, `wait_for_server_health`, and 3 more for the Python router binding. Other gateway components import it to reuse configuration, control flow, or shared utilities. / 该模块为 Python 路由绑定层 实现了 函数 `setup_logger`, `run_server`, `launch_server_process`, `wait_for_server_health`, and 3 more，其他网关组件会导入它以复用配置、控制流程或共享工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and dependencies
```python
import argparse
import asyncio
import copy
import logging
import multiprocessing as mp
import os
import random
import signal
import sys
import time
from typing import List

import requests
from setproctitle import setproctitle
from sglang_router.launch_router import RouterArgs, launch_router

from sglang.srt.server_args import ServerArgs
from sglang.srt.utils.network import is_port_available
```
**EN:** This block imports `argparse`, `asyncio`, `copy`, `logging`, and 12 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 21-34: function `setup_logger`
```python
def setup_logger():
    logger = logging.getLogger("router")
    logger.setLevel(logging.INFO)

    formatter = logging.Formatter(
        "[Router (Python)] %(asctime)s - %(levelname)s - %(message)s - %(filename)s:%(lineno)d",
        datefmt="%Y-%m-%d %H:%M:%S",
    )

    handler = logging.StreamHandler()
    handler.setFormatter(formatter)
    logger.addHandler(handler)

    return logger
```
**EN:** function `setup_logger` encapsulates `setup logger` logic for callers in this module.
**CN:** 函数 `setup_logger` 封装了与 `setup logger` 相关的逻辑，供本模块中的调用方复用。

### Lines 37-37: Module constants and configuration
```python
logger = setup_logger()
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 38-83: function `run_server`
```python


# Create new process group
def run_server(server_args, dp_rank):
    """
    Note:

    1. Without os.setpgrp(), all processes share the same PGID. When you press Ctrl+C, the terminal sends SIGINT to all processes in the group simultaneously.
    This can cause leaf processes to terminate first, which messes up the cleaning order and produces orphaned processes.

    Terminal (PGID=100)
    └── Main Python Process (PGID=100)
        └── Server Process 1 (PGID=100)
            └── Scheduler 1
            └── Detokenizer 1
        └── Server Process 2 (PGID=100)
            └── Scheduler 2
            └── Detokenizer 2

    2. With os.setpgrp(), the main Python process and its children are in a separate group. Now:

    Terminal (PGID=100)
    └── Main Python Process (PGID=200)
        └── Server Process 1 (PGID=300)
            └── Scheduler 1
            └── Detokenizer 1
        └── Server Process 2 (PGID=400)
            └── Scheduler 2
            └── Detokenizer 2
    """
    # create new process group
    os.setpgrp()

    setproctitle("sglang::server")
    # Set SGLANG_DP_RANK environment variable
    os.environ["SGLANG_DP_RANK"] = str(dp_rank)

    # Launch server in appropriate mode (HTTP or gRPC)
    if server_args.grpc_mode:
        from sglang.srt.entrypoints.grpc_server import serve_grpc

        asyncio.run(serve_grpc(server_args))
    else:
        from sglang.srt.entrypoints.http_server import launch_server

        launch_server(server_args)
```
**EN:** function `run_server` is described as: Note: 1.
**CN:** 函数 `run_server` 封装了与 `run server` 相关的逻辑，供本模块中的调用方复用。

### Lines 86-97: function `launch_server_process`
```python
def launch_server_process(
    server_args: ServerArgs, worker_port: int, dp_id: int
) -> mp.Process:
    """Launch a single server process with the given args and port."""
    server_args = copy.deepcopy(server_args)
    server_args.port = worker_port
    server_args.base_gpu_id = dp_id * server_args.tp_size
    server_args.dp_size = 1

    proc = mp.Process(target=run_server, args=(server_args, dp_id))
    proc.start()
    return proc
```
**EN:** function `launch_server_process` is described as: Launch a single server process with the given args and port.
**CN:** 函数 `launch_server_process` 封装了与 `launch server process` 相关的逻辑，供本模块中的调用方复用。

### Lines 100-113: function `wait_for_server_health`
```python
def wait_for_server_health(host: str, port: int, timeout: int = 300) -> bool:
    """Wait for server to be healthy by checking /health endpoint."""
    start_time = time.perf_counter()
    url = f"http://{host}:{port}/health"

    while time.perf_counter() - start_time < timeout:
        try:
            response = requests.get(url, timeout=5)
            if response.status_code == 200:
                return True
        except requests.exceptions.RequestException:
            pass
        time.sleep(1)
    return False
```
**EN:** function `wait_for_server_health` is described as: Wait for server to be healthy by checking /health endpoint.
**CN:** 函数 `wait_for_server_health` 封装了与 `wait for server health` 相关的逻辑，供本模块中的调用方复用。

### Lines 116-126: function `find_available_ports`
```python
def find_available_ports(base_port: int, count: int) -> List[int]:
    """Find consecutive available ports starting from base_port."""
    available_ports = []
    current_port = base_port

    while len(available_ports) < count:
        if is_port_available(current_port):
            available_ports.append(current_port)
        current_port += random.randint(100, 1000)

    return available_ports
```
**EN:** function `find_available_ports` is described as: Find consecutive available ports starting from base_port.
**CN:** 函数 `find_available_ports` 封装了与 `find available ports` 相关的逻辑，供本模块中的调用方复用。

### Lines 129-150: function `cleanup_processes`
```python
def cleanup_processes(processes: List[mp.Process]):
    for process in processes:
        logger.info(f"Terminating process group {process.pid}")
        try:
            os.killpg(process.pid, signal.SIGTERM)
        except ProcessLookupError:
            # Process group may already be terminated
            pass

    # Wait for processes to terminate
    for process in processes:
        process.join(timeout=5)
        if process.is_alive():
            logger.warning(
                f"Process {process.pid} did not terminate gracefully, forcing kill"
            )
            try:
                os.killpg(process.pid, signal.SIGKILL)
            except ProcessLookupError:
                pass

    logger.info("All process groups terminated")
```
**EN:** function `cleanup_processes` encapsulates `cleanup processes` logic for callers in this module.
**CN:** 函数 `cleanup_processes` 封装了与 `cleanup processes` 相关的逻辑，供本模块中的调用方复用。

### Lines 153-209: function `main`
```python
def main():
    # CUDA runtime isn't fork-safe, which can lead to subtle bugs or crashes
    mp.set_start_method("spawn")

    parser = argparse.ArgumentParser(
        description="Launch SGLang router and server processes"
    )

    ServerArgs.add_cli_args(parser)
    RouterArgs.add_cli_args(parser, use_router_prefix=True, exclude_host_port=True)
    parser.add_argument(
        "--router-dp-worker-base-port",
        type=int,
        default=31000,
        help="Base port number for data parallel workers",
    )
    # No extra retry/CB flags here; RouterArgs.add_cli_args already defines them with router- prefix

    args = parser.parse_args()
    server_args = ServerArgs.from_cli_args(args)
    router_args = RouterArgs.from_cli_args(args, use_router_prefix=True)

    # Find available ports for workers
    worker_ports = find_available_ports(
        args.router_dp_worker_base_port, server_args.dp_size
    )

    # Start server processes
    server_processes = []

    for i, worker_port in enumerate(worker_ports):
        logger.info(f"Launching DP server process {i} on port {worker_port}")
        proc = launch_server_process(server_args, worker_port, i)
        server_processes.append(proc)

    signal.signal(signal.SIGINT, lambda sig, frame: cleanup_processes(server_processes))
    signal.signal(
        signal.SIGTERM, lambda sig, frame: cleanup_processes(server_processes)
    )
    signal.signal(
        signal.SIGQUIT, lambda sig, frame: cleanup_processes(server_processes)
    )

    # Update router args with worker URLs
    # Use grpc:// protocol if server is in gRPC mode, otherwise http://
    protocol = "grpc" if server_args.grpc_mode else "http"
    router_args.worker_urls = [
        f"{protocol}://{server_args.host}:{port}" for port in worker_ports
    ]

    # Start the router
    try:
        launch_router(router_args)
    except Exception as e:
        logger.error(f"Failed to start router: {e}")
        cleanup_processes(server_processes)
        sys.exit(1)
```
**EN:** function `main` encapsulates `main` logic for callers in this module.
**CN:** 函数 `main` 封装了与 `main` 相关的逻辑，供本模块中的调用方复用。

### Lines 212-213: Command-line entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This main-guard block makes the file executable as a script by invoking the module's top-level routine only when run directly.
**CN:** 这个主入口保护块让文件可以作为脚本直接执行，并仅在被直接运行时调用模块的顶层流程。

## Key Concepts / 关键概念
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Asynchronous coordination / 异步协调
- Command-line argument parsing / 命令行参数解析
- Reusable functions: `setup_logger`, `run_server`, `launch_server_process`, `wait_for_server_health`, and 3 more / 可复用函数：`setup_logger`, `run_server`, `launch_server_process`, `wait_for_server_health`, and 3 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `copy`, `logging`, `multiprocessing`, `os`, `random`, `signal`, `sys`, `time`, `typing`
- **Third-party / 第三方**: `requests`, `setproctitle`, `sglang.srt.entrypoints.grpc_server`, `sglang.srt.entrypoints.http_server`, `sglang.srt.server_args`, `sglang.srt.utils.network`
- **Internal / 内部模块**: `sglang_router.launch_router`
