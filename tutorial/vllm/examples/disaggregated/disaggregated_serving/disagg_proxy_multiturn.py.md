# disagg_proxy_multiturn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/disaggregated_serving/disagg_proxy_multiturn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Disaggregated Prefill/Decode Proxy with Bidirectional KV Transfer This proxy sits between clients and a vLLM Prefill/Decode (P/D) deployment, routing multi-turn chat requests so that each turn reuses KV cache blocks / 演示解耦服务相关的基础组件。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Disaggregated Prefill/Decode Proxy with Bidirectional KV Transfer

This proxy sits between clients and a vLLM Prefill/Decode (P/D) deployment,
routing multi-turn chat requests so that each turn reuses KV cache blocks
from the previous turn's Decode node via bidirectional KV transfer.

Architecture:
    Client  ──►  Proxy  ──►  Prefill (P)  ──►  Decode (D)
                   │              │                 │
                   │   kv_transfer_params flow:     │
                   │   D finish ──► proxy caches    │
# ... key logic omitted for brevity ...
    python disagg_proxy_multiturn.py \\
        --host 0.0.0.0 --port 8000 \\
        --prefiller-host 10.0.0.1 --prefiller-port 8100 \\
        --decoder-host 10.0.0.2 --decoder-port 8200

Dependencies:
    pip install fastapi uvicorn httpx
