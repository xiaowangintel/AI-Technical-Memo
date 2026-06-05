# bench_serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/bench_serving.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache serving. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hicache serving 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 6-41: Imports and setup / 导入与初始化
```python
"""
Benchmark online serving with dynamic requests.

Usage:
python3 -m sglang.bench_serving --backend sglang --num-prompt 10

python3 -m sglang.bench_serving --backend sglang --dataset-name random --num-prompts 3000 --random-input 1024 --random-output 1024 --random-range-ratio 0.5
python3 -m sglang.bench_serving --backend sglang --dataset-name random --request-rate-range 1,2,4,8,16,32 --random-input 4096 --random-output 1024 --random-range-ratio 0.125 --multi
"""

import argparse
import asyncio
import json
import os
import random
import sys
import time
import traceback
import warnings
from argparse import ArgumentParser
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any, AsyncGenerator, Dict, List, Optional, Tuple

import aiohttp
import numpy as np
import requests
from data_processing import MsgContent, SampleOutput, get_dataset
from tqdm.asyncio import tqdm
from transformers import PreTrainedTokenizerBase

from sglang.benchmark.utils import get_tokenizer, remove_prefix, set_ulimit

AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=20 * 60 * 60)

global args
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 45-45: Class `RequestFuncInput` declaration / 类 `RequestFuncInput` 声明
```python
class RequestFuncInput:
```
**EN:** This block introduces class `RequestFuncInput`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `RequestFuncInput`，用于通过统一接口组织相关行为。

### Lines 46-54: Class-level state / 类级状态
```python
    prompts: List[Tuple[MsgContent, int, int]]
    api_url: str
    model: str
    lora_name: str
    extra_request_body: Dict[str, Any]

    # For multiturn chat, store the context
    prev_messages: List = field(default_factory=list)
    finished_prompts: int = 0
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。

### Lines 58-58: Class `RequestFuncOutput` declaration / 类 `RequestFuncOutput` 声明
```python
class RequestFuncOutput:
```
**EN:** This block introduces class `RequestFuncOutput`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `RequestFuncOutput`，用于通过统一接口组织相关行为。

### Lines 59-67: Class-level state / 类级状态
```python
    generated_text: List[str] = field(default_factory=list)
    prompt_len: List[int] = field(default_factory=list)
    output_len: List[int] = field(default_factory=list)
    latency: List[float] = field(default_factory=list)
    ttft: List[float] = field(default_factory=list)
    itl: List[float] = field(default_factory=list)  # List of inter-token latencies

    success: bool = False
    error: str = ""
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it measures runtime latency, throughput, or other benchmark metrics and processes tokenized prompts or decoded outputs.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。

### Lines 71-201: Async function `async_request_openai_completions` / 异步函数 `async_request_openai_completions`
```python
async def async_request_openai_completions(
    request_func_input: RequestFuncInput,
    queue: asyncio.Queue,
    tokenizer: PreTrainedTokenizerBase,
    pbar: Optional[tqdm] = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith(
        "completions"
    ), "OpenAI Completions API URL must end with 'completions'."

    async with aiohttp.ClientSession(timeout=AIOHTTP_TIMEOUT) as session:
        payload = {
            "model": request_func_input.model,
            "temperature": 0.0,
            "best_of": 1,
            "stream": not args.disable_stream,
            "stream_options": {"include_usage": True},
            "ignore_eos": not args.disable_ignore_eos,
            **request_func_input.extra_request_body,
        }
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}",
        }

        output = RequestFuncOutput()

        prompt_idx = request_func_input.finished_prompts
        messages = request_func_input.prev_messages
        prompt, input_len, max_tokens = request_func_input.prompts[prompt_idx]
        prompt_len = sum(
            prompt[1] + prompt[2]  # input_len + output_len
            for prompt in request_func_input.prompts[:prompt_idx]
        )
        prompt_len += input_len

        # Messages
        messages.append(
            {
                "role": "user",
                "content": prompt,
            }
        )
        payload["messages"] = messages
        payload["max_tokens"] = max_tokens

        # output.prompt_len = request_func_input.prompt_len
        # print(payload)

        generated_text = ""
        ttft = 0.0
        st = time.perf_counter()
        most_recent_timestamp = st
        try:
            async with session.post(
                url=api_url, json=payload, headers=headers
            ) as response:
                if response.status == 200:
                    actual_prompt_len = prompt_len - 1
                    actual_output_len = 0
                    async for chunk_bytes in response.content:
                        chunk_bytes = chunk_bytes.strip()
                        if not chunk_bytes:
                            continue

                        chunk = remove_prefix(chunk_bytes.decode("utf-8"), "data: ")
                        latency = time.perf_counter() - st
                        if chunk == "[DONE]":
                            pass
                        else:
                            data = json.loads(chunk)
                            timestamp = time.perf_counter()
                            # NOTE: Some completion API might have a last
                            # usage summary response without a token so we
                            # want to check a token was generated
                            if data["usage"] is not None and len(data["usage"]) > 0:
                                actual_prompt_len = data["usage"]["prompt_tokens"]
                                actual_output_len = data["usage"]["completion_tokens"]
                                continue
                            delta = data["choices"][0]["delta"]

                            if delta.get("content", None):
                                # First token
                                if ttft == 0.0:
                                    ttft = time.perf_counter() - st
                                    output.ttft.append(ttft)

                                # Decoding phase
                                else:
                                    output.itl.append(timestamp - most_recent_timestamp)

                                generated_text += delta["content"]
                            most_recent_timestamp = timestamp

                    output.prompt_len.append(actual_prompt_len)  # truncate <s>
                    output.output_len.append(actual_output_len)
                    output.generated_text.append(generated_text)
                    output.success = True
                    output.latency.append(latency)

                    # Prepare for the new request
                    request_func_input.prompts[prompt_idx] = (
                        prompt,
                        input_len,
                        actual_output_len,  # changes from max_tokens to output_len
                    )
                    prompt_idx += 1
                    messages.append(
                        {
                            "role": "assistant",
                            "content": generated_text,
                        }
                    )

                    # Move the new request to the end of the queue
                    if prompt_idx < len(request_func_input.prompts):
                        request_func_input.finished_prompts = prompt_idx
                        request_func_input.prev_messages = messages
                        await queue.put(request_func_input)
                else:
                    output.error = response.reason or ""
                    output.success = False
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

    if pbar:
        pbar.update(1)
    return output
