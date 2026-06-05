# util.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/prefill_only/util.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on prefill only. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 prefill only 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and setup / 导入与初始化
```python
"""
Common utilities for SGLang benchmark scripts.

This module contains shared code for benchmarking different SGLang APIs
including scoring, embeddings, and other endpoints.
"""

import asyncio
import concurrent.futures
import json
import os
import random
from statistics import mean
from typing import Any, Callable, Dict, List, Optional, Tuple

import aiohttp
import numpy as np
from tqdm import tqdm
from transformers import AutoTokenizer
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 22-22: Class `BenchmarkConfig` declaration / 类 `BenchmarkConfig` 声明
```python
class BenchmarkConfig:
```
**EN:** This block introduces class `BenchmarkConfig`, which measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块引入类 `BenchmarkConfig`，用于测量运行时延迟、吞吐或其他基准指标。

### Lines 23-23: Class-level state / 类级状态
```python
    """Configuration for benchmark parameters."""
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标。

### Lines 25-43: Method `BenchmarkConfig.__init__` / 方法 `BenchmarkConfig.__init__`
```python
    def __init__(self):
        # Common benchmark settings
        self.server_type = "HTTP"
        self.rps_values = [70]
        self.duration_secs_values = [60]
        self.num_unique_requests = 100
        self.distribution = "POISSON"  # Options: "CONSTANT", "POISSON"
        self.profile = False

        # Garbage Collection Control
        self.freeze_gc = True  # Enable/disable garbage collection freezing

        # Profiler configuration
        self.profiler_dir = (
            os.getcwd()
        )  # Default profiler output directory (current working directory)

        # Special token for text generation
        self.special_replicated_token = "<|im_start|>"
```
**EN:** `BenchmarkConfig.__init__` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. Notable calls include `os.getcwd`.
**CN:** `BenchmarkConfig.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `os.getcwd`。

### Lines 46-84: Function `generate_text_with_token_count` / 函数 `generate_text_with_token_count`
```python
def generate_text_with_token_count(
    model_path: str,
    num_tokens: int,
    special_token: str = "<|im_start|>",
    tokenizer: Optional[Any] = None,
) -> str:
    """
    Generate text with precise token count using a replicated token.

    Args:
        model_path: Path to the model for tokenizer
        num_tokens: Target number of tokens
        special_token: Token to replicate
        tokenizer: Optional pre-loaded tokenizer to avoid repeated loading

    Returns:
        Generated text with approximately the target token count
    """
    if tokenizer is None:
        tokenizer = AutoTokenizer.from_pretrained(model_path)

    # Verify token count
    special_token_count = len(tokenizer.encode(special_token, add_special_tokens=False))

    if special_token_count == 1:
        # Simple case: token maps to exactly 1 token
        return special_token * num_tokens
    else:
        print(f"Special token '{special_token}' produces {special_token_count} tokens")
        # Handle case where special token produces multiple tokens
        repetitions = (num_tokens + special_token_count - 1) // special_token_count
        text = special_token * repetitions

        # Verify we got the expected token count
        actual_tokens = len(tokenizer.encode(text, add_special_tokens=False))
        if actual_tokens < num_tokens:
            print(f"Warning: Generated {actual_tokens} tokens, expected {num_tokens}")

        return text
```
**EN:** `generate_text_with_token_count` is a function that processes tokenized prompts or decoded outputs. The docstring frames it as: Generate text with precise token count using a replicated token. It returns `special_token * num_tokens` to the caller. Notable calls include `len`, `AutoTokenizer.from_pretrained`, `tokenizer.encode`.
**CN:** `generate_text_with_token_count` 是一个函数，用于处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `special_token * num_tokens`。其中较关键的调用包括 `len`, `AutoTokenizer.from_pretrained`, `tokenizer.encode`。

### Lines 87-103: Function `setup_profiler` / 函数 `setup_profiler`
```python
def setup_profiler(config: BenchmarkConfig, benchmark_name: str) -> None:
    """
    Set up profiler environment if profiling is enabled.

    Args:
        config: Benchmark configuration
        benchmark_name: Name of the benchmark (used in directory path)
    """
    if config.profile:
        # Create benchmark-specific subdirectory
        profiler_path = os.path.join(
            config.profiler_dir, benchmark_name.lower().replace("_", "-")
        )
        os.environ["SGLANG_TORCH_PROFILER_DIR"] = profiler_path
        print(f"Profiler enabled. Output directory: {profiler_path}")
    else:
        print("Profiler disabled")
