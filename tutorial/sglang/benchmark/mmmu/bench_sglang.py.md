# bench_sglang.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmmu/bench_sglang.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmmu sglang. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 mmmu sglang 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Imports and setup / 导入与初始化
```python
"""
Bench the sglang-hosted vLM with benchmark MMMU

Usage:
    Host the VLM: python -m sglang.launch_server --model-path Qwen/Qwen2-VL-7B-Instruct --port 30000

    Benchmark: python benchmark/mmmu/bench_sglang.py --port 30000 --concurrency 16

The eval output will be logged
"""

import argparse
import asyncio
import base64
import mimetypes
import re
import sys
import time
import traceback
from dataclasses import dataclass, field
from pathlib import Path
from typing import Any, List, Optional, Tuple

import aiohttp
import openai
from data_utils import save_json
from eval_utils import (
    EvalArgs,
    eval_result,
    get_sampling_params,
    prepare_samples,
    process_result,
)
from tqdm import tqdm

from sglang.test.test_utils import add_common_sglang_args_and_parse

AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=20 * 60 * 60)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 42-42: Class `RequestFuncOutput` declaration / 类 `RequestFuncOutput` 声明
```python
class RequestFuncOutput:
```
**EN:** This block introduces class `RequestFuncOutput`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `RequestFuncOutput`，用于通过统一接口组织相关行为。

### Lines 43-51: Class-level state / 类级状态
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

### Lines 54-69: Async function `async_request_profile` / 异步函数 `async_request_profile`
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

### Lines 72-76: Function `_get_prefix_suffix` / 函数 `_get_prefix_suffix`
```python
def _get_prefix_suffix(prompt: str) -> Tuple[str, str]:
    """Split the prompt into prefix and suffix."""
    prefix = prompt.split("<")[0]
    suffix = prompt.split(">", 1)[1]
    return prefix, suffix
```
**EN:** `_get_prefix_suffix` is a function that implements the core logic for this scope. The docstring frames it as: Split the prompt into prefix and suffix. It returns `(prefix, suffix)` to the caller. Notable calls include `prompt.split`.
**CN:** `_get_prefix_suffix` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(prefix, suffix)`。其中较关键的调用包括 `prompt.split`。

### Lines 79-124: Async function `process_sample` / 异步函数 `process_sample`
```python
async def process_sample(
    client: Any,
    sample: dict,
    sampling_params: dict,
    model: str,
    reasoning_effort: Optional[str] = None,
    lora_path: Optional[str] = None,
) -> Tuple[dict, str]:
    """Send a single sample to the LLM and return (sample, response)."""
    prompt = sample["final_input_prompt"]
    prefix, suffix = _get_prefix_suffix(prompt)
    image = sample["image"]
    assert image is not None
    image_path = sample["image_path"]
    if image_path and not image_path.startswith(("http://", "https://", "data:")):
        p = Path(image_path)
        mime = mimetypes.guess_type(str(p))[0] or "image/png"
        with open(p, "rb") as f:
            b64 = base64.b64encode(f.read()).decode()
        image_url = f"data:{mime};base64,{b64}"
    else:
        image_url = image_path
    extra_body = {"lora_path": lora_path} if lora_path else None
    payload = {
        "model": model,
        "messages": [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prefix},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": suffix},
                ],
            }
        ],
        "extra_body": extra_body,
        **sampling_params,
    }
    if reasoning_effort:
        payload["reasoning_effort"] = reasoning_effort
    response = await client.chat.completions.create(**payload)
    msg = response.choices[0].message
    content = msg.content
    if content is None:
        content = getattr(msg, "reasoning_content", None)
    return sample, content
```
**EN:** `process_sample` is a async function that sends requests to serving or OpenAI-compatible APIs, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. The docstring frames it as: Send a single sample to the LLM and return (sample, response). It returns `(sample, content)` to the caller. Notable calls include `_get_prefix_suffix`, `Path`, `client.chat.completions.create`.
**CN:** `process_sample` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行、准备张量并调用 GPU 内核。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(sample, content)`。其中较关键的调用包括 `_get_prefix_suffix`, `Path`, `client.chat.completions.create`。

### Lines 127-140: Async function `process_sample_with_semaphore` / 异步函数 `process_sample_with_semaphore`
```python
async def process_sample_with_semaphore(
    semaphore: asyncio.Semaphore,
    client: Any,
    sample: dict,
    sampling_params: dict,
    model: str,
    reasoning_effort: Optional[str] = None,
    lora_path: Optional[str] = None,
) -> Tuple[dict, str]:
    """Wrap process_sample with a semaphore for concurrency control."""
    async with semaphore:
        return await process_sample(
            client, sample, sampling_params, model, reasoning_effort, lora_path
        )
