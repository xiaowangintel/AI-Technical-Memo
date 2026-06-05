# benchmark_serving_structured_output.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_serving_structured_output.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, serving-side benchmarks, centered around `MILLISECONDS_TO_SECONDS_CONVERSION`, `BenchmarkMetrics`, `SampleRequest`, `sample_requests`. / 实现与基准测试编排、服务侧基准相关的逻辑，核心符号包括 `MILLISECONDS_TO_SECONDS_CONVERSION`, `BenchmarkMetrics`, `SampleRequest`, `sample_requests`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-21)
```python
r"""Benchmark online serving throughput with structured outputs.

On the server side, run one of the following commands:
    (vLLM OpenAI API server)
    vllm serve <your_model>

On the client side, run:
    python benchmarks/benchmark_serving_structured_output.py \
        --backend <backend> \
        --model <your_model> \
        --dataset json \
        --structured-output-ratio 1.0 \
        --request-rate 10 \
        --num-prompts 1000

    when using tgi backend, add
        --endpoint /generate_stream
    to the end of the command above.
"""
```
**EN:** The module docstring introduces Benchmark online serving throughput with structured outputs. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark online serving throughput with structured outputs 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 23-46)
```python
import argparse
import asyncio
import copy
import dataclasses
import json
import os
import random
import time
import uuid
import warnings
from collections.abc import AsyncGenerator
from contextlib import nullcontext
from dataclasses import dataclass

import datasets
import numpy as np
import pandas as pd
from backend_request_func import (
    ASYNC_REQUEST_FUNCS,
    RequestFuncInput,
    RequestFuncOutput,
)
from tqdm.asyncio import tqdm
from transformers import PreTrainedTokenizerBase
```
**EN:** This block gathers standard-library helpers such as `argparse`, `asyncio`, `copy`, `dataclasses`, `json`, `os`; third-party packages such as `datasets`, `numpy`, `pandas`, `tqdm.asyncio`, `transformers`; project-local modules such as `backend_request_func`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `asyncio`, `copy`, `dataclasses`, `json`, `os`；第三方依赖，如 `datasets`, `numpy`, `pandas`, `tqdm.asyncio`, `transformers`；项目内部模块，如 `backend_request_func`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 48-56)
```python
try:
    from vllm.tokenizers import get_tokenizer
except ImportError:
    from backend_request_func import get_tokenizer

try:
    from vllm.utils.argparse_utils import FlexibleArgumentParser
except ImportError:
    from argparse import ArgumentParser as FlexibleArgumentParser
```
**EN:** This top-level block prepares shared state such as module-level state. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 module-level state。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Additional imports (lines 58-60)
```python
from vllm.v1.structured_output.backend_xgrammar import (
    has_xgrammar_unsupported_json_features,
)
```
**EN:** This block gathers project-local modules such as `vllm.v1.structured_output.backend_xgrammar`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.v1.structured_output.backend_xgrammar`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 62-62)
```python
MILLISECONDS_TO_SECONDS_CONVERSION = 1000
```
**EN:** This top-level block prepares shared state such as `MILLISECONDS_TO_SECONDS_CONVERSION`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `MILLISECONDS_TO_SECONDS_CONVERSION`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `BenchmarkMetrics` (lines 66-92)
```python
class BenchmarkMetrics:
    completed: int
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
    # a request to receiving a complete response.
    mean_e2el_ms: float
    median_e2el_ms: float
    std_e2el_ms: float
    percentiles_e2el_ms: list[tuple[float, float]]
```
**EN:** Class `BenchmarkMetrics` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `BenchmarkMetrics` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Class `SampleRequest` (lines 96-112)
```python
class SampleRequest:
    """A class representing a single inference request for benchmarking.

    Attributes:
        prompt: The input text prompt for the model.
        multi_modal_data: Optional dictionary containing multi-modal data (e.g.
            images).
        prompt_len: The length of the prompt in tokens.
        expected_output_len: The expected length of the output in tokens.
    """

    prompt: str
    prompt_len: int
    expected_output_len: int
    schema: dict
    structure_type: str
    completion: str = None
