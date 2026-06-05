# disagg_epd_proxy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/disaggregated_encoder/disagg_epd_proxy.py`
- **Repository**: vllm-project/vllm
- **Purpose**: disagg_encoder_proxy.py Proxy that routes OpenAI-compatible “/v1/chat/completions” requests to two clusters: • encode (multimodal feature extraction) • decode (language-model inference) For MM input we: 1 / 演示解耦服务相关的基础组件。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
disagg_encoder_proxy.py

Proxy that routes OpenAI-compatible “/v1/chat/completions” requests to two
clusters:
  • encode  (multimodal feature extraction)
  • decode  (language-model inference)

For MM input we:
    1. Extract *every* image/audio item.
    2. Fire N concurrent requests to the encoder cluster
       (one request per item, with **all text removed**).
    3. Wait for all of them to succeed.
    4. Forward the *original* request to a decode server.
"""
```
**EN:** disagg_encoder_proxy.py Proxy that routes OpenAI-compatible “/v1/chat/completions” requests to two clusters: • encode (multimodal feature extraction) • decode (language-model inference) For MM input we: 1.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from __future__ import annotations

import argparse
import asyncio
import logging
import os
import random
import uuid
from collections.abc import AsyncIterator

import aiohttp
import uvicorn
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse, StreamingResponse
```
**EN:** This block loads helper libraries such as __future__, argparse, asyncio, logging, and os.
**CN:** 这一部分加载 __future__、argparse、asyncio、logging，以及 os 等辅助库。

### Top-level setup
```python
logging.basicConfig(
    level=logging.DEBUG, format="%(asctime)s %(levelname)s: %(message)s"
)
logger = logging.getLogger("proxy")

app = FastAPI()
encode_session: aiohttp.ClientSession | None = None
prefill_session: aiohttp.ClientSession | None = None
decode_session: aiohttp.ClientSession | None = None

###############################################################################
# Utils
###############################################################################


MM_TYPES = {"image_url", "audio_url", "input_audio"}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as logger, app, encode_session, prefill_session, decode_session, and MM_TYPES. It also performs early helper calls such as logging.basicConfig, logging.getLogger, and FastAPI.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 logger、app、encode_session、prefill_session、decode_session，以及 MM_TYPES 等变量。它还会提前执行 logging.basicConfig、logging.getLogger，以及 FastAPI 等辅助调用。

### Async function: fanout_encoder_primer
```python
async def fanout_encoder_primer(
    orig_request: dict,
    e_urls: list[str],
    req_id: str,
) -> None:
    """
    1. Build one request *per MM item* with all text removed.
    2. Send them concurrently to the encode cluster.
    3. Raise if any of them fails.
    """
    logger.info("[%s] Processing multimodal items...", req_id)

    mm_items = extract_mm_items(orig_request)
    if not mm_items:
        logger.info("[%s] No multimodal items, skipping encoder", req_id)
        return  # nothing to do

    logger.info("[%s] got %d multimodal items...", req_id, len(mm_items))

    tasks = []
    # ... key logic omitted for brevity ...
                "[%s] Encoder request #%d returned status %s: %s",
                req_id,
                idx,
                r.status,
                detail,
            )
            raise HTTPException(
                status_code=r.status,
                detail=f"Encoder request failed: {detail}",
            )

    logger.info(
        "[%s] All %d encoder requests completed successfully", req_id, len(mm_items)
    )
```
**EN:** 1.. It works with parameters such as orig_request, e_urls, and req_id. Key operations include logger.info, len, enumerate, logger.error, and HTTPException.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 orig_request、e_urls，以及 req_id 等参数。关键操作包括 logger.info、len、enumerate、logger.error，以及 HTTPException。

### Async function: process_prefill_stage
```python
async def process_prefill_stage(
    req_data: dict,
    p_url: str,
    req_id: str,
) -> dict:
    """Process request through Prefill stage and return kv_transfer_params"""
    logger.info("[%s] Sending prefill request to: %s", req_id, p_url)

    prefill_request = req_data.copy()
    prefill_request["kv_transfer_params"] = {
        "do_remote_decode": True,
        "do_remote_prefill": False,
        "remote_engine_id": None,
        "remote_block_ids": None,
        "remote_host": None,
        "remote_port": None,
    }
    prefill_request["stream"] = False
    prefill_request["max_tokens"] = 1
    if "max_completion_tokens" in prefill_request:
    # ... key logic omitted for brevity ...
            raise HTTPException(
                status_code=prefill_response.status,
                detail={"error": "Prefill request failed", "message": error_text},
            )
        logger.info("[%s] Prefill request completed successfully", req_id)

        return prefill_response

    except Exception as e:
        logger.error("Prefill processing failed: %s", str(e))
        raise HTTPException(
            status_code=500,
            detail={"error": "Prefill processing error", "message": str(e)},
        ) from e
```
**EN:** Process request through Prefill stage and return kv_transfer_params. It works with parameters such as req_data, p_url, and req_id. Key operations include logger.info, logger.error, HTTPException, str, and req_data.copy. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 req_data、p_url，以及 req_id 等参数。关键操作包括 logger.info、logger.error、HTTPException、str，以及 req_data.copy。其返回值会继续传给示例管线的下一阶段。

### Async function: log_requests
```python
async def log_requests(request: Request, call_next):
    """Middleware to log all incoming requests and responses"""
    req_id = request.headers.get("x-request-id", str(uuid.uuid4()))

    # Log incoming request
    logger.info(
        ">>> [%s] %s %s from %s",
        req_id,
        request.method,
        request.url.path,
        request.client.host if request.client else "unknown",
    )

    try:
        # Process request
        response = await call_next(request)

        # Log response
        logger.info(
            "<<< [%s] %s %s completed with status %d",
            req_id,
            request.method,
            request.url.path,
            response.status_code,
        )

        return response
    except Exception as e:
        # Log errors
        logger.exception(
            "!!! [%s] %s %s failed with error: %s",
            req_id,
            request.method,
            request.url.path,
            str(e),
        )
        raise
```
**EN:** Middleware to log all incoming requests and responses. It works with parameters such as request and call_next. Key operations include logger.info, str, app.middleware, request.headers.get, and uuid.uuid4. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式发送请求并处理返回结果。它会处理 request 和 call_next 等参数。关键操作包括 logger.info、str、app.middleware、request.headers.get，以及 uuid.uuid4。其返回值会继续传给示例管线的下一阶段。

### Async function: on_startup
```python
async def on_startup() -> None:
    global encode_session, prefill_session, decode_session
    timeout = aiohttp.ClientTimeout(total=100_000)
    connector = aiohttp.TCPConnector(limit=0, force_close=False)
    encode_session = aiohttp.ClientSession(timeout=timeout, connector=connector)
    if app.state.p_urls:
        # only setup if prefill instance(s) exist
        prefill_session = aiohttp.ClientSession(timeout=timeout, connector=connector)
    decode_session = aiohttp.ClientSession(timeout=timeout, connector=connector)
```
**EN:** This function asynchronously encapsulates a reusable step in the example. Key operations include aiohttp.ClientSession, app.on_event, aiohttp.ClientTimeout, and aiohttp.TCPConnector.
**CN:** 该函数以异步方式封装示例中的可复用步骤。关键操作包括 aiohttp.ClientSession、app.on_event、aiohttp.ClientTimeout，以及 aiohttp.TCPConnector。

### Async function: forward_non_stream
```python
async def forward_non_stream(
    req_data: dict, req_id: str, e_urls: list[str], p_url: str, d_url: str
) -> dict:
    try:
        # Step 1: Process through Encoder instance (if has MM input)
        await fanout_encoder_primer(req_data, e_urls, req_id)

        # Step 2: Process through Prefill instance
        req_data = await maybe_prefill(req_data, p_url, req_id)

        # Step 3: Process through Decode instance
        logger.info("[%s] Forwarding to decode: %s", req_id, d_url)
        headers = {"x-request-id": req_id}

        # Non-streaming response
        async with decode_session.post(
            f"{d_url}/v1/chat/completions", json=req_data, headers=headers
        ) as resp:
            resp.raise_for_status()
            return await resp.json()

    except HTTPException:
        raise
    except Exception as e:
        logger.exception("[%s] Error in forward_non_stream: %s", req_id, str(e))
        raise HTTPException(status_code=500, detail=f"Proxy error: {str(e)}") from e
```
**EN:** This function asynchronously streams incremental updates to the caller. It works with parameters such as req_data, req_id, e_urls, p_url, and d_url. Key operations include str, logger.info, fanout_encoder_primer, maybe_prefill, and decode_session.post. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 req_data、req_id、e_urls、p_url，以及 d_url 等参数。关键操作包括 str、logger.info、fanout_encoder_primer、maybe_prefill，以及 decode_session.post。其返回值会继续传给示例管线的下一阶段。

### Async function: forward_stream
```python
async def forward_stream(
    req_data: dict, req_id: str, e_urls: list[str], p_url: str, d_url: str
) -> AsyncIterator[str]:
    try:
        # Step 1: Process through Encoder instance (if has MM input)
        await fanout_encoder_primer(req_data, e_urls, req_id)

        # Step 2: Process through Prefill instance
        req_data = await maybe_prefill(req_data, p_url, req_id)

        # Step 3: Process through Decode instance
        logger.info("[%s] Starting streaming from decode: %s", req_id, d_url)
        headers = {"x-request-id": req_id}

        # Streaming response
        async with decode_session.post(
            f"{d_url}/v1/chat/completions",
            json=req_data,
            headers=headers,
        ) as resp:
            resp.raise_for_status()
            async for chunk in resp.content.iter_chunked(1024):
                if chunk:
                    yield chunk.decode("utf-8", errors="ignore")

        logger.info("[%s] Streaming completed", req_id)

    except HTTPException:
        logger.exception("[%s] HTTPException in forward_stream", req_id)
        raise
    except Exception as e:
        logger.exception("[%s] Error in forward_stream: %s", req_id, str(e))
        raise HTTPException(
            status_code=500, detail=f"Proxy streaming error: {str(e)}"
        ) from e
```
**EN:** This function asynchronously streams incremental updates to the caller. It works with parameters such as req_data, req_id, e_urls, p_url, and d_url. Key operations include logger.info, logger.exception, str, fanout_encoder_primer, and maybe_prefill. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 req_data、req_id、e_urls、p_url，以及 d_url 等参数。关键操作包括 logger.info、logger.exception、str、fanout_encoder_primer，以及 maybe_prefill。它会逐步产出数据，而不是只等待单次最终结果。

### Async function: chat_completions
```python
async def chat_completions(request: Request):
    try:
        req_data = await request.json()
        req_id = request.headers.get("x-request-id", str(uuid.uuid4()))

        e_urls = app.state.e_urls  # we want the full list for fan-out
        p_url = random.choice(app.state.p_urls) if app.state.p_urls else None
        d_url = random.choice(app.state.d_urls)

        is_streaming = req_data.get("stream", False)

        if is_streaming:
            return StreamingResponse(
                forward_stream(req_data, req_id, e_urls, p_url, d_url),
                media_type="text/event-stream",
            )
        result = await forward_non_stream(req_data, req_id, e_urls, p_url, d_url)
        return JSONResponse(content=result)

    except HTTPException:
        raise
    except Exception as e:
        logger.exception("Error in chat_completions endpoint: %s", str(e))
        raise HTTPException(
            status_code=500, detail=f"Request processing error: {str(e)}"
        ) from e
```
**EN:** This function asynchronously encapsulates a reusable step in the example. It works with parameters such as request. Key operations include str, random.choice, app.post, request.headers.get, and req_data.get. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 request 等参数。关键操作包括 str、random.choice、app.post、request.headers.get，以及 req_data.get。其返回值会继续传给示例管线的下一阶段。

### Async function: start_profile
```python
async def start_profile(request: Request):
    body = await request.json()
    # TODO: handle multi urls properly
    e_url = random.choice(app.state.e_urls)
    p_url = random.choice(app.state.p_urls) if app.state.p_urls else None
    d_url = random.choice(app.state.d_urls)
    return await _profile_cmd("start", body, e_url, p_url, d_url)
```
**EN:** This function asynchronously encapsulates a reusable step in the example. It works with parameters such as request. Key operations include random.choice, app.post, request.json, and _profile_cmd. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 request 等参数。关键操作包括 random.choice、app.post、request.json，以及 _profile_cmd。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", default="0.0.0.0")
    parser.add_argument("--port", type=int, default=8000)
    parser.add_argument(
        "--encode-servers-urls",
        required=True,
        help='Comma-separated encode URLs ("http://e1:8001,http://e2:8001")',
    )
    parser.add_argument(
        "--prefill-servers-urls",
        required=True,
    # ... key logic omitted for brevity ...
    uvicorn.run(
        app,
        host=args.host,
        port=args.port,
        log_level="info",
        loop="uvloop",
        access_log=True,
    )
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to logger.info, u.strip, parser.add_argument, and argparse.ArgumentParser.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 logger.info、u.strip、parser.add_argument，以及 argparse.ArgumentParser。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Interactive application wrappers / 交互式应用封装**: UI or service layers turn model calls into an interactive user experience. / 界面或服务层把模型调用包装成交互式用户体验。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `__future__`, `argparse`, `asyncio`, `logging`, `os`, `random`, `uuid`, `collections.abc` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `fanout_encoder_primer`, `process_prefill_stage`, `log_requests`, `on_startup`, `forward_non_stream`, `forward_stream`, `chat_completions`, `start_profile` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `logging.basicConfig`, `logging.getLogger`, `FastAPI`, `logger.info`, `len`, `enumerate`, `logger.error`, `HTTPException` reveal the main execution path / 这些调用体现了主要执行链路。
