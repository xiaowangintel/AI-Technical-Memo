# bench_warm_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/bench_warm_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache warm cache. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hicache warm cache 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 3-40: Imports and setup / 导入与初始化
```python
"""
Benchmark warm-cache serving with exact shared-prefix control.

This benchmark is designed for cache-focused studies where each request has a
fixed total input length and an exactly controlled shared-prefix ratio. For each
shared-prefix percentage, the benchmark:

1. Flushes the server KV cache.
2. Builds prompts with an identical shared prefix and random unique suffixes.
3. Warms only the shared prefix once.
4. Benchmarks the full prompts through SGLang's native /generate endpoint.

Compared with the existing hicache shared-prefix benchmarks, this benchmark
provides direct control over total length, shared-prefix length, and suffix
length at the token-id level.
"""

import argparse
import asyncio
import json
import random
import time
import warnings
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional, Tuple

import aiohttp
import numpy as np
import requests
from transformers import PreTrainedTokenizerBase

from sglang.benchmark.utils import get_tokenizer, remove_prefix, set_ulimit

AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=20 * 60 * 60)
AIOHTTP_READ_BUFSIZE = 10 * 1024**2


global args
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 44-44: Class `RequestFuncOutput` declaration / 类 `RequestFuncOutput` 声明
```python
class RequestFuncOutput:
```
**EN:** This block introduces class `RequestFuncOutput`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `RequestFuncOutput`，用于通过统一接口组织相关行为。

### Lines 45-53: Class-level state / 类级状态
```python
    generated_text: str = ""
    success: bool = False
    latency: float = 0.0
    ttft: float = 0.0
    itl: List[float] = field(default_factory=list)
    prompt_len: int = 0
    error: str = ""
    output_len: int = 0
    start_time: float = 0.0
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标。

### Lines 57-57: Class `BenchmarkMetrics` declaration / 类 `BenchmarkMetrics` 声明
```python
class BenchmarkMetrics:
```
**EN:** This block introduces class `BenchmarkMetrics`, which measures runtime latency, throughput, or other benchmark metrics and computes evaluation scores and aggregate statistics.
**CN:** 该代码块引入类 `BenchmarkMetrics`，用于测量运行时延迟、吞吐或其他基准指标、计算评测分数与聚合统计结果。

### Lines 58-87: Class-level state / 类级状态
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

### Lines 90-94: Function `_create_bench_client_session` / 函数 `_create_bench_client_session`
```python
def _create_bench_client_session() -> aiohttp.ClientSession:
    return aiohttp.ClientSession(
        timeout=AIOHTTP_TIMEOUT,
        read_bufsize=AIOHTTP_READ_BUFSIZE,
    )
```
**EN:** `_create_bench_client_session` is a function that implements the core logic for this scope. It returns `aiohttp.ClientSession(timeout=AIOHTTP_TIMEOUT, read_bufsize=AIOHTTP_READ_BUFSIZE)` to the caller. Notable calls include `aiohttp.ClientSession`.
**CN:** `_create_bench_client_session` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `aiohttp.ClientSession(timeout=AIOHTTP_TIMEOUT, read_bufsize=AIOHTTP_READ_BUFSIZE)`。其中较关键的调用包括 `aiohttp.ClientSession`。

