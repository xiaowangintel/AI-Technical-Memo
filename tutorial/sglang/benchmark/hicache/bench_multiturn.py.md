# bench_multiturn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hicache/bench_multiturn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hicache multiturn. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 hicache multiturn 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and setup / 导入与初始化
```python
import argparse
import asyncio
import json
import queue
import random
import threading
import time
from datetime import datetime

import numpy as np
import requests
from tqdm.asyncio import tqdm

from sglang.bench_serving import RequestFuncOutput
from sglang.benchmark.datasets.random import sample_random_requests
from sglang.benchmark.utils import get_tokenizer
from sglang.test.kits.cache_hit_kit import (
    async_request_openai_chat_completions,
    async_request_sglang_generate,
    gen_payload,
    gen_payload_openai,
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

### Lines 25-168: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Script to benchmark concurrent requests to a server."
    )
    parser.add_argument(
        "--num-clients",
        type=int,
        default=256,
        help="Number of concurrent clients",
    )
    parser.add_argument(
        "--max-parallel",
        type=int,
        default=128,
        help="Maximum number of parallel requests",
    )
    parser.add_argument(
        "--request-length",
        type=int,
        default=512,
        help="Length of each new request",
    )
    parser.add_argument(
        "--output-length",
        type=int,
        default=64,
        help="Length of each output",
    )
    parser.add_argument(
        "--num-rounds",
        type=int,
        default=5,
        help="Number of rounds per client",
    )
    parser.add_argument(
        "--distribution",
        type=str,
        default="poisson",
        choices=["poisson", "uniform"],
        help="Distribution type for request intervals (poisson or uniform)",
    )
    parser.add_argument(
        "--request-rate",
        type=float,
        default=1.0,
        help="Average number of requests per second",
    )
    parser.add_argument(
        "--host",
        type=str,
        default="localhost",
        help="Server hostname or IP (default: localhost)",
    )
    parser.add_argument(
        "--port",
        type=int,
        default=30000,
        help="Server port (default: 30000)",
    )
    parser.add_argument(
        "--model-path",
        type=str,
        default="meta-llama/Llama-3.1-8B-Instruct",
        help="model path compatible with Hugging Face Transformers",
    )
    parser.add_argument(
        "--dataset-path",
        type=str,
        default="",
        help="local dataset to sample tokens from",
    )
    parser.add_argument(
        "--log-file",
        type=str,
        default="performance_metrics.jsonl",
        help="File to log performance metrics",
    )
    parser.add_argument(
        "--disable-auto-run",
        action="store_true",
        help="If set, disable automatically testing with a range of request rates.",
    )
    parser.add_argument(
        "--disable-random-sample",
        action="store_true",
        help="If set, disable random sampling of requests from the ShareGPT dataset.",
    )
    parser.add_argument(
        "--enable-round-barrier",
        action="store_true",
        help="If set, only send i-th turn requests after all (i-1)-th turn requests finished.",
    )
    parser.add_argument(
        "--sub-question-input-length",
        type=int,
        default=0,
        help="Length of the sub question input for each request, if set 0 use request_length",
    )
    parser.add_argument(
        "--ready-queue-policy",
        type=str,
        default="random",
        help="Policy for popping requests from the ready queue (random or fifo)",
    )
    parser.add_argument(
        "--tag",
        type=str,
        default="",
        help="Tag of a certain run in the log file",
    )
    parser.add_argument(
        "--min-rounds",
        type=int,
        default=0,
        help="Min rounds per client (0 = use --num-rounds)",
    )
    parser.add_argument(
        "--max-rounds",
        type=int,
        default=0,
        help="Max rounds per client (0 = use --num-rounds)",
    )
    parser.add_argument(
        "--range-ratio",
        type=float,
        default=1.0,
        help="Length variation ratio for prompts and outputs (1.0 = no variation, 0.5 = 50%% variation)",
    )
    parser.add_argument("--seed", type=int, default=1, help="The random seed.")
    parser.add_argument(
        "--lora-path",
        type=str,
        default="",
        help="String of LoRA path. Currently we only support benchmarking on a single LoRA adaptor.",
    )
    parser.add_argument(
        "--api-format",
        type=str,
        default="sglang",
        choices=["sglang", "openai"],
        help="API format to use: 'sglang' for native /generate endpoint, "
        "'openai' for OpenAI-compatible /v1/chat/completions endpoint.",
    )
    return parser.parse_args()
```
**EN:** `parse_args` is a function that builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. It returns `parser.parse_args()` to the caller. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `parse_args` 是一个函数，用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `parser.parse_args()`。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 171-180: Function `log_to_jsonl_file` / 函数 `log_to_jsonl_file`
```python
def log_to_jsonl_file(data, file_path="performance_metrics.jsonl", tag=""):
    """Append the data with a timestamp and tag to the specified JSONL file."""
    timestamped_data = {"timestamp": datetime.now().isoformat(), "tag": tag, **data}
    try:
        with open(file_path, "a") as file:
            file.write(
                json.dumps(timestamped_data) + "\n"
            )  # Write as a single line in JSONL format
    except IOError as e:
        print(f"Error writing to JSONL file: {e}")
```
**EN:** `log_to_jsonl_file` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics. The docstring frames it as: Append the data with a timestamp and tag to the specified JSONL file. Notable calls include `datetime.now().isoformat`, `open`, `file.write`.
**CN:** `log_to_jsonl_file` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `datetime.now().isoformat`, `open`, `file.write`。