```
**EN:** `process_sample_with_semaphore` is a async function that coordinates asynchronous or parallel execution. The docstring frames it as: Wrap process_sample with a semaphore for concurrency control. It returns `await process_sample(client, sample, sampling_params, model, reasoning_effort, lora_path)` to the caller. Notable calls include `process_sample`.
**CN:** `process_sample_with_semaphore` 是一个异步函数，用于协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `await process_sample(client, sample, sampling_params, model, reasoning_effort, lora_path)`。其中较关键的调用包括 `process_sample`。

### Lines 143-233: Async function `eval_mmmu` / 异步函数 `eval_mmmu`
```python
async def eval_mmmu(args) -> None:
    """Main evaluation loop with concurrency control."""
    eval_args = EvalArgs.from_cli_args(args)
    sampling_params = get_sampling_params(eval_args)
    samples = prepare_samples(eval_args)
    model = args.model
    reasoning_effort = eval_args.reasoning_effort
    lora_path = eval_args.lora_path
    answer_dict = {}
    out_samples = {}
    client = openai.AsyncOpenAI(
        api_key="sk",
        base_url=f"http://127.0.0.1:{args.port}/v1",
        timeout=20 * 60 * 60,
    )
    start = time.perf_counter()
    base_url = f"http://127.0.0.1:{args.port}"

    if args.profile:
        print("Starting profiler...")
        profile_output = await async_request_profile(
            api_url=f"{base_url}/start_profile"
        )
        if profile_output.success:
            print("Profiler started")

        samples = samples[: args.profile_number]

    if args.concurrency == 1:
        # For concurrency == 1, run in sequential mode to ensure consistent order
        # this is mainly for profiling
        for sample in tqdm(samples):
            _, response = await process_sample(
                client, sample, sampling_params, model, reasoning_effort, lora_path
            )
            sample["original_response"] = response
            answer = (
                re.search(args.response_answer_regex, response)
                if response is not None
                else None
            )
            process_result(
                answer.group(1).strip() if answer else response,
                sample,
                answer_dict,
                out_samples,
            )
    else:
        semaphore = asyncio.Semaphore(args.concurrency)
        tasks = [
            process_sample_with_semaphore(
                semaphore,
                client,
                sample,
                sampling_params,
                model,
                reasoning_effort,
                lora_path,
            )
            for sample in samples
        ]

        for coro in tqdm(asyncio.as_completed(tasks), total=len(tasks)):
            sample, response = await coro
            sample["original_response"] = response
            answer = (
                re.search(args.response_answer_regex, response)
                if response is not None
                else None
            )
            process_result(
                answer.group(1).strip() if answer else response,
                sample,
                answer_dict,
                out_samples,
            )

    if args.profile:
        print("Stopping profiler...")
        profile_output = await async_request_profile(api_url=f"{base_url}/stop_profile")
        if profile_output.success:
            print("Profiler stopped")

    print(f"Benchmark time: {time.perf_counter() - start}")
    args.output_path = "./answer_sglang.json"
    save_json(args.output_path, out_samples)
    eval_result(
        model_answer_path=args.output_path,
        answer_dict=answer_dict,
        eval_output_path="./val_sglang.json",
    )
```
**EN:** `eval_mmmu` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Main evaluation loop with concurrency control. Notable calls include `EvalArgs.from_cli_args`, `get_sampling_params`, `prepare_samples`.
**CN:** `eval_mmmu` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `EvalArgs.from_cli_args`, `get_sampling_params`, `prepare_samples`。

### Lines 236-246: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--model",
        type=str,
        default="default",
        help="Model name to use in API requests.",
    )
    EvalArgs.add_cli_args(parser)
    args = add_common_sglang_args_and_parse(parser)
    return args
```
**EN:** `parse_args` is a function that builds command-line arguments and runtime configuration and sends requests to serving or OpenAI-compatible APIs. It returns `args` to the caller. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `EvalArgs.add_cli_args`.
**CN:** `parse_args` 是一个函数，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求。它会向调用方返回 `args`。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `EvalArgs.add_cli_args`。

### Lines 249-251: Function `main` / 函数 `main`
```python
def main():
    args = parse_args()
    asyncio.run(eval_mmmu(args))
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration and coordinates asynchronous or parallel execution. Notable calls include `parse_args`, `asyncio.run`, `eval_mmmu`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、协调异步或并行执行。其中较关键的调用包括 `parse_args`, `asyncio.run`, `eval_mmmu`。

### Lines 254-255: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
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
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `base64`, `mimetypes`, `re`, `sys`, `time`, `traceback`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方依赖**: `aiohttp`, `openai`, `data_utils`, `eval_utils`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`