### Lines 97-175: Async function `async_request_sglang_generate` / 异步函数 `async_request_sglang_generate`
```python
async def async_request_sglang_generate(
    api_url: str,
    input_ids: List[int],
    prompt_len: int,
    output_len: int,
    pbar: Optional[Any] = None,
) -> RequestFuncOutput:
    async with _create_bench_client_session() as session:
        payload = {
            "input_ids": input_ids,
            "sampling_params": {
                "temperature": 0.0,
                "max_new_tokens": output_len,
                "ignore_eos": not args.disable_ignore_eos,
            },
            "stream": True,
            **args.extra_request_body,
        }

        output = RequestFuncOutput(prompt_len=prompt_len)

        generated_text = ""
        ttft = 0.0
        st = time.perf_counter()
        output.start_time = st
        most_recent_timestamp = st
        last_output_len = 0
        latency = 0.0

        try:
            async with session.post(url=api_url, json=payload) as response:
                if response.status == 200:
                    async for chunk_bytes in response.content:
                        chunk_bytes = chunk_bytes.strip()
                        if not chunk_bytes:
                            continue

                        chunk = remove_prefix(chunk_bytes.decode("utf-8"), "data: ")
                        latency = time.perf_counter() - st
                        if chunk == "[DONE]":
                            continue

                        data = json.loads(chunk)

                        if "text" in data and data["text"]:
                            timestamp = time.perf_counter()
                            generated_text = data["text"]
                            current_output_len = data["meta_info"]["completion_tokens"]

                            if ttft == 0.0:
                                ttft = timestamp - st
                                output.ttft = ttft
                            else:
                                num_new_tokens = current_output_len - last_output_len
                                if num_new_tokens == 0:
                                    continue
                                chunk_gap = timestamp - most_recent_timestamp
                                adjust_itl = chunk_gap / num_new_tokens
                                output.itl.extend([adjust_itl] * num_new_tokens)

                            most_recent_timestamp = timestamp
                            last_output_len = current_output_len
                            output.output_len = current_output_len

                    output.generated_text = generated_text
                    output.success = True
                    output.latency = latency
                else:
                    output.error = (
                        (response.reason or "") + ": " + (await response.text())
                    )
                    output.success = False
        except Exception as exc:
            output.success = False
            output.error = str(exc)

    if pbar:
        pbar.update(1)
    return output
```
**EN:** `async_request_sglang_generate` is a async function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. It returns `output` to the caller. Notable calls include `_create_bench_client_session`, `RequestFuncOutput`, `time.perf_counter`.
**CN:** `async_request_sglang_generate` 是一个异步函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、准备张量并调用 GPU 内核。它会向调用方返回 `output`。其中较关键的调用包括 `_create_bench_client_session`, `RequestFuncOutput`, `time.perf_counter`。

### Lines 178-206: Async function `run_batch` / 异步函数 `run_batch`
```python
async def run_batch(
    api_url: str,
    prompts: List[Dict[str, Any]],
    output_len: int,
    max_concurrency: Optional[int],
    pbar: Optional[Any] = None,
) -> List[RequestFuncOutput]:
    semaphore = asyncio.Semaphore(max_concurrency) if max_concurrency else None

    async def limited_request(prompt: Dict[str, Any]) -> RequestFuncOutput:
        if semaphore is None:
            return await async_request_sglang_generate(
                api_url=api_url,
                input_ids=prompt["input_ids"],
                prompt_len=prompt["prompt_len"],
                output_len=output_len,
                pbar=pbar,
            )
        async with semaphore:
            return await async_request_sglang_generate(
                api_url=api_url,
                input_ids=prompt["input_ids"],
                prompt_len=prompt["prompt_len"],
                output_len=output_len,
                pbar=pbar,
            )

    tasks = [asyncio.create_task(limited_request(prompt)) for prompt in prompts]
    return await asyncio.gather(*tasks)
```
**EN:** `run_batch` is a async function that coordinates asynchronous or parallel execution. It returns `await asyncio.gather(*tasks)` to the caller. Notable calls include `asyncio.Semaphore`, `asyncio.create_task`, `asyncio.gather`.
**CN:** `run_batch` 是一个异步函数，用于协调异步或并行执行。它会向调用方返回 `await asyncio.gather(*tasks)`。其中较关键的调用包括 `asyncio.Semaphore`, `asyncio.create_task`, `asyncio.gather`。

### Lines 209-211: Function `flush_cache` / 函数 `flush_cache`
```python
def flush_cache(base_url: str) -> None:
    response = requests.post(f"{base_url}/flush_cache", timeout=30)
    response.raise_for_status()
```
**EN:** `flush_cache` is a function that sends requests to serving or OpenAI-compatible APIs. Notable calls include `requests.post`, `response.raise_for_status`.
**CN:** `flush_cache` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求。其中较关键的调用包括 `requests.post`, `response.raise_for_status`。

### Lines 214-221: Function `gen_token_ids` / 函数 `gen_token_ids`
```python
def gen_token_ids(
    vocab_ids: List[int],
    token_num: int,
    rng: random.Random,
) -> List[int]:
    if token_num <= 0:
        return []
    return rng.choices(vocab_ids, k=token_num)
```
**EN:** `gen_token_ids` is a function that processes tokenized prompts or decoded outputs. It returns `rng.choices(vocab_ids, k=token_num)` to the caller. Notable calls include `rng.choices`.
**CN:** `gen_token_ids` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `rng.choices(vocab_ids, k=token_num)`。其中较关键的调用包括 `rng.choices`。

