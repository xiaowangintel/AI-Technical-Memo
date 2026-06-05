# disagg_proxy_demo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/disaggregated_serving/disagg_proxy_demo.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This file provides a disaggregated prefilling proxy demo to demonstrate an example usage of XpYd disaggregated prefilling / 演示解耦服务相关的基础组件。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This file provides a disaggregated prefilling proxy demo to demonstrate an
example usage of XpYd disaggregated prefilling.
We can launch multiple vllm instances (2 for prefill and 2 for decode), and
launch this proxy demo through:
  python3 examples/disaggregated/disaggregated_serving/disagg_proxy_demo.py  \
       --model $model_name  \
       --prefill localhost:8100 localhost:8101   \
       --decode localhost:8200 localhost:8201   \
       --port 8000

Note: This demo will be removed once the PDController implemented in PR 15343
(https://github.com/vllm-project/vllm/pull/15343) supports XpYd.
"""
```
**EN:** This file provides a disaggregated prefilling proxy demo to demonstrate an example usage of XpYd disaggregated prefilling.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import ipaddress
import itertools
import json
import logging
import os
import sys
from abc import ABC, abstractmethod
from collections.abc import Callable

import aiohttp
import requests
import uvicorn
from fastapi import APIRouter, Depends, FastAPI, Header, HTTPException, Request, status
from fastapi.responses import JSONResponse, StreamingResponse
```
**EN:** This block loads helper libraries such as argparse, ipaddress, itertools, json, and logging.
**CN:** 这一部分加载 argparse、ipaddress、itertools、json，以及 logging 等辅助库。

### Top-level setup
```python
AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=6 * 60 * 60)
logger = logging.getLogger()
logging.basicConfig(level=logging.INFO)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as AIOHTTP_TIMEOUT and logger. It also performs early helper calls such as aiohttp.ClientTimeout, logging.getLogger, and logging.basicConfig.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 AIOHTTP_TIMEOUT 和 logger 等变量。它还会提前执行 aiohttp.ClientTimeout、logging.getLogger，以及 logging.basicConfig 等辅助调用。

### Class: SchedulingPolicy
```python
class SchedulingPolicy(ABC):
    @abstractmethod
    def schedule(self, cycler: itertools.cycle):
        raise NotImplementedError("Scheduling Proxy is not set.")
```
**EN:** This class packages the SchedulingPolicy abstraction used by the example. It extends ABC. Notable methods include schedule.
**CN:** 该类封装了示例中使用的 SchedulingPolicy 抽象。它继承自 ABC。较重要的方法包括 schedule。

### Class: Proxy
```python
class Proxy:
    def __init__(
        self,
        prefill_instances: list[str],
        decode_instances: list[str],
        model: str,
        scheduling_policy: SchedulingPolicy,
        custom_create_completion: Callable[[Request], StreamingResponse] | None = None,
        custom_create_chat_completion: Callable[[Request], StreamingResponse]
        | None = None,
    ):
        self.prefill_instances = prefill_instances
        self.decode_instances = decode_instances
        self.prefill_cycler = itertools.cycle(prefill_instances)
        self.decode_cycler = itertools.cycle(decode_instances)
        self.model = model
        self.scheduling_policy = scheduling_policy
        self.custom_create_completion = custom_create_completion
        self.custom_create_chat_completion = custom_create_chat_completion
        self.router = APIRouter()
    # ... key logic omitted for brevity ...
            error_messages = [str(e) for e in exc_info if e]
            print("Error occurred in disagg proxy server")
            print(error_messages)
            return StreamingResponse(
                content=iter(error_messages), media_type="text/event-stream"
            )

    def remove_instance_endpoint(self, instance_type, instance):
        if instance_type == "decode" and instance in self.decode_instances:
            self.decode_instances.remove(instance)
            self.decode_cycler = itertools.cycle(self.decode_instances)
        if instance_type == "prefill" and instance in self.prefill_instances:
            self.prefill_instances.remove(instance)
            self.prefill_cycler = itertools.cycle(self.prefill_instances)
```
**EN:** This class packages the Proxy abstraction used by the example. Notable methods include __init__, setup_routes, validate_json_request, api_key_authenticate, validate_instance, add_instance_endpoint, forward_request, and schedule.
**CN:** 该类封装了示例中使用的 Proxy 抽象。较重要的方法包括 __init__、setup_routes、validate_json_request、api_key_authenticate、validate_instance、add_instance_endpoint、forward_request，以及 schedule。

### Class: RoundRobinSchedulingPolicy
```python
class RoundRobinSchedulingPolicy(SchedulingPolicy):
    def __init__(self):
        super().__init__()

    def schedule(self, cycler: itertools.cycle) -> str:
        return next(cycler)
```
**EN:** This class packages the RoundRobinSchedulingPolicy abstraction used by the example. It extends SchedulingPolicy. Notable methods include __init__ and schedule.
**CN:** 该类封装了示例中使用的 RoundRobinSchedulingPolicy 抽象。它继承自 SchedulingPolicy。较重要的方法包括 __init__ 和 schedule。

### Class: ProxyServer
```python
class ProxyServer:
    def __init__(
        self,
        args: argparse.Namespace,
        scheduling_policy: SchedulingPolicy | None = None,
        create_completion: Callable[[Request], StreamingResponse] | None = None,
        create_chat_completion: Callable[[Request], StreamingResponse] | None = None,
    ):
        self.validate_parsed_serve_args(args)
        self.port = args.port
        self.proxy_instance = Proxy(
            prefill_instances=[] if args.prefill is None else args.prefill,
            decode_instances=[] if args.decode is None else args.decode,
            model=args.model,
            scheduling_policy=(
                scheduling_policy
                if scheduling_policy is not None
                else RoundRobinSchedulingPolicy()
            ),
            custom_create_completion=create_completion,
    # ... key logic omitted for brevity ...
                        )
                else:
                    raise ValueError(f"Cannot get model id from {instance}!")
            except requests.RequestException as e:
                raise ValueError(
                    f"Error communicating with {instance}: {str(e)}"
                ) from e

    def run_server(self):
        app = FastAPI()
        app.include_router(self.proxy_instance.router)
        config = uvicorn.Config(app, port=self.port, loop="uvloop")
        server = uvicorn.Server(config)
        server.run()
```
**EN:** This class packages the ProxyServer abstraction used by the example. Notable methods include __init__, validate_parsed_serve_args, validate_instances, verify_model_config, and run_server.
**CN:** 该类封装了示例中使用的 ProxyServer 抽象。较重要的方法包括 __init__、validate_parsed_serve_args、validate_instances、verify_model_config，以及 run_server。

### Function: parse_args
```python
def parse_args():
    # Todo: allow more config
    parser = argparse.ArgumentParser("vLLM disaggregated proxy server.")
    parser.add_argument("--model", "-m", type=str, required=True, help="Model name")

    parser.add_argument(
        "--prefill",
        "-p",
        type=str,
        nargs="+",
        help="List of prefill node URLs (host:port)",
    )

    parser.add_argument(
        "--decode",
        "-d",
        type=str,
        nargs="+",
        help="List of decode node URLs (host:port)",
    )

    parser.add_argument(
        "--port",
        type=int,
        default=8000,
        help="Server port number",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    proxy_server = ProxyServer(args=args)
    proxy_server.run_server()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args, ProxyServer, and proxy_server.run_server.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args、ProxyServer，以及 proxy_server.run_server。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Interactive application wrappers / 交互式应用封装**: UI or service layers turn model calls into an interactive user experience. / 界面或服务层把模型调用包装成交互式用户体验。
- **Tool calling / 工具调用**: Model outputs are mapped to callable tools or structured action requests. / 模型输出会映射为可调用工具或结构化动作请求。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `ipaddress`, `itertools`, `json`, `logging`, `os`, `sys`, `abc` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `SchedulingPolicy`, `Proxy`, `RoundRobinSchedulingPolicy`, `ProxyServer`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `aiohttp.ClientTimeout`, `logging.getLogger`, `logging.basicConfig`, `NotImplementedError`, `HTTPException`, `str`, `logger.error`, `itertools.cycle` reveal the main execution path / 这些调用体现了主要执行链路。