```
**EN:** `setup_profiler` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and computes evaluation scores and aggregate statistics. The docstring frames it as: Set up profiler environment if profiling is enabled. Notable calls include `os.path.join`, `print`, `benchmark_name.lower().replace`.
**CN:** `setup_profiler` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `os.path.join`, `print`, `benchmark_name.lower().replace`。

### Lines 106-184: Function `prepare_all_requests_parallel` / 函数 `prepare_all_requests_parallel`
```python
def prepare_all_requests_parallel(
    num_requests: int,
    item_count: int,
    build_request_func: Callable[[int, int], Tuple[int, Any]],
    config: BenchmarkConfig,
    description: str = "requests",
) -> List[Any]:
    """
    Generic function to generate unique requests in parallel, then reuse them.

    Args:
        num_requests: Total number of requests needed
        item_count: Number of items per request (batch size)
        build_request_func: Function that takes (index, item_count) and returns (index, request_data)
        config: Benchmark configuration
        description: Description for progress bars

    Returns:
        List of request data objects
    """

    def build_request_wrapper(index):
        """Wrapper to call the provided build_request_func."""
        try:
            return build_request_func(index, item_count)
        except Exception as e:
            print(f"Error building request {index}: {e}")
            return (index, None)

    # Generate only the unique requests
    unique_requests = [None] * config.num_unique_requests
    max_workers = min(8, os.cpu_count() or 1)  # Limit to 8 threads max

    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = []
        for i in tqdm(
            range(config.num_unique_requests),
            desc=f"Submitting {description} generation tasks",
        ):
            future = executor.submit(build_request_wrapper, i)
            futures.append(future)

        # Collect results as they complete
        for f in tqdm(
            concurrent.futures.as_completed(futures),
            desc=f"Building unique {description}",
            total=config.num_unique_requests,
        ):
            try:
                index, req_data = f.result()
                if req_data is not None:
                    unique_requests[index] = req_data
                else:
                    print(f"Failed to build request {index}")
            except Exception as e:
                print(f"Error processing request result: {e}")

    # Check if we have any valid requests
    valid_requests = [req for req in unique_requests if req is not None]
    if not valid_requests:
        raise RuntimeError("Failed to generate any valid requests")

    print(
        f"Successfully generated {len(valid_requests)} out of "
        f"{config.num_unique_requests} unique {description}"
    )

    # Create the full request list by cycling through unique requests
    print(
        f"Reusing {len(valid_requests)} unique {description} to create "
        f"{num_requests} total requests..."
    )
    all_requests = []
    for i in tqdm(range(num_requests), desc=f"Reusing {description}"):
        unique_index = i % len(valid_requests)
        all_requests.append(valid_requests[unique_index])

    print(f"All {description} prepared.\n")
    return all_requests
```
**EN:** `prepare_all_requests_parallel` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Generic function to generate unique requests in parallel, then reuse them. It returns `all_requests` to the caller. Notable calls include `min`, `print`, `tqdm`.
**CN:** `prepare_all_requests_parallel` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `all_requests`。其中较关键的调用包括 `min`, `print`, `tqdm`。

### Lines 187-205: Async function `sleep_with_distribution` / 异步函数 `sleep_with_distribution`
```python
async def sleep_with_distribution(distribution: str, rps: float) -> None:
    """
    Sleep according to the specified distribution pattern.

    Args:
        distribution: "CONSTANT" or "POISSON"
        rps: Requests per second rate
    """
    if distribution == "CONSTANT":
        interval = 1 / rps
        await asyncio.sleep(interval)
    elif distribution == "POISSON":
        # For Poisson process, inter-arrival times follow exponential distribution
        interval = random.expovariate(rps)
        await asyncio.sleep(interval)
    else:
        raise ValueError(
            f"Unknown distribution: {distribution}. Use 'CONSTANT' or 'POISSON'."
        )
