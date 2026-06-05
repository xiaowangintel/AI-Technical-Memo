# serve.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/serve.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Benchmark online serving throughput. / 该文件的核心目的为：Benchmark online serving throughput.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-19)
```python
r"""Benchmark online serving throughput.

On the server side, run one of the following commands
to launch the vLLM OpenAI API server:
    vllm serve <your_model> <engine arguments>

On the client side, run:
    vllm bench serve \
        --backend <backend or endpoint type. Default 'openai'> \
        --label <benchmark result label. Default using backend> \
        --model <your_model. Optional, defaults to first model from server> \
        --dataset-name <dataset_name. Default 'random'> \
        --input-len <general input length. Optional, maps to dataset-specific args> \
        --output-len <general output length. Optional, maps to dataset-specific args> \
        --request-rate <request_rate. Default inf> \
        --num-prompts <num_prompts. Default 1000>
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 21-56)
```python
import argparse

import asyncio

import contextlib

import importlib.util

import json

import os

import random

import shutil

import ssl

import time

import uuid

import warnings

from collections.abc import AsyncGenerator, Iterable

from dataclasses import dataclass

from datetime import datetime

from enum import Enum

from pathlib import Path

from typing import Any, Literal
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 58-62)
```python
MILLISECONDS_TO_SECONDS_CONVERSION = 1000

TERM_PLOTLIB_AVAILABLE = (importlib.util.find_spec("termplotlib") is not None) and (
    shutil.which("gnuplot") is not None
)
```
**EN:** This constant/configuration block defines `MILLISECONDS_TO_SECONDS_CONVERSION`, `TERM_PLOTLIB_AVAILABLE`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `MILLISECONDS_TO_SECONDS_CONVERSION`, `TERM_PLOTLIB_AVAILABLE`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Function `get_first_model_from_server` (lines 65-92)
```python
async def get_first_model_from_server(
    base_url: str,
    headers: dict | None = None,
    ssl_context: ssl.SSLContext | bool | None = None,
) -> tuple[str, str]:
    """Fetch the first model from the server's /v1/models endpoint."""
    models_url = f"{base_url}/v1/models"
    connector = aiohttp.TCPConnector(ssl=ssl_context)
    async with aiohttp.ClientSession(connector=connector) as session:
        try:
            async with session.get(models_url, headers=headers) as response:
                response.raise_for_status()
                data = await response.json()
                if "data" in data and len(data["data"]) > 0:
                    return data["data"][0]["id"], data["data"][0]["root"]
                else:
                    raise ValueError(
                        f"No models found on the server at {base_url}. "
                        "Make sure the server is running and has models loaded."
                    )
        except (aiohttp.ClientError, json.JSONDecodeError) as e:
            raise RuntimeError(
                f"Failed to fetch models from server at {models_url}. "
                "Check that:\n"
                "1. The server is running\n"
                "2. The server URL is correct\n"
                f"Error: {e}"
            ) from e
```
**EN:** Function `get_first_model_from_server` coordinates benchmarking or serving-oriented control flow. The docstring highlights: Fetch the first model from the server's /v1/models endpoint. Key calls such as `aiohttp.TCPConnector`, `aiohttp.ClientSession`, `session.get`, `response.raise_for_status`, `response.json` show the concrete execution path.
**CN:** Function `get_first_model_from_server` 负责协调基准测试或服务侧控制流程。 文档字符串强调：Fetch the first model from the server's /v1/models endpoint. 像 `aiohttp.TCPConnector`, `aiohttp.ClientSession`, `session.get`, `response.raise_for_status`, `response.json` 这样的关键调用展示了该代码块的具体执行路径。

### Class `SpecDecodeMetrics` (lines 96-102)
```python
class SpecDecodeMetrics:
    """Speculative decoding metrics from the server's Prometheus endpoint."""

    num_drafts: int
    num_draft_tokens: int
    num_accepted_tokens: int
    accepted_per_pos: dict[int, int]
```
**EN:** Class `SpecDecodeMetrics` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. The class docstring says: Speculative decoding metrics from the server's Prometheus endpoint.
**CN:** 类 `SpecDecodeMetrics` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 类文档说明：Speculative decoding metrics from the server's Prometheus endpoint.