```
**EN:** Class `SampleRequest` is the main object-oriented wrapper for this module. Its docstring says: A class representing a single inference request for benchmarking. It extends `object` and organizes behavior through its methods.
**CN:** 类 `SampleRequest` 是该模块中的主要面向对象封装。文档字符串指出：A class representing a single inference request for benchmarking。它继承自 `object`，并通过 its methods 组织行为。

### Function `sample_requests` (lines 115-300)
```python
def sample_requests(
    tokenizer: PreTrainedTokenizerBase, args: argparse.Namespace
) -> list[SampleRequest]:
    def _apply_random_prefix(
        tokenizer: PreTrainedTokenizerBase,
        requests: list[SampleRequest],
        prefix_len: int,
        seed: int,
    ) -> list[SampleRequest]:
        if prefix_len <= 0:
            return requests
        rng = np.random.default_rng(seed)
        vocab_size = tokenizer.vocab_size
        prohibited = getattr(tokenizer, "all_special_ids", None) or []
        allowed = np.array([i for i in range(vocab_size) if i not in prohibited])
        if len(allowed) == 0:
            return requests
        prefix_ids = rng.integers(0, len(allowed), size=prefix_len)
        prefix_token_ids = allowed[prefix_ids].tolist()
        out = []
        for req in requests:
            prompt_ids = tokenizer(req.prompt, add_special_tokens=False).input_ids
            full_ids = prefix_token_ids + prompt_ids
            full_prompt = tokenizer.decode(full_ids, skip_special_tokens=False)
            out.append(
                SampleRequest(
                    prompt=full_prompt,
                    prompt_len=len(tokenizer(full_prompt).input_ids),
                    expected_output_len=req.expected_output_len,
                    schema=req.schema,
                    structure_type=req.structure_type,
                    completion=req.completion,
                )
            )
    # ... omitted for brevity ...
                    completion=completion,
                )
            )

    requests = _apply_random_prefix(
        tokenizer, requests, args.random_prefix_len, args.seed
    )
    return requests
```
**EN:** `sample_requests` implements backend request helpers. It mainly works with `tokenizer`, `args` and relies on `np.random.default_rng`, `getattr`, `np.array`, `range`, `len`, `rng.integers` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `sample_requests` 负责实现后端请求辅助逻辑。 它主要处理 `tokenizer`, `args`，并结合 `np.random.default_rng`, `getattr`, `np.array`, `range`, `len`, `rng.integers` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Async function `get_request` (lines 303-345)
```python
async def get_request(
    input_requests: list[SampleRequest],
    request_rate: float,
    burstiness: float = 1.0,
) -> AsyncGenerator[tuple[int, SampleRequest], None]:
    """
    Asynchronously generates requests at a specified rate
    with OPTIONAL burstiness.

    Args:
        input_requests:
            A list of input requests, each represented as a tuple.
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
    """
    input_requests = iter(input_requests)

    # Calculate scale parameter theta to maintain the desired request_rate.
    assert burstiness > 0, (
        f"A positive burstiness factor is expected, but given {burstiness}."
    )
    theta = 1.0 / (request_rate * burstiness)

    for i, request in enumerate(input_requests):
        yield i, request

        if request_rate == float("inf"):
            # If the request rate is infinity, then we don't need to wait.
            continue

        # Sample the request interval from the gamma distribution.
        # If burstiness is 1, it follows exponential distribution.
        interval = np.random.gamma(shape=burstiness, scale=theta)
        # The next request will be sent after the interval.
        await asyncio.sleep(interval)