```
**EN:** `sleep_with_distribution` is a async function that sends requests to serving or OpenAI-compatible APIs and coordinates asynchronous or parallel execution. The docstring frames it as: Sleep according to the specified distribution pattern. Notable calls include `asyncio.sleep`, `random.expovariate`, `ValueError`.
**CN:** `sleep_with_distribution` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `asyncio.sleep`, `random.expovariate`, `ValueError`。

### Lines 208-218: Function `build_http_request_json` / 函数 `build_http_request_json`
```python
def build_http_request_json(request_data: Any) -> str:
    """
    Generic function to build HTTP request JSON.

    Args:
        request_data: The data to serialize to JSON

    Returns:
        JSON string representation of the request data
    """
    return json.dumps(request_data)
```
**EN:** `build_http_request_json` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. The docstring frames it as: Generic function to build HTTP request JSON. It returns `json.dumps(request_data)` to the caller. Notable calls include `json.dumps`.
**CN:** `build_http_request_json` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `json.dumps(request_data)`。其中较关键的调用包括 `json.dumps`。

### Lines 221-281: Async function `make_http_call` / 异步函数 `make_http_call`
```python
async def make_http_call(
    session: aiohttp.ClientSession,
    request_data: Any,
    request_id: int,
    results_queue: asyncio.Queue,
    http_url: str,
    response_validator: Callable[[Dict[str, Any]], bool],
    api_name: str = "API",
) -> None:
    """
    Generic HTTP call function for API requests.

    Args:
        session: aiohttp client session
        request_data: Data to send in the request
        request_id: Unique identifier for this request
        results_queue: Queue to put results
        http_url: URL to send the request to
        response_validator: Function to validate the response JSON
        api_name: Name of the API for error messages
    """
    try:
        start_time = asyncio.get_running_loop().time()

        request_json = build_http_request_json(request_data)
        headers = {"Content-Type": "application/json"}

        async with session.post(http_url, data=request_json, headers=headers) as resp:
            resp_text = await resp.text()

            if resp.status != 200:
                print(
                    f"[HTTP] {api_name} Request {request_id} failed with status "
                    f"{resp.status}: {resp_text}"
                )
                completion_time = asyncio.get_running_loop().time()
                await results_queue.put((request_id, 0, False, completion_time))
                return

            # Parse and validate response
            try:
                response_data = json.loads(resp_text)
                success = response_validator(response_data)
                if not success:
                    print(
                        f"[HTTP] {api_name} Request {request_id} failed response validation"
                    )
            except json.JSONDecodeError:
                print(
                    f"[HTTP] {api_name} Request {request_id} failed to parse JSON response"
                )
                success = False

        completion_time = asyncio.get_running_loop().time()
        elapsed_time = (completion_time - start_time) * 1000
        await results_queue.put((request_id, elapsed_time, success, completion_time))

    except Exception as e:
        print(f"[HTTP] {api_name} Error for request {request_id}: {e}")
        completion_time = asyncio.get_running_loop().time()
        await results_queue.put((request_id, 0, False, completion_time))