"""
```
**EN:** Disaggregated Prefill/Decode Proxy with Bidirectional KV Transfer This proxy sits between clients and a vLLM Prefill/Decode (P/D) deployment, routing multi-turn chat requests so that each turn reuses KV cache blocks...
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from __future__ import annotations

import argparse
import itertools
import json
import logging
import os
import time
import uuid
from contextlib import asynccontextmanager
from dataclasses import dataclass, field
from typing import Any

import httpx
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse, StreamingResponse
```
**EN:** This block loads helper libraries such as __future__, argparse, itertools, json, and logging.
**CN:** 这一部分加载 __future__、argparse、itertools、json，以及 logging 等辅助库。

### Top-level setup
```python
logging.basicConfig(
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    level=logging.INFO,
)
logger = logging.getLogger("disagg_proxy")
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as logger. It also performs early helper calls such as logging.basicConfig and logging.getLogger.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 logger 等变量。它还会提前执行 logging.basicConfig 和 logging.getLogger 等辅助调用。

### Top-level setup
```python
kv_cache = ConversationKVCache(
    ttl_seconds=450.0
)  # Must be < VLLM_NIXL_ABORT_REQUEST_TIMEOUT (480s)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as kv_cache. It also performs early helper calls such as ConversationKVCache.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 kv_cache 等变量。它还会提前执行 ConversationKVCache 等辅助调用。

### Async function: _stream_from_decode
```python
async def _stream_from_decode(
    client: ServiceClient,
    endpoint: str,
    req_data: dict[str, Any],
    request_id: str,
    conversation_id: str,
) -> tuple[str, str | None, dict[str, Any] | None, str, str | None, int | None]:
    """Stream response from D, capturing text and kv_transfer_params.

    Returns (collected_text, finish_reason, kv_params, response_id, created).
    Also stores kv_params in the conversation cache.
    """
    payload = req_data.copy()
    payload["stream"] = True

    collected_text = ""
    finish_reason: str | None = None
    response_id: str | None = None
    model_name: str | None = None
    created: int | None = None
    # ... key logic omitted for brevity ...
            if kv_params:
                kv_params["remote_host"] = client.host
                captured_kv = kv_params
                if conversation_id:
                    kv_cache.put(conversation_id, kv_params)

    return (
        collected_text,
        finish_reason,
        captured_kv,
        response_id or request_id,
        model_name,
        created,
    )
```
**EN:** Stream response from D, capturing text and kv_transfer_params.. It works with parameters such as client, endpoint, req_data, request_id, and conversation_id. Key operations include chunk.get, choice.get, req_data.copy, client.client.stream, and resp.raise_for_status. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 client、endpoint、req_data、request_id，以及 conversation_id 等参数。关键操作包括 chunk.get、choice.get、req_data.copy、client.client.stream，以及 resp.raise_for_status。其返回值会继续传给示例管线的下一阶段。

### Async function: _stream_from_decode_sse
```python
async def _stream_from_decode_sse(
    client: ServiceClient,
    endpoint: str,
    req_data: dict[str, Any],
    request_id: str,
    conversation_id: str,
):
    """Yield SSE chunks from D to the client, capturing kv_transfer_params."""
    payload = req_data.copy()
    payload["stream"] = True

    async with client.client.stream(
        "POST",
        endpoint,
        json=payload,
        headers=_make_headers(request_id),
    ) as resp:
        resp.raise_for_status()
        async for line in resp.aiter_lines():
            if not line:
                yield "\n"
                continue

            if line.startswith("data: ") and line != "data: [DONE]":
                try:
                    chunk = json.loads(line[6:])
                    kv_params = chunk.get("kv_transfer_params")
                    if kv_params and conversation_id:
                        kv_params["remote_host"] = client.host
                        kv_cache.put(conversation_id, kv_params)
                except json.JSONDecodeError:
                    pass

            yield line + "\n"
```
**EN:** Yield SSE chunks from D to the client, capturing kv_transfer_params.. It works with parameters such as client, endpoint, req_data, request_id, and conversation_id. Key operations include req_data.copy, client.client.stream, resp.raise_for_status, resp.aiter_lines, and _make_headers. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 client、endpoint、req_data、request_id，以及 conversation_id 等参数。关键操作包括 req_data.copy、client.client.stream、resp.raise_for_status、resp.aiter_lines，以及 _make_headers。它会逐步产出数据，而不是只等待单次最终结果。

### Async function: lifespan
```python
async def lifespan(app: FastAPI):
    """Initialize HTTP clients for P and D instances."""
    app.state.prefill_clients: list[ServiceClient] = []
    app.state.decode_clients: list[ServiceClient] = []

    for i, (host, port) in enumerate(global_args.prefiller_instances):
        app.state.prefill_clients.append(
            ServiceClient(
                client=httpx.AsyncClient(
                    timeout=None,
                    base_url=f"http://{host}:{port}/v1",
                ),
                host=host,
                port=port,
                id=i,
            )
        )

    for i, (host, port) in enumerate(global_args.decoder_instances):
        app.state.decode_clients.append(
            ServiceClient(
                client=httpx.AsyncClient(
                    timeout=None,
                    base_url=f"http://{host}:{port}/v1",
                ),
                host=host,
                port=port,
                id=i,
            )
        )

    app.state.prefill_iter = itertools.cycle(range(len(app.state.prefill_clients)))
    app.state.decode_iter = itertools.cycle(range(len(app.state.decode_clients)))

    logger.info(
        "Ready: %d prefill, %d decode instances",
        len(app.state.prefill_clients),
        len(app.state.decode_clients),
    )
    yield

    for sc in app.state.prefill_clients + app.state.decode_clients:
        await sc.client.aclose()
```
**EN:** Initialize HTTP clients for P and D instances.. It works with parameters such as app. Key operations include len, enumerate, itertools.cycle, range, and ServiceClient. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式在执行核心任务时持续产出增量结果。它会处理 app 等参数。关键操作包括 len、enumerate、itertools.cycle、range，以及 ServiceClient。它会逐步产出数据，而不是只等待单次最终结果。

### Top-level setup
```python
app = FastAPI(title="Disaggregated P/D Proxy (Multi-turn)", lifespan=lifespan)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as app. It also performs early helper calls such as FastAPI.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 app 等变量。它还会提前执行 FastAPI 等辅助调用。

### Async function: _handle_request
```python
async def _handle_request(api_path: str, request: Request):
    """Core request handler for both /v1/chat/completions and /v1/completions."""
    req_data = await request.json()
    request_id = str(uuid.uuid4())
    conversation_id: str = req_data.pop("conversation_id", "")
    client_wants_stream = req_data.get("stream", False)

    if not conversation_id:
        logger.warning(
            "[%s] No conversation_id provided — KV cache reuse disabled "
            "for this request. Add a 'conversation_id' field to enable "
            "cross-turn KV sharing.",
            request_id,
        )

    # Step 1: Look up cached D blocks from the previous turn
    cached_kv = kv_cache.get(conversation_id) if conversation_id else None

    if cached_kv:
        # Tell P to read D's blocks (bidirectional transfer)
    # ... key logic omitted for brevity ...
            "object": "text_completion",
            "created": created or int(time.time()),
            "model": model or req_data.get("model", ""),
            "choices": [
                {
                    "index": 0,
                    "text": text,
                    "logprobs": None,
                    "finish_reason": finish_reason,
                }
            ],
            "usage": None,
        }
    return JSONResponse(content=body)
```
**EN:** Core request handler for both /v1/chat/completions and /v1/completions.. It works with parameters such as api_path and request. Key operations include time.time, req_data.get, logger.info, _next_client, and int. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式发送请求并处理返回结果。它会处理 api_path 和 request 等参数。关键操作包括 time.time、req_data.get、logger.info、_next_client，以及 int。其返回值会继续传给示例管线的下一阶段。

### Async function: chat_completions
```python
async def chat_completions(request: Request):
    return await _handle_request("/chat/completions", request)
```
**EN:** This function asynchronously encapsulates a reusable step in the example. It works with parameters such as request. Key operations include app.post and _handle_request. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 request 等参数。关键操作包括 app.post 和 _handle_request。其返回值会继续传给示例管线的下一阶段。

### Function: parse_args
```python
def parse_args() -> argparse.Namespace:
    p = argparse.ArgumentParser(
        description="Disaggregated P/D proxy with bidirectional KV transfer",
    )
    p.add_argument("--host", default="0.0.0.0")
    p.add_argument("--port", type=int, default=8000)
    p.add_argument(
        "--prefiller-host",
        "--prefiller-hosts",
        dest="prefiller_hosts",
        nargs="+",
        default=["localhost"],
    )
    p.add_argument(
        "--prefiller-port",
        "--prefiller-ports",
        dest="prefiller_ports",
        type=int,
        nargs="+",
        default=[8100],
    # ... key logic omitted for brevity ...
        type=int,
        nargs="+",
        default=[8200],
    )
    args = p.parse_args()

    if len(args.prefiller_hosts) != len(args.prefiller_ports):
        p.error("Number of prefiller hosts must match ports")
    if len(args.decoder_hosts) != len(args.decoder_ports):
        p.error("Number of decoder hosts must match ports")

    args.prefiller_instances = list(zip(args.prefiller_hosts, args.prefiller_ports))
    args.decoder_instances = list(zip(args.decoder_hosts, args.decoder_ports))
    return args
```
**EN:** This function builds and validates command-line arguments. Key operations include p.add_argument, len, list, p.error, and zip. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 p.add_argument、len、list、p.error，以及 zip。其返回值会继续传给示例管线的下一阶段。

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
- **External libraries / 外部库**: `__future__`, `argparse`, `itertools`, `json`, `logging`, `os`, `time`, `uuid` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `_stream_from_decode`, `_stream_from_decode_sse`, `lifespan`, `_handle_request`, `chat_completions`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `logging.basicConfig`, `logging.getLogger`, `ConversationKVCache`, `chunk.get`, `choice.get`, `req_data.copy`, `client.client.stream`, `resp.raise_for_status` reveal the main execution path / 这些调用体现了主要执行链路。