```
**EN:** `get_request` Asynchronously generates requests at a specified rate with OPTIONAL burstiness. It mainly works with `input_requests`, `request_rate`, `burstiness` and relies on `iter`, `enumerate`, `float`, `np.random.gamma`, `asyncio.sleep` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `get_request` 的职责是：Asynchronously generates requests at a specified rate with OPTIONAL burstiness。 它主要处理 `input_requests`, `request_rate`, `burstiness`，并结合 `iter`, `enumerate`, `float`, `np.random.gamma`, `asyncio.sleep` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `calculate_metrics` (lines 348-458)
```python
def calculate_metrics(
    input_requests: list[tuple[str, int, int]],
    outputs: list[RequestFuncOutput],
    dur_s: float,
    tokenizer: PreTrainedTokenizerBase,
    selected_percentile_metrics: list[str],
    selected_percentiles: list[float],
    goodput_config_dict: dict[str, float] | None = None,
) -> tuple[BenchmarkMetrics, list[int]]:
    actual_output_lens: list[int] = []
    total_input = 0
    completed = 0
    good_completed = 0
    itls: list[float] = []
    tpots: list[float] = []
    all_tpots: list[float] = []
    ttfts: list[float] = []
    e2els: list[float] = []
    for i in range(len(outputs)):
        if outputs[i].success:
            # We use the tokenizer to count the number of output tokens for all
            # serving backends instead of looking at len(outputs[i].itl) since
            # multiple output tokens may be bundled together
            # Note : this may inflate the output token count slightly
            output_len = len(
                tokenizer(outputs[i].generated_text, add_special_tokens=False).input_ids
            )
            actual_output_lens.append(output_len)
            total_input += input_requests[i].prompt_len
            tpot = 0
            if output_len > 1:
                latency_minus_ttft = outputs[i].latency - outputs[i].ttft
                tpot = latency_minus_ttft / (output_len - 1)
                tpots.append(tpot)
    # ... omitted for brevity ...
        std_e2el_ms=np.std(e2els or 0) * 1000,
        median_e2el_ms=np.median(e2els or 0) * 1000,
        percentiles_e2el_ms=[
            (p, np.percentile(e2els or 0, p) * 1000) for p in selected_percentiles
        ],
    )

    return metrics, actual_output_lens
