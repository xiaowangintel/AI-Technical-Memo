# benchmark_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/benchmark_batch/benchmark_batch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on batch. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 batch 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and setup / 导入与初始化
```python
import concurrent.futures
import os
import random
import time
from concurrent.futures import ProcessPoolExecutor
from statistics import mean

import requests
from tqdm import tqdm
from transformers import AutoTokenizer

from sglang.lang.backend.runtime_endpoint import RuntimeEndpoint

###############################################################################
# CONFIG
###############################################################################
ENDPOINT_URL = "http://127.0.0.1:30000"
TOKENIZER_DIR = "/models/meta-llama/Llama-3.2-3B"

# Benchmark configurations
NUM_REQUESTS = 10  # Total number of requests (each with BATCH_SIZE prompts)
NUM_TOKENS = 32000  # Tokens per prompt
BATCH_SIZE = 8  # Number of prompts per request
GEN_TOKENS = 0  # Tokens to generate per prompt
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 30-40: Function `generate_random_prompt` / 函数 `generate_random_prompt`
```python
def generate_random_prompt(index, tokenizer_dir, num_tokens):
    """Generate a single random prompt with specified token count."""
    tokenizer = AutoTokenizer.from_pretrained(tokenizer_dir)
    vocab_size = tokenizer.vocab_size

    def generate_random_text(num_toks):
        random_token_ids = [random.randint(0, vocab_size - 1) for _ in range(num_toks)]
        return tokenizer.decode(random_token_ids, clean_up_tokenization_spaces=True)

    random_text = generate_random_text(num_tokens)
    return f"Prompt {index}: {random_text}"
```
**EN:** `generate_random_prompt` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. The docstring frames it as: Generate a single random prompt with specified token count. It returns `f'Prompt {index}: {random_text}'` to the caller. Notable calls include `AutoTokenizer.from_pretrained`, `generate_random_text`, `tokenizer.decode`.
**CN:** `generate_random_prompt` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `f'Prompt {index}: {random_text}'`。其中较关键的调用包括 `AutoTokenizer.from_pretrained`, `generate_random_text`, `tokenizer.decode`。

### Lines 43-69: Function `prepare_all_prompts` / 函数 `prepare_all_prompts`
```python
def prepare_all_prompts(num_requests, batch_size, num_tokens, tokenizer_dir):
    """Generate prompts for all requests in parallel."""
    total_prompts = num_requests * batch_size
    all_prompts = [None] * total_prompts
    max_workers = min(os.cpu_count() or 1, total_prompts)

    with ProcessPoolExecutor(max_workers=max_workers) as executor:
        futures = [
            executor.submit(generate_random_prompt, i, tokenizer_dir, num_tokens)
            for i in range(total_prompts)
        ]
        for future in tqdm(
            concurrent.futures.as_completed(futures),
            total=total_prompts,
            desc="Generating prompts",
        ):
            index = futures.index(future)
            all_prompts[index] = future.result()

    batched_prompts = [
        all_prompts[i * batch_size : (i + 1) * batch_size] for i in range(num_requests)
    ]

    print(
        f"Generated {total_prompts} prompts with {num_tokens} tokens each, grouped into {num_requests} requests of {batch_size} prompts.\n"
    )
    return batched_prompts
```
**EN:** `prepare_all_prompts` is a function that sends requests to serving or OpenAI-compatible APIs, coordinates asynchronous or parallel execution, and processes tokenized prompts or decoded outputs. The docstring frames it as: Generate prompts for all requests in parallel. It returns `batched_prompts` to the caller. Notable calls include `min`, `print`, `ProcessPoolExecutor`.
**CN:** `prepare_all_prompts` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `batched_prompts`。其中较关键的调用包括 `min`, `print`, `ProcessPoolExecutor`。

### Lines 75-98: Function `send_batch_request` / 函数 `send_batch_request`
```python
def send_batch_request(endpoint, prompts, gen_tokens, request_id):
    """Send a batch of prompts to the /generate endpoint synchronously."""
    sampling_params = {
        "max_new_tokens": gen_tokens,
        "temperature": 0.7,
        "stop": "\n",
    }
    data = {"text": prompts, "sampling_params": sampling_params}

    start_time = time.perf_counter()
    try:
        response = requests.post(
            endpoint.base_url + "/generate", json=data, timeout=3600
        )
        if response.status_code != 200:
            error = response.json()
            raise RuntimeError(f"Request {request_id} failed: {error}")
        result = response.json()
        elapsed_time = (time.perf_counter() - start_time) * 1000  # Convert to ms
        avg_per_prompt = elapsed_time / len(prompts) if prompts else 0
        return request_id, elapsed_time, avg_per_prompt, True, len(prompts)
    except Exception as e:
        print(f"[Request] Error for request {request_id}: {e}")
        return request_id, 0, 0, False, len(prompts)