### Class `TaskType` (lines 169-171)
```python
class TaskType(Enum):
    GENERATION = "generation"
    POOLING = "pooling"
```
**EN:** Class `TaskType` is a structured building block in this module. It inherits from `Enum`.
**CN:** 类 `TaskType` 是该模块中的结构化构件，继承自 `Enum`。

### Class `BenchmarkMetrics` (lines 175-206)
```python
class BenchmarkMetrics:
    completed: int
    failed: int
    total_input: int
    total_output: int
    request_throughput: float
    request_goodput: float
    output_throughput: float
    total_token_throughput: float
    mean_ttft_ms: float
    median_ttft_ms: float
    std_ttft_ms: float
    percentiles_ttft_ms: list[tuple[float, float]]
    mean_tpot_ms: float
    median_tpot_ms: float
    std_tpot_ms: float
    percentiles_tpot_ms: list[tuple[float, float]]
    mean_itl_ms: float
    median_itl_ms: float
    std_itl_ms: float
    percentiles_itl_ms: list[tuple[float, float]]
    # E2EL stands for end-to-end latency per request.
    # It is the time taken on the client side from sending
    # ... omitted for brevity ...
    max_concurrent_requests: int
    rtfx: float = 0.0  # Inverse Real-Time Factor for ASR benchmarks
```
**EN:** Class `BenchmarkMetrics` is a structured building block in this module and uses a dataclass-style declaration to store explicit state.
**CN:** 类 `BenchmarkMetrics` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。

### Class `EmbedBenchmarkMetrics` (lines 210-219)
```python
class EmbedBenchmarkMetrics:
    completed: int
    failed: int
    total_input: int
    request_throughput: float
    total_token_throughput: float
    mean_e2el_ms: float
    std_e2el_ms: float
    median_e2el_ms: float
    percentiles_e2el_ms: float
```
**EN:** Class `EmbedBenchmarkMetrics` is a structured building block in this module and uses a dataclass-style declaration to store explicit state.
**CN:** 类 `EmbedBenchmarkMetrics` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。

### Function `get_request` (lines 247-344)
```python
async def get_request(
    input_requests: list[SampleRequest],
    request_rate: float,
    burstiness: float = 1.0,
    ramp_up_strategy: Literal["linear", "exponential"] | None = None,
    ramp_up_start_rps: int | None = None,
    ramp_up_end_rps: int | None = None,
) -> AsyncGenerator[tuple[SampleRequest, float], None]:
    """
    Asynchronously generates requests at a specified rate
    with OPTIONAL burstiness and OPTIONAL ramp-up strategy.

    Args:
        input_requests:
            A list of input requests, each represented as a SampleRequest.
        request_rate:
            The rate at which requests are generated (requests/s).
        burstiness (optional):
            The burstiness factor of the request generation.
            Only takes effect when request_rate is not inf.
            Default value is 1, which follows a Poisson process.
            Otherwise, the request intervals follow a gamma distribution.
            A lower burstiness value (0 < burstiness < 1) results
            in more bursty requests, while a higher burstiness value
            (burstiness > 1) results in a more uniform arrival of requests.
    # ... omitted for brevity ...
                await asyncio.sleep(sleep_interval_s)
        yield request, request_rates[request_index]
```
**EN:** Function `get_request` provides a reusable helper around the module's main workflow. The docstring highlights: Asynchronously generates requests at a specified rate with OPTIONAL burstiness and OPTIONAL ramp-up strategy. Key calls such as `isinstance`, `list`, `len`, `enumerate`, `_get_current_request_rate` show the concrete execution path.
**CN:** Function `get_request` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Asynchronously generates requests at a specified rate with OPTIONAL burstiness and OPTIONAL ramp-up strategy. 像 `isinstance`, `list`, `len`, `enumerate`, `_get_current_request_rate` 这样的关键调用展示了该代码块的具体执行路径。