```
**EN:** `calculate_metrics` implements a helper used by `benchmark_serving_structured_output.py`. It mainly works with `input_requests`, `outputs`, `dur_s`, `tokenizer`, `selected_percentile_metrics`, ... and relies on `range`, `len`, `tokenizer`, `actual_output_lens.append`, `tpots.append`, `all_tpots.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `calculate_metrics` 负责实现 `benchmark_serving_structured_output.py` 使用的辅助逻辑。 它主要处理 `input_requests`, `outputs`, `dur_s`, `tokenizer`, `selected_percentile_metrics`, ...，并结合 `range`, `len`, `tokenizer`, `actual_output_lens.append`, `tpots.append`, `all_tpots.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Async function `benchmark` (lines 461-703)
```python
async def benchmark(
    backend: str,
    api_url: str,
    base_url: str,
    model_id: str,
    tokenizer: PreTrainedTokenizerBase,
    input_requests: list[SampleRequest],
    request_rate: float,
    burstiness: float,
    disable_tqdm: bool,
    profile: bool,
    selected_percentile_metrics: list[str],
    selected_percentiles: list[str],
    ignore_eos: bool,
    max_concurrency: int | None,
    structured_output_ratio: float,
    goodput_config_dict: dict[str, float] | None = None,
):
    if backend in ASYNC_REQUEST_FUNCS:
        request_func = ASYNC_REQUEST_FUNCS[backend]
    else:
        raise ValueError(f"Unknown backend: {backend}")

    def prepare_extra_body(request) -> dict:
        extra_body = {}
        # Add the schema to the extra_body
        extra_body["structured_outputs"] = {}
        extra_body["structured_outputs"][request.structure_type] = request.schema
        return extra_body

    print("Starting initial single prompt test run...")
    structured_output_req_idx = random.sample(
        range(len(input_requests)), int(len(input_requests) * structured_output_ratio)
    )
    # ... omitted for brevity ...
            output_len=test_request.expected_output_len,
            extra_body={test_request.structure_type: test_request.schema},
        )
        profile_output = await request_func(request_func_input=profile_input)
        if profile_output.success:
            print("Profiler stopped")

    return result, ret
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `backend`, `api_url`, `base_url`, `model_id`, `tokenizer`, ... and relies on `ValueError`, `print`, `random.sample`, `range`, `len`, `int` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `backend`, `api_url`, `base_url`, `model_id`, `tokenizer`, ...，并结合 `ValueError`, `print`, `random.sample`, `range`, `len`, `int` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `evaluate` (lines 706-750)
```python
def evaluate(ret, args):
    def _eval_correctness_json(expected, actual):
        # extract json string from string using regex
        import regex as re

        actual = actual.replace("\n", "").replace(" ", "").strip()
        try:
            actual = re.search(r"\{.*\}", actual).group()
            actual = json.loads(actual)
        except Exception:
            return False

        return True

    def _eval_correctness_choice(expected, actual):
        return actual in args.choice

    def _eval_correctness_regex(expected, actual):
        import regex as re

        return re.match(args.regex, actual) is not None

    def _eval_correctness(expected, actual):
        if args.structure_type == "json":
            return _eval_correctness_json(expected, actual)
        elif args.structure_type == "regex":
            return _eval_correctness_regex(expected, actual)
        elif args.structure_type == "choice":
            return _eval_correctness_choice(expected, actual)
        else:
            return None

    scores = []
    for res in ret:
        score = _eval_correctness(res["expected"], res["generated"])
        res["correctness"] = score
        scores.append(score)

    not_none_scores = [score for score in scores if score is not None]

    return (
        (sum(not_none_scores) / len(not_none_scores) * 100)
        if len(not_none_scores) > 0
        else None
    )
```
**EN:** `evaluate` implements a helper used by `benchmark_serving_structured_output.py`. It mainly works with `ret`, `args` and relies on `actual.replace.replace.strip`, `actual.replace.replace`, `actual.replace`, `re.search.group`, `re.search`, `json.loads` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `evaluate` 负责实现 `benchmark_serving_structured_output.py` 使用的辅助逻辑。 它主要处理 `ret`, `args`，并结合 `actual.replace.replace.strip`, `actual.replace.replace`, `actual.replace`, `re.search.group`, `re.search`, `json.loads` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Function `parse_goodput` (lines 753-766)
```python
def parse_goodput(slo_pairs):
    goodput_config_dict = {}
    try:
        for slo_pair in slo_pairs:
            slo_name, slo_val = slo_pair.split(":")
            goodput_config_dict[slo_name] = float(slo_val)
    except ValueError as err:
        raise argparse.ArgumentTypeError(
            "Invalid format found for service level objectives. "
            'Specify service level objectives for goodput as "KEY:VALUE" '
            "pairs, where the key is a metric name, and the value is a "
            "number in milliseconds."
        ) from err
    return goodput_config_dict
```
**EN:** `parse_goodput` parses external input into structured benchmark settings. It mainly works with `slo_pairs` and relies on `slo_pair.split`, `float`, `argparse.ArgumentTypeError` plus iteration and error handling to move data through this part of the benchmark pipeline.
**CN:** `parse_goodput` 负责把外部输入解析为结构化的基准配置。 它主要处理 `slo_pairs`，并结合 `slo_pair.split`, `float`, `argparse.ArgumentTypeError` 以及 循环迭代、异常处理 来完成这一段基准测试流程。

