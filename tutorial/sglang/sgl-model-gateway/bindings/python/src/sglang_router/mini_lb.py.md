# mini_lb.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/bindings/python/src/sglang_router/mini_lb.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements classes `MiniLoadBalancer` and functions `maybe_wrap_ipv6_address`, `health_check`, `health_generate`, `flush_cache`, and 11 more for the Python router binding. Other gateway components import it to reuse configuration, control flow, or shared utilities. / 该模块为 Python 路由绑定层 实现了 类 `MiniLoadBalancer`与函数 `maybe_wrap_ipv6_address`, `health_check`, `health_generate`, `flush_cache`, and 11 more，其他网关组件会导入它以复用配置、控制流程或共享工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Module docstring
```python
"""
Minimal HTTP load balancer for prefill and decode servers for testing.
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 5-20: Imports and dependencies
```python
import asyncio
import ipaddress
import logging
import random
import urllib
import warnings
from http import HTTPStatus
from itertools import chain
from typing import Optional

import aiohttp
import orjson
import uvicorn
from fastapi import FastAPI, HTTPException
from fastapi.responses import ORJSONResponse, Response, StreamingResponse
from sglang_router.router_args import RouterArgs
```
**EN:** This block imports `asyncio`, `ipaddress`, `logging`, `random`, and 11 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 22-26: Module constants and configuration
```python
logger = logging.getLogger(__name__)

AIOHTTP_STREAM_READ_CHUNK_SIZE = (
    1024 * 64
)  # 64KB, to prevent aiohttp's "Chunk too big" error
```
**EN:** This section defines module-level names such as `logger`, `AIOHTTP_STREAM_READ_CHUNK_SIZE`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 29-34: function `maybe_wrap_ipv6_address`
```python
def maybe_wrap_ipv6_address(address: str) -> str:
    try:
        ipaddress.IPv6Address(address)
        return f"[{address}]"
    except ValueError:
        return address
