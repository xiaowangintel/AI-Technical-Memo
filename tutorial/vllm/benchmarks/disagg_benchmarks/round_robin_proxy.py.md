# round_robin_proxy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/disagg_benchmarks/round_robin_proxy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, disaggregated serving experiments, centered around `RoundRobinProxy`, `main`. / 实现与基准测试编排、解耦服务实验相关的逻辑，核心符号包括 `RoundRobinProxy`, `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-8)
```python
import asyncio
import itertools

import aiohttp
from aiohttp import web
```
**EN:** This block gathers standard-library helpers such as `asyncio`, `itertools`; third-party packages such as `aiohttp`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `asyncio`, `itertools`；第三方依赖，如 `aiohttp`。这些依赖构成了后续基准测试逻辑的基础。

### Class `RoundRobinProxy` (lines 11-43)
```python
class RoundRobinProxy:
    def __init__(self, target_ports):
        self.target_ports = target_ports
        self.port_cycle = itertools.cycle(self.target_ports)

    async def handle_request(self, request):
        target_port = next(self.port_cycle)
        target_url = f"http://localhost:{target_port}{request.path_qs}"

        async with aiohttp.ClientSession() as session:
            try:
                # Forward the request
                async with session.request(
                    method=request.method,
                    url=target_url,
                    headers=request.headers,
                    data=request.content,
                ) as response:
                    # Start sending the response
                    resp = web.StreamResponse(
                        status=response.status, headers=response.headers
                    )
                    await resp.prepare(request)

                    # Stream the response content
                    async for chunk in response.content.iter_any():
                        await resp.write(chunk)

                    await resp.write_eof()
                    return resp

            except Exception as e:
                return web.Response(text=f"Error: {str(e)}", status=500)
```
**EN:** Class `RoundRobinProxy` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `handle_request`.
**CN:** 类 `RoundRobinProxy` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `handle_request` 等方法。

### Method `RoundRobinProxy.__init__` (lines 12-14)
```python
    def __init__(self, target_ports):
        self.target_ports = target_ports
        self.port_cycle = itertools.cycle(self.target_ports)
```
**EN:** `__init__` implements a helper used by `round_robin_proxy.py`. It mainly works with `target_ports` and relies on `itertools.cycle` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `round_robin_proxy.py` 使用的辅助逻辑。 它主要处理 `target_ports`，并结合 `itertools.cycle` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `RoundRobinProxy.handle_request` (lines 16-43)
```python
    async def handle_request(self, request):
        target_port = next(self.port_cycle)
        target_url = f"http://localhost:{target_port}{request.path_qs}"

        async with aiohttp.ClientSession() as session:
            try:
                # Forward the request
                async with session.request(
                    method=request.method,
                    url=target_url,
                    headers=request.headers,
                    data=request.content,
                ) as response:
                    # Start sending the response
                    resp = web.StreamResponse(
                        status=response.status, headers=response.headers
                    )
                    await resp.prepare(request)

                    # Stream the response content
                    async for chunk in response.content.iter_any():
                        await resp.write(chunk)

                    await resp.write_eof()
                    return resp

            except Exception as e:
                return web.Response(text=f"Error: {str(e)}", status=500)
```
**EN:** `handle_request` implements backend request helpers. It mainly works with `request` and relies on `next`, `aiohttp.ClientSession`, `session.request`, `web.StreamResponse`, `resp.prepare`, `response.content.iter_any` plus iteration, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `handle_request` 负责实现后端请求辅助逻辑。 它主要处理 `request`，并结合 `next`, `aiohttp.ClientSession`, `session.request`, `web.StreamResponse`, `resp.prepare`, `response.content.iter_any` 以及 循环迭代、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `main` (lines 46-59)
```python
async def main():
    proxy = RoundRobinProxy([8100, 8200])
    app = web.Application()
    app.router.add_route("*", "/{path:.*}", proxy.handle_request)

    runner = web.AppRunner(app)
    await runner.setup()
    site = web.TCPSite(runner, "localhost", 8000)
    await site.start()

    print("Proxy server started on http://localhost:8000")

    # Keep the server running
    await asyncio.Event().wait()
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `RoundRobinProxy`, `web.Application`, `app.router.add_route`, `web.AppRunner`, `runner.setup`, `web.TCPSite` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `RoundRobinProxy`, `web.Application`, `app.router.add_route`, `web.AppRunner`, `runner.setup`, `web.TCPSite` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Entry point (lines 62-63)
```python
if __name__ == "__main__":
    asyncio.run(main())
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `asyncio.run`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `asyncio.run`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。

## Dependencies / 依赖关系
- **EN:** Standard library: `asyncio`, `itertools`.
- **CN:** 标准库依赖：`asyncio`, `itertools`。
- **EN:** Third-party packages: `aiohttp`.
- **CN:** 第三方依赖：`aiohttp`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