### Lines 183-183: Class `ReadyQueue` declaration / 类 `ReadyQueue` 声明
```python
class ReadyQueue:
```
**EN:** This block introduces class `ReadyQueue`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `ReadyQueue`，用于通过统一接口组织相关行为。

### Lines 184-186: Class-level state / 类级状态
```python
    """
    Thread-safe queue that can pop requests in different orders based on given policy.
    """
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it sends requests to serving or OpenAI-compatible APIs.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求。

### Lines 188-191: Method `ReadyQueue.__init__` / 方法 `ReadyQueue.__init__`
```python
    def __init__(self, init_requests=None, policy="random"):
        self.lock = threading.Lock()
        self.requests = init_requests or []
        self.policy = policy
```
**EN:** `ReadyQueue.__init__` is a method that sends requests to serving or OpenAI-compatible APIs. Notable calls include `threading.Lock`.
**CN:** `ReadyQueue.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求。其中较关键的调用包括 `threading.Lock`。

### Lines 193-195: Method `ReadyQueue.append` / 方法 `ReadyQueue.append`
```python
    def append(self, item):
        with self.lock:
            self.requests.append(item)
```
**EN:** `ReadyQueue.append` is a method that sends requests to serving or OpenAI-compatible APIs. Notable calls include `self.requests.append`.
**CN:** `ReadyQueue.append` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求。其中较关键的调用包括 `self.requests.append`。

### Lines 197-208: Method `ReadyQueue.pop` / 方法 `ReadyQueue.pop`
```python
    def pop(self):
        with self.lock:
            if not self.requests:
                return None
            if self.policy == "random":
                index = random.randrange(len(self.requests))
                return self.requests.pop(index)
            elif self.policy == "fifo":
                return self.requests.pop(0)
            else:
                # todo, varying thinking time of clients
                raise ValueError(f"{self.policy} not implemented")
```
**EN:** `ReadyQueue.pop` is a method that sends requests to serving or OpenAI-compatible APIs. It returns `None` to the caller. Notable calls include `random.randrange`, `self.requests.pop`, `len`.
**CN:** `ReadyQueue.pop` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求。它会向调用方返回 `None`。其中较关键的调用包括 `random.randrange`, `self.requests.pop`, `len`。