### Lines 224-242: Function `build_prompts` / 函数 `build_prompts`
```python
def build_prompts(
    vocab_ids: List[int],
    total_tokens: int,
    shared_pct: int,
    num_prompts: int,
    rng: random.Random,
) -> List[Dict[str, Any]]:
    prefix_len = total_tokens * shared_pct // 100
    suffix_len = total_tokens - prefix_len

    shared_prefix = gen_token_ids(vocab_ids, prefix_len, rng)

    prompts: List[Dict[str, Any]] = []
    for _ in range(num_prompts):
        suffix = gen_token_ids(vocab_ids, suffix_len, rng)
        input_ids = shared_prefix + suffix
        prompts.append({"input_ids": input_ids, "prompt_len": len(input_ids)})

    return prompts
```
**EN:** `build_prompts` is a function that processes tokenized prompts or decoded outputs. It returns `prompts` to the caller. Notable calls include `gen_token_ids`, `range`, `prompts.append`.
**CN:** `build_prompts` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `prompts`。其中较关键的调用包括 `gen_token_ids`, `range`, `prompts.append`。

### Lines 245-259: Async function `warm_shared_prefix` / 异步函数 `warm_shared_prefix`
```python
async def warm_shared_prefix(api_url: str, shared_prefix_ids: List[int]) -> None:
    if not shared_prefix_ids:
        return

    warmup = await async_request_sglang_generate(
        api_url=api_url,
        input_ids=shared_prefix_ids,
        prompt_len=len(shared_prefix_ids),
        output_len=1,
    )
    if not warmup.success:
        raise RuntimeError(
            "Warmup failed - Please make sure benchmark arguments are correctly "
            f"specified. Error: {warmup.error}"
        )
```
**EN:** `warm_shared_prefix` is a async function that measures runtime latency, throughput, or other benchmark metrics and coordinates asynchronous or parallel execution. Notable calls include `async_request_sglang_generate`, `RuntimeError`, `len`.
**CN:** `warm_shared_prefix` 是一个异步函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `async_request_sglang_generate`, `RuntimeError`, `len`。

### Lines 262-335: Function `calculate_metrics` / 函数 `calculate_metrics`
```python
def calculate_metrics(
    outputs: List[RequestFuncOutput],
    dur_s: float,
    tokenizer: PreTrainedTokenizerBase,
) -> Tuple[BenchmarkMetrics, List[int]]:
    output_lens: List[int] = []
    retokenized_output_lens: List[int] = []
    total_input = 0
    completed = 0
    itls: List[float] = []
    tpots: List[float] = []
    ttfts: List[float] = []
    e2e_latencies: List[float] = []

    for output in outputs:
        if output.success:
            output_len = output.output_len
            output_lens.append(output_len)
            retokenized_output_len = len(
                tokenizer.encode(output.generated_text, add_special_tokens=False)
            )
            retokenized_output_lens.append(retokenized_output_len)
            total_input += output.prompt_len
            if output_len > 1:
                tpots.append((output.latency - output.ttft) / (output_len - 1))
            itls += output.itl
            ttfts.append(output.ttft)
            e2e_latencies.append(output.latency)
            completed += 1
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
        mean_ttft_ms=np.mean(ttfts or 0) * 1000,
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
**EN:** `calculate_metrics` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. It returns `(metrics, output_lens)` to the caller. Notable calls include `BenchmarkMetrics`, `warnings.warn`, `output_lens.append`.
**CN:** `calculate_metrics` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。它会向调用方返回 `(metrics, output_lens)`。其中较关键的调用包括 `BenchmarkMetrics`, `warnings.warn`, `output_lens.append`。

### Lines 338-410: Function `print_benchmark_result` / 函数 `print_benchmark_result`
```python
def print_benchmark_result(
    metrics: BenchmarkMetrics,
    benchmark_duration: float,
    backend: str,
    request_rate: float,
    max_concurrency: Optional[int],
) -> None:
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
```
**EN:** `print_benchmark_result` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. Notable calls include `print`, `'\n{s:{c}^{n}}'.format`, `'{:<40} {:<10}'.format`.
**CN:** `print_benchmark_result` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `print`, `'\n{s:{c}^{n}}'.format`, `'{:<40} {:<10}'.format`。

### Lines 413-469: Function `maybe_write_summary_jsonl` / 函数 `maybe_write_summary_jsonl`
```python
def maybe_write_summary_jsonl(
    pct: int,
    prefix_len: int,
    suffix_len: int,
    metrics: BenchmarkMetrics,
    output_file: Optional[str],
    benchmark_duration: float,
) -> None:
    if not output_file:
        return

    result = {
        "backend": args.backend,
        "dataset_name": "warm-cache",
        "request_rate": float("inf"),
        "max_concurrency": args.max_concurrency,
        "shared_prefix_pct": pct,
        "prefix_len": prefix_len,
        "suffix_len": suffix_len,
        "total_tokens": args.total_tokens,
        "num_prompts": args.num_prompts,
        "output_len": args.output_len,
        "completed": metrics.completed,
        "benchmark_duration": benchmark_duration,
        "total_input": metrics.total_input,
        "total_output": metrics.total_output,
        "total_output_retokenized": metrics.total_output_retokenized,
        "request_throughput": metrics.request_throughput,
        "input_throughput": metrics.input_throughput,
        "output_throughput": metrics.output_throughput,
        "output_throughput_retokenized": metrics.output_throughput_retokenized,
        "total_throughput": metrics.total_throughput,
        "total_throughput_retokenized": metrics.total_throughput_retokenized,
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
        "mean_e2e_latency_ms": metrics.mean_e2e_latency_ms,
        "median_e2e_latency_ms": metrics.median_e2e_latency_ms,
        "std_e2e_latency_ms": metrics.std_e2e_latency_ms,
        "p99_e2e_latency_ms": metrics.p99_e2e_latency_ms,
        "concurrency": metrics.concurrency,
    }

    with open(output_file, "a", encoding="utf-8") as fout:
        fout.write(json.dumps(result) + "\n")