```
**EN:** `send_batch_request` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. The docstring frames it as: Send a batch of prompts to the /generate endpoint synchronously. It returns `(request_id, elapsed_time, avg_per_prompt, True, len(prompts))` to the caller. Notable calls include `time.perf_counter`, `requests.post`, `response.json`.
**CN:** `send_batch_request` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(request_id, elapsed_time, avg_per_prompt, True, len(prompts))`。其中较关键的调用包括 `time.perf_counter`, `requests.post`, `response.json`。

### Lines 101-124: Function `run_benchmark` / 函数 `run_benchmark`
```python
def run_benchmark(endpoint, batched_prompts, batch_size, gen_tokens):
    """Run the benchmark sequentially."""
    results = []
    num_requests = len(batched_prompts)

    # Record start time for total latency
    benchmark_start_time = time.perf_counter()

    for i, batch_prompts in enumerate(batched_prompts):
        request_id = i + 1
        assert (
            len(batch_prompts) == batch_size
        ), f"Request {request_id} should have {batch_size} prompts, got {len(batch_prompts)}"

        print(
            f"[Request] Sending request {request_id}/{num_requests} with {len(batch_prompts)} prompts at {int(time.time()*1000)}"
        )
        result = send_batch_request(endpoint, batch_prompts, gen_tokens, request_id)
        results.append(result)

    # Calculate total latency
    total_latency = (time.perf_counter() - benchmark_start_time) * 1000  # Convert to ms

    return results, total_latency
```
**EN:** `run_benchmark` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. The docstring frames it as: Run the benchmark sequentially. It returns `(results, total_latency)` to the caller. Notable calls include `len`, `time.perf_counter`, `enumerate`.
**CN:** `run_benchmark` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(results, total_latency)`。其中较关键的调用包括 `len`, `time.perf_counter`, `enumerate`。

### Lines 130-161: Function `process_results` / 函数 `process_results`
```python
def process_results(results, total_latency, num_requests):
    """Process and display benchmark results."""
    total_time = 0
    successful_requests = 0
    failed_requests = 0
    request_latencies = []
    per_prompt_latencies = []
    total_prompts = 0

    for request_id, elapsed_time, avg_per_prompt, success, batch_size in results:
        if success:
            successful_requests += 1
            total_prompts += batch_size
            request_latencies.append(elapsed_time)
            per_prompt_latencies.append(avg_per_prompt)
            total_time += elapsed_time / 1000  # Convert to seconds
        else:
            failed_requests += 1

    avg_request_latency = mean(request_latencies) if request_latencies else 0
    avg_per_prompt_latency = mean(per_prompt_latencies) if per_prompt_latencies else 0
    throughput = total_prompts / total_time if total_time > 0 else 0

    print("\nBenchmark Summary:")
    print(f"  Total requests sent:         {len(results)}")
    print(f"  Total prompts sent:          {total_prompts}")
    print(f"  Successful requests:         {successful_requests}")
    print(f"  Failed requests:             {failed_requests}")
    print(f"  Total latency (all requests): {total_latency:.2f} ms")
    print(f"  Avg per request latency:     {avg_request_latency:.2f} ms")
    print(f"  Avg per prompt latency:      {avg_per_prompt_latency:.2f} ms")
    print(f"  Throughput:                  {throughput:.2f} prompts/second\n")
```
**EN:** `process_results` is a function that sends requests to serving or OpenAI-compatible APIs and measures runtime latency, throughput, or other benchmark metrics. The docstring frames it as: Process and display benchmark results. Notable calls include `print`, `mean`, `request_latencies.append`.
**CN:** `process_results` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `print`, `mean`, `request_latencies.append`。

### Lines 167-188: Function `main` / 函数 `main`
```python
def main():
    # Initialize endpoint
    endpoint = RuntimeEndpoint(ENDPOINT_URL)

    # Generate prompts
    batched_prompts = prepare_all_prompts(
        NUM_REQUESTS, BATCH_SIZE, NUM_TOKENS, TOKENIZER_DIR
    )

    # Flush cache before benchmark
    # endpoint.flush_cache()

    # Run benchmark
    print(
        f"Starting benchmark: NUM_TOKENS={NUM_TOKENS}, BATCH_SIZE={BATCH_SIZE}, NUM_REQUESTS={NUM_REQUESTS}\n"
    )
    results, total_latency = run_benchmark(
        endpoint, batched_prompts, BATCH_SIZE, GEN_TOKENS
    )

    # Process and display results
    process_results(results, total_latency, NUM_REQUESTS)
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. Notable calls include `RuntimeEndpoint`, `prepare_all_prompts`, `print`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `RuntimeEndpoint`, `prepare_all_prompts`, `print`。

### Lines 191-193: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    random.seed(0)
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `concurrent.futures`, `os`, `random`, `time`, `statistics`
- **Third-party / 第三方依赖**: `requests`, `tqdm`, `transformers`
- **Internal / 项目内部依赖**: `sglang.lang.backend.runtime_endpoint`
