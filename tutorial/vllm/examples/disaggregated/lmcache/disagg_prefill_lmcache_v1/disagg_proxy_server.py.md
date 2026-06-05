# disagg_proxy_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/lmcache/disagg_prefill_lmcache_v1/disagg_proxy_server.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates lmcache-backed disaggregated serving and cache reuse. / 演示基于 LMCache 的解耦服务与缓存复用。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
import argparse
import os
import time
from contextlib import asynccontextmanager

import httpx
import numpy as np
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
```
**EN:** This block loads helper libraries such as argparse, os, time, contextlib, and httpx.
**CN:** 这一部分加载 argparse、os、time、contextlib，以及 httpx 等辅助库。

### Async function: lifespan
```python
async def lifespan(app: FastAPI):
    """
    Lifespan context manager to handle startup and shutdown events.
    """
    # Startup: Initialize clients
    prefiller_base_url = (
        f"http://{global_args.prefiller_host}:{global_args.prefiller_port}/v1"
    )
    decoder_base_url = (
        f"http://{global_args.decoder_host}:{global_args.decoder_port}/v1"
    )

    app.state.prefill_client = httpx.AsyncClient(
        timeout=None,
        base_url=prefiller_base_url,
        limits=httpx.Limits(
            max_connections=None,
            max_keepalive_connections=None,
        ),
    )
    app.state.decode_client = httpx.AsyncClient(
        timeout=None,
        base_url=decoder_base_url,
        limits=httpx.Limits(
            max_connections=None,
            max_keepalive_connections=None,
        ),
    )

    yield

    # Shutdown: Close clients
    await app.state.prefill_client.aclose()
    await app.state.decode_client.aclose()
```
**EN:** Lifespan context manager to handle startup and shutdown events.. It works with parameters such as app. Key operations include httpx.AsyncClient, httpx.Limits, app.state.prefill_client.aclose, and app.state.decode_client.aclose. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式在执行核心任务时持续产出增量结果。它会处理 app 等参数。关键操作包括 httpx.AsyncClient、httpx.Limits、app.state.prefill_client.aclose，以及 app.state.decode_client.aclose。它会逐步产出数据，而不是只等待单次最终结果。

### Top-level setup
```python
app = FastAPI(lifespan=lifespan)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as app. It also performs early helper calls such as FastAPI.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 app 等变量。它还会提前执行 FastAPI 等辅助调用。

### Class: StatsCalculator
```python
class StatsCalculator:
    def __init__(self):
        self._stats = []
        self._last_log_time = time.time()

    def add(self, value):
        self._stats.append(value)
        if time.time() - self._last_log_time > 5:
            self._log_stats()
            self._last_log_time = time.time()

    def _log_stats(self):
        # Print average, median, and 99th percentile
        np_arr = np.array(self._stats)
        output_str = (
            f"\nNum requests: {len(self._stats)}"
            "\nPrefill node TTFT stats:"
            f"\n - Average (ms): {np.mean(np_arr)}"
            f"\n - Median (ms): {np.median(np_arr)}"
            f"\n - 99th Percentile (ms): {np.percentile(np_arr, 99)}\n"
        )
        print(
            "===============================",
            output_str,
            "===============================",
        )
```
**EN:** This class packages the StatsCalculator abstraction used by the example. Notable methods include __init__, add, and _log_stats.
**CN:** 该类封装了示例中使用的 StatsCalculator 抽象。较重要的方法包括 __init__、add，以及 _log_stats。

### Top-level setup
```python
stats_calculator = StatsCalculator()
counter = 0
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as stats_calculator and counter. It also performs early helper calls such as StatsCalculator.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 stats_calculator 和 counter 等变量。它还会提前执行 StatsCalculator 等辅助调用。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument("--port", type=int, default=8000)
    parser.add_argument("--host", type=str, default="localhost")
    parser.add_argument("--prefiller-host", type=str, default="localhost")
    parser.add_argument("--prefiller-port", type=int, default=8100)
    parser.add_argument("--decoder-host", type=str, default="localhost")
    parser.add_argument("--decoder-port", type=int, default=8200)
    args = parser.parse_args()
    return args
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
app.state.prefill_client = None
app.state.decode_client = None
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。

### Async function: send_request_to_service
```python
async def send_request_to_service(
    client: httpx.AsyncClient, endpoint: str, req_data: dict
):
    """
    Send a request to a service using a persistent client.
    """
    req_data = req_data.copy()
    req_data["max_tokens"] = 1
    if "max_completion_tokens" in req_data:
        req_data["max_completion_tokens"] = 1

    headers = {"Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}"}
    response = await client.post(endpoint, json=req_data, headers=headers)
    response.raise_for_status()

    # read/consume the response body to release the connection
    # otherwise, it would http.ReadError
    await response.aread()

    return response