```
**EN:** `maybe_write_summary_jsonl` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs. Notable calls include `float`, `open`, `fout.write`.
**CN:** `maybe_write_summary_jsonl` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `float`, `open`, `fout.write`。

### Lines 472-552: Async function `benchmark_shared_prefix_pct` / 异步函数 `benchmark_shared_prefix_pct`
```python
async def benchmark_shared_prefix_pct(
    api_url: str,
    base_url: str,
    tokenizer: PreTrainedTokenizerBase,
    vocab_ids: List[int],
    rng: random.Random,
    pct: int,
) -> Tuple[BenchmarkMetrics, float, int, int, int]:
    prefix_len = args.total_tokens * pct // 100
    suffix_len = args.total_tokens - prefix_len

    print(f"\n{'=' * 70}")
    print(
        f"shared_prefix={pct}%  prefix_len={prefix_len}  "
        f"suffix_len={suffix_len}  total={prefix_len + suffix_len}"
    )
    print(f"{'=' * 70}")

    print("Flushing KV cache ...")
    flush_cache(base_url)
    time.sleep(1)

    print(f"Building {args.num_prompts} prompts ...")
    prompts = build_prompts(
        vocab_ids=vocab_ids,
        total_tokens=args.total_tokens,
        shared_pct=pct,
        num_prompts=args.num_prompts,
        rng=rng,
    )

    if prefix_len > 0:
        print(f"Warming shared prefix only ({prefix_len} tokens) ...")
        await warm_shared_prefix(
            api_url=api_url, shared_prefix_ids=prompts[0]["input_ids"][:prefix_len]
        )

    print(f"Sending requests (max_concurrency={args.max_concurrency}) ...")
    benchmark_start_time = time.perf_counter()
    outputs = await run_batch(
        api_url=api_url,
        prompts=prompts,
        output_len=args.output_len,
        max_concurrency=args.max_concurrency,
        pbar=None,
    )
    benchmark_duration = time.perf_counter() - benchmark_start_time

    failed_outputs = [output for output in outputs if not output.success]
    if failed_outputs:
        print(f"WARNING: {len(failed_outputs)}/{len(outputs)} requests failed")
        for output in failed_outputs[:5]:
            print(f"  {output.error[:160]}")

    metrics, _ = calculate_metrics(
        outputs=outputs,
        dur_s=benchmark_duration,
        tokenizer=tokenizer,
    )

    if metrics.completed == 0:
        raise RuntimeError("All requests failed for this shared-prefix percentage.")

    print_benchmark_result(
        metrics=metrics,
        benchmark_duration=benchmark_duration,
        backend=args.backend,
        request_rate=float("inf"),
        max_concurrency=args.max_concurrency,
    )

    maybe_write_summary_jsonl(
        pct=pct,
        prefix_len=prefix_len,
        suffix_len=suffix_len,
        metrics=metrics,
        output_file=args.output_file,
        benchmark_duration=benchmark_duration,
    )

    return metrics, benchmark_duration, prefix_len, suffix_len, len(outputs)