### Lines 211-211: Class `WorkloadGenerator` declaration / 类 `WorkloadGenerator` 声明
```python
class WorkloadGenerator:
```
**EN:** This block introduces class `WorkloadGenerator`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `WorkloadGenerator`，用于通过统一接口组织相关行为。

### Lines 212-386: Method `WorkloadGenerator.__init__` / 方法 `WorkloadGenerator.__init__`
```python
    def __init__(self, args):
        self.api_format = args.api_format
        self.model_path = args.model_path

        # Construct the base URL and select request/payload functions
        if self.api_format == "openai":
            self.url = f"http://{args.host}:{args.port}/v1/chat/completions"
            self.request_func = async_request_openai_chat_completions
        else:
            self.url = f"http://{args.host}:{args.port}/generate"
            self.request_func = async_request_sglang_generate

        self.tokenizer = get_tokenizer(args.model_path)
        self.distribution = args.distribution
        self.request_rate = args.request_rate
        self.start_time = None
        self.finished_time = None
        self.lora_path = args.lora_path

        self.sent_requests = 0
        self.completed_requests = 0

        # Resolve per-client round counts
        min_rounds = args.min_rounds
        max_rounds = args.max_rounds
        if min_rounds == 0 and max_rounds == 0:
            # Backward compat: all clients use --num-rounds
            min_rounds = args.num_rounds
            max_rounds = args.num_rounds
        elif min_rounds == 0:
            min_rounds = max_rounds
        elif max_rounds == 0:
            max_rounds = min_rounds
        if min_rounds < 1:
            raise ValueError(f"--min-rounds must be >= 1, got {min_rounds}")
        if min_rounds > max_rounds:
            raise ValueError(
                f"--min-rounds ({min_rounds}) must be <= --max-rounds ({max_rounds})"
            )

        self.min_rounds = min_rounds
        self.max_rounds = max_rounds

        if min_rounds == max_rounds:
            # All clients have the same round count; skip randint to preserve random state
            self.client_total_rounds = [min_rounds] * args.num_clients
        else:
            self.client_total_rounds = [
                random.randint(min_rounds, max_rounds) for _ in range(args.num_clients)
            ]

        # clients_per_round[r] = number of clients participating in round r
        self.clients_per_round = [
            sum(1 for t in self.client_total_rounds if t > r) for r in range(max_rounds)
        ]
        self.total_requests = sum(self.client_total_rounds)

        range_ratio = args.range_ratio

        # Use return_text=False to get token ids instead of text
        first_round_samples = sample_random_requests(
            input_len=args.request_length,
            output_len=args.output_length,
            num_prompts=args.num_clients,
            range_ratio=range_ratio,
            tokenizer=self.tokenizer,
            dataset_path=args.dataset_path,
            random_sample=not args.disable_random_sample,
            return_text=False,
        )
        # Store per-sample output_len for first round
        first_round_output_lens = [row.output_len for row in first_round_samples]
        # r.prompt is now List[int] when return_text=False
        self.candidate_inputs = [list(i.prompt) for i in first_round_samples]

        if args.sub_question_input_length != 0:
            sub_question_input_length = args.sub_question_input_length
        else:
            sub_question_input_length = args.request_length

        num_sub_questions = sum(max(t - 1, 0) for t in self.client_total_rounds)

        self.sub_question_inputs = sample_random_requests(
            input_len=sub_question_input_length,
            output_len=args.output_length,
            num_prompts=max(num_sub_questions, 1),
            range_ratio=range_ratio,
            tokenizer=self.tokenizer,
            dataset_path=args.dataset_path,
            random_sample=not args.disable_random_sample,
            return_text=False,
        )

        if self.api_format == "openai":
            # OpenAI mode: history is a messages list for /v1/chat/completions
            initial_messages = {
                i: [
                    {
                        "role": "user",
                        "content": self.tokenizer.decode(self.candidate_inputs[i]),
                    }
                ]
                for i in range(args.num_clients)
            }
            init_requests = [
                (
                    i,
                    gen_payload_openai(
                        initial_messages[i],
                        first_round_output_lens[i],
                        self.model_path,
                    ),
                )
                for i in range(args.num_clients)
            ]
            self.client_records = {
                i: {
                    "round": 0,
                    "history": initial_messages[i],
                    "total_rounds": self.client_total_rounds[i],
                }
                for i in range(args.num_clients)
            }
        else:
            # SGLang mode: history is List[int] (token ids)
            init_requests = [
                (
                    i,
                    gen_payload(
                        self.candidate_inputs[i],
                        first_round_output_lens[i],
                        args.lora_path,
                    ),
                )
                for i in range(args.num_clients)
            ]
            self.client_records = {
                i: {
                    "round": 0,
                    "history": list(self.candidate_inputs[i]),
                    "total_rounds": self.client_total_rounds[i],
                }
                for i in range(args.num_clients)
            }
        self.ready_queue = ReadyQueue(
            init_requests=init_requests, policy=args.ready_queue_policy
        )
        self.candidate_inputs = self.candidate_inputs[args.num_clients :]

        self.response_queue = queue.Queue()
        self.pbar = tqdm(total=self.total_requests)
        self.performance_metrics = {
            "ttft": [],
            "itl": [],
            "latency": [],
            "prompt_len": [],
            "cached_tokens": [],
            "generated_len": [],
        }
        self.enable_round_barrier = args.enable_round_barrier
        if self.enable_round_barrier:
            # Add round-specific metrics while preserving the original structure
            for i in range(self.max_rounds):
                self.performance_metrics[f"round_{i}"] = {
                    "ttft": [],
                    "latency": [],
                    "prompt_len": [],
                    "cached_tokens": [],
                    "generated_len": [],
                }
        self.num_clients = args.num_clients

        self.num_rounds = self.max_rounds
        self.max_parallel = args.max_parallel
        self.output_length = args.output_length
```
**EN:** `WorkloadGenerator.__init__` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `get_tokenizer`, `sum`, `sample_random_requests`.
**CN:** `WorkloadGenerator.__init__` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `get_tokenizer`, `sum`, `sample_random_requests`。