```
**EN:** function `maybe_wrap_ipv6_address` encapsulates `maybe wrap ipv6 address` logic for callers in this module.
**CN:** 函数 `maybe_wrap_ipv6_address` 封装了与 `maybe wrap ipv6 address` 相关的逻辑，供本模块中的调用方复用。

### Lines 37-236: Class `MiniLoadBalancer`
```python
class MiniLoadBalancer:
    def __init__(
        self,
        router_args: RouterArgs,
    ):
        self._validate_router_args(router_args)

        self.host = router_args.host
        self.port = router_args.port
        self.timeout = router_args.request_timeout_secs
        self.prefill_urls = [url[0] for url in router_args.prefill_urls]
        self.prefill_bootstrap_ports = [url[1] for url in router_args.prefill_urls]
        self.decode_urls = router_args.decode_urls
        self.test_external_dp_routing = router_args.test_external_dp_routing
        self.prefill_dp_size = None
        self.decode_dp_size = None

    def _validate_router_args(self, router_args: RouterArgs):
        logger.warning(
            "\x1b[33mMiniLB is only for debugging purposes, it only supports random policy!\033[0m"
        )

        # NOTE: too many arguments unsupported, just validate some important ones
        if router_args.policy != "random":
            logger.warning("[MiniLB] Overriding policy to random")
            router_args.policy = "random"

        if not router_args.pd_disaggregation:
            raise ValueError("MiniLB only supports PD disaggregation mode")

        if len(router_args.prefill_urls) == 0 or len(router_args.decode_urls) == 0:
            raise ValueError(
                "MiniLB requires at least one prefill and one decode server"
            )

    def start(self):
        global lb
        lb = self
        uvicorn.run(app, host=self.host, port=self.port)

    async def _ensure_dp_sizes(self):
        if self.prefill_dp_size is not None:
            return
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{self.prefill_urls[0]}/server_info") as resp:
                info = await resp.json()
                self.prefill_dp_size = len(info.get("internal_states", [1]))
            async with session.get(f"{self.decode_urls[0]}/server_info") as resp:
                info = await resp.json()
                self.decode_dp_size = len(info.get("internal_states", [1]))
        logger.info(
            f"[MiniLB] DP sizes: prefill={self.prefill_dp_size}, decode={self.decode_dp_size}"
        )

    def _fork_dp_requests(self, request):
        p_rank = random.randint(0, self.prefill_dp_size - 1)
        d_rank = random.randint(0, self.decode_dp_size - 1)

        prefill_req = request.copy()
        decode_req = request.copy()
        prefill_req["routed_dp_rank"] = p_rank
        decode_req["routed_dp_rank"] = d_rank
        decode_req["disagg_prefill_dp_rank"] = p_rank

        return prefill_req, decode_req, d_rank

    def select_pair(self):
        assert len(self.prefill_urls) > 0, "No prefill servers available"
        assert len(self.decode_urls) > 0, "No decode servers available"
        pidx = random.randint(0, len(self.prefill_urls) - 1)
        didx = random.randint(0, len(self.decode_urls) - 1)
        return (
            self.prefill_urls[pidx],
            self.prefill_bootstrap_ports[pidx],
            self.decode_urls[didx],
        )

    async def generate(
        self, modified_request, prefill_server, decode_server, endpoint
    ) -> ORJSONResponse:
        assert endpoint[0] != "/", f"Endpoint should not start with '/': {endpoint}"

        expected_decode_dp_rank = None
        if self.test_external_dp_routing:
            await self._ensure_dp_sizes()
            prefill_req, decode_req, expected_decode_dp_rank = self._fork_dp_requests(
                modified_request
            )
        else:
            prefill_req = modified_request
            decode_req = modified_request

        async with aiohttp.ClientSession(
            timeout=aiohttp.ClientTimeout(
                total=self.timeout
            )  # Add timeout for request reliability
        ) as session:

            tasks = [
                session.post(f"{prefill_server}/{endpoint}", json=prefill_req),
                session.post(f"{decode_server}/{endpoint}", json=decode_req),
            ]

            # Wait for both responses to complete. Prefill should end first.
            prefill_response, decode_response = await asyncio.gather(*tasks)

            if "return_logprob" in modified_request:

                prefill_json = await prefill_response.json()
                ret_json = await decode_response.json()

                # merge `meta_info.input_token_logprobs` from prefill to decode
                if "meta_info" in ret_json:
                    if "input_token_logprobs" in ret_json["meta_info"]:
                        ret_json["meta_info"]["input_token_logprobs"] = (
                            prefill_json["meta_info"]["input_token_logprobs"]
                            + ret_json["meta_info"]["input_token_logprobs"]
                        )
            else:
                ret_json = await decode_response.json()

            if expected_decode_dp_rank is not None:
                actual = ret_json.get("meta_info", {}).get("dp_rank")
                if actual != expected_decode_dp_rank:
                    return ORJSONResponse(
                        content={
                            "error": f"DP rank mismatch: expected {expected_decode_dp_rank}, got {actual}"
                        },
                        status_code=500,
                    )

            return ORJSONResponse(
                content=ret_json,
                status_code=decode_response.status,
            )

    async def generate_stream(
        self, modified_request, prefill_server, decode_server, endpoint="generate"
    ):

        if self.test_external_dp_routing:
            warnings.warn("--test-external-dp-routing is not supported with streaming")

        assert endpoint[0] != "/", f"Endpoint should not start with '/': {endpoint}"

        async def stream_results():
            async with aiohttp.ClientSession(
                timeout=aiohttp.ClientTimeout(
                    total=self.timeout
                )  # Add timeout for request reliability
            ) as session:
                # Create the tasks for both prefill and decode requests
                tasks = [
                    session.post(f"{prefill_server}/{endpoint}", json=modified_request),
                    session.post(f"{decode_server}/{endpoint}", json=modified_request),
                ]

                # Wait for both responses to complete. Since this is streaming, they return immediately.
                prefill_response, decode_response = await asyncio.gather(*tasks)

                if modified_request.get("return_logprob", False):
                    prefill_chunks = []
                    async for chunk in prefill_response.content:
                        prefill_chunks.append(chunk)

                    first_prefill_chunk = (
                        prefill_chunks[0].decode("utf-8")[5:].strip("\n")
                    )
                    first_prefill_chunk_json = orjson.loads(first_prefill_chunk)

                    async for chunk in decode_response.content:
                        # Note: This is inefficient
                        # merge prefill input_token_logprobs, output_token_logprobs to decode
                        decoded_chunk = chunk.decode("utf-8")
                        if (
                            decoded_chunk
                            and decoded_chunk.startswith("data:")
                            and "[DONE]" not in decoded_chunk
                        ):
                            ret_json = orjson.loads(decoded_chunk[5:].strip("\n"))
                            ret_json["meta_info"]["input_token_logprobs"] = (
                                first_prefill_chunk_json["meta_info"][
                                    "input_token_logprobs"
                                ]
                                + ret_json["meta_info"]["input_token_logprobs"]
                            )

                            yield b"data: " + orjson.dumps(ret_json) + b"\n\n"
                        else:
                            yield chunk
                else:
                    async for chunk in decode_response.content.iter_chunked(
                        AIOHTTP_STREAM_READ_CHUNK_SIZE
                    ):
                        yield chunk

        return StreamingResponse(
            stream_results(),
            media_type="text/event-stream",
        )