```
**EN:** Send a request to a service using a persistent client.. It works with parameters such as client, endpoint, and req_data. Key operations include req_data.copy, response.raise_for_status, client.post, response.aread, and os.environ.get. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式发送请求并处理返回结果。它会处理 client、endpoint，以及 req_data 等参数。关键操作包括 req_data.copy、response.raise_for_status、client.post、response.aread，以及 os.environ.get。其返回值会继续传给示例管线的下一阶段。

### Async function: stream_service_response
```python
async def stream_service_response(
    client: httpx.AsyncClient, endpoint: str, req_data: dict
):
    """
    Asynchronously stream the response from a service using a persistent client.
    """
    headers = {"Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}"}
    async with client.stream(
        "POST", endpoint, json=req_data, headers=headers
    ) as response:
        response.raise_for_status()
        async for chunk in response.aiter_bytes():
            yield chunk
```
**EN:** Asynchronously stream the response from a service using a persistent client.. It works with parameters such as client, endpoint, and req_data. Key operations include client.stream, response.raise_for_status, response.aiter_bytes, and os.environ.get. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 client、endpoint，以及 req_data 等参数。关键操作包括 client.stream、response.raise_for_status、response.aiter_bytes，以及 os.environ.get。它会逐步产出数据，而不是只等待单次最终结果。

### Async function: handle_completions
```python
async def handle_completions(request: Request):
    global counter, stats_calculator
    counter += 1

    st = time.time()
    try:
        req_data = await request.json()

        # Send request to prefill service, ignore the response
        await send_request_to_service(
            app.state.prefill_client, "/completions", req_data
        )

        et = time.time()
        stats_calculator.add(et - st)

        # Stream response from decode service
        async def generate_stream():
            async for chunk in stream_service_response(
                app.state.decode_client, "/completions", req_data
            ):
                yield chunk

        return StreamingResponse(generate_stream(), media_type="text/event-stream")

    except Exception as e:
        import sys
        import traceback

        exc_info = sys.exc_info()
        print("Error occurred in disagg prefill proxy server - completions endpoint")
        print(e)
        print("".join(traceback.format_exception(*exc_info)))
        raise
```
**EN:** This function asynchronously streams incremental results while performing its core task. It works with parameters such as request. Key operations include print, time.time, app.post, stats_calculator.add, and StreamingResponse. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式在执行核心任务时持续产出增量结果。它会处理 request 等参数。关键操作包括 print、time.time、app.post、stats_calculator.add，以及 StreamingResponse。它会逐步产出数据，而不是只等待单次最终结果。

### Async function: handle_chat_completions
```python
async def handle_chat_completions(request: Request):
    global counter, stats_calculator
    counter += 1

    st = time.time()
    try:
        req_data = await request.json()

        # Send request to prefill service, ignore the response
        await send_request_to_service(
            app.state.prefill_client, "/chat/completions", req_data
        )

        et = time.time()
        stats_calculator.add(et - st)

        # Stream response from decode service
        async def generate_stream():
            async for chunk in stream_service_response(
                app.state.decode_client, "/chat/completions", req_data
            ):
                yield chunk

        return StreamingResponse(generate_stream(), media_type="text/event-stream")

    except Exception as e:
        import sys
        import traceback

        exc_info = sys.exc_info()
        print(
            "Error occurred in disagg prefill proxy server  - chat completions endpoint"
        )
        print(e)
        print("".join(traceback.format_exception(*exc_info)))
        raise
```
**EN:** This function asynchronously streams incremental results while performing its core task. It works with parameters such as request. Key operations include print, time.time, app.post, stats_calculator.add, and StreamingResponse. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式在执行核心任务时持续产出增量结果。它会处理 request 等参数。关键操作包括 print、time.time、app.post、stats_calculator.add，以及 StreamingResponse。它会逐步产出数据，而不是只等待单次最终结果。

### Entry point
```python
if __name__ == "__main__":
    global global_args
    global_args = parse_args()

    import uvicorn

    uvicorn.run(app, host=global_args.host, port=global_args.port)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and uvicorn.run.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 uvicorn.run。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Interactive application wrappers / 交互式应用封装**: UI or service layers turn model calls into an interactive user experience. / 界面或服务层把模型调用包装成交互式用户体验。
- **Tool calling / 工具调用**: Model outputs are mapped to callable tools or structured action requests. / 模型输出会映射为可调用工具或结构化动作请求。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `os`, `time`, `contextlib`, `httpx`, `numpy`, `fastapi`, `fastapi.responses` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `lifespan`, `StatsCalculator`, `parse_args`, `send_request_to_service`, `stream_service_response`, `handle_completions`, `handle_chat_completions` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `httpx.AsyncClient`, `httpx.Limits`, `app.state.prefill_client.aclose`, `app.state.decode_client.aclose`, `FastAPI`, `time.time`, `self._stats.append`, `np.array` reveal the main execution path / 这些调用体现了主要执行链路。