### Lines 388-400: Async method `WorkloadGenerator.handle_request` / 异步方法 `WorkloadGenerator.handle_request`
```python
    async def handle_request(self, item):
        client_id, payload = item
        try:
            response = await self.request_func(payload, self.url, self.pbar)
            if self.pbar.n == self.pbar.total:
                self.finished_time = time.perf_counter()
            self.response_queue.put((client_id, response))
        except Exception as e:
            print(f"Request failed for client {client_id}: {e}")
            failed_response = RequestFuncOutput()
            failed_response.success = False
            failed_response.error = str(e)
            self.response_queue.put((client_id, failed_response))
```
**EN:** `WorkloadGenerator.handle_request` is a async method that measures runtime latency, throughput, or other benchmark metrics and coordinates asynchronous or parallel execution. Notable calls include `self.response_queue.put`, `self.request_func`, `time.perf_counter`.
**CN:** `WorkloadGenerator.handle_request` 是一个异步方法，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `self.response_queue.put`, `self.request_func`, `time.perf_counter`。

### Lines 402-433: Method `WorkloadGenerator.request_sender` / 方法 `WorkloadGenerator.request_sender`
```python
    def request_sender(self):
        async def request_loop():
            while True:
                if self.sent_requests - self.completed_requests < self.max_parallel:
                    new_request = self.ready_queue.pop()
                    if new_request:
                        asyncio.create_task(self.handle_request(new_request))
                        self.sent_requests += 1
                else:
                    await asyncio.sleep(0.05)
                    continue

                if self.pbar.n == self.pbar.total:
                    break

                # Calculate Poisson-distributed wait time
                if self.distribution == "poisson":
                    sleep_time = random.expovariate(self.request_rate)
                elif self.distribution == "uniform":
                    avg_interval = (
                        1.0 / self.request_rate if self.request_rate > 0 else 1.0
                    )
                    sleep_time = random.uniform(0, 2 * avg_interval)
                else:
                    raise ValueError("Invalid distribution type")
                await asyncio.sleep(sleep_time)  # Wait before sending the next request

        # Create and run the event loop for asynchronous requests
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        loop.run_until_complete(request_loop())
        loop.close()
```
**EN:** `WorkloadGenerator.request_sender` is a method that sends requests to serving or OpenAI-compatible APIs and coordinates asynchronous or parallel execution. Notable calls include `asyncio.new_event_loop`, `asyncio.set_event_loop`, `loop.run_until_complete`.
**CN:** `WorkloadGenerator.request_sender` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行。其中较关键的调用包括 `asyncio.new_event_loop`, `asyncio.set_event_loop`, `loop.run_until_complete`。