```
**EN:** `async_request_openai_completions` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. It returns `output` to the caller. Notable calls include `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`.
**CN:** `async_request_openai_completions` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。它会向调用方返回 `output`。其中较关键的调用包括 `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`。

### Lines 204-219: Async function `async_request_profile` / 异步函数 `async_request_profile`
```python
async def async_request_profile(api_url: str) -> RequestFuncOutput:
    async with aiohttp.ClientSession(timeout=AIOHTTP_TIMEOUT) as session:
        output = RequestFuncOutput()
        try:
            async with session.post(url=api_url) as response:
                if response.status == 200:
                    output.success = True
                else:
                    output.error = response.reason or ""
                    output.success = False
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

    return output
```
**EN:** `async_request_profile` is a async function that implements the core logic for this scope. It returns `output` to the caller. Notable calls include `aiohttp.ClientSession`, `RequestFuncOutput`, `session.post`.
**CN:** `async_request_profile` 是一个异步函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `output`。其中较关键的调用包括 `aiohttp.ClientSession`, `RequestFuncOutput`, `session.post`。

### Lines 222-226: Top-level execution logic / 顶层执行逻辑
```python
ASYNC_REQUEST_FUNCS = {
    "sglang": async_request_openai_completions,
    "vllm": async_request_openai_completions,
    "lmdeploy": async_request_openai_completions,
}
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers. In practice it sends requests to serving or OpenAI-compatible APIs.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求。

### Lines 230-230: Class `BenchmarkMetrics` declaration / 类 `BenchmarkMetrics` 声明
```python
class BenchmarkMetrics:
```
**EN:** This block introduces class `BenchmarkMetrics`, which measures runtime latency, throughput, or other benchmark metrics and computes evaluation scores and aggregate statistics.
**CN:** 该代码块引入类 `BenchmarkMetrics`，用于测量运行时延迟、吞吐或其他基准指标、计算评测分数与聚合统计结果。

### Lines 231-260: Class-level state / 类级状态
```python
    completed: int
    total_input: int
    total_output: int
    total_output_retokenized: int
    request_throughput: float
    input_throughput: float
    output_throughput: float
    output_throughput_retokenized: float
    total_throughput: float
    total_throughput_retokenized: float
    mean_ttft_ms: float
    median_ttft_ms: float
    std_ttft_ms: float
    p90_ttft_ms: float
    p99_ttft_ms: float
    mean_tpot_ms: float
    median_tpot_ms: float
    std_tpot_ms: float
    p90_tpot_ms: float
    p99_tpot_ms: float
    mean_itl_ms: float
    median_itl_ms: float
    std_itl_ms: float
    p90_itl_ms: float
    p99_itl_ms: float
    mean_e2e_latency_ms: float
    median_e2e_latency_ms: float
    std_e2e_latency_ms: float
    p99_e2e_latency_ms: float
    concurrency: float
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it measures runtime latency, throughput, or other benchmark metrics and processes tokenized prompts or decoded outputs.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。

### Lines 263-283: Async function `get_requests` / 异步函数 `get_requests`
```python
async def get_requests(
    input_requests_queue: asyncio.Queue,
    request_rate: float,
    num_actual_requests: int,
) -> AsyncGenerator[RequestFuncInput, None]:
    for _ in range(num_actual_requests):
        try:
            request = await asyncio.wait_for(
                input_requests_queue.get(), timeout=300
            )  # Wait for 5 minutes then abort
        except Exception as e:
            print(f"exception: {e}")
            break

        yield request

        if request_rate == float("inf"):
            continue

        interval = np.random.exponential(1.0 / request_rate)
        await asyncio.sleep(interval)
