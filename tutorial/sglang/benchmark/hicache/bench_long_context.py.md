# bench_long_context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/bench_long_context.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache long context. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hicache long context 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
import json
import queue
import time

import requests
from bench_multiturn import (
    ReadyQueue,
    WorkloadGenerator,
    gen_payload,
    log_to_jsonl_file,
    parse_args,
)
from tqdm.asyncio import tqdm

from sglang.benchmark.utils import get_tokenizer
from sglang.test.kits.cache_hit_kit import async_request_sglang_generate
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 19-19: Class `ContextWorkloadGenerator` declaration / 类 `ContextWorkloadGenerator` 声明
```python
class ContextWorkloadGenerator(WorkloadGenerator):
```
**EN:** This block introduces class `ContextWorkloadGenerator`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `ContextWorkloadGenerator`，用于通过统一接口组织相关行为。

### Lines 20-66: Method `ContextWorkloadGenerator.__init__` / 方法 `ContextWorkloadGenerator.__init__`
```python
    def __init__(self, args):
        self.url = f"http://{args.host}:{args.port}/generate"
        self.request_func = async_request_sglang_generate

        self.tokenizer = get_tokenizer(args.model_path)
        self.distribution = args.distribution
        self.request_rate = args.request_rate
        self.start_time = None
        self.finished_time = None

        self.sent_requests = 0
        self.completed_requests = 0

        self.dataset = json.load(open(args.dataset_path))
        num_requests = min(args.num_clients, len(self.dataset["queries"]))

        init_requests = []
        for i in range(num_requests):
            context_id = self.dataset["queries"][i]["context"]
            # Tokenize the context + question to get input_ids
            prompt_text = (
                self.dataset["contexts"][context_id]
                + self.dataset["queries"][i]["question"]
            )
            input_ids = self.tokenizer.encode(prompt_text)
            output_len = len(
                self.tokenizer(self.dataset["queries"][i]["reference_answer"])[
                    "input_ids"
                ]
            )
            init_requests.append((i, gen_payload(input_ids, output_len)))
        self.ready_queue = ReadyQueue(init_requests=init_requests)

        self.response_queue = queue.Queue()
        self.pbar = tqdm(total=num_requests)
        self.performance_metrics = {
            "ttft": [],
            "latency": [],
            "itl": [],
            "prompt_len": [],
            "cached_tokens": [],
            "generated_len": [],
        }

        self.max_parallel = args.max_parallel
        self.logfile = args.log_file
        self.enable_round_barrier = False
```
**EN:** `ContextWorkloadGenerator.__init__` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets. Notable calls include `get_tokenizer`, `json.load`, `min`.
**CN:** `ContextWorkloadGenerator.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `get_tokenizer`, `json.load`, `min`。

### Lines 68-86: Method `ContextWorkloadGenerator.response_handler` / 方法 `ContextWorkloadGenerator.response_handler`
```python
    def response_handler(self):
        while True:
            try:
                client_id, response = self.response_queue.get(
                    timeout=10
                )  # Block until response is available
                if not response.success:
                    raise ValueError(f"Request failed with error: {response.error}")
                self.performance_metrics["ttft"].append(response.ttft)
                self.performance_metrics["itl"].extend(response.itl)
                self.performance_metrics["latency"].append(response.latency)
                self.performance_metrics["prompt_len"].append(response.prompt_len)
                self.performance_metrics["cached_tokens"].append(response.cached_tokens)
                self.performance_metrics["generated_len"].append(response.generated_len)
                self.completed_requests += 1

            except queue.Empty:
                if self.pbar.n == self.pbar.total:
                    break
```
**EN:** `ContextWorkloadGenerator.response_handler` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. Notable calls include `self.response_queue.get`, `self.performance_metrics['ttft'].append`, `self.performance_metrics['itl'].extend`.
**CN:** `ContextWorkloadGenerator.response_handler` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `self.response_queue.get`, `self.performance_metrics['ttft'].append`, `self.performance_metrics['itl'].extend`。

### Lines 89-100: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    args = parse_args()
    args.num_rounds = 1
    args.max_parallel = 24
    flush_cache_url = f"http://{args.host}:{args.port}/flush_cache"

    for request_rate in [24, 16, 12, 8, 4, 2, 1]:
        args.request_rate = request_rate
        requests.post(flush_cache_url)
        time.sleep(1)
        performance_data = ContextWorkloadGenerator(args).run()
        log_to_jsonl_file(performance_data, args.log_file, args.tag)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `queue`, `time`
- **Third-party / 第三方依赖**: `requests`, `bench_multiturn`, `tqdm.asyncio`
- **Internal / 项目内部依赖**: `sglang.benchmark.utils`, `sglang.test.kits.cache_hit_kit`