### Lines 435-544: Method `WorkloadGenerator.response_handler` / 方法 `WorkloadGenerator.response_handler`
```python
    def response_handler(self):
        next_round_reqs = []
        current_barrier_round = 0
        barrier_round_completed = 0
        while True:
            try:
                client_id, response = self.response_queue.get(
                    timeout=10
                )  # Block until response is available
                if not response.success:
                    print(f"Request failed for client {client_id}: {response.error}")
                    self.completed_requests += 1
                    continue
                # Extend history with response
                if self.api_format == "openai":
                    if response.generated_text:
                        self.client_records[client_id]["history"].append(
                            {"role": "assistant", "content": response.generated_text}
                        )
                else:
                    self.client_records[client_id]["history"].extend(
                        response.output_ids
                    )
                current_round = self.client_records[client_id]["round"]
                self.client_records[client_id]["round"] += 1
                self.performance_metrics["ttft"].append(response.ttft)
                self.performance_metrics["itl"].extend(response.itl)
                self.performance_metrics["latency"].append(response.latency)
                self.performance_metrics["prompt_len"].append(response.prompt_len)
                self.performance_metrics["cached_tokens"].append(response.cached_tokens)
                self.performance_metrics["generated_len"].append(response.generated_len)
                if self.enable_round_barrier:
                    self.performance_metrics[f"round_{current_round}"]["ttft"].append(
                        response.ttft
                    )
                    self.performance_metrics[f"round_{current_round}"][
                        "latency"
                    ].append(response.latency)
                    self.performance_metrics[f"round_{current_round}"][
                        "prompt_len"
                    ].append(response.prompt_len)
                    self.performance_metrics[f"round_{current_round}"][
                        "cached_tokens"
                    ].append(response.cached_tokens)
                    self.performance_metrics[f"round_{current_round}"][
                        "generated_len"
                    ].append(response.generated_len)
                self.completed_requests += 1

                client_total = self.client_records[client_id]["total_rounds"]
                if self.client_records[client_id]["round"] < client_total:
                    sub_q = self.sub_question_inputs.pop()
                    if self.api_format == "openai":
                        # Append sub-question as a new user message
                        sub_q_text = self.tokenizer.decode(list(sub_q.prompt))
                        self.client_records[client_id]["history"].append(
                            {"role": "user", "content": sub_q_text}
                        )
                        new_req = (
                            client_id,
                            gen_payload_openai(
                                self.client_records[client_id]["history"],
                                sub_q.output_len,
                                self.model_path,
                            ),
                        )
                    else:
                        # Append sub-question token ids to client's history
                        sub_q_ids = list(sub_q.prompt)
                        self.client_records[client_id]["history"].extend(sub_q_ids)
                        new_req = (
                            client_id,
                            gen_payload(
                                self.client_records[client_id]["history"],
                                sub_q.output_len,
                                self.lora_path,
                            ),
                        )
                    if self.enable_round_barrier:
                        next_round_reqs.append(new_req)
                    else:
                        self.ready_queue.append(new_req)

                # Barrier logic: release next round when all clients for
                # current barrier round have completed
                if (
                    self.enable_round_barrier
                    and current_barrier_round < self.max_rounds
                ):
                    barrier_round_completed += 1
                    expected = self.clients_per_round[current_barrier_round]
                    if barrier_round_completed == expected:
                        print(
                            f"\n  Barrier: round {current_barrier_round} complete "
                            f"({expected} clients), releasing {len(next_round_reqs)} "
                            f"requests for round {current_barrier_round + 1}"
                        )
                        self._send_heartbeat(input_len=100, output_len=100)
                        time.sleep(10)
                        for req in next_round_reqs:
                            self.ready_queue.append(req)
                        next_round_reqs = []
                        current_barrier_round += 1
                        barrier_round_completed = 0
            except queue.Empty:
                if self.pbar.n == self.pbar.total:
                    break
            except ValueError as e:
                print(f"Error processing response for client {client_id}: {e}")
                continue
```
**EN:** `WorkloadGenerator.response_handler` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `self.response_queue.get`, `self.performance_metrics['ttft'].append`, `self.performance_metrics['itl'].extend`.
**CN:** `WorkloadGenerator.response_handler` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `self.response_queue.get`, `self.performance_metrics['ttft'].append`, `self.performance_metrics['itl'].extend`。