```
**EN:** `make_http_call` is a async function that sends requests to serving or OpenAI-compatible APIs, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. The docstring frames it as: Generic HTTP call function for API requests. Notable calls include `asyncio.get_running_loop().time`, `build_http_request_json`, `session.post`.
**CN:** `make_http_call` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `asyncio.get_running_loop().time`, `build_http_request_json`, `session.post`。

### Lines 284-328: Async function `send_profile_request` / 异步函数 `send_profile_request`
```python
async def send_profile_request(
    profile_text: str, http_url: str, session: Optional[aiohttp.ClientSession] = None
) -> None:
    """
    Send a profile request (START_PROFILE or STOP_PROFILE) and wait for completion.

    Args:
        profile_text: "START_PROFILE" or "STOP_PROFILE"
        http_url: Base HTTP URL (will derive profile endpoints from this)
        session: Optional aiohttp session to use
    """
    try:
        if session:
            print(f"Sending {profile_text} request via HTTP...")

            # Determine the correct endpoint
            if "/v1/" in http_url:
                base_url = http_url.rsplit("/v1/", 1)[0]  # Remove /v1/xxx
            else:
                base_url = http_url.rsplit("/", 1)[0]  # Remove last path component

            if profile_text == "START_PROFILE":
                endpoint_url = f"{base_url}/start_profile"
            elif profile_text == "STOP_PROFILE":
                endpoint_url = f"{base_url}/stop_profile"
            else:
                print(f"Unknown profile request: {profile_text}")
                return

            headers = {"Content-Type": "application/json"}

            async with session.post(endpoint_url, headers=headers) as resp:
                resp_text = await resp.text()
                if resp.status == 200:
                    print(f"{profile_text} request completed")
                else:
                    print(
                        f"{profile_text} request failed with status "
                        f"{resp.status}: {resp_text}"
                    )
        else:
            print(f"Cannot send {profile_text} request - missing session")

    except Exception as e:
        print(f"Error sending {profile_text} request: {e}")
```
**EN:** `send_profile_request` is a async function that sends requests to serving or OpenAI-compatible APIs, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Send a profile request (START_PROFILE or STOP_PROFILE) and wait for completion. Notable calls include `print`, `session.post`, `http_url.rsplit`.
**CN:** `send_profile_request` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `session.post`, `http_url.rsplit`。

### Lines 331-356: Async function `call_freeze_gc_http` / 异步函数 `call_freeze_gc_http`
```python
async def call_freeze_gc_http(session: aiohttp.ClientSession, http_url: str) -> None:
    """
    Call the /freeze_gc HTTP endpoint.

    Args:
        session: aiohttp client session
        http_url: Base HTTP URL to derive the freeze_gc endpoint from
    """
    try:
        # Derive freeze_gc endpoint from the API URL
        if "/v1/" in http_url:
            freeze_gc_url = http_url.rsplit("/v1/", 1)[0] + "/freeze_gc"
        else:
            freeze_gc_url = http_url.rsplit("/", 1)[0] + "/freeze_gc"

        print(f"Calling freeze_gc endpoint: {freeze_gc_url}")

        async with session.post(freeze_gc_url) as resp:
            if resp.status == 200:
                print("freeze_gc called successfully")
            else:
                resp_text = await resp.text()
                print(f"freeze_gc failed with status {resp.status}: {resp_text}")

    except Exception as e:
        print(f"Failed to call freeze_gc: {e}")
```
**EN:** `call_freeze_gc_http` is a async function that sends requests to serving or OpenAI-compatible APIs and coordinates asynchronous or parallel execution. The docstring frames it as: Call the /freeze_gc HTTP endpoint. Notable calls include `print`, `session.post`, `http_url.rsplit`.
**CN:** `call_freeze_gc_http` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `session.post`, `http_url.rsplit`。

### Lines 359-395: Async function `send_warmup_requests` / 异步函数 `send_warmup_requests`
```python
async def send_warmup_requests(
    session: aiohttp.ClientSession,
    http_url: str,
    build_warmup_request_func: Callable[[], Any],
    num_warmup: int = 3,
) -> None:
    """
    Send warmup requests to HTTP server.

    Args:
        session: aiohttp client session
        http_url: URL to send warmup requests to
        build_warmup_request_func: Function that returns a warmup request object
        num_warmup: Number of warmup requests to send
    """
    print(f"Sending {num_warmup} HTTP warmup requests...")

    for i in range(num_warmup):
        try:
            warmup_data = build_warmup_request_func()
            request_json = build_http_request_json(warmup_data)
            headers = {"Content-Type": "application/json"}

            async with session.post(
                http_url, data=request_json, headers=headers
            ) as resp:
                if resp.status == 200:
                    print(f"Warmup request {i+1}/{num_warmup} completed successfully")
                else:
                    print(
                        f"Warmup request {i+1}/{num_warmup} failed with status {resp.status}"
                    )

        except Exception as e:
            print(f"Warmup request {i+1}/{num_warmup} failed with error: {e}")

    print("HTTP warmup requests completed")