```
**EN:** Class `MiniLoadBalancer` groups related state and behavior. It exposes 8 method(s) that implement the module's primary abstraction.
**CN:** 类 `MiniLoadBalancer` 将相关状态与行为封装在一起，提供 8 个方法来实现本模块的核心抽象。

### Lines 239-240: Module constants and configuration
```python
app = FastAPI()
lb: Optional[MiniLoadBalancer] = None
```
**EN:** This section defines module-level names such as `app`, `lb`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 241-245: Async function `health_check`
```python


@app.get("/health")
async def health_check():
    return Response(status_code=200)
```
**EN:** Async function `health_check` encapsulates `health check` logic for callers in this module.
**CN:** 异步函数 `health_check` 封装了与 `health check` 相关的逻辑，供本模块中的调用方复用。

### Lines 246-257: Async function `health_generate`
```python


@app.get("/health_generate")
async def health_generate():
    async with aiohttp.ClientSession() as session:
        # Create the tasks
        tasks = []
        for server in chain(lb.prefill_urls, lb.decode_urls):
            tasks.append(session.get(f"{server}/health_generate"))
        for i, response in enumerate(asyncio.as_completed(tasks)):
            await response
    return Response(status_code=200)
```
**EN:** Async function `health_generate` encapsulates `health generate` logic for callers in this module.
**CN:** 异步函数 `health_generate` 封装了与 `health generate` 相关的逻辑，供本模块中的调用方复用。

### Lines 258-269: Async function `flush_cache`
```python


@app.post("/flush_cache")
async def flush_cache():
    async with aiohttp.ClientSession() as session:
        # Create the tasks
        tasks = []
        for server in chain(lb.prefill_urls, lb.decode_urls):
            tasks.append(session.post(f"{server}/flush_cache"))
        for i, response in enumerate(asyncio.as_completed(tasks)):
            await response
    return Response(status_code=200)
```
**EN:** Async function `flush_cache` encapsulates `flush cache` logic for callers in this module.
**CN:** 异步函数 `flush_cache` 封装了与 `flush cache` 相关的逻辑，供本模块中的调用方复用。

### Lines 270-310: Async function `get_server_info`
```python


# TODO: Remove `/get_server_info` alias after one release-cycle deprecation window.
@app.get("/server_info")
@app.get("/get_server_info")
async def get_server_info():
    prefill_infos = []
    decode_infos = []
    all_internal_states = []

    async with aiohttp.ClientSession() as session:
        for server in lb.prefill_urls:
            server_info = await session.get(f"{server}/server_info")
            prefill_infos.append(await server_info.json())
        for server in lb.decode_urls:
            server_info = await session.get(f"{server}/server_info")
            info_json = await server_info.json()
            decode_infos.append(info_json)
            # Extract internal_states from decode servers
            if "internal_states" in info_json:
                all_internal_states.extend(info_json["internal_states"])

    # Return format expected by bench_one_batch_server.py
    if all_internal_states:
        return {
            "internal_states": all_internal_states,
            "prefill": prefill_infos,
            "decode": decode_infos,
        }
    else:
        # Fallback with dummy data if no internal states found
        return {
            "internal_states": [
                {
                    "last_gen_throughput": 0.0,
                    "avg_spec_accept_length": None,
                }
            ],
            "prefill": prefill_infos,
            "decode": decode_infos,
        }