### Function `calculate_metrics` (lines 396-605)
```python
def calculate_metrics(
    input_requests: list[SampleRequest],
    outputs: list[RequestFuncOutput],
    dur_s: float,
    tokenizer: TokenizerLike,
    selected_percentiles: list[float],
    goodput_config_dict: dict[str, float],
) -> tuple[BenchmarkMetrics, list[int]]:
    """Calculate the metrics for the benchmark.

    Args:
        input_requests: The input requests.
        outputs: The outputs of the requests.
        dur_s: The duration of the benchmark.
        tokenizer: The tokenizer to use.
        selected_percentiles: The percentiles to select.
        goodput_config_dict: The goodput configuration.

    Returns:
        A tuple of the benchmark metrics and the actual output lengths.
    """
    actual_output_lens: list[int] = []
    total_input = 0
    completed = 0
    good_completed = 0
    # ... omitted for brevity ...

    return metrics, actual_output_lens
```
**EN:** Function `calculate_metrics` provides a reusable helper around the module's main workflow. The docstring highlights: Calculate the metrics for the benchmark. Key calls such as `range`, `len`, `tokenizer`, `actual_output_lens.append`, `tpots.append` show the concrete execution path.
**CN:** Function `calculate_metrics` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Calculate the metrics for the benchmark. 像 `range`, `len`, `tokenizer`, `actual_output_lens.append`, `tpots.append` 这样的关键调用展示了该代码块的具体执行路径。

### Function `benchmark` (lines 608-1127)
```python
async def benchmark(
    task_type: TaskType,
    endpoint_type: str,
    api_url: str,
    base_url: str,
    model_id: str,
    model_name: str,
    tokenizer: TokenizerLike,
    input_requests: list[SampleRequest],
    logprobs: int | None,
    request_rate: float,
    burstiness: float,
    disable_tqdm: bool,
    num_warmups: int,
    profile: bool,
    selected_percentile_metrics: list[str],
    selected_percentiles: list[float],
    ignore_eos: bool,
    goodput_config_dict: dict[str, float],
    max_concurrency: int | None,
    lora_modules: Iterable[str] | None,
    extra_headers: dict | None,
    extra_body: dict | None,
    lora_assignment: Literal["random", "round-robin"] = "random",
    ramp_up_strategy: Literal["linear", "exponential"] | None = None,
    # ... omitted for brevity ...
    await session.close()
    return result
```
**EN:** Function `benchmark` coordinates benchmarking or serving-oriented control flow. Key calls such as `ValueError`, `aiohttp.TCPConnector`, `aiohttp.ClientSession`, `aiohttp.ClientTimeout`, `print` show the concrete execution path.
**CN:** Function `benchmark` 负责协调基准测试或服务侧控制流程。 像 `ValueError`, `aiohttp.TCPConnector`, `aiohttp.ClientSession`, `aiohttp.ClientTimeout`, `print` 这样的关键调用展示了该代码块的具体执行路径。

### Function `save_to_pytorch_benchmark_format` (lines 1168-1200)
```python
def save_to_pytorch_benchmark_format(
    args: argparse.Namespace, results: dict[str, Any], file_name: str
) -> None:
    metrics = [
        "median_ttft_ms",
        "mean_ttft_ms",
        "std_ttft_ms",
        "p99_ttft_ms",
        "mean_tpot_ms",
        "median_tpot_ms",
        "std_tpot_ms",
        "p99_tpot_ms",
        "median_itl_ms",
        "mean_itl_ms",
        "std_itl_ms",
        "p99_itl_ms",
    ]
    # These raw data might be useful, but they are rather big. They can be added
    # later if needed
    ignored_metrics = ["ttfts", "itls", "generated_texts", "errors"]
    pt_records = convert_to_pytorch_benchmark_format(
        args=args,
        metrics={k: [results[k]] for k in metrics if k in results},
        extra_info={
            k: results[k]
    # ... omitted for brevity ...
        pt_file = f"{os.path.splitext(file_name)[0]}.pytorch.json"
        write_to_json(pt_file, pt_records)
```
**EN:** Function `save_to_pytorch_benchmark_format` serializes data and writes it to a target representation. Key calls such as `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` show the concrete execution path.
**CN:** Function `save_to_pytorch_benchmark_format` 负责序列化数据并写入目标表示。 像 `convert_to_pytorch_benchmark_format`, `os.path.splitext`, `write_to_json` 这样的关键调用展示了该代码块的具体执行路径。