```
**EN:** `send_warmup_requests` is a async function that sends requests to serving or OpenAI-compatible APIs and loads, filters, or serializes benchmark datasets. The docstring frames it as: Send warmup requests to HTTP server. Notable calls include `print`, `range`, `build_warmup_request_func`.
**CN:** `send_warmup_requests` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `range`, `build_warmup_request_func`。

### Lines 398-427: Async function `perform_global_warmup_and_freeze` / 异步函数 `perform_global_warmup_and_freeze`
```python
async def perform_global_warmup_and_freeze(
    config: BenchmarkConfig,
    http_url: str,
    build_warmup_request_func: Callable[[], Any],
) -> None:
    """
    Perform warmup and optionally GC freeze operations once before all benchmark runs.

    Args:
        config: Benchmark configuration
        http_url: URL for API requests
        build_warmup_request_func: Function that returns a warmup request object
    """
    print("=" * 80)
    print(f"PERFORMING GLOBAL WARMUP{' AND GC FREEZE' if config.freeze_gc else ''}")
    print("=" * 80)

    print(f"Performing HTTP warmup{' and GC freeze' if config.freeze_gc else ''}...")
    async with aiohttp.ClientSession() as session:
        await send_warmup_requests(session, http_url, build_warmup_request_func)
        if config.freeze_gc:
            await call_freeze_gc_http(session, http_url)
        print(
            f"HTTP warmup{' and GC freeze' if config.freeze_gc else ''} completed successfully."
        )

    print(
        f"Global warmup{' and GC freeze' if config.freeze_gc else ''} operations completed."
    )
    print("=" * 80)