```
**EN:** Async function `get_server_info` encapsulates `get server info` logic for callers in this module.
**CN:** 异步函数 `get_server_info` 封装了与 `get server info` 相关的逻辑，供本模块中的调用方复用。

### Lines 313-343: Async function `_get_model_info_impl`
```python
async def _get_model_info_impl():
    if not lb or not lb.prefill_urls:
        raise HTTPException(
            status_code=HTTPStatus.SERVICE_UNAVAILABLE,
            detail="There is no server registered",
        )

    target_server_url = lb.prefill_urls[0]
    endpoint_url = f"{target_server_url}/model_info"

    async with aiohttp.ClientSession() as session:
        try:
            async with session.get(endpoint_url) as response:
                if response.status != 200:
                    error_text = await response.text()
                    raise HTTPException(
                        status_code=HTTPStatus.BAD_GATEWAY,
                        detail=(
                            f"Failed to get model info from {target_server_url}"
                            f"Status: {response.status}, Response: {error_text}"
                        ),
                    )

                model_info_json = await response.json()
                return ORJSONResponse(content=model_info_json)

        except aiohttp.ClientError as e:
            raise HTTPException(
                status_code=HTTPStatus.SERVICE_UNAVAILABLE,
                detail=f"Failed to get model info from backend",
            )
```
**EN:** Async function `_get_model_info_impl` encapsulates `get model info impl` logic for callers in this module.
**CN:** 异步函数 `_get_model_info_impl` 封装了与 `get model info impl` 相关的逻辑，供本模块中的调用方复用。

### Lines 344-348: Async function `model_info`
```python


@app.get("/model_info")
async def model_info():
    return await _get_model_info_impl()
```
**EN:** Async function `model_info` encapsulates `model info` logic for callers in this module.
**CN:** 异步函数 `model_info` 封装了与 `model info` 相关的逻辑，供本模块中的调用方复用。

### Lines 349-353: Async function `get_model_info`
```python


@app.get("/get_model_info")
async def get_model_info():
    return await _get_model_info_impl()
```
**EN:** Async function `get_model_info` encapsulates `get model info` logic for callers in this module.
**CN:** 异步函数 `get_model_info` 封装了与 `get model info` 相关的逻辑，供本模块中的调用方复用。

### Lines 354-392: Async function `handle_generate_request`
```python


@app.post("/generate")
async def handle_generate_request(request_data: dict):
    prefill_server, bootstrap_port, decode_server = lb.select_pair()

    # Parse and transform prefill_server for bootstrap data
    parsed_url = urllib.parse.urlparse(prefill_server)
    hostname = maybe_wrap_ipv6_address(parsed_url.hostname)
    modified_request = request_data.copy()

    batch_size = _get_request_batch_size(modified_request)
    if batch_size is not None:
        modified_request.update(
            {
                "bootstrap_host": [hostname] * batch_size,
                "bootstrap_port": [bootstrap_port] * batch_size,
                "bootstrap_room": [
                    _generate_bootstrap_room() for _ in range(batch_size)
                ],
            }
        )
    else:
        modified_request.update(
            {
                "bootstrap_host": hostname,
                "bootstrap_port": bootstrap_port,
                "bootstrap_room": _generate_bootstrap_room(),
            }
        )

    if request_data.get("stream", False):
        return await lb.generate_stream(
            modified_request, prefill_server, decode_server, "generate"
        )
    else:
        return await lb.generate(
            modified_request, prefill_server, decode_server, "generate"
        )
```
**EN:** Async function `handle_generate_request` encapsulates `handle generate request` logic for callers in this module.
**CN:** 异步函数 `handle_generate_request` 封装了与 `handle generate request` 相关的逻辑，供本模块中的调用方复用。

### Lines 395-423: Async function `_forward_to_backend`
```python
async def _forward_to_backend(request_data: dict, endpoint_name: str):
    prefill_server, bootstrap_port, decode_server = lb.select_pair()

    # Parse and transform prefill_server for bootstrap data
    parsed_url = urllib.parse.urlparse(prefill_server)
    hostname = maybe_wrap_ipv6_address(parsed_url.hostname)
    modified_request = request_data.copy()
    modified_request.update(
        {
            "bootstrap_host": hostname,
            "bootstrap_port": bootstrap_port,
            "bootstrap_room": _generate_bootstrap_room(),
        }
    )

    if request_data.get("stream", False):
        return await lb.generate_stream(
            modified_request,
            prefill_server,
            decode_server,
            endpoint=endpoint_name,
        )
    else:
        return await lb.generate(
            modified_request,
            prefill_server,
            decode_server,
            endpoint=endpoint_name,
        )