```
**EN:** `get_requests` is a async function that sends requests to serving or OpenAI-compatible APIs and coordinates asynchronous or parallel execution. Notable calls include `range`, `np.random.exponential`, `float`.
**CN:** `get_requests` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。其中较关键的调用包括 `range`, `np.random.exponential`, `float`。

### Lines 286-371: Function `calculate_metrics` / 函数 `calculate_metrics`
```python
def calculate_metrics(
    outputs: List[RequestFuncOutput],
    dur_s: float,
    tokenizer: PreTrainedTokenizerBase,
    backend: str,
) -> Tuple[BenchmarkMetrics, List[int]]:
    output_lens: List[int] = []
    retokenized_output_lens: List[int] = []
    total_input = 0
    completed = 0
    itls: List[float] = []
    tpots: List[float] = []
    ttfts: List[float] = []
    e2e_latencies: List[float] = []
    output_success = 0
    for i in range(len(outputs)):
        if outputs[i].success:
            output_success += 1
            assert len(outputs[i].generated_text) == len(outputs[i].latency)
            assert len(outputs[i].generated_text) == len(outputs[i].ttft)
            for j in range(len(outputs[i].generated_text)):
                output_len = outputs[i].output_len[j]
                output_lens.append(output_len)
                retokenized_output_len = len(
                    tokenizer.encode(
                        outputs[i].generated_text[j], add_special_tokens=False
                    )
                )
                retokenized_output_lens.append(retokenized_output_len)
                total_input += outputs[i].prompt_len[j]
                if output_len > 1:
                    tpots.append(
                        (outputs[i].latency[j] - outputs[i].ttft[j]) / (output_len - 1)
                    )

                completed += 1
            itls += outputs[i].itl
            ttfts += outputs[i].ttft
            e2e_latencies += outputs[i].latency

        else:
            output_lens.append(0)
            retokenized_output_lens.append(0)

    if completed == 0:
        warnings.warn(
            "All requests failed. This is likely due to a misconfiguration "
            "on the benchmark arguments.",
            stacklevel=2,
        )
    metrics = BenchmarkMetrics(
        completed=completed,
        total_input=total_input,
        total_output=sum(output_lens),
        total_output_retokenized=sum(retokenized_output_lens),
        request_throughput=completed / dur_s,
        input_throughput=total_input / dur_s,
        output_throughput=sum(output_lens) / dur_s,
        output_throughput_retokenized=sum(retokenized_output_lens) / dur_s,
        total_throughput=(total_input + sum(output_lens)) / dur_s,
        total_throughput_retokenized=(total_input + sum(retokenized_output_lens))
        / dur_s,
        mean_ttft_ms=np.mean(ttfts or 0)
        * 1000,  # ttfts is empty if streaming is not supported by backend
        median_ttft_ms=np.median(ttfts or 0) * 1000,
        std_ttft_ms=np.std(ttfts or 0) * 1000,
        p90_ttft_ms=np.percentile(ttfts or 0, 90) * 1000,
        p99_ttft_ms=np.percentile(ttfts or 0, 99) * 1000,
        mean_tpot_ms=np.mean(tpots or 0) * 1000,
        median_tpot_ms=np.median(tpots or 0) * 1000,
        std_tpot_ms=np.std(tpots or 0) * 1000,
        p90_tpot_ms=np.percentile(tpots or 0, 90) * 1000,
        p99_tpot_ms=np.percentile(tpots or 0, 99) * 1000,
        mean_itl_ms=np.mean(itls or 0) * 1000,
        median_itl_ms=np.median(itls or 0) * 1000,
        std_itl_ms=np.std(itls or 0) * 1000,
        p90_itl_ms=np.percentile(itls or 0, 90) * 1000,
        p99_itl_ms=np.percentile(itls or 0, 99) * 1000,
        mean_e2e_latency_ms=np.mean(e2e_latencies) * 1000,
        median_e2e_latency_ms=np.median(e2e_latencies) * 1000,
        std_e2e_latency_ms=np.std(e2e_latencies) * 1000,
        p99_e2e_latency_ms=np.percentile(e2e_latencies, 99) * 1000,
        concurrency=np.sum(e2e_latencies) / dur_s,
    )

    return metrics, output_lens