```
**EN:** `perform_global_warmup_and_freeze` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Perform warmup and optionally GC freeze operations once before all benchmark runs. Notable calls include `print`, `aiohttp.ClientSession`, `send_warmup_requests`.
**CN:** `perform_global_warmup_and_freeze` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `aiohttp.ClientSession`, `send_warmup_requests`。

### Lines 430-570: Async function `process_results` / 异步函数 `process_results`
```python
async def process_results(
    results_queue: asyncio.Queue,
    num_requests: int,
    send_duration: float,
    total_duration: float,
    rps: int,
    duration_secs: int,
    item_count: int,
    test_start_time: float,
    config: BenchmarkConfig,
    http_mode: str = "UNKNOWN",
) -> List[Dict[str, Any]]:
    """
    Process benchmark results and group them by minute intervals.

    Args:
        results_queue: Queue containing result tuples
        num_requests: Total number of requests sent
        send_duration: Time taken to send all requests
        total_duration: Total time for all requests to complete
        rps: Target requests per second
        duration_secs: Test duration in seconds
        item_count: Number of items per request
        test_start_time: Start time of the test
        config: Benchmark configuration
        http_mode: Description of the HTTP mode/API being tested

    Returns:
        List of dictionaries containing minute-by-minute results
    """
    all_results = []

    # Collect all results
    for _ in range(num_requests):
        result = await results_queue.get()
        request_id, elapsed_time, success, completion_time = result
        all_results.append(
            {
                "request_id": request_id,
                "elapsed_time": elapsed_time,
                "success": success,
                "completion_time": completion_time,
            }
        )

    # Group results by minute intervals
    minute_results = []
    num_minutes = int(duration_secs // 60) + (1 if duration_secs % 60 > 0 else 0)

    for minute in range(num_minutes):
        minute_start = test_start_time + (minute * 60)
        minute_end = test_start_time + ((minute + 1) * 60)

        # Filter results that completed in this minute
        minute_data = [
            r for r in all_results if minute_start <= r["completion_time"] < minute_end
        ]

        response_times = [r["elapsed_time"] for r in minute_data if r["success"]]
        successful_requests = len([r for r in minute_data if r["success"]])
        failed_requests = len([r for r in minute_data if not r["success"]])

        avg_response_time = mean(response_times) if response_times else 0

        # Calculate percentiles using numpy
        if response_times:
            p50 = np.percentile(response_times, 50)
            p90 = np.percentile(response_times, 90)
            p99 = np.percentile(response_times, 99)
        else:
            p50 = p90 = p99 = 0

        minute_result = {
            "test_duration_secs": duration_secs,
            "minute_interval": minute + 1,
            "target_rps": rps,
            "item_count": item_count,
            "server_type": config.server_type,
            "distribution": config.distribution,
            "unique_requests": config.num_unique_requests,
            "total_requests": len(minute_data),
            "successful_requests": successful_requests,
            "failed_requests": failed_requests,
            "send_duration_secs": send_duration,
            "total_duration_secs": total_duration,
            "avg_response_time_ms": avg_response_time,
            "p50_response_time_ms": p50,
            "p90_response_time_ms": p90,
            "p99_response_time_ms": p99,
        }

        minute_results.append(minute_result)

        print(
            f"\nMinute {minute + 1} Summary for RPS {rps}, "
            f"Duration {duration_secs}s, Item Count {item_count}:"
        )
        print(f"  Requests completed in minute: {len(minute_data)}")
        print(f"  Successful requests:   {successful_requests}")
        print(f"  Failed requests:       {failed_requests}")
        print(f"  Average response time: {avg_response_time:.2f} ms")
        print(f"  P50 response time:     {p50:.2f} ms")
        print(f"  P90 response time:     {p90:.2f} ms")
        print(f"  P99 response time:     {p99:.2f} ms")

    # Print overall summary
    all_response_times = [r["elapsed_time"] for r in all_results if r["success"]]
    total_successful = len([r for r in all_results if r["success"]])
    total_failed = len([r for r in all_results if not r["success"]])

    overall_avg = mean(all_response_times) if all_response_times else 0
    if all_response_times:
        overall_p50 = np.percentile(all_response_times, 50)
        overall_p90 = np.percentile(all_response_times, 90)
        overall_p99 = np.percentile(all_response_times, 99)
    else:
        overall_p50 = overall_p90 = overall_p99 = 0

    print(
        f"\nOverall Summary for RPS {rps}, Duration {duration_secs}s, "
        f"Item Count {item_count}:"
    )
    print(f"  Test duration:         {duration_secs} seconds")
    print(f"  Server type:           {config.server_type}")
    print(f"  HTTP mode:             {http_mode}")
    print(f"  Target RPS:            {rps}")
    print(f"  Achieved RPS:          {len(all_results) / total_duration:.2f}")
    print(f"  Item count:            {item_count}")
    print(f"  Distribution:          {config.distribution}")
    print(f"  Unique requests generated: {config.num_unique_requests}")
    print(f"  Total requests sent:   {num_requests}")
    print(f"  Successful requests:   {total_successful}")
    print(f"  Failed requests:       {total_failed}")
    print(f"  Time to send all requests: {send_duration:.2f} seconds")
    print(f"  Time for all requests to complete: {total_duration:.2f} seconds")
    print(f"  Average response time: {overall_avg:.2f} ms")
    print(f"  P50 response time:     {overall_p50:.2f} ms")
    print(f"  P90 response time:     {overall_p90:.2f} ms")
    print(f"  P99 response time:     {overall_p99:.2f} ms\n")

    return minute_results
```
**EN:** `process_results` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Process benchmark results and group them by minute intervals. It returns `minute_results` to the caller. Notable calls include `range`, `len`, `print`.
**CN:** `process_results` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `minute_results`。其中较关键的调用包括 `range`, `len`, `print`。

### Lines 573-625: Function `print_csv_results` / 函数 `print_csv_results`
```python
def print_csv_results(all_results: List[Dict[str, Any]]) -> None:
    """
    Print benchmark results in CSV format.

    Args:
        all_results: List of result dictionaries from process_results
    """
    print("\n" + "=" * 80)
    print("FINAL CSV RESULTS:")
    print("=" * 80)

    # CSV Header
    headers = [
        "test_duration_secs",
        "minute_interval",
        "target_rps",
        "item_count",
        "server_type",
        "distribution",
        "unique_requests",
        "total_requests",
        "successful_requests",
        "failed_requests",
        "send_duration_secs",
        "total_duration_secs",
        "avg_response_time_ms",
        "p50_response_time_ms",
        "p90_response_time_ms",
        "p99_response_time_ms",
    ]
    print(",".join(headers))

    # CSV Data
    for result in all_results:
        row = [
            result["test_duration_secs"],
            result["minute_interval"],
            result["target_rps"],
            result["item_count"],
            result["server_type"],
            result["distribution"],
            result["unique_requests"],
            result["total_requests"],
            result["successful_requests"],
            result["failed_requests"],
            f"{result['send_duration_secs']:.2f}",
            f"{result['total_duration_secs']:.2f}",
            f"{result['avg_response_time_ms']:.2f}",
            f"{result['p50_response_time_ms']:.2f}",
            f"{result['p90_response_time_ms']:.2f}",
            f"{result['p99_response_time_ms']:.2f}",
        ]
        print(",".join(map(str, row)))
```
**EN:** `print_csv_results` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Print benchmark results in CSV format. Notable calls include `print`, `','.join`, `map`.
**CN:** `print_csv_results` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `','.join`, `map`。

### Lines 628-693: Async function `run_benchmark_main` / 异步函数 `run_benchmark_main`
```python
async def run_benchmark_main(
    config: BenchmarkConfig,
    run_single_benchmark_func,
    benchmark_name: str,
    http_url: str,
    item_count_values: List[int],
    additional_info: Optional[Dict[str, Any]] = None,
    build_warmup_request_func: Optional[Callable[[], Any]] = None,
) -> None:
    """
    Main benchmark orchestration function.

    Args:
        config: Benchmark configuration
        run_single_benchmark_func: Async function to run a single benchmark
        benchmark_name: Name of the benchmark (e.g., "SCORING", "EMBEDDINGS")
        http_url: URL of the API endpoint
        item_count_values: List of item counts to test
        additional_info: Additional information to print in the header
        build_warmup_request_func: Optional function to build warmup requests
    """
    total_combinations = (
        len(config.duration_secs_values)
        * len(config.rps_values)
        * len(item_count_values)
    )

    print(
        f"Running benchmarks for {len(config.duration_secs_values)} duration "
        f"values, {len(config.rps_values)} RPS values, and "
        f"{len(item_count_values)} item count values = "
        f"{total_combinations} total combinations"
    )
    print(f"Server Type: {config.server_type}")
    print(f"HTTP Mode: {benchmark_name}")
    print(f"API URL: {http_url}")

    if additional_info:
        for key, value in additional_info.items():
            print(f"{key}: {value}")

    print(f"Items per request (batch size): {item_count_values}")
    print(f"Profiling Enabled: {config.profile}")
    print(f"Duration values: {config.duration_secs_values}")
    print(f"RPS values: {config.rps_values}")
    print(f"Item count values: {item_count_values}")
    print("=" * 80)

    # Set up profiler environment
    setup_profiler(config, benchmark_name)

    # Perform global warmup and GC freeze operations if warmup function is provided
    if build_warmup_request_func is not None:
        await perform_global_warmup_and_freeze(
            config, http_url, build_warmup_request_func
        )

    all_results = []

    for duration_secs in config.duration_secs_values:
        for rps in config.rps_values:
            for item_count in item_count_values:
                result = await run_single_benchmark_func(rps, duration_secs, item_count)
                all_results.extend(result)  # Extend with minute results

    print_csv_results(all_results)
```
**EN:** `run_benchmark_main` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Main benchmark orchestration function. Notable calls include `print`, `setup_profiler`, `print_csv_results`.
**CN:** `run_benchmark_main` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `setup_profiler`, `print_csv_results`。

### Lines 696-814: Async function `run_generic_benchmark` / 异步函数 `run_generic_benchmark`
```python
async def run_generic_benchmark(
    rps: int,
    duration_secs: int,
    item_count: int,
    config: BenchmarkConfig,
    http_url: str,
    build_request_func: Callable[[int, int], Tuple[int, Any]],
    response_validator: Callable[[Dict[str, Any]], bool],
    api_name: str,
    request_description: str = "requests",
) -> List[Dict[str, Any]]:
    """
    Generic benchmark runner that can be used for different APIs.

    Args:
        rps: Requests per second
        duration_secs: Duration of the test in seconds
        item_count: Number of items per request (batch size)
        config: Benchmark configuration
        http_url: URL of the API endpoint
        build_request_func: Function to build individual requests
        response_validator: Function to validate API responses
        api_name: Name of the API for logging
        request_description: Description for progress bars

    Returns:
        List of dictionaries containing minute-by-minute results
    """
    num_requests = int(rps * duration_secs)
    print(
        f"Starting benchmark with RPS={rps}, Duration={duration_secs}s, "
        f"Item Count={item_count}, num_requests={num_requests}"
    )
    print(f"Server Type: {config.server_type}")
    print(f"HTTP Mode: {api_name}")
    print(f"Profiling Enabled: {config.profile}")

    # Build requests in parallel (unmeasured)
    all_requests = prepare_all_requests_parallel(
        num_requests, item_count, build_request_func, config, request_description
    )

    results_queue = asyncio.Queue()
    tasks = []

    # Track timing for sending requests
    send_start_time = asyncio.get_running_loop().time()

    # HTTP implementation
    async with aiohttp.ClientSession(
        timeout=aiohttp.ClientTimeout(total=300)
    ) as session:

        # Send START_PROFILE if profiling is enabled
        if config.profile:
            await send_profile_request("START_PROFILE", http_url, session=session)

        # Add progress bar for sending requests
        with tqdm(
            total=len(all_requests),
            desc=f"Sending HTTP {request_description} at {rps} RPS",
            unit="req",
        ) as pbar:
            for i, request_data in enumerate(all_requests):
                request_id = i + 1
                tasks.append(
                    asyncio.create_task(
                        make_http_call(
                            session,
                            request_data,
                            request_id,
                            results_queue,
                            http_url,
                            response_validator,
                            api_name,
                        )
                    )
                )

                # Update progress bar
                pbar.update(1)

                # Throttle based on distribution
                if i < len(all_requests) - 1:
                    await sleep_with_distribution(config.distribution, rps)

        send_end_time = asyncio.get_running_loop().time()
        send_duration = send_end_time - send_start_time

        # Wait for all requests to complete with progress tracking
        print(f"Waiting for {len(tasks)} HTTP {request_description} to complete...")
        with tqdm(
            total=len(tasks), desc=f"Completing HTTP {request_description}", unit="req"
        ) as completion_pbar:
            completed_tasks = []
            for task in asyncio.as_completed(tasks):
                await task
                completed_tasks.append(task)
                completion_pbar.update(1)

        # Send STOP_PROFILE if profiling is enabled
        if config.profile:
            await send_profile_request("STOP_PROFILE", http_url, session=session)

    completion_end_time = asyncio.get_running_loop().time()
    total_duration = completion_end_time - send_start_time

    return await process_results(
        results_queue,
        num_requests,
        send_duration,
        total_duration,
        rps,
        duration_secs,
        item_count,
        send_start_time,
        config,
        api_name,
    )
```
**EN:** `run_generic_benchmark` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Generic benchmark runner that can be used for different APIs. It returns `await process_results(results_queue, num_requests, send_duration, total_duration, rps, duration_s...` to the caller. Notable calls include `int`, `print`, `prepare_all_requests_parallel`.
**CN:** `run_generic_benchmark` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `await process_results(results_queue, num_requests, send_duration, total_duration, rps, duration_s...`。其中较关键的调用包括 `int`, `print`, `prepare_all_requests_parallel`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `concurrent.futures`, `json`, `os`, `random`, `statistics`, `typing`
- **Third-party / 第三方依赖**: `aiohttp`, `numpy`, `tqdm`, `transformers`