```
**EN:** `benchmark_shared_prefix_pct` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. It returns `(metrics, benchmark_duration, prefix_len, suffix_len, len(outputs))` to the caller. Notable calls include `print`, `flush_cache`, `time.sleep`.
**CN:** `benchmark_shared_prefix_pct` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。它会向调用方返回 `(metrics, benchmark_duration, prefix_len, suffix_len, len(outputs))`。其中较关键的调用包括 `print`, `flush_cache`, `time.sleep`。

### Lines 555-668: Async function `main` / 异步函数 `main`
```python
async def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument(
        "--backend",
        type=str,
        default="sglang",
        choices=["sglang"],
        help="Warm-cache benchmark currently supports the native SGLang /generate endpoint.",
    )
    parser.add_argument(
        "--base-url",
        type=str,
        default=None,
        help="Server base url if not using host and port.",
    )
    parser.add_argument("--host", type=str, default="127.0.0.1")
    parser.add_argument("--port", type=int, default=30000)
    parser.add_argument(
        "--model",
        type=str,
        required=True,
        help="Name or path of the model. Used to load the tokenizer and vocab ids.",
    )
    parser.add_argument(
        "--tokenizer",
        type=str,
        default=None,
        help="Name or path of the tokenizer. Defaults to --model.",
    )
    parser.add_argument(
        "--num-prompts",
        type=int,
        default=64,
        help="Number of prompts to process per shared-prefix percentage.",
    )
    parser.add_argument(
        "--total-tokens",
        type=int,
        default=70000,
        help="Total input tokens per request (shared prefix + unique suffix).",
    )
    parser.add_argument(
        "--output-len",
        type=int,
        default=200,
        help="Output length for each request.",
    )
    parser.add_argument(
        "--max-concurrency",
        type=int,
        default=4,
        help="Maximum number of concurrent requests.",
    )
    parser.add_argument(
        "--pcts",
        type=str,
        default="0,10,20,30,40,50,60,70,80,90,92,95,97,99",
        help="Comma-separated shared-prefix percentages to sweep.",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=42,
        help="Random seed for synthetic prompt generation.",
    )
    parser.add_argument(
        "--disable-ignore-eos",
        action="store_true",
        help="Disable ignoring EOS.",
    )
    parser.add_argument(
        "--output-file",
        type=str,
        default=None,
        help="Optional JSONL file to append one result object per shared-prefix percentage.",
    )
    parser.add_argument(
        "--extra-request-body",
        metavar='{"key1": "value1", "key2": "value2"}',
        type=str,
        help="Append given JSON object to the request payload. You can use this to specify additional generate params.",
    )
    global args
    args = parser.parse_args()

    args.extra_request_body = (
        json.loads(args.extra_request_body) if args.extra_request_body else {}
    )

    base_url = args.base_url or f"http://{args.host}:{args.port}"
    api_url = f"{base_url}/generate"
    pcts = [int(p.strip()) for p in args.pcts.split(",") if p.strip()]
    rng = random.Random(args.seed)

    tokenizer_id = args.tokenizer if args.tokenizer is not None else args.model
    tokenizer = get_tokenizer(tokenizer_id)
    vocab_ids = list(tokenizer.get_vocab().values())

    print(f"{args}\n")
    print(f"Loading tokenizer from {tokenizer_id} ...")
    print(f"Tokenizer loaded (vocab_size={len(vocab_ids)})")

    for pct in pcts:
        await benchmark_shared_prefix_pct(
            api_url=api_url,
            base_url=base_url,
            tokenizer=tokenizer,
            vocab_ids=vocab_ids,
            rng=rng,
            pct=pct,
        )

    if args.output_file:
        print(f"JSONL results saved to {args.output_file}")
```
**EN:** `main` is a async function that builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `main` 是一个异步函数，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 671-673: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    set_ulimit()
    asyncio.run(main())
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it coordinates asynchronous or parallel execution.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会协调异步或并行执行。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `random`, `time`, `warnings`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `aiohttp`, `numpy`, `requests`, `transformers`
- **Internal / 项目内部依赖**: `sglang.benchmark.utils`