### Function `add_cli_args` (lines 1246-1630)
```python
def add_cli_args(parser: argparse.ArgumentParser):
    add_dataset_parser(parser)
    parser.add_argument(
        "--label",
        type=str,
        default=None,
        help="The label (prefix) of the benchmark results. If not specified, "
        "the value of '--backend' will be used as the label.",
    )
    parser.add_argument(
        "--backend",
        type=str,
        default="openai",
        choices=list(ASYNC_REQUEST_FUNCS.keys()),
        help="The type of backend or endpoint to use for the benchmark.",
    )
    parser.add_argument(
        "--base-url",
        type=str,
        default=None,
        help="Server or API base url if not using http host and port.",
    )
    # Use 127.0.0.1 here instead of localhost to force the use of ipv4
    parser.add_argument("--host", type=str, default="127.0.0.1")
    parser.add_argument("--port", type=int, default=8000)
    # ... omitted for brevity ...
        "prompt tokens, output tokens, and combined token distributions.",
    )
```
**EN:** Function `add_cli_args` parses configuration, arguments, or structured metadata. Key calls such as `add_dataset_parser`, `parser.add_argument`, `list`, `ASYNC_REQUEST_FUNCS.keys`, `float` show the concrete execution path.
**CN:** Function `add_cli_args` 负责解析配置、参数或结构化元数据。 像 `add_dataset_parser`, `parser.add_argument`, `list`, `ASYNC_REQUEST_FUNCS.keys`, `float` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main` (lines 1633-1634)
```python
def main(args: argparse.Namespace) -> dict[str, Any]:
    return asyncio.run(main_async(args))
```
**EN:** Function `main` provides a reusable helper around the module's main workflow. Key calls such as `asyncio.run`, `main_async` show the concrete execution path.
**CN:** Function `main` 为模块主流程提供可复用的辅助逻辑。 像 `asyncio.run`, `main_async` 这样的关键调用展示了该代码块的具体执行路径。

### Function `main_async` (lines 1637-1999)
```python
async def main_async(args: argparse.Namespace) -> dict[str, Any]:
    print(args)
    random.seed(args.seed)
    np.random.seed(args.seed)

    # Validate timeline ITL thresholds
    if args.plot_timeline:
        try:
            itl_thresholds = [
                float(t.strip()) for t in args.timeline_itl_thresholds.split(",")
            ]
            if len(itl_thresholds) != 2:
                raise ValueError(
                    f"Expected 2 ITL threshold values, got {len(itl_thresholds)}"
                )
        except ValueError as e:
            raise ValueError(f"Invalid --timeline-itl-thresholds format: {e}") from e

    # Validate ramp-up arguments
    if args.ramp_up_strategy is not None:
        if args.request_rate != float("inf"):
            raise ValueError(
                "When using ramp-up, do not specify --request-rate. "
                "The request rate will be controlled by ramp-up parameters. "
                "Please remove the --request-rate argument."
    # ... omitted for brevity ...

    return result_json
```
**EN:** Function `main_async` provides a reusable helper around the module's main workflow. Key calls such as `print`, `random.seed`, `np.random.seed`, `float`, `t.strip` show the concrete execution path.
**CN:** Function `main_async` 为模块主流程提供可复用的辅助逻辑。 像 `print`, `random.seed`, `np.random.seed`, `float`, `t.strip` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **CLI workflow / 命令行流程**
  - **EN:** The module exposes a command-line entrypoint so it can be run as a standalone tool.
  - **CN:** 该模块暴露了命令行入口，因此可以作为独立工具运行。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import argparse`, `import asyncio`, `import contextlib`, `import importlib.util`, `import json`, `import os`, `import random`, `import shutil`, `import ssl`, `import time`, `import uuid`, `import warnings`
- **Third-party / 第三方**: `import aiohttp`, `import numpy as np`, `from tqdm.asyncio import tqdm`
- **vLLM internal / vLLM 内部依赖**: `from vllm.benchmarks.datasets import SampleRequest, add_dataset_parser, get_samples`, `from vllm.benchmarks.lib.endpoint_request_func import ASYNC_REQUEST_FUNCS, OPENAI_COMPATIBLE_BACKENDS, POOLING_BACKENDS, RequestFuncInput, RequestFuncOutput`, `from vllm.benchmarks.lib.ready_checker import wait_for_endpoint`, `from vllm.benchmarks.lib.utils import convert_to_pytorch_benchmark_format, write_to_json`, `from vllm.tokenizers import TokenizerLike, get_tokenizer`, `from vllm.utils.gc_utils import freeze_gc_heap`, `from vllm.utils.network_utils import join_host_port`