```
**EN:** `calculate_metrics` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. It returns `(metrics, output_lens)` to the caller. Notable calls include `range`, `BenchmarkMetrics`, `len`.
**CN:** `calculate_metrics` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。它会向调用方返回 `(metrics, output_lens)`。其中较关键的调用包括 `range`, `BenchmarkMetrics`, `len`。

### Lines 374-684: Async function `benchmark` / 异步函数 `benchmark`
```python
async def benchmark(
    backend: str,
    api_url: str,
    base_url: str,
    model_id: str,
    tokenizer: PreTrainedTokenizerBase,
    input_requests: SampleOutput,
    request_rate: float,
    max_concurrency: Optional[int],
    disable_tqdm: bool,
    lora_name: str,
    extra_request_body: Dict[str, Any],
    profile: bool,
    enable_shared_prefix: bool,
):
    if backend in ASYNC_REQUEST_FUNCS:
        request_func = ASYNC_REQUEST_FUNCS[backend]
    else:
        raise ValueError(f"Unknown backend: {backend}")

    # Limit concurrency
    # From https://github.com/vllm-project/vllm/pull/9390
    semaphore = asyncio.Semaphore(max_concurrency) if max_concurrency else None

    async def limited_request_func(request_func_input, queue, tokenizer, pbar):
        if semaphore is None:
            return await request_func(
                request_func_input=request_func_input,
                queue=queue,
                tokenizer=tokenizer,
                pbar=pbar,
            )
        async with semaphore:
            return await request_func(
                request_func_input=request_func_input,
                queue=queue,
                tokenizer=tokenizer,
                pbar=pbar,
            )

    num_actual_requests = sum(len(r) for r in input_requests)
    print(f"Num of shared prefixes or conversations: {len(input_requests)}")
    print(f"Num of total requests: {num_actual_requests}")

    # flatten the requests for shared prefix
    if enable_shared_prefix:
        input_requests = [[r] for requests in input_requests for r in requests]
    inputs_requests_queue = asyncio.Queue(maxsize=len(input_requests))
    print("Starting initial single prompt test run...")
    # NOTE: Just use the first request of the first conversation for warmup
    test_input = RequestFuncInput(
        model=model_id,
        prompts=input_requests[0][:1],
        api_url=api_url,
        lora_name=lora_name,
        extra_request_body=extra_request_body,
    )
    test_output = await request_func(
        request_func_input=test_input, queue=inputs_requests_queue, tokenizer=tokenizer
    )
    if not test_output.success:
        raise ValueError(
            "Initial test run failed - Please make sure benchmark arguments "
            f"are correctly specified. Error: {test_output.error}"
        )
    else:
        print("Initial test run completed. Starting main benchmark run...")

    # Check the states
    assert inputs_requests_queue.empty()

    # Flush cache
    if "sglang" in backend:
        requests.post(base_url + "/flush_cache")

    time.sleep(1.0)

    # Start profiler
    if profile:
        print("Starting profiler...")
        profile_output = await async_request_profile(
            api_url=base_url + "/start_profile"
        )
        if profile_output.success:
            print("Profiler started")

    for request in input_requests:
        request_func_input = RequestFuncInput(
            model=model_id,
            prompts=request,
            api_url=api_url,
            lora_name=lora_name,
            extra_request_body=extra_request_body,
        )
        inputs_requests_queue.put_nowait(request_func_input)
    if (
        not args.enable_multiturn
        and not args.enable_shared_prefix
        and not args.dataset_name == "generated-shared-prefix"
    ):
        assert len(input_requests) == num_actual_requests

    pbar = None if disable_tqdm else tqdm(total=num_actual_requests)

    benchmark_start_time = time.perf_counter()
    tasks: List[asyncio.Task] = []
    async for request in get_requests(
        inputs_requests_queue, request_rate, num_actual_requests
    ):
        tasks.append(
            asyncio.create_task(
                limited_request_func(
                    request_func_input=request,
                    queue=inputs_requests_queue,
                    tokenizer=tokenizer,
                    pbar=pbar,
                )
            )
        )
    outputs: List[RequestFuncOutput] = await asyncio.gather(*tasks)

    # Stop profiler
    if profile:
        print("Stopping profiler...")
        profile_output = await async_request_profile(api_url=base_url + "/stop_profile")
        if profile_output.success:
            print("Profiler stopped")

    if pbar is not None:
        pbar.close()

    # Compute metrics and print results
    benchmark_duration = time.perf_counter() - benchmark_start_time
    metrics, output_lens = calculate_metrics(
        outputs=outputs,
        dur_s=benchmark_duration,
        tokenizer=tokenizer,
        backend=backend,
    )

    print("\n{s:{c}^{n}}".format(s=" Serving Benchmark Result ", n=50, c="="))
    print("{:<40} {:<10}".format("Backend:", backend))
    print("{:<40} {:<10}".format("Traffic request rate:", request_rate))
    print(
        "{:<40} {:<10}".format(
            "Max request concurrency:",
            max_concurrency if max_concurrency else "not set",
        )
    )
    print("{:<40} {:<10}".format("Successful requests:", metrics.completed))
    print("{:<40} {:<10.2f}".format("Benchmark duration (s):", benchmark_duration))
    print("{:<40} {:<10}".format("Total input tokens:", metrics.total_input))
    print("{:<40} {:<10}".format("Total generated tokens:", metrics.total_output))
    print(
        "{:<40} {:<10}".format(
            "Total generated tokens (retokenized):", metrics.total_output_retokenized
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Request throughput (req/s):", metrics.request_throughput
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Input token throughput (tok/s):", metrics.input_throughput
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Output token throughput (tok/s):", metrics.output_throughput
        )
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Total token throughput (tok/s):", metrics.total_throughput
        )
    )
    print("{:<40} {:<10.2f}".format("Concurrency:", metrics.concurrency))
    print("{s:{c}^{n}}".format(s="End-to-End Latency", n=50, c="-"))
    print(
        "{:<40} {:<10.2f}".format("Mean E2E Latency (ms):", metrics.mean_e2e_latency_ms)
    )
    print(
        "{:<40} {:<10.2f}".format(
            "Median E2E Latency (ms):", metrics.median_e2e_latency_ms
        )
    )
    print("{s:{c}^{n}}".format(s="Time to First Token", n=50, c="-"))
    print("{:<40} {:<10.2f}".format("Mean TTFT (ms):", metrics.mean_ttft_ms))
    print("{:<40} {:<10.2f}".format("Median TTFT (ms):", metrics.median_ttft_ms))
    print("{:<40} {:<10.2f}".format("P90 TTFT (ms):", metrics.p90_ttft_ms))
    print("{:<40} {:<10.2f}".format("P99 TTFT (ms):", metrics.p99_ttft_ms))
    print(
        "{s:{c}^{n}}".format(s="Time per Output Token (excl. 1st token)", n=50, c="-")
    )
    print("{:<40} {:<10.2f}".format("Mean TPOT (ms):", metrics.mean_tpot_ms))
    print("{:<40} {:<10.2f}".format("Median TPOT (ms):", metrics.median_tpot_ms))
    print("{:<40} {:<10.2f}".format("P90 TPOT (ms):", metrics.p90_tpot_ms))
    print("{:<40} {:<10.2f}".format("P99 TPOT (ms):", metrics.p99_tpot_ms))
    print("{s:{c}^{n}}".format(s="Inter-token Latency", n=50, c="-"))
    print("{:<40} {:<10.2f}".format("Mean ITL (ms):", metrics.mean_itl_ms))
    print("{:<40} {:<10.2f}".format("Median ITL (ms):", metrics.median_itl_ms))
    print("{:<40} {:<10.2f}".format("P90 ITL (ms):", metrics.p90_itl_ms))
    print("{:<40} {:<10.2f}".format("P99 ITL (ms):", metrics.p99_itl_ms))
    print("=" * 50)

    if (
        metrics.median_ttft_ms is not None
        and metrics.mean_itl_ms is not None
        and metrics.output_throughput is not None
    ):
        result = {
            # Arguments
            "backend": args.backend,
            "dataset_name": args.dataset_name,
            "request_rate": request_rate,
            "max_concurrency": max_concurrency,
            "fixed_output_len": args.fixed_output_len,
            "random_input_len": args.random_input_len,
            "random_output_len": args.random_output_len,
            "random_range_ratio": args.random_range_ratio,
            # Results
            "duration": benchmark_duration,
            "completed": metrics.completed,
            "total_input_tokens": metrics.total_input,
            "total_output_tokens": metrics.total_output,
            "total_output_tokens_retokenized": metrics.total_output_retokenized,
            "request_throughput": metrics.request_throughput,
            "input_throughput": metrics.input_throughput,
            "output_throughput": metrics.output_throughput,
            "mean_e2e_latency_ms": metrics.mean_e2e_latency_ms,
            "median_e2e_latency_ms": metrics.median_e2e_latency_ms,
            "std_e2e_latency_ms": metrics.std_e2e_latency_ms,
            "p99_e2e_latency_ms": metrics.p99_e2e_latency_ms,
            "mean_ttft_ms": metrics.mean_ttft_ms,
            "median_ttft_ms": metrics.median_ttft_ms,
            "std_ttft_ms": metrics.std_ttft_ms,
            "p99_ttft_ms": metrics.p99_ttft_ms,
            "mean_tpot_ms": metrics.mean_tpot_ms,
            "median_tpot_ms": metrics.median_tpot_ms,
            "std_tpot_ms": metrics.std_tpot_ms,
            "p99_tpot_ms": metrics.p99_tpot_ms,
            "mean_itl_ms": metrics.mean_itl_ms,
            "median_itl_ms": metrics.median_itl_ms,
            "std_itl_ms": metrics.std_itl_ms,
            "p99_itl_ms": metrics.p99_itl_ms,
            "concurrency": metrics.concurrency,
            "input_throughput": metrics.input_throughput,
            "output_throughput": metrics.output_throughput,
            "fixed_output_len": args.fixed_output_len,
            "random_input_len": args.random_input_len,
            "random_output_len": args.random_output_len,
            "random_range_ratio": args.random_range_ratio,
            "duration": benchmark_duration,
            "completed": metrics.completed,
        }
    else:
        print(f"Error running benchmark for request rate: {request_rate}")
        print("-" * 30)

    # Determine output file name
    if args.output_file:
        output_file_name = args.output_file
    else:
        now = datetime.now().strftime("%m%d")
        if args.dataset_name == "random":
            output_file_name = f"{args.backend}_{now}_{args.num_prompts}_{args.random_input_len}_{args.random_output_len}.jsonl"
        else:
            output_file_name = (
                f"{args.backend}_{now}_{args.num_prompts}_{args.dataset_name}.jsonl"
            )

    # Append results to a JSONL file
    with open(output_file_name, "a") as file:
        file.write(json.dumps(result) + "\n")

    result = {
        "duration": benchmark_duration,
        "completed": metrics.completed,
        "total_input_tokens": metrics.total_input,
        "total_output_tokens": metrics.total_output,
        "total_output_tokens_retokenized": metrics.total_output_retokenized,
        "request_throughput": metrics.request_throughput,
        "input_throughput": metrics.input_throughput,
        "output_throughput": metrics.output_throughput,
        "mean_ttft_ms": metrics.mean_ttft_ms,
        "median_ttft_ms": metrics.median_ttft_ms,
        "std_ttft_ms": metrics.std_ttft_ms,
        "p90_ttft_ms": metrics.p90_ttft_ms,
        "p99_ttft_ms": metrics.p99_ttft_ms,
        "mean_tpot_ms": metrics.mean_tpot_ms,
        "median_tpot_ms": metrics.median_tpot_ms,
        "std_tpot_ms": metrics.std_tpot_ms,
        "p90_tpot_ms": metrics.p90_tpot_ms,
        "p99_tpot_ms": metrics.p99_tpot_ms,
        "mean_itl_ms": metrics.mean_itl_ms,
        "median_itl_ms": metrics.median_itl_ms,
        "std_itl_ms": metrics.std_itl_ms,
        "p90_itl_ms": metrics.p90_itl_ms,
        "p99_itl_ms": metrics.p99_itl_ms,
        "input_lens": [output.prompt_len for output in outputs],
        "output_lens": output_lens,
        "ttfts": [output.ttft for output in outputs],
        "itls": [output.itl for output in outputs],
        "generated_texts": [output.generated_text for output in outputs],
        "errors": [output.error for output in outputs],
        "mean_e2e_latency_ms": metrics.mean_e2e_latency_ms,
        "median_e2e_latency_ms": metrics.median_e2e_latency_ms,
    }
    return result