```
**EN:** Async function `_forward_to_backend` encapsulates `forward to backend` logic for callers in this module.
**CN:** 异步函数 `_forward_to_backend` 封装了与 `forward to backend` 相关的逻辑，供本模块中的调用方复用。

### Lines 424-428: Async function `handle_chat_completion_request`
```python


@app.post("/v1/chat/completions")
async def handle_chat_completion_request(request_data: dict):
    return await _forward_to_backend(request_data, "v1/chat/completions")
```
**EN:** Async function `handle_chat_completion_request` encapsulates `handle chat completion request` logic for callers in this module.
**CN:** 异步函数 `handle_chat_completion_request` 封装了与 `handle chat completion request` 相关的逻辑，供本模块中的调用方复用。

### Lines 429-433: Async function `handle_completion_request`
```python


@app.post("/v1/completions")
async def handle_completion_request(request_data: dict):
    return await _forward_to_backend(request_data, "v1/completions")
```
**EN:** Async function `handle_completion_request` encapsulates `handle completion request` logic for callers in this module.
**CN:** 异步函数 `handle_completion_request` 封装了与 `handle completion request` 相关的逻辑，供本模块中的调用方复用。

### Lines 436-437: function `_generate_bootstrap_room`
```python
def _generate_bootstrap_room():
    return random.randint(0, 2**63 - 1)
```
**EN:** function `_generate_bootstrap_room` encapsulates `generate bootstrap room` logic for callers in this module.
**CN:** 函数 `_generate_bootstrap_room` 封装了与 `generate bootstrap room` 相关的逻辑，供本模块中的调用方复用。

### Lines 438-446: function `_get_request_batch_size`
```python


# We may utilize `GenerateReqInput`'s logic later
def _get_request_batch_size(request):
    if (text := request.get("text")) is not None:
        return None if isinstance(text, str) else len(text)
    if (input_ids := request.get("input_ids")) is not None:
        return None if isinstance(input_ids[0], int) else len(input_ids)
    return None
```
**EN:** function `_get_request_batch_size` encapsulates `get request batch size` logic for callers in this module.
**CN:** 函数 `_get_request_batch_size` 封装了与 `get request batch size` 相关的逻辑，供本模块中的调用方复用。

### Lines 447-462: Async function `get_models`
```python


@app.get("/v1/models")
async def get_models():
    prefill_server = lb.prefill_urls[0]  # Get the first prefill server
    async with aiohttp.ClientSession() as session:
        try:
            response = await session.get(f"{prefill_server}/v1/models")
            if response.status != 200:
                raise HTTPException(
                    status_code=response.status,
                    detail=f"Prefill server error: Status {response.status}",
                )
            return ORJSONResponse(content=await response.json())
        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))
```
**EN:** Async function `get_models` encapsulates `get models` logic for callers in this module.
**CN:** 异步函数 `get_models` 封装了与 `get models` 相关的逻辑，供本模块中的调用方复用。

## Key Concepts / 关键概念
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Asynchronous coordination / 异步协调
- Primary classes: `MiniLoadBalancer` / 主要类：`MiniLoadBalancer`
- Reusable functions: `maybe_wrap_ipv6_address`, `health_check`, `health_generate`, `flush_cache`, and 11 more / 可复用函数：`maybe_wrap_ipv6_address`, `health_check`, `health_generate`, `flush_cache`, and 11 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `http`, `ipaddress`, `itertools`, `logging`, `random`, `typing`, `urllib`, `warnings`
- **Third-party / 第三方**: `aiohttp`, `fastapi`, `fastapi.responses`, `orjson`, `uvicorn`
- **Internal / 内部模块**: `sglang_router.router_args`