### Function `check_goodput_args` (lines 769-787)
```python
def check_goodput_args(args):
    goodput_config_dict = {}
    VALID_NAMES = ["ttft", "tpot", "e2el"]
    if args.goodput:
        goodput_config_dict = parse_goodput(args.goodput)
        for slo_name, slo_val in goodput_config_dict.items():
            if slo_name not in VALID_NAMES:
                raise ValueError(
                    f"Invalid metric name found, {slo_name}: {slo_val}. "
                    "The service level objective name should be one of "
                    f"{str(VALID_NAMES)}. "
                )
            if slo_val < 0:
                raise ValueError(
                    f"Invalid value found, {slo_name}: {slo_val}. "
                    "The service level objective value should be "
                    "non-negative."
                )
    return goodput_config_dict
```
**EN:** `check_goodput_args` implements a helper used by `benchmark_serving_structured_output.py`. It mainly works with `args` and relies on `parse_goodput`, `goodput_config_dict.items`, `ValueError`, `str` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `check_goodput_args` 负责实现 `benchmark_serving_structured_output.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `parse_goodput`, `goodput_config_dict.items`, `ValueError`, `str` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 790-884)
```python
def main(args: argparse.Namespace):
    print(args)
    random.seed(args.seed)
    np.random.seed(args.seed)

    backend = args.backend
    model_id = args.model
    tokenizer_id = args.tokenizer if args.tokenizer is not None else args.model

    if args.base_url is not None:
        api_url = f"{args.base_url}{args.endpoint}"
        base_url = f"{args.base_url}"
    else:
        api_url = f"http://{args.host}:{args.port}{args.endpoint}"
        base_url = f"http://{args.host}:{args.port}"

    tokenizer = get_tokenizer(
        tokenizer_id,
        trust_remote_code=args.trust_remote_code,
        tokenizer_mode=args.tokenizer_mode,
    )

    if args.dataset == "grammar":
        args.structure_type = "grammar"
    elif args.dataset == "regex":
        args.structure_type = "regex"
    elif args.dataset == "choice":
        args.structure_type = "choice"
    else:
        args.structure_type = "json"

    if args.no_structured_output:
        args.structured_output_ratio = 0
    if args.save_results:
    # ... omitted for brevity ...

        # Save to file
        if args.result_filename:
            result_file_name = args.result_filename
        if args.result_dir:
            result_file_name = os.path.join(args.result_dir, result_file_name)
        with open(result_file_name, "w", encoding="utf-8") as outfile:
            json.dump(results, outfile, indent=4)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `random.seed`, `np.random.seed`, `get_tokenizer`, `args.model.split`, `sample_requests` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `random.seed`, `np.random.seed`, `get_tokenizer`, `args.model.split`, `sample_requests` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Function `create_argument_parser` (lines 887-1079)
```python
def create_argument_parser():
    parser = FlexibleArgumentParser(
        description="Benchmark the online serving throughput."
    )
    parser.add_argument(
        "--backend",
        type=str,
        default="vllm",
        choices=list(ASYNC_REQUEST_FUNCS.keys()),
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
    parser.add_argument(
        "--endpoint",
        type=str,
        default="/v1/completions",
        help="API endpoint.",
    )
    parser.add_argument(
        "--dataset",
        default="json",
        choices=["json", "json-unique", "grammar", "regex", "choice", "xgrammar_bench"],
    )
    parser.add_argument(
        "--json-schema-path", type=str, default=None, help="Path to json schema."
    )
    parser.add_argument(
    # ... omitted for brevity ...
    parser.add_argument(
        "--structured-output-ratio",
        type=float,
        default=1.0,
        help="Ratio of Structured Outputs requests",
    )

    return parser
```
**EN:** `create_argument_parser` constructs shared runtime objects for the benchmark. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `list`, `ASYNC_REQUEST_FUNCS.keys`, `float` plus value production to move data through this part of the benchmark pipeline.
**CN:** `create_argument_parser` 负责构建基准测试共享的运行时对象。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `list`, `ASYNC_REQUEST_FUNCS.keys`, `float` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 1082-1085)
```python
if __name__ == "__main__":
    parser = create_argument_parser()
    args = parser.parse_args()
    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `create_argument_parser`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `create_argument_parser`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `asyncio`, `copy`, `dataclasses`, `json`, `os`.
- **CN:** 标准库依赖：`argparse`, `asyncio`, `copy`, `dataclasses`, `json`, `os`。
- **EN:** Third-party packages: `datasets`, `numpy`, `pandas`, `tqdm.asyncio`, `transformers`.
- **CN:** 第三方依赖：`datasets`, `numpy`, `pandas`, `tqdm.asyncio`, `transformers`。
- **EN:** Internal modules: `backend_request_func`, `vllm.v1.structured_output.backend_xgrammar`.
- **CN:** 内部模块：`backend_request_func`, `vllm.v1.structured_output.backend_xgrammar`。