```
**EN:** `benchmark` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. It returns `result` to the caller. Notable calls include `sum`, `print`, `asyncio.Queue`.
**CN:** `benchmark` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。它会向调用方返回 `result`。其中较关键的调用包括 `sum`, `print`, `asyncio.Queue`。

### Lines 687-791: Function `run_benchmark` / 函数 `run_benchmark`
```python
def run_benchmark(args_: argparse.Namespace):
    global args
    args = args_

    # Set default value for max_concurrency if not present
    if not hasattr(args, "max_concurrency"):
        args.max_concurrency = None

    # Set global environments
    set_ulimit()
    random.seed(args.seed)
    np.random.seed(args.seed)

    extra_request_body = {}
    if args.extra_request_body:
        extra_request_body = json.loads(args.extra_request_body)

    # Set url
    if args.port is None:
        args.port = {
            "sglang": 30000,
            "lmdeploy": 23333,
            "vllm": 8000,
        }.get(args.backend, 30000)

    model_url = (
        f"{args.base_url}/v1/models"
        if args.base_url
        else f"http://{args.host}:{args.port}/v1/models"
    )

    if args.backend in ["sglang", "vllm", "lmdeploy"]:
        api_url = (
            f"{args.base_url}/v1/chat/completions"
            if args.base_url
            else f"http://{args.host}:{args.port}/v1/chat/completions"
        )
    base_url = (
        f"http://{args.host}:{args.port}" if args.base_url is None else args.base_url
    )

    # Get model name
    if args.model is None:
        if args.backend == "truss":
            print(
                "Please provide a model with `--model` when using truss backend. e.g. --model meta-llama/Llama-3.1-8B-Instruct"
            )
            sys.exit(1)
        try:
            response = requests.get(model_url)
            model_list = response.json().get("data", [])
            args.model = model_list[0]["id"] if model_list else None
        except Exception as e:
            print(f"Failed to fetch model from {model_url}. Error: {e}")
            print(
                "Please specify the correct host and port using `--host` and `--port`."
            )
            sys.exit(1)

    if args.model is None:
        print("No model specified or found. Please provide a model using `--model`.")
        sys.exit(1)

    # Dataset compatibility check
    if args.enable_multiturn:
        # TODO: Support multiturn for random
        if args.dataset_name not in ["sharegpt", "ultrachat", "loogle", "nextqa"]:
            print(
                "Multiturn conversation is only supported for sharegpt, ultrachat, loogle, and nextqa datasets."
            )
            sys.exit(1)

    if args.enable_shared_prefix:
        if args.dataset_name not in ["loogle", "nextqa"]:
            print("Shared prefix is only supported for loogle and nextqa datasets.")
            sys.exit(1)

    print(f"{args}\n")

    # Read dataset
    backend = args.backend
    model_id = args.model
    tokenizer_id = args.tokenizer if args.tokenizer is not None else args.model

    tokenizer = get_tokenizer(tokenizer_id)

    input_requests = get_dataset(args, tokenizer)

    return asyncio.run(
        benchmark(
            backend=backend,
            api_url=api_url,
            base_url=base_url,
            model_id=model_id,
            tokenizer=tokenizer,
            input_requests=input_requests,
            request_rate=args.request_rate,
            max_concurrency=args.max_concurrency,
            disable_tqdm=args.disable_tqdm,
            lora_name=args.lora_name,
            extra_request_body=extra_request_body,
            profile=args.profile,
            enable_shared_prefix=args.enable_shared_prefix,
        )
    )