### Lines 546-553: Method `WorkloadGenerator._send_heartbeat` / 方法 `WorkloadGenerator._send_heartbeat`
```python
    def _send_heartbeat(self, input_len=100, output_len=20):
        """Send a small heartbeat request to the server."""
        heartbeat_input = [1] * input_len
        payload = gen_payload(heartbeat_input, output_len, self.lora_path)
        try:
            requests.post(self.url, json=payload, timeout=30)
        except Exception as e:
            print(f"Heartbeat request failed: {e}")
```
**EN:** `WorkloadGenerator._send_heartbeat` is a method that sends requests to serving or OpenAI-compatible APIs and loads, filters, or serializes benchmark datasets. The docstring frames it as: Send a small heartbeat request to the server. Notable calls include `gen_payload`, `requests.post`, `print`.
**CN:** `WorkloadGenerator._send_heartbeat` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `gen_payload`, `requests.post`, `print`。

### Lines 555-733: Method `WorkloadGenerator.run` / 方法 `WorkloadGenerator.run`
```python
    def run(self):
        request_thread = threading.Thread(target=self.request_sender, daemon=True)
        response_thread = threading.Thread(target=self.response_handler, daemon=True)

        self.start_time = time.perf_counter()
        request_thread.start()
        response_thread.start()

        request_thread.join()
        response_thread.join()
        self.pbar.close()

        duration = self.finished_time - self.start_time
        sorted_ttft = sorted(self.performance_metrics["ttft"])
        sorted_latency = sorted(self.performance_metrics["latency"])
        sorted_itl = sorted(self.performance_metrics["itl"])
        sorted_prompt_len = sorted(self.performance_metrics["prompt_len"])
        sorted_output_len = sorted(self.performance_metrics["generated_len"])

        def percentile(sorted_vals, q):
            if not sorted_vals:
                return 0.0
            idx = int(q * len(sorted_vals))
            if idx >= len(sorted_vals):
                idx = len(sorted_vals) - 1
            return sorted_vals[idx]

        def max_or_zero(sorted_vals):
            return sorted_vals[-1] if sorted_vals else 0.0

        performance_data = {
            "summary": {
                "total_requests": len(self.performance_metrics["ttft"]),
                "request_rate": self.request_rate,
                "average_prompt_len": (
                    sum(self.performance_metrics["prompt_len"])
                    / len(self.performance_metrics["prompt_len"])
                    if self.performance_metrics["prompt_len"]
                    else 0.0
                ),
                "average_output_len": (
                    sum(self.performance_metrics["generated_len"])
                    / len(self.performance_metrics["generated_len"])
                    if self.performance_metrics["generated_len"]
                    else 0.0
                ),
                "p90_prompt_len": percentile(sorted_prompt_len, 0.9),
                "p99_prompt_len": percentile(sorted_prompt_len, 0.99),
                "p90_output_len": percentile(sorted_output_len, 0.9),
                "p99_output_len": percentile(sorted_output_len, 0.99),
                "average_ttft": sum(self.performance_metrics["ttft"])
                / len(self.performance_metrics["ttft"]),
                "p90_ttft": percentile(sorted_ttft, 0.9),
                "p99_ttft": percentile(sorted_ttft, 0.99),
                "median_ttft": percentile(sorted_ttft, 0.5),
                "max_ttft": max_or_zero(sorted_ttft),
                "average_itl": (
                    sum(self.performance_metrics["itl"])
                    / len(self.performance_metrics["itl"])
                    if self.performance_metrics["itl"]
                    else 0.0
                ),
                "p90_itl": percentile(sorted_itl, 0.9),
                "p99_itl": percentile(sorted_itl, 0.99),
                "median_itl": percentile(sorted_itl, 0.5),
                "max_itl": max_or_zero(sorted_itl),
                "average_latency": sum(self.performance_metrics["latency"])
                / len(self.performance_metrics["latency"]),
                "p90_latency": percentile(sorted_latency, 0.9),
                "p99_latency": percentile(sorted_latency, 0.99),
                "median_latency": percentile(sorted_latency, 0.5),
                "max_latency": max_or_zero(sorted_latency),
                "input_token_throughput": sum(self.performance_metrics["prompt_len"])
                / duration,
                "output_token_throughput": sum(
                    self.performance_metrics["generated_len"]
                )
                / duration,
                "throughput": self.pbar.total / duration,
                "cache_hit_rate": (
                    0
                    if sum(self.performance_metrics["prompt_len"]) == 0
                    else sum(self.performance_metrics["cached_tokens"])
                    / sum(self.performance_metrics["prompt_len"])
                ),
            },
        }
        if self.enable_round_barrier:
            performance_data["round"] = {}
            for round_num in range(self.num_rounds):
                round_key = f"round_{round_num}"
                round_metrics = self.performance_metrics[round_key]
                performance_data["round"][round_key] = {
                    "average_ttft": (
                        sum(round_metrics["ttft"]) / len(round_metrics["ttft"])
                        if round_metrics["ttft"]
                        else 0
                    ),
                    "cache_hit_rate": (
                        0
                        if sum(round_metrics["prompt_len"]) == 0
                        else sum(round_metrics["cached_tokens"])
                        / sum(round_metrics["prompt_len"])
                    ),
                    "request_count": len(round_metrics["ttft"]),
                }
        print("All requests completed")
        print("Performance metrics summary:")
        print(
            f"  Total requests: {performance_data['summary']['total_requests']} at {performance_data['summary']['request_rate']} requests per second"
        )
        print(
            f"  Average Prompt Length: {performance_data['summary']['average_prompt_len']:.2f} tokens"
        )
        print(
            f"  Average Output Length: {performance_data['summary']['average_output_len']:.2f} tokens"
        )
        print(
            f"  P90 Prompt Length: {performance_data['summary']['p90_prompt_len']:.0f} tokens"
        )
        print(
            f"  P99 Prompt Length: {performance_data['summary']['p99_prompt_len']:.0f} tokens"
        )
        print(
            f"  P90 Output Length: {performance_data['summary']['p90_output_len']:.0f} tokens"
        )
        print(
            f"  P99 Output Length: {performance_data['summary']['p99_output_len']:.0f} tokens"
        )
        print(f"  Average TTFT: {performance_data['summary']['average_ttft']:.2f}")
        print(f"  P90 TTFT: {performance_data['summary']['p90_ttft']:.2f}")
        print(f"  P99 TTFT: {performance_data['summary']['p99_ttft']:.2f}")
        print(f"  Median TTFT: {performance_data['summary']['median_ttft']:.2f}")
        print(f"  Max TTFT: {performance_data['summary']['max_ttft']:.2f}")
        print(f"  Average ITL: {performance_data['summary']['average_itl']:.4f}")
        print(f"  P90 ITL: {performance_data['summary']['p90_itl']:.4f}")
        print(f"  P99 ITL: {performance_data['summary']['p99_itl']:.4f}")
        print(f"  Median ITL: {performance_data['summary']['median_itl']:.4f}")
        print(f"  Max ITL: {performance_data['summary']['max_itl']:.4f}")
        print(
            f"  Average latency: {performance_data['summary']['average_latency']:.2f}"
        )
        print(f"  P90 latency: {performance_data['summary']['p90_latency']:.2f}")
        print(f"  P99 latency: {performance_data['summary']['p99_latency']:.2f}")
        print(f"  Median latency: {performance_data['summary']['median_latency']:.2f}")
        print(f"  Max latency: {performance_data['summary']['max_latency']:.2f}")
        print(
            f"  Input token throughput: {performance_data['summary']['input_token_throughput']:.2f} tokens per second"
        )
        print(
            f"  Output token throughput: {performance_data['summary']['output_token_throughput']:.2f} tokens per second"
        )
        print(
            f"  Request Throughput: {performance_data['summary']['throughput']:.2f} requests per second"
        )
        print(f"  Cache Hit Rate: {performance_data['summary']['cache_hit_rate']:.6f}")

        if self.enable_round_barrier:
            # Print round-basedsummary
            print("Per-round metrics:")
            if "round" in performance_data:
                for round_num in range(self.num_rounds):
                    round_key = f"round_{round_num}"
                    if round_key in performance_data["round"]:
                        round_data = performance_data["round"][round_key]
                        avg_ttft = round_data["average_ttft"]
                        cache_hit_rate = round_data["cache_hit_rate"]
                        request_count = round_data["request_count"]
                        clients_in_round = self.clients_per_round[round_num]
                        print(
                            f"  Round {round_num}: Average TTFT = {avg_ttft:.2f}s, "
                            f"Cache Hit Rate = {cache_hit_rate:.6f} "
                            f"({request_count} requests, "
                            f"{clients_in_round} clients)"
                        )
                    else:
                        print(f"  Round {round_num}: No requests completed")

        return performance_data
```
**EN:** `WorkloadGenerator.run` is a method that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and processes tokenized prompts or decoded outputs. It returns `performance_data` to the caller. Notable calls include `threading.Thread`, `time.perf_counter`, `request_thread.start`.
**CN:** `WorkloadGenerator.run` 是一个方法，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、处理分词后的提示词或解码后的输出。它会向调用方返回 `performance_data`。其中较关键的调用包括 `threading.Thread`, `time.perf_counter`, `request_thread.start`。

### Lines 736-755: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    args = parse_args()
    flush_cache_url = f"http://{args.host}:{args.port}/flush_cache"

    random.seed(args.seed)
    np.random.seed(args.seed)

    if args.disable_auto_run:
        print("Running with specified request rate...")
        request_rates = [args.request_rate]
    else:
        print("Auto-running with different request rates...")
        request_rates = [16, 14, 12, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

    for rate in request_rates:
        args.request_rate = rate
        requests.post(flush_cache_url)
        time.sleep(1)
        performance_data = WorkloadGenerator(args).run()
        log_to_jsonl_file(performance_data, args.log_file, tag=args.tag)
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
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `queue`, `random`, `threading`, `time`, `datetime`
- **Third-party / 第三方依赖**: `numpy`, `requests`, `tqdm.asyncio`
- **Internal / 项目内部依赖**: `sglang.bench_serving`, `sglang.benchmark.datasets.random`, `sglang.benchmark.utils`, `sglang.test.kits.cache_hit_kit`