```
**EN:** `run_benchmark` is a function that builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. It returns `asyncio.run(benchmark(backend=backend, api_url=api_url, base_url=base_url, model_id=model_id, tok...` to the caller. Notable calls include `set_ulimit`, `random.seed`, `np.random.seed`.
**CN:** `run_benchmark` 是一个函数，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `asyncio.run(benchmark(backend=backend, api_url=api_url, base_url=base_url, model_id=model_id, tok...`。其中较关键的调用包括 `set_ulimit`, `random.seed`, `np.random.seed`。

### Lines 794-1031: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = ArgumentParser(description="Benchmark the online serving throughput.")
    parser.add_argument(
        "--backend",
        type=str,
        choices=list(ASYNC_REQUEST_FUNCS.keys()),
        default="sglang",
        help="Must specify a backend, depending on the LLM Inference Engine.",
    )
    parser.add_argument(
        "--base-url",
        type=str,
        default=None,
        help="Server or API base url if not using http host and port.",
    )
    parser.add_argument(
        "--host", type=str, default="0.0.0.0", help="Default host is 0.0.0.0."
    )
    parser.add_argument(
        "--port",
        type=int,
        help="If not set, the default port is configured according to its default value for different LLM Inference Engines.",
    )
    parser.add_argument(
        "--dataset-name",
        type=str,
        default="sharegpt",
        choices=[
            "sharegpt",
            "random",
            "generated-shared-prefix",
            "ultrachat",
            "loogle",
            "nextqa",
        ],
        help="Name of the dataset to benchmark on.",
    )
    parser.add_argument(
        "--dataset-path", type=str, default="", help="Path to the dataset."
    )
    parser.add_argument(
        "--model",
        type=str,
        help="Name or path of the model. If not set, the default model will request /v1/models for conf.",
    )
    parser.add_argument(
        "--tokenizer",
        type=str,
        help="Name or path of the tokenizer. If not set, using the model conf.",
    )
    parser.add_argument(
        "--chat-template",
        type=str,
        help="The buliltin chat template name or the path of the chat template file. This is only used for OpenAI-compatible API server.",
    )
    parser.add_argument(
        "--num-prompts",
        type=int,
        default=1000,
        help="Number of prompts to process. Default is 1000.",
    )
    parser.add_argument(
        "--fixed-output-len",
        type=int,
        default=None,
        help="Output length for each request. Overrides the output length from the dataset.",
    )
    parser.add_argument(
        "--sharegpt-context-len",
        type=int,
        default=None,
        help="The context length of the model for the ShareGPT dataset. Requests longer than the context length will be dropped.",
    )
    parser.add_argument(
        "--random-input-len",
        type=int,
        default=1024,
        help="Number of input tokens per request, used only for random dataset.",
    )
    parser.add_argument(
        "--random-output-len",
        default=1024,
        type=int,
        help="Number of output tokens per request, used only for random dataset.",
    )
    parser.add_argument(
        "--random-range-ratio",
        type=float,
        default=0.0,
        help="Range of sampled ratio of input/output length, "
        "used only for random dataset.",
    )
    parser.add_argument(
        "--request-rate",
        type=float,
        default=float("inf"),
        help="Number of requests per second. If this is inf, then all the requests are sent at time 0. "
        "Otherwise, we use Poisson process to synthesize the request arrival times. Default is inf.",
    )
    parser.add_argument(
        "--max-concurrency",
        type=int,
        default=None,
        help="Maximum number of concurrent requests. This can be used "
        "to help simulate an environment where a higher level component "
        "is enforcing a maximum number of concurrent requests. While the "
        "--request-rate argument controls the rate at which requests are "
        "initiated, this argument will control how many are actually allowed "
        "to execute at a time. This means that when used in combination, the "
        "actual request rate may be lower than specified with --request-rate, "
        "if the server is not processing requests fast enough to keep up.",
    )
    parser.add_argument(
        "--multi",
        action="store_true",
        help="Use request rate range rather than single value.",
    )
    parser.add_argument(
        "--request-rate-range",
        type=str,
        default="2,34,2",
        help="Range of request rates in the format start,stop,step. Default is 2,34,2. It also supports a list of request rates, requiring the parameters to not equal three.",
    )
    parser.add_argument("--output-file", type=str, help="Output JSONL file name.")
    parser.add_argument(
        "--enable-multiturn",
        action="store_true",
        help="Enable multiturn chat for online serving benchmarking. "
        "This option is effective on the following datasets: "
        "sharegpt, ultrachat, loogle, nextqa",
    )
    parser.add_argument(
        "--enable-shared-prefix",
        action="store_true",
        help="Enable shared prefix for online serving benchmarking. "
        "This option is effective on the following datasets: "
        "loogle, nextqa",
    )

    parser.add_argument(
        "--disable-shuffle",
        action="store_true",
        help="Disable shuffling datasets. This is useful to generate stable output "
        "in benchmarking",
    )
    parser.add_argument(
        "--disable-tqdm",
        action="store_true",
        help="Specify to disable tqdm progress bar.",
    )
    parser.add_argument(
        "--disable-stream",
        action="store_true",
        help="Disable streaming mode.",
    )
    parser.add_argument(
        "--return-logprob",
        action="store_true",
        help="Return logprob.",
    )
    parser.add_argument("--seed", type=int, default=1, help="The random seed.")
    parser.add_argument(
        "--disable-ignore-eos",
        action="store_true",
        help="Disable ignoring EOS.",
    )
    parser.add_argument(
        "--extra-request-body",
        metavar='{"key1": "value1", "key2": "value2"}',
        type=str,
        help="Append given JSON object to the request payload. You can use this to specify"
        "additional generate params like sampling params.",
    )
    parser.add_argument(
        "--apply-chat-template",
        action="store_true",
        help="Apply chat template",
    )
    parser.add_argument(
        "--profile",
        action="store_true",
        help="Use Torch Profiler. The endpoint must be launched with "
        "SGLANG_TORCH_PROFILER_DIR to enable profiler.",
    )
    parser.add_argument(
        "--lora-name",
        type=str,
        default=None,
        help="The name of LoRA adapter",
    )

    group = parser.add_argument_group("generated-shared-prefix dataset arguments")
    group.add_argument(
        "--gsp-num-groups",
        type=int,
        default=64,
        help="Number of system prompt groups for generated-shared-prefix dataset",
    )
    group.add_argument(
        "--gsp-prompts-per-group",
        type=int,
        default=16,
        help="Number of prompts per system prompt group for generated-shared-prefix dataset",
    )
    group.add_argument(
        "--gsp-system-prompt-len",
        type=int,
        default=2048,
        help="Target length in tokens for system prompts in generated-shared-prefix dataset",
    )
    group.add_argument(
        "--gsp-question-len",
        type=int,
        default=128,
        help="Target length in tokens for questions in generated-shared-prefix dataset",
    )
    group.add_argument(
        "--gsp-output-len",
        type=int,
        default=256,
        help="Target length in tokens for outputs in generated-shared-prefix dataset",
    )
    # videos specific
    parser.add_argument(
        "--max-frames",
        type=int,
        default=sys.maxsize,
        help="The maximum number of frames to extract from each video. "
        "This option is specific to the nextqa dataset (video benchmark). ",
    )
    args = parser.parse_args()

    if args.enable_multiturn and args.enable_shared_prefix:
        parser.error(
            "--enable-multiturn and --enable-shared-prefix cannot be set at the same time."
        )

    run_benchmark(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `os`, `random`, `sys`, `time`, `traceback`, `warnings`, `dataclasses`, `datetime`, `typing`
- **Third-party / 第三方依赖**: `aiohttp`, `numpy`, `requests`, `data_processing`, `tqdm.asyncio`, `transformers`
- **Internal / 项目内部依赖**: `sglang.benchmark.utils`
